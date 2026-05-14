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

		// ── Mexico overview using world-atlas countries topojson ─────────────────
		// Use a simple inline equirectangular for Mexico overview —
		// we only need Mexico's rough outline + Nayarit's location
		// Fetch the same Nayarit GeoJSON and project it onto a Mexico overview
		// using a wider bbox so all of Mexico is visible
		const mxBbox = { minLng: -118.5, maxLng: -86.5, minLat: 14.2, maxLat: 32.8 };
		const mxCLng = (mxBbox.minLng + mxBbox.maxLng) / 2;
		const mxCLat = (mxBbox.minLat + mxBbox.maxLat) / 2;

		// Build a Mexico-scale projection
		const REF = 150;
		const mxRefProj = d3
			.geoMercator()
			.center([mxCLng, mxCLat])
			.scale(REF)
			.translate([W / 2, H / 2]);
		const [mx0] = mxRefProj([mxBbox.minLng, mxCLat]) ?? [0];
		const [mx1] = mxRefProj([mxBbox.maxLng, mxCLat]) ?? [0];
		const [, my0] = mxRefProj([mxCLng, mxBbox.minLat]) ?? [0, 0];
		const [, my1] = mxRefProj([mxCLng, mxBbox.maxLat]) ?? [0, 0];
		const mxScale = REF * 0.88 * Math.min(W / Math.abs(mx1 - mx0), H / Math.abs(my1 - my0));
		const mxProj = d3
			.geoMercator()
			.center([mxCLng, mxCLat])
			.scale(mxScale)
			.translate([W / 2, H / 2]);
		const mxPath = d3.geoPath().projection(mxProj);

		// Project Nayarit on the Mexico map and get its bbox for the zoom rectangle
		mexicoPaths = nayGeo.features
			.map((f: any) => {
				try {
					return { d: mxPath(f) ?? '', isNayarit: true };
				} catch {
					return { d: '', isNayarit: true };
				}
			})
			.filter((o) => o.d);

		// Nayarit bounding box in Mexico map SVG coordinates
		const nayMinLng = -106.7,
			nayMaxLng = -103.7,
			nayMinLat = 20.6,
			nayMaxLat = 23.1;
		const [bx0, by1] = mxProj([nayMinLng, nayMinLat]) ?? [0, 0];
		const [bx1, by0] = mxProj([nayMaxLng, nayMaxLat]) ?? [0, 0];
		nayaritBox = [
			Math.min(bx0, bx1) - 3,
			Math.min(by0, by1) - 3,
			Math.abs(bx1 - bx0) + 6,
			Math.abs(by1 - by0) + 6
		];

		// Fetch the Mexico admin-1 boundaries for the grey context states
		// Use GADM-lite via a public CDN — falls back gracefully if unavailable
		try {
			const mxStates = await fetch(
				'https://raw.githubusercontent.com/nvkelso/natural-earth-vector/master/geojson/ne_50m_admin_1_states_provinces.geojson'
			).then((r) => r.json());
			const mxOnly = {
				type: 'FeatureCollection',
				features: mxStates.features.filter((f: any) => f.properties?.adm0_a3 === 'MEX')
			};
			const statePaths = mxOnly.features
				.map((f: any) => {
					try {
						return { d: mxPath(f) ?? '', isNayarit: false };
					} catch {
						return { d: '', isNayarit: false };
					}
				})
				.filter((o: any) => o.d);

			// Layer: grey states first, then Nayarit highlighted on top
			mexicoPaths = [...statePaths, ...mexicoPaths];
		} catch {
			// If CDN unavailable, just show Nayarit polygons alone — still readable
		}

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
