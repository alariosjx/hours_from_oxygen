<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import type * as D3Type from 'd3';
	import type * as TopoType from 'topojson-client';
	import type { Topology, Objects } from 'topojson-specification';
	import type { FeatureCollection, Feature } from 'geojson';

	let d3!: typeof D3Type;
	let topojson!: typeof TopoType;

	// ── Story steps ───────────────────────────────────────────────────────────────
	const STEPS = [
		{
			id: 'mexico',
			heading: '~800,000 deaths',
			body: 'An estimated 800,000 people died in Mexico during the COVID-19 pandemic — one of the highest death tolls in Latin America. Healthcare workers and underfunded rural hospitals bore a disproportionate share of the crisis.',
			sub: 'But what does geography tell us about who was most at risk?'
		},
		{
			id: 'pacific',
			heading: 'The Pacific Coast',
			body: "Along Mexico's Pacific coast, the states of Nayarit, Jalisco, Colima, and Sinaloa became early epicenters — tourist regions where international visitors continued to arrive even as COVID-19 cases surged.",
			sub: ''
		},
		{
			id: 'nayarit',
			heading: 'Nayarit',
			body: 'Nayarit spans 27,000 km² of coastline, mountains, and valleys. Its 20 municipalities range from wealthy coastal resort towns to remote Sierra communities accessible only by narrow mountain roads.',
			sub: ''
		},
		{
			id: 'nayarit-cls',
			heading: 'Urban vs. Rural',
			body: "Nayarit's limited medical resources were concentrated in its three most populous municipalities: Tepic (the capital), Bahía de Banderas (the tourism hub), and Ixtlán del Río. For the rest of the state, COVID-19 patients were hours from a hospital with oxygen.",
			sub: ''
		},
		{
			id: 'ahuacatlan',
			heading: 'Ahuacatlán',
			body: "Deep in the Sierra Madre foothills — roughly 60 kilometers and over an hour's drive from Tepic — sits Ahuacatlán. When Imelda's father's oxygen levels began to fall, every minute of that drive felt like an eternity.",
			sub: ''
		}
	] as const;

	type StepId = (typeof STEPS)[number]['id'];

	// ── Municipality identifiers (GADM GID_2) ────────────────────────────────────
	const URBAN_GIDS = new Set([
		'MEX.18.18_2', // Tepic
		'MEX.18.4_2',  // Bahía de Banderas
		'MEX.18.8_2'   // Ixtlán del Río
	]);
	const AHUACATLAN_GID = 'MEX.18.2_2';

	const MUN_LABELS: Record<string, string> = {
		'MEX.18.18_2': 'Tepic',
		'MEX.18.4_2':  'Bahía de Banderas',
		'MEX.18.8_2':  'Ixtlán del Río',
		'MEX.18.2_2':  'Ahuacatlán'
	};

	const PACIFIC = new Set(['Nayarit', 'Jalisco', 'Colima', 'Sinaloa']);

	// ── Color palette ─────────────────────────────────────────────────────────────
	const C = {
		bg:              '#f5f0e8',
		state:           '#c9b8e8',
		dim:             '#e8e2d8',
		pacific:         '#9070c8',
		nayaritFill:     '#d4c5f0',
		munBorder:       '#a07010',
		stateBorder:     'rgba(80,50,140,0.5)',
		urban:           '#c04040',
		rural:           '#4a7a8a',
		highlight:       '#c8960a',
		highlightBorder: 'rgba(180,130,0,0.9)',
		text:            '#2a1a0e'
	} as const;

	// ── DOM refs ──────────────────────────────────────────────────────────────────
	let canvasEl: HTMLCanvasElement;
	let outerEl: HTMLElement;

	// ── Reactive state ────────────────────────────────────────────────────────────
	let currentStep = 0;
	let ready = false;

	// ── Internal map state ────────────────────────────────────────────────────────
	let projection: D3Type.GeoProjection;
	let pathCtx: D3Type.GeoPath;           // path generator bound to canvas context
	let W = 0;
	let H = 0;
	let dpr = 1;
	const PAD = 48;

	let mexicoFeatures: FeatureCollection;
	let nayaritFeatures: FeatureCollection;
	let topo!: Topology<Objects>;           // raw topology (for mesh())

	// Animated opacity for the municipality layer (0 = hidden, 1 = visible)
	let munOpacity = 0;
	// Current step id, used inside draw() to pick colors
	let activeId: StepId = 'mexico';

	// ── Helpers ───────────────────────────────────────────────────────────────────
	function gid(f: Feature): string {
		return f.properties?.['GID_2'] ?? '';
	}

	function stateName(f: Feature): string {
		return f.properties?.['name'] ?? '';
	}

	function getStateFill(f: Feature): string {
		const name = stateName(f);
		if (activeId === 'mexico') return C.state;
		if (activeId === 'pacific') return PACIFIC.has(name) ? C.pacific : C.dim;
		return name === 'Nayarit' ? 'transparent' : C.dim;
	}

	function getMunFill(f: Feature): string {
		const g = gid(f);
		if (activeId === 'ahuacatlan') return g === AHUACATLAN_GID ? C.highlight : C.dim;
		if (activeId === 'nayarit-cls') return URBAN_GIDS.has(g) ? C.urban : C.rural;
		return C.nayaritFill;
	}

	function fitParams(geo: FeatureCollection) {
		const p = d3.geoMercator().fitExtent(
			[[PAD, PAD], [W - PAD, H - PAD]],
			geo
		);
		return {
			center:    p.center()    as [number, number],
			scale:     p.scale(),
			translate: p.translate() as [number, number]
		};
	}

	function pacificCollection(): FeatureCollection {
		return {
			type: 'FeatureCollection',
			features: mexicoFeatures.features.filter((f) => PACIFIC.has(stateName(f)))
		};
	}

	function ahuacatlanFeature(): Feature {
		return nayaritFeatures.features.find((f) => gid(f) === AHUACATLAN_GID)!;
	}

	function zoomTarget(id: StepId): FeatureCollection | Feature {
		if (id === 'mexico')    return mexicoFeatures;
		if (id === 'pacific')   return pacificCollection();
		if (id === 'nayarit' || id === 'nayarit-cls') return nayaritFeatures;
		return ahuacatlanFeature();
	}

	// ── Canvas draw ───────────────────────────────────────────────────────────────
	function draw() {
		const ctx = canvasEl.getContext('2d')!;
		ctx.save();
		ctx.scale(dpr, dpr);
		ctx.clearRect(0, 0, W, H);

		// 1. State fills
		for (const feat of mexicoFeatures.features) {
			const fill = getStateFill(feat);
			if (fill === 'transparent') continue;
			ctx.beginPath();
			pathCtx(feat);
			ctx.fillStyle = fill;
			ctx.fill();
		}

		// 2. Municipality fills (drawn before borders so borders appear on top)
		if (munOpacity > 0) {
			ctx.globalAlpha = munOpacity;
			for (const feat of nayaritFeatures.features) {
				ctx.beginPath();
				pathCtx(feat);
				ctx.fillStyle = getMunFill(feat);
				ctx.fill();
			}
			ctx.globalAlpha = 1;
		}

		// 3. State borders (mesh — each shared edge drawn once)
		ctx.beginPath();
		pathCtx(topojson.mesh(topo, topo.objects.states));
		ctx.strokeStyle = C.stateBorder;
		ctx.lineWidth = activeId === 'mexico' ? 1 : 0.5;
		ctx.stroke();

		// 4. Municipality borders (only when visible)
		if (munOpacity > 0) {
			ctx.globalAlpha = munOpacity;

			// Highlight Ahuacatlán border separately
			if (activeId === 'ahuacatlan') {
				const ahua = ahuacatlanFeature();
				ctx.beginPath();
				pathCtx(ahua);
				ctx.strokeStyle = C.highlightBorder;
				ctx.lineWidth = 3;
				ctx.stroke();

				// All other mun borders
				ctx.beginPath();
				pathCtx(
					topojson.mesh(
						topo,
						topo.objects.municipalities,
						(a: any, b: any) => a !== b || a.properties?.GID_2 !== AHUACATLAN_GID
					)
				);
				ctx.strokeStyle = C.munBorder;
				ctx.lineWidth = 1.5;
				ctx.stroke();
			} else {
				ctx.beginPath();
				pathCtx(topojson.mesh(topo, topo.objects.municipalities));
				ctx.strokeStyle = C.munBorder;
				ctx.lineWidth = 1.5;
				ctx.stroke();
			}

			ctx.globalAlpha = 1;
		}

		// 5. Labels for key municipalities
		if (munOpacity > 0.5) {
			ctx.globalAlpha = Math.min(1, (munOpacity - 0.5) * 2);
			ctx.fillStyle = C.text;
			ctx.font = `600 ${Math.round(11 * Math.min(1, W / 800))}px Georgia, serif`;
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			for (const feat of nayaritFeatures.features) {
				const g = gid(feat);
				const showLabel =
					(activeId === 'nayarit-cls' && (URBAN_GIDS.has(g) || g === AHUACATLAN_GID)) ||
					(activeId === 'ahuacatlan' && g === AHUACATLAN_GID);
				if (!showLabel) continue;

				const label = MUN_LABELS[g];
				if (!label) continue;

				const [cx, cy] = pathCtx.centroid(feat);
				if (!isNaN(cx) && !isNaN(cy)) {
					// Subtle text shadow for readability
					ctx.strokeStyle = C.bg;
					ctx.lineWidth = 3;
					ctx.lineJoin = 'round';
					ctx.strokeText(label, cx, cy);
					ctx.fillText(label, cx, cy);
				}
			}
			ctx.globalAlpha = 1;
		}

		ctx.restore();
	}

	// ── Projection + opacity animation ────────────────────────────────────────────
	function animateStep(idx: number, animate = true) {
		if (!d3 || !mexicoFeatures || !nayaritFeatures) return;

		const id = STEPS[idx].id;
		activeId = id;

		const target = fitParams(zoomTarget(id) as FeatureCollection);
		const showMuns = id !== 'mexico' && id !== 'pacific';
		const dur = animate ? 850 : 0;

		const iCenter = d3.interpolate(projection.center() as [number, number], target.center);
		const iScale  = d3.interpolate(projection.scale(), target.scale);
		const iTrans  = d3.interpolate(projection.translate() as [number, number], target.translate);
		const iOpac   = d3.interpolate(munOpacity, showMuns ? 1 : 0);

		// Use a D3 transition on the canvas element for timing + easing
		d3.select(canvasEl)
			.transition('step')
			.duration(dur)
			.ease(d3.easeCubicInOut)
			.tween('render', () => (t: number) => {
				projection
					.center(iCenter(t))
					.scale(iScale(t))
					.translate(iTrans(t));
				munOpacity = iOpac(t);
				draw();
			});
	}

	// ── Scroll → step detection ────────────────────────────────────────────────────
	let rafId = 0;

	function onScroll() {
		cancelAnimationFrame(rafId);
		rafId = requestAnimationFrame(() => {
			if (!outerEl) return;
			const sectionTop = outerEl.offsetTop;
			const scrollable = outerEl.offsetHeight - window.innerHeight;
			const scrolledInto = window.scrollY - sectionTop;
			const progress = scrollable > 0
				? Math.min(1, Math.max(0, scrolledInto / scrollable))
				: 0;
			const next = Math.round(progress * (STEPS.length - 1));
			if (next !== currentStep) {
				currentStep = next;
				animateStep(currentStep);
			}
		});
	}

	// ── Cleanup ───────────────────────────────────────────────────────────────────
	let ro: ResizeObserver | undefined;

	onDestroy(() => {
		if (browser) {
			window.removeEventListener('scroll', onScroll);
			cancelAnimationFrame(rafId);
		}
		ro?.disconnect();
	});

	// ── Init ──────────────────────────────────────────────────────────────────────
	onMount(async () => {
		[d3, topojson] = await Promise.all([
			import('d3'),
			import('topojson-client')
		]);

		// Size canvas for device pixel ratio (crisp on retina)
		dpr = window.devicePixelRatio || 1;
		W   = canvasEl.clientWidth  || window.innerWidth;
		H   = canvasEl.clientHeight || window.innerHeight;
		canvasEl.width  = W * dpr;
		canvasEl.height = H * dpr;

		// Load single TopoJSON file (contains both layers)
		topo = await fetch('/geo/mexico-simple.topojson').then((r) => {
			if (!r.ok) throw new Error(`mexico-simple.topojson: ${r.status}`);
			return r.json();
		});

		mexicoFeatures  = topojson.feature(topo, topo.objects.states) as FeatureCollection;
		nayaritFeatures = topojson.feature(topo, topo.objects.municipalities) as FeatureCollection;

		// Set up projection + canvas path generator
		projection = d3.geoMercator().fitExtent(
			[[PAD, PAD], [W - PAD, H - PAD]],
			mexicoFeatures
		);
		const ctx = canvasEl.getContext('2d')!;
		pathCtx = d3.geoPath(projection, ctx);

		ready = true;
		animateStep(0, false);

		window.addEventListener('scroll', onScroll, { passive: true });

		ro = new ResizeObserver(() => {
			dpr = window.devicePixelRatio || 1;
			W   = canvasEl.clientWidth;
			H   = canvasEl.clientHeight;
			canvasEl.width  = W * dpr;
			canvasEl.height = H * dpr;

			// Refit to Mexico then re-apply current step instantly
			projection.fitExtent([[PAD, PAD], [W - PAD, H - PAD]], mexicoFeatures);
			animateStep(currentStep, false);
		});
		ro.observe(canvasEl);
	});
