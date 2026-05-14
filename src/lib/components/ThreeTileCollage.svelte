<!-- src/lib/components/ThreePhotoCollage.svelte
  Three-tile collage — supports portrait and landscape photos.
  Tile layout:
    [  tile-1 tall  ] [  tile-2 square  ]
    [  tile-1 tall  ] [  tile-3 wide    ]

  USAGE:
    <ThreePhotoCollage
      images="/grandparents/jose.jpg, /grandparents/guty.jpg, /grandparents/family.jpg"
      alts="Jose Jimenez, Augustina Delgado, Familia Jimenez-Delgado"
      caption="Jose y Augustina, Valle Verde."
    />

  Put photos in static/grandparents/
-->

<script lang="ts">
	import { base } from '$app/paths';
	import { onMount, onDestroy, tick } from 'svelte';
	import { browser } from '$app/environment';

	export let images: string = '';
	export let alts: string = '';
	export let caption: string = '';
	export let width: string = '95vw';

	// ── Path normalization for deployment (handles base path / GitHub Pages) ──
	function normalizePath(raw: string): string {
		const s = raw.trim();
		if (!s) return '';
		if (/^data:/i.test(s) || /^https?:\/\//i.test(s)) return s;
		if (s.startsWith('/')) return base + s;
		return base + '/' + s;
	}

	function inferType(src: string): 'video' | 'image' {
		return /\.(mp4|webm|ogg|mov)$/i.test(src.trim()) ? 'video' : 'image';
	}

	type Tile = { src: string; alt: string; type: 'video' | 'image' };

	$: tiles = ((): Tile[] => {
		const srcs = images.split(',').map((s) => s.trim());
		const altList = alts.split(',').map((s) => s.trim());
		return srcs.map((src, i) => ({
			src: normalizePath(src),
			alt: altList[i] ?? '',
			type: inferType(src)
		}));
	})();

	// ── Lazy load ──
	let sectionEl: HTMLElement;
	let isVisible = false;
	let observer: IntersectionObserver | null = null;
	let videoEls: (HTMLVideoElement | null)[] = [null, null, null];

	function playAll() {
		videoEls.forEach((v) => {
			if (v) v.play().catch(() => {});
		});
	}

	// ── Scroll reveal state ──
	let revealed = false;

	onMount(() => {
		if (!browser) return;

		observer = new IntersectionObserver(
			async (entries) => {
				if (!entries[0].isIntersecting) return;
				isVisible = true;
				observer?.disconnect();
				observer = null;
				await tick();
				playAll();
				// Small delay so tiles stagger in after paint
				requestAnimationFrame(() => {
					setTimeout(() => {
						revealed = true;
					}, 60);
				});
			},
			{ rootMargin: '0px 0px -80px 0px' }
		);

		if (sectionEl) observer.observe(sectionEl);
	});

	onDestroy(() => {
		observer?.disconnect();
	});
</script>

<div class="collage-bleed" bind:this={sectionEl}>
	<div class="collage-inner" style="width: {width}">
		{#if isVisible}
			<div class="grid" class:revealed>
				<!-- Left column: tall portrait tile -->
				<div class="left-col tile-wrap delay-0">
					<div class="tile tile-1">
						{#if tiles[0]?.type === 'video'}
							<video
								bind:this={videoEls[0]}
								src={tiles[0].src}
								aria-label={tiles[0].alt}
								autoplay
								muted
								loop
								playsinline
							>
								<track kind="captions" />
							</video>
						{:else}
							<img src={tiles[0]?.src ?? ''} alt={tiles[0]?.alt ?? ''} loading="lazy" />
						{/if}
					</div>
				</div>

				<!-- Right column: square + wide, stacked -->
				<div class="right-col">
					<div class="tile-wrap delay-1">
						<div class="tile tile-2">
							{#if tiles[1]?.type === 'video'}
								<video
									bind:this={videoEls[1]}
									src={tiles[1].src}
									aria-label={tiles[1].alt}
									autoplay
									muted
									loop
									playsinline
								>
									<track kind="captions" />
								</video>
							{:else}
								<img src={tiles[1]?.src ?? ''} alt={tiles[1]?.alt ?? ''} loading="lazy" />
							{/if}
						</div>
					</div>

					<div class="tile-wrap delay-2">
						<div class="tile tile-3">
							{#if tiles[2]?.type === 'video'}
								<video
									bind:this={videoEls[2]}
									src={tiles[2].src}
									aria-label={tiles[2].alt}
									autoplay
									muted
									loop
									playsinline
								>
									<track kind="captions" />
								</video>
							{:else}
								<img src={tiles[2]?.src ?? ''} alt={tiles[2]?.alt ?? ''} loading="lazy" />
							{/if}
						</div>
					</div>

					<!-- Caption sits below right column -->
					<!-- eslint-disable-next-line svelte/no-at-html-tags -->
					<p class="caption">{@html caption}</p>
				</div>
			</div>
		{:else}
			<!-- Placeholder maintains height before images load -->
			<div class="grid">
				<div class="left-col">
					<div class="tile tile-1 placeholder-tile"></div>
				</div>
				<div class="right-col">
					<div class="tile tile-2 placeholder-tile"></div>
					<div class="tile tile-3 placeholder-tile"></div>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	/* ── Full bleed ── */
	.collage-bleed {
		width: 100vw;
		position: relative;
		left: 50%;
		margin-left: -50vw;
		margin-right: -50vw;
		margin-top: 6rem;
		margin-bottom: 6rem;
		display: flex;
		justify-content: center;
		box-sizing: border-box;
	}

	.collage-inner {
		--gap: 10px;
		box-sizing: border-box;
	}

	.grid {
		display: flex;
		gap: var(--gap);
		width: 100%;
		align-items: flex-start;
	}

	/* ── Columns ── */
	.left-col {
		flex: 0 0 52%;
		min-width: 0;
	}

	.right-col {
		flex: 1;
		min-width: 0;
		display: flex;
		flex-direction: column;
		gap: var(--gap);
	}

	/* ── Tile sizing ──
       tile-1: tall portrait
       tile-2: square
       tile-3: landscape / wide
    */
	.tile {
		overflow: hidden;
		margin: 0;
		padding: 0;
		outline: 1px solid rgba(200, 150, 10, 0.12);
	}

	.tile img,
	.tile video {
		width: 100%;
		height: 100%;
		object-fit: cover;
		display: block;
		filter: saturate(0.88) contrast(1.04);
		transition: filter 0.5s ease;
	}

	.tile img:hover,
	.tile video:hover {
		filter: saturate(1) contrast(1);
	}

	.tile-1 {
		aspect-ratio: 3 / 4;
	} /* portrait */
	.tile-2 {
		aspect-ratio: 1 / 1;
	} /* square */
	.tile-3 {
		aspect-ratio: 16 / 9;
	} /* landscape */

	/* ── Caption ── */
	.caption {
		margin: 1rem 0 0 0;
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.85rem;
		font-style: italic;
		line-height: 1.65;
		color: #5a4a3a;
	}

	/* ── Scroll reveal — tiles slide up and fade in with stagger ── */
	.tile-wrap {
		opacity: 0;
		transform: translateY(28px);
		transition:
			opacity 0.7s ease,
			transform 0.7s ease;
	}

	.revealed .tile-wrap {
		opacity: 1;
		transform: translateY(0);
	}

	.revealed .delay-0 {
		transition-delay: 0ms;
	}
	.revealed .delay-1 {
		transition-delay: 120ms;
	}
	.revealed .delay-2 {
		transition-delay: 240ms;
	}

	.placeholder-tile {
		background: rgba(200, 150, 10, 0.05);
	}

	/* ── Mobile ── */
	@media (max-width: 768px) {
		.collage-bleed {
			width: 100%;
			position: static;
			left: auto;
			margin-left: 0;
			margin-right: 0;
		}
		.collage-inner {
			width: 100% !important;
		}
		.grid {
			flex-direction: column;
		}
		.left-col,
		.right-col {
			width: 100%;
			flex: none;
		}
		.tile-1,
		.tile-2,
		.tile-3 {
			aspect-ratio: 4 / 3;
		}
	}
</style>
