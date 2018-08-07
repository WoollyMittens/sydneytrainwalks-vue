<template>
	<figure class="guide-details-map">
		<l-map
			ref="map"
			:zoom="zoom"
			:center="center"
			:minZoom="minZoom"
			:maxZoom="maxZoom"
			:bounds="bounds"
			:maxBounds="maxBounds">
			<l-control-scale
				:imperial="false"/>
      <l-tile-layer
				:url="url"
				:attribution="attribution"
				v-on:tileerror="onTileError($event)"/>
			<l-geo-json
				:geojson="route"
				:options="options"/>
			<l-marker
				v-if="mapLocation"
				:lat-lng="mapLocation"
				:icon="pickIcon('location')"/>
			<l-marker
				v-if="photo"
				:lat-lng="photo"
				:icon="pickIcon('photo')"/>
			<l-marker
				v-for="(marker, id) in guide.markers"
				:key="id"
				:icon="pickIcon(marker.type)"
        :lat-lng="marker.lat ? marker : dimensions[id]">
				<l-popup
					v-if="marker.description"
					:content="marker.description"/>
      </l-marker>
		</l-map>
	</figure>
</template>

<script>
	import {LMap, LTileLayer, LGeoJson, LMarker, LPopup, LControlScale} from 'vue2-leaflet';
	import "leaflet/dist/leaflet.css"
  import Config from '../data/config.json';
	import LocationMarker from "../markers/marker-location.png";
	import TrainMarker from "../markers/marker-train.png";
	import FerryMarker from "../markers/marker-ferry.png";
	import BusMarker from "../markers/marker-bus.png";
	import InfoMarker from "../markers/marker-info.png";
	import WarningMarker from "../markers/marker-warning.png";
	import ToiletMarker from "../markers/marker-toilet.png";
	import KioskMarker from "../markers/marker-kiosk.png";
	import LandmarkMarker from "../markers/marker-landmark.png";
	import PhotoMarker from "../markers/marker-photo.png";
	import TentMarker from "../markers/marker-tent.png";
	import HotelMarker from "../markers/marker-hotel.png";

	export default {
		props: ['id', 'guide', 'route', 'photo'],
		components: {LMap, LTileLayer, LGeoJson, LMarker, LPopup, LControlScale},
		data () {
			var dimensions = this.calculateDimensions(this.route, this.photo);
			return {
				dimensions: dimensions,
				zoom: dimensions.zoom,
				minZoom: 10,
				maxZoom: 15,
				bounds: dimensions.limits,
				maxBounds: dimensions.limits,
				center: dimensions.center,
				options: {
	        style: {
						color: '#ff6600',
						weight: 5,
						opacity: 0.66
					}
	      },
				url: Config.remoteMapURL,
				attribution: Config.mapAttribution,
				mapLocation: null
			}
		},
		methods: {
			onTileError(evt) {
				console.log('onTileError', evt);
				if (!/\/tiles\//i.test(evt.tile.src)) {
					evt.tile.src = Config.localMapURL.replace("{x}", evt.coords.x).replace("{y}", evt.coords.y).replace("{z}", evt.coords.z);
				}
			},
			onUpdatedLocation(result) {
				this.mapLocation = {lat: result.coords.latitude, lng: result.coords.longitude};
			},
			onFailedLocation(e) {
				console.log(e);
			},
			flattenCoordinates(route) {
				const features = route.features;
				var segments = features.map(
					feature => (feature.geometry.coordinates[0][0] instanceof Array)
					? [].concat.apply([], feature.geometry.coordinates)
					: feature.geometry.coordinates);
				return [].concat.apply([], segments);
			},
			pickIcon(type) {
				var marker;
				switch(type) {
					case "location": marker = LocationMarker; break;
					case "train": marker = TrainMarker; break;
					case "ferry": marker = FerryMarker; break;
					case "bus": marker = BusMarker; break;
					case "warning": marker = WarningMarker; break;
					case "toilet": marker = ToiletMarker; break;
					case "kiosk": marker = KioskMarker; break;
					case "landmark": marker = LandmarkMarker; break;
					case "photo": marker = PhotoMarker; break;
					case "tent": marker = TentMarker; break;
					case "hotel": marker = HotelMarker; break;
					case "info": marker = InfoMarker; break;
					default: marker = LocationMarker;
				}
				return new L.icon({
					iconUrl: marker,
					iconSize: [32, 32],
					iconAnchor: [16, 32],
					popupAnchor: [0, -16]
				});
			},
			calculateDimensions(route, photo) {
				var minLat = 999,
					minLng = 999,
					maxLat = -999,
					maxLng = -999;
				var points = this.flattenCoordinates(route);
				points.forEach(point => {
					minLng = (point[0] < minLng) ? point[0] : minLng;
					minLat = (point[1] < minLat) ? point[1] : minLat;
					maxLng = (point[0] > maxLng) ? point[0] : maxLng;
					maxLat = (point[1] > maxLat) ? point[1] : maxLat;
				});
				var maxWidth = Math.round(13 - (maxLng - minLng) / 0.333 * 3);
				var maxHeight = Math.round(13 - (maxLat - minLat) / 0.333 * 3);
				var maxZoom = Math.max(Math.min(maxWidth, maxHeight, 13), 10);
				return {
					center: photo ? [photo.lat, photo.lng] : [
						(maxLat - minLat) / 2 + minLat,
						(maxLng - minLng) / 2 + minLng
					],
					limits: [
						[minLat - 0.01, minLng - 0.01],
						[maxLat + 0.01, maxLng + 0.01]
					],
					zoom: photo ? 14 : maxZoom,
					start: {lng: points[0][0], lat: points[0][1]},
					end: {lng: points[points.length - 1][0], lat: points[points.length - 1][1]}
				};
			}
		},
		updated() {
			this.$nextTick(() => {
				if (this.photo) {
					this.map.setView(this.photo, 14);
				}
      });
		},
		mounted() {
			this.$nextTick(() => {
				this.map = this.$refs.map.mapObject;
      });
			this.watcher = ("geolocation" in navigator)
				? navigator.geolocation.watchPosition(this.onUpdatedLocation.bind(this), this.onFailedLocation.bind(this), {
					enableHighAccuracy: true,
					maximumAge: 30000,
					timeout: 27000
				})
				: null;
		}
	}
</script>

<style lang="scss">
	.guide-details-map {
		width: 100%;
		height: 100%;
		position: relative;

		.leaflet-container {
			position: absolute;
			width: 100%;
			height: 100%;
		}
	}
</style>
