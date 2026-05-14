<script context="module" lang="ts">
	// Empty module block — keeps this consistent with other glob-registered components.
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

	// ── Map constants ──────────────────────────────────────────────────────
	// Bounds calibrated to fit the full Nayarit state outline
	const B = {
		minLat: 20.45,
		maxLat: 23.2,
		minLng: -106.2,
		maxLng: -103.6
	};
	const W = 800,
		H = 600;

	function proj(lat: number, lng: number): [number, number] {
		const x = ((lng - B.minLng) / (B.maxLng - B.minLng)) * W;
		const y = H - ((lat - B.minLat) / (B.maxLat - B.minLat)) * H;
		return [Math.round(x * 10) / 10, Math.round(y * 10) / 10];
	}

	// Nayarit state boundary (clockwise from NW)
	const NAYARIT_BOUNDARY =
		'M 147.7,26.2 L 200.0,21.8 L 230.8,30.5 L 307.7,48.0 L 384.6,61.1 L 455.4,76.4 L 516.9,98.2 L 560.0,120.0 L 584.6,148.4 L 621.5,185.5 L 652.3,222.5 L 701.5,266.2 L 732.3,309.8 L 744.6,353.5 L 732.3,397.1 L 713.8,440.7 L 683.1,475.6 L 652.3,506.2 L 590.8,541.1 L 529.2,556.4 L 455.4,562.9 L 393.8,556.4 L 332.3,549.8 L 270.8,541.1 L 209.2,519.3 L 160.0,497.5 L 129.2,462.5 L 116.9,418.9 L 98.5,375.3 L 86.2,331.6 L 67.7,288.0 L 55.4,244.4 L 46.2,200.7 L 55.4,157.1 L 67.7,113.5 L 86.2,82.9 L 116.9,54.5 L 147.7,26.2 Z';

	// Major roads
	const ROADS = [
		{
			id: 'mex15_free',
			label: 'MEX-15 (Free highway)',
			d: 'M 160.0,39.3 L 178.5,76.4 L 200.0,141.8 L 230.8,207.3 L 252.3,266.2 L 283.1,316.4 L 283.1,362.2 L 313.8,392.7 L 353.8,436.4 L 375.4,480.0 L 415.4,523.6',
			toll: false
		},
		{
			id: 'mex15d_toll',
			label: 'MEX-15D (Toll highway — cuota)',
			d: 'M 215.4,91.6 L 246.2,152.7 L 270.8,222.5 L 313.8,288.0 L 344.6,353.5 L 384.6,403.6 L 400.0,447.3 L 424.6,497.5',
			toll: true
		},
		{
			id: 'mex68',
			label: 'MEX-68 (Tepic–Durango)',
			d: 'M 403.1,370.9 L 492.3,360.0 L 569.2,349.1 L 621.5,338.2 L 670.8,322.9',
			toll: false
		},
		{
			id: 'mex161',
			label: 'MEX-161 (Tepic–Acaponeta)',
			d: 'M 403.1,370.9 L 400.0,322.9 L 393.8,279.3 L 384.6,235.6 L 344.6,192.0 L 307.7,148.4',
			toll: false
		},
		{
			id: 'coastal',
			label: 'MEX-200 (Coastal road)',
			d: 'M 252.3,523.6 L 230.8,480.0 L 221.5,436.4 L 209.2,392.7 L 190.8,362.2',
			toll: false
		}
	];

	// City labels
	const CITIES = [
		{ name: 'Tepic', lat: 21.5, lng: -104.89, capital: true },
		{ name: 'Acaponeta', lat: 22.5, lng: -105.37, capital: false },
		{ name: 'Ixtlán del Río', lat: 21.03, lng: -104.36, capital: false },
		{ name: 'Bahía de Banderas', lat: 20.75, lng: -105.25, capital: false },
		{ name: 'Santiago Ixcuintla', lat: 21.81, lng: -105.22, capital: false }
	];

	// Valle Verde story pin
	const VALLE_VERDE = proj(21.054, -104.485);

	// ── Color + style config ───────────────────────────────────────────────
	const COLORS: Record<string, string> = {
		imss_clinic: '#c8960a',
		imss_hospital: '#e04040',
		imss_bienestar: '#9b6fcc',
		imss_bienestar_hospital: '#c89fe8',
		ssa: '#4a9b7f',
		issste: '#4a7fb5',
		private: '#666',
		private_hospital: '#888',
		other: '#555',
		other_hospital: '#777'
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

	// ── Scroll state (mirrors Scrolly.svelte exactly) ─────────────────────
	let facilities: Facility[] = [];
	let isLoading = true;
	let bgIndex = 0;
	let released = false;
	let prevCarryOut = false;
	let carryOut = false;
	let rafPending = false;
	let nearViewport = false;

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
			const threshold = i === lastIdx ? carryY : 1;
			if (top <= threshold) passed = i;
			else break;
		}
		return passed;
	}

	function updateCarryOut() {
		const el = textBoxEls[STEPS.length - 1];
		if (!el) {
			carryOut = false;
			released = false;
			prevCarryOut = false;
			return;
		}
		const top = el.getBoundingClientRect().top;
		carryOut = top <= window.innerHeight * CARRY_POINT;
		if (carryOut && !prevCarryOut) released = true;
		if (!carryOut && prevCarryOut) released = false;
		prevCarryOut = carryOut;
	}

	function updateFromScroll() {
		rafPending = false;
		const passed = computePassedIndex();
		bgIndex = Math.min(STEPS.length - 1, Math.max(0, passed + 1));
		updateCarryOut();
	}

	function onScrollOrResize() {
		if (!browser || rafPending) return;
		rafPending = true;
		requestAnimationFrame(updateFromScroll);
	}

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
					const res = await fetch(`${base}/data/nayarit_medical_centers.json`);
					facilities = await res.json();
				} catch (e) {
					console.error('[ScrollyMap] Failed to load:', e);
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

	onDestroy(() => {
		if (browser) {
			window.removeEventListener('scroll', onScrollOrResize);
			window.removeEventListener('resize', onScrollOrResize);
		}
		observer?.disconnect();
	});

	// ── Derived ────────────────────────────────────────────────────────────
	$: activeStep = STEPS[bgIndex] ?? STEPS[0];

	$: projected = facilities.map((f) => {
		const [x, y] = proj(f.lat, f.lng);
		return { ...f, x, y };
	});

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
				.reduce((acc, [k]) => acc + (categoryCounts[k] ?? 0), 0);
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

<!-- ═══════════════════════════════════════════════════════════════════
       TEMPLATE
       Full-bleed override: negative margins break out of content-wrapper.
       Mirrors Scrolly.svelte's sticky-bg + scrolly-steps DOM structure.
  ════════════════════════════════════════════════════════════════════ -->
<div class="scrolly-map-bleed" bind:this={sectionEl}>
	<!-- Sticky map — same as .scrolly-bg in Scrolly.svelte -->
	<div class={'scrolly-map-bg' + (released ? ' unstick' : '')}>
		{#if !nearViewport || isLoading}
			<div class="map-skeleton">
				<div class="map-spinner"></div>
				<p class="map-skeleton-label">Loading Nayarit healthcare map…</p>
			</div>
		{:else}
			<svg
				class="map-svg"
				viewBox="0 0 {W} {H}"
				preserveAspectRatio="xMidYMid meet"
				aria-label="Map of Nayarit public healthcare facilities"
				role="img"
			>
				<!-- ── Base layer: sky gradient ── -->
				<defs>
					<linearGradient id="skyGrad" x1="0" y1="0" x2="0" y2="1">
						<stop offset="0%" stop-color="#0d0520" />
						<stop offset="100%" stop-color="#1a0a38" />
					</linearGradient>
					<filter id="glow">
						<feGaussianBlur stdDeviation="2" result="coloredBlur" />
						<feMerge><feMergeNode in="coloredBlur" /><feMergeNode in="SourceGraphic" /></feMerge>
					</filter>
				</defs>

				<rect width={W} height={H} fill="url(#skyGrad)" />

				<!-- ── Nayarit state boundary ── -->
				<!-- Background fill — interior of state -->
				<path d={NAYARIT_BOUNDARY} fill="rgba(42,14,88,0.35)" stroke="none" />
				<!-- Outer glow -->
				<path
					d={NAYARIT_BOUNDARY}
					fill="none"
					stroke="rgba(123,79,166,0.25)"
					stroke-width="6"
					stroke-linejoin="round"
				/>
				<!-- Crisp border -->
				<path
					d={NAYARIT_BOUNDARY}
					fill="none"
					stroke="rgba(123,79,166,0.65)"
					stroke-width="1.5"
					stroke-linejoin="round"
				/>
				<!-- State label -->
				<text
					x="200"
					y="290"
					fill="rgba(123,79,166,0.3)"
					font-family="'Syne', sans-serif"
					font-size="22"
					font-weight="800"
					letter-spacing="0.25em"
					transform="rotate(-65, 200, 290)">NAYARIT</text
				>

				<!-- ── Roads ── -->
				{#each ROADS as road}
					<!-- Road shadow -->
					<path
						d={road.d}
						fill="none"
						stroke="rgba(0,0,0,0.4)"
						stroke-width={road.toll ? 3 : 2}
						stroke-linecap="round"
						stroke-linejoin="round"
					/>
					<!-- Road line -->
					<path
						d={road.d}
						fill="none"
						stroke={road.toll ? '#8b3a3a' : 'rgba(180,150,80,0.45)'}
						stroke-width={road.toll ? 2 : 1.2}
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-dasharray={road.toll ? '8,4' : 'none'}
						opacity={road.toll ? 0.8 : 0.6}
					/>
				{/each}

				<!-- ── City labels ── -->
				{#each CITIES as city}
					{@const [cx, cy] = proj(city.lat, city.lng)}
					<circle
						{cx}
						{cy}
						r={city.capital ? 4 : 2.5}
						fill={city.capital ? '#c8960a' : 'rgba(200,150,10,0.5)'}
						opacity="0.7"
					/>
					<text
						x={cx + 7}
						y={cy + 4}
						fill={city.capital ? 'rgba(200,150,10,0.7)' : 'rgba(200,150,10,0.4)'}
						font-family="'Syne', sans-serif"
						font-size={city.capital ? 9 : 7.5}
						font-weight={city.capital ? '700' : '400'}
						letter-spacing="0.06em">{city.name}</text
					>
				{/each}

				<!-- ── Facility dots: dim layer ── -->
				<g>
					{#each projected as f (f.clues)}
						{@const op = getDotOpacity(f.category, activeStep.id)}
						{#if op < 0.5 && op > 0}
							<circle
								cx={f.x}
								cy={f.y}
								r={dotRadius(f.category)}
								fill={COLORS[f.category] ?? '#666'}
								opacity={op}
							/>
						{/if}
					{/each}
				</g>

				<!-- ── Facility dots: highlight layer ── -->
				<g>
					{#each projected as f (f.clues)}
						{@const op = getDotOpacity(f.category, activeStep.id)}
						{#if op >= 0.5}
							<circle
								cx={f.x}
								cy={f.y}
								r={dotRadius(f.category)}
								fill={COLORS[f.category] ?? '#666'}
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

				<!-- ── Valle Verde story pin ── -->
				{#if bgIndex > 0}
					<g class="story-pin">
						<circle
							cx={VALLE_VERDE[0]}
							cy={VALLE_VERDE[1]}
							r="18"
							fill="none"
							stroke="#c8960a"
							stroke-width="1"
							opacity="0.2"
						/>
						<circle
							cx={VALLE_VERDE[0]}
							cy={VALLE_VERDE[1]}
							r="10"
							fill="none"
							stroke="#c8960a"
							stroke-width="1.5"
							opacity="0.5"
						/>
						<circle
							cx={VALLE_VERDE[0]}
							cy={VALLE_VERDE[1]}
							r="4"
							fill="#c8960a"
							opacity="0.95"
							filter="url(#glow)"
						/>
						<text
							x={VALLE_VERDE[0] + 14}
							y={VALLE_VERDE[1] - 4}
							fill="#c8960a"
							font-family="'Crimson Text', Georgia, serif"
							font-style="italic"
							font-size="11"
							opacity="0.95">Valle Verde</text
						>
						<text
							x={VALLE_VERDE[0] + 14}
							y={VALLE_VERDE[1] + 8}
							fill="rgba(200,150,10,0.55)"
							font-family="'Syne', sans-serif"
							font-size="7.5"
							letter-spacing="0.1em">AHUACATLÁN</text
						>
					</g>
				{/if}

				<!-- ── Tooltip ── -->
				{#if tooltip}
					{@const tx = tooltip.x > W - 195 ? tooltip.x - 192 : tooltip.x + 12}
					{@const ty = tooltip.y > H - 78 ? tooltip.y - 76 : tooltip.y + 8}
					<g>
						<rect
							x={tx - 2}
							y={ty - 2}
							width="190"
							height="72"
							rx="3"
							fill="rgba(10,4,20,0.95)"
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
							>{tooltip.f.name.length > 26
								? tooltip.f.name.slice(0, 25) + '…'
								: tooltip.f.name}</text
						>
						<text
							x={tx + 8}
							y={ty + 29}
							fill="rgba(255,255,255,0.5)"
							font-family="'Syne', sans-serif"
							font-size="7.5">{tooltip.f.typology.slice(0, 32)}</text
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
							cy={ty + 58}
							r="4"
							fill={COLORS[tooltip.f.category] ?? '#c8960a'}
						/>
						<text
							x={tx + 22}
							y={ty + 62}
							fill="rgba(255,255,255,0.35)"
							font-family="'Syne', sans-serif"
							font-size="7">{LABELS[tooltip.f.category] ?? ''}</text
						>
					</g>
				{/if}
			</svg>

			<!-- Map overlay: legend + count -->
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

	<!-- Scrolly steps — same as .scrolly-steps in Scrolly.svelte -->
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
									<a href={step.sourceUrl} target="_blank" rel="noopener noreferrer">
										{step.source} ↗
									</a>
								{:else}
									{step.source}
								{/if}
							</p>
						{/if}
					</div>
				</div>
			{:else}
				<!-- Explore / filter step -->
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
									>
										CLUES — Catálogo Nacional de Establecimientos de Salud ↗
									</a>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Cl%C3%ADnica"
										target="_blank"
										rel="noopener noreferrer"
									>
										IMSS Directorio — Clínicas, Nayarit ↗
									</a>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Hospital"
										target="_blank"
										rel="noopener noreferrer"
									>
										IMSS Directorio — Hospitales, Nayarit ↗
									</a>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=IMSS%20Bienestar"
										target="_blank"
										rel="noopener noreferrer"
									>
										IMSS-Bienestar Directorio, Nayarit ↗
									</a>
								</li>
								<li>
									ANEXO 1 — Listado Oficial de Unidades Transferidas al IMSS-Bienestar, Nayarit (231
									unidades, Gobierno de México, 2023)
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
	/* ═══════════════════════════════════════════════════════════════════
       FULL BLEED — breaks out of .content-wrapper's Bootstrap column.
       Uses negative margins matching the column's left offset.
       This is the same technique used by .full-bleed in Geography.svelte.
    ════════════════════════════════════════════════════════════════════ */
	.scrolly-map-bleed {
		position: relative;
		/* Break out of any centered container */
		width: 100vw;
		left: 50%;
		margin-left: -50vw;
		/* Reset so content inside uses normal flow */
		box-sizing: border-box;
	}

	/* ── Sticky map background — mirrors .scrolly-bg exactly ── */
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

	/* ── SVG fills entire sticky container ── */
	.map-svg {
		width: 100%;
		height: 100%;
		display: block;
	}

	/* ── Skeleton ── */
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

	/* ── Interactive dots ── */
	.dot-active {
		cursor: pointer;
		transition: opacity 0.4s ease;
	}
	.dot-active:hover {
		opacity: 1 !important;
		filter: brightness(1.4);
	}

	/* ── Valle Verde pin pulse ── */
	.story-pin {
		animation: pinPulse 2.4s ease-in-out infinite;
	}
	@keyframes pinPulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.65;
		}
	}

	/* ── Map overlay (bottom-left) ── */
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
		border-top: 1px dashed #8b3a3a;
	}
	.legend-road--free {
		background: rgba(180, 150, 80, 0.55);
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

	/* ── Steps column — same as .scrolly-steps ── */
	.scrolly-map-steps {
		position: relative;
		z-index: 1;
		pointer-events: none;
		/* Negative margin pulls steps up to overlay the sticky bg */
		margin-top: -100vh;
	}

	/* ── Individual step ── */
	.map-step {
		min-height: 100vh;
		display: flex;
		align-items: center;
		padding: 2rem 3rem;
		pointer-events: none;
	}

	/* Explore step: right-aligned */
	.map-step--explore {
		justify-content: flex-end;
	}

	/* ── Step card ── */
	.step-box {
		background: rgba(10, 4, 20, 0.86);
		backdrop-filter: blur(14px);
		-webkit-backdrop-filter: blur(14px);
		border: 1px solid rgba(123, 79, 166, 0.2);
		border-radius: 4px;
		padding: 1.75rem 1.75rem;
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

	/* ── Step typography ── */
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

	/* ── Stats ── */
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

	/* ── Source ── */
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

	/* ── Filter toggles ── */
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

	/* ── Sources ── */
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

	/* ── Responsive ── */
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
