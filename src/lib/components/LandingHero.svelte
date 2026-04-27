<!--
  ============================================================
  LandingHero.svelte
  Fullscreen lo-fi animated landing header for "El Viaje Eterno"

  WHAT THIS COMPONENT DOES:
  - Fills the full viewport height on desktop (100vh)
  - On mobile (≤640px) shrinks to a compact 260px strip showing
    only the car and mountains — headline/arrows hidden
  - Pure CSS animations — no JS libraries needed
  - Sky: layered dusk gradient (indigo → purple → magenta → burnt orange)
  - Stars: fixed-position dots with staggered twinkle animations
  - Moon: crescent made from two overlapping circles
  - Mountains: 4-layer SVG silhouettes with cloud wisps at the peak
  - Road: single div animated via background-position (no flicker)
  - Car: grey/silver boxy old SUV SVG driving left to right on loop
  - Headline: fades in on load with Cora-inspired diamond pattern
  - Scroll indicator: bouncing gold chevron arrows at the bottom

  HOW TO USE:
  1. Copy this file to src/lib/components/LandingHero.svelte
  2. In your +page.svelte, import and use it:
       <script>
         import LandingHero from '$lib/components/LandingHero.svelte';
       </script>
       <LandingHero />
  3. The component uses Crimson Text from your existing fonts.css.
     Make sure static/fonts/fonts.css is loaded in your app.html or app.scss.

  CUSTOMIZATION:
  - To swap in a real video background later, replace the .sky div
    with a <video autoplay muted loop playsinline> element
  - Car speed: change animation-duration on .car-wrap (currently 10s)
  - Road dash speed: change animation-duration on .road-dashes (currently 1.4s)
  - Title/subtitle: edit the <h1> and .translation text below
  ============================================================
-->

<script>
	// No JS needed for the animations — everything is pure CSS.
	// This script block is here for future use (e.g. swapping in
	// a real video src once you have the footage ready).
</script>

<!-- ── HERO SECTION ── fills full viewport height on desktop -->
<section class="hero" aria-label="Animated dusk landscape of Nayarit, Mexico">
	<!-- SKY: layered dusk gradient from deep indigo at top to
         magenta/burnt-orange at the horizon -->
	<div class="sky" aria-hidden="true"></div>

	<!-- MOON: crescent effect — two overlapping circles.
         .moon-b is the same color as the upper sky behind it,
         creating the illusion of a crescent shape. -->
	<div class="moon-a" aria-hidden="true"></div>
	<div class="moon-b" aria-hidden="true"></div>

	<!-- STARS: fixed-position dots with varying twinkle speeds.
         Each star uses CSS custom property --d for its twinkle duration,
         and an inline animation-delay to stagger them so they don't
         all pulse at the same time.
         Sizes alternate between 1px and 2px for depth variation. -->
	<div
		class="star"
		aria-hidden="true"
		style="top:3%;left:6%;width:2px;height:2px;--d:3.2s;animation-delay:.1s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:7%;left:12%;width:1px;height:1px;--d:4.1s;animation-delay:1s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:2%;left:21%;width:2px;height:2px;--d:2.9s;animation-delay:.6s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:10%;left:29%;width:1px;height:1px;--d:3.7s;animation-delay:1.9s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:5%;left:38%;width:2px;height:2px;--d:3s;animation-delay:.4s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:1%;left:50%;width:1px;height:1px;--d:4.3s;animation-delay:2s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:8%;left:58%;width:2px;height:2px;--d:2.7s;animation-delay:.8s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:4%;left:67%;width:1px;height:1px;--d:3.9s;animation-delay:2.4s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:12%;left:75%;width:2px;height:2px;--d:3.4s;animation-delay:.2s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:6%;left:84%;width:1px;height:1px;--d:4.6s;animation-delay:1.5s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:14%;left:92%;width:2px;height:2px;--d:3s;animation-delay:.9s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:16%;left:17%;width:1px;height:1px;--d:3.1s;animation-delay:2.9s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:18%;left:44%;width:2px;height:2px;--d:2.8s;animation-delay:2.1s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:13%;left:63%;width:1px;height:1px;--d:3.6s;animation-delay:.5s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:19%;left:80%;width:2px;height:2px;--d:3.3s;animation-delay:1.6s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:21%;left:4%;width:1px;height:1px;--d:2.6s;animation-delay:2.7s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:23%;left:34%;width:2px;height:2px;--d:4s;animation-delay:.3s"
	></div>
	<div
		class="star"
		aria-hidden="true"
		style="top:25%;left:70%;width:1px;height:1px;--d:3.5s;animation-delay:1.2s"
	></div>

	<!-- MOUNTAINS: 4 SVG path layers at increasing opacity/darkness,
         simulating atmospheric depth (like the Sierra Madre in your photo).
         Layer 1 = farthest/haziest  →  Layer 4 = nearest/darkest.
         Cloud wisps sit just above the main peak center-right.
         preserveAspectRatio="xMidYMax meet" anchors the mountain bases
         to the bottom of the container as the viewport scales. -->
	<div class="mtns" aria-hidden="true">
		<!--
			preserveAspectRatio="none" stretches the SVG to exactly fill the
			.mtns container at every screen size, so the mountain silhouettes
			always span edge-to-edge. Slight vertical distortion on non-16:9
			screens is invisible on organic shapes like hills.
		-->
		<svg
			viewBox="0 0 1440 400"
			preserveAspectRatio="none"
			style="width:100%;height:100%;display:block"
			xmlns="http://www.w3.org/2000/svg"
		>
			<!-- Layer 1: farthest ridge — lightest, most transparent -->
			<path
				d="M0,400 L0,230
             Q100,170 210,192 Q330,215 430,155
             Q492,120 545,138 Q588,105 645,115
             Q695,125 745,92 Q795,62 852,82
             Q912,102 992,128 Q1072,152 1152,118
             Q1232,86 1330,110 Q1400,128 1440,115
             L1440,400 Z"
				fill="#4a2080"
				opacity=".6"
			/>
			<!-- Layer 2: second ridge, slightly darker and lower -->
			<path
				d="M0,400 L0,272
             Q85,230 185,248 Q308,268 415,208
             Q488,168 548,186 Q605,155 660,165
             Q714,142 768,155 Q828,170 910,194
             Q992,218 1072,185 Q1162,152 1262,178
             Q1372,205 1440,190
             L1440,400 Z"
				fill="#361660"
				opacity=".78"
			/>
			<!-- Layer 3: main peak mass — tallest, most defined ridge -->
			<path
				d="M0,400 L0,315
             Q62,288 142,298 Q225,308 308,270
             Q375,242 428,254 Q478,222 530,232
             Q580,200 628,210 Q675,182 722,192
             Q768,202 835,220 Q908,240 985,260
             Q1065,280 1152,248 Q1260,212 1362,238
             Q1420,252 1440,245
             L1440,400 Z"
				fill="#28104a"
				opacity=".92"
			/>
			<!-- Cloud wisps near the main peak — two soft ellipses -->
			<ellipse cx="722" cy="196" rx="65" ry="13" fill="#5a2890" opacity=".40" />
			<ellipse cx="758" cy="186" rx="40" ry="9" fill="#5a2890" opacity=".28" />
			<!-- Layer 4: immediate dark foreground hills -->
			<path
				d="M0,400 L0,358
             Q180,336 360,352 Q540,368 720,348
             Q900,330 1080,350 Q1260,370 1440,356
             L1440,400 Z"
				fill="#1e0c38"
			/>
		</svg>
	</div>

	<!-- ROAD: a single div with a repeating CSS gradient stripe pattern.
         Animating background-position-x (not DOM elements) means the
         dashes scroll smoothly with zero flicker.
         Gold border lines top and bottom echo the Cora color palette. -->
	<div class="road" aria-hidden="true">
		<div class="road-dashes"></div>
	</div>

	<!-- GROUND: thin dark strip at the very bottom below the road -->
	<div class="ground" aria-hidden="true"></div>

	<!-- CAR: lo-fi SVG of a grey/silver boxy old SUV (the real Ford Explorer
         driven that night by the Jimenez sisters).
         .car-wrap handles the horizontal drive animation.
         Details include: roof rack, dark windows, worn door panels,
         dark wheels with hub detail, faint red taillights. -->
	<div class="car-wrap" aria-hidden="true">
		<svg width="120" height="42" viewBox="0 0 150 54" xmlns="http://www.w3.org/2000/svg">
			<!-- Ground shadow -->
			<ellipse cx="75" cy="52" rx="58" ry="4" fill="#000" opacity=".3" />
			<!-- Main body -->
			<rect x="4" y="23" width="140" height="22" rx="3" fill="#585860" />
			<!-- Boxy SUV cab/roof -->
			<path d="M26,23 L34,7 L110,7 L122,23 Z" fill="#4c4c54" />
			<!-- Roof rack -->
			<rect x="36" y="6" width="72" height="3" rx="1" fill="#38383e" />
			<!-- Windows — dark night glass -->
			<rect x="36" y="9" width="30" height="12" rx="2" fill="#0c1420" opacity=".9" />
			<rect x="70" y="9" width="30" height="12" rx="2" fill="#0c1420" opacity=".9" />
			<!-- Subtle window reflections -->
			<rect x="38" y="10" width="7" height="4" rx="1" fill="#3a5068" opacity=".5" />
			<rect x="72" y="10" width="7" height="4" rx="1" fill="#3a5068" opacity=".5" />
			<!-- Door seam -->
			<line x1="68" y1="24" x2="68" y2="44" stroke="#404048" stroke-width="1" />
			<!-- Age/wear marks on body — it's a 25-year-old truck -->
			<rect x="30" y="34" width="22" height="2" rx="1" fill="#424244" opacity=".7" />
			<rect x="88" y="30" width="14" height="2" rx="1" fill="#424244" opacity=".55" />
			<!-- Front wheel -->
			<circle cx="36" cy="45" r="9" fill="#22222a" />
			<circle cx="36" cy="45" r="5.5" fill="#363640" />
			<circle cx="36" cy="45" r="2" fill="#545458" />
			<!-- Rear wheel -->
			<circle cx="112" cy="45" r="9" fill="#22222a" />
			<circle cx="112" cy="45" r="5.5" fill="#363640" />
			<circle cx="112" cy="45" r="2" fill="#545458" />
			<!-- Taillights — faint red glow, driving into darkness -->
			<rect x="4" y="26" width="6" height="5" rx="1" fill="#660000" opacity=".7" />
			<rect x="4" y="33" width="6" height="4" rx="1" fill="#440000" opacity=".5" />
			<!-- Bumpers -->
			<rect x="4" y="38" width="14" height="4" rx="1" fill="#323238" />
			<rect x="134" y="38" width="12" height="4" rx="1" fill="#323238" />
		</svg>
	</div>

	<!-- HEADLINE: fades in 2.6s after load. Uses Crimson Text serif.
         clamp() keeps the font fluid between 32px (mobile) and 62px (desktop).
         The translation gives English readers context without dominating.
         The Cora diamond row references the geometric patterns of the
         indigenous Cora and Huichol peoples of Nayarit. -->
	<div class="headline">
		<h1>El Viaje Eterno</h1>
		<!-- Small italic English translation -->
		<p class="translation">The Eternal Trip</p>
		<!-- Cora-inspired decorative diamond row: gold + terracotta red -->
		<div class="cora-row" aria-hidden="true">
			<div class="cd cd-sm"></div>
			<div class="cd cd-md"></div>
			<div class="cd cd-lg"></div>
			<div class="cd cd-md" style="background:#c8960a"></div>
			<div class="cd cd-lg" style="background:#e05050"></div>
			<div class="cd cd-lg"></div>
			<div class="cd cd-lg" style="background:#e05050"></div>
			<div class="cd cd-md" style="background:#c8960a"></div>
			<div class="cd cd-lg"></div>
			<div class="cd cd-md"></div>
			<div class="cd cd-sm"></div>
		</div>
	</div>

	<!-- SCROLL INDICATOR: two stacked chevron arrows that bob up and down.
         Gold color matches road border and Cora diamonds.
         Hidden on mobile — no room in the compact 260px hero. -->
	<div class="scroll-hint" aria-label="Scroll down to continue">
		<div class="arr1"></div>
		<div class="arr2"></div>
	</div>
