<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import type * as D3Type from 'd3';
	import type { FeatureCollection, Feature, GeoJsonObject } from 'geojson';

	// D3 loaded dynamically in onMount — avoids SSR "window is not defined" errors.
	let d3: typeof D3Type = undefined as any;

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

	// ── Municipality classifications (INEGI CVE_MUN codes) ───────────────────────
	const URBAN_MUNS = new Set(['017', '020', '006']); // Tepic, Bahía de Banderas, Ixtlán del Río
	const AHUACATLAN = '002';

	// ── Pacific-coast states ──────────────────────────────────────────────────────
	const PACIFIC = new Set(['Nayarit', 'Jalisco', 'Colima', 'Sinaloa']);

	// ── Color palette ─────────────────────────────────────────────────────────────
	const C = {
		bg: '#f5f0e8',
		state: '#c9b8e8',
		dim: '#e8e2d8',
		pacific: '#9070c8',
		nayaritFill: '#d4c5f0',
		munBorder: '#c8960a',
		urban: '#c04040',
		rural: '#4a7a8a',
		highlight: '#c8960a',
		stateBorder: 'rgba(80,50,140,0.55)',
		highlightBorder: 'rgba(180,130,0,0.9)',
		text: '#2a1a0e'
	} as const;

	// ── DOM refs ──────────────────────────────────────────────────────────────────
	let svgEl: SVGSVGElement;
	let outerEl: HTMLElement;

	// ── Reactive state ────────────────────────────────────────────────────────────
	let currentStep = 0;
	let ready = false;

	// ── D3 internals — declared at module scope so helpers can reference them ─────
	// These are only assigned inside onMount, after D3 is imported.
	let projection: d3.GeoProjection;
	let pathGen: d3.GeoPath;
	let gMap: d3.Selection<SVGGElement, unknown, null, undefined>;
	let W = 0;
	let H = 0;
	const PAD = 48;

	// ── Geo data ──────────────────────────────────────────────────────────────────
	let mexicoGeo: FeatureCollection;
	let nayaritGeo: FeatureCollection;

	// ── Geo helpers ───────────────────────────────────────────────────────────────
	// IMPORTANT: these functions are called AFTER d3, mexicoGeo, nayaritGeo, and
	// gMap are all assigned inside onMount. Do not call them before that point.

	function fitParams(geo: GeoJsonObject) {
		const p = d3.geoMercator().fitExtent(
			[
				[PAD, PAD],
				[W - PAD, H - PAD]
			],
			geo as FeatureCollection
		);
		return {
			center: p.center() as [number, number],
			scale: p.scale(),
			translate: p.translate() as [number, number]
		};
	}
	function pacificCollection(): FeatureCollection {
		return {
			type: 'FeatureCollection',
			features: mexicoGeo.features.filter((f) => {
				const name = (f.properties as any)?.name;
				return PACIFIC.has(name);
			})
		};
	}

	function ahuacatlanFeature(): Feature {
		return nayaritGeo.features.find((f) => (f.properties as any)?.CVE_MUN === AHUACATLAN)!;
	}

	function zoomTargetFor(id: StepId): GeoJsonObject {
		if (id === 'mexico') return mexicoGeo;
		if (id === 'pacific') return pacificCollection();
		if (id === 'nayarit' || id === 'nayarit-cls') return nayaritGeo;
		return ahuacatlanFeature();
	}

	// ── Projection tween ──────────────────────────────────────────────────────────
	// Animates the D3 projection's center/scale/translate between two geo extents.
	// Writing directly into the projection and re-calling pathGen means no DOM
	// transform attribute is needed — paths recompute in place.
	function tweenProjection(geo: GeoJsonObject, dur: number) {
		const target = fitParams(geo);
		console.log('Tweening projection to:', target);

		const iCenter = d3.interpolate(projection.center() as [number, number], target.center);
		const iScale = d3.interpolate(projection.scale(), target.scale);
		const iTrans = d3.interpolate(projection.translate() as [number, number], target.translate);

		d3.select(svgEl)
			.transition('proj')
			.duration(dur)
			.ease(d3.easeCubicInOut)
			.tween('projection', () => (t: number) => {
				projection.center(iCenter(t)).scale(iScale(t)).translate(iTrans(t));

				gMap
					.selectAll<SVGPathElement, Feature>('.s-state, .s-mun')
					.attr('d', (d) => pathGen(d) ?? '');

				gMap
					.selectAll<SVGTextElement, Feature>('.s-label')
					.attr('x', (d) => pathGen.centroid(d)[0])
					.attr('y', (d) => pathGen.centroid(d)[1]);
			});
	}

	// ── Full step → map update ────────────────────────────────────────────────────
	function applyStep(idx: number, animate = true) {
		// Guard: bail if D3 or the map aren't initialised yet.
		// This can happen if onScroll fires before onMount completes.
		if (!d3 || !gMap || !mexicoGeo || !nayaritGeo) return;

		const dur = animate ? 850 : 0;
		const id = STEPS[idx].id;

		// 1. Zoom projection to the target geo extent
		tweenProjection(zoomTargetFor(id), dur);

		// 2. State layer fills
		gMap
			.selectAll<SVGPathElement, Feature>('.s-state')
			.transition('fill')
			.duration(dur)
			.ease(d3.easeCubicInOut)
			.attr('fill', (d) => {
				const name = (d.properties as any)?.name;
				if (id === 'mexico') return C.state;
				if (id === 'pacific') return PACIFIC.has(name) ? C.pacific : C.dim;
				// Nayarit steps: Nayarit itself is covered by the mun layer → transparent
				return name === 'Nayarit' ? 'transparent' : C.dim;
			})
			.attr('stroke', (d) => {
				const name = (d.properties as any)?.name;
				if (id === 'pacific' && name === 'Nayarit') return C.highlightBorder;
				return C.stateBorder;
			})
			.attr('stroke-opacity', id === 'mexico' ? 1 : 0.3);

		// 3. Municipality fills + visibility
		// showMuns is true for steps nayarit, nayarit-cls, ahuacatlan
		const showMuns = id !== 'mexico' && id !== 'pacific';

		gMap
			.selectAll<SVGPathElement, Feature>('.s-mun')
			.transition('munfill')
			.duration(dur)
			.ease(d3.easeCubicInOut)
			// FIX: set opacity BEFORE fill so the transition doesn't start from 0
			// on a newly-visible element with no fill set yet.
			.attr('opacity', showMuns ? 1 : 0)
			.attr('fill', (d) => {
				const code = (d.properties as any)?.CVE_MUN;
				if (id === 'ahuacatlan') return code === AHUACATLAN ? C.highlight : C.dim;
				if (id === 'nayarit-cls') return URBAN_MUNS.has(code) ? C.urban : C.rural;
				return C.nayaritFill;
			})
			.attr('stroke', (d) => {
				if (id === 'ahuacatlan' && (d.properties as any)?.CVE_MUN === AHUACATLAN)
					return C.highlight;
				return C.munBorder;
			})
			.attr('stroke-width', (d) => {
				if (id === 'ahuacatlan' && (d.properties as any)?.CVE_MUN === AHUACATLAN) return 3;
				return 1.5;
			});

		// 4. Municipality name labels
		gMap
			.selectAll<SVGTextElement, Feature>('.s-label')
			.transition('label')
			.duration(dur)
			.attr('opacity', (d) => {
				if (!showMuns) return 0;
				const code = (d.properties as any)?.CVE_MUN;
				if (id === 'ahuacatlan') return code === AHUACATLAN ? 1 : 0;
				if (id === 'nayarit-cls') return URBAN_MUNS.has(code) || code === AHUACATLAN ? 1 : 0;
				return 0;
			});
	}

	// ── Scroll → step detection ────────────────────────────────────────────────────
	// FIX: The original used getBoundingClientRect().top which is viewport-relative.
	// That gives the wrong "scrolled" value when the section is mid-page.
	// We now compute progress using the section's offsetTop (document-relative)
	// compared to window.scrollY, which is also document-relative.
	let rafId = 0;

	function onScroll() {
		cancelAnimationFrame(rafId);
		rafId = requestAnimationFrame(() => {
			if (!outerEl) return;

			// How far the top of the section is from the top of the document (px)
			const sectionTop = outerEl.offsetTop;
			// Total scrollable distance within this section:
			// section height minus one viewport (the sticky panel itself)
			const scrollable = outerEl.offsetHeight - window.innerHeight;
			// How far the user has scrolled INTO this section
			// (negative before we reach it, positive once we're inside)
			const scrolledInto = window.scrollY - sectionTop;
			// Clamp to [0, 1]
			const progress = scrollable > 0 ? Math.min(1, Math.max(0, scrolledInto / scrollable)) : 0;

			// Map progress → step index
			// Using Math.min ensures we never exceed the last step
			const next = Math.round(progress * (STEPS.length - 1));

			if (next !== currentStep) {
				currentStep = next;
				applyStep(currentStep);
			}
		});
	}

	// ── Cleanup refs (populated in onMount, used in onDestroy) ───────────────────
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
		// 1. Import D3 client-side only
		d3 = await import('d3');

		// 2. Measure the SVG viewport
		W = svgEl.clientWidth || window.innerWidth;
		H = svgEl.clientHeight || window.innerHeight;
		console.log('[Geography] SVG dimensions:', W, H);

		// 3. Load GeoJSON files in parallel from /static/geo/
		[mexicoGeo, nayaritGeo] = await Promise.all([
			fetch('/geo/mexico-states.geojson').then((r) => {
				if (!r.ok) throw new Error(`Failed to load mexico-states.geojson: ${r.status}`);
				return r.json();
			}),
			fetch('/geo/nayarit-mun.geojson').then((r) => {
				if (!r.ok) throw new Error(`Failed to load nayarit-mun.geojson: ${r.status}`);
				return r.json();
			})
		]);
		console.log('[Geography] Mexico GeoJSON:', mexicoGeo);
		console.log('[Geography] Nayarit GeoJSON:', nayaritGeo);

		// Debug GeoJSON bounds
		console.log('[Geography] Mexico GeoJSON bounds:', d3.geoBounds(mexicoGeo));
		console.log('[Geography] Nayarit GeoJSON bounds:', d3.geoBounds(nayaritGeo));

		// 4. Set up projection fitted to full Mexico
		projection = d3.geoMercator().fitExtent(
			[
				[PAD, PAD],
				[W - PAD, H - PAD]
			],
			mexicoGeo
		);
		console.log('[Geography] Projection parameters:', {
			center: projection.center(),
			scale: projection.scale(),
			translate: projection.translate()
		});
		pathGen = d3.geoPath(projection);

		// 5. Create the main <g> that holds all map paths
		const svg = d3.select(svgEl);
		gMap = svg.append('g');

		// 6. Draw Mexico state layer
		gMap
			.selectAll('.s-state')
			.data(mexicoGeo.features)
			.join('path')
			.attr('class', 's-state')
			.attr('d', (d) => pathGen(d as Feature) ?? '')
			.attr('fill', C.state)
			.attr('stroke', C.stateBorder)
			.attr('stroke-width', 1);
		console.log('[Geography] State paths:', gMap.selectAll('.s-state').nodes());

		// 7. Draw Nayarit municipality layer (opacity:0 — hidden until step 2)
		gMap
			.selectAll('.s-mun')
			.data(nayaritGeo.features)
			.join('path')
			.attr('class', 's-mun')
			.attr('d', (d) => pathGen(d as Feature) ?? '')
			.attr('fill', C.nayaritFill)
			.attr('stroke', C.munBorder)
			.attr('stroke-width', 1.5)
			.attr('opacity', 0); // hidden on load — applyStep will show them
		console.log('[Geography] Municipality paths:', gMap.selectAll('.s-mun').nodes());

		// 8. Draw municipality labels (opacity:0 — shown selectively per step)
		gMap
			.selectAll('.s-label')
			.data(nayaritGeo.features)
			.join('text')
			.attr('class', 's-label')
			.attr('x', (d) => pathGen.centroid(d as Feature)[0])
			.attr('y', (d) => pathGen.centroid(d as Feature)[1])
			.attr('text-anchor', 'middle')
			.attr('dominant-baseline', 'middle')
			.attr('fill', C.text)
			.attr('font-size', '9')
			.attr('font-family', 'Crimson Text, Georgia, serif')
			.attr('pointer-events', 'none')
			.attr('opacity', 0)
			.text((d) => (d.properties as any).NOMGEO ?? '');

		// 9. Mark component ready and draw step 0 immediately (no animation)
		ready = true;
		applyStep(0, false);

		// 10. Attach scroll listener
		window.addEventListener('scroll', onScroll, { passive: true });

		// 11. Handle resize — refit projection and redraw all paths
		ro = new ResizeObserver(() => {
			W = svgEl.clientWidth;
			H = svgEl.clientHeight;
			projection.fitExtent(
				[
					[PAD, PAD],
					[W - PAD, H - PAD]
				],
				mexicoGeo
			);
			gMap
				.selectAll<SVGPathElement, Feature>('.s-state, .s-mun')
				.attr('d', (d) => pathGen(d) ?? '');
			gMap
				.selectAll<SVGTextElement, Feature>('.s-label')
				.attr('x', (d) => pathGen.centroid(d)[0])
				.attr('y', (d) => pathGen.centroid(d)[1]);
			applyStep(currentStep, false);
		});
		ro.observe(svgEl);
	});
</script>

<!--
  Geography.svelte — Scrollytelling map section
  5 steps: Full Mexico → Pacific coast → Nayarit → Urban/Rural → Ahuacatlán
  The SVG map is sticky; D3 tweens the projection between steps as user scrolls.

  Required files in /static/geo/:
    - mexico-states.geojson   (property: name — English state names)
    - nayarit-mun.geojson     (properties: CVE_MUN, NOMGEO — from INEGI, WGS84)
-->
<section class="geo-section" bind:this={outerEl}>
	<!-- Sticky panel: map + text overlay pins to viewport top while spacer scrolls -->
	<div class="geo-sticky">
		<svg
			bind:this={svgEl}
			class="geo-svg"
			role="img"
			aria-label="Scrolling map showing Mexico, the Pacific coast, Nayarit, and Ahuacatlán"
		></svg>

		{#if ready}
			<!-- Text card — left side for Mexico/Pacific steps, right for Nayarit steps -->
			<div class="geo-card" class:right-side={currentStep >= 2}>
				<p class="card-heading">{STEPS[currentStep].heading}</p>
				<p class="card-body">{STEPS[currentStep].body}</p>
				{#if STEPS[currentStep].sub}
					<p class="card-sub">{STEPS[currentStep].sub}</p>
				{/if}
			</div>

			<!-- Legend: only shown on step 3 (urban/rural classification) -->
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

			<!-- Scroll nudge — only on first step -->
			{#if currentStep === 0}
				<div class="scroll-nudge" aria-hidden="true">scroll to explore ›</div>
			{/if}
		{/if}
	</div>

	<!-- Scroll spacer: 5 steps × 100vh of runway each -->
	<div class="geo-spacer"></div>
</section>

<style>
	.geo-section {
		position: relative;
		background: #f5f0e8;
	}

	/* Sticky map — 100vh tall, full width, clips overflow during transitions */
	.geo-sticky {
		position: sticky;
		top: 0;
		height: 100vh;
		width: 100%;
		overflow: hidden;
	}

	.geo-svg {
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
		transition:
			left 0.5s ease,
			right 0.5s ease;
	}

	/* Shift card right for Nayarit-focused steps */
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
	.legend-row:last-child {
		margin-bottom: 0;
	}

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

	/* ── Scroll spacer ─────────────────────────────────────────────────── */
	/* 5 steps × 100vh each. Increase to 150vh per step for slower transitions. */
	.geo-spacer {
		height: 500vh;
	}

	/* ── Mobile ────────────────────────────────────────────────────────── */
	@media (max-width: 640px) {
		.geo-card,
		.geo-card.right-side {
			left: 1rem;
			right: 1rem;
			max-width: none;
			bottom: 2.5rem;
		}
		.geo-legend {
			top: 1rem;
			left: 1rem;
		}
	}
</style>
