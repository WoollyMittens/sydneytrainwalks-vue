<template>
	<section class="guide-details">
		<header>
			<h2>From {{ guide.markers.start.location }} to {{ guide.markers.end.location }} via {{ guide.location }}</h2>
		</header>
		<div class="guide-details-row">
			<guide-details-map
				:id="id"
				:guide="guide"
				:route="route"
				:photo="photo"/>
			<article>
				<p v-html="guide.description.join(' ')"></p>
				<ul><li><a :href="'../assets/' + id + '.gpx'">Download GPX</a></li></ul>
				<guide-details-landmark
					v-for="(caption, image) in guide.landmarks"
					:key="'landmarks-' + image"
					:alias="alias"
					:image="image"
					:caption="caption"
					v-on:pick-photo="pickPhoto"/>
			</article>
		</div>
		<div class="guide-details-row">
			<aside>
				<guide-details-landmark
					v-for="(coords, image) in album"
					:key="'gallery-' + image"
					:alias="alias"
					:image="image"
					caption=""
					v-on:pick-photo="pickPhoto"/>
			</aside>
		</div>
		<footer>
			<button v-on:click="$emit('reset-guide')">Home</button>
		</footer>
	</section>
</template>

<script>
	import GuideDetailsMap from './GuideDetailsMap.vue';
	import GuideDetailsLandmark from './GuideDetailsLandmark.vue';

	export default {
		props: ['id', 'alias', 'guide', 'route', 'album'],
		components: {GuideDetailsMap, GuideDetailsLandmark},
		data () {
			return {
				photo: null
			}
		},
		methods: {
      pickPhoto(image) {
				this.photo = this.album[image];
      }
    },
		mounted() {
			var externalLinks = this.$el.querySelectorAll('a[href^=http]');
			var openExternal = function(evt) {
				evt.preventDefault();
				window.open(evt.target.getAttribute('href'), '_blank');
			};
			externalLinks.forEach(function(link){
				link.addEventListener('click', openExternal.bind(this));
			});
		}
	}
</script>

<style lang="scss">
	.guide-details {
		flex: 1 1 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: stretch;

		> header {
			flex: 0 1 auto;
			padding: 0 1rem;
			position: relative;
			min-height: 70px;
			z-index: 2;
			box-shadow: 0 0.3rem 0.3rem rgba(0,0,0,0.3);
		}
		.guide-details-row {
			flex: 1 1 50%;
			display: flex;
			flex-direction: row;
			flex-wrap: nowrap;
			justify-content: center;
			align-items: stretch;
			position: relative;
			z-index: 1;

			> figure {
				flex: 1 1 50%;
				height: 100%;
				margin: 0;
			}
			> article {
				flex: 1 1 50%;
				height: calc(100vh - 70px);
				overflow: auto;
				padding: 0 1rem;

				> figure {
					margin: 0 0 1rem 0;
					display: flex;
					align-items: center;

					> span {
						flex: 0 1 auto;
						display: flex;
						justify-content: center;
						align-items: center;
						background-repeat: no-repeat;
						background-position: center;
						background-size: cover;
						width: 8rem;
						min-width: 8rem;
						height: 8rem;
						position: relative;

						&:before {
							content: '';
							display: block;
							position: absolute;
							top: 0;
							right: 0;
							bottom: 0;
							left: 0;
							background-color: rgba(0,0,0,0.6);
							z-index: 0;
						}

						img {
							width: auto;
							height: auto;
							max-width: 8rem;
							max-height: 8rem;
							position: relative;
							z-index: 1;
						}
					}
					> figcaption {
						flex: 1 1 auto;
						padding: 0 1rem;
					}
				}
			}
			> aside {
				flex: 1 1 50%;
				height: 100%;
				overflow: auto;
				display: flex;
				flex-wrap: wrap;

				> figure {
					margin: 1px;
					flex: 1 1 auto;
					display: flex;

					> span {
						flex: 1 1 auto;
						background-repeat: no-repeat;
						background-position: center;
						background-size: cover;

						img {
							visibility: hidden;
							vertical-align: top;
							width: auto;
							height: auto;
							max-height: 8rem;
							max-width: 100%;
						}
					}
					> figcaption {
						display: none;
					}
				}
			}
		}
		> footer {
			flex: 0 1 auto;
			padding: 1rem;
			position: relative;
			z-index: 2;
			box-shadow: 0 -0.3rem 0.3rem rgba(0,0,0,0.3);

			button {
				display: block;
        padding: 1rem;
        width: 100%;
			}
		}
	}
</style>
