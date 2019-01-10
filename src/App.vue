<template>
  <div id="app">
    <h1>Zone Movie Listings</h1>
    <ul id="movie-list" class="movie-list">
      <li v-for="(movie, index) in movies" :key="index">
        <MovieListItem :movie-data="movie" :genre-ids="genres" :poster-base-url="tmdbPosterBaseUrl"></MovieListItem>
      </li>
    </ul>
  </div>
</template>

<script>
import MovieListItem from "./components/MovieListItem.vue";
import axios from 'axios';

export default {
  name: "app",
  components: {
    MovieListItem
  },
  data() {
    return {
      tmdbApiGenreListUrl: "https://api.themoviedb.org/3/genre/movie/list",
      tmdbApiNowPlayingUrl: "https://api.themoviedb.org/3/movie/now_playing",
      tmdbPosterBaseUrl: "http://image.tmdb.org/t/p/w185/",
      tmdbApiNowPlayingPage: 1,
      tmdbApiNowPlayingTotalPages: null,
      maxNumPages: 1,
      apiKey: "b884f44ea27f88e5011e90348d52928d",  // Note: This should be moved out of this client-side code in to an enviromnent variable
      tmdbApiPage: 1,
      numPagesToLoad: 5,
      movies: [],
      genres: [],
      numMovies: 0,
      errors: [],
    }
  },
  computed: {
    
  },
  created() {
    this.loadGenres()
    this.loadMovies()
  },
  methods: {
    loadGenres: function() {
      axios({
      method: 'get',
      url: this.tmdbApiGenreListUrl,
      params: {
        api_key: this.apiKey,
      },
    }).then(response => {
      this.genres = response.data.genres
    }).catch(e => {
      this.errors.push(e)
    })
    },
    loadMovies: function() {
      axios({
      method: 'get',
      url: this.tmdbApiNowPlayingUrl,
      params: {
        api_key: this.apiKey,
        page: this.tmdbApiNowPlayingPage
      },
    }).then(response => {
      this.tmdbApiNowPlayingTotalPages = response.data.total_pages
      this.movies.push(...response.data.results)
      this.numMovies = this.movies.length
      if (this.tmdbApiNowPlayingPage < this.maxNumPages && this.tmdbApiNowPlayingPage < this.tmdbApiNowPlayingTotalPages) {
        this.tmdbApiNowPlayingPage++;
        this.loadMovies()
      }
    }).catch(e => {
      this.errors.push(e)
    })
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 2rem 1rem;
}
.movie-list {
  list-style: none;
  padding: 1rem;
  display: grid;
	grid-template-columns: 1fr;
	grid-gap: 1rem;
	padding: 0;
  margin-top: 1rem;
}
@media (min-width: 480px) {
  .movie-list {
    grid-template-columns: 1fr 1fr;
  }
}
@media (min-width: 690px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
@media (min-width: 930px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1160px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1380px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1610px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
</style>