</section>

<style>
	/* ── RESET ── scoped to this component only */
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	/* ── HERO CONTAINER ──
       100vh fills the entire first screen on desktop.
       overflow:hidden clips the car as it enters/exits frame edges.
       Background color is a fallback while the gradient paints. */
	.hero {
		width: 100%;
		height: 100vh;
		position: relative;
		overflow: hidden;
		background: #1a0848;
	}

	/* ── SKY ──
       10-stop gradient simulating a dusk sky over Nayarit's Pacific coast.
       Top: deep indigo-navy
       Middle: rich purple → magenta
       Horizon: burnt orange-red glow
       Bottom: dark maroon blending into the mountain silhouettes */
	.sky {
		position: absolute;
		inset: 0;
		background: linear-gradient(
			to bottom,
			#1a0848 0%,
			#2e1268 18%,
			#4a1e88 35%,
			#6a2898 48%,
			#8a3098 58%,
			#a83880 68%,
			#c04060 76%,
			#d05840 83%,
			#b84030 90%,
			#5a1828 96%,
			#2a0c18 100%
		);
	}

	/* ── STARS ──
       Each .star is a tiny rounded square with custom --d duration.
       Twinkle fades between 70% and 10% opacity on loop.
       Position, size, and delay are set via inline styles on each element. */
	.star {
		position: absolute;
		background: #e8e0ff;
		border-radius: 50%;
		animation: tw var(--d, 3s) ease-in-out infinite;
	}
	@keyframes tw {
		0%,
		100% {
			opacity: 0.7;
		}
		50% {
			opacity: 0.1;
		}
	}

	/* ── MOON ──
       .moon-a = bright gold disc
       .moon-b = same color as the upper sky (#4a1e88), positioned
       slightly left and overlapping, masking a crescent out of .moon-a.
       Note: if you change the sky gradient, update moon-b's background
       color to match the sky color at top:36px. */
	.moon-a {
		position: absolute;
		top: 36px;
		right: 150px;
		width: 34px;
		height: 34px;
		border-radius: 50%;
		background: #e8dc80;
		opacity: 0.85;
	}
	.moon-b {
		position: absolute;
		top: 36px;
		right: 138px;
		width: 30px;
		height: 30px;
		border-radius: 50%;
		background: #4a1e88; /* must match sky color at this position */
	}

	/* ── MOUNTAINS ──
       left:0 / right:0 pins the container edge-to-edge.
       clamp() makes the height fluid:
         min 220px  → mobile landscape / very short viewports
         preferred  → 42% of the viewport height (scales with screen)
         max 500px  → prevents mountains from dominating on tall monitors
       bottom: 42px sits the base just above the road strip.
       The SVG inside uses preserveAspectRatio="none" so it stretches
       to fill this container at every width — no side gaps ever. */
	.mtns {
		position: absolute;
		left: 0;
		right: 0;
		bottom: 42px;
		height: clamp(220px, 42vh, 500px);
	}

	/* ── ROAD ──
       Slim dark strip with gold border lines top and bottom.
       Gold color (#c89010) ties into the Cora decorative palette. */
	.road {
		position: absolute;
		bottom: 16px;
		left: 0;
		right: 0;
		height: 28px;
		background: #1a1828;
		border-top: 1.5px solid #c89010;
		border-bottom: 1.5px solid #c89010;
	}

	/* ── ROAD DASHES ──
       repeating-linear-gradient creates the center line pattern:
         0–55px: transparent gap
         55–110px: gold dash
         (repeats every 165px)
       Animating background-position-x by one full tile (-165px)
       creates a seamless infinite scroll — no DOM elements, no flicker.
       will-change: background-position hints to the browser to use
       the GPU compositor for this animation. */
	.road-dashes {
		position: absolute;
		top: 50%;
		left: 0;
		right: 0;
		height: 2px;
		transform: translateY(-50%);
		background: repeating-linear-gradient(
			to right,
			transparent 0px,
			transparent 55px,
			#d8b020 55px,
			#d8b020 110px
		);
		background-size: 165px 2px;
		animation: roadscroll 1.4s linear infinite;
		will-change: background-position;
	}
	@keyframes roadscroll {
		from {
			background-position-x: 0;
		}
		to {
			background-position-x: -165px;
		}
	}

	/* ── GROUND ──
       Thin dark strip below the road, simulating a dark verge. */
	.ground {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
		height: 18px;
		background: #180a20;
	}

	/* ── CAR ──
       .car-wrap moves horizontally from -170px (offscreen left) to 108%
       (offscreen right) over 10 seconds, then loops.
       bottom: 18px places the wheels on the road surface.
       To change speed: adjust the 10s duration on this animation. */
	.car-wrap {
		position: absolute;
		bottom: 18px;
		animation: drive 10s linear infinite;
	}
	@keyframes drive {
		from {
			left: -170px;
		}
		to {
			left: 108%;
		}
	}

	/* ── HEADLINE ──
       Vertically centered at 38% (slightly above true center) to leave
       visual breathing room above the mountains.
       Fades in from slightly below its resting position (opacity 0 → 1,
       translateY -44% → -50%) over 2.6 seconds. */
	.headline {
		position: absolute;
		top: 38%;
		left: 50%;
		transform: translate(-50%, -50%);
		text-align: center;
		z-index: 20;
		animation: fin 2.6s ease forwards;
		opacity: 0;
		width: 90%;
	}
	@keyframes fin {
		from {
			opacity: 0;
			transform: translate(-50%, -44%);
		}
		to {
			opacity: 1;
			transform: translate(-50%, -50%);
		}
	}

	.headline h1 {
		font-size: clamp(32px, 5.5vw, 62px);
		font-weight: 700;
		letter-spacing: 0.07em;
		color: #fff5d8;
		/* Two-layer shadow: sharp drop shadow + wide dark halo.
         Both ensure legibility over the colorful gradient sky. */
		text-shadow:
			0 2px 30px rgba(0, 0, 0, 0.6),
			0 0 60px rgba(0, 0, 0, 0.5);
		font-family: 'Crimson Text', 'Georgia', 'Times New Roman', serif;
		line-height: 1.05;
	}

	/* ── TRANSLATION LINE ──
       Sits directly below the Spanish title.
       Small, italic, muted warm-white at 65% opacity — reads as
       secondary information, not competing with the main title. */
	.translation {
		margin-top: 6px;
		font-size: clamp(12px, 1.6vw, 16px);
		font-style: italic;
		letter-spacing: 0.18em;
		color: rgba(255, 240, 200, 0.65);
		font-family: 'Crimson Text', 'Georgia', 'Times New Roman', serif;
	}

	/* ── CORA DIAMOND ROW ──
       Three diamond sizes create a symmetric decorative motif.
       Inspired by the geometric tile/yarn art of the Cora and Huichol
       peoples indigenous to Nayarit (visible on buildings in the region).
       Each .cd is a square div rotated 45deg to form a diamond shape. */
	.cora-row {
		display: flex;
		gap: 5px;
		justify-content: center;
		margin-top: 12px;
		align-items: center;
	}
	.cd {
		transform: rotate(45deg);
	}
	.cd-lg {
		width: 11px;
		height: 11px;
		background: #c8960a;
	} /* Cora gold */
	.cd-md {
		width: 8px;
		height: 8px;
		background: #e05050;
	} /* terracotta red */
	.cd-sm {
		width: 5px;
		height: 5px;
		background: #c8960a;
		opacity: 0.55;
	} /* faded gold */

	/* ── SCROLL INDICATOR ──
       Two chevron shapes (CSS borders rotated 45deg) stacked and offset.
       The bob animation moves them 6px down then back up over 2.2s.
       Upper arrow is solid gold; lower arrow is 40% opacity for depth. */
	.scroll-hint {
		position: absolute;
		bottom: 4px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 3px;
		z-index: 30;
		animation: bob 2.2s ease-in-out infinite;
	}
	@keyframes bob {
		0%,
		100% {
			transform: translateX(-50%) translateY(0);
		}
		55% {
			transform: translateX(-50%) translateY(6px);
		}
	}
	.arr1 {
		width: 16px;
		height: 16px;
		border-right: 2.5px solid #c8960a;
		border-bottom: 2.5px solid #c8960a;
		transform: rotate(45deg);
	}
	.arr2 {
		width: 11px;
		height: 11px;
		border-right: 2px solid rgba(200, 150, 10, 0.4);
		border-bottom: 2px solid rgba(200, 150, 10, 0.4);
		transform: rotate(45deg);
		margin-top: -9px; /* overlap with arr1 for a stacked chevron look */
	}

	/* ── RESPONSIVE: MEDIUM SCREENS (641px – 991px, e.g. tablets, small laptops) ──
       Hero stays full viewport height. Mountains use clamp() so no change needed.
       Headline font is already fluid via clamp() on h1.
       Moon shifts slightly inward so it doesn't crowd a narrower sky.
       Road and ground stay the same — they're full-width by default. */
	@media (min-width: 641px) and (max-width: 991px) {
		.moon-a {
			top: 28px;
			right: 100px;
			width: 28px;
			height: 28px;
		}
		.moon-b {
			top: 28px;
			right: 90px;
			width: 24px;
			height: 24px;
		}
		/* Scroll hint is visible but moved up a touch on shorter tablet heights */
		.scroll-hint {
			bottom: 8px;
		}
	}

	/* ── RESPONSIVE: MOBILE (max-width 640px) ──
       On phones the hero collapses to 260px — just enough to show the
       car driving across the mountains as a decorative header strip.
       Headline and scroll arrows are hidden since there's no vertical room.
       All absolute positions are recalculated to fit the smaller frame. */
	@media (max-width: 640px) {
		.hero {
			height: 260px;
		}
		.headline {
			display: none;
		}
		.scroll-hint {
			display: none;
		}
		.road {
			height: 20px;
			bottom: 12px;
		}
		.ground {
			height: 14px;
		}
		.mtns {
			/* On mobile the hero is 260px tall, so clamp needs a tighter max */
			bottom: 30px;
			height: clamp(160px, 65%, 240px);
		}
		.car-wrap {
			bottom: 14px;
		}
		.moon-a {
			top: 18px;
			right: 80px;
			width: 24px;
			height: 24px;
		}
		.moon-b {
			top: 18px;
			right: 70px;
			width: 20px;
			height: 20px;
		}
	}
	/* ── RESPONSIVE: MOBILE (max-width 640px) ── */
	@media (max-width: 640px) {
		.hero {
			height: 260px;
		}
		.headline {
			display: block; /* Ensure the headline remains visible */
			top: 50%; /* Center vertically within the smaller hero */
			transform: translate(-50%, -50%);
			width: 90%; /* Allow the headline to shrink width-wise */
			text-align: center;
		}
		.headline h1 {
			font-size: clamp(20px, 4.5vw, 32px); /* Adjust font size for smaller screens */
			line-height: 1.2; /* Slightly tighter line height for better fit */
		}
		.translation {
			font-size: clamp(10px, 1.2vw, 14px); /* Adjust translation font size */
			margin-top: 4px; /* Reduce spacing between title and translation */
		}
		.scroll-hint {
			display: none; /* Hide scroll hint on mobile */
		}
		.road {
			height: 20px;
			bottom: 12px;
		}
		.ground {
			height: 14px;
		}
		.mtns {
			bottom: 30px;
			height: clamp(160px, 65%, 240px);
		}
		.car-wrap {
			bottom: 14px;
		}
		.moon-a {
			top: 18px;
			right: 80px;
			width: 24px;
			height: 24px;
		}
		.moon-b {
			top: 18px;
			right: 70px;
			width: 20px;
			height: 20px;
		}
	}
</style>
