<!--
  ============================================================
  FirstBanner.svelte
  Intro banner — sits directly below LandingHero.svelte

  WHAT THIS COMPONENT DOES:
  - Displays a one-sentence intro lede, byline, and date
  - God's eye (ojo de dios) SVG borders top and bottom,
    inspired by Cora/Huichol woven craft patterns of Nayarit
  - Light warm background — adjust --banner-bg to taste in VS Code
  - Bootstrap 5 grid handles all responsive column widths:
      phone:   full width (col-12)
      tablet:  col-sm-10 / col-md-8
      desktop: col-lg-6  / col-xl-5  (centered, readable measure)
  - Crimson Text serif matches LandingHero headline font

  HOW TO USE:
  1. Place in src/lib/components/FirstBanner.svelte
  2. In +page.svelte, import right after LandingHero:
       <script>
         import LandingHero  from '$lib/components/LandingHero.svelte';
         import FirstBanner  from '$lib/components/FirstBanner.svelte';
       </script>
       <LandingHero />
       <FirstBanner />

  CUSTOMIZATION:
  - Background color: change --banner-bg CSS variable below
  - Lede sentence: edit the <p class="lede"> text
  - Date: edit the <span class="byline-date"> text
  - Border height/color: adjust the SVG viewBox height and fill values
  ============================================================
-->

<!-- No JS needed — purely presentational -->

<!-- God's eye TOP border SVG
     Zigzag/chevron pattern: gold wave + terracotta fill triangles + gold dots.
     preserveAspectRatio="none" stretches it to full width at any screen size. -->
<svg
	class="gods-eye-border"
	viewBox="0 0 1440 28"
	preserveAspectRatio="none"
	xmlns="http://www.w3.org/2000/svg"
	aria-hidden="true"
>
	<!-- Background strip — matches hero bottom color for seamless join -->
	<rect width="1440" height="28" fill="#2a0e58" />
	<!-- Gold zigzag wave -->
	<polyline
		points="0,0 18,14 36,0 54,14 72,0 90,14 108,0 126,14 144,0 162,14 180,0 198,14
               216,0 234,14 252,0 270,14 288,0 306,14 324,0 342,14 360,0 378,14 396,0
               414,14 432,0 450,14 468,0 486,14 504,0 522,14 540,0 558,14 576,0 594,14
               612,0 630,14 648,0 666,14 684,0 702,14 720,0 738,14 756,0 774,14 792,0
               810,14 828,0 846,14 864,0 882,14 900,0 918,14 936,0 954,14 972,0 990,14
               1008,0 1026,14 1044,0 1062,14 1080,0 1098,14 1116,0 1134,14 1152,0 1170,14
               1188,0 1206,14 1224,0 1242,14 1260,0 1278,14 1296,0 1314,14 1332,0 1350,14
               1368,0 1386,14 1404,0 1422,14 1440,0"
		fill="none"
		stroke="#c8960a"
		stroke-width="2.5"
	/>
	<!-- Terracotta fill triangles hanging from each valley -->
	<g fill="#c04040">
		<polygon points="18,14 12,28 24,28" /><polygon points="54,14 48,28 60,28" />
		<polygon points="90,14 84,28 96,28" /><polygon points="126,14 120,28 132,28" />
		<polygon points="162,14 156,28 168,28" /><polygon points="198,14 192,28 204,28" />
		<polygon points="234,14 228,28 240,28" /><polygon points="270,14 264,28 276,28" />
		<polygon points="306,14 300,28 312,28" /><polygon points="342,14 336,28 348,28" />
		<polygon points="378,14 372,28 384,28" /><polygon points="414,14 408,28 420,28" />
		<polygon points="450,14 444,28 456,28" /><polygon points="486,14 480,28 492,28" />
		<polygon points="522,14 516,28 528,28" /><polygon points="558,14 552,28 564,28" />
		<polygon points="594,14 588,28 600,28" /><polygon points="630,14 624,28 636,28" />
		<polygon points="666,14 660,28 672,28" /><polygon points="702,14 696,28 708,28" />
		<polygon points="738,14 732,28 744,28" /><polygon points="774,14 768,28 780,28" />
		<polygon points="810,14 804,28 816,28" /><polygon points="846,14 840,28 852,28" />
		<polygon points="882,14 876,28 888,28" /><polygon points="918,14 912,28 924,28" />
		<polygon points="954,14 948,28 960,28" /><polygon points="990,14 984,28 996,28" />
		<polygon points="1026,14 1020,28 1032,28" /><polygon points="1062,14 1056,28 1068,28" />
		<polygon points="1098,14 1092,28 1104,28" /><polygon points="1134,14 1128,28 1140,28" />
		<polygon points="1170,14 1164,28 1176,28" /><polygon points="1206,14 1200,28 1212,28" />
		<polygon points="1242,14 1236,28 1248,28" /><polygon points="1278,14 1272,28 1284,28" />
		<polygon points="1314,14 1308,28 1320,28" /><polygon points="1350,14 1344,28 1356,28" />
		<polygon points="1386,14 1380,28 1392,28" /><polygon points="1422,14 1416,28 1428,28" />
	</g>
	<!-- Gold dots at each peak tip -->
	<g fill="#c8960a">
		<circle cx="0" cy="0" r="3" /><circle cx="36" cy="0" r="3" />
		<circle cx="72" cy="0" r="3" /><circle cx="108" cy="0" r="3" />
		<circle cx="144" cy="0" r="3" /><circle cx="180" cy="0" r="3" />
		<circle cx="216" cy="0" r="3" /><circle cx="252" cy="0" r="3" />
		<circle cx="288" cy="0" r="3" /><circle cx="324" cy="0" r="3" />
		<circle cx="360" cy="0" r="3" /><circle cx="396" cy="0" r="3" />
		<circle cx="432" cy="0" r="3" /><circle cx="468" cy="0" r="3" />
		<circle cx="504" cy="0" r="3" /><circle cx="540" cy="0" r="3" />
		<circle cx="576" cy="0" r="3" /><circle cx="612" cy="0" r="3" />
		<circle cx="648" cy="0" r="3" /><circle cx="684" cy="0" r="3" />
		<circle cx="720" cy="0" r="3" /><circle cx="756" cy="0" r="3" />
		<circle cx="792" cy="0" r="3" /><circle cx="828" cy="0" r="3" />
		<circle cx="864" cy="0" r="3" /><circle cx="900" cy="0" r="3" />
		<circle cx="936" cy="0" r="3" /><circle cx="972" cy="0" r="3" />
		<circle cx="1008" cy="0" r="3" /><circle cx="1044" cy="0" r="3" />
		<circle cx="1080" cy="0" r="3" /><circle cx="1116" cy="0" r="3" />
		<circle cx="1152" cy="0" r="3" /><circle cx="1188" cy="0" r="3" />
		<circle cx="1224" cy="0" r="3" /><circle cx="1260" cy="0" r="3" />
		<circle cx="1296" cy="0" r="3" /><circle cx="1332" cy="0" r="3" />
		<circle cx="1368" cy="0" r="3" /><circle cx="1404" cy="0" r="3" />
		<circle cx="1440" cy="0" r="3" />
	</g>
</svg>

<!-- MAIN CONTENT
        Bootstrap container-fluid + responsive column grid.
        Text narrows toward a comfortable reading measure on large screens
        and expands to full width on phones. -->
<section class="banner">
	<div class="container-fluid px-0">
		<div class="row justify-content-center mx-0">
			<div class="col-12 col-sm-10 col-md-8 col-lg-6 col-xl-5 px-4 px-md-3">
				<!-- LEDE SENTENCE
                Replace this placeholder with your actual intro sentence.
                Keep it to 1–2 sentences max — this is a hook, not a summary. -->
				<p class="lede">
					When money determines who lives and dies in rural Mexico's healthcare system.
				</p>

				<!-- DIVIDER: thin gold line with center diamond,
                echoing the Cora diamond motif from the hero -->
				<div class="divider" aria-hidden="true">
					<div class="divider-line"></div>
					<div class="divider-diamond"></div>
					<div class="divider-line"></div>
				</div>

				<!-- BYLINE: name + date -->
				<div class="byline">
					<div class="byline-name">By Andres Jimenez Larios</div>
					<div class="byline-date">
						April 2025 &nbsp;·&nbsp; University of California, Berkeley Graduate School of
						Journalism
					</div>
				</div>
			</div>
		</div>
	</div>
</section>

<!-- God's eye BOTTOM border — vertically mirrored version of the top -->
<svg
	class="gods-eye-border"
	viewBox="0 0 1440 28"
	preserveAspectRatio="none"
	xmlns="http://www.w3.org/2000/svg"
	aria-hidden="true"
>
	<rect width="1440" height="28" fill="#2a0e58" />
	<polyline
		points="0,28 18,14 36,28 54,14 72,28 90,14 108,28 126,14 144,28 162,14 180,28
               198,14 216,28 234,14 252,28 270,14 288,28 306,14 324,28 342,14 360,28
               378,14 396,28 414,14 432,28 450,14 468,28 486,14 504,28 522,14 540,28
               558,14 576,28 594,14 612,28 630,14 648,28 666,14 684,28 702,14 720,28
               738,14 756,28 774,14 792,28 810,14 828,28 846,14 864,28 882,14 900,28
               918,14 936,28 954,14 972,28 990,14 1008,28 1026,14 1044,28 1062,14
               1080,28 1098,14 1116,28 1134,14 1152,28 1170,14 1188,28 1206,14 1224,28
               1242,14 1260,28 1278,14 1296,28 1314,14 1332,28 1350,14 1368,28 1386,14
               1404,28 1422,14 1440,28"
		fill="none"
		stroke="#c8960a"
		stroke-width="2.5"
	/>
	<g fill="#c04040">
		<polygon points="18,14 12,0 24,0" /><polygon points="54,14 48,0 60,0" />
		<polygon points="90,14 84,0 96,0" /><polygon points="126,14 120,0 132,0" />
		<polygon points="162,14 156,0 168,0" /><polygon points="198,14 192,0 204,0" />
		<polygon points="234,14 228,0 240,0" /><polygon points="270,14 264,0 276,0" />
		<polygon points="306,14 300,0 312,0" /><polygon points="342,14 336,0 348,0" />
		<polygon points="378,14 372,0 384,0" /><polygon points="414,14 408,0 420,0" />
		<polygon points="450,14 444,0 456,0" /><polygon points="486,14 480,0 492,0" />
		<polygon points="522,14 516,0 528,0" /><polygon points="558,14 552,0 564,0" />
		<polygon points="594,14 588,0 600,0" /><polygon points="630,14 624,0 636,0" />
		<polygon points="666,14 660,0 672,0" /><polygon points="702,14 696,0 708,0" />
		<polygon points="738,14 732,0 744,0" /><polygon points="774,14 768,0 780,0" />
		<polygon points="810,14 804,0 816,0" /><polygon points="846,14 840,0 852,0" />
		<polygon points="882,14 876,0 888,0" /><polygon points="918,14 912,0 924,0" />
		<polygon points="954,14 948,0 960,0" /><polygon points="990,14 984,0 996,0" />
		<polygon points="1026,14 1020,0 1032,0" /><polygon points="1062,14 1056,0 1068,0" />
		<polygon points="1098,14 1092,0 1104,0" /><polygon points="1134,14 1128,0 1140,0" />
		<polygon points="1170,14 1164,0 1176,0" /><polygon points="1206,14 1200,0 1212,0" />
		<polygon points="1242,14 1236,0 1248,0" /><polygon points="1278,14 1272,0 1284,0" />
		<polygon points="1314,14 1308,0 1320,0" /><polygon points="1350,14 1344,0 1356,0" />
		<polygon points="1386,14 1380,0 1392,0" /><polygon points="1422,14 1416,0 1428,0" />
	</g>
	<g fill="#c8960a">
		<circle cx="0" cy="28" r="3" /><circle cx="36" cy="28" r="3" />
		<circle cx="72" cy="28" r="3" /><circle cx="108" cy="28" r="3" />
		<circle cx="144" cy="28" r="3" /><circle cx="180" cy="28" r="3" />
		<circle cx="216" cy="28" r="3" /><circle cx="252" cy="28" r="3" />
		<circle cx="288" cy="28" r="3" /><circle cx="324" cy="28" r="3" />
		<circle cx="360" cy="28" r="3" /><circle cx="396" cy="28" r="3" />
		<circle cx="432" cy="28" r="3" /><circle cx="468" cy="28" r="3" />
		<circle cx="504" cy="28" r="3" /><circle cx="540" cy="28" r="3" />
		<circle cx="576" cy="28" r="3" /><circle cx="612" cy="28" r="3" />
		<circle cx="648" cy="28" r="3" /><circle cx="684" cy="28" r="3" />
		<circle cx="720" cy="28" r="3" /><circle cx="756" cy="28" r="3" />
		<circle cx="792" cy="28" r="3" /><circle cx="828" cy="28" r="3" />
		<circle cx="864" cy="28" r="3" /><circle cx="900" cy="28" r="3" />
		<circle cx="936" cy="28" r="3" /><circle cx="972" cy="28" r="3" />
		<circle cx="1008" cy="28" r="3" /><circle cx="1044" cy="28" r="3" />
		<circle cx="1080" cy="28" r="3" /><circle cx="1116" cy="28" r="3" />
		<circle cx="1152" cy="28" r="3" /><circle cx="1188" cy="28" r="3" />
		<circle cx="1224" cy="28" r="3" /><circle cx="1260" cy="28" r="3" />
		<circle cx="1296" cy="28" r="3" /><circle cx="1332" cy="28" r="3" />
		<circle cx="1368" cy="28" r="3" /><circle cx="1404" cy="28" r="3" />
		<circle cx="1440" cy="28" r="3" />
	</g>
</svg>

<style>
	/* ── BANNER BACKGROUND ──
        Change --banner-bg here to adjust the color in VS Code.
        Currently a warm cream — light enough to contrast the dark hero. */
	.banner {
		--banner-bg: #f5f0e8; /* ← easy to change: try #fff8f0, #fdf6ee, #ede8e0 */
		background: var(--banner-bg);
		padding: 3.5rem 0 3rem;
		position: relative;
		z-index: 10; /* sits above the sticky hero when scrolling */
	}

	/* ── GOD'S EYE BORDERS ──
        Full-width SVG strips that stretch to any screen width.
        preserveAspectRatio="none" on the SVG handles the stretching.
        display:block removes the default inline bottom gap. */
	.gods-eye-border {
		display: block;
		width: 100%;
		height: 28px; /* increase for a taller/thicker border */
	}

	/* ── LEDE SENTENCE ──
        Crimson Text italic — same family as the hero headline.
        clamp() keeps it readable from 320px phones to 4K monitors. */
	.lede {
		font-family: 'Crimson Text', Georgia, 'Times New Roman', serif;
		font-size: clamp(1.2rem, 2.8vw, 1.7rem);
		line-height: 1.7;
		color: #2a1a0e; /* deep warm brown — readable on cream */
		font-style: italic;
		text-align: center;
		margin-bottom: 0;
	}

	/* ── GOLD DIVIDER ──
        Thin line with a rotated center diamond.
        Mirrors the Cora diamond motif used in the hero. */
	.divider {
		display: flex;
		align-items: center;
		gap: 12px;
		margin: 1.8rem auto;
		max-width: 280px;
	}
	.divider-line {
		flex: 1;
		height: 1px;
		background: #c8960a;
		opacity: 0.5;
	}
	.divider-diamond {
		width: 8px;
		height: 8px;
		background: #c8960a;
		transform: rotate(45deg);
		flex-shrink: 0;
	}

	/* ── BYLINE ──
        Name in small-caps gold, date in muted italic below. */
	.byline {
		text-align: center;
		font-family: 'Crimson Text', Georgia, 'Times New Roman', serif;
	}
	.byline-name {
		font-size: clamp(0.9rem, 1.8vw, 1.05rem);
		letter-spacing: 0.16em;
		color: #8a6010; /* darker gold — readable on cream */
		font-weight: 600;
		text-transform: uppercase;
	}
	.byline-date {
		font-size: clamp(0.8rem, 1.5vw, 0.9rem);
		color: #a08060; /* muted warm brown */
		letter-spacing: 0.08em;
		margin-top: 5px;
		font-style: italic;
	}

	/* ── RESPONSIVE NOTES ──
        Layout is handled by Bootstrap 5 column classes on the div:
          col-12        → full width on phones (≤576px)
          col-sm-10     → 83% on small tablets (≥576px)
          col-md-8      → 67% on tablets (≥768px)
          col-lg-6      → 50% on laptop (≥992px)
          col-xl-5      → 42% on desktop (≥1200px)
        No media queries needed here — Bootstrap handles it.
        Only add CSS overrides below if you need to tweak
        something Bootstrap doesn't control (like font sizes). */
</style>
