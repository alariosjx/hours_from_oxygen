<script lang="ts">
	import { onMount } from 'svelte';

	// Canvas is 220vw wide. Elements use vw/vh for position and size.
	// left/top = distance from canvas left/top edge
	// width/height = element dimensions
	// z = stacking order (higher = in front)
	type ColEl =
		| {
				type: 'intro-media';
				mediaType: 'image' | 'video' | 'text';
				location: string;
				date: string;
				src?: string;
				caption?: string;
				left: string;
				top: string;
				width: string;
				height: string;
				z: number;
		  }
		| {
				type: 'snippet';
				headline: string;
				dek: string;
				left: string;
				top: string;
				width: string;
				height: string;
				z: number;
		  }
		| {
				type: 'search';
				query: string;
				left: string;
				top: string;
				width: string;
				height: string;
				z: number;
		  };

	const elements: ColEl[] = [
		// ── Far left — visible on entry ──
		{
			type: 'intro-media',
			mediaType: 'image',
			location: 'Mexico City',
			date: 'Mar 20, 2020',
			src: '/media/mexico-city-lockdown.jpg',
			left: '2vw',
			top: '12vh',
			width: '18vw',
			height: '65vh',
			z: 10
		},
		{
			type: 'snippet',
			headline: 'Mexico orders nationwide lockdown as COVID-19 surges',
			dek: 'Authorities declared a national health emergency, shutting down nonessential businesses across the country.',
			left: '17vw',
			top: '4vh',
			width: '24vw',
			height: '22vh',
			z: 18
		},
		{
			type: 'intro-media',
			mediaType: 'video',
			location: 'Cancún, Q. Roo',
			date: 'Jul 15, 2020',
			src: '/media/cancun-tourists.mp4',
			left: '25vw',
			top: '18vh',
			width: '14vw',
			height: '70vh',
			z: 22
		},
		// ── Mid-left ──
		{
			type: 'search',
			query: '¿Dónde comprar tanque de oxígeno en México?',
			left: '36vw',
			top: '72vh',
			width: '30vw',
			height: '7vh',
			z: 28
		},
		{
			type: 'intro-media',
			mediaType: 'text',
			location: 'Tepic, Nayarit',
			date: 'Dec 10, 2020',
			caption:
				'Hospitals in Tepic reached full capacity. Families waited outside in the cold for any news about their loved ones.',
			left: '40vw',
			top: '3vh',
			width: '20vw',
			height: '62vh',
			z: 14
		},
		// ── Center ──
		{
			type: 'search',
			query: 'hospitales saturados COVID México diciembre 2020',
			left: '57vw',
			top: '72vh',
			width: '30vw',
			height: '7vh',
			z: 24
		},
		{
			type: 'intro-media',
			mediaType: 'image',
			location: 'Guadalajara, Jalisco',
			date: 'Feb 5, 2021',
			src: '/media/guadalajara-vaccine.jpg',
			left: '62vw',
			top: '6vh',
			width: '20vw',
			height: '65vh',
			z: 16
		},
		// ── Mid-right ──
		{
			type: 'snippet',
			headline: 'When money determines who lives and who dies',
			dek: 'In rural Nayarit, access to oxygen during the COVID-19 surge often came down to wealth — and proximity to a city.',
			left: '80vw',
			top: '32vh',
			width: '26vw',
			height: '24vh',
			z: 26
		},
		{
			type: 'intro-media',
			mediaType: 'image',
			location: 'Cancún, Q. Roo',
			date: 'Jul 15, 2020',
			src: '/media/cancun-tourists.jpg',
			caption: 'Tourists flocked to Cancún despite rising COVID-19 cases.',
			left: '88vw',
			top: '2vh',
			width: '17vw',
			height: '68vh',
			z: 12
		},
		// ── Far right ──
		{
			type: 'search',
			query: '¿Cuánto cuesta una cama UCI en hospital privado México?',
			left: '102vw',
			top: '72vh',
			width: '32vw',
			height: '7vh',
			z: 22
		}
	];

	let outerEl: HTMLElement;
	let offsetX = 0;
	// 'before' = panel flows normally above viewport
	// 'active' = panel is fixed to top of viewport, canvas translating
	// 'after'  = panel flows normally below viewport (section scrolled past)
	let phase: 'before' | 'active' | 'after' = 'before';

	// Must match the CSS width of .timeline-canvas
	const CANVAS_VW = 220;

	onMount(() => {
		const mq = window.matchMedia('(max-width: 640px)');
		let rafId = 0;

		function setHeight() {
			if (!outerEl || mq.matches) {
				if (outerEl) outerEl.style.height = 'auto';
				return;
			}
			// Outer needs: 100vh (to sit over viewport) + horizontal travel distance
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;
			outerEl.style.height = `calc(100vh + ${travel}px)`;
		}

		function update() {
			if (mq.matches || !outerEl) return;
			const rect = outerEl.getBoundingClientRect();
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;

			if (rect.top > 0) {
				// Section hasn't reached the top of the viewport yet
				phase = 'before';
				offsetX = 0;
			} else if (rect.bottom <= window.innerHeight) {
				// Section has fully scrolled past — panel rests at bottom of outer
				phase = 'after';
				offsetX = -travel;
			} else {
				// Section is active — progress through the travel distance
				phase = 'active';
				const scrollable = outerEl.offsetHeight - window.innerHeight;
				const progress = Math.min(1, -rect.top / scrollable);
				offsetX = -(progress * travel);
			}
		}

		function onScroll() {
			cancelAnimationFrame(rafId);
			rafId = requestAnimationFrame(update);
		}

		const ro = new ResizeObserver(() => {
			setHeight();
			update();
		});
		ro.observe(document.body);
		setHeight();
		window.addEventListener('scroll', onScroll, { passive: true });
		update();

		function onMqChange(e: MediaQueryListEvent) {
			if (e.matches) {
				outerEl.style.height = 'auto';
				offsetX = 0;
				phase = 'before';
			} else {
				setHeight();
				update();
			}
		}
		mq.addEventListener('change', onMqChange);

		return () => {
			window.removeEventListener('scroll', onScroll);
			cancelAnimationFrame(rafId);
			ro.disconnect();
			mq.removeEventListener('change', onMqChange);
		};
	});
