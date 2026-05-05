<!-- src/lib/components/ImageEmbed.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import { base } from '$app/paths';

	export let src: string | undefined;
	export let alt: string | undefined;
	export let caption: string | undefined;
	export let size: 'full' | 'large' | 'fit' = 'large';
	export let bordered: boolean = false;
	export let maxWidth: string | undefined = undefined;

	let embeddedSrc: string | null = null;
	let embeddedAlt: string | null = null;

	function normalizeStaticPath(raw: string) {
		const s = raw.trim();
		if (/^data:/i.test(s) || /^https?:\/\//i.test(s)) return s; // Absolute URLs or data URIs
		if (s.startsWith('/')) return base + s; // Prepend `base` to paths starting with `/`
		return base + '/' + s; // Prepend `base` to other relative paths
	}

	let hostEl: HTMLElement | null = null;

	onMount(() => {
		if (src) return;

		const el = hostEl;
		if (!el) return;

		let n: Node | null = el.nextSibling;

		while (n) {
			if (n.nodeType === 1) {
				const elem = n as Element;

				if (elem.tagName.toLowerCase() === 'img') {
					embeddedSrc = (elem as HTMLImageElement).getAttribute('src');
					embeddedAlt = (elem as HTMLImageElement).getAttribute('alt');
					elem.remove();
					break;
				}

				if (elem.tagName.toLowerCase() === 'figure') {
					const img = elem.querySelector('img');
					if (img) {
						embeddedSrc = img.getAttribute('src');
						embeddedAlt = img.getAttribute('alt');
						elem.remove();
						break;
					}
				}
			}
			n = n.nextSibling;
		}
	});

	$: finalSrc = src ? normalizeStaticPath(src) : embeddedSrc;
	$: finalAlt = (alt && alt.trim().length ? alt : embeddedAlt) ?? '';
	$: shouldRender = !!(finalSrc && finalSrc.trim().length);
</script>

<!-- anchor node so we can locate the DOM position -->
<span bind:this={hostEl} style="display:none"></span>

{#if shouldRender}
	{#if size === 'full'}
		<figure class="my-3 full-bleed">
			<img
				src={finalSrc}
				alt={finalAlt}
				class="img-fluid"
				class:gods-eye={bordered}
				style={maxWidth ? `max-width:${maxWidth};display:block;margin:0 auto` : ''}
			/>
			{#if caption}
				<figcaption class="mt-2 text-muted small">{caption}</figcaption>
			{/if}
		</figure>
	{:else if size === 'large'}
		<!-- svelte-ignore a11y_figcaption_parent -->
		<figure class="my-3 full-bleed">
			<div class="container-fluid">
				<div class="row justify-content-center">
					<div class="col-12 col-lg-10 col-xxl-8">
						<img
							src={finalSrc}
							alt={finalAlt}
							class="img-fluid"
							class:gods-eye={bordered}
							style={maxWidth ? `max-width:${maxWidth};display:block;margin:0 auto` : ''}
						/>
						{#if caption}
							<figcaption class="mt-2 text-muted small">{caption}</figcaption>
						{/if}
					</div>
				</div>
			</div>
		</figure>
	{:else}
		<!-- fit -->
		<figure class="my-3">
			<img
				src={finalSrc}
				alt={finalAlt}
				class="img-fluid"
				class:gods-eye={bordered}
				style={maxWidth ? `max-width:${maxWidth};display:block;margin:0 auto` : ''}
			/>
			{#if caption}
				<figcaption class="mt-2 text-muted small">{caption}</figcaption>
			{/if}
		</figure>
	{/if}
{/if}

<style>
	.gods-eye {
		border: 2px solid #c8960a;
		box-shadow:
			0 0 0 5px #2a0e58,
			0 0 0 7px #c8960a;
		border-radius: 2px;
	}
</style>
