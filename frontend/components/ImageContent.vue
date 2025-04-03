<template>
	<div v-if="currentSvgContent" v-html="currentSvgContent"></div>
	<img
		v-if="imageData && imageData.length > 0"
		:id="'graph-image-' + slugify(title)"
		class="image-content"
		:src="currentDateUrl"
		:alt="title"
	>
	<button
		class="move-button"
		type="button"
		@click="move(-1)"
	>
		<!-- eslint-disable-next-line vue/no-parsing-error -->
		<<
	</button>
	<input
		id="slider"
		type="range"
		:min="0"
		:max="imageData.length - 1"
		step="1"
		:value="currentIndex"
		@input="updateIndex($event.target.value)"
	>
	<button
		class="move-button"
		type="button"
		@click="move(1)"
	>
		>>
	</button>
	<a
		:href="currentDataUrl"
		:download="'graph-image-' + slugify(title) + '-' + (currentIndex + 1) + '.svg'"
	>
		<span class="sr-only">Download image</span>
		<svg
			class="download-svg"
			xmlns="http://www.w3.org/2000/svg"
			role="img"
			fill="none"
			viewBox="0 0 24 24"
			stroke-width="1.5"
			stroke="currentColor"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3"
			/>
		</svg>
	</a>
</template>

<script>
export default {
	name: 'ImageContent',
	props: {
		imageData: {
			type: Array,
			required: true
		},
		title: {
			type: String,
			required: true
		},
	},
	data() {
		return {
			index: null,
			tempVar: '<svg width="100" height="100" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg"><circle cx="50" cy="50" r="40" fill="red"/></svg>',
	
		}
	},
	computed: {
		currentIndex() {
			return this.index !== null && this.index !== undefined
			? this.index
			: (this.imageData && this.imageData.length > 0 ? this.imageData.length - 1 : 0);
		},
		currentSvgContent() {
			if(!this.imageData || this.imageData.length == 0) return null;
			const currentData = this.imageData[this.currentIndex];
			// Extract SVG using regex
			const svgMatch = this.extractSvg(currentData);
			return svgMatch ? svgMatch : currentData;
		},
		currentDataUrl() {
			if (!this.currentSvgContent) return '';
			
			try {
			// Debug logging
			//console.log('BEFORE CLEANUP:');
			//console.log(this.currentSvgContent);
			
			// First, normalize all path data attributes
			const normalizedSvg = this.currentSvgContent.replace(
				/d="([^"]*)"/g,
				(match, pathData) => {
				const cleanedPath = pathData
					.replace(/\\n/g, '')                  // Remove literal \n
					.replace(/\n/g, '')                   // Remove newlines
					.replace(/\s+/g, ' ')                 // Normalize spaces
					.replace(/([MLHVCSQTAZ])(\s+)/g, '$1') // Remove spaces after commands
					.replace(/\s*([MLHVCSQTAZ])/g, '$1'); // Remove spaces before commands
					
				//console.log('Cleaned path:', cleanedPath);
				return `d="${cleanedPath}"`;
				}
			);
			
			// Then apply general cleanup
			const cleanedSvg = normalizedSvg
				.replace(/<!--.*?-->/g, '')                    // Remove comments
				.replace(/\n/g, ' ')                          // Replace newlines
				.replace(/\s+/g, ' ')                         // Collapse whitespace
				.replace(/>\s+</g, '><')                      // Clean up tag spacing
				.trim();
			
			//console.log('\nAFTER CLEANUP:');
			//console.log(cleanedSvg);
			return '';
			//return 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(cleanedSvg);
			} catch (error) {
			console.error('Error cleaning SVG:', error);
			return 'data:image/svg+xml;charset=utf-8,' + encodeURIComponent(this.currentSvgContent);
			}
		}
	},
	methods: {
		extractSvg(text) {
			// Pattern to match opening SVG tag, content, and closing tag
			const pattern = /(<svg[\s\S]*?<\/svg>)/i;
			const match = String(text).match(pattern);
			return match ? match[0] : null;
		},
		move(direction) {
			if (direction === -1)
				this.updateIndex(0)
			else
				this.updateIndex(this.imageData.length -1)
		},
		slugify(str) {
			return str.toLowerCase().trim().replace(/[^\w\s-]|(?:^-+)|(?:-+$)/g, '').replace(/\s+/g, '-')
			// I believe these are equivalent but the above is less replace operations and is more clear what is being replaced. If we run into issues, revert back to this
			//return str.toLowerCase().trim().replace(/[^\w\s-]/g, '').replace(/[\s_-]+/g, '-').replace(/(?:^-+)|(?:-+$)/g, '')
		},
		updateIndex(value) {
			if (!this.imageData || this.imageData.length === 0) return;

			value = Number(value);

			value = Math.max(0, Math.min(value, this.imageData.length - 1));

			if (value >= this.imageData.length - 1)
				this.index = null;
			else
				this.index = value;
		}
	}
}
</script>