<script context="module" lang="ts">
	// Empty module block — DocRenderer invokes as [[Expenditure]] with no props.
</script>

<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	// ── Data baked in — WHO GHED 2024 release, Mexico (MEX), 2015–2023 ────
	// Source: WHO Global Health Expenditure Database
	// https://apps.who.int/nha/database
	const YEARS = [2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023];

	const DATA = {
		// Total health expenditure, constant 2023 USD (millions)
		che_usd2023: [90430, 89662, 90099, 90603, 91335, 95565, 98403, 99144, 98731],
		// Government health expenditure, constant 2023 USD (millions)
		gghed_usd2023: [47205, 45979, 45532, 44965, 44907, 50537, 49296, 51448, 48010],
		// Total CHE per capita, current USD
		che_pc_usd: [554, 490, 513, 528, 550, 535, 606, 651, 761],
		// Government CHE per capita, current USD
		gghed_pc_usd: [289, 251, 259, 262, 270, 283, 304, 338, 370],
		// Out-of-pocket per capita, current USD
		oop_pc_usd: [227, 203, 214, 224, 232, 207, 251, 254, 314],
		// Total CHE as % of GDP
		che_gdp: [5.53, 5.38, 5.31, 5.24, 5.3, 6.05, 5.88, 5.71, 5.5],
		// Government CHE as % of total CHE
		gghed_che: [52.2, 51.3, 50.5, 49.6, 49.2, 52.9, 50.1, 51.9, 48.6],
		// Out-of-pocket as % of total CHE
		oops_che: [41.0, 41.4, 41.8, 42.3, 42.3, 38.8, 41.4, 39.1, 41.2]
	};

	type View = 'spending' | 'breakdown' | 'percapita' | 'gdp';
	let activeView: View = 'spending';

	// ── Chart rendering ────────────────────────────────────────────────────
	let canvasEl: HTMLCanvasElement;
	let chartInstance: any = null;
	let chartReady = false;

	const COLORS = {
		gold: '#c8960a',
		goldFaint: 'rgba(200,150,10,0.12)',
		purple: '#7b4fa6',
		purpleFaint: 'rgba(123,79,166,0.12)',
		teal: '#1D9E75',
		coral: '#e04040',
		amber: '#BA7517',
		blue: '#378ADD'
	};

	const ANNOTATIONS: Record<View, string> = {
		spending:
			"Mexico's real health expenditure grew from $90B to nearly $99B (constant 2023 USD) between 2015–2023. The 2020 COVID spike added $4B above the prior trend — but by 2023 spending had plateaued, suggesting no structural shift in the baseline.",
		breakdown:
			'Government coverage dipped below 50% in 2017–2019, then briefly recovered during COVID (52.9% in 2020). Out-of-pocket costs — the direct burden on families like the Jimenez-Delgados — remained stuck around 40–42% of all health spending throughout this period, one of the highest rates in the OECD.',
		percapita:
			'Government spending per person rose from $289 to $370 (current USD) over 9 years. But out-of-pocket costs also climbed — from $227 to $314 per person — meaning families absorbed more in absolute terms even as the state spent more. In 2020, OOP actually fell as people avoided clinics during the pandemic.',
		gdp: "Mexico spent a consistent 5.2–5.5% of GDP on health — well below the OECD average of ~9%. The 2020 COVID peak (6.1% of GDP) was a one-year anomaly. By 2023 it had returned to the structural floor, revealing that COVID did not shift Mexico's underlying health investment commitment."
	};

	interface ViewConfig {
		datasets: any[];
		yLabel: string;
		legendItems: { color: string; label: string; dash?: boolean }[];
	}

	function buildConfig(view: View): ViewConfig {
		if (view === 'spending')
			return {
				datasets: [
					{
						label: 'Total health expenditure',
						data: DATA.che_usd2023.map((v) => +(v / 1000).toFixed(1)),
						borderColor: COLORS.gold,
						backgroundColor: COLORS.goldFaint,
						fill: true,
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 4,
						pointBackgroundColor: COLORS.gold
					},
					{
						label: 'Government portion',
						data: DATA.gghed_usd2023.map((v) => +(v / 1000).toFixed(1)),
						borderColor: COLORS.teal,
						backgroundColor: 'transparent',
						fill: false,
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 3,
						borderDash: [6, 3]
					}
				],
				yLabel: 'USD billions (constant 2023)',
				legendItems: [
					{ color: COLORS.gold, label: 'Total health expenditure' },
					{ color: COLORS.teal, label: 'Government portion', dash: true }
				]
			};

		if (view === 'breakdown')
			return {
				datasets: [
					{
						label: 'Government share',
						data: DATA.gghed_che,
						borderColor: COLORS.teal,
						backgroundColor: 'transparent',
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 4,
						pointBackgroundColor: COLORS.teal
					},
					{
						label: 'Out-of-pocket share',
						data: DATA.oops_che,
						borderColor: COLORS.coral,
						backgroundColor: 'transparent',
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 3,
						borderDash: [6, 3]
					}
				],
				yLabel: '% of current health expenditure',
				legendItems: [
					{ color: COLORS.teal, label: 'Government share' },
					{ color: COLORS.coral, label: 'Out-of-pocket share', dash: true }
				]
			};

		if (view === 'percapita')
			return {
				datasets: [
					{
						label: 'Total per capita',
						data: DATA.che_pc_usd,
						borderColor: COLORS.gold,
						backgroundColor: 'transparent',
						tension: 0.35,
						borderWidth: 1.5,
						pointRadius: 3,
						borderDash: [3, 3]
					},
					{
						label: 'Government per capita',
						data: DATA.gghed_pc_usd,
						borderColor: COLORS.teal,
						backgroundColor: 'transparent',
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 4,
						pointBackgroundColor: COLORS.teal
					},
					{
						label: 'Out-of-pocket per capita',
						data: DATA.oop_pc_usd,
						borderColor: COLORS.coral,
						backgroundColor: 'transparent',
						tension: 0.35,
						borderWidth: 2,
						pointRadius: 3,
						borderDash: [6, 3]
					}
				],
				yLabel: 'Current USD per capita',
				legendItems: [
					{ color: COLORS.gold, label: 'Total per capita', dash: true },
					{ color: COLORS.teal, label: 'Government per capita' },
					{ color: COLORS.coral, label: 'Out-of-pocket per capita', dash: true }
				]
			};

		// gdp
		return {
			datasets: [
				{
					label: 'Total health spending % of GDP',
					data: DATA.che_gdp,
					borderColor: COLORS.purple,
					backgroundColor: COLORS.purpleFaint,
					fill: true,
					tension: 0.35,
					borderWidth: 2,
					pointRadius: 4,
					pointBackgroundColor: DATA.che_gdp.map((_, i) =>
						YEARS[i] === 2020 ? '#e04040' : COLORS.purple
					),
					pointRadius: DATA.che_gdp.map((_, i) => (YEARS[i] === 2020 ? 7 : 4))
				},
				{
					label: 'Govt health spending % of GDP',
					data: DATA.gghed_che.map((g, i) => +((DATA.che_gdp[i] * g) / 100).toFixed(2)),
					borderColor: COLORS.teal,
					backgroundColor: 'transparent',
					tension: 0.35,
					borderWidth: 2,
					pointRadius: 3,
					borderDash: [6, 3]
				}
			],
			yLabel: '% of GDP',
			legendItems: [
				{ color: COLORS.purple, label: 'Total health spending % GDP' },
				{ color: COLORS.teal, label: 'Govt health spending % GDP', dash: true }
			]
		};
	}

	async function renderChart(view: View) {
		if (!browser || !canvasEl) return;

		// Lazy-load Chart.js from CDN if not already loaded
		if (!(window as any).Chart) {
			await new Promise<void>((resolve, reject) => {
				const s = document.createElement('script');
				s.src = 'https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js';
				s.onload = () => resolve();
				s.onerror = reject;
				document.head.appendChild(s);
			});
		}

		const Chart = (window as any).Chart;
		const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
		const gridColor = isDark ? 'rgba(255,255,255,0.07)' : 'rgba(0,0,0,0.12)';
		const textColor = isDark ? 'rgba(255,255,255,0.75)' : 'rgba(0,0,0,0.85)';

		if (chartInstance) {
			chartInstance.destroy();
			chartInstance = null;
		}

		const cfg = buildConfig(view);

		// Update legend
		legendItems = cfg.legendItems;

		chartInstance = new Chart(canvasEl, {
			type: 'line',
			data: { labels: YEARS, datasets: cfg.datasets },
			options: {
				responsive: true,
				maintainAspectRatio: false,
				interaction: { mode: 'index', intersect: false },
				plugins: {
					legend: { display: false },
					tooltip: {
						backgroundColor: isDark ? '#1a1030' : '#fff',
						borderColor: isDark ? 'rgba(255,255,255,0.12)' : 'rgba(0,0,0,0.1)',
						borderWidth: 1,
						titleColor: isDark ? 'rgba(255,255,255,0.8)' : 'rgba(0,0,0,0.8)',
						bodyColor: isDark ? 'rgba(255,255,255,0.6)' : 'rgba(0,0,0,0.6)',
						padding: 10,
						callbacks: {
							label: (ctx: any) => {
								const v = ctx.parsed.y;
								if (view === 'spending') return ` ${ctx.dataset.label}: $${v.toFixed(1)}B`;
								if (view === 'gdp') return ` ${ctx.dataset.label}: ${v.toFixed(2)}%`;
								if (view === 'breakdown') return ` ${ctx.dataset.label}: ${v.toFixed(1)}%`;
								return ` ${ctx.dataset.label}: $${Math.round(v)}`;
							}
						}
					}
				},
				scales: {
					x: {
						grid: { color: gridColor },
						ticks: {
							color: textColor,
							font: { size: 11 },
							autoSkip: false,
							maxRotation: 0
						}
					},
					y: {
						grid: { color: gridColor },
						ticks: { color: textColor, font: { size: 11 } },
						title: {
							display: true,
							text: cfg.yLabel,
							color: textColor,
							font: { size: 10 }
						}
					}
				}
			}
		});

		chartReady = true;
	}

	let legendItems: { color: string; label: string; dash?: boolean }[] = [];

	function setView(v: View) {
		activeView = v;
		renderChart(v);
	}

	onMount(() => {
		if (browser) renderChart(activeView);
	});

	onDestroy(() => {
		if (chartInstance) {
			chartInstance.destroy();
			chartInstance = null;
		}
	});
