<script lang="ts">
	import { onDestroy, onMount, tick } from 'svelte';
	import { browser } from '$app/environment';
	import { base } from '$app/paths'; // Import base from $app/paths

	import ScrollyStep from './scrolly/ScrollyStep.svelte';
	import {
		type Step,
		POS_CLASS,
		toNum,
		toMedia,
		tryParseStepsFromBodyHtml
	} from './scrolly/utils.js';

	// ---- Props ----

	export let steps: Step[] | undefined;
	export let bodyHtml: string | undefined;

	/** Minimum height of each step in vh. */
	export let vhPerStep: number | string | undefined = 100;

	/** Crossfade duration in ms. */
	export let fadeMs: number | string | undefined = 600;

	/** How far up from the bottom the text box sits (vh). 0 = bottom edge. */
	export let textOffsetFromBottomVh: number | string | undefined = 0;

	/**
	 * Fraction of the viewport height (0–1) the last text box must reach before
	 * the background un-sticks and scrolls away with the content.
	 */
	export let lastCarryPoint: number | string | undefined = 0.5;

	// ---- Derived numeric values ----

	$: stepHeightVh = toNum(vhPerStep, 100);
	$: fadeDurationMs = toNum(fadeMs, 600);
	$: textBottomVh = toNum(textOffsetFromBottomVh, 0);
	$: carryPoint = Math.max(0, Math.min(1, toNum(lastCarryPoint, 0.5)));

	// ---- Step resolution ----

	$: resolvedSteps =
		(steps && steps.length ? steps : null) ?? tryParseStepsFromBodyHtml(bodyHtml) ?? [];
</script>

{#if resolvedSteps.length}
	<section
		bind:this={sectionEl}
		class="scrolly full-bleed"
		style={`--fade-ms:${fadeDurationMs}ms; --text-bottom:${textBottomVh}vh;`}
	>
		<!-- Sticky background: one layer per step, CSS-driven crossfade via opacity -->
		<div
			class={'scrolly-bg ' + (released ? 'unstick' : '')}
			role="img"
			aria-label={activeAlt}
			style="height: 100vh; width: 100vw;"
		>
			{#each resolvedSteps as step, i (i)}
				{@const m = toMedia(step)}
				<div class="layer" class:active={i === bgIndex} aria-hidden="true">
					{#if loadedIndices.has(i)}
						{#if m.kind === 'image'}
							<div
								class="media image"
								style={`background-image:url("${encodeURI(base + m.url)}")`}
							></div>
						{:else}
							<video
								class="media video"
								bind:this={videoEls[i]}
								src={base + m.url}
								muted
								playsinline
								preload="metadata"
								on:ended={() => handleVideoEnded(i)}
							></video>
						{/if}
					{/if}
				</div>
			{/each}
		</div>

		<!-- Scrolling steps -->
		<div class="scrolly-steps">
			{#each resolvedSteps as step, i (i)}
				<ScrollyStep
					{step}
					{stepHeightVh}
					posClass={POS_CLASS[step.pos ?? 'center']}
					vState={videoStates[i] ?? 'idle'}
					hasVideoButton={stepHasVideoButton(i)}
					bind:textBoxEl={textBoxEls[i]}
					on:videoplay={() => handleStepVideoPlay(i)}
				/>
			{/each}
		</div>
	</section>
{/if}
