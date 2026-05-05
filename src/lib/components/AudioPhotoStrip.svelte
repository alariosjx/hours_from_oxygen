<script lang="ts">
  import { onDestroy } from 'svelte';
  import { browser } from '$app/environment';

  type PhotoStep = {
    img: string;
    alt?: string;
    timing: number | string;
    duration?: number | string;
    ease?: string;
  };

  export let audio: string = '';
  export let steps: PhotoStep[] | undefined = undefined;
  export let bodyHtml: string | undefined = undefined;

  function normalizeStaticPath(raw: string): string {
    const s = raw.trim();
    if (!s || /^https?:\/\//i.test(s)) return s;
    return s.startsWith('/') ? s : '/' + s;
  }

  // Parses a JSON photo-step array embedded inside HTML (from bodyHtml shortcode body).
  function parseStepsFromHtml(html: string): PhotoStep[] | null {
    const txt = html
      .replace(/<br\s*\/?>/gi, '\n')
      .replace(/<[^>]+>/g, '')
      .replace(/&[^;]+;/g, ' ');
    const start = txt.indexOf('[');
    const end = txt.lastIndexOf(']');
    if (start === -1 || end === -1 || end <= start) return null;
    try {
      const arr = JSON.parse(txt.slice(start, end + 1));
      if (!Array.isArray(arr)) return null;
      return arr
        .map((x: any) => ({
          img: String(x.img ?? ''),
          alt: typeof x.alt === 'string' ? x.alt : undefined,
          timing: parseFloat(String(x.timing ?? '0')),
          duration: parseFloat(String(x.duration ?? '0.5')),
          ease: typeof x.ease === 'string' ? x.ease : 'easeIn'
        }))
        .filter((s: PhotoStep) => s.img);
    } catch {
      return null;
    }
  }

  $: resolvedSteps =
    (steps && steps.length > 0 ? steps : null) ??
    (bodyHtml ? parseStepsFromHtml(bodyHtml) : null) ??
    [];

  $: audioSrc = audio ? normalizeStaticPath(audio) : '';

  // ── Playback state ────────────────────────────────────────────
  let audioEl: HTMLAudioElement | null = null;
  let playing = false;
  let currentTime = 0;
  let totalDuration = 0;
  let currentIdx = 0;
  let rafId = 0;

  function getActiveIndex(t: number): number {
    let idx = 0;
    for (let i = 0; i < resolvedSteps.length; i++) {
      if (parseFloat(String(resolvedSteps[i].timing)) <= t) idx = i;
    }
    return idx;
  }

  function rafTick() {
    if (!audioEl || audioEl.paused) return;
    currentTime = audioEl.currentTime;
    currentIdx = getActiveIndex(currentTime);
    rafId = requestAnimationFrame(rafTick);
  }

  function togglePlay() {
    if (!audioEl) return;
    if (playing) {
      audioEl.pause();
    } else {
      audioEl.play().catch(() => {});
    }
  }

  function seek(e: Event) {
    if (!audioEl) return;
    audioEl.currentTime = Number((e.target as HTMLInputElement).value);
  }

  function onPlay() {
    playing = true;
    cancelAnimationFrame(rafId);
    rafId = requestAnimationFrame(rafTick);
  }

  function onPause() {
    playing = false;
    cancelAnimationFrame(rafId);
  }

  function onEnded() {
    playing = false;
    cancelAnimationFrame(rafId);
  }

  function onTimeUpdate() {
    if (audioEl) {
      currentTime = audioEl.currentTime;
      currentIdx = getActiveIndex(currentTime);
    }
  }

  function onLoadedMetadata() {
    if (audioEl) totalDuration = audioEl.duration;
  }

  function formatTime(t: number): string {
    if (!isFinite(t)) return '0:00';
    const m = Math.floor(t / 60);
    const s = Math.floor(t % 60);
    return `${m}:${s.toString().padStart(2, '0')}`;
  }

  onDestroy(() => {
    cancelAnimationFrame(rafId);
  });
</script>

{#if resolvedSteps.length > 0}
  <figure class="audiostrip">
    <!-- Photo display: one layer per step, CSS crossfade driven by .active -->
    <div class="photo-stage" aria-live="polite">
      {#each resolvedSteps as step, i (i)}
        <div class="photo-layer" class:active={i === currentIdx} aria-hidden={i !== currentIdx}>
          <img
            src={normalizeStaticPath(step.img)}
            alt={step.alt ?? ''}
            loading={i === 0 ? 'eager' : 'lazy'}
          />
        </div>
      {/each}
    </div>

    <!-- Controls bar: play/pause · time · scrubber · total -->
    <div class="controls">
      <button class="play-btn" on:click={togglePlay} aria-label={playing ? 'Pause' : 'Play'}>
        {#if playing}
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <rect x="6" y="4" width="4" height="16" rx="1" fill="currentColor" />
            <rect x="14" y="4" width="4" height="16" rx="1" fill="currentColor" />
          </svg>
        {:else}
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <polygon points="5,3 19,12 5,21" fill="currentColor" />
          </svg>
        {/if}
      </button>

      <span class="time-cur">{formatTime(currentTime)}</span>

      <input
        class="scrubber"
        type="range"
        min="0"
        max={totalDuration || 1}
        step="0.1"
        value={currentTime}
        on:input={seek}
        aria-label="Audio position"
      />

      <span class="time-total">{formatTime(totalDuration)}</span>
    </div>

    <!-- Caption shows the alt text of the current photo -->
    {#if resolvedSteps[currentIdx]?.alt}
      <figcaption class="strip-caption">{resolvedSteps[currentIdx].alt}</figcaption>
    {/if}

    <!-- Step dots: one per photo, gold when active -->
    {#if resolvedSteps.length > 1}
      <div class="step-dots" aria-hidden="true">
        {#each resolvedSteps as _, i (i)}
          <span class="dot" class:active={i === currentIdx}></span>
        {/each}
      </div>
    {/if}
  </figure>

  <!-- Native audio element (hidden) — only rendered in browser -->
  {#if browser && audioSrc}
    <!-- svelte-ignore a11y_media_has_caption -->
    <audio
      bind:this={audioEl}
      src={audioSrc}
      preload="metadata"
      on:play={onPlay}
      on:pause={onPause}
      on:ended={onEnded}
      on:timeupdate={onTimeUpdate}
      on:loadedmetadata={onLoadedMetadata}
    ></audio>
  {/if}
{/if}

<style>
  .audiostrip {
    margin: 2rem 0;
    font-family: 'Crimson Text', Georgia, serif;
  }

  /* Photo stage: stacks all layers, crossfades via opacity */
  .photo-stage {
    position: relative;
    width: 100%;
    aspect-ratio: 4 / 3;
    background: #1a0848;
    overflow: hidden;
    border-radius: 6px 6px 0 0;
  }

  .photo-layer {
    position: absolute;
    inset: 0;
    opacity: 0;
    transition: opacity 0.6s ease;
  }

  .photo-layer.active {
    opacity: 1;
  }

  .photo-layer img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }

  /* Controls bar */
  .controls {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 0.6rem 0.75rem;
    background: #f5f0e8;
    border: 1px solid rgba(200, 150, 10, 0.25);
    border-top: none;
  }

  .play-btn {
    flex-shrink: 0;
    width: 36px;
    height: 36px;
    border-radius: 50%;
    background: #c8960a;
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    padding: 0;
    transition: background 0.15s;
  }

  .play-btn:hover {
    background: #a07008;
  }

  .play-btn svg {
    width: 16px;
    height: 16px;
  }

  .time-cur,
  .time-total {
    font-size: 0.78rem;
    color: #5a4020;
    white-space: nowrap;
    min-width: 2.6rem;
    text-align: center;
    font-variant-numeric: tabular-nums;
  }

  .scrubber {
    flex: 1;
    -webkit-appearance: none;
    appearance: none;
    height: 4px;
    border-radius: 2px;
    background: #d8c9a8;
    outline: none;
    cursor: pointer;
  }

  .scrubber::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #c8960a;
    cursor: pointer;
  }

  .scrubber::-moz-range-thumb {
    width: 14px;
    height: 14px;
    border-radius: 50%;
    background: #c8960a;
    border: none;
    cursor: pointer;
  }

  /* Caption under controls */
  .strip-caption {
    display: block;
    font-size: 0.82rem;
    color: #5a4020;
    font-style: italic;
    padding: 0.45rem 0.1rem 0;
    margin: 0;
    min-height: 1.4em;
    border-top: none;
  }

  /* Step dots */
  .step-dots {
    display: flex;
    gap: 6px;
    justify-content: center;
    padding: 0.55rem 0 0;
  }

  .dot {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    background: #d8c9a8;
    transition: background 0.3s;
    flex-shrink: 0;
  }

  .dot.active {
    background: #c8960a;
  }

  @media (max-width: 640px) {
    .photo-stage {
      aspect-ratio: 3 / 2;
    }
  }
</style>