</script>

<!-- Outer controls scroll runway. Sticky inner stays pinned while user scrolls through it. -->
<section class="timeline-section">
	<div class="timeline-outer" bind:this={outerEl}>
		<!-- phase drives position: fixed (active) vs absolute (before/after) -->
		<div class="timeline-panel" data-phase={phase}>
			<!-- Wide canvas — slides left as user scrolls down through the outer wrapper -->
			<div
				class="timeline-canvas"
				style="transform: translateX({offsetX}px)"
			>
				{#each elements as el (el.type + el.left)}
					<div
						class="scrolly-el"
						style="left:{el.left}; top:{el.top}; width:{el.width}; height:{el.height}; z-index:{el.z}"
					>
						{#if el.type === 'intro-media'}
							<div class="intro-media">
								<header>
									<h4>{el.location}</h4>
									<h5>{el.date}</h5>
								</header>
								<div class="media">
									{#if el.mediaType === 'video'}
										<video class="media-el" src={el.src} autoplay muted loop playsinline></video>
									{:else if el.mediaType === 'image'}
										<img class="media-el" src={el.src} alt={el.caption ?? el.location} />
									{:else}
										<p class="media-text">{el.caption}</p>
									{/if}
								</div>
							</div>
						{:else if el.type === 'snippet'}
							<div class="snippet">
								<h4>{el.headline}</h4>
								<p class="dek">{el.dek}</p>
							</div>
						{:else if el.type === 'search'}
							<div class="search">
								<svg class="search-icon" viewBox="0 0 24 24" aria-hidden="true">
									<circle cx="10.5" cy="10.5" r="6.5" fill="none" stroke="#9aa0a6" stroke-width="2" />
									<line x1="15.5" y1="15.5" x2="22" y2="22" stroke="#9aa0a6" stroke-width="2" stroke-linecap="round" />
								</svg>
								<p class="term">{el.query}</p>
							</div>
						{/if}
					</div>
				{/each}
			</div>

			<!-- Scroll nudge — fixed in viewport, fades via CSS once user starts scrolling -->
			<div class="scroll-nudge" aria-hidden="true">
				<span>scroll to explore</span>
				<svg viewBox="0 0 24 24" width="14" height="14">
					<path d="M5 12h14M13 6l6 6-6 6" stroke="#a08060" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"/>
				</svg>
			</div>
		</div>
	</div>
</section>

<style>
	.timeline-section {
		background: #f5f0e8;
	}

	/* ── OUTER: height set by JS to equal 100vh + horizontal scroll distance ── */
	.timeline-outer {
		position: relative;
	}

	/* ── PANEL: three-state positioning driven by JS phase variable ──
       before/after → absolute (in normal flow at top/bottom of outer)
       active       → fixed (pinned to viewport while canvas scrolls) */
	.timeline-panel {
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #f5f0e8;
	}
	.timeline-panel[data-phase='before'] {
		position: absolute;
		top: 0;
	}
	.timeline-panel[data-phase='active'] {
		position: fixed;
		top: 0;
		left: 0;
	}
	.timeline-panel[data-phase='after'] {
		position: absolute;
		bottom: 0;
	}

	/* ── CANVAS: 220vw wide, translates left as scroll progresses ── */
	.timeline-canvas {
		position: absolute;
		top: 0;
		left: 0;
		width: 220vw; /* must match CANVAS_VW in script */
		height: 100%;
		will-change: transform;
		transition: transform 0.06s linear;
	}

	/* ── COLLAGE ELEMENTS: absolutely positioned via inline styles ── */
	.scrolly-el {
		position: absolute;
	}

	/* ── INTRO MEDIA (image / video / text card) ── */
	.intro-media {
		width: 100%;
		height: 100%;
		background: #fff;
		border-radius: 10px;
		overflow: hidden;
		box-shadow:
			0 4px 20px rgba(26, 8, 72, 0.18),
			0 1px 4px rgba(26, 8, 72, 0.08);
		display: flex;
		flex-direction: column;
	}

	.intro-media header {
		padding: 0.55rem 0.75rem 0.45rem;
		border-bottom: 1px solid #f0ebe0;
		flex-shrink: 0;
	}

	.intro-media h4 {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.82rem;
		font-weight: 700;
		color: #c04040;
		margin: 0;
		line-height: 1.2;
	}

	.intro-media h5 {
		font-size: 0.68rem;
		color: #999;
		margin: 0;
		font-weight: 400;
		font-family: 'Crimson Text', Georgia, serif;
	}

	.media {
		flex: 1;
		overflow: hidden;
		background: #1a0848;
		min-height: 0;
	}

	.media-el {
		width: 100%;
		height: 100%;
		object-fit: cover;
		display: block;
	}

	.media-text {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 1.2rem;
		margin: 0;
		background: linear-gradient(160deg, #1a0848 0%, #2e1268 55%, #c04040 130%);
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.78rem, 1.1vw, 1rem);
		line-height: 1.55;
		color: #fff5d8;
		text-align: center;
	}

	/* ── SNIPPET (shared news article card) ── */
	.snippet {
		width: 100%;
		height: 100%;
		background: #fff;
		border-radius: 10px;
		box-shadow:
			0 2px 16px rgba(26, 8, 72, 0.14),
			0 1px 3px rgba(26, 8, 72, 0.06);
		padding: 1rem 1.1rem;
		display: flex;
		flex-direction: column;
		justify-content: center;
		/* thin gold top border echoes story color palette */
		border-top: 3px solid #c8960a;
	}

	.snippet h4 {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.82rem, 1.15vw, 1rem);
		font-weight: 700;
		color: #1a0848;
		line-height: 1.35;
		margin: 0 0 0.45rem;
	}

	.snippet .dek {
		font-size: clamp(0.68rem, 0.85vw, 0.78rem);
		color: #666;
		line-height: 1.5;
		margin: 0;
		font-family: 'Crimson Text', Georgia, serif;
	}

	/* ── SEARCH BAR (mock Google search question) ── */
	.search {
		width: 100%;
		height: 100%;
		background: #fff;
		border-radius: 24px;
		box-shadow:
			0 2px 10px rgba(0, 0, 0, 0.16),
			0 0 0 1px rgba(0, 0, 0, 0.06);
		padding: 0 1rem 0 2.6rem;
		display: flex;
		align-items: center;
		position: relative;
	}

	.search-icon {
		position: absolute;
		left: 0.85rem;
		width: 18px;
		height: 18px;
		flex-shrink: 0;
	}

	.search .term {
		font-size: clamp(0.72rem, 0.95vw, 0.88rem);
		color: #202124;
		margin: 0;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		font-family: 'Roboto', 'Arial', sans-serif;
		letter-spacing: 0;
	}

	/* ── SCROLL NUDGE ── */
	.scroll-nudge {
		position: absolute;
		bottom: 1.5rem;
		right: 2rem;
		display: flex;
		align-items: center;
		gap: 0.4rem;
		color: #a08060;
		font-size: 0.72rem;
		font-family: 'Crimson Text', Georgia, serif;
		font-style: italic;
		letter-spacing: 0.05em;
		pointer-events: none;
		z-index: 200;
	}

	/* ── MOBILE (≤640px — matches LandingHero and FirstBanner breakpoints) ──
       Kill the sticky/horizontal mechanism entirely.
       Render elements as a simple vertical feed. */
	@media (max-width: 640px) {
		.timeline-panel,
		.timeline-panel[data-phase='before'],
		.timeline-panel[data-phase='active'],
		.timeline-panel[data-phase='after'] {
			position: static;
			height: auto;
			overflow: visible;
			padding: 2.5rem 0 1rem;
		}

		.timeline-canvas {
			position: static;
			width: 100%;
			height: auto;
			transform: none !important;
			transition: none;
			display: flex;
			flex-direction: column;
			gap: 1rem;
			padding: 0 1rem;
		}

		.scrolly-el {
			position: static !important;
			width: 100% !important;
			height: auto !important;
		}

		.intro-media {
			height: auto;
		}

		.media {
			aspect-ratio: 4 / 3;
			height: auto;
		}

		.media-text {
			height: auto;
			min-height: 160px;
		}

		.snippet {
			height: auto;
		}

		.search {
			height: 52px;
			border-radius: 12px;
		}

		.scroll-nudge {
			display: none;
		}
	}
</style>
