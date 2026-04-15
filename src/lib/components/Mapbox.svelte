<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import 'mapbox-gl/dist/mapbox-gl.css';

  // Default map centers on Berkeley, CA
  export let longitude :number | -122.2585;
  export let latitude: number | 37.8719

  // Sizing wrangling from other components //
	export let size: 'full' | 'large' | 'fit' = 'fit';
	//list allowed sizes
	const allowedSizes = new Set(['full', 'large', 'fit']);
	// Merge into normalizedSize, fallback to fit if they set something else or don't set anything
	$: normalizedSize = allowedSizes.has(String(size)) ? (size as any) : 'fit';

	let map;
	let mapContainer;

	const zoom = 14; // initial zoom level of map
	let initialState = { longitude, latitude, zoom };

	// Fetches token from env. You will need to put your own token in there!
	const token = import.meta.env.VITE_MAPBOX_ACCESS_TOKEN;

	onMount(async () => {
		// Need to load library onmount if not disabling ssr for the whole page
		const mapboxgl = await import('mapbox-gl');

		map = new mapboxgl.Map({
			container: mapContainer,
			accessToken: token,
			style: 'mapbox://styles/mapbox/streets-v11',
			center: [initialState.longitude, initialState.latitude],
			zoom: initialState.zoom
		});
	});

	onDestroy(() => {
		if (map) map.remove();
	});
</script>

{#if normalizedSize === 'full'}
	<figure class="my-3 full-bleed">
		<div bind:this={mapContainer} class="map"></div>
	</figure>
{:else if normalizedSize === 'large'}
	<figure class="my-3 full-bleed">
		<div class="container-fluid">
			<div class="row justify-content-center">
				<div class="col-12 col-lg-10 col-xxl-8">
					<div bind:this={mapContainer} class="map"></div>
				</div>
			</div>
		</div>
	</figure>
{:else}
	<figure class="my-3">
		<div bind:this={mapContainer} class="map"></div>
	</figure>
{/if}

<style>
	.map {
		height: 500px; 
		background-color: #f0f0f0;
	}
</style>
