<template>
	<li class="guide-list-item">
		<button v-on:click="$emit('pick-guide', id)">
			From {{ guide.markers.start.location }} to {{ guide.markers.end.location }} via {{ guide.location }}
		</button>
	</li>
</template>

<script>
	export default {
		props: ['id', 'guide']
	}
</script>

<style lang="scss">
	.guide-list-item {}
</style>
