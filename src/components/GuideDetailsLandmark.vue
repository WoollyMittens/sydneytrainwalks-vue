<template>
	<figure class="guide-details-landmark">
		<span :style="backgroundUrl()" v-on:click="$emit('pick-photo', image)"><img alt="" :src="photoUrl()"/></span>
		<figcaption>{{caption}}</figcaption>
	</figure>
</template>

<script>
	export default {
		props: ['alias', 'image', 'caption'],
		methods: {
			photoUrl() {
				return require('../thumbnails/' + this.alias + '/' + this.image + '.jpg');
			},
			backgroundUrl() {
				return 'background-image:url(' + require('../thumbnails/' + this.alias + '/' + this.image + '.jpg') + ')';
			}
		}
	}
</script>

<style lang="scss">
	.guide-details-landmark {
		span {
			cursor: pointer;

			img {
				vertical-align: top;
			}
		}
		figcaption {}
	}
</style>
