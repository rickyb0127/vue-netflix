<template>
  <div v-if="category" class="video-carousel">
    <h3 class="category-title">{{ category.title }}</h3>
    <div @mouseenter="displayScrollIcons" @mouseleave="hideScrollIcons" class="carousel-container">
      <i @click="decreaseContentGroupIndex" class="scroll-left fa-solid fa-chevron-left" :id="`scroll-left-${category.id}`"></i>
      <div v-if="contentGroups && contentGroups.length" class="row content-row">
        <div @mouseenter="expandContent(content)" @mouseleave="hideExpandContent(content)" :id="`content-cell-${content.id}`" v-for="content in contentGroups[contentGroupIndex]" :key="content.id" class="content-cell col-6" :style="{backgroundImage: `url(${imgBaseUrl}${content.poster_path})`}">
          <div :id="`content-cell-overlay-${content.id}`" class="content-cell-overlay">
            <div class="overlay-bg" :style="{backgroundImage: `url(${imgBaseUrl}${content.poster_path})`}"></div>
            <div class="overlay-info">
              <div class="overlay-icons">
                <div class="left-side-overlay">
                  <i class="overlay-icon-spacing fa-solid fa-circle-play"></i>
                  <i class="overlay-icon-spacing fa-solid fa-circle-plus"></i>
                  <i class="overlay-icon-spacing fa-solid fa-thumbs-up"></i>
                </div>
                <div class="right-side-overlay">
                  <i @click="openContentModal" class="fa-solid fa-circle-chevron-down"></i>
                </div>
              </div>
              <div class="overlay-details">
                <div v-if="contentDetails.rating" class="content-rating">
                  {{ contentDetails.rating }}
                </div>
                <div v-if="contentDetails.number_of_episodes" class="content-detail-spacing">
                  {{ contentDetails.number_of_episodes }} Episodes
                </div>
                <div v-if="contentDetails.runtime">
                  {{ getReadableRuntime(contentDetails.runtime) }}
                </div>
              </div>
              <div v-if="contentDetails.genres" class="overlay-details">
                <div v-for="genre, index in contentDetails.genres" :key="index" class="genre-list">
                  {{ genre }}
                  <i v-if="index !== contentDetails.genres.length - 1" class="genre-seperator fa-solid fa-circle"></i>
                </div>
              </div>
            </div>
          </div>        
        </div>
      </div>
      <i @click="increaseContentGroupIndex" class="scroll-right fa-solid fa-chevron-right" :id="`scroll-right-${category.id}`"></i>
    </div>
    <ContentModal @content-modal-closed="closeContentModal" v-if="showContentModal" :contentDetails="contentDetails" />
  </div>
</template>

<script>
import ContentModal from './ContentModal.vue';

export default {
  name: 'VideoCarousel',
  props: {
    category: Object
  },
  components: {
    ContentModal
  },
  data() {
    return {
      contents: [],
      contentGroups: [],
      contentGroupIndex: 0,
      contentDetails: {},
      imgBaseUrl: 'https://www.themoviedb.org/t/p/w440_and_h660_face/',
      showContentModal: null
    }
  },  
  methods: {
    async fetchContent() {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/${this.category.searchQuery}/top_rated?api_key=177095cacff5e15a4fcc830553cfd31b&language=en-US&page=1`);
        
        const responseJson = await response.json();
        return responseJson.results;
      } catch(err) {
        console.log(err);
      }
    },
    async fetchContentDetails(contentId) {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/${this.category.searchQuery}/${contentId}?api_key=177095cacff5e15a4fcc830553cfd31b&language=en-US&page=1`);

        const responseJson = await response.json();
        return responseJson;
      } catch(err) {
        console.log(err);
      }
    },
    async fetchContentRatingUS(contentId) {
      try {
        const response = await fetch(`https://api.themoviedb.org/3/${this.category.searchQuery}/${contentId}/content_ratings?api_key=177095cacff5e15a4fcc830553cfd31b&language=en-US&page=1`);
        
        const responseJson = await response.json();
        const ratingUS = responseJson.results.find(res => res.iso_3166_1 === 'US').rating;
        return ratingUS;
      } catch(err) {
        console.log(err);
      }
    },
    decreaseContentGroupIndex() {
      const newIndex = this.contentGroupIndex - 1;
      if(newIndex < 0) {
        this.contentGroupIndex = this.contentGroups.length - 1;
      } else {
        this.contentGroupIndex = newIndex;
      }
    },
    increaseContentGroupIndex() {
      const newIndex = this.contentGroupIndex + 1;
      if(newIndex < this.contentGroups.length - 1) {
        this.contentGroupIndex = newIndex;
      } else {
        this.contentGroupIndex = 0;
      }
    },
    displayScrollIcons() {
      const scrollLeft = document.getElementById(`scroll-left-${this.category.id}`);
      const scrollRight = document.getElementById(`scroll-right-${this.category.id}`);

      scrollLeft.style.display = 'block';
      scrollRight.style.display = 'block';
    },
    hideScrollIcons() {
      const scrollLeft = document.getElementById(`scroll-left-${this.category.id}`);
      const scrollRight = document.getElementById(`scroll-right-${this.category.id}`);

      scrollLeft.style.display = 'none';
      scrollRight.style.display = 'none';
    },
    async expandContent(content) {
      const rating = this.category.searchQuery === 'tv' ? await this.fetchContentRatingUS(content.id) : undefined;
      const contentDetails = await this.fetchContentDetails(content.id);
      let genres = undefined;
      if(contentDetails.genres) {
        const maxIndex = contentDetails.genres.length > 3 ? 3 : contentDetails.genres.length;
        genres = [...contentDetails.genres].splice(0, maxIndex).map(genre => genre.name);
      }
      
      this.contentDetails = {
        ...contentDetails,
        genres,
        rating
      }
      
      const contentElem = document.getElementById(`content-cell-overlay-${content.id}`);

      contentElem.style.display = 'block';
    },
    async hideExpandContent(content) {
      const contentElem = document.getElementById(`content-cell-overlay-${content.id}`);

      contentElem.style.display = 'none';
    },
    openContentModal() {
      this.showContentModal = true;

      const body = document.getElementsByTagName("BODY")[0];
      body.classList.add('no-scroll');
    },
    closeContentModal() {
      this.showContentModal = false;

      const body = document.getElementsByTagName("BODY")[0];
      body.classList.remove('no-scroll');
    },
    getReadableRuntime(totalMinutes) {
      if(totalMinutes >= 60) {
        const hours = (totalMinutes/60).toFixed(0);
        const minutes = totalMinutes%60;

        return `${hours}h ${minutes}m`;
      } else {
        return `${totalMinutes}m`;
      }
    }
  },
  async created() {    
    this.contents = await this.fetchContent();

    const numContentsPerGroup = 6;
    this.contentGroups = this.contents.reduce((result, content, index) => { 
      const chunkIndex = Math.floor(index/numContentsPerGroup);

      if(!result[chunkIndex]) {
        result[chunkIndex] = [];
      }

      result[chunkIndex].push(content);

      return result;
    }, []);
  }
}
</script>

<style scoped>
.video-carousel {
  height: 200px;
}

.carousel-container {
  position: relative;
  color: white;
}

.category-title {
  color: white;
  padding-left: 55px;
}

.content-row {
  padding-left: 50px;
  padding-right: 50px;
}

.content-cell {
  border: 1px solid transparent;
  height: 125px;
  border-radius: 5px;
  background-size: cover;
  background-repeat: no-repeat;
  cursor: pointer;
  position: relative;
}

.content-cell-overlay {
  display: none;
  transition: transform .6s;
  width: 300px;
  height: 350px;
  border-radius: 5px;
  margin: 0 auto;
  position: absolute;
  left: -18%;
  top: -70%;
  z-index: 1000;
  background-color: black;
}

.content-cell-overlay:hover {
  transform: scale(1.1);
}

.overlay-bg {
  height: 170px;
  background-position: center;
  background-size: cover;
}

.overlay-info {
  padding: 20px;
}

.overlay-icons {
  color: white;
  font-size: 35px;
  display: flex;
}

.overlay-details {
  color: white;
  display: flex;
  padding-top: 20px;
}

.genre-seperator {
  font-size: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

.overlay-icon-spacing {
  padding-right: 10px;
}

.right-side-overlay {
  margin-left: auto;
  cursor: pointer;
}

.scroll-left {
  display: none;
  position: absolute;
  top: 40px;
  left: 15px;
  font-size: 50px;
  cursor: pointer;
}

.scroll-right {
  display: none;
  position: absolute;
  top: 40px;
  right: 15px;
  font-size: 50px;
  cursor: pointer;
}

.genre-list {
  display: flex;
  align-items: center;
}
</style>