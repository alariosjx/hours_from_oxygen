<!--
  SectionBreak.svelte
  Seven concentric-diamond god's-eye motifs — purple / gold / purple / red —
  floating unconnected across a horizontal strip.
  Drop as [[SectionBreak]] in the Google Doc.
-->

<div class="section-break" aria-hidden="true">
	<svg
		class="gods-eye-rule"
		viewBox="0 0 670 40"
		xmlns="http://www.w3.org/2000/svg"
		aria-hidden="true"
	>
		<!-- x=50 -->
		<polygon points="50,7 63,20 50,33 37,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="50,10 60,20 50,30 40,20" fill="#c8960a" />
		<polygon points="50,13 57,20 50,27 43,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="50,16 54,20 50,24 46,20" fill="#c04040" />

		<!-- x=145 -->
		<polygon points="145,7 158,20 145,33 132,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="145,10 155,20 145,30 135,20" fill="#c8960a" />
		<polygon points="145,13 152,20 145,27 138,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="145,16 149,20 145,24 141,20" fill="#c04040" />

		<!-- x=240 -->
		<polygon points="240,7 253,20 240,33 227,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="240,10 250,20 240,30 230,20" fill="#c8960a" />
		<polygon points="240,13 247,20 240,27 233,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="240,16 244,20 240,24 236,20" fill="#c04040" />

		<!-- x=335 — center -->
		<polygon points="335,7 348,20 335,33 322,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="335,10 345,20 335,30 325,20" fill="#c8960a" />
		<polygon points="335,13 342,20 335,27 328,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="335,16 339,20 335,24 331,20" fill="#c04040" />

		<!-- x=430 -->
		<polygon points="430,7 443,20 430,33 417,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="430,10 440,20 430,30 420,20" fill="#c8960a" />
		<polygon points="430,13 437,20 430,27 423,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="430,16 434,20 430,24 426,20" fill="#c04040" />

		<!-- x=525 -->
		<polygon points="525,7 538,20 525,33 512,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="525,10 535,20 525,30 515,20" fill="#c8960a" />
		<polygon points="525,13 532,20 525,27 518,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="525,16 529,20 525,24 521,20" fill="#c04040" />

		<!-- x=620 -->
		<polygon points="620,7 633,20 620,33 607,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="620,10 630,20 620,30 610,20" fill="#c8960a" />
		<polygon points="620,13 627,20 620,27 613,20" fill="#2a0e58" opacity="0.85" />
		<polygon points="620,16 624,20 620,24 616,20" fill="#c04040" />
	</svg>
</div>

<style>
	.section-break {
		display: flex;
		justify-content: center;
		padding: 3.5rem 0;
	}

	.gods-eye-rule {
		width: min(640px, 90vw);
		height: auto;
	}
</style>
