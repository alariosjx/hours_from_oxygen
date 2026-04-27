<script lang="ts">
	import rawBlocks from '$lib/doc.blocks.json';
	import DocRenderer from '$lib/components/DocRenderer.svelte';
	import type { Block } from '$lib/components/DocRenderer.svelte';
	// Animated lo-fi landing hero — edit in src/lib/components/LandingHero.svelte
	import LandingHero from '$lib/components/LandingHero.svelte';
	import Firstbanner from '$lib/components/FirstBanner.svelte';
	import CovidTimeline from '$lib/components/CovidTimeline.svelte';
	import Geography from '$lib/components/Geography.svelte';

	const blocks = rawBlocks as Block[];
</script>

<!-- Full-viewport animated hero section -->
<LandingHero />
<section class="content-wrapper">
	<Firstbanner />
</section>
<CovidTimeline />
<Geography />

<!-- DO NOT EDIT (except to modify classes). This will become code imported from Google Doc. -->
<section class="content-wrapper">
	<div class="container">
		<div class="row justify-content-center">
			<div class="col-12 col-sm-10 col-lg-8 col-xxl-6">
				<DocRenderer {blocks} />
			</div>
		</div>
	</div>
</section>

<!-- Hard-code any custom code that should appear AFTER Google Doc below here. -->
<footer class="container-fluid bg-dark text-white p-5">
	<p class="text-center">
		See template on <a
			class="text-white"
			href="https://github.com/alariosjx/hours-from-oxygen"
			target="_blank">Github</a
		>
	</p>
</footer>

<style>
	.content-wrapper {
		position: relative;
		z-index: 1;
		background: #f5f0e8;
		padding: 2rem 0;
	}
</style>
