<script lang="ts">
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';
	import Chart from 'chart.js/auto'; // Import Chart.js

	let canvasEl: HTMLCanvasElement;

	// Default data for the chart
	export let labels: string[] = [
		'IMSS',
		'ISSSTE',
		'Ministry of Health\n(Sec. Salud)',
		'Private hospitals'
	];
	export let data: number[] = [50.3, 45.1, 43.7, 23.9];
	export let colors: string[] = ['#1a0848', '#2d1a6e', '#3a2270', '#c04040'];
	export let description: string =
		'Horizontal bar chart of COVID-19 in-hospital fatality rates by institution in Mexico, 2020–2023. IMSS 50.3%, ISSSTE 45.1%, Ministry of Health 43.7%, Private 23.9%.';
	export let source: string =
		'Source: Informe COVID 2024, Cap. 3, Gráfica 3, pp. 115–116. Rates calculated from raw totals: IMSS 182,600 / 362,888; ISSSTE 25,215 / 55,909; Sec. Salud 100,197 / 229,185; Privado 7,360 / 30,769. Emergency period: March 2020 – May 9, 2023.';
	export let annotation: string =
		'Private hospitals recorded a fatality rate of 23.9% — less than half the rate seen at IMSS, where more than 1 in 2 hospitalized patients died. Even after controlling for age, comorbidities, and municipality, patients at IMSS had approximately 2.6× the odds of dying compared to those at Ministry of Health facilities.';
	export let citation: string =
		'Comisión Independiente de Investigación sobre la Pandemia de COVID-19 en México, Informe COVID 2024';

	onMount(() => {
		if (!browser) return; // Ensure this code only runs in the browser

		new Chart(canvasEl, {
			type: 'bar',
			data: {
				labels,
				datasets: [
					{
						data,
						backgroundColor: colors,
						borderColor: colors,
						borderWidth: 0,
						borderRadius: 2,
						barThickness: 18
					}
				]
			},
			options: {
				indexAxis: 'y',
				responsive: true,
				maintainAspectRatio: false,
				plugins: {
					legend: { display: false },
					tooltip: {
						backgroundColor: '#1a0848',
						titleFont: { family: "'Crimson Text', Georgia, serif", size: 13 },
						bodyFont: { family: "'Crimson Text', Georgia, serif", size: 13 },
						callbacks: {
							label: (ctx) => ` ${ctx.parsed.x.toFixed(1)}% of hospitalized patients died`
						}
					}
				},
				scales: {
					x: {
						min: 0,
						max: 60,
						grid: { color: 'rgba(26,8,72,0.07)', lineWidth: 0.5 },
						border: { display: false },
						ticks: {
							color: '#9a9080',
							font: { family: "'Crimson Text', Georgia, serif", size: 12 },
							callback: (v) => v + '%'
						}
					},
					y: {
						grid: { display: false },
						border: { display: false },
						ticks: {
							color: '#3a2e20',
							font: { family: "'Crimson Text', Georgia, serif", size: 13 },
							crossAlign: 'far'
						}
					}
				}
			},
			plugins: [
				{
					afterDatasetsDraw(chart) {
						const ctx = chart.ctx;
						const ds = chart.data.datasets[0];
						const meta = chart.getDatasetMeta(0);
						meta.data.forEach((bar, j) => {
							const val = ds.data[j];
							ctx.fillStyle = '#3a2e20';
							ctx.font = "500 12px 'Crimson Text', Georgia, serif";
							ctx.textAlign = 'left';
							ctx.fillText(val.toFixed(1) + '%', bar.x + 5, bar.y + 4.5);
						});
					}
				}
			]
		});
	});
</script>

<div class="hfo-wrap">
	<div class="hfo-legend">
		<span><span class="hfo-swatch" style="background:#1a0848;"></span>Public sector</span>
		<span><span class="hfo-swatch" style="background:#c04040;"></span>Private sector</span>
	</div>

	<div style="position:relative; width:100%; height:180px;">
		<canvas bind:this={canvasEl} aria-label={description}>{description}</canvas>
	</div>

	<blockquote class="hfo-bq">
		<p>{annotation}</p>
		<cite>{citation}</cite>
	</blockquote>

	<p class="hfo-source">{source}</p>
</div>

<style>
	.hfo-wrap {
		font-family: 'Crimson Text', Georgia, serif;
		padding: 1.5rem 0 0.5rem;
		max-width: 640px;
		margin: 0 auto;
	}
	.hfo-legend {
		display: flex;
		gap: 20px;
		margin-bottom: 1.2rem;
		font-size: 13px;
		color: #5a5040;
	}
	.hfo-legend span {
		display: flex;
		align-items: center;
		gap: 6px;
	}
	.hfo-swatch {
		width: 10px;
		height: 10px;
		border-radius: 1px;
	}
	blockquote.hfo-bq {
		margin: 1.4rem 0 0 0;
		padding: 0 0 0 1.2rem;
		border-left: 3px solid #c04040;
		font-size: 1.15rem;
		font-style: italic;
		color: #3a2e20;
		line-height: 1.65;
	}
	blockquote.hfo-bq p {
		margin: 0 0 0.4rem;
	}
	blockquote.hfo-bq cite {
		font-style: normal;
		font-size: 0.78rem;
		color: #c04040;
		letter-spacing: 0.04em;
		text-transform: lowercase;
		display: block;
		margin-top: 0.35rem;
	}
	.hfo-source {
		font-size: 11px;
		color: #9a9080;
		margin-top: 0.9rem;
		font-style: italic;
		line-height: 1.5;
	}
</style>
