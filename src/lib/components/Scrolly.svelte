<script context="module" lang="ts">
	export type { Step } from './scrolly/utils.js';
</script>

<script lang="ts">
	import { onDestroy, onMount, tick } from 'svelte';
	import { browser } from '$app/environment';
	import { asset } from '$app/paths';

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

	// ---- Background crossfade state ----

	let bgIndex = 0;
	let loadedIndices: Set<number> = new Set();
	let nearViewport = false;

	$: activeAlt = resolvedSteps[bgIndex] ? toMedia(resolvedSteps[bgIndex]).alt : '';

	// ---- DOM refs ----

	let sectionEl: HTMLElement | null = null;
	let textBoxEls: (HTMLElement | null)[] = [];
	let videoEls: (HTMLVideoElement | null)[] = [];

	// ---- Per-step video play state ----

	let videoStates: Record<number, 'idle' | 'playing' | 'ended'> = {};

	function stepHasVideoButton(i: number): boolean {
		const step = resolvedSteps[i];
		return !!step?.videoActionText && toMedia(step).kind === 'video';
	}

	async function ensureVideoPlaying(el: HTMLVideoElement | null, stepIndex: number) {
		if (!el) return;
		if (stepHasVideoButton(stepIndex) && (videoStates[stepIndex] ?? 'idle') === 'idle') return;
		try {
			await tick();
			const p = el.play();
			if (p && typeof (p as Promise<void>).catch === 'function')
				(p as Promise<void>).catch(() => {});
		} catch {
			/* empty */
		}
	}

	function handleVideoEnded(stepIndex: number) {
		if (stepHasVideoButton(stepIndex)) {
			videoStates = { ...videoStates, [stepIndex]: 'ended' };
		}
	}

	async function handleStepVideoPlay(stepIndex: number) {
		const el = videoEls[stepIndex];
		if (!el) return;

		const state = videoStates[stepIndex] ?? 'idle';

		if (state === 'playing') {
			el.pause();
			videoStates = { ...videoStates, [stepIndex]: 'idle' };
			return;
		}

		el.loop = false;
		if (state === 'ended') el.currentTime = 0;

		videoStates = { ...videoStates, [stepIndex]: 'playing' };
		try {
			await tick();
			const p = el.play();
			if (p && typeof (p as Promise<void>).catch === 'function')
				(p as Promise<void>).catch(() => {});
		} catch {
			/* empty */
		}
	}

	// ---- Step transition ----

	async function showStep(newIndex: number) {
		if (!resolvedSteps.length) return;

		const clamped = Math.max(0, Math.min(newIndex, resolvedSteps.length - 1));
		if (clamped === bgIndex && loadedIndices.has(clamped)) return;

		if (stepHasVideoButton(bgIndex)) {
			videoEls[bgIndex]?.pause();
			videoStates = { ...videoStates, [bgIndex]: 'idle' };
		}

		// eslint-disable-next-line svelte/prefer-svelte-reactivity
		const next = new Set(loadedIndices);
		for (const n of [clamped - 1, clamped, clamped + 1]) {
			if (n >= 0 && n < resolvedSteps.length) next.add(n);
		}
		loadedIndices = next;
		bgIndex = clamped;

		await tick();

		const el = videoEls[clamped];
		if (el) {
			el.loop = !resolvedSteps[clamped].videoActionText;
			await ensureVideoPlaying(el, clamped);
		}
	}

	// ---- Scroll / sticky mechanics ----

	let released = false;
	let prevCarryOut = false;
	let carryOut = false;
	let rafPending = false;

	function computePassedIndex(): number {
		const eps = 1;
		let passed = -1;
		const lastIdx = textBoxEls.length - 1;
		const carryY = window.innerHeight * carryPoint;

		for (let i = 0; i < textBoxEls.length; i++) {
			const el = textBoxEls[i];
			if (!el) continue;
			const top = el.getBoundingClientRect().top;
			const threshold = i === lastIdx ? carryY : eps;
			if (top <= threshold) passed = i;
			else break;
		}

		return passed;
	}

	function updateCarryOut() {
		if (!resolvedSteps.length) {
			carryOut = false;
			released = false;
			prevCarryOut = false;
			return;
		}

		const lastIdx = resolvedSteps.length - 1;
		const el = textBoxEls[lastIdx];
		if (!el) {
			carryOut = false;
			released = false;
			prevCarryOut = false;
			return;
		}

		const top = el.getBoundingClientRect().top;
		const carryY = window.innerHeight * carryPoint;

		carryOut = top <= carryY;

		if (carryOut && !prevCarryOut) {
			released = true;
		}

		if (!carryOut && prevCarryOut) {
			released = false;
		}

		prevCarryOut = carryOut;
	}

	function updateFromScroll() {
		rafPending = false;
		if (!resolvedSteps.length) return;

		const passed = computePassedIndex();
		const wantedBg = Math.min(resolvedSteps.length - 1, Math.max(0, passed + 1));
		showStep(wantedBg);
		updateCarryOut();
	}

	function onScrollOrResize() {
		if (!browser || rafPending) return;
		rafPending = true;
		requestAnimationFrame(updateFromScroll);
	}

	// ---- Init when steps change ----

	$: if (resolvedSteps.length) {
		bgIndex = 0;
		if (!nearViewport) loadedIndices = new Set();
		carryOut = false;
		videoStates = {};
		textBoxEls.length = resolvedSteps.length;
		videoEls.length = resolvedSteps.length;
	} else {
		bgIndex = 0;
		loadedIndices = new Set();
		carryOut = false;
		textBoxEls = [];
		videoEls = [];
	}

	// ---- Intersection observer (lazy-load media) ----

	let observer: IntersectionObserver | null = null;

	onMount(() => {
		if (!browser) return;

		window.addEventListener('scroll', onScrollOrResize, { passive: true });
		window.addEventListener('resize', onScrollOrResize);

		observer = new IntersectionObserver(
			async (entries) => {
				if (!entries[0].isIntersecting) return;
				observer?.disconnect();
				observer = null;

				nearViewport = true;
				loadedIndices = new Set([0, 1].filter((i) => i < resolvedSteps.length));

				await tick();

				const el = videoEls[0];
				if (el) {
					el.loop = !resolvedSteps[0]?.videoActionText;
					await ensureVideoPlaying(el, 0);
				}

				onScrollOrResize();
			},
			{ rootMargin: '500px 0px' }
		);

		if (sectionEl) observer.observe(sectionEl);
	});

	onDestroy(() => {
		if (browser) {
			window.removeEventListener('scroll', onScrollOrResize);
			window.removeEventListener('resize', onScrollOrResize);
		}
		observer?.disconnect();
	});
</script>

{#if resolvedSteps.length}
	<section
		bind:this={sectionEl}
		class="scrolly full-bleed"
		style={`--fade-ms:${fadeDurationMs}ms; --text-bottom:${textBottomVh}vh;`}
	>
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
								style={`background-image:url("${encodeURI(asset(m.url))}")`}
							></div>
						{:else}
							<video
								class="media video"
								bind:this={videoEls[i]}
								src={asset(m.url)}
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

<style>
	.scrolly {
		position: relative;
	}

	.scrolly-bg {
		position: sticky;
		top: 0;
		left: 0;
		overflow: hidden;
		z-index: 0;
		margin-bottom: 2rem;
	}

	.layer {
		position: absolute;
		inset: 0;
		opacity: 0;
		transition: opacity var(--fade-ms) ease;
		transform: translateZ(0);
	}

	.layer.active {
		opacity: 1;
	}

	.media {
		position: absolute;
		inset: 0;
	}

	.media.image {
		background-size: cover;
		background-position: center;
	}

	.media.video {
		width: 100%;
		height: 100%;
		object-fit: cover;
		object-position: center;
	}

	.scrolly-steps {
		position: relative;
		z-index: 1;
	}
</style>
