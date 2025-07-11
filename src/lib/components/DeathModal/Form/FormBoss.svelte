<script>
  import { slide } from 'svelte/transition'
  import { onMount, getContext } from 'svelte'

  import { fetchRoute } from '$utils/fetchers'
  import { read, readdata, readStarter, getGameStore } from '$lib/store'

  import { Expanded as Games } from '$lib/data/games.js'
  import { AutoComplete, Picture, PIcon } from '$c/core'
  import TypeBadge from '$c/type-badge.svelte'

  export let bossPlaceholder = 'Opponent Trainer'
  export let pokePlaceholder = 'Opponent Pokémon'
  export let movePlaceholder = 'Moved used to defeat your Pokémon'

  export let result = {}
  let ctx = {}

  const { getLeague, getPkmns } = getContext('game')

  let league,
    gyms = []
  onMount(() => {
    const [, key, gameId] = readdata()

    getGameStore(gameId).subscribe(
      read((data) => {
        const starter = readStarter(data)

        getLeague(key, starter).then((l) => (league = l))
        fetchRoute(Games[key].pid).then(
          (data = []) => (gyms = data.filter((g) => g.type === 'gym'))
        )
      })
    )
  })

  let leaderMons = [],
    leaderRef = {},
    opponents
  $: getPkmns(leaderMons.map((p) => p.name)).then((ref) => (leaderRef = ref))
  $: leaderMons =
    league && ctx.trainer ? league[ctx.trainer.value]?.pokemon || [] : []
  $: opponents = leaderMons.map((m) => ({ ...m, ...leaderRef[m.name] }))

  const resetattack = () => {
    delete ctx.attack
    delete ctx.attackSearch
  }

  const resetopponent = (t) => {
    delete ctx.opponent
    delete ctx.opponentSearch
    resetattack()
  }

  const bossImage = (data) => {
    let img = typeof data.img === 'string' ? { src: data.img } : data.img

    if(img.src.startsWith('/leaders/')) {
      img.src = `/assets/img${img.src}`;
    }
    if(img.src.startsWith('/sprite/')) {
      img.src = `/assets/img/pokemon/base-${img.src.slice(8)}`;
    }

    return img;
  }

  // Chang detection between the opponent selection & trainer
  let prevTrainer
  $: {
    if (prevTrainer != ctx?.trainer?.value) {
      resetopponent()
      prevTrainer = ctx?.trainer?.value
    }
  }

  // Format ctx data into common schema
  $: result = {
    location: {
      name: ctx?.trainer?.name
    },
    attack: {
      name: ctx?.attack?.name,
      type: ctx?.attack?.type
    },
    trainer: {
      name: ctx?.trainer?.boss,
      id: ctx?.trainer?.value,
      speciality: ctx?.trainer?.value
        ? league[ctx?.trainer?.value]?.speciality
        : null,
      type: ctx?.trainer?.group
    },
    pokemon: {
      name: ctx?.opponent?.name,
      types: ctx?.opponent?.types,
      id: ctx?.opponent?.alias
    }
  }
</script>

<slot name="label" />

<!-- Boss selector -->
<AutoComplete
  bind:selected={ctx.trainer}
  name={bossPlaceholder}
  placeholder={bossPlaceholder}
  itemF={(_) => gyms}
  labelF={(_) => _.boss}
>
  <span class="item item--leader" slot="option" let:option={item} let:label>
    <span>
      {@html label}
      {#if item.name.startsWith('Route')} on {item.name}{/if}
      {#if !item.name.startsWith('Route')} at {item.name}{/if}
    </span>

    {#if league && (league[item.value].img?.src || league[item.value].img)}
      <Picture
        src={bossImage(league[item.value]).src}
        alt={item.name}
        pixelated
        className="w-16 md:w-18 translate-x-1/4 float-right"
        aspect="72x52"
      />
    {/if}
  </span>
</AutoComplete>

<!-- Team selcetor -->
{#if ctx?.trainer}
  <div in:slide class="w-full">
    <AutoComplete
      on:change={resetattack}
      bind:search={ctx.opponentSearch}
      bind:selected={ctx.opponent}
      name={pokePlaceholder}
      placeholder={pokePlaceholder}
      itemF={(_) => opponents}
      labelF={(_) => _.name}
      className="w-full"
    >
      <span class="item" slot="option" let:option={item} let:label>
        <span>{@html label}</span>
        <PIcon className="float-right -mr-4" name={item.alias} />
      </span>
    </AutoComplete>
  </div>
{/if}

{#if ctx?.opponent}
  <div in:slide class="w-full">
    <AutoComplete
      bind:search={ctx.attackSearch}
      bind:selected={ctx.attack}
      name={movePlaceholder}
      placeholder={movePlaceholder}
      itemF={(_) => ctx?.opponent.moves}
      labelF={(_) => _.name}
      class="w-full"
    >
      <span class="item" slot="option" let:option={item} let:label>
        <span>{@html label}</span>
        <div class="inline-flex gap-x-1">
          <TypeBadge type={item.type} />
          <TypeBadge type={item.damage_class} />
        </div>
      </span>
    </AutoComplete>
  </div>
{/if}

<!-- Move selcetor -->
<style>
  span.item--leader > span {
    @apply line-clamp-1;
  }
  span.item {
    @apply inline-flex h-4
    w-full items-center justify-between
    overflow-hidden px-3
    py-4 text-xs md:h-8 md:py-5;
  }
</style>
