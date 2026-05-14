<!--
  DropCap.svelte
  Classic journalist drop-cap opener for the first paragraph of a story.
  The first letter is enlarged to span multiple lines, with body text wrapping around it.
  Props:
    letter — the single drop-cap character (default "E")
    text   — the rest of the opening paragraph (everything after the first letter)
-->

<script>
	export let letter = 'T';
	export let text =
		'he oxygen monitor on Jose Jimenez’s, 82, hand started beeping late one night in December 2020. His blood oxygen was tanking. His daughter, Imelda, knew he would die without medical attention. COVID-19 ravaged across the Mexican countryside, limiting the number of hospital beds, so the family had to gamble which medical center would be able to accept their ailing father and quickly get him ready for the drive from the farmlands to the nearest major city.';
</script>

<p class="story-opener">
	<span class="drop-cap">{letter}</span>{text}
</p>

<style>
	.story-opener {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(1.05rem, 2vw, 1.25rem);
		line-height: 1.75;
		color: #2a1a0e;
		margin: 0 0 1.5rem 0;
		overflow: hidden;
	}

	.drop-cap {
		float: left;
		font-family: 'Crimson Text', Georgia, serif;
		font-size: clamp(4rem, 10vw, 6.5rem);
		line-height: 0.78;
		padding-top: 0.12em;
		padding-right: 0.08em;
		padding-bottom: 0;
		color: #c8960a;
		font-weight: 600;
		font-style: normal;
	}
</style>
