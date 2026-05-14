<script context="module" lang="ts">
	// DocRenderer invokes this as [[ScrollyMap]] with no props.
</script>

<script lang="ts">
	import { onMount, onDestroy, tick } from 'svelte';
	import { browser } from '$app/environment';
	import { base } from '$app/paths';

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

	type StepId =
		| 'all'
		| 'clinics'
		| 'drain'
		| 'hospitals'
		| 'bienestar'
		| 'bienestar_hospitals_only'
		| 'bienestar_hospitals'
		| 'explore';

	interface MapStep {
		id: StepId;
		headline: string;
		body: string;
		source?: string;
		sourceUrl?: string;
		stats?: { n: string; label: string }[];
	}

	// ── Steps ─────────────────────────────────────────────────────────────────
	const STEPS: MapStep[] = [
		{
			id: 'all',
			headline: "Nayarit's Public Health System",
			body: 'When COVID-19 arrived in rural Nayarit in 2020, it entered a healthcare system already stretched thin. More than 600 active medical facilities were spread across mountains, valleys, and coast — but access was never equal.',
			source: 'CLUES — Catálogo de Unidades de Salud, Secretaría de Salud, 2024',
			sourceUrl:
				'https://www.gob.mx/salud/documentos/datos-abiertos-de-establecimientos-de-salud-clues',
			stats: [{ n: '633', label: 'Total active facilities' }]
		},
		{
			id: 'clinics',
			headline: 'First-Level Clinics: The First Line of Defense',
			body: 'The bulk of rural healthcare runs through 116 IMSS family medicine units — the first point of contact for most low-income Nayaritas. In Ahuacatlán, UMF 17 served communities like Valle Verde and Tetitlán. These clinics were not built to handle a respiratory pandemic requiring oxygen or intensive care.',
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
			body: "As COVID overwhelmed hospitals in Tepic and Ixtlán del Río, the government reassigned rural doctors to staff them. The villages they left behind had to manage alone — or make the long drive for care that wasn't guaranteed to be available.",
			source: 'CLUES / IMSS Directorio, Nayarit, 2020–2023',
			stats: [{ n: '7', label: 'IMSS hospitals absorbing COVID patients' }]
		},
		{
			id: 'hospitals',
			headline: 'Only 7 IMSS Hospitals for 1.2 Million People',
			body: "When Jose Jimenez's oxygen dropped, his family drove to the IMSS-Bienestar hospital in Ahuacatlán. When his wife Augustina fell ill days later, she was driven further — north to the IMSS general hospital in Tepic. Nayarit had just 7 active IMSS general hospitals, all concentrated far from rural communities.",
			source: 'CLUES / IMSS Directorio, Nayarit, 2020–2023',
			sourceUrl: 'https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Hospital',
			stats: [
				{ n: '7', label: 'IMSS general hospitals' },
				{ n: '1.2M', label: 'State population' }
			]
		},
		{
			id: 'bienestar',
			headline: 'IMSS-Bienestar: Widespread but Underpowered',
			body: "IMSS-Bienestar — designed for informal workers and rural residents — operated 257 clinics and 14 small hospitals across Nayarit's most remote communities. Valle Verde was served by one. But most of these facilities were outpatient only. When the pandemic required oxygen or intensive care, they couldn't deliver.",
			source: 'IMSS-Bienestar Directorio / ANEXO 1, 2023',
			sourceUrl:
				'https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=IMSS%20Bienestar',
			stats: [
				{ n: '257', label: 'IMSS-Bienestar clinics' },
				{ n: '14', label: 'IMSS-Bienestar hospitals' }
			]
		},
		{
			id: 'bienestar_hospitals_only',
			headline: 'The 14 IMSS-Bienestar Hospitals',
			body: 'IMSS-Bienestar operated 14 small hospitals — more than double the IMSS count — but scattered across a vast state. They provided some intermediate care, but most lacked the oxygen reserves, ICU beds, and specialist staff that COVID patients needed.',
			source: 'IMSS-Bienestar Directorio / ANEXO 1, 2023',
			stats: [{ n: '14', label: 'IMSS-Bienestar hospitals' }]
		},
		{
			id: 'bienestar_hospitals',
			headline: "When Clinics Weren't Enough",
			body: 'Together, just 21 hospital-level facilities served the entire state — 7 IMSS general and 14 IMSS-Bienestar. Families who needed hospital care had to travel, often far, hoping a bed would be available. The geography of hospitals is the geography of inequality.',
			source: 'CLUES + IMSS Directorio + IMSS-Bienestar Directorio, 2024',
			stats: [
				{ n: '7', label: 'IMSS hospitals' },
				{ n: '14', label: 'Bienestar hospitals' },
				{ n: '21', label: 'Total hospitals' }
			]
		},
		{
			id: 'explore',
			headline: 'Explore the Map',
			body: 'Filter by institution type. Hospitals cluster near Tepic and the coast. The Sierra Madre valleys — where Valle Verde sits — are the most medically isolated regions in the state.',
			source: 'CLUES + IMSS Directorio + IMSS-Bienestar Directorio, 2024',
			sourceUrl: 'https://www.imss.gob.mx/directorio?dom_estado=Nayarit'
		}
	];

	// ── Colors ────────────────────────────────────────────────────────────────
	const COLORS: Record<string, string> = {
		imss_clinic: '#FBBF24', // amber triangle
		imss_hospital: '#F87171', // coral red cross
		imss_bienestar: '#A78BFA', // lavender diamond
		imss_bienestar_hospital: '#C4B5FD', // light lavender cross
		ssa: '#34D399', // emerald square
		issste: '#60A5FA', // sky blue square
		private: '#6B7280', // gray circle
		private_hospital: '#9CA3AF', // light gray cross
		other: '#4B5563',
		other_hospital: '#6B7280'
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
		other: 'Other Clinic',
		other_hospital: 'Other Hospital'
	};

	// Shape types per category
	type Shape = 'triangle' | 'cross' | 'diamond' | 'square' | 'circle';
	const SHAPES: Record<string, Shape> = {
		imss_clinic: 'triangle',
		imss_hospital: 'cross',
		imss_bienestar: 'diamond',
		imss_bienestar_hospital: 'cross',
		ssa: 'square',
		issste: 'square',
		private: 'circle',
		private_hospital: 'cross',
		other: 'circle',
		other_hospital: 'cross'
	};

	// Returns an SVG path `d` attribute centered at (0,0) for each shape
	function shapePath(shape: Shape, isHospital: boolean): string {
		const s = isHospital ? 7 : 4.5;
		switch (shape) {
			case 'triangle': {
				// Upward-pointing triangle
				const h = s * 1.73;
				return `M 0,${-h * 0.67} L ${s},${h * 0.33} L ${-s},${h * 0.33} Z`;
			}
			case 'cross': {
				// Plus/cross — large for hospitals
				const t = isHospital ? 2.8 : 2;
				return `M ${-t},${-s} L ${t},${-s} L ${t},${-t} L ${s},${-t} L ${s},${t} L ${t},${t} L ${t},${s} L ${-t},${s} L ${-t},${t} L ${-s},${t} L ${-s},${-t} L ${-t},${-t} Z`;
			}
			case 'diamond': {
				return `M 0,${-s} L ${s},0 L 0,${s} L ${-s},0 Z`;
			}
			case 'square': {
				const h = s * 0.88;
				return `M ${-h},${-h} L ${h},${-h} L ${h},${h} L ${-h},${h} Z`;
			}
			case 'circle':
			default: {
				// Approximate circle with a polygon for SVG path
				const pts = 12;
				const arr = [];
				for (let i = 0; i < pts; i++) {
					const a = (i / pts) * Math.PI * 2 - Math.PI / 2;
					arr.push(
						`${i === 0 ? 'M' : 'L'} ${(s * Math.cos(a)).toFixed(2)},${(s * Math.sin(a)).toFixed(2)}`
					);
				}
				return arr.join(' ') + ' Z';
			}
		}
	}

	// Opacity by step
	function opacity(cat: string, id: StepId): number {
		if (id === 'all') return 0.88;
		if (id === 'clinics') return cat === 'imss_clinic' ? 0.95 : 0.05;
		if (id === 'drain') return cat === 'imss_hospital' ? 0.95 : cat === 'imss_clinic' ? 0.1 : 0.04;
		if (id === 'hospitals') return cat === 'imss_hospital' ? 0.95 : 0.05;
		if (id === 'bienestar')
			return cat === 'imss_bienestar' || cat === 'imss_bienestar_hospital' ? 0.92 : 0.05;
		if (id === 'bienestar_hospitals_only') return cat === 'imss_bienestar_hospital' ? 0.95 : 0.05;
		if (id === 'bienestar_hospitals') {
			if (cat === 'imss_hospital' || cat === 'imss_bienestar_hospital') return 0.95;
			if (cat === 'imss_bienestar') return 0.1;
			return 0.04;
		}
		if (id === 'explore') return filterState[cat] !== false ? 0.88 : 0;
		return 0.05;
	}

	function legendFor(id: StepId): string[] {
		if (id === 'all')
			return [
				'imss_clinic',
				'imss_hospital',
				'imss_bienestar',
				'imss_bienestar_hospital',
				'ssa',
				'private_hospital'
			];
		if (id === 'clinics') return ['imss_clinic'];
		if (id === 'drain') return ['imss_hospital', 'imss_clinic'];
		if (id === 'hospitals') return ['imss_hospital'];
		if (id === 'bienestar') return ['imss_bienestar', 'imss_bienestar_hospital'];
		if (id === 'bienestar_hospitals_only') return ['imss_bienestar_hospital'];
		if (id === 'bienestar_hospitals') return ['imss_hospital', 'imss_bienestar_hospital'];
		if (id === 'explore')
			return [
				'imss_clinic',
				'imss_hospital',
				'imss_bienestar',
				'imss_bienestar_hospital',
				'ssa',
				'private',
				'private_hospital'
			];
		return [];
	}

	// Facility count breakdown for step 1 table
	const TABLE_ROWS = [
		{ cat: 'imss_clinic', label: 'IMSS Clinic (UMF)' },
		{ cat: 'imss_hospital', label: 'IMSS Hospital' },
		{ cat: 'imss_bienestar', label: 'IMSS-Bienestar Clinic' },
		{ cat: 'imss_bienestar_hospital', label: 'IMSS-Bienestar Hospital' },
		{ cat: 'ssa', label: 'State Health (SSA)' },
		{ cat: 'private', label: 'Private Clinic' },
		{ cat: 'private_hospital', label: 'Private Hospital' },
		{ cat: 'other', label: 'Other' }
	];

	// ── State ─────────────────────────────────────────────────────────────────
	let facilities: Facility[] = [];
	let isLoading = true;
	let mapReady = false;
	let boundaryPaths: string[] = [];
	let projectedFacs: (Facility & { x: number; y: number })[] = [];
	let cityPts: { name: string; x: number; y: number; capital: boolean }[] = [];
	let valleVerde: [number, number] = [0, 0];

	const CITY_DEFS = [
		{ name: 'Tepic', lat: 21.5, lng: -104.89, capital: true },
		{ name: 'Acaponeta', lat: 22.5, lng: -105.37, capital: false },
		{ name: 'Ixtlán del Río', lat: 21.03, lng: -104.36, capital: false },
		{ name: 'Bahía de Banderas', lat: 20.75, lng: -105.25, capital: false },
		{ name: 'Santiago Ixcuintla', lat: 21.81, lng: -105.22, capital: false }
	];

	const W = 900,
		H = 620;

	let stepIdx = 0;
	$: activeStep = STEPS[stepIdx] ?? STEPS[0];
	$: activeLegend = legendFor(activeStep.id);

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

	let tooltip: { x: number; y: number; f: Facility } | null = null;
	let sectionEl: HTMLElement | null = null;
	let stepEls: (HTMLElement | null)[] = [];
	let released = false,
		prevCarryOut = false,
		rafPending = false,
		nearViewport = false;
	let observer: IntersectionObserver | null = null;

	const TRIGGER = 0.4;
	const CARRY = 0.15; // release only when last step nearly scrolled off top

	function onScroll() {
		if (!browser || rafPending) return;
		rafPending = true;
		requestAnimationFrame(() => {
			rafPending = false;
			if (!stepEls.length) return;
			const vh = window.innerHeight;
			let found = 0;
			for (let i = 0; i < stepEls.length; i++) {
				const el = stepEls[i];
				if (!el) continue;
				if (el.getBoundingClientRect().top <= vh * TRIGGER) found = i;
			}
			stepIdx = found;
			const lastEl = stepEls[stepEls.length - 1];
			if (lastEl) {
				const nowCarry = lastEl.getBoundingClientRect().top <= vh * CARRY;
				if (nowCarry && !prevCarryOut) released = true;
				if (!nowCarry && prevCarryOut) released = false;
				prevCarryOut = nowCarry;
			}
		});
	}

	function loadD3(): Promise<void> {
		if ((window as any).d3) return Promise.resolve();
		return new Promise((res, rej) => {
			const s = document.createElement('script');
			s.src = 'https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js';
			s.onload = () => res();
			s.onerror = rej;
			document.head.appendChild(s);
		});
	}

	async function buildProjection(d3: any, geo: any) {
		// Collect extents via loop — spread of 4000+ items blows the JS stack
		let minLng = Infinity,
			maxLng = -Infinity,
			minLat = Infinity,
			maxLat = -Infinity;
		const features = geo.type === 'FeatureCollection' ? geo.features : [geo];
		for (const f of features) {
			const geom = f.geometry;
			const polys = geom.type === 'Polygon' ? [geom.coordinates] : geom.coordinates;
			for (const poly of polys)
				for (const ring of poly)
					for (const pt of ring) {
						if (pt[0] < minLng) minLng = pt[0];
						if (pt[0] > maxLng) maxLng = pt[0];
						if (pt[1] < minLat) minLat = pt[1];
						if (pt[1] > maxLat) maxLat = pt[1];
					}
		}
		const cLng = (minLng + maxLng) / 2,
			cLat = (minLat + maxLat) / 2;

		// Compute scale from reference projection (avoids fitSize crash on tiny polygons)
		const REF = 150;
		const refProj = d3
			.geoMercator()
			.center([cLng, cLat])
			.scale(REF)
			.translate([W / 2, H / 2]);
		const [x0] = refProj([minLng, cLat]) ?? [0];
		const [x1] = refProj([maxLng, cLat]) ?? [0];
		const [, y0] = refProj([cLng, minLat]) ?? [0, 0];
		const [, y1] = refProj([cLng, maxLat]) ?? [0, 0];
		const scale = REF * 0.88 * Math.min(W / Math.abs(x1 - x0), H / Math.abs(y1 - y0));

		const proj = d3
			.geoMercator()
			.center([cLng, cLat])
			.scale(scale)
			.translate([W / 2, H / 2]);
		const path = d3.geoPath().projection(proj);

		boundaryPaths = features
			.map((f: any) => {
				try {
					return path(f) ?? '';
				} catch {
					return '';
				}
			})
			.filter(Boolean);

		projectedFacs = facilities.map((f) => {
			const [x, y] = proj([f.lng, f.lat]) ?? [0, 0];
			return { ...f, x: +x.toFixed(1), y: +y.toFixed(1) };
		});

		cityPts = CITY_DEFS.map((c) => {
			const [x, y] = proj([c.lng, c.lat]) ?? [0, 0];
			return { name: c.name, x: Math.round(x), y: Math.round(y), capital: c.capital };
		});

		const [vx, vy] = proj([-104.485, 21.054]) ?? [0, 0];
		valleVerde = [Math.round(vx), Math.round(vy)];

		mapReady = true;
	}

	onMount(async () => {
		if (!browser) return;
		window.addEventListener('scroll', onScroll, { passive: true });
		window.addEventListener('resize', onScroll, { passive: true });

		observer = new IntersectionObserver(
			async ([entry]) => {
				if (!entry.isIntersecting) return;
				observer?.disconnect();
				observer = null;
				nearViewport = true;
				try {
					const [, facRes, geoRes] = await Promise.all([
						loadD3(),
						fetch(`${base}/data/nayarit_medical_centers.json`),
						fetch(`${base}/data/nayarit_municipalities.geojson`)
					]);
					facilities = await facRes.json();
					const geo = await geoRes.json();
					await buildProjection((window as any).d3, geo);
				} catch (e) {
					console.error('[ScrollyMap] load error:', e);
				} finally {
					isLoading = false;
				}
				await tick();
				onScroll();
			},
			{ rootMargin: '400px 0px' }
		);

		if (sectionEl) observer.observe(sectionEl);
	});

	onDestroy(() => {
		if (!browser) return;
		window.removeEventListener('scroll', onScroll);
		window.removeEventListener('resize', onScroll);
		observer?.disconnect();
	});

	$: counts = (() => {
		const c: Record<string, number> = {};
		for (const f of facilities) c[f.category] = (c[f.category] ?? 0) + 1;
		return c;
	})();

	$: shown = (() => {
		const id = activeStep.id;
		if (id === 'all') return facilities.length;
		if (id === 'clinics') return counts['imss_clinic'] ?? 0;
		if (id === 'drain') return counts['imss_hospital'] ?? 0;
		if (id === 'hospitals') return counts['imss_hospital'] ?? 0;
		if (id === 'bienestar')
			return (counts['imss_bienestar'] ?? 0) + (counts['imss_bienestar_hospital'] ?? 0);
		if (id === 'bienestar_hospitals_only') return counts['imss_bienestar_hospital'] ?? 0;
		if (id === 'bienestar_hospitals')
			return (counts['imss_hospital'] ?? 0) + (counts['imss_bienestar_hospital'] ?? 0);
		if (id === 'explore')
			return Object.entries(filterState)
				.filter(([, v]) => v)
				.reduce((a, [k]) => a + (counts[k] ?? 0), 0);
		return 0;
	})();
