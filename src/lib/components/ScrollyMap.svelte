<script context="module" lang="ts">
	// DocRenderer invokes this as [[ScrollyMap]] with no props.
</script>

<script lang="ts">
	import { onMount, onDestroy, tick } from 'svelte';
	import { browser } from '$app/environment';
	import { base } from '$app/paths';

	// ── Types ──────────────────────────────────────────────────────────────
	interface Facility {
		clues: string;
		name: string;
		institution: string;
		type: string;
		typology: string;
		level: string;
		municipality: string;
		locality: string;
		lat: number;
		lng: number;
		category: string;
		estrato: string;
	}

	type StepId = 'all' | 'clinics' | 'drain' | 'hospitals' | 'bienestar' | 'explore';

	interface MapStep {
		id: StepId;
		headline: string;
		body: string;
		source?: string;
		sourceUrl?: string;
		stats?: { n: string; label: string }[];
	}

	// ── Steps ──────────────────────────────────────────────────────────────
	const STEPS: MapStep[] = [
		{
			id: 'all',
			headline: "Nayarit's Public Health System",
			body: 'When COVID-19 arrived in rural Nayarit in 2020, it entered a healthcare system already stretched thin. More than 600 active public medical facilities were spread across mountains, valleys, and coast — but access was never equal. These are all the public and private healthcare facilities operating in Nayarit during the pandemic.',
			source: 'CLUES — Catálogo de Unidades de Salud, Secretaría de Salud, 2024',
			sourceUrl:
				'https://www.gob.mx/salud/documentos/datos-abiertos-de-establecimientos-de-salud-clues',
			stats: [
				{ n: '633', label: 'Active facilities' },
				{ n: '116', label: 'IMSS clinics' },
				{ n: '271', label: 'IMSS-Bienestar' },
				{ n: '7', label: 'IMSS hospitals' }
			]
		},
		{
			id: 'clinics',
			headline: 'First-Level Clinics: The First Line of Defense',
			body: 'The bulk of rural healthcare runs through 116 IMSS family medicine units (UMFs) — the first point of contact for most low-income Nayaritas. In Ahuacatlán, the UMF 17 served communities like Valle Verde and Tetitlán. But these clinics were never designed to handle a respiratory pandemic requiring oxygen, ICU beds, or specialist care.',
			source: 'IMSS Directorio de Clínicas, Nayarit, 2020–2021',
			sourceUrl:
				'https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Cl%C3%ADnica',
			stats: [
				{ n: '116', label: 'IMSS clinics (UMF)' },
				{ n: '27', label: 'Municipalities served' }
			]
		},
		{
			id: 'drain',
			headline: 'The Doctor Drain',
			body: "As COVID overwhelmed urban hospitals, the government reassigned rural doctors to support responses in Tepic, Ixtlán del Río, and other cities. Family doctor Ernesto Bueno Cortez left Valle Verde to staff the IMSS branch in Ixtlán. The villages he left behind had to manage alone — or drive an hour for care that wasn't guaranteed to be available.",
			source: 'Interview: Dr. Ernesto Bueno Cortez, Valle Verde, 2024',
			stats: [
				{ n: '7', label: 'IMSS hospitals absorbing COVID patients' },
				{ n: '60%', label: 'IMSS COVID hospitalization mortality' }
			]
		},
		{
			id: 'hospitals',
			headline: 'Only 7 IMSS Hospitals for 1.2 Million People',
			body: "When Jose Jimenez's oxygen dropped, his family drove an hour to reach an IMSS hospital bed. That's because Nayarit had just 7 active IMSS general hospitals — in Tepic, Acaponeta, Tuxpan, Santiago Ixcuintla, Las Varas, and Bahía de Banderas. Rural areas had 1.4 hospital beds per 100,000 people, compared to 80.4 in urban centers.",
			source: 'CLUES / Research: Abascal Miguel et al., UCSF, 2023',
			stats: [
				{ n: '1.4', label: 'Beds per 100K — rural' },
				{ n: '80.4', label: 'Beds per 100K — urban' },
				{ n: '3×', label: 'Lower mortality at private hospitals' }
			]
		},
		{
			id: 'bienestar',
			headline: 'IMSS-Bienestar: Present but Inconsistent',
			body: "IMSS-Bienestar — the program for informal workers and rural residents — operated 257 clinics and 14 small hospitals across Nayarit's most remote communities. Valle Verde itself was served by one. But the 2019–2020 INSABI transition left facilities without clear funding. Families like the Jimenez-Delgados technically had coverage, but the system couldn't reliably deliver.",
			source: 'IMSS-Bienestar Directorio / ANEXO 1, 231 units officially transferred, 2023',
			sourceUrl:
				'https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=IMSS%20Bienestar',
			stats: [
				{ n: '257', label: 'IMSS-Bienestar clinics' },
				{ n: '14', label: 'IMSS-Bienestar hospitals' },
				{ n: '2019', label: 'Year INSABI replaced Seguro Popular' }
			]
		},
		{
			id: 'explore',
			headline: 'Explore the Full Map',
			body: 'Filter the facilities below by institution type. The geography of healthcare in Nayarit reflects the geography of inequality — hospitals and well-staffed clinics cluster near Tepic and the coast, while the Sierra Madre valleys where Valle Verde sits remain the most medically isolated regions in the state.',
			source: 'CLUES + IMSS Directorio + IMSS-Bienestar Directorio, 2024',
			sourceUrl: 'https://www.imss.gob.mx/directorio?dom_estado=Nayarit'
		}
	];

	// ── Visual config ──────────────────────────────────────────────────────
	const W = 900,
		H = 620; // SVG canvas size

	const COLORS: Record<string, string> = {
		imss_clinic: '#c8960a',
		imss_hospital: '#e04040',
		imss_bienestar: '#9b6fcc',
		imss_bienestar_hospital: '#c89fe8',
		ssa: '#4a9b7f',
		issste: '#4a7fb5',
		private: '#555',
		private_hospital: '#777',
		other: '#444',
		other_hospital: '#666'
	};

	const LABELS: Record<string, string> = {
		imss_clinic: 'IMSS Clinic (UMF)',
		imss_hospital: 'IMSS Hospital',
		imss_bienestar: 'IMSS-Bienestar Clinic',
		imss_bienestar_hospital: 'IMSS-Bienestar Hospital',
		ssa: 'State Health (SSA)',
		issste: 'ISSSTE',
		private: 'Private Clinic',
		private_hospital: 'Private Hospital',
		other: 'Other',
		other_hospital: 'Other Hospital'
	};

	function dotRadius(cat: string): number {
		return cat.includes('hospital') ? 6 : 3;
	}

	function getDotOpacity(cat: string, stepId: StepId): number {
		if (stepId === 'all') return 0.85;
		if (stepId === 'clinics') return cat === 'imss_clinic' ? 0.9 : 0.06;
		if (stepId === 'drain')
			return cat === 'imss_hospital' ? 0.95 : cat === 'imss_clinic' ? 0.12 : 0.04;
		if (stepId === 'hospitals') return cat === 'imss_hospital' ? 0.95 : 0.06;
		if (stepId === 'bienestar')
			return cat === 'imss_bienestar' || cat === 'imss_bienestar_hospital' ? 0.9 : 0.06;
		if (stepId === 'explore') return filterState[cat] ? 0.85 : 0;
		return 0.06;
	}

	// ── State ──────────────────────────────────────────────────────────────
	let facilities: Facility[] = [];
	let boundaryGeoJSON: any = null; // raw GeoJSON, passed to D3
	let isLoading = true;
	let mapReady = false; // true once D3 has drawn the SVG

	// D3-projected data (populated after load)
	let boundaryPaths: string[] = [];
	let projectedFacilities: (Facility & { x: number; y: number })[] = [];
	let roadPaths: { d: string; toll: boolean; label: string }[] = [];
	let cityPoints: { name: string; x: number; y: number; capital: boolean }[] = [];
	let valleVerde: [number, number] = [0, 0];

	// Road waypoints [lat, lng]
	const ROAD_WAYPOINTS = [
		{
			label: 'MEX-15 (Free highway)',
			toll: false,
			pts: [
				[23.02, -105.68],
				[22.85, -105.62],
				[22.55, -105.55],
				[22.25, -105.45],
				[21.98, -105.38],
				[21.75, -105.28],
				[21.54, -105.28],
				[21.4, -105.18],
				[21.2, -105.05],
				[21.0, -104.98],
				[20.8, -104.85]
			] as [number, number][]
		},
		{
			label: 'MEX-15D (Toll — cuota)',
			toll: true,
			pts: [
				[22.78, -105.5],
				[22.5, -105.4],
				[22.18, -105.32],
				[21.88, -105.18],
				[21.58, -105.08],
				[21.35, -104.95],
				[21.15, -104.9],
				[20.92, -104.82]
			] as [number, number][]
		},
		{
			label: 'MEX-68 (Tepic–Durango)',
			toll: false,
			pts: [
				[21.5, -104.89],
				[21.55, -104.6],
				[21.6, -104.35],
				[21.65, -104.18],
				[21.72, -104.02]
			] as [number, number][]
		},
		{
			label: 'MEX-161 (Tepic–Acaponeta)',
			toll: false,
			pts: [
				[21.5, -104.89],
				[21.72, -104.9],
				[21.92, -104.92],
				[22.12, -104.95],
				[22.32, -105.08],
				[22.52, -105.2]
			] as [number, number][]
		},
		{
			label: 'MEX-200 (Coastal road)',
			toll: false,
			pts: [
				[20.8, -105.38],
				[21.0, -105.45],
				[21.2, -105.48],
				[21.4, -105.52],
				[21.54, -105.58]
			] as [number, number][]
		}
	];

	const CITY_DEFS = [
		{ name: 'Tepic', lat: 21.5, lng: -104.89, capital: true },
		{ name: 'Acaponeta', lat: 22.5, lng: -105.37, capital: false },
		{ name: 'Ixtlán del Río', lat: 21.03, lng: -104.36, capital: false },
		{ name: 'Bahía de Banderas', lat: 20.75, lng: -105.25, capital: false },
		{ name: 'Santiago Ixcuintla', lat: 21.81, lng: -105.22, capital: false }
	];

	// ── Scroll state ───────────────────────────────────────────────────────
	let bgIndex = 0;
	let released = false,
		prevCarryOut = false,
		carryOut = false;
	let rafPending = false,
		nearViewport = false;
	let sectionEl: HTMLElement | null = null;
	let textBoxEls: (HTMLElement | null)[] = new Array(STEPS.length).fill(null);
	let tooltip: { x: number; y: number; f: Facility } | null = null;

	let filterState: Record<string, boolean> = {
		imss_clinic: true,
		imss_hospital: true,
		imss_bienestar: true,
		imss_bienestar_hospital: true,
		ssa: true,
		issste: false,
		private: false,
		private_hospital: false,
		other: false,
		other_hospital: false
	};

	const CARRY_POINT = 0.5;

	function computePassedIndex(): number {
		let passed = -1;
		const lastIdx = textBoxEls.length - 1;
		const carryY = window.innerHeight * CARRY_POINT;
		for (let i = 0; i < textBoxEls.length; i++) {
			const el = textBoxEls[i];
			if (!el) continue;
			const top = el.getBoundingClientRect().top;
			if (top <= (i === lastIdx ? carryY : 1)) passed = i;
			else break;
		}
		return passed;
	}

	function updateCarryOut() {
		const el = textBoxEls[STEPS.length - 1];
		if (!el) {
			carryOut = released = false;
			prevCarryOut = false;
			return;
		}
		carryOut = el.getBoundingClientRect().top <= window.innerHeight * CARRY_POINT;
		if (carryOut && !prevCarryOut) released = true;
		if (!carryOut && prevCarryOut) released = false;
		prevCarryOut = carryOut;
	}

	function updateFromScroll() {
		rafPending = false;
		bgIndex = Math.min(STEPS.length - 1, Math.max(0, computePassedIndex() + 1));
		updateCarryOut();
	}

	function onScrollOrResize() {
		if (!browser || rafPending) return;
		rafPending = true;
		requestAnimationFrame(updateFromScroll);
	}

	// ── D3 projection setup ───────────────────────────────────────────────
	// Called once data is loaded. Uses d3.geoMercator().fitSize() so the
	// boundary GeoJSON defines the projection — dots use the SAME projector.
	async function setupProjection(d3: any, geo: any) {
		// Build a combined FeatureCollection for fitSize — use all municipality polygons
		const projector = d3.geoMercator().fitSize([W, H], geo);

		const pathGen = d3.geoPath().projection(projector);

		// Boundary paths — one per feature, guarded so one bad polygon never kills the map
		if (geo.type === 'FeatureCollection') {
			boundaryPaths = geo.features
				.map((f: any) => {
					try {
						return pathGen(f);
					} catch {
						return '';
					}
				})
				.filter(Boolean);
		} else {
			try {
				boundaryPaths = [pathGen(geo)].filter(Boolean);
			} catch {
				boundaryPaths = [];
			}
		}

		// Project facilities using the SAME projector
		projectedFacilities = facilities.map((f) => {
			const [x, y] = projector([f.lng, f.lat]) ?? [0, 0];
			return { ...f, x: Math.round(x * 10) / 10, y: Math.round(y * 10) / 10 };
		});

		// Project road waypoints
		roadPaths = ROAD_WAYPOINTS.map((road) => {
			const d =
				'M ' +
				road.pts
					.map(([lat, lng]) => {
						const [x, y] = projector([lng, lat]) ?? [0, 0];
						return `${Math.round(x)},${Math.round(y)}`;
					})
					.join(' L ');
			return { d, toll: road.toll, label: road.label };
		});

		// Project cities
		cityPoints = CITY_DEFS.map((c) => {
			const [x, y] = projector([c.lng, c.lat]) ?? [0, 0];
			return { name: c.name, x: Math.round(x), y: Math.round(y), capital: c.capital };
		});

		// Project Valle Verde
		const [vx, vy] = projector([-104.485, 21.054]) ?? [0, 0];
		valleVerde = [Math.round(vx), Math.round(vy)];

		mapReady = true;
	}

	// ── Lifecycle ──────────────────────────────────────────────────────────
	let d3Loaded = false;
	let observer: IntersectionObserver | null = null;

	onMount(async () => {
		if (!browser) return;
		window.addEventListener('scroll', onScrollOrResize, { passive: true });
		window.addEventListener('resize', onScrollOrResize);

		observer = new IntersectionObserver(
			async (entries) => {
				if (!entries[0].isIntersecting) return;
				observer?.disconnect();
				observer = null;
				nearViewport = true;

				try {
					// Load D3, data, and GeoJSON in parallel
					const [, facRes, geoRes] = await Promise.all([
						loadD3(),
						fetch(`${base}/data/nayarit_medical_centers.json`),
						fetch(`${base}/data/nayarit_municipalities.geojson`)
					]);

					facilities = await facRes.json();
					const geo = await geoRes.json();
					boundaryGeoJSON = geo;

					const d3 = (window as any).d3;
					await setupProjection(d3, geo);
				} catch (e) {
					console.error('[ScrollyMap] load error:', e);
				} finally {
					isLoading = false;
				}

				await tick();
				onScrollOrResize();
			},
			{ rootMargin: '500px 0px' }
		);

		if (sectionEl) observer.observe(sectionEl);
	});

	function loadD3(): Promise<void> {
		if ((window as any).d3) return Promise.resolve();
		return new Promise((resolve, reject) => {
			const s = document.createElement('script');
			s.src = 'https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js';
			s.onload = () => resolve();
			s.onerror = reject;
			document.head.appendChild(s);
		});
	}

	onDestroy(() => {
		if (browser) {
			window.removeEventListener('scroll', onScrollOrResize);
			window.removeEventListener('resize', onScrollOrResize);
		}
		observer?.disconnect();
	});

	// ── Reactive ───────────────────────────────────────────────────────────
	$: activeStep = STEPS[bgIndex] ?? STEPS[0];

	$: categoryCounts = (() => {
		const c: Record<string, number> = {};
		for (const f of facilities) c[f.category] = (c[f.category] ?? 0) + 1;
		return c;
	})();

	$: highlightCount = (() => {
		const id = activeStep.id;
		if (id === 'all') return facilities.length;
		if (id === 'clinics') return categoryCounts['imss_clinic'] ?? 0;
		if (id === 'drain') return categoryCounts['imss_hospital'] ?? 0;
		if (id === 'hospitals') return categoryCounts['imss_hospital'] ?? 0;
		if (id === 'bienestar')
			return (
				(categoryCounts['imss_bienestar'] ?? 0) + (categoryCounts['imss_bienestar_hospital'] ?? 0)
			);
		if (id === 'explore')
			return Object.entries(filterState)
				.filter(([, v]) => v)
				.reduce((a, [k]) => a + (categoryCounts[k] ?? 0), 0);
		return 0;
	})();

	$: legendItems = (() => {
		const id = activeStep.id;
		if (id === 'all' || id === 'explore')
			return ['imss_clinic', 'imss_hospital', 'imss_bienestar', 'imss_bienestar_hospital', 'ssa'];
		if (id === 'clinics') return ['imss_clinic'];
		if (id === 'drain') return ['imss_hospital', 'imss_clinic'];
		if (id === 'hospitals') return ['imss_hospital'];
		if (id === 'bienestar') return ['imss_bienestar', 'imss_bienestar_hospital'];
		return [];
	})();
