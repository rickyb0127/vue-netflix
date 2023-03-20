<template>
  <NavBar />
  <video class="bg-video" autoplay="autoplay" muted>
    <source src="./assets/earth-video.mp4">
    Your browser does not support the video tag.
  </video>
  <div v-for="category in categories" :key="category.id">
    <VideoCarousel :category="category" />
  </div>
  <FooterComponent />
</template>

<script>
import NavBar from './components/NavBar.vue'
import VideoCarousel from './components/VideoCarousel.vue'
import FooterComponent from './components/FooterComponent.vue'

export default {
  name: 'App',
  components: {
    NavBar,
    VideoCarousel,
    FooterComponent
  },
  data() {
    return {
      categories: [
        {
          id: 1,
          title: 'Movies',
          searchQuery: 'movie'
        },
        {
          id: 2,
          title: 'TV Shows',
          searchQuery: 'tv'
        }
      ]
    }
  }
}
</script>

<style scoped>
.bg-video {
  width: 100%;
  height: 800px;
  object-fit: fill;
}
</style>