</script>

<!-- ═══════════════════════════════════════════════════════════════ -->
<div class="sm-wrap" bind:this={sectionEl}>
	<!-- Sticky map -->
	<div class="sm-sticky {released ? 'sm-released' : ''}">
		{#if !nearViewport || isLoading || !mapReady}
			<div class="sm-skeleton">
				<div class="sm-spinner"></div>
				<p class="sm-skeleton-label">Loading Nayarit healthcare map…</p>
			</div>
		{:else}
			<svg
				viewBox="0 0 {W} {H}"
				preserveAspectRatio="xMidYMid meet"
				class="sm-svg"
				role="img"
				aria-label="Map of Nayarit public healthcare facilities"
			>
				<rect width={W} height={H} fill="#0d0520" />

				<!-- Municipality fills -->
				{#each boundaryPaths as d}
					<path {d} fill="rgba(42,14,88,0.32)" stroke="none" />
				{/each}
				<!-- Internal municipality lines — very transparent -->
				{#each boundaryPaths as d}
					<path {d} fill="none" stroke="rgba(155,111,204,0.18)" stroke-width="0.6" />
				{/each}
				<!-- Outer state border — crisp -->
				{#each boundaryPaths as d}
					<path {d} fill="none" stroke="rgba(155,111,204,0.75)" stroke-width="1.4" />
				{/each}

				<!-- City markers -->
				{#each cityPts as c}
					<circle
						cx={c.x}
						cy={c.y}
						r={c.capital ? 3.5 : 2}
						fill={c.capital ? '#FBBF24' : 'rgba(251,191,36,0.4)'}
						opacity="0.8"
					/>
					<text
						x={c.x + 6}
						y={c.y + 4}
						fill={c.capital ? 'rgba(251,191,36,0.8)' : 'rgba(251,191,36,0.4)'}
						font-family="'Syne', sans-serif"
						font-size={c.capital ? 8.5 : 7}
						font-weight={c.capital ? '700' : '400'}
						letter-spacing="0.04em">{c.name}</text
					>
				{/each}

				<!-- Dim symbols -->
				{#each projectedFacs as f (f.clues)}
					{@const op = opacity(f.category, activeStep.id)}
					{#if op > 0 && op < 0.5}
						<path
							d={shapePath(SHAPES[f.category] ?? 'circle', f.category.includes('hospital'))}
							transform="translate({f.x},{f.y})"
							fill={COLORS[f.category] ?? '#555'}
							opacity={op}
						/>
					{/if}
				{/each}

				<!-- Highlight symbols -->
				{#each projectedFacs as f (f.clues)}
					{@const op = opacity(f.category, activeStep.id)}
					{#if op >= 0.5}
						<path
							d={shapePath(SHAPES[f.category] ?? 'circle', f.category.includes('hospital'))}
							transform="translate({f.x},{f.y})"
							fill={COLORS[f.category] ?? '#555'}
							opacity={op}
							class="sm-dot"
							role="img"
							aria-label="{f.name}, {f.municipality}"
							on:mouseenter={() => (tooltip = { x: f.x, y: f.y, f })}
							on:mouseleave={() => (tooltip = null)}
						/>
					{/if}
				{/each}

				<!-- Valle Verde story pin — label only, NO circle (would look like a facility) -->
				{#if stepIdx > 0}
					<g class="sm-pin">
						<!-- Small crosshair instead of circle -->
						<line
							x1={valleVerde[0] - 8}
							y1={valleVerde[1]}
							x2={valleVerde[0] + 8}
							y2={valleVerde[1]}
							stroke="#FBBF24"
							stroke-width="1.5"
							opacity="0.8"
						/>
						<line
							x1={valleVerde[0]}
							y1={valleVerde[1] - 8}
							x2={valleVerde[0]}
							y2={valleVerde[1] + 8}
							stroke="#FBBF24"
							stroke-width="1.5"
							opacity="0.8"
						/>
						<circle cx={valleVerde[0]} cy={valleVerde[1]} r="3" fill="#FBBF24" opacity="0.95" />
						<text
							x={valleVerde[0] + 11}
							y={valleVerde[1] - 3}
							fill="#FBBF24"
							font-family="'Crimson Text', Georgia, serif"
							font-style="italic"
							font-size="11"
							opacity="0.95">Valle Verde</text
						>
						<text
							x={valleVerde[0] + 11}
							y={valleVerde[1] + 8}
							fill="rgba(251,191,36,0.55)"
							font-family="'Syne', sans-serif"
							font-size="7.5"
							letter-spacing="0.1em">AHUACATLÁN</text
						>
					</g>
				{/if}

				<!-- Tooltip -->
				{#if tooltip}
					{@const tx = tooltip.x > W - 200 ? tooltip.x - 196 : tooltip.x + 12}
					{@const ty = tooltip.y > H - 80 ? tooltip.y - 78 : tooltip.y + 8}
					<rect
						x={tx - 2}
						y={ty - 2}
						width="194"
						height="62"
						rx="3"
						fill="rgba(8,3,18,0.96)"
						stroke={COLORS[tooltip.f.category] ?? '#FBBF24'}
						stroke-width="1"
					/>
					<text
						x={tx + 8}
						y={ty + 15}
						fill="#FBBF24"
						font-family="'Syne', sans-serif"
						font-size="8.5"
						font-weight="700"
						>{tooltip.f.name.length > 27 ? tooltip.f.name.slice(0, 26) + '…' : tooltip.f.name}</text
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
					<path
						d={shapePath(
							SHAPES[tooltip.f.category] ?? 'circle',
							tooltip.f.category.includes('hospital')
						)}
						transform="translate({tx + 12},{ty + 55})"
						fill={COLORS[tooltip.f.category] ?? '#FBBF24'}
					/>
					<text
						x={tx + 22}
						y={ty + 59}
						fill="rgba(255,255,255,0.4)"
						font-family="'Syne', sans-serif"
						font-size="7">{LABELS[tooltip.f.category] ?? ''}</text
					>
				{/if}
			</svg>

			<!-- Legend — top right -->
			<div class="sm-legend">
				<p class="sm-legend-hed">Legend</p>
				{#each activeLegend as cat}
					<div class="sm-legend-row">
						<!-- Shape icon in legend using inline SVG -->
						<svg width="18" height="18" viewBox="-9 -9 18 18" class="sm-legend-icon">
							<path
								d={shapePath(SHAPES[cat] ?? 'circle', cat.includes('hospital'))}
								fill={COLORS[cat]}
								opacity="0.9"
							/>
						</svg>
						<span class="sm-legend-label">{LABELS[cat]}</span>
					</div>
				{/each}
				<div class="sm-legend-count">
					<span class="sm-count-n">{shown}</span>
					<span class="sm-count-l">shown</span>
				</div>
			</div>
		{/if}
	</div>

	<!-- Steps -->
	<div class="sm-steps">
		{#each STEPS as step, i}
			<div class="sm-step" bind:this={stepEls[i]}>
				{#if step.id !== 'explore'}
					<div class="sm-card" class:sm-card--active={stepIdx === i}>
						<div class="sm-step-num">{String(i + 1).padStart(2, '0')}</div>
						<h3 class="sm-hed">{step.headline}</h3>
						<p class="sm-body">{step.body}</p>

						<!-- Step 1: facility breakdown table -->
						{#if step.id === 'all' && mapReady}
							<table class="sm-table">
								<thead>
									<tr>
										<th></th>
										<th>Type</th>
										<th class="sm-table-n">Count</th>
									</tr>
								</thead>
								<tbody>
									{#each TABLE_ROWS as row}
										{#if (counts[row.cat] ?? 0) > 0}
											<tr>
												<td>
													<svg width="14" height="14" viewBox="-7 -7 14 14">
														<path
															d={shapePath(
																SHAPES[row.cat] ?? 'circle',
																row.cat.includes('hospital')
															)}
															fill={COLORS[row.cat]}
															opacity="0.9"
														/>
													</svg>
												</td>
												<td class="sm-table-label">{row.label}</td>
												<td class="sm-table-n">{counts[row.cat] ?? 0}</td>
											</tr>
										{/if}
									{/each}
									<tr class="sm-table-total">
										<td></td>
										<td class="sm-table-label">Total</td>
										<td class="sm-table-n">{facilities.length}</td>
									</tr>
								</tbody>
							</table>
						{/if}

						{#if step.stats}
							<div class="sm-stats">
								{#each step.stats as s}
									<div class="sm-stat">
										<span class="sm-stat-n">{s.n}</span>
										<span class="sm-stat-l">{s.label}</span>
									</div>
								{/each}
							</div>
						{/if}

						{#if step.source}
							<p class="sm-source">
								{#if step.sourceUrl}
									<a href={step.sourceUrl} target="_blank" rel="noopener noreferrer"
										>{step.source} ↗</a
									>
								{:else}{step.source}{/if}
							</p>
						{/if}
					</div>
				{:else}
					<!-- Explore / filter step -->
					<div class="sm-card sm-card--wide" class:sm-card--active={stepIdx === i}>
						<div class="sm-step-num">{String(i + 1).padStart(2, '0')}</div>
						<h3 class="sm-hed">{step.headline}</h3>
						<p class="sm-body">{step.body}</p>

						<p class="sm-filter-hed">Filter by type</p>
						<div class="sm-filters">
							{#each Object.entries(LABELS) as [cat, label]}
								<label class="sm-filter" class:sm-filter--on={filterState[cat] !== false}>
									<input type="checkbox" bind:checked={filterState[cat]} />
									<svg width="14" height="14" viewBox="-7 -7 14 14" class="sm-filter-icon">
										<path
											d={shapePath(SHAPES[cat] ?? 'circle', cat.includes('hospital'))}
											fill={COLORS[cat]}
											opacity="0.9"
										/>
									</svg>
									<span class="sm-filter-name">{label}</span>
									<span class="sm-filter-n">{counts[cat] ?? 0}</span>
								</label>
							{/each}
						</div>

						<div class="sm-sources">
							<p class="sm-sources-hed">Data Sources</p>
							<ul class="sm-sources-list">
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
										rel="noopener noreferrer">IMSS Directorio — Clínicas ↗</a
									>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=Hospital"
										target="_blank"
										rel="noopener noreferrer">IMSS Directorio — Hospitales ↗</a
									>
								</li>
								<li>
									<a
										href="https://www.imss.gob.mx/directorio?dom_estado=Nayarit&tipo_de_servicio=IMSS%20Bienestar"
										target="_blank"
										rel="noopener noreferrer">IMSS-Bienestar Directorio ↗</a
									>
								</li>
								<li>INEGI Marco Geoestadístico 2023 — Shapefile 18mun</li>
							</ul>
						</div>
					</div>
				{/if}
			</div>
		{/each}
	</div>
</div>

<style>
	.sm-wrap {
		position: relative;
		width: 100vw;
		left: 50%;
		margin-left: -50vw;
		box-sizing: border-box;
	}

	.sm-sticky {
		position: sticky;
		top: 0;
		width: 100vw;
		height: 100vh;
		overflow: hidden;
		background: #0d0520;
		z-index: 0;
		margin-bottom: 2rem;
	}
	.sm-released {
		position: relative;
	}
	.sm-svg {
		width: 100%;
		height: 100%;
		display: block;
	}

	.sm-skeleton {
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 1.25rem;
	}
	.sm-spinner {
		width: 48px;
		height: 48px;
		border-radius: 50%;
		border: 2px solid rgba(123, 79, 166, 0.25);
		border-top-color: #fbbf24;
		animation: sm-spin 0.9s linear infinite;
	}
	@keyframes sm-spin {
		to {
			transform: rotate(360deg);
		}
	}
	.sm-skeleton-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.2);
		margin: 0;
	}

	.sm-dot {
		cursor: pointer;
		transition: opacity 0.3s ease;
	}
	.sm-dot:hover {
		opacity: 1 !important;
		filter: brightness(1.4);
	}
	.sm-pin {
		animation: sm-pulse 2.6s ease-in-out infinite;
	}
	@keyframes sm-pulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.55;
		}
	}

	/* Legend */
	.sm-legend {
		position: absolute;
		top: 1.25rem;
		right: 1.5rem;
		background: rgba(8, 3, 18, 0.85);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		border: 1px solid rgba(155, 111, 204, 0.3);
		border-radius: 6px;
		padding: 0.9rem 1.1rem;
		min-width: 190px;
		pointer-events: none;
	}
	.sm-legend-hed {
		font-family: 'Syne', sans-serif;
		font-size: 0.55rem;
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.35);
		margin: 0 0 0.65rem;
		font-weight: 700;
	}
	.sm-legend-row {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-bottom: 0.38rem;
	}
	.sm-legend-icon {
		flex-shrink: 0;
		overflow: visible;
	}
	.sm-legend-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.63rem;
		letter-spacing: 0.04em;
		color: rgba(255, 255, 255, 0.72);
	}
	.sm-legend-count {
		display: flex;
		align-items: baseline;
		gap: 0.35rem;
		margin-top: 0.65rem;
		border-top: 1px solid rgba(255, 255, 255, 0.08);
		padding-top: 0.55rem;
	}
	.sm-count-n {
		font-family: 'Playfair Display', Georgia, serif;
		font-size: 1.6rem;
		font-weight: 900;
		color: rgba(251, 191, 36, 0.8);
		line-height: 1;
	}
	.sm-count-l {
		font-family: 'Syne', sans-serif;
		font-size: 0.55rem;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.25);
	}

	/* Steps */
	.sm-steps {
		position: relative;
		z-index: 1;
		pointer-events: none;
		margin-top: -100vh;
	}
	.sm-step {
		min-height: 100vh;
		display: flex;
		align-items: center;
		padding: 2rem 3rem;
		pointer-events: none;
	}
	.sm-step:last-child {
		justify-content: flex-end;
		min-height: 120vh;
		padding-top: 10vh;
		align-items: flex-start;
	}

	/* Card */
	.sm-card {
		background: rgba(10, 4, 20, 0.88);
		backdrop-filter: blur(16px);
		-webkit-backdrop-filter: blur(16px);
		border: 1px solid rgba(155, 111, 204, 0.18);
		border-radius: 6px;
		padding: 1.6rem;
		max-width: 360px;
		pointer-events: all;
		box-shadow: 0 16px 48px rgba(0, 0, 0, 0.65);
		transition: border-color 0.3s;
	}
	.sm-card--wide {
		max-width: 420px;
	}
	.sm-card--active {
		border-color: rgba(251, 191, 36, 0.3);
	}
	.sm-step-num {
		font-family: 'Syne', sans-serif;
		font-size: 0.55rem;
		letter-spacing: 0.28em;
		color: #fbbf24;
		font-weight: 700;
		margin-bottom: 0.45rem;
		text-transform: uppercase;
	}
	.sm-hed {
		font-family: 'Playfair Display', Georgia, serif;
		font-size: clamp(1rem, 2vw, 1.3rem);
		font-weight: 700;
		color: #f0e8d0;
		line-height: 1.25;
		margin: 0 0 0.7rem;
	}
	.sm-body {
		font-family: 'Source Serif 4', Georgia, serif;
		font-size: 0.875rem;
		line-height: 1.75;
		color: rgba(255, 255, 255, 0.68);
		margin: 0 0 0.85rem;
	}

	/* Facility table */
	.sm-table {
		width: 100%;
		border-collapse: collapse;
		margin: 0 0 0.85rem;
	}
	.sm-table thead tr {
		border-bottom: 1px solid rgba(255, 255, 255, 0.1);
	}
	.sm-table th {
		font-family: 'Syne', sans-serif;
		font-size: 0.52rem;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.3);
		padding: 0 0 0.35rem;
		text-align: left;
		font-weight: 600;
	}
	.sm-table td {
		padding: 0.28rem 0.15rem;
		vertical-align: middle;
	}
	.sm-table-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.65rem;
		color: rgba(255, 255, 255, 0.65);
	}
	.sm-table-n {
		text-align: right;
		font-family: 'Syne', sans-serif;
		font-size: 0.68rem;
		font-weight: 700;
		color: rgba(255, 255, 255, 0.55);
		padding-right: 0.25rem;
	}
	.sm-table-total td {
		border-top: 1px solid rgba(255, 255, 255, 0.1);
		padding-top: 0.4rem;
	}
	.sm-table-total .sm-table-label {
		color: rgba(255, 255, 255, 0.8);
		font-weight: 700;
	}
	.sm-table-total .sm-table-n {
		color: #fbbf24;
	}

	/* Stats */
	.sm-stats {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(75px, 1fr));
		gap: 0.5rem;
		border-top: 1px solid rgba(251, 191, 36, 0.18);
		padding-top: 0.7rem;
		margin-bottom: 0.7rem;
	}
	.sm-stat {
		display: flex;
		flex-direction: column;
		gap: 0.1rem;
	}
	.sm-stat-n {
		font-family: 'Playfair Display', serif;
		font-size: 1.35rem;
		font-weight: 900;
		color: #fbbf24;
		line-height: 1;
	}
	.sm-stat-l {
		font-family: 'Syne', sans-serif;
		font-size: 0.51rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.32);
		line-height: 1.3;
	}

	/* Source */
	.sm-source {
		font-family: 'Syne', sans-serif;
		font-size: 0.54rem;
		letter-spacing: 0.06em;
		color: rgba(255, 255, 255, 0.2);
		border-left: 2px solid rgba(251, 191, 36, 0.2);
		padding-left: 0.5rem;
		margin: 0;
		line-height: 1.5;
	}
	.sm-source a {
		color: rgba(251, 191, 36, 0.45);
		text-decoration: none;
	}
	.sm-source a:hover {
		color: #fbbf24;
		text-decoration: underline;
	}

	/* Filter */
	.sm-filter-hed {
		font-family: 'Syne', sans-serif;
		font-size: 0.54rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.3);
		margin: 0.7rem 0 0.4rem;
	}
	.sm-filters {
		display: flex;
		flex-direction: column;
		gap: 0.18rem;
		margin-bottom: 1rem;
	}
	.sm-filter {
		display: flex;
		align-items: center;
		gap: 0.45rem;
		cursor: pointer;
		padding: 0.22rem 0.35rem;
		border-radius: 3px;
		opacity: 0.3;
		transition:
			opacity 0.15s,
			background 0.15s;
	}
	.sm-filter--on {
		opacity: 1;
	}
	.sm-filter:hover {
		background: rgba(255, 255, 255, 0.05);
		opacity: 1;
	}
	.sm-filter input {
		display: none;
	}
	.sm-filter-icon {
		flex-shrink: 0;
		overflow: visible;
	}
	.sm-filter-name {
		flex: 1;
		font-family: 'Syne', sans-serif;
		font-size: 0.64rem;
		letter-spacing: 0.04em;
		color: rgba(255, 255, 255, 0.75);
	}
	.sm-filter-n {
		font-family: 'Syne', sans-serif;
		font-size: 0.58rem;
		color: rgba(255, 255, 255, 0.25);
		min-width: 24px;
		text-align: right;
	}

	/* Sources */
	.sm-sources {
		border-top: 1px solid rgba(255, 255, 255, 0.06);
		padding-top: 0.85rem;
	}
	.sm-sources-hed {
		font-family: 'Syne', sans-serif;
		font-size: 0.54rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: rgba(251, 191, 36, 0.4);
		margin: 0 0 0.4rem;
	}
	.sm-sources-list {
		list-style: none;
		padding: 0;
		margin: 0;
		display: flex;
		flex-direction: column;
		gap: 0.28rem;
	}
	.sm-sources-list li {
		font-family: 'Syne', sans-serif;
		font-size: 0.57rem;
		color: rgba(255, 255, 255, 0.2);
		line-height: 1.5;
	}
	.sm-sources-list a {
		color: rgba(251, 191, 36, 0.4);
		text-decoration: none;
	}
	.sm-sources-list a:hover {
		color: #fbbf24;
		text-decoration: underline;
	}

	@media (max-width: 767px) {
		.sm-step {
			align-items: flex-end;
			padding: 1.25rem 1rem 2rem;
		}
		.sm-step:last-child {
			justify-content: flex-start;
		}
		.sm-card,
		.sm-card--wide {
			max-width: 100%;
		}
		.sm-legend {
			top: 0.75rem;
			right: 0.75rem;
			min-width: 150px;
		}
	}
</style>
