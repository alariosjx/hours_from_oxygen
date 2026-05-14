<script context="module" lang="ts">
	// DocRenderer invokes as [[LocatorMap]] — no props.
</script>

<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import { base } from '$app/paths';

	// ── State ─────────────────────────────────────────────────────────────────
	let ready = false;
	let panel: 'mexico' | 'nayarit' = 'mexico';

	// SVG canvas dimensions
	const W = 260,
		H = 300;

	// Rendered paths
	let mexicoPaths: { d: string; isNayarit: boolean }[] = [];
	let nayaritPaths: string[] = [];
	let nayaritCities: { name: string; x: number; y: number; capital: boolean; story?: boolean }[] =
		[];
	let nayaritBox: [number, number, number, number] = [0, 0, W, H]; // bbox in Mexico SVG for zoom rect

	// Sticky behavior
	let wrapEl: HTMLElement | null = null;
	let visible = false;
	let observer: IntersectionObserver | null = null;

	// Mexico national outline — 170 coords from Natural Earth 110m (world-atlas@2, id:484)
	// Embedded directly so no external fetch is needed for the Mexico panel
	const MEXICO_COORDS: [number, number][] = [
		[-117.13, 32.53],
		[-115.99, 32.61],
		[-114.72, 32.72],
		[-114.81, 32.52],
		[-113.31, 32.04],
		[-111.02, 31.33],
		[-109.04, 31.34],
		[-108.24, 31.34],
		[-108.24, 31.75],
		[-106.51, 31.75],
		[-106.14, 31.4],
		[-105.63, 31.08],
		[-105.04, 30.64],
		[-104.71, 30.12],
		[-104.46, 29.57],
		[-103.94, 29.27],
		[-103.11, 28.97],
		[-102.48, 29.76],
		[-101.66, 29.78],
		[-100.96, 29.38],
		[-100.46, 28.7],
		[-100.11, 28.11],
		[-99.52, 27.54],
		[-99.3, 26.84],
		[-99.02, 26.37],
		[-98.24, 26.06],
		[-97.53, 25.84],
		[-97.14, 25.87],
		[-97.53, 24.99],
		[-97.7, 24.27],
		[-97.78, 22.93],
		[-97.87, 22.44],
		[-97.7, 21.9],
		[-97.39, 21.41],
		[-97.19, 20.64],
		[-96.53, 19.89],
		[-96.29, 19.32],
		[-95.9, 18.83],
		[-94.84, 18.56],
		[-94.43, 18.14],
		[-93.55, 18.42],
		[-92.79, 18.53],
		[-92.04, 18.71],
		[-91.41, 18.88],
		[-90.77, 19.28],
		[-90.53, 19.87],
		[-90.45, 20.71],
		[-90.28, 21],
		[-89.6, 21.26],
		[-88.54, 21.49],
		[-87.66, 21.46],
		[-87.05, 21.54],
		[-86.81, 21.33],
		[-86.85, 20.85],
		[-87.38, 20.26],
		[-87.62, 19.65],
		[-87.44, 19.47],
		[-87.59, 19.04],
		[-87.84, 18.26],
		[-88.09, 18.52],
		[-88.3, 18.5],
		[-88.49, 18.49],
		[-88.85, 17.88],
		[-89.03, 18],
		[-89.15, 17.96],
		[-89.14, 17.81],
		[-90.07, 17.82],
		[-91, 17.82],
		[-91, 17.25],
		[-91.45, 17.25],
		[-91.08, 16.92],
		[-90.71, 16.69],
		[-90.6, 16.47],
		[-90.44, 16.41],
		[-90.46, 16.07],
		[-91.75, 16.07],
		[-92.23, 15.25],
		[-92.09, 15.06],
		[-92.2, 14.83],
		[-92.23, 14.54],
		[-93.36, 15.62],
		[-93.88, 15.94],
		[-94.69, 16.2],
		[-95.25, 16.13],
		[-96.05, 15.75],
		[-96.56, 15.65],
		[-97.26, 15.92],
		[-98.01, 16.11],
		[-98.95, 16.57],
		[-99.7, 16.71],
		[-100.83, 17.17],
		[-101.67, 17.65],
		[-101.92, 17.92],
		[-102.48, 17.98],
		[-103.5, 18.29],
		[-103.92, 18.75],
		[-104.99, 19.32],
		[-105.49, 19.95],
		[-105.73, 20.43],
		[-105.4, 20.53],
		[-105.5, 20.82],
		[-105.27, 21.08],
		[-105.27, 21.42],
		[-105.6, 21.87],
		[-105.69, 22.27],
		[-106.03, 22.77],
		[-106.91, 23.77],
		[-107.92, 24.55],
		[-108.4, 25.17],
		[-109.26, 25.58],
		[-109.44, 25.83],
		[-109.29, 26.44],
		[-109.8, 26.68],
		[-110.39, 27.16],
		[-110.64, 27.86],
		[-111.18, 27.94],
		[-111.76, 28.47],
		[-112.23, 28.96],
		[-112.27, 29.27],
		[-112.81, 30.02],
		[-113.17, 30.79],
		[-113.15, 31.17],
		[-113.87, 31.57],
		[-114.21, 31.52],
		[-114.78, 31.8],
		[-114.94, 31.39],
		[-114.77, 30.91],
		[-114.67, 30.16],
		[-114.33, 29.75],
		[-113.59, 29.06],
		[-113.42, 28.83],
		[-113.27, 28.76],
		[-113.14, 28.41],
		[-112.96, 28.43],
		[-112.76, 27.78],
		[-112.46, 27.53],
		[-112.24, 27.17],
		[-111.62, 26.66],
		[-111.29, 25.73],
		[-110.99, 25.29],
		[-110.71, 24.83],
		[-110.66, 24.3],
		[-110.17, 24.27],
		[-109.77, 23.81],
		[-109.41, 23.36],
		[-109.43, 23.19],
		[-109.85, 22.82],
		[-110.03, 22.82],
		[-110.3, 23.43],
		[-110.95, 24],
		[-111.67, 24.49],
		[-112.18, 24.74],
		[-112.15, 25.47],
		[-112.3, 26.01],
		[-112.78, 26.32],
		[-113.46, 26.77],
		[-113.6, 26.64],
		[-113.85, 26.9],
		[-114.46, 27.14],
		[-115.06, 27.72],
		[-114.98, 27.8],
		[-114.57, 27.74],
		[-114.2, 28.12],
		[-114.16, 28.57],
		[-114.93, 29.28],
		[-115.52, 29.56],
		[-115.89, 30.18],
		[-116.26, 30.84],
		[-116.72, 31.64],
		[-117.13, 32.53]
	];

	const CITY_DEFS = [
		{ name: 'Tepic', lat: 21.5, lng: -104.89, capital: true },
		{ name: 'Acaponeta', lat: 22.5, lng: -105.37, capital: false },
		{ name: 'Bahía de\nBanderas', lat: 20.75, lng: -105.25, capital: false },
		{ name: 'Ixtlán del Río', lat: 21.03, lng: -104.36, capital: false },
		{ name: 'Valle Verde', lat: 21.054, lng: -104.485, capital: false, story: true }
	];

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

	// Scale computation that avoids fitSize (crashes on tiny polygons)
	function makeProjection(d3: any, geo: any, w: number, h: number, pad = 0.88) {
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
		const REF = 150;
		const rp = d3
			.geoMercator()
			.center([cLng, cLat])
			.scale(REF)
			.translate([w / 2, h / 2]);
		const [x0] = rp([minLng, cLat]) ?? [0];
		const [x1] = rp([maxLng, cLat]) ?? [0];
		const [, y0] = rp([cLng, minLat]) ?? [0, 0];
		const [, y1] = rp([cLng, maxLat]) ?? [0, 0];
		const scale = REF * pad * Math.min(w / Math.abs(x1 - x0), h / Math.abs(y1 - y0));
		return d3
			.geoMercator()
			.center([cLng, cLat])
			.scale(scale)
			.translate([w / 2, h / 2]);
	}

	async function build() {
		const d3 = (window as any).d3;

		// ── Nayarit municipalities ───────────────────────────────────────────────
		const nayGeo = await fetch(`${base}/data/nayarit_municipalities.geojson`).then((r) => r.json());
		const nayProj = makeProjection(d3, nayGeo, W, H);
		const nayPath = d3.geoPath().projection(nayProj);

		nayaritPaths = nayGeo.features
			.map((f: any) => {
				try {
					return nayPath(f) ?? '';
				} catch {
					return '';
				}
			})
			.filter(Boolean);

		nayaritCities = CITY_DEFS.map((c) => {
			const [x, y] = nayProj([c.lng, c.lat]) ?? [0, 0];
			return { ...c, x: Math.round(x), y: Math.round(y) };
		});

		// ── Mexico overview — project the embedded outline + Nayarit municipalities ──
		// Use the MEXICO_COORDS outline (Natural Earth 110m, embedded above) for the
		// country silhouette, and Nayarit municipalities for the highlighted region.
		const mxGeo = {
			type: 'Feature',
			geometry: { type: 'Polygon', coordinates: [MEXICO_COORDS] }
		};
		const mxProj = makeProjection(d3, { type: 'FeatureCollection', features: [mxGeo] }, W, H, 0.9);
		const mxPath = d3.geoPath().projection(mxProj);

		// Mexico country fill (single path)
		const mxOutline = mxPath(mxGeo);
		mexicoPaths = [{ d: mxOutline ?? '', isNayarit: false }];

		// Nayarit municipalities projected onto Mexico scale
		const nayOnMx = nayGeo.features
			.map((f: any) => {
				try {
					return { d: mxPath(f) ?? '', isNayarit: true };
				} catch {
					return { d: '', isNayarit: true };
				}
			})
			.filter((o) => o.d);
		mexicoPaths = [...mexicoPaths, ...nayOnMx];

		// Nayarit bounding box in Mexico SVG coords (for the zoom rectangle)
		const nayMinLng = -106.7,
			nayMaxLng = -103.7,
			nayMinLat = 20.6,
			nayMaxLat = 23.1;
		const [bx0, by1] = mxProj([nayMinLng, nayMinLat]) ?? [0, 0];
		const [bx1, by0] = mxProj([nayMaxLng, nayMaxLat]) ?? [0, 0];
		nayaritBox = [
			Math.min(bx0, bx1) - 2,
			Math.min(by0, by1) - 2,
			Math.abs(bx1 - bx0) + 4,
			Math.abs(by1 - by0) + 4
		];

		ready = true;
	}

	onMount(async () => {
		if (!browser) return;
		observer = new IntersectionObserver(
			async ([e]) => {
				if (!e.isIntersecting) return;
				observer?.disconnect();
				observer = null;
				visible = true;
				await loadD3();
				await build();
			},
			{ rootMargin: '200px 0px' }
		);
		if (wrapEl) observer.observe(wrapEl);
	});

	onDestroy(() => {
		observer?.disconnect();
	});
