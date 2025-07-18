<script>
  import { FormalInvitation as Pattern } from '$lib/utils/pattern'
  import { bossToImage } from '$lib/utils/rewrites.js'
  import { Wrapper as SettingWrapper } from '$lib/components/Settings'
  import { color } from '$lib/data/colors.ts'

  export let atkType = null,
    boss = null

  export let atkStats
  export let defStats

  const rColDefault = '#cd4184',
    lColDefault = '#2f78df'

  const lCol = (tid) => (atkType ? color(atkType, tid) : lColDefault)
  const rCol = (tid) =>
    boss?.speciality ? color(boss?.speciality, tid) : rColDefault

  const center = 50

  let img = bossToImage(boss);
</script>

{#if boss?.img}
  <img
    alt='Boss image for {boss.name}'
    class="absolute right-4 -top-5 -z-10 scale-150 md:-top-7 md:right-8 md:scale-200"
    src="{img.src}.png"
  />
{/if}

<SettingWrapper id="theme" let:setting={themeId}>
  <!-- Desktop gradients inlined -->
  <div
    class="bbheading desk-only transition"
    style="background: linear-gradient(90deg, {lCol(themeId)} 0%, {lCol(
      themeId
    )} {center - 1}%, {rCol(themeId)} {center + 1}%, {rCol(themeId)});"
  >
    <div
      class="absolute inset-0 opacity-30 mix-blend-overlay"
      style={`background-image: url("${Pattern('white')}")`}
    />
  </div>

  <div
    class="anim-bg bbheading bbanim desk-only"
    style="background: linear-gradient(180deg, transparent 0%, white 48%, white 52%, transparent 100%);"
  />

  <div
    class="bbheading desk-only"
    style="background: linear-gradient(60deg, transparent 25%, var(--bbbgcol) {center -
      2}%, var(--bbbgcol) {center}%, var(--bbbgcol) {center +
      2}%, transparent 75%);"
  />

  <div
    class="desk-only absolute left-0 top-24 right-0 z-50 h-[2px] -translate-y-4 md:top-28"
    style="background: linear-gradient(90deg, {lCol(themeId)}, {rCol(
      themeId
    )});"
  />

  <!-- Base stat totals -->

  <div style="--b-col: {lCol(themeId)}" class="bst"><b>{atkStats}</b></div>
  <div style="--b-col: {rCol(themeId)}" class="bst"><b>{defStats}</b></div>
</SettingWrapper>

<style>
  img {
    image-rendering: pixelated;
  }

  .desk-only {
    @apply flex;
  }

  .bst,
  .bst b::before {
    @apply z-50 bg-white;
  }

  :global(.dark) .bst,
  :global(.dark) .bst b::before {
    @apply bg-gray-900;
  }

  .bst {
    border-color: var(--b-col);
    @apply absolute top-20 left-2 z-50 translate-y-1 scale-90 rounded-lg border-2 px-2 md:left-5;
  }

  .bst b {
    @apply z-50;
  }

  .bst b::before {
    content: '';
    z-index: -10;
    width: calc(100% + 4px);
    left: -2px;
    bottom: -2px;
    height: 14px;
    @apply absolute;
  }

  .bst {
    @apply max-md:top-16;
  }

  .bst ~ .bst {
    @apply left-auto right-2 md:right-5;
  }

  :root {
    --bbbgcol: white;
  }

  :global(.dark) {
    --bbbgcol: theme('colors.gray.900');
  }

  .bbheading {
    @apply absolute inset-0 top-6 h-20 w-full translate-y-2 overflow-hidden rounded-t-lg;
  }

  @media (min-width: theme('screens.md')) {
    .bbheading {
      @apply inset-0 top-6 h-20;
    }
  }

  .bbanim {
    animation: animVsPulse 6s ease infinite forwards;
  }

  @keyframes animVsPulse {
    0%,
    100% {
      opacity: 90%;
    }
    50% {
      opacity: 50%;
    }
  }
</style>
