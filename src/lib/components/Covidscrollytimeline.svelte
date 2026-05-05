<script lang="ts">
	import { onMount } from 'svelte';

	// Canvas is 240vw wide. Elements use vw/vh for position and size.
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

	// ─────────────────────────────────────────────────────────────────────────────
	// CONTENT
	// Chronological left → right across the 240vw canvas.
	// Photo source notes are in comments beside each src field.
	// ─────────────────────────────────────────────────────────────────────────────
	const elements: ColEl[] = [
		// ── PANEL 1 · Feb–Mar 2020 · First case, national emergency ──────────────

		{
			// Photo rec: AFP / Getty — Mexico City Zócalo emptied, Mar 24 2020.
			// Search: Getty Images "Mexico City empty Zocalo COVID march 2020"
			// Alt: AP Photo/Marco Ugarte — similar angle, same week.
			type: 'intro-media',
			mediaType: 'image',
			location: 'Ciudad de México',
			date: '28 feb 2020',
			src: '/media/cdmx-primer-caso.jpg',
			caption: 'Primer caso confirmado de COVID-19 en México.',
			left: '2vw',
			top: '10vh',
			width: '17vw',
			height: '68vh',
			z: 10
		},
		{
			type: 'snippet',
			headline: 'México confirma su primer caso de COVID-19',
			dek: 'Un hombre de 35 años regresó de Italia con el virus. En tres semanas, el gobierno declararía emergencia sanitaria nacional.',
			left: '16vw',
			top: '5vh',
			width: '22vw',
			height: '20vh',
			z: 18
		},
		{
			// Photo rec: EFE / Gobierno de Jalisco — Gov. Alfaro press conference,
			// Mar 18 2020. Search: "Enrique Alfaro COVID conferencia marzo 2020"
			// or Gobierno de Jalisco Flickr (CC-licensed).
			type: 'intro-media',
			mediaType: 'image',
			location: 'Guadalajara, Jalisco',
			date: '18 mar 2020',
			src: '/media/jalisco-emergencia.jpg',
			caption: 'Jalisco declara emergencia sanitaria antes que el gobierno federal.',
			left: '24vw',
			top: '18vh',
			width: '13vw',
			height: '65vh',
			z: 22
		},

		// ── PANEL 2 · Mar–Apr 2020 · Nayarit first cases, Sana Distancia ─────────

		{
			// Text card — no photo needed; use gradient treatment from original.
			type: 'intro-media',
			mediaType: 'text',
			location: 'Tepic, Nayarit',
			date: '24 mar 2020',
			caption:
				'Los primeros casos en Nayarit llegaron silenciosamente. Tepic, capital del estado, cerró mercados y plazas. Fuera de la ciudad, muchas comunidades costeras y serranas apenas se enteraban.',
			left: '38vw',
			top: '4vh',
			width: '18vw',
			height: '60vh',
			z: 14
		},
		{
			type: 'search',
			query: '¿Hay COVID en Tepic Nayarit?',
			left: '34vw',
			top: '72vh',
			width: '28vw',
			height: '7vh',
			z: 28
		},
		{
			// Photo rec: Gobierno de Nayarit / SSA Nayarit social media —
			// Jornada Nacional de Sana Distancia signage. Check their Twitter/FB
			// archives, Mar–Apr 2020. CC or government public domain.
			type: 'intro-media',
			mediaType: 'image',
			location: 'Nayarit',
			date: 'abr 2020',
			src: '/media/nayarit-sana-distancia.jpg',
			caption: 'Señalización de la Jornada Nacional de Sana Distancia en Tepic.',
			left: '54vw',
			top: '14vh',
			width: '13vw',
			height: '50vh',
			z: 16
		},

		// ── PANEL 3 · Jun–Aug 2020 · Semáforo launched, tourism tensions ─────────

		{
			type: 'snippet',
			headline: 'El semáforo epidemiológico: un sistema de colores para vivir con el virus',
			dek: 'Desde el 1° de junio, cada estado amaneció en rojo, naranja, amarillo o verde. Jalisco y Nayarit arrancaron en naranja — pero el turismo no se detuvo.',
			left: '64vw',
			top: '5vh',
			width: '24vw',
			height: '22vh',
			z: 20
		},
		{
			// Photo rec: AP Photo or Reuters — Riviera Nayarit / Nuevo Vallarta
			// beach with tourists, summer 2020. Search: AP Images
			// "Nuevo Vallarta tourists COVID 2020" or "Bucerías beach 2020".
			// Alt: local newspaper El Meridiano de Nayarit may have CC photos.
			type: 'intro-media',
			mediaType: 'image',
			location: 'Nuevo Vallarta, Nayarit',
			date: 'jul 2020',
			src: '/media/nuevo-vallarta-turistas.jpg',
			caption: 'Turistas en playas de Riviera Nayarit pese a restricciones en naranja.',
			left: '66vw',
			top: '30vh',
			width: '14vw',
			height: '55vh',
			z: 24
		},
		{
			type: 'search',
			query: 'playas abiertas Nayarit semáforo naranja 2020',
			left: '60vw',
			top: '72vh',
			width: '30vw',
			height: '7vh',
			z: 26
		},

		// ── PANEL 4 · Oct–Dec 2020 · Second wave, oxygen shortage ────────────────

		{
			// Photo rec: AFP or Reuters — oxygen tanks outside Mexican hospital,
			// Dec 2020 / Jan 2021. Search: Getty "Mexico oxygen COVID hospital
			// 2020" or Reuters Pictures. Strong visual, widely licensed.
			// Alt: EFE "tanques oxígeno México COVID".
			type: 'intro-media',
			mediaType: 'image',
			location: 'México',
			date: 'dic 2020',
			src: '/media/mexico-oxigeno-escasez.jpg',
			caption: 'La escasez de oxígeno medicinal se convirtió en crisis paralela a la pandemia.',
			left: '82vw',
			top: '8vh',
			width: '15vw',
			height: '65vh',
			z: 12
		},
		{
			type: 'search',
			query: '¿Dónde conseguir oxígeno medicinal en Jalisco?',
			left: '80vw',
			top: '72vh',
			width: '32vw',
			height: '7vh',
			z: 22
		},
		{
			type: 'snippet',
			headline: 'En el Nayarit rural, el dinero decidía quién recibía oxígeno',
			dek: 'Con hospitales en Tepic al límite, las familias de comunidades alejadas pagaron precios inflados a revendedores o perdieron a sus enfermos en el camino.',
			left: '94vw',
			top: '4vh',
			width: '24vw',
			height: '24vh',
			z: 26
		},

		// ── PANEL 5 · Dec 2020 · Vaccination begins ──────────────────────────────

		{
			// Photo rec: Gobierno de Jalisco / Secretaría de Salud Jalisco —
			// First vaccination of healthcare worker, late Dec 2020 / Jan 2021.
			// Official government photos are public domain.
			// Search: "vacunación COVID Jalisco enero 2021 enfermera"
			// Alt: AP "Mexico COVID vaccine first doses".
			type: 'intro-media',
			mediaType: 'image',
			location: 'Guadalajara, Jalisco',
			date: 'ene 2021',
			src: '/media/jalisco-primera-vacuna.jpg',
			caption: 'Jalisco inicia vacunación a trabajadores de salud con dosis Pfizer.',
			left: '114vw',
			top: '16vh',
			width: '14vw',
			height: '65vh',
			z: 18
		},
		{
			type: 'snippet',
			headline: '24 de diciembre de 2020: México comienza a vacunar',
			dek: 'Las primeras dosis Pfizer-BioNTech llegaron a personal médico en la Ciudad de México. Jalisco y Nayarit recibirían sus primeras partidas semanas después.',
			left: '108vw',
			top: '4vh',
			width: '24vw',
			height: '22vh',
			z: 20
		},

		// ── PANEL 6 · Jan–Feb 2021 · Deadliest months ────────────────────────────

		{
			// Text card — high emotional weight; gradient treatment appropriate.
			type: 'intro-media',
			mediaType: 'text',
			location: 'Tepic, Nayarit',
			date: 'ene–feb 2021',
			caption:
				'Los meses más letales de la pandemia. El IMSS Tepic operó al límite. Familias esperaban noticias en el estacionamiento. Los médicos recibían turno tras turno sin dormir.',
			left: '126vw',
			top: '5vh',
			width: '18vw',
			height: '62vh',
			z: 14
		},
		{
			type: 'search',
			query: 'hospitales saturados Nayarit enero 2021',
			left: '124vw',
			top: '72vh',
			width: '30vw',
			height: '7vh',
			z: 24
		},
		{
			// Photo rec: Reuters or AFP — mass graves / overflowing cemeteries
			// Mexico, Jan–Feb 2021. OR: EFE "panteón COVID México 2021".
			// Alternatively, IMSS Jalisco facilities exterior — lower emotional
			// weight if preferred. Check Reuters Pictures ID: 2021-01-xx.
			type: 'intro-media',
			mediaType: 'image',
			location: 'México',
			date: 'feb 2021',
			src: '/media/mexico-meses-letales.jpg',
			caption: 'Enero y febrero de 2021 concentraron los picos de muertes diarias en todo el país.',
			left: '142vw',
			top: '12vh',
			width: '15vw',
			height: '62vh',
			z: 16
		},

		// ── PANEL 7 · Mar–Jun 2021 · Mass vaccination ────────────────────────────

		{
			// Photo rec: Gobierno de Jalisco — Expo Guadalajara mass vaccination
			// center, Mar 2021. Official photos freely available.
			// Search: "Expo Guadalajara vacunación masiva 2021" or
			// Jalisco government Flickr / Twitter archive.
			type: 'intro-media',
			mediaType: 'image',
			location: 'Guadalajara, Jalisco',
			date: 'mar 2021',
			src: '/media/expo-gdl-vacunacion.jpg',
			caption:
				'La Expo Guadalajara se convirtió en uno de los centros de vacunación masiva más grandes del occidente de México.',
			left: '156vw',
			top: '10vh',
			width: '16vw',
			height: '65vh',
			z: 18
		},
		{
			type: 'snippet',
			headline: 'Brigadas móviles llevan vacunas a la sierra y costa de Nayarit',
			dek: 'En abril de 2021, equipos de salud de Nayarit recorrieron comunidades de Compostela, Bahía de Banderas y la sierra huichola para vacunar a adultos mayores sin acceso a centros urbanos.',
			left: '170vw',
			top: '5vh',
			width: '26vw',
			height: '24vh',
			z: 22
		},

		// ── PANEL 8 · Jun–Sep 2021 · Delta wave ──────────────────────────────────

		{
			type: 'search',
			query: 'variante Delta síntomas México 2021',
			left: '166vw',
			top: '72vh',
			width: '30vw',
			height: '7vh',
			z: 24
		},
		{
			// Photo rec: AP or AFP — Puerto Vallarta / Vallarta-Nayarit corridor
			// COVID signage or testing tent, summer 2021.
			// Search: AP Images "Puerto Vallarta COVID Delta 2021"
			// Alt: Municipio de Bahía de Banderas social media posts — free.
			type: 'intro-media',
			mediaType: 'image',
			location: 'corredor Vallarta–Nayarit',
			date: 'ago 2021',
			src: '/media/vallarta-nayarit-delta.jpg',
			caption:
				'El corredor turístico compartido entre Jalisco y Nayarit se convirtió en foco de la ola Delta.',
			left: '194vw',
			top: '15vh',
			width: '14vw',
			height: '60vh',
			z: 16
		},

		// ── PANEL 9 · Dec 2021–Feb 2022 · Omicron ────────────────────────────────

		{
			// Text card — Omicron season in Sayulita/San Pancho.
			type: 'intro-media',
			mediaType: 'text',
			location: 'Sayulita / San Pancho, Nayarit',
			date: 'ene 2022',
			caption:
				'Ómicron llegó en plena temporada alta en las playas bohemias de Nayarit. Grupos de WhatsApp de expatriados y grupos de Facebook de vecinos se llenaron de avisos de contagios, cancelaciones de rentas y debates sobre cubrebocas en la playa.',
			left: '178vw',
			top: '6vh',
			width: '18vw',
			height: '58vh',
			z: 14
		},
		{
			type: 'snippet',
			headline: 'Jalisco bate récords de casos con Ómicron, aunque con menos muertes',
			dek: 'En enero de 2022, la variante Ómicron disparó los contagios a niveles sin precedente en el área metropolitana de Guadalajara, pero la vacunación atenuó la presión hospitalaria.',
			left: '208vw',
			top: '4vh',
			width: '26vw',
			height: '24vh',
			z: 20
		},
		{
			type: 'search',
			query: '¿cuánto cuesta cama UCI hospital privado Guadalajara 2022?',
			left: '196vw',
			top: '72vh',
			width: '36vw',
			height: '7vh',
			z: 22
		}
	];

	let outerEl: HTMLElement;
	let offsetX = 0;
	let phase: 'before' | 'active' | 'after' = 'before';

	// Must match the CSS width of .timeline-canvas
	const CANVAS_VW = 240;

	onMount(() => {
		const mq = window.matchMedia('(max-width: 640px)');
		let rafId = 0;

		function setHeight() {
			if (!outerEl || mq.matches) {
				if (outerEl) outerEl.style.height = 'auto';
				return;
			}
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;
			outerEl.style.height = `calc(100vh + ${travel}px)`;
		}

		function update() {
			if (mq.matches || !outerEl) return;
			const rect = outerEl.getBoundingClientRect();
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;

			if (rect.top > 0) {
				phase = 'before';
				offsetX = 0;
			} else if (rect.bottom <= window.innerHeight) {
				phase = 'after';
				offsetX = -travel;
			} else {
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
<section class="timeline-section full-bleed">
	<div class="timeline-outer" bind:this={outerEl}>
		<div class="timeline-panel" data-phase={phase}>
			<div class="timeline-canvas" style="transform: translateX({offsetX}px)">
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
									<circle
										cx="10.5"
										cy="10.5"
										r="6.5"
										fill="none"
										stroke="#9aa0a6"
										stroke-width="2"
									/>
									<line
										x1="15.5"
										y1="15.5"
										x2="22"
										y2="22"
										stroke="#9aa0a6"
										stroke-width="2"
										stroke-linecap="round"
									/>
								</svg>
								<p class="term">{el.query}</p>
							</div>
						{/if}
					</div>
				{/each}
			</div>

			<div class="scroll-nudge" aria-hidden="true">
				<span>desplázate para explorar</span>
				<svg viewBox="0 0 24 24" width="14" height="14">
					<path
						d="M5 12h14M13 6l6 6-6 6"
						stroke="#a08060"
						stroke-width="2"
						fill="none"
						stroke-linecap="round"
						stroke-linejoin="round"
					/>
				</svg>
			</div>
		</div>
	</div>
</section>

<style>
	/* ─── Google Fonts — load in your app's <head> or +layout.svelte ────────────
	   <link rel="preconnect" href="https://fonts.googleapis.com">
	   <link href="https://fonts.googleapis.com/css2?family=Crimson+Text:ital,wght@0,400;0,600;0,700;1,400&display=swap" rel="stylesheet">
	   ───────────────────────────────────────────────────────────────────────── */

	.timeline-section {
		background: #f5f0e8;
	}

	.timeline-outer {
		position: relative;
	}

	/* Three-state panel positioning — driven by JS phase variable */
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

	/* 240vw canvas — translates left as scroll progresses */
	.timeline-canvas {
		position: absolute;
		top: 0;
		left: 0;
		width: 240vw; /* must match CANVAS_VW in script */
		height: 100%;
		will-change: transform;
		transition: transform 0.06s linear;
	}

	.scrolly-el {
		position: absolute;
	}

	/* ── Intro media card ── */
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

	/* Gradient text treatment — used for text-only cards */
	.media-text {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 1.2rem;
		margin: 0;
		/* Deep indigo → violet → warm red — evokes gravity without sensationalism */
		background: linear-gradient(160deg, #1a0848 0%, #2e1268 55%, #c04040 130%);
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.78rem, 1.1vw, 1rem);
		line-height: 1.6;
		color: #fff5d8;
		text-align: center;
	}

	/* ── Snippet card ── */
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

	/* ── Mock Google search bar ── */
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

	/* ── Scroll nudge ── */
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

	/* ── Mobile (≤640px) ────────────────────────────────────────────────────────
	   Kill horizontal scrolly mechanism entirely.
	   Render as a simple vertical stacked feed.
	   ───────────────────────────────────────────────────────────────────────── */
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
