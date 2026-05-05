<!--
  ChartPlaceholder.svelte
  Development placeholder for charts, maps, and photos that haven't been built yet.
  Replace this block in doc.blocks.json with the real component when ready.

  Props:
    label  — descriptive name shown inside the box (e.g. "Chart: Mortality rate by institution")
    type   — 'chart' | 'map' | 'photo' — changes the icon
    size   — 'full' | 'large' | 'fit'  — matches the sizing convention of other embeds
    aspect — CSS aspect-ratio string (default "16/9")
-->
<script lang="ts">
  export let label: string = 'Graphic coming soon';
  export let type: 'chart' | 'map' | 'photo' = 'chart';
  export let size: 'full' | 'large' | 'fit' = 'large';
  export let aspect: string = '16/9';

  const icons: Record<string, string> = {
    chart: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <rect x="3" y="3" width="18" height="18" rx="2"/>
      <path d="M7 16l3-4 3 3 3-5"/>
    </svg>`,
    map: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <polygon points="3,6 9,3 15,6 21,3 21,18 15,21 9,18 3,21"/>
      <line x1="9" y1="3" x2="9" y2="18"/>
      <line x1="15" y1="6" x2="15" y2="21"/>
    </svg>`,
    photo: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
      <rect x="3" y="3" width="18" height="18" rx="2"/>
      <circle cx="8.5" cy="8.5" r="1.5"/>
      <polyline points="21,15 16,10 5,21"/>
    </svg>`
  };
</script>

{#if size === 'full'}
  <figure class="placeholder full-bleed" style="--aspect: {aspect}">
    <div class="inner">
      <span class="icon">{@html icons[type]}</span>
      <p class="lbl">{label}</p>
      <p class="hint">placeholder — swap for real component in doc.blocks.json</p>
    </div>
  </figure>

{:else if size === 'large'}
  <figure class="placeholder full-bleed" style="--aspect: {aspect}">
    <div class="container-fluid">
      <div class="row justify-content-center">
        <div class="col-12 col-lg-10 col-xxl-8">
          <div class="inner">
            <span class="icon">{@html icons[type]}</span>
            <p class="lbl">{label}</p>
            <p class="hint">placeholder — swap for real component in doc.blocks.json</p>
          </div>
        </div>
      </div>
    </div>
  </figure>

{:else}
  <figure class="placeholder" style="--aspect: {aspect}">
    <div class="inner">
      <span class="icon">{@html icons[type]}</span>
      <p class="lbl">{label}</p>
      <p class="hint">placeholder — swap for real component in doc.blocks.json</p>
    </div>
  </figure>
{/if}

<style>
  .placeholder {
    margin: 1.5rem 0;
    font-family: 'Crimson Text', Georgia, serif;
  }

  .inner {
    aspect-ratio: var(--aspect, 16/9);
    background: #ede8de;
    border: 2px dashed rgba(200, 150, 10, 0.45);
    border-radius: 6px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 0.6rem;
    padding: 1.5rem;
    text-align: center;
    color: #8a6010;
  }

  .icon {
    display: block;
    width: 40px;
    height: 40px;
    opacity: 0.55;
  }

  .icon :global(svg) {
    width: 100%;
    height: 100%;
  }

  .lbl {
    margin: 0;
    font-size: clamp(0.9rem, 1.5vw, 1.1rem);
    font-weight: 600;
    color: #6a4a08;
    line-height: 1.3;
  }

  .hint {
    margin: 0;
    font-size: 0.72rem;
    font-style: italic;
    color: rgba(106, 74, 8, 0.55);
    letter-spacing: 0.03em;
  }
</style>