</script>

<div class="lm-wrap" bind:this={wrapEl}>
	<div class="lm-card">
		<!-- Tab switcher -->
		<div class="lm-tabs">
			<button
				class="lm-tab"
				class:lm-tab--active={panel === 'mexico'}
				on:click={() => (panel = 'mexico')}
			>
				Mexico
			</button>
			<button
				class="lm-tab"
				class:lm-tab--active={panel === 'nayarit'}
				on:click={() => (panel = 'nayarit')}
			>
				Nayarit
			</button>
		</div>

		<!-- Map area -->
		<div class="lm-map-area">
			{#if !ready}
				<div class="lm-skeleton">
					<div class="lm-spinner"></div>
				</div>
			{:else if panel === 'mexico'}
				<!-- Mexico overview: all states in grey, Nayarit in purple, zoom box -->
				<svg
					viewBox="0 0 {W} {H}"
					class="lm-svg"
					role="img"
					aria-label="Mexico map with Nayarit highlighted"
				>
					<!-- Grey context states -->
					{#each mexicoPaths.filter((p) => !p.isNayarit) as p}
						<path d={p.d} fill="#ddd3c8" stroke="#b8a898" stroke-width="0.4" />
					{/each}

					<!-- Nayarit highlighted -->
					{#each mexicoPaths.filter((p) => p.isNayarit) as p}
						<path d={p.d} fill="#b89de0" stroke="#8b5e00" stroke-width="0.8" />
					{/each}

					<!-- Zoom rectangle around Nayarit -->
					<rect
						x={nayaritBox[0]}
						y={nayaritBox[1]}
						width={nayaritBox[2]}
						height={nayaritBox[3]}
						fill="none"
						stroke="#c8960a"
						stroke-width="1.2"
						stroke-dasharray="3,2"
						rx="1"
					/>

					<!-- Nayarit label -->
					<text
						x={nayaritBox[0] + nayaritBox[2] + 3}
						y={nayaritBox[1] + nayaritBox[3] / 2 + 3}
						font-family="Georgia,serif"
						font-size="8"
						font-weight="bold"
						fill="#8b5e00">Nayarit</text
					>

					<!-- Pacific Ocean label -->
					<text
						x="18"
						y="210"
						font-family="Georgia,serif"
						font-size="7.5"
						font-style="italic"
						fill="#50328c"
						opacity="0.4"
						transform="rotate(-30 18 210)">Pacific Ocean</text
					>

					<text
						x="4"
						y={H - 4}
						font-family="Georgia,serif"
						font-size="6"
						fill="#2a1a0e"
						opacity="0.4">GADM 4.1 / Natural Earth</text
					>
				</svg>
			{:else}
				<!-- Nayarit detail: municipalities + cities -->
				<svg
					viewBox="0 0 {W} {H}"
					class="lm-svg"
					role="img"
					aria-label="Nayarit state with key locations"
				>
					<!-- Municipality fills -->
					{#each nayaritPaths as d}
						<path {d} fill="#d4c5f0" stroke="#8b5e00" stroke-width="0.35" stroke-opacity="0.45" />
					{/each}

					<!-- Outer state border — crisp gold line, drawn over fills -->
					{#each nayaritPaths as d}
						<path {d} fill="none" stroke="#8b5e00" stroke-width="1.3" stroke-linejoin="round" />
					{/each}

					<!-- City markers -->
					{#each nayaritCities as c}
						{#if c.story}
							<!-- Valle Verde — crosshair, no dot -->
							<line
								x1={c.x - 6}
								y1={c.y}
								x2={c.x + 6}
								y2={c.y}
								stroke="#c8960a"
								stroke-width="1.4"
							/>
							<line
								x1={c.x}
								y1={c.y - 6}
								x2={c.x}
								y2={c.y + 6}
								stroke="#c8960a"
								stroke-width="1.4"
							/>
							<circle cx={c.x} cy={c.y} r="2" fill="#c8960a" />
							<!-- Label — try to avoid overlaps -->
							<text
								x={c.x + 7}
								y={c.y - 2}
								font-family="Georgia,serif"
								font-style="italic"
								font-size="7.5"
								font-weight="bold"
								fill="#8b5e00">Valle Verde</text
							>
						{:else}
							<circle
								cx={c.x}
								cy={c.y}
								r={c.capital ? 3.5 : 2.2}
								fill={c.capital ? '#c8960a' : '#50328c'}
								opacity={c.capital ? 1 : 0.75}
							/>
							{#each c.name.split('\n') as line, li}
								<text
									x={c.x + (c.capital ? 5 : 4)}
									y={c.y + 3 + li * 8}
									font-family="Georgia,serif"
									font-size={c.capital ? 8.5 : 7}
									font-weight={c.capital ? 'bold' : 'normal'}
									fill={c.capital ? '#8b5e00' : '#2a1a0e'}
									opacity={c.capital ? 1 : 0.75}>{line}</text
								>
							{/each}
						{/if}
					{/each}

					<text
						x="4"
						y={H - 4}
						font-family="Georgia,serif"
						font-size="6"
						fill="#2a1a0e"
						opacity="0.4">INEGI Marco Geoestadístico 2023</text
					>
				</svg>
			{/if}
		</div>

		<!-- Caption -->
		<p class="lm-caption">
			{#if panel === 'mexico'}
				Nayarit is a Pacific coast state roughly 800 km northwest of Mexico City.
			{:else}
				Valle Verde sits in Ahuacatlán municipality, equidistant from Tepic and Ixtlán del Río.
			{/if}
		</p>
	</div>
</div>

<style>
	/* Sits inline in doc flow — no sticky/float tricks that break layout */
	.lm-wrap {
		/* Float right of the article column on wide screens */
		float: right;
		clear: right;
		margin: 0.25rem 0 1.5rem 2rem;
		/* Push into the right margin beyond the article column */
		margin-right: max(-200px, calc(50% - 530px));
		width: 260px;
		position: relative;
		z-index: 5;
	}

	.lm-card {
		background: #f5f0e8;
		border: 1px solid rgba(160, 112, 16, 0.28);
		border-radius: 5px;
		padding: 0.7rem;
		box-shadow: 0 3px 16px rgba(42, 26, 14, 0.09);
	}

	/* Tabs */
	.lm-tabs {
		display: flex;
		gap: 3px;
		margin-bottom: 0.6rem;
	}
	.lm-tab {
		flex: 1;
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		padding: 4px 6px;
		border-radius: 3px;
		border: 1px solid rgba(160, 112, 16, 0.25);
		background: transparent;
		color: rgba(42, 26, 14, 0.45);
		cursor: pointer;
		transition: all 0.15s;
	}
	.lm-tab--active {
		background: #2a0e58;
		border-color: #2a0e58;
		color: #f0c040;
	}
	.lm-tab:not(.lm-tab--active):hover {
		background: rgba(42, 14, 88, 0.07);
		color: rgba(42, 26, 14, 0.75);
	}

	/* Map area */
	.lm-map-area {
		position: relative;
		background: #ede7da;
		border-radius: 3px;
		overflow: hidden;
		min-height: 160px;
	}

	.lm-svg {
		width: 100%;
		height: auto;
		display: block;
	}

	/* Skeleton */
	.lm-skeleton {
		display: flex;
		align-items: center;
		justify-content: center;
		height: 200px;
	}
	.lm-spinner {
		width: 28px;
		height: 28px;
		border-radius: 50%;
		border: 2px solid rgba(80, 50, 140, 0.2);
		border-top-color: #c8960a;
		animation: lm-spin 0.8s linear infinite;
	}
	@keyframes lm-spin {
		to {
			transform: rotate(360deg);
		}
	}

	/* Caption */
	.lm-caption {
		font-family: 'Georgia', serif;
		font-size: 0.68rem;
		line-height: 1.5;
		color: rgba(42, 26, 14, 0.55);
		font-style: italic;
		margin: 0.5rem 0 0;
	}

	/* Responsive */
	@media (max-width: 1100px) {
		.lm-wrap {
			float: none;
			margin: 1.5rem auto;
			width: min(280px, 100%);
			margin-right: auto;
		}
	}

	@media (max-width: 600px) {
		.lm-wrap {
			display: none;
		}
	}
</style>
