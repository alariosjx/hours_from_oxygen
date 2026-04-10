<script lang="ts">
	/**
	 * ImageCaption — story photo with caption/credit, Bootstrap-aligned.
	 *
	 * Usage as a shortcode in doc.blocks.json:
	 * {
	 *   "type": "shortcode",
	 *   "name": "ImageCaption",
	 *   "attrs": {
	 *     "src": "/media/photo.jpg",
	 *     "alt": "Description of photo",
	 *     "caption": "Caption text here.",
	 *     "credit": "Photographer Name",
	 *     "align": "full"
	 *   }
	 * }
	 *
	 * align options:
	 *   "full"  — fills the text column (default, matches DocRenderer col width)
	 *   "wide"  — breaks slightly wider (col-10/col-lg-10)
	 *   "left"  — floats left inside the text column, text wraps right
	 *   "right" — floats right inside the text column, text wraps left
	 */

	export let src: string = '';
	export let alt: string = '';
	export let caption: string = '';
	export let credit: string = '';
	export let align: 'full' | 'wide' | 'left' | 'right' = 'full';

	// Map align to Bootstrap col classes. "full" matches the DocRenderer column
	// (col-12 col-sm-10 col-lg-8 col-xxl-6); "wide" steps one tier wider.
	const colClass: Record<string, string> = {
		full: 'col-12 col-sm-10 col-lg-8 col-xxl-6',
		wide: 'col-12 col-sm-11 col-lg-10 col-xxl-8',
		left: 'col-12 col-sm-10 col-lg-8 col-xxl-6',
		right: 'col-12 col-sm-10 col-lg-8 col-xxl-6'
	};
</script>

<div class="container-fluid story-image-wrap story-image-wrap--{align}">
	<div class="row justify-content-center">
		<div class="{colClass[align]}">
			<figure class="story-image story-image--{align}">
				<img {src} {alt} loading="lazy" class="img-fluid" />
				{#if caption || credit}
					<figcaption>
						{#if caption}<span class="caption-text">{caption}</span>{/if}
						{#if credit}<span class="caption-credit">{credit}</span>{/if}
					</figcaption>
				{/if}
			</figure>
		</div>
	</div>
</div>

<style>
	.story-image-wrap {
		margin: 2rem 0;
	}

	.story-image {
		margin: 0;
	}

	.story-image img {
		display: block;
		width: 100%;
		height: auto;
		border-radius: 4px;
		background: #d9d2c4; /* placeholder while image loads */
	}

	figcaption {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		align-items: flex-start;
		gap: 0.2rem 1rem;
		margin-top: 0.5rem;
		padding-top: 0.4rem;
		border-top: 1px solid #d9d2c4;
	}

	.caption-text {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.82rem;
		line-height: 1.5;
		color: #555;
		flex: 1 1 60%;
	}

	.caption-credit {
		font-family: 'Crimson Text', Georgia, serif;
		font-size: 0.75rem;
		color: #999;
		font-style: italic;
		white-space: nowrap;
		align-self: flex-end;
	}

	/* Float variants — only inside the text column on wider screens */
	@media (min-width: 641px) {
		.story-image--left img {
			float: left;
			width: 44%;
			margin: 0.1rem 1.5rem 0.75rem 0;
			border-radius: 3px;
		}

		.story-image--right img {
			float: right;
			width: 44%;
			margin: 0.1rem 0 0.75rem 1.5rem;
			border-radius: 3px;
		}

		/* figcaption sits below the float clearfix */
		.story-image--left figcaption,
		.story-image--right figcaption {
			clear: both;
		}
	}
</style>