</script>

<!-- ═══════════════════════════════════════════════════════════════════
       Mexico Health Expenditure 2015–2023
       Source: WHO Global Health Expenditure Database (GHED), 2024 release
       Invoked as [[Expenditure]] via DocRenderer shortcode
  ════════════════════════════════════════════════════════════════════ -->
<section class="expenditure-section">
	<!-- Section header -->
	<div class="exp-header">
		<p class="exp-eyebrow">Context</p>
		<h2 class="exp-title">How Mexico Funds Healthcare</h2>
		<p class="exp-dek">
			Nine years of spending data reveal a system built at the floor — consistently underinvested,
			with families bearing an outsized share of the cost.
		</p>
	</div>

	<!-- Stat cards -->
	<div class="stat-grid">
		<div class="stat-card">
			<span class="stat-n">$99B</span>
			<span class="stat-l">Total health spending, 2022 peak (constant 2023 USD)</span>
		</div>
		<div class="stat-card">
			<span class="stat-n">6.1%</span>
			<span class="stat-l">Health spending as % of GDP, 2020 COVID peak</span>
		</div>
		<div class="stat-card">
			<span class="stat-n">41%</span>
			<span class="stat-l">Out-of-pocket share of total spending, 2023</span>
		</div>
		<div class="stat-card stat-card--highlight">
			<span class="stat-n">~9%</span>
			<span class="stat-l">OECD average health spending as % of GDP — nearly double Mexico's</span>
		</div>
	</div>

	<!-- View toggle -->
	<div class="view-tabs" role="tablist" aria-label="Chart view">
		{#each [['spending', 'Total spending'], ['breakdown', 'Govt vs out-of-pocket'], ['percapita', 'Per capita'], ['gdp', '% of GDP']] as const as [v, label]}
			<button
				role="tab"
				aria-selected={activeView === v}
				class="tab-btn"
				class:tab-btn--active={activeView === v}
				on:click={() => setView(v)}>{label}</button
			>
		{/each}
	</div>

	<!-- Legend -->
	<div class="chart-legend" aria-hidden="true">
		{#each legendItems as item}
			<span class="legend-item">
				<span
					class="legend-swatch"
					style="background:{item.dash ? 'transparent' : item.color}; border-top: {item.dash
						? `2px dashed ${item.color}`
						: 'none'}; width:{item.dash ? '18px' : '10px'}; height:{item.dash ? '0' : '10px'}"
				></span>
				{item.label}
			</span>
		{/each}
	</div>

	<!-- Chart -->
	<div class="chart-wrap">
		<canvas
			bind:this={canvasEl}
			role="img"
			aria-label="Mexico health expenditure trends 2015–2023, WHO GHED data"
			>Mexico health spending data 2015–2023. Source: WHO Global Health Expenditure Database.</canvas
		>
	</div>

	<!-- Annotation -->
	<div class="annotation-box">
		<p class="annotation-text">{ANNOTATIONS[activeView]}</p>
	</div>

	<!-- COVID callout -->
	{#if activeView === 'breakdown' || activeView === 'percapita'}
		<div class="callout-box">
			<p class="callout-label">The family's burden</p>
			<p class="callout-body">
				When Jose Jimenez needed oxygen, his family paid out of pocket. Mexico's out-of-pocket share
				— stuck above 40% — means that even with nominal insurance coverage, rural families
				routinely absorb the cost of a system that cannot deliver. An oxygen tank sold on the black
				market for 14,000 pesos (~$700 USD) in late 2020.
			</p>
		</div>
	{/if}

	<!-- Source -->
	<div class="source-line">
		<p>
			Source: <a href="https://apps.who.int/nha/database" target="_blank" rel="noopener noreferrer"
				>WHO Global Health Expenditure Database (GHED), 2024 release</a
			>. NCU = Mexican Peso. Constant USD adjusted to 2023 base year. Years 2015–2023.
		</p>
	</div>
</section>

<style>
	.expenditure-section {
		padding: 4rem 0 3rem;
		max-width: 780px;
		margin: 0 auto;
	}

	/* ── Header ── */
	.exp-eyebrow {
		font-family: 'Syne', sans-serif;
		font-size: 0.62rem;
		letter-spacing: 0.22em;
		text-transform: uppercase;
		color: #c8960a;
		margin: 0 0 0.5rem;
		font-weight: 700;
	}

	.exp-title {
		font-family: 'Playfair Display', Georgia, serif;
		font-size: clamp(1.5rem, 3vw, 2rem);
		font-weight: 700;
		color: #2a0e58;
		margin: 0 0 0.75rem;
		line-height: 1.2;
	}

	.exp-dek {
		font-family: 'Source Serif 4', Georgia, serif;
		font-size: 1rem;
		line-height: 1.7;
		color: #4a3a2a;
		margin: 0 0 2rem;
		max-width: 600px;
	}

	/* ── Stat cards ── */
	.stat-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
		gap: 0.75rem;
		margin-bottom: 2rem;
	}

	.stat-card {
		display: flex;
		flex-direction: column;
		gap: 0.3rem;
		background: #f5f0e8;
		border: 1px solid rgba(200, 150, 10, 0.2);
		border-radius: 3px;
		padding: 1rem 1.1rem;
	}

	.stat-card--highlight {
		background: rgba(42, 14, 88, 0.04);
		border-color: rgba(42, 14, 88, 0.15);
	}

	.stat-n {
		font-family: 'Playfair Display', serif;
		font-size: 1.75rem;
		font-weight: 900;
		color: #c8960a;
		line-height: 1;
	}

	.stat-l {
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: #8a7060;
		line-height: 1.4;
	}

	/* ── Tabs ── */
	.view-tabs {
		display: flex;
		flex-wrap: wrap;
		gap: 6px;
		margin-bottom: 0.75rem;
	}

	.tab-btn {
		font-family: 'Syne', sans-serif;
		font-size: 0.68rem;
		letter-spacing: 0.08em;
		padding: 6px 14px;
		border-radius: 2px;
		cursor: pointer;
		background: transparent;
		border: 1px solid rgba(42, 14, 88, 0.18);
		color: #8a7060;
		transition: all 0.15s;
	}

	.tab-btn:hover {
		background: rgba(42, 14, 88, 0.05);
		color: #2a0e58;
	}

	.tab-btn--active {
		background: #2a0e58;
		border-color: #2a0e58;
		color: #f0c040;
	}

	/* ── Legend ── */
	.chart-legend {
		display: flex;
		flex-wrap: wrap;
		gap: 12px 20px;
		margin-bottom: 6px;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 5px;
		font-family: 'Syne', sans-serif;
		font-size: 0.65rem;
		color: #8a7060;
		letter-spacing: 0.06em;
	}

	.legend-swatch {
		flex-shrink: 0;
		border-radius: 2px;
		display: inline-block;
	}

	/* ── Chart ── */
	.chart-wrap {
		position: relative;
		width: 100%;
		height: 280px;
		margin-bottom: 1.25rem;
	}

	/* ── Annotation ── */
	.annotation-box {
		border-left: 3px solid rgba(200, 150, 10, 0.4);
		padding: 0.75rem 1rem;
		margin-bottom: 1.5rem;
		background: rgba(200, 150, 10, 0.04);
	}

	.annotation-text {
		font-family: 'Source Serif 4', Georgia, serif;
		font-size: 0.9rem;
		line-height: 1.72;
		color: #4a3a2a;
		margin: 0;
	}

	/* ── COVID callout ── */
	.callout-box {
		background: rgba(42, 14, 88, 0.05);
		border: 1px solid rgba(42, 14, 88, 0.12);
		border-radius: 3px;
		padding: 1.25rem 1.5rem;
		margin-bottom: 1.5rem;
	}

	.callout-label {
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		letter-spacing: 0.2em;
		text-transform: uppercase;
		color: #7b4fa6;
		font-weight: 700;
		margin: 0 0 0.5rem;
	}

	.callout-body {
		font-family: 'Source Serif 4', Georgia, serif;
		font-size: 0.88rem;
		line-height: 1.75;
		color: #4a3a2a;
		margin: 0;
	}

	/* ── Source ── */
	.source-line {
		border-top: 1px solid rgba(0, 0, 0, 0.08);
		padding-top: 0.85rem;
	}

	.source-line p {
		font-family: 'Syne', sans-serif;
		font-size: 0.6rem;
		letter-spacing: 0.06em;
		color: #a09080;
		margin: 0;
		line-height: 1.6;
	}

	.source-line a {
		color: #c8960a;
		text-decoration: none;
	}
	.source-line a:hover {
		text-decoration: underline;
	}

	/* ── Responsive ── */
	@media (max-width: 600px) {
		.expenditure-section {
			padding: 2.5rem 0 2rem;
		}
		.stat-grid {
			grid-template-columns: 1fr 1fr;
		}
		.chart-wrap {
			height: 220px;
		}
		.view-tabs {
			gap: 4px;
		}
		.tab-btn {
			font-size: 0.6rem;
			padding: 5px 10px;
		}
	}
</style>
