<template>
  <div id="app">
    <ul v-if="!id" class="guide-list">
      <guide-list-item
        v-for="(guide, id) in guides"
        :key="id"
        :id="id"
        :guide="guide"
        v-on:pick-guide="pickGuide"/>
    </ul>
    <guide-details
      v-if="id"
      :id="id"
      :alias="alias"
      :guide="guides[id]"
      :album="albums[alias]"
      :route="routes[id]"
      v-on:reset-guide="resetGuide"/>
  </div>
</template>

<script>
  import GuideListItem from './components/GuideListItem.vue';
  import GuideDetails from './components/GuideDetails.vue';
  import guides from './data/guides.json';
  import albums from './data/albums.json';
  import routes from './data/routes.json';

  export default {
    components: {
      GuideListItem,
      GuideDetails
    },
    methods: {
      resetGuide() {
        this.id = null;
        this.alias = null;
      },
      pickGuide(id) {
        this.id = id;
        this.alias = this.guides[id].assets ? this.guides[id].assets.prefix : id;
      }
    },
    mounted() {
      var thumbnails = require.context("./thumbnails", true, /.*\.jpg$/);
      thumbnails.keys().forEach(key => { thumbnails(key); });

      var tiles = require.context("./tiles", true, /.*\.jpg$/);
      tiles.keys().forEach(key => { tiles(key); });
    },
  	data() {
  		return {
        id: null,
        alias: null,
        guides: guides,
        albums: albums,
        routes: routes
  		}
  	}
  }
</script>

<style lang="scss">
  *, *:before, *:after {
    box-sizing: border-box;
  }
  body {
    font-family: sans-serif;
    margin: 0;
    padding: 0;
    overflow: hidden;
  }
  #app {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: stretch;
    width: 100vw;
    height: 100vh;
    position: relative;
  }
  .guide-list {
    width: 100%;
    height: 100%;
    overflow: auto;
    margin: 0;
    padding: 1rem;
    list-style: none;
    li {
      display: block;
      margin: 0 0 1rem;

      button {
        display: block;
        padding: 1rem;
        width: 100%;
      }
    }
  }
</style>