</script>

<!--
  Geography.svelte — Canvas + TopoJSON scrollytelling map
  Data: /static/geo/mexico-simple.topojson  (GADM 4.1, simplified ~0.05°)
  Steps: Mexico → Pacific coast → Nayarit → Urban/Rural → Ahuacatlán
-->
<section class="geo-section" bind:this={outerEl}>
	<div class="geo-sticky">
		<canvas bind:this={canvasEl} class="geo-canvas" aria-label="Scrolling map"></canvas>

		{#if ready}
			<div class="geo-card" class:right-side={currentStep >= 2}>
				<p class="card-heading">{STEPS[currentStep].heading}</p>
				<p class="card-body">{STEPS[currentStep].body}</p>
				{#if STEPS[currentStep].sub}
					<p class="card-sub">{STEPS[currentStep].sub}</p>
				{/if}
			</div>

			{#if currentStep === 3}
				<div class="geo-legend">
					<div class="legend-row">
						<span class="dot" style="background:{C.urban}"></span>
						Urban municipalities
					</div>
					<div class="legend-row">
						<span class="dot" style="background:{C.rural}"></span>
						Rural municipalities
					</div>
				</div>
			{/if}

			{#if currentStep === 0}
				<div class="scroll-nudge" aria-hidden="true">scroll to explore ›</div>
			{/if}
		{/if}
	</div>

	<div class="geo-spacer"></div>
</section>

<style>
	.geo-section {
		position: relative;
		background: #f5f0e8;
	}

	.geo-sticky {
		position: sticky;
		top: 0;
		height: 100vh;
		width: 100%;
		overflow: hidden;
	}

	.geo-canvas {
		width: 100%;
		height: 100%;
		display: block;
		background: #f5f0e8;
	}

	/* ── Text card ─────────────────────────────────────────────────────── */
	.geo-card {
		position: absolute;
		bottom: 10vh;
		left: 5vw;
		max-width: min(400px, 42vw);
		background: rgba(245, 240, 232, 0.92);
		border: 1px solid rgba(140, 100, 0, 0.4);
		border-radius: 8px;
		padding: 1.3rem 1.5rem;
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		transition: left 0.5s ease, right 0.5s ease;
	}

	.geo-card.right-side {
		left: auto;
		right: 5vw;
	}

	.card-heading {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(1.05rem, 2vw, 1.45rem);
		font-weight: 700;
		color: #8a6010;
		margin: 0 0 0.5rem;
		line-height: 1.2;
	}

	.card-body {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.82rem, 1.3vw, 0.97rem);
		color: #2a1a0e;
		line-height: 1.65;
		margin: 0;
	}

	.card-sub {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(0.78rem, 1.1vw, 0.88rem);
		color: rgba(42, 26, 14, 0.6);
		font-style: italic;
		margin: 0.6rem 0 0;
	}

	/* ── Legend ────────────────────────────────────────────────────────── */
	.geo-legend {
		position: absolute;
		top: 2.5rem;
		left: 5vw;
		background: rgba(245, 240, 232, 0.92);
		border: 1px solid rgba(140, 100, 0, 0.35);
		border-radius: 6px;
		padding: 0.75rem 1rem;
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.82rem;
		color: #2a1a0e;
	}

	.legend-row {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-bottom: 0.3rem;
	}
	.legend-row:last-child { margin-bottom: 0; }

	.dot {
		width: 10px;
		height: 10px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	/* ── Scroll nudge ──────────────────────────────────────────────────── */
	.scroll-nudge {
		position: absolute;
		bottom: 2rem;
		right: 3vw;
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.75rem;
		color: rgba(140, 100, 0, 0.7);
		letter-spacing: 0.06em;
		font-style: italic;
		pointer-events: none;
	}

	/* ── Scroll spacer — 5 steps × 100vh ──────────────────────────────── */
	.geo-spacer { height: 500vh; }

	@media (max-width: 640px) {
		.geo-card,
		.geo-card.right-side {
			left: 1rem;
			right: 1rem;
			max-width: none;
			bottom: 2.5rem;
		}
		.geo-legend { top: 1rem; left: 1rem; }
	}
</style>
