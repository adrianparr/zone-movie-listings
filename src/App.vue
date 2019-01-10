<template>
  <div id="app">
    <h1>Zone Movie Listings</h1>
    <ul id="movie-list" class="movie-list">
      <li v-for="movie in movies">
        <MovieListItem :movie-data="movie" :poster-base-url="tmdbPosterBaseUrl"></MovieListItem>
      </li>
    </ul>
  </div>
</template>

<script>
import MovieListItem from "./components/MovieListItem.vue";
import axios from 'axios';
import _ from 'lodash';

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

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
