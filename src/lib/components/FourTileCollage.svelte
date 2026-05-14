<!-- src/lib/components/FourPhotoCollage.svelte
  Four-tile collage — supports portrait, square, and landscape photos.
  Tile layout:
    [ caption ] [   tile-1 landscape   ] [ margin ]
    [ tile-2 portrait ] [ tile-3 sq ] [ tile-4 sq ]

  USAGE:
    <FourPhotoCollage
      images="/g/jose.jpg, /g/guty.jpg, /g/home.jpg, /g/valle.jpg"
      alts="Jose, Guty, Casa, Valle Verde"
      caption="Jose y Augustina, Valle Verde, Nayarit."
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
	export let layout: 'default' | 'sketch' = 'default';

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

	let sectionEl: HTMLElement;
	let isVisible = false;
	let observer: IntersectionObserver | null = null;
	let videoEls: (HTMLVideoElement | null)[] = [null, null, null, null];
	let revealed = false;

	function playAll() {
		videoEls.forEach((v) => {
			if (v) v.play().catch(() => {});
		});
	}

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
			<div class:revealed class:sketch={layout === 'sketch'}>
				<!-- Top row: caption + tile-1 landscape + right margin -->
				<div class="row row-top">
					<div class="caption-block tile-wrap delay-0">
						<!-- eslint-disable-next-line svelte/no-at-html-tags -->
						<p class="caption">{@html caption}</p>
					</div>

					<div class="tile-wrap delay-1" style="flex: 0 0 54%">
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

					<div class="right-margin"></div>
				</div>

				<!-- Bottom row: tile-2 portrait + tile-3 square + tile-4 square -->
				<div class="row row-bottom">
					<div class="tile-wrap delay-2" style="flex: 0 0 38%">
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

					<div class="right-col">
						<div class="tile-wrap delay-3">
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

						<div class="tile-wrap delay-4">
							<div class="tile tile-4">
								{#if tiles[3]?.type === 'video'}
									<video
										bind:this={videoEls[3]}
										src={tiles[3].src}
										aria-label={tiles[3].alt}
										autoplay
										muted
										loop
										playsinline
									>
										<track kind="captions" />
									</video>
								{:else}
									<img src={tiles[3]?.src ?? ''} alt={tiles[3]?.alt ?? ''} loading="lazy" />
								{/if}
							</div>
						</div>
					</div>
				</div>
			</div>
		{:else}
			<!-- Placeholder -->
			<div>
				<div class="row row-top">
					<div class="caption-block"></div>
					<div class="tile tile-1 placeholder-tile" style="flex: 0 0 54%"></div>
					<div class="right-margin"></div>
				</div>
				<div class="row row-bottom">
					<div class="tile tile-2 placeholder-tile" style="flex: 0 0 38%"></div>
					<div class="right-col">
						<div class="tile tile-3 placeholder-tile"></div>
						<div class="tile tile-4 placeholder-tile"></div>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
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
		display: flex;
		flex-direction: column;
		gap: var(--gap);
		box-sizing: border-box;
	}

	.row {
		display: flex;
		gap: var(--gap);
		width: 100%;
	}

	.row-top {
		align-items: flex-end;
	}
	.row-bottom {
		align-items: flex-start;
	}

	/* ── Caption ── */
	.caption-block {
		flex: 1;
		min-width: 0;
		align-self: flex-end;
		padding-bottom: 0.75rem;
		padding-left: 2rem;
	}

	.caption {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.85rem;
		font-style: italic;
		line-height: 1.65;
		color: #5a4a3a;
		margin: 0;
	}

	.right-margin {
		width: 6%;
		flex-shrink: 0;
	}

	/* ── Tiles ── */
	.tile {
		overflow: hidden;
		margin: 0;
		padding: 0;
		outline: 1px solid rgba(200, 150, 10, 0.12);
		width: 100%;
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

	/* Aspect ratios — mix of portrait, landscape, square */
	.tile-1 {
		aspect-ratio: 5 / 4;
	} /* landscape — top hero */
	.tile-2 {
		aspect-ratio: 3 / 4;
	} /* portrait  — bottom left */
	.tile-3 {
		aspect-ratio: 1 / 1;
	} /* square */
	.tile-4 {
		aspect-ratio: 5 / 4;
	} /* landscape */

	.right-col {
		flex: 1;
		min-width: 0;
		display: flex;
		flex-direction: column;
		gap: var(--gap);
	}

	/* ── Scroll reveal — staggered fade + slide up ── */
	.tile-wrap {
		opacity: 0;
		transform: translateY(32px);
		transition:
			opacity 0.75s ease,
			transform 0.75s ease;
	}

	.revealed .tile-wrap {
		opacity: 1;
		transform: translateY(0);
	}

	.revealed .delay-0 {
		transition-delay: 0ms;
	}
	.revealed .delay-1 {
		transition-delay: 100ms;
	}
	.revealed .delay-2 {
		transition-delay: 200ms;
	}
	.revealed .delay-3 {
		transition-delay: 300ms;
	}
	.revealed .delay-4 {
		transition-delay: 400ms;
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
		.row {
			flex-direction: column;
		}
		.row-top,
		.row-bottom {
			align-items: stretch;
		}
		.caption-block {
			padding-left: 0;
		}
		.right-margin {
			display: none;
		}
		.right-col {
			width: 100%;
		}
		.tile-1,
		.tile-2,
		.tile-3,
		.tile-4 {
			aspect-ratio: 4 / 3 !important;
		}
	}
	/* sketch layout variant */
	:global(.sketch) .tile-1 {
		aspect-ratio: 1 / 1;
	}
	:global(.sketch) .tile-2 {
		aspect-ratio: 16 / 9;
	}
	:global(.sketch) .tile-3 {
		aspect-ratio: 5 / 4;
	}
	:global(.sketch) .tile-4 {
		aspect-ratio: 1 / 1;
	}
</style>
