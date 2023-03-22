<template>
  <div v-if="contentDetails" class="modal-container">
    <div class="modal">
      <div class="modal-bg" :style="{backgroundImage: `url(${imgBaseUrl}${contentDetails.poster_path})`}">
        <div class="close-icon-container">
          <i @click="$emit('content-modal-closed')" class="fa-regular fa-circle-xmark"></i>
        </div>
        <div class="bg-controls-overlay content-modal-container ">
          <div class="play-btn">
            <div class="play-btn-wrapper">
              <i class="fa-solid fa-play"></i>
              <span class="play-text">Play</span>
            </div>
          </div>
          <i class="overlay-icon fa-solid fa-circle-plus"></i>
          <i class="overlay-icon fa-solid fa-thumbs-up"></i>
        </div>
      </div>
      <div class="detail-row content-modal-container">
        <div>{{ getAirYear() }}</div>
        <div v-if="contentDetails.rating" class="content-detail-spacing">
          <div class="content-rating">{{ contentDetails.rating }}</div>
        </div>
        <div v-if="contentDetails.seasons && contentDetails.seasons.length > 1" class="content-detail-spacing">
          {{ contentDetails.seasons.length - 1 }} Seasons
        </div>
        <div v-if="contentDetails.runtime" class="content-detail-spacing">
          {{ getReadableRuntime(contentDetails.runtime) }}
        </div>
      </div>
      <div v-if="contentDetails.number_of_episodes">
        <div class="episode-selection-container">
          <h3>Episodes</h3>
          <div v-if="contentDetails.seasons && contentDetails.seasons.length > 2" class="select-wrapper">
            <select name="season" v-model="selectedSeason">
              <option v-for="_season, index in contentDetails.seasons.filter((_, idx) => idx !== contentDetails.seasons.length - 1)" :key="index" :value="index + 1">Season {{ index + 1 }}</option>
            </select>
          </div>
        </div>
        <div class="episode-container content-modal-container">
          <div v-for="episode in episodes" :key="episode.id" class="episode-wrapper">
            <div class="episode-box">
              <div class="episode-number">{{ episode.episode_number }}</div>
              <div v-if="episode.image" class="episode-img" :style="{backgroundImage: `url(${imgBaseUrl}${episode.image})`}"></div>
              <div class="episode-info">
                <div class="episode-title">{{ episode.name }}</div>
                <div class="episode-summary">{{ episode.overview }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-else>
        <div class="movie-info-container content-modal-container">
          <h3>{{ contentDetails.title }}</h3>
          <div>{{ contentDetails.overview }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ContentModal',
  props: {
    contentDetails: Object
  },
  data() {
    return {
      episodes: [],
      imgBaseUrl: 'https://www.themoviedb.org/t/p/w440_and_h660_face/',
      selectedSeason: 1
    }
  },
  watch: {
    async selectedSeason(newVal, oldVal) {
      if(newVal && newVal !== oldVal) {
        await this.setEpisodes(newVal);
      }
    }
  },
  methods: {
    getAirYear() {
      const airDate = this.contentDetails.release_date ? this.contentDetails.release_date : this.contentDetails.first_air_date;

      return airDate?.substring(0, 4);
    },
    async setEpisodes(seasonNumber) {
      const episodeData = await this.fetchEpisodeData(seasonNumber);
      let imagesList = [];

      if(episodeData) {
        for(let i = 0; i < episodeData.length - 2; i++) {
          const images = await this.fetchSeasonImage(seasonNumber, i + 1);

          imagesList.push(images ? images[0].file_path : null);
        }

        this.episodes = episodeData.map((episode, index) => {
          let image = null;
          if(index < imagesList.length - 1) {
            image = imagesList[index];
          }

          return {
            ...episode,
            image
          }
        });
      }
    },
    async fetchEpisodeData(seasonNumber) {
      const seasonNumberQuery = seasonNumber < this.contentDetails.seasons.length ? seasonNumber : 1;

      try {
        const response = await fetch(`https://api.themoviedb.org/3/tv/${this.contentDetails.id}/season/${seasonNumberQuery}?api_key=177095cacff5e15a4fcc830553cfd31b&language=en-US&page=1`);

        const responseJson = await response.json();
        return responseJson.episodes;
      } catch(err) {
        console.log(err);
      }
    },
    async fetchSeasonImage(seasonNumber, episodeNumber) {
      const seasonNumberQuery = seasonNumber < this.contentDetails.seasons.length ? seasonNumber : 1;

      try {
        const response = await fetch(`https://api.themoviedb.org/3/tv/${this.contentDetails.id}/season/${seasonNumberQuery}/episode/${episodeNumber}/images?api_key=177095cacff5e15a4fcc830553cfd31b&language=en-US&page=1`);

        const responseJson = await response.json();
        return responseJson.stills;
      } catch(err) {
        console.log(err);
      }
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
    if(this.contentDetails && this.contentDetails.seasons) {
      await this.setEpisodes(1);
    }
  }
}
</script>

<style scoped>
.modal-container {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  overflow-y: scroll;
}

.modal {
  background: black;
  box-shadow: 2px 2px 20px 1px;
  overflow-x: auto;
  width: 800px;
  height: 600px;
  border-radius: 5px;
  position: relative;
}

.close-icon-container {
  position: absolute;
  top: 10px;
  right: 10px;
}

.close-icon-container .fa-circle-xmark {
  font-size: 40px;
  cursor: pointer;
  color: white;
}

.modal-bg {
  height: 450px;
  background-position: center;
  background-size: cover;
  position: relative;
}

.bg-controls-overlay {
  position: absolute;
  bottom: 50px;
  display: flex;
  align-items: center;
  cursor: pointer;
}

.overlay-icon {
  color: white;
  font-size: 25px;
  padding-left: 10px;
}

.play-btn {
  width: 90px;
  height: 30px;
  border: 1px solid transparent;
  background-color: white;
  border-radius: 5px;
}

.play-btn .fa-play {
  font-size: 25px;
}

.play-btn-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}

.play-text {
  padding-left: 10px;
}

.content-modal-container {
  padding-left: 40px;
  padding-right: 40px;
}

.detail-row {
  padding-top: 20px;
  padding-bottom: 20px;
  display: flex;
  color: white;
}

.episode-selection-container {
  display: flex;
  align-items: center;
  color: white;
  border-bottom: 1px solid gray;
  margin-left: 44px;
  margin-right: 44px;
}

.select-wrapper {
  margin-left: auto;
}

select {
  height: 30px;
  width: 120px;
  background: gray;
  color: white;
  border-radius: 5px;
}

.episode-container {
  color: white;
}

.episode-wrapper {
  display: flex;
  height: 150px;
  padding-left: 44px;
  padding-right: 44px;
  border-bottom: 1px solid gray;
}

.episode-box {
  display: flex;
  align-items: center;
}

.episode-number {
  padding-right: 20px;
}

.episode-img {
  padding-right: 20px;
  height: 50px;
  width: 70px;
  background-size: contain;
  background-repeat: no-repeat;
}

.episode-title {
  font-weight: bold;
  padding-bottom: 5px;
}

.movie-info-container {
  color: white;
}
</style>