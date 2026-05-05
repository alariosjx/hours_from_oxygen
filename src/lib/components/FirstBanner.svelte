<!--
  FirstBanner.svelte
  Sticky 200vh hold. Single vertical god's-eye border centered on the page.
  No rotation — SVG is natively vertical (viewBox="0 0 28 1440").
-->

<div class="sticky-wrapper">
	<div class="sticky-inner">
		<!-- Vertical god's eye border — native vertical points, centered absolutely -->
		<svg
			class="gods-eye-border"
			viewBox="0 0 28 1440"
			preserveAspectRatio="none"
			xmlns="http://www.w3.org/2000/svg"
			aria-hidden="true"
		>
			<rect width="28" height="1440" fill="#2a0e58" />
			<polyline
				points="28,0 14,18 28,36 14,54 28,72 14,90 28,108 14,126 28,144 14,162 28,180 14,198
				  28,216 14,234 28,252 14,270 28,288 14,306 28,324 14,342 28,360 14,378 28,396
				  14,414 28,432 14,450 28,468 14,486 28,504 14,522 28,540 14,558 28,576 14,594
				  28,612 14,630 28,648 14,666 28,684 14,702 28,720 14,738 28,756 14,774 28,792
				  14,810 28,828 14,846 28,864 14,882 28,900 14,918 28,936 14,954 28,972 14,990
				  28,1008 14,1026 28,1044 14,1062 28,1080 14,1098 28,1116 14,1134 28,1152 14,1170
				  28,1188 14,1206 28,1224 14,1242 28,1260 14,1278 28,1296 14,1314 28,1332 14,1350
				  28,1368 14,1386 28,1404 14,1422 28,1440"
				fill="none"
				stroke="#c8960a"
				stroke-width="2.5"
			/>
			<g fill="#c04040">
				<polygon points="14,18 0,12 0,24" /><polygon points="14,54 0,48 0,60" />
				<polygon points="14,90 0,84 0,96" /><polygon points="14,126 0,120 0,132" />
				<polygon points="14,162 0,156 0,168" /><polygon points="14,198 0,192 0,204" />
				<polygon points="14,234 0,228 0,240" /><polygon points="14,270 0,264 0,276" />
				<polygon points="14,306 0,300 0,312" /><polygon points="14,342 0,336 0,348" />
				<polygon points="14,378 0,372 0,384" /><polygon points="14,414 0,408 0,420" />
				<polygon points="14,450 0,444 0,456" /><polygon points="14,486 0,480 0,492" />
				<polygon points="14,522 0,516 0,528" /><polygon points="14,558 0,552 0,564" />
				<polygon points="14,594 0,588 0,600" /><polygon points="14,630 0,624 0,636" />
				<polygon points="14,666 0,660 0,672" /><polygon points="14,702 0,696 0,708" />
				<polygon points="14,738 0,732 0,744" /><polygon points="14,774 0,768 0,780" />
				<polygon points="14,810 0,804 0,816" /><polygon points="14,846 0,840 0,852" />
				<polygon points="14,882 0,876 0,888" /><polygon points="14,918 0,912 0,924" />
				<polygon points="14,954 0,948 0,960" /><polygon points="14,990 0,984 0,996" />
				<polygon points="14,1026 0,1020 0,1032" /><polygon points="14,1062 0,1056 0,1068" />
				<polygon points="14,1098 0,1092 0,1104" /><polygon points="14,1134 0,1128 0,1140" />
				<polygon points="14,1170 0,1164 0,1176" /><polygon points="14,1206 0,1200 0,1212" />
				<polygon points="14,1242 0,1236 0,1248" /><polygon points="14,1278 0,1272 0,1284" />
				<polygon points="14,1314 0,1308 0,1320" /><polygon points="14,1350 0,1344 0,1356" />
				<polygon points="14,1386 0,1380 0,1392" /><polygon points="14,1422 0,1416 0,1428" />
			</g>
			<g fill="#c8960a">
				<circle cx="28" cy="0" r="3" /><circle cx="28" cy="36" r="3" />
				<circle cx="28" cy="72" r="3" /><circle cx="28" cy="108" r="3" />
				<circle cx="28" cy="144" r="3" /><circle cx="28" cy="180" r="3" />
				<circle cx="28" cy="216" r="3" /><circle cx="28" cy="252" r="3" />
				<circle cx="28" cy="288" r="3" /><circle cx="28" cy="324" r="3" />
				<circle cx="28" cy="360" r="3" /><circle cx="28" cy="396" r="3" />
				<circle cx="28" cy="432" r="3" /><circle cx="28" cy="468" r="3" />
				<circle cx="28" cy="504" r="3" /><circle cx="28" cy="540" r="3" />
				<circle cx="28" cy="576" r="3" /><circle cx="28" cy="612" r="3" />
				<circle cx="28" cy="648" r="3" /><circle cx="28" cy="684" r="3" />
				<circle cx="28" cy="720" r="3" /><circle cx="28" cy="756" r="3" />
				<circle cx="28" cy="792" r="3" /><circle cx="28" cy="828" r="3" />
				<circle cx="28" cy="864" r="3" /><circle cx="28" cy="900" r="3" />
				<circle cx="28" cy="936" r="3" /><circle cx="28" cy="972" r="3" />
				<circle cx="28" cy="1008" r="3" /><circle cx="28" cy="1044" r="3" />
				<circle cx="28" cy="1080" r="3" /><circle cx="28" cy="1116" r="3" />
				<circle cx="28" cy="1152" r="3" /><circle cx="28" cy="1188" r="3" />
				<circle cx="28" cy="1224" r="3" /><circle cx="28" cy="1260" r="3" />
				<circle cx="28" cy="1296" r="3" /><circle cx="28" cy="1332" r="3" />
				<circle cx="28" cy="1368" r="3" /><circle cx="28" cy="1404" r="3" />
				<circle cx="28" cy="1440" r="3" />
			</g>
		</svg>

		<!-- MAIN CONTENT -->
		<section class="banner">
			<div class="container-fluid px-0">
				<div class="row justify-content-center mx-0">
					<div class="col-12 col-sm-10 col-md-8 col-lg-6 col-xl-5 px-4 px-md-3">
						<p class="lede">
							When money determines who lives and dies in rural Mexico's healthcare system.
						</p>
						<div class="divider" aria-hidden="true">
							<div class="divider-line"></div>
							<div class="divider-diamond"></div>
							<div class="divider-line"></div>
						</div>
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
	</div>
</div>

<style>
	.sticky-wrapper {
		height: 200vh;
		position: relative;
	}

	.sticky-inner {
		position: sticky;
		top: 0;
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		background: #f5f0e8;
		overflow: hidden;
	}

	.gods-eye-border {
		display: block;
		width: 28px;
		height: 100vh;
		position: absolute;
		top: 0;
		left: 50%;
		margin-left: -14px; /* center: half of 28px */
		/* No transform, no rotation */
	}

	.banner {
		--banner-bg: #f5f0e8;
		background: var(--banner-bg);
		padding: 3.5rem 0 3rem;
		position: relative;
		z-index: 10;
	}

	.lede {
		font-family: 'Crimson Text', Georgia, 'Times New Roman', serif;
		font-size: clamp(1.2rem, 2.8vw, 1.7rem);
		line-height: 1.7;
		color: #2a1a0e;
		font-style: italic;
		text-align: center;
		margin-bottom: 0;
	}

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

	.byline {
		text-align: center;
		font-family: 'Crimson Text', Georgia, 'Times New Roman', serif;
	}
	.byline-name {
		font-size: clamp(0.9rem, 1.8vw, 1.05rem);
		letter-spacing: 0.16em;
		color: #8a6010;
		font-weight: 600;
		text-transform: uppercase;
	}
	.byline-date {
		font-size: clamp(0.8rem, 1.5vw, 0.9rem);
		color: #a08060;
		letter-spacing: 0.08em;
		margin-top: 5px;
		font-style: italic;
	}
</style>