</script>

<div class="scrolly-map-bleed" bind:this={sectionEl}>
	<!-- Sticky map -->
	<div class={'scrolly-map-bg' + (released ? ' unstick' : '')}>
		{#if !nearViewport || isLoading || !mapReady}
			<div class="map-skeleton">
				<div class="map-spinner"></div>
				<p class="map-skeleton-label">
					{isLoading ? 'Loading Nayarit healthcare map…' : 'Preparing map…'}
				</p>
			</div>
		{:else}
			<svg
				class="map-svg"
				viewBox="0 0 {W} {H}"
				preserveAspectRatio="xMidYMid meet"
				aria-label="Map of Nayarit public healthcare facilities"
				role="img"
			>
				<defs>
					<linearGradient id="smSkyGrad" x1="0" y1="0" x2="0" y2="1">
						<stop offset="0%" stop-color="#0d0520" />
						<stop offset="100%" stop-color="#1a0a38" />
					</linearGradient>
				</defs>

				<rect width={W} height={H} fill="url(#smSkyGrad)" />

				<!-- Municipality fills — D3-projected, guaranteed aligned -->
				{#each boundaryPaths as d}
					<path {d} fill="rgba(42,14,88,0.28)" stroke="none" />
				{/each}

				<!-- Municipality border lines -->
				{#each boundaryPaths as d}
					<path
						{d}
						fill="none"
						stroke="rgba(123,79,166,0.22)"
						stroke-width="0.7"
						stroke-linejoin="round"
					/>
				{/each}

				<!-- Outer glow (same paths, wide soft stroke) -->
				{#each boundaryPaths as d}
					<path
						{d}
						fill="none"
						stroke="rgba(123,79,166,0.1)"
						stroke-width="7"
						stroke-linejoin="round"
					/>
				{/each}

				<!-- Crisp state border -->
				{#each boundaryPaths as d}
					<path
						{d}
						fill="none"
						stroke="rgba(155,111,204,0.7)"
						stroke-width="1.3"
						stroke-linejoin="round"
					/>
				{/each}

				<!-- Roads -->
				{#each roadPaths as road}
					<path
						d={road.d}
						fill="none"
						stroke="rgba(0,0,0,0.4)"
						stroke-width={road.toll ? 3.5 : 2.5}
						stroke-linecap="round"
					/>
					<path
						d={road.d}
						fill="none"
						stroke={road.toll ? '#8b3a3a' : 'rgba(180,150,70,0.5)'}
						stroke-width={road.toll ? 2 : 1.3}
						stroke-linecap="round"
						stroke-dasharray={road.toll ? '9,5' : 'none'}
						opacity={road.toll ? 0.9 : 0.65}
					/>
				{/each}

				<!-- City dots + labels -->
				{#each cityPoints as city}
					<circle
						cx={city.x}
						cy={city.y}
						r={city.capital ? 4 : 2.5}
						fill={city.capital ? '#c8960a' : 'rgba(200,150,10,0.5)'}
						opacity="0.8"
					/>
					<text
						x={city.x + 7}
						y={city.y + 4}
						fill={city.capital ? 'rgba(200,150,10,0.8)' : 'rgba(200,150,10,0.45)'}
						font-family="'Syne', sans-serif"
						font-size={city.capital ? 9 : 7.5}
						font-weight={city.capital ? '700' : '400'}
						letter-spacing="0.05em">{city.name}</text
					>
				{/each}

				<!-- Dim dots -->
				<g>
					{#each projectedFacilities as f (f.clues)}
						{@const op = getDotOpacity(f.category, activeStep.id)}
						{#if op > 0 && op < 0.5}
							<circle
								cx={f.x}
								cy={f.y}
								r={dotRadius(f.category)}
								fill={COLORS[f.category] ?? '#555'}
								opacity={op}
							/>
						{/if}
					{/each}
				</g>

				<!-- Highlight dots -->
				<g>
					{#each projectedFacilities as f (f.clues)}
						{@const op = getDotOpacity(f.category, activeStep.id)}
						{#if op >= 0.5}
							<circle
								cx={f.x}
								cy={f.y}
								r={dotRadius(f.category)}
								fill={COLORS[f.category] ?? '#555'}
								opacity={op}
								class="dot-active"
								role="img"
								aria-label="{f.name}, {f.municipality}"
								on:mouseenter={() => {
									tooltip = { x: f.x, y: f.y, f };
								}}
								on:mouseleave={() => {
									tooltip = null;
								}}
							/>
						{/if}
					{/each}
				</g>

				<!-- Valle Verde story pin -->
				{#if bgIndex > 0}
					<g class="story-pin">
						<circle
							cx={valleVerde[0]}
							cy={valleVerde[1]}
							r="20"
							fill="none"
							stroke="#c8960a"
							stroke-width="1"
							opacity="0.15"
						/>
						<circle
							cx={valleVerde[0]}
							cy={valleVerde[1]}
							r="11"
							fill="none"
							stroke="#c8960a"
							stroke-width="1.5"
							opacity="0.45"
						/>
						<circle cx={valleVerde[0]} cy={valleVerde[1]} r="4.5" fill="#c8960a" opacity="0.95" />
						<text
							x={valleVerde[0] + 15}
							y={valleVerde[1] - 4}
							fill="#c8960a"
							font-family="'Crimson Text', Georgia, serif"
							font-style="italic"
							font-size="11.5"
							opacity="0.95">Valle Verde</text
						>
						<text
							x={valleVerde[0] + 15}
							y={valleVerde[1] + 9}
							fill="rgba(200,150,10,0.5)"
							font-family="'Syne', sans-serif"
							font-size="8"
							letter-spacing="0.1em">AHUACATLÁN</text
						>
					</g>
				{/if}

				<!-- Tooltip -->
				{#if tooltip}
					{@const tx = tooltip.x > W - 200 ? tooltip.x - 196 : tooltip.x + 12}
					{@const ty = tooltip.y > H - 80 ? tooltip.y - 78 : tooltip.y + 8}
					<g>
						<rect
							x={tx - 2}
							y={ty - 2}
							width="194"
							height="74"
							rx="3"
							fill="rgba(8,3,18,0.96)"
							stroke={COLORS[tooltip.f.category] ?? '#c8960a'}
							stroke-width="1"
						/>
						<text
							x={tx + 8}
							y={ty + 15}
							fill="#f0c040"
							font-family="'Syne', sans-serif"
							font-size="8.5"
							font-weight="700"
							>{tooltip.f.name.length > 27
								? tooltip.f.name.slice(0, 26) + '…'
								: tooltip.f.name}</text
						>
						<text
							x={tx + 8}
							y={ty + 29}
							fill="rgba(255,255,255,0.5)"
							font-family="'Syne', sans-serif"
							font-size="7.5">{tooltip.f.typology.slice(0, 33)}</text
						>
						<text
							x={tx + 8}
							y={ty + 43}
							fill="rgba(255,255,255,0.35)"
							font-family="'Syne', sans-serif"
							font-size="7">{tooltip.f.municipality}, Nayarit</text
						>
						<circle
							cx={tx + 12}
							cy={ty + 59}
							r="4"
							fill={COLORS[tooltip.f.category] ?? '#c8960a'}
						/>
						<text
							x={tx + 22}
							y={ty + 63}
							fill="rgba(255,255,255,0.35)"
							font-family="'Syne', sans-serif"
							font-size="7">{LABELS[tooltip.f.category] ?? ''}</text
						>
					</g>
				{/if}
			</svg>

			<!-- Legend overlay -->
			<div class="map-overlay">
				<div class="map-legend">
					{#each legendItems as cat}
						<div class="legend-row">
							<span class="legend-dot" style="background:{COLORS[cat]}"></span>
							<span class="legend-label">{LABELS[cat]}</span>
						</div>
					{/each}
					<div class="legend-road-row">
						<span class="legend-road legend-road--toll"></span>
						<span class="legend-label">Toll road (cuota)</span>
					</div>
					<div class="legend-road-row">
						<span class="legend-road legend-road--free"></span>
						<span class="legend-label">Free highway</span>
					</div>
				</div>
				<div class="map-count">
					<span class="count-n">{highlightCount}</span>
					<span class="count-label">shown</span>
				</div>
			</div>
		{/if}
	</div>

	<!-- Steps -->
	<div class="scrolly-map-steps">
		{#each STEPS as step, i}
			{#if step.id !== 'explore'}
				<div class="map-step" class:map-step--active={bgIndex === i} bind:this={textBoxEls[i]}>
					<div class="step-box">
						<div class="step-num">{String(i + 1).padStart(2, '0')}</div>
						<h3 class="step-hed">{step.headline}</h3>
						<p class="step-body">{step.body}</p>
						{#if step.stats}
							<div class="step-stats">
								{#each step.stats as s}
									<div class="step-stat">
										<span class="stat-n">{s.n}</span>
										<span class="stat-l">{s.label}</span>
									</div>
								{/each}
							</div>
						{/if}
						{#if step.source}
							<p class="step-source">
								{#if step.sourceUrl}
									<a href={step.sourceUrl} target="_blank" rel="noopener noreferrer"
										>{step.source} ↗</a
									>
								{:else}{step.source}{/if}
							</p>
						{/if}
					</div>
				</div>
			{:else}
				<div class="map-step map-step--explore" bind:this={textBoxEls[i]}>
					<div class="step-box step-box--wide">
						<div class="step-num">{String(i + 1).padStart(2, '0')}</div>
						<h3 class="step-hed">{step.headline}</h3>
						<p class="step-body">{step.body}</p>
						<div class="filter-list">
							{#each Object.entries(LABELS) as [cat, label]}
								<label class="filter-item" class:filter-item--on={filterState[cat]}>
									<input type="checkbox" bind:checked={filterState[cat]} />
									<span class="filter-swatch" style="background:{COLORS[cat]}"></span>
									<span class="filter-name">{label}</span>
									<span class="filter-n">{categoryCounts[cat] ?? 0}</span>
								</label>
							{/each}
						</div>
						<div class="sources-block">
							<p class="sources-hed">Data Sources</p>
							<ul class="sources-list">
								<li>
									<a
										href="https://www.gob.mx/salud/documentos/datos-abiertos-de-establecimientos-de-salud-clues"
										target="_blank"
										rel="noopener noreferrer"
										>CLUES — Catálogo Nacional de Establecimientos de Salud ↗</a
									>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Cl%C3%ADnica"
										target="_blank"
										rel="noopener noreferrer">IMSS Directorio — Clínicas, Nayarit ↗</a
									>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Hospital"
										target="_blank"
										rel="noopener noreferrer">IMSS Directorio — Hospitales, Nayarit ↗</a
									>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=IMSS%20Bienestar"
										target="_blank"
										rel="noopener noreferrer">IMSS-Bienestar Directorio, Nayarit ↗</a
									>
								</li>
								<li>
									INEGI Marco Geoestadístico 2023 — Shapefile 18mun (Nayarit municipalities,
									LCC→WGS84)
								</li>
								<li>
									ANEXO 1 — Listado Oficial de Unidades Transferidas al IMSS-Bienestar, Nayarit
									(2023)
								</li>
							</ul>
						</div>
					</div>
				</div>
			{/if}
		{/each}
	</div>
</div>

<style>
	.scrolly-map-bleed {
		position: relative;
		width: 100vw;
		left: 50%;
		margin-left: -50vw;
		box-sizing: border-box;
	}
	.scrolly-map-bg {
		position: sticky;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		z-index: 0;
		background: #0d0520;
		margin-bottom: 2rem;
	}
	.scrolly-map-bg.unstick {
		position: relative;
	}
	.map-svg {
		width: 100%;
		height: 100%;
		display: block;
	}

	.map-skeleton {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 1.25rem;
		background: #0d0520;
	}
	.map-spinner {
		width: 52px;
		height: 52px;
		border-radius: 50%;
		border: 2px solid rgba(123, 79, 166, 0.2);
		border-top-color: #c8960a;
		animation: spin 0.9s linear infinite;
	}
	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}
	.map-skeleton-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.62rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.2);
		margin: 0;
	}

	.dot-active {
		cursor: pointer;
		transition: opacity 0.4s ease;
	}
	.dot-active:hover {
		opacity: 1 !important;
		filter: brightness(1.5);
	}
	.story-pin {
		animation: pinPulse 2.4s ease-in-out infinite;
	}
	@keyframes pinPulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.6;
		}
	}

	.map-overlay {
		position: absolute;
		bottom: 1.5rem;
		left: 1.75rem;
		display: flex;
		align-items: flex-end;
		gap: 2.5rem;
		pointer-events: none;
	}
	.map-legend {
		display: flex;
		flex-direction: column;
		gap: 0.28rem;
	}
	.legend-row,
	.legend-road-row {
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}
	.legend-dot {
		width: 8px;
		height: 8px;
		border-radius: 50%;
		flex-shrink: 0;
	}
	.legend-road {
		width: 18px;
		height: 2px;
		flex-shrink: 0;
	}
	.legend-road--toll {
		background: #8b3a3a;
	}
	.legend-road--free {
		background: rgba(180, 150, 70, 0.6);
	}
	.legend-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.57rem;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.45);
	}
	.map-count {
		display: flex;
		flex-direction: column;
		align-items: flex-end;
		gap: 0.1rem;
	}
	.count-n {
		font-family: 'Playfair Display', Georgia, serif;
		font-size: 2.4rem;
		font-weight: 900;
		color: rgba(200, 150, 10, 0.65);
		line-height: 1;
	}
	.count-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.52rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.2);
	}

	.scrolly-map-steps {
		position: relative;
		z-index: 1;
		pointer-events: none;
		margin-top: -100vh;
	}
	.map-step {
		min-height: 100vh;
		display: flex;
		align-items: center;
		padding: 2rem 3rem;
		pointer-events: none;
	}
	.map-step--explore {
		justify-content: flex-end;
	}
	.step-box {
		background: rgba(10, 4, 20, 0.86);
		backdrop-filter: blur(14px);
		-webkit-backdrop-filter: blur(14px);
		border: 1px solid rgba(123, 79, 166, 0.2);
		border-radius: 4px;
		padding: 1.75rem;
		max-width: 360px;
		pointer-events: all;
		box-shadow: 0 12px 40px rgba(0, 0, 0, 0.6);
		transition: border-color 0.3s ease;
	}
	.step-box--wide {
		max-width: 420px;
	}
	.map-step--active .step-box {
		border-color: rgba(200, 150, 10, 0.3);
	}
	.step-num {
		font-family: 'Syne', sans-serif;
		font-size: 0.56rem;
		letter-spacing: 0.26em;
		color: #c8960a;
		font-weight: 700;
		margin-bottom: 0.55rem;
		text-transform: uppercase;
	}
	.step-hed {
		font-family: 'Playfair Display', Georgia, serif;
		font-size: clamp(1.05rem, 2vw, 1.4rem);
		font-weight: 700;
		color: #f0e8d0;
		line-height: 1.25;
		margin: 0 0 0.8rem;
	}
	.step-body {
		font-family: 'Source Serif 4', Georgia, serif;
		font-size: 0.88rem;
		line-height: 1.78;
		color: rgba(255, 255, 255, 0.65);
		margin: 0 0 1rem;
	}
	.step-stats {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
		gap: 0.6rem;
		border-top: 1px solid rgba(200, 150, 10, 0.18);
		padding-top: 0.8rem;
		margin-bottom: 0.8rem;
	}
	.step-stat {
		display: flex;
		flex-direction: column;
		gap: 0.12rem;
	}
	.stat-n {
		font-family: 'Playfair Display', serif;
		font-size: 1.5rem;
		font-weight: 900;
		color: #c8960a;
		line-height: 1;
	}
	.stat-l {
		font-family: 'Syne', sans-serif;
		font-size: 0.53rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.35);
		line-height: 1.3;
	}
	.step-source {
		font-family: 'Syne', sans-serif;
		font-size: 0.56rem;
		letter-spacing: 0.06em;
		color: rgba(255, 255, 255, 0.22);
		border-left: 2px solid rgba(200, 150, 10, 0.22);
		padding-left: 0.5rem;
		margin: 0;
		line-height: 1.5;
	}
	.step-source a {
		color: rgba(200, 150, 10, 0.5);
		text-decoration: none;
	}
	.step-source a:hover {
		color: #c8960a;
		text-decoration: underline;
	}
	.filter-list {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		margin: 0.85rem 0;
	}
	.filter-item {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		cursor: pointer;
		padding: 0.28rem 0.4rem;
		border-radius: 3px;
		opacity: 0.32;
		transition:
			opacity 0.15s,
			background 0.15s;
	}
	.filter-item--on {
		opacity: 1;
	}
	.filter-item:hover {
		background: rgba(255, 255, 255, 0.05);
		opacity: 1;
	}
	.filter-item input {
		display: none;
	}
	.filter-swatch {
		width: 10px;
		height: 10px;
		border-radius: 50%;
		flex-shrink: 0;
	}
	.filter-name {
		flex: 1;
		font-family: 'Syne', sans-serif;
		font-size: 0.67rem;
		letter-spacing: 0.05em;
		color: rgba(255, 255, 255, 0.72);
	}
	.filter-n {
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		color: rgba(255, 255, 255, 0.28);
		min-width: 26px;
		text-align: right;
	}
	.sources-block {
		border-top: 1px solid rgba(255, 255, 255, 0.06);
		padding-top: 0.9rem;
		margin-top: 0.3rem;
	}
	.sources-hed {
		font-family: 'Syne', sans-serif;
		font-size: 0.56rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(200, 150, 10, 0.45);
		margin: 0 0 0.45rem;
	}
	.sources-list {
		list-style: none;
		padding: 0;
		margin: 0;
		display: flex;
		flex-direction: column;
		gap: 0.3rem;
	}
	.sources-list li {
		font-family: 'Syne', sans-serif;
		font-size: 0.58rem;
		color: rgba(255, 255, 255, 0.24);
		line-height: 1.5;
	}
	.sources-list a {
		color: rgba(200, 150, 10, 0.45);
		text-decoration: none;
	}
	.sources-list a:hover {
		color: #c8960a;
		text-decoration: underline;
	}
	@media (max-width: 767px) {
		.map-step {
			align-items: flex-end;
			padding: 1.25rem 1rem 2rem;
		}
		.map-step--explore {
			justify-content: flex-start;
		}
		.step-box,
		.step-box--wide {
			max-width: 100%;
		}
		.map-overlay {
			bottom: auto;
			top: 1rem;
		}
	}
</style>
