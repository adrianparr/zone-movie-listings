<template>
  <div id="app">
    <v-app id="inspire" px-2>
      <h1>Zone Movie Listings</h1>
      <v-card class="movie-list-item">
        <div class="checkbox-container">
          <v-checkbox v-for="(genre, index) in onlyLoadedGenres" :key="index"
            :label="genre.name"
            :value="genre.id"
            v-model="genre.value"
            color="primary"
          ></v-checkbox>
        </div>
        <h2>Minimum rating: {{minimumRating}}</h2>
        <input type="range" id="minimum-rating-slider" name="minimum-rating-slider" min="0" max="10" step="0.5" :value="minimumRating" @input="onMinimumRatingInput($event)">
      </v-card>
      <p>Number of movies: {{numFilteredMovies}}</p>
      <ul id="movie-list" class="movie-list">
        <li v-for="movie in filteredMovies" :key="movie.id">
          <MovieListItem :movie-data="movie" :genre-ids="genres" :poster-base-url="tmdbPosterBaseUrl"></MovieListItem>
        </li>
      </ul>
    </v-app>
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
      allMovies: [],
      genres: [],
      numMovies: 0,
      numGenres: 0,
      errors: [],
      minimumRating: 3,
      onlyLoadedGenres: []
    }
  },
  computed: {
    firstThreeMovies: function() {
      return this.allMovies.slice(0, 3);
    },
    filteredMovies: function() {
      return this.allMovies.filter(function(movie) {
        const arrayDifferenceLength = _.difference(movie.genre_ids, this.filteredGenreIds).length
        const movieGenreLength = movie.genre_ids.length
        if (movie.vote_average >= this.minimumRating && arrayDifferenceLength !== movieGenreLength) {
          return movie
        }
      }, this);
    },
    numFilteredMovies: function() {
      return this.filteredMovies.length
    },
    filteredGenreIds: function() {
      let filteredGenreIds = []
      filteredGenreIds = this.onlyLoadedGenres.map(function(genre) {
        if (genre.value !== null) {
          return genre.id
        } else {

        }
      }, this)
      filteredGenreIds = _.compact(filteredGenreIds);
      return filteredGenreIds
    }
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
      this.numGenres = this.genres.length
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
      this.allMovies.push(...response.data.results)
      this.numMovies = this.allMovies.length
      if (this.genres.length > 0) {
        this.updateLoadedGenres()
      }
      if (this.tmdbApiNowPlayingPage < this.maxNumPages && this.tmdbApiNowPlayingPage < this.tmdbApiNowPlayingTotalPages) {
        this.tmdbApiNowPlayingPage++;
        this.loadMovies()
      }
    }).catch(e => {
      this.errors.push(e)
    })
    },
    onMinimumRatingInput: function(e) {
      this.minimumRating = e.target.value
    },
    updateLoadedGenres: function() {
      let onlyLoadedGenreIds = []
      this.allMovies.forEach(function (movie) {
        onlyLoadedGenreIds.push(...movie.genre_ids)
      });
      onlyLoadedGenreIds = _.uniq(onlyLoadedGenreIds);
      this.onlyLoadedGenres = onlyLoadedGenreIds.map(function(id) {
        return {
          'id': id,
          'name': _.find(this.genres, {'id': id}).name,
          'value': id,
        }
      }, this);
      this.onlyLoadedGenres = _.sortBy(this.onlyLoadedGenres, ['name'])
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
  // padding: 2rem 1rem;
  background-color: #eeece8;
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



// var onlyLoadedGenreIds = []
//       this.allMovies.forEach(function (movie) {
//         onlyLoadedGenreIds.push(...movie.genre_ids)
//       });
//       onlyLoadedGenreIds = _.uniq(onlyLoadedGenreIds);
//       onlyLoadedGenreIds = onlyLoadedGenreIds.map(function(id) {
//         return {
//           'id': id,
//           'name': _.find(this.genres, {'id': id}).name,
//           'checked': true,
//         }
//       }, this);
//       onlyLoadedGenreIds = _.sortBy(onlyLoadedGenreIds, ['name'])
//       return onlyLoadedGenreIds.sort()