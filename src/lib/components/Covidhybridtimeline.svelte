<script lang="ts">
	import { onMount } from 'svelte';

	// ─── Horizontal scrolly section data ────────────────────────────────────────
	// Two independent canvases, each 180vw wide.
	// Photo src paths are placeholders — replace with your own assets.
	// Photo licensing notes are in comments beside each element.

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

	// ── SCROLLY A · Second wave / oxygen crisis · Oct–Dec 2020 ──────────────────
	const scrollyA: ColEl[] = [
		{
			// Photo rec: AFP or EFE — oxygen tanks lined up outside a hospital,
			// Mexico, Dec 2020. Search: Getty "tanques oxígeno México hospital 2020"
			// or Reuters Pictures "Mexico oxygen COVID".
			type: 'intro-media',
			mediaType: 'image',
			location: 'México',
			date: 'dic 2020',
			src: '/media/oxigeno-tanques.jpg',
			caption: 'Familias hacen fila para recargar tanques de oxígeno medicinal.',
			left: '3vw',
			top: '10vh',
			width: '17vw',
			height: '68vh',
			z: 10
		},
		{
			type: 'search',
			query: '¿Dónde conseguir oxígeno medicinal en Jalisco?',
			left: '2vw',
			top: '74vh',
			width: '32vw',
			height: '7vh',
			z: 20
		},
		{
			// Social media clip rec: screenshot from Twitter/X — viral post about
			// oxygen shortage in Tepic, Dec 2020. Crop and add byline.
			type: 'intro-media',
			mediaType: 'text',
			location: 'Tepic, Nayarit',
			date: 'dic 2020',
			caption:
				'"El hospital ya no tiene camas. Mi papá lleva 6 horas esperando en el carro con el oxígeno que le trajimos de Guadalajara." — publicación viral, diciembre 2020',
			left: '18vw',
			top: '5vh',
			width: '18vw',
			height: '62vh',
			z: 16
		},
		{
			type: 'snippet',
			headline: 'En el Nayarit rural, el dinero decidía quién recibía oxígeno',
			dek: 'Con los hospitales del IMSS Tepic al límite, las familias de comunidades alejadas pagaron precios inflados a revendedores — o perdieron a sus enfermos en el camino.',
			left: '34vw',
			top: '8vh',
			width: '24vw',
			height: '24vh',
			z: 22
		},
		{
			type: 'search',
			query: 'hospitales saturados Nayarit diciembre 2020',
			left: '36vw',
			top: '74vh',
			width: '34vw',
			height: '7vh',
			z: 18
		},
		{
			// Photo rec: AP Photo or Reuters — IMSS hospital exterior Guadalajara,
			// families waiting outside, Dec 2020 / Jan 2021.
			// Search: AP Images "IMSS Guadalajara COVID families 2020".
			type: 'intro-media',
			mediaType: 'image',
			location: 'Guadalajara, Jalisco',
			date: 'dic 2020',
			src: '/media/imss-gdl-familias.jpg',
			caption: 'Familiares esperan noticias afuera del IMSS. Las salas de espera estaban cerradas.',
			left: '56vw',
			top: '14vh',
			width: '16vw',
			height: '62vh',
			z: 14
		}
	];

	// ── SCROLLY B · Deadliest months · Jan–Feb 2021 ─────────────────────────────
	const scrollyB: ColEl[] = [
		{
			// Photo rec: AFP or Reuters — healthcare workers exhausted, Mexico
			// hospital, Jan 2021. Search: AFP "médicos agotados México COVID 2021"
			// or Getty "Mexico nurses COVID January 2021 tired".
			type: 'intro-media',
			mediaType: 'image',
			location: 'México',
			date: 'ene 2021',
			src: '/media/medicos-agotados.jpg',
			caption: 'Personal médico tras un turno de 24 horas durante el pico de la pandemia.',
			left: '3vw',
			top: '8vh',
			width: '16vw',
			height: '70vh',
			z: 10
		},
		{
			// Social media / journalism clip rec: screenshot from a local Nayarit
			// news outlet (e.g. NVI Noticias or Meridiano de Nayarit) showing
			// daily death count graphic, Jan–Feb 2021.
			type: 'intro-media',
			mediaType: 'text',
			location: 'Tepic, Nayarit',
			date: 'ene–feb 2021',
			caption:
				'El IMSS Tepic operó por semanas sobre su capacidad. Médicos recibían turno tras turno sin dormir. Familias esperaban en el estacionamiento — las salas de espera, cerradas desde meses antes.',
			left: '17vw',
			top: '5vh',
			width: '18vw',
			height: '64vh',
			z: 16
		},
		{
			type: 'search',
			query: '¿cuántos muertos por COVID en México hoy?',
			left: '2vw',
			top: '74vh',
			width: '40vw',
			height: '7vh',
			z: 20
		},
		{
			type: 'snippet',
			headline: 'Los meses más letales de la pandemia en México',
			dek: 'Enero y febrero de 2021 concentraron los picos de muertes diarias en todo el país. México superó las 160,000 muertes oficiales acumuladas — aunque las estimaciones de exceso de mortalidad sugerían el doble.',
			left: '33vw',
			top: '10vh',
			width: '24vw',
			height: '26vh',
			z: 22
		},
		{
			// Photo rec: Reuters or AFP — mass burial / overflowing cemetery
			// Mexico City or Guadalajara, Jan–Feb 2021.
			// Search: Reuters "Mexico mass graves COVID 2021" or
			// AFP "panteón COVID México enero 2021".
			// If too graphic: use exterior of IMSS hospital at night instead.
			type: 'intro-media',
			mediaType: 'image',
			location: 'México',
			date: 'feb 2021',
			src: '/media/mexico-muertes-pico.jpg',
			caption:
				'Trabajadores de panteones municipales en México en los meses más duros de la pandemia.',
			left: '55vw',
			top: '12vh',
			width: '16vw',
			height: '64vh',
			z: 14
		},
		{
			type: 'search',
			query: 'exceso de mortalidad México 2021',
			left: '42vw',
			top: '74vh',
			width: '32vw',
			height: '7vh',
			z: 18
		}
	];

	// ─── Scrolly engine — one instance per canvas ────────────────────────────────
	type ScrollyState = { offsetX: number; phase: 'before' | 'active' | 'after' };

	let outerA: HTMLElement;
	let outerB: HTMLElement;
	let stateA: ScrollyState = { offsetX: 0, phase: 'before' };
	let stateB: ScrollyState = { offsetX: 0, phase: 'before' };

	const CANVAS_VW = 180; // must match .scrolly-canvas width in CSS

	function makeScrollyUpdater(
		getOuter: () => HTMLElement | undefined,
		getState: () => ScrollyState,
		setState: (s: ScrollyState) => void
	) {
		return function update() {
			const outer = getOuter();
			if (!outer) return;
			const rect = outer.getBoundingClientRect();
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;

			if (rect.top > 0) {
				setState({ offsetX: 0, phase: 'before' });
			} else if (rect.bottom <= window.innerHeight) {
				setState({ offsetX: -travel, phase: 'after' });
			} else {
				const scrollable = outer.offsetHeight - window.innerHeight;
				const progress = Math.min(1, -rect.top / scrollable);
				setState({ offsetX: -(progress * travel), phase: 'active' });
			}
		};
	}

	onMount(() => {
		const mq = window.matchMedia('(max-width: 640px)');
		let rafId = 0;

		function setHeights() {
			if (mq.matches) return;
			const travel = (CANVAS_VW / 100 - 1) * window.innerWidth;
			const h = `calc(100vh + ${travel}px)`;
			if (outerA) outerA.style.height = h;
			if (outerB) outerB.style.height = h;
		}

		const updateA = makeScrollyUpdater(
			() => outerA,
			() => stateA,
			(s) => (stateA = s)
		);
		const updateB = makeScrollyUpdater(
			() => outerB,
			() => stateB,
			(s) => (stateB = s)
		);

		function onScroll() {
			cancelAnimationFrame(rafId);
			rafId = requestAnimationFrame(() => {
				updateA();
				updateB();
			});
		}

		const ro = new ResizeObserver(() => {
			setHeights();
			updateA();
			updateB();
		});
		ro.observe(document.body);
		setHeights();
		window.addEventListener('scroll', onScroll, { passive: true });
		updateA();
		updateB();

		function onMqChange(e: MediaQueryListEvent) {
			if (e.matches) {
				if (outerA) outerA.style.height = 'auto';
				if (outerB) outerB.style.height = 'auto';
				stateA = { offsetX: 0, phase: 'before' };
				stateB = { offsetX: 0, phase: 'before' };
			} else {
				setHeights();
				updateA();
				updateB();
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

	function canvasStyle(offsetX: number) {
		return `transform: translateX(${offsetX}px)`;
	}
</script>

<!--
	FONT: Add to +layout.svelte <svelte:head> or app.html <head>:
	<link rel="preconnect" href="https://fonts.googleapis.com">
	<link href="https://fonts.googleapis.com/css2?family=Crimson+Text:ital,wght@0,400;0,600;0,700;1,400&display=swap" rel="stylesheet">
-->

<article class="covid-narrative full-bleed">
	<!-- ═══════════════════════════════════════════════════════════════
	     SECTION 1 · First case → Jalisco emergency · Feb–Mar 2020
	     ═══════════════════════════════════════════════════════════════ -->
	<section class="narrative-block">
		<div class="text-col">
			<p class="dateline">28 de febrero de 2020</p>
			<h2>El primer caso</h2>
			<p>
				Un hombre de 35 años regresó de Italia con el virus. México entraba oficialmente a la
				pandemia. En tres semanas, el gobierno federal declararía emergencia sanitaria nacional y
				activaría la Jornada Nacional de Sana Distancia.
			</p>
			<p>
				Jalisco se adelantó. El gobernador Enrique Alfaro cerró escuelas y canceló eventos masivos
				el 18 de marzo — días antes que la declaratoria federal. Las redes sociales se llenaron de
				debates sobre si era exagerado.
			</p>
		</div>

		<!-- Your photo: Mar 2020 CDMX empty street or Jalisco emergency presser -->
		<!-- Rec: AP/Marco Ugarte — Zócalo vacío, mar 24 2020 -->
		<!-- Rec: Gobierno de Jalisco Flickr — Alfaro conferencia, mar 18 2020 (CC) -->
		<figure class="media-col">
			<div class="media-frame tall">
				<img
					src="/media/jalisco-emergencia.jpg"
					alt="Gobernador Alfaro anuncia medidas de emergencia en Jalisco, marzo 2020."
				/>
			</div>
			<figcaption>Guadalajara, Jalisco · 18 mar 2020</figcaption>
		</figure>
	</section>

	<!-- ═══════════════════════════════════════════════════════════════
	     SECTION 2 · Nayarit arrives + Sana Distancia · Mar–May 2020
	     ═══════════════════════════════════════════════════════════════ -->
	<section class="narrative-block reverse">
		<!-- Your photo: Tepic street empty, or SSA Nayarit Sana Distancia signage -->
		<!-- Rec: SSA Nayarit Twitter/FB archive, Mar–Apr 2020. Gobierno = public domain -->
		<figure class="media-col">
			<div class="media-frame tall">
				<img
					src="/media/tepic-sana-distancia.jpg"
					alt="Señalización de Sana Distancia en calles de Tepic, Nayarit."
				/>
			</div>
			<figcaption>Tepic, Nayarit · abr 2020</figcaption>
		</figure>

		<div class="text-col">
			<p class="dateline">24 de marzo de 2020</p>
			<h2>Nayarit entra a la pandemia</h2>
			<p>
				Los primeros casos en Nayarit llegaron silenciosamente. Tepic, la capital, cerró mercados y
				plazas. En la costa — Sayulita, San Pancho, Bucerías — la temporada alta acababa de terminar
				y las comunidades de expatriados comenzaban a recibir alertas por grupos de WhatsApp.
			</p>
			<p>
				En la sierra, muchas comunidades huicholas apenas se enteraban. Las brigadas de salud
				tardaron semanas en llegar.
			</p>
		</div>
	</section>

	<!-- ═══════════════════════════════════════════════════════════════
	     SECTION 3 · Semáforo + tourism tensions · Jun–Aug 2020
	     ═══════════════════════════════════════════════════════════════ -->
	<section class="narrative-block">
		<div class="text-col">
			<p class="dateline">1° de junio de 2020</p>
			<h2>El semáforo y la presión del turismo</h2>
			<p>
				El gobierno federal lanzó el semáforo epidemiológico: rojo, naranja, amarillo, verde.
				Jalisco y Nayarit arrancaron en naranja. Pero el corredor turístico compartido entre Puerto
				Vallarta y Nuevo Vallarta no se detuvo.
			</p>
			<p>
				Las cuentas municipales en redes sociales publicaban advertencias. Los grupos de Facebook de
				locales documentaban playas concurridas. El debate entre economía y salud pública se volvió
				el ruido de fondo del verano.
			</p>
		</div>

		<!-- Your photo or social media clip: beach crowds Riviera Nayarit summer 2020 -->
		<!-- Rec: AP Images "Nuevo Vallarta tourists COVID 2020" -->
		<!-- Alt: clip de cuenta municipal de Bahía de Banderas (free) -->
		<figure class="media-col">
			<div class="media-frame">
				<img
					src="/media/nuevo-vallarta-verano.jpg"
					alt="Playas de Riviera Nayarit con turistas pese a restricciones, verano 2020."
				/>
			</div>
			<figcaption>Nuevo Vallarta, Nayarit · jul 2020</figcaption>
		</figure>
	</section>

	<!-- ═══════════════════════════════════════════════════════════════
	     ▓ HORIZONTAL SCROLLY A · Segunda ola / crisis del oxígeno
	       Oct–Dec 2020
	     ═══════════════════════════════════════════════════════════════ -->
	<div class="scrolly-label">
		<span>octubre — diciembre 2020</span>
		<h2>La crisis del oxígeno</h2>
		<p>
			La segunda ola llegó antes de que terminara el año. Los hospitales del IMSS en Tepic y
			Guadalajara volvieron a llenarse. Esta vez se sumó una escasez de oxígeno medicinal que
			convirtió el acceso al tanque correcto en una cuestión de vida o muerte.
		</p>
	</div>

	<div class="scrolly-outer" bind:this={outerA}>
		<div class="scrolly-panel" data-phase={stateA.phase}>
			<div class="scrolly-canvas" style={canvasStyle(stateA.offsetX)}>
				{#each scrollyA as el (el.type + el.left)}
					<div
						class="scrolly-el"
						style="left:{el.left}; top:{el.top}; width:{el.width}; height:{el.height}; z-index:{el.z}"
					>
						{#if el.type === 'intro-media'}
							<div class="card-media">
								<header>
									<h4>{el.location}</h4>
									<h5>{el.date}</h5>
								</header>
								<div class="card-inner">
									{#if el.mediaType === 'video'}
										<video class="card-img" src={el.src} autoplay muted loop playsinline></video>
									{:else if el.mediaType === 'image'}
										<img class="card-img" src={el.src} alt={el.caption ?? el.location} />
									{:else}
										<p class="card-text">{el.caption}</p>
									{/if}
								</div>
							</div>
						{:else if el.type === 'snippet'}
							<div class="card-snippet">
								<h4>{el.headline}</h4>
								<p class="dek">{el.dek}</p>
							</div>
						{:else if el.type === 'search'}
							<div class="card-search">
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
				<span>desplázate</span>
				<svg viewBox="0 0 24 24" width="12" height="12">
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

	<!-- ═══════════════════════════════════════════════════════════════
	     SECTION 4 · Vaccination begins · Dec 2020–Mar 2021
	     ═══════════════════════════════════════════════════════════════ -->
	<section class="narrative-block reverse">
		<!-- Your photo: first vaccination Jalisco, or Expo GDL mass vax center -->
		<!-- Rec: Gobierno de Jalisco Flickr — primera dosis enfermera, dic 2020–ene 2021 (CC) -->
		<!-- Alt: AP "Mexico COVID vaccine first doses" -->
		<figure class="media-col">
			<div class="media-frame">
				<img
					src="/media/jalisco-primera-vacuna.jpg"
					alt="Primera vacuna COVID aplicada a trabajadora de salud en Jalisco."
				/>
			</div>
			<figcaption>Guadalajara, Jalisco · ene 2021</figcaption>
		</figure>

		<div class="text-col">
			<p class="dateline">24 de diciembre de 2020</p>
			<h2>Las primeras vacunas</h2>
			<p>
				México comenzó a vacunar el 24 de diciembre con dosis Pfizer-BioNTech para personal médico
				en la Ciudad de México. Jalisco y Nayarit recibieron sus primeras partidas semanas después.
			</p>
			<p>
				En marzo, la Expo Guadalajara se convirtió en uno de los centros de vacunación masiva más
				grandes del occidente del país. Las filas se documentaban en tiempo real en Twitter e
				Instagram. En abril, brigadas móviles comenzaron a recorrer comunidades de Compostela, Bahía
				de Banderas y la sierra huichola de Nayarit.
			</p>
		</div>
	</section>

	<!-- ═══════════════════════════════════════════════════════════════
	     ▓ HORIZONTAL SCROLLY B · Meses más letales · Jan–Feb 2021
	     ═══════════════════════════════════════════════════════════════ -->
	<div class="scrolly-label">
		<span>enero — febrero 2021</span>
		<h2>Los meses más letales</h2>
		<p>
			Mientras llegaban las vacunas para los médicos, los hospitales vivían su peor momento. Enero y
			febrero de 2021 concentraron los picos de muertes diarias en todo México. En Nayarit y
			Jalisco, el sistema de salud operó semanas al límite.
		</p>
	</div>

	<div class="scrolly-outer" bind:this={outerB}>
		<div class="scrolly-panel" data-phase={stateB.phase}>
			<div class="scrolly-canvas" style={canvasStyle(stateB.offsetX)}>
				{#each scrollyB as el (el.type + el.left)}
					<div
						class="scrolly-el"
						style="left:{el.left}; top:{el.top}; width:{el.width}; height:{el.height}; z-index:{el.z}"
					>
						{#if el.type === 'intro-media'}
							<div class="card-media">
								<header>
									<h4>{el.location}</h4>
									<h5>{el.date}</h5>
								</header>
								<div class="card-inner">
									{#if el.mediaType === 'video'}
										<video class="card-img" src={el.src} autoplay muted loop playsinline></video>
									{:else if el.mediaType === 'image'}
										<img class="card-img" src={el.src} alt={el.caption ?? el.location} />
									{:else}
										<p class="card-text">{el.caption}</p>
									{/if}
								</div>
							</div>
						{:else if el.type === 'snippet'}
							<div class="card-snippet">
								<h4>{el.headline}</h4>
								<p class="dek">{el.dek}</p>
							</div>
						{:else if el.type === 'search'}
							<div class="card-search">
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
				<span>desplázate</span>
				<svg viewBox="0 0 24 24" width="12" height="12">
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

	<!-- ═══════════════════════════════════════════════════════════════
	     SECTION 5 · Delta → Omicron → endemic · mid 2021–2022
	     ═══════════════════════════════════════════════════════════════ -->
	<section class="narrative-block">
		<div class="text-col">
			<p class="dateline">junio 2021 — 2022</p>
			<h2>Delta, Ómicron y el fin del semáforo</h2>
			<p>
				La variante Delta llegó en el verano de 2021 y golpeó con fuerza el corredor turístico
				Vallarta–Nayarit. El municipio de Bahía de Banderas emitió alertas que circularon
				ampliamente en redes; hoteles comenzaron a exigir comprobantes de vacunación.
			</p>
			<p>
				Ómicron disparó los contagios en enero de 2022 — la mayor cifra diaria de toda la pandemia
				en Jalisco — pero la vacunación atenuó la presión hospitalaria. En febrero, el semáforo
				epidemiológico fue retirado. México iniciaba la transición hacia el manejo endémico.
			</p>
			<p>
				En las playas bohemias de Nayarit — Sayulita, San Pancho — los grupos de WhatsApp de
				expatriados y los foros de vecinos en Facebook registraron el último gran debate colectivo
				de la pandemia: cubrebocas en la playa, rentas canceladas, temporada alta en riesgo.
			</p>
		</div>

		<!-- Your photo or social clip: Sayulita/San Pancho Omicron season, Jan 2022 -->
		<!-- Alt: Vallarta-Nayarit corridor COVID signage, Delta wave, Aug 2021 -->
		<!-- Rec: municipio Bahía de Banderas social media posts (free) -->
		<figure class="media-col">
			<div class="media-frame tall">
				<img
					src="/media/sayulita-omicron.jpg"
					alt="Playa de Sayulita durante la temporada de Ómicron, enero 2022."
				/>
			</div>
			<figcaption>Sayulita, Nayarit · ene 2022</figcaption>
		</figure>
	</section>
</article>

<style>
	/* ─── Typography ────────────────────────────────────────────────────────────
	   Matches the Crimson Text + neutral palette of the collage component.
	   ───────────────────────────────────────────────────────────────────────── */
	.covid-narrative {
		font-family: 'Crimson Text', Georgia, serif;
		color: #1a0848;
	}

	/* ─── Narrative block (vertical sections) ───────────────────────────────────
	   Two-column: text + media side by side.
	   .reverse flips the order for visual rhythm.
	   ───────────────────────────────────────────────────────────────────────── */
	.narrative-block {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 3rem;
		align-items: center;
		padding: 5rem 1.5rem;
		border-top: 0.5px solid #e0d8c8;
		max-width: 1100px;
		margin-left: auto;
		margin-right: auto;
	}
	.narrative-block.reverse {
		direction: rtl;
	}
	.narrative-block.reverse > * {
		direction: ltr;
	}

	.text-col {
		max-width: 480px;
	}
	.text-col p {
		font-size: 1.1rem;
		line-height: 1.7;
		color: #3a2e20;
		margin: 0 0 1rem;
	}
	.text-col h2 {
		font-size: 1.9rem;
		font-weight: 700;
		line-height: 1.2;
		margin: 0.25rem 0 1.1rem;
		color: #1a0848;
	}

	.dateline {
		font-size: 0.78rem !important;
		font-style: italic;
		color: #c04040 !important;
		letter-spacing: 0.04em;
		margin: 0 0 0.25rem !important;
		text-transform: lowercase;
	}

	/* ─── Media figures (vertical sections) ────────────────────────────────────*/
	.media-col {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
		margin: 0;
	}
	.media-frame {
		width: 100%;
		aspect-ratio: 4 / 3;
		overflow: hidden;
		border-radius: 8px;
		background: #1a0848;
		box-shadow:
			0 4px 24px rgba(26, 8, 72, 0.15),
			0 1px 4px rgba(26, 8, 72, 0.08);
	}
	.media-frame.tall {
		aspect-ratio: 3 / 4;
	}
	.media-frame img,
	/* Removed unused CSS selector ".media-frame video" */
	figcaption {
		font-size: 0.72rem;
		color: #999;
		font-style: italic;
		line-height: 1.4;
	}

	/* ─── Scrolly section label ─────────────────────────────────────────────────
	   Full-bleed intro text before each horizontal canvas.
	   ───────────────────────────────────────────────────────────────────────── */
	.scrolly-label {
		background: #f5f0e8;
		padding: 4rem 1.5rem 2.5rem;
		border-top: 0.5px solid #e0d8c8;
		max-width: 1100px;
		margin-left: auto;
		margin-right: auto;
	}
	.scrolly-label span {
		display: block;
		font-size: 0.78rem;
		font-style: italic;
		color: #c04040;
		letter-spacing: 0.04em;
		margin-bottom: 0.25rem;
	}
	.scrolly-label h2 {
		font-size: 2rem;
		font-weight: 700;
		color: #1a0848;
		margin: 0 0 0.75rem;
	}
	.scrolly-label p {
		font-size: 1.05rem;
		line-height: 1.65;
		color: #3a2e20;
		max-width: 640px;
		margin: 0;
	}

	/* ─── Horizontal scrolly engine ─────────────────────────────────────────────
	   Identical mechanics to the original component.
	   Height is set by JS (100vh + travel distance).
	   ───────────────────────────────────────────────────────────────────────── */
	.scrolly-outer {
		position: relative;
		/* height injected by JS */
		/* break out of the max-width container so before/after absolute panel spans full viewport */
		width: 100vw;
		left: 50%;
		margin-left: -50vw;
		margin-right: -50vw;
	}

	.scrolly-panel {
		width: 100%;
		height: 100vh;
		overflow: hidden;
		background: #f5f0e8;
	}
	.scrolly-panel[data-phase='before'] {
		position: absolute;
		top: 0;
	}
	.scrolly-panel[data-phase='active'] {
		position: fixed;
		top: 0;
		left: 0;
	}
	.scrolly-panel[data-phase='after'] {
		position: absolute;
		bottom: 0;
	}

	.scrolly-canvas {
		position: absolute;
		top: 0;
		left: 0;
		width: 180vw; /* must match CANVAS_VW in script */
		height: 100%;
		will-change: transform;
		transition: transform 0.06s linear;
	}

	.scrolly-el {
		position: absolute;
	}

	/* ─── Collage card types ─────────────────────────────────────────────────── */
	.card-media {
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
	.card-media header {
		padding: 0.55rem 0.75rem 0.45rem;
		border-bottom: 1px solid #f0ebe0;
		flex-shrink: 0;
	}
	.card-media h4 {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.82rem;
		font-weight: 700;
		color: #c04040;
		margin: 0;
		line-height: 1.2;
	}
	.card-media h5 {
		font-size: 0.68rem;
		color: #999;
		margin: 0;
		font-weight: 400;
		font-family: 'Crimson Text', Georgia, serif;
	}
	.card-inner {
		flex: 1;
		overflow: hidden;
		background: #1a0848;
		min-height: 0;
	}
	.card-img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		display: block;
	}
	.card-text {
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
		line-height: 1.6;
		color: #fff5d8;
		text-align: center;
		font-style: italic;
	}

	.card-snippet {
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
	.card-snippet h4 {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.82rem, 1.15vw, 1rem);
		font-weight: 700;
		color: #1a0848;
		line-height: 1.35;
		margin: 0 0 0.45rem;
	}
	.card-snippet .dek {
		font-size: clamp(0.68rem, 0.85vw, 0.78rem);
		color: #666;
		line-height: 1.5;
		margin: 0;
		font-family: 'Crimson Text', Georgia, serif;
	}

	.card-search {
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
	}
	.card-search .term {
		font-size: clamp(0.72rem, 0.95vw, 0.88rem);
		color: #202124;
		margin: 0;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		font-family: 'Roboto', 'Arial', sans-serif;
	}

	.scroll-nudge {
		position: absolute;
		bottom: 1.5rem;
		right: 2rem;
		display: flex;
		align-items: center;
		gap: 0.35rem;
		color: #a08060;
		font-size: 0.7rem;
		font-family: 'Crimson Text', Georgia, serif;
		font-style: italic;
		letter-spacing: 0.05em;
		pointer-events: none;
		z-index: 200;
	}

	/* ─── Mobile (≤640px) ───────────────────────────────────────────────────────
	   Narrative blocks → single column.
	   Scrolly canvases → simple vertical stack.
	   ───────────────────────────────────────────────────────────────────────── */
	@media (max-width: 640px) {
		.narrative-block {
			grid-template-columns: 1fr;
			gap: 1.5rem;
			padding: 3rem 1.5rem;
		}
		.narrative-block.reverse {
			direction: ltr;
		}
		.media-frame,
		.media-frame.tall {
			aspect-ratio: 4 / 3;
		}
		.text-col h2 {
			font-size: 1.5rem;
		}
		.scrolly-label h2 {
			font-size: 1.5rem;
		}

		.scrolly-panel,
		.scrolly-panel[data-phase='before'],
		.scrolly-panel[data-phase='active'],
		.scrolly-panel[data-phase='after'] {
			position: static;
			height: auto;
			overflow: visible;
			padding: 1rem 0 2rem;
		}
		.scrolly-canvas {
			position: static;
			width: 100%;
			height: auto;
			transform: none !important;
			transition: none;
			display: flex;
			flex-direction: column;
			gap: 1rem;
		}
		.scrolly-el {
			position: static !important;
			width: 100% !important;
			height: auto !important;
		}
		.card-media {
			height: auto;
		}
		.card-inner {
			aspect-ratio: 4 / 3;
			height: auto;
		}
		.card-text {
			height: auto;
			min-height: 160px;
		}
		.card-snippet {
			height: auto;
		}
		.card-search {
			height: 52px;
			border-radius: 12px;
		}
		.scroll-nudge {
			display: none;
		}
	}
</style>
