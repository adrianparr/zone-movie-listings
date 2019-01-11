<template>
  <div id="app">
    <v-app dark>
      <h1 class="display-3 font-weight-light main-heading">Zone Movie Listings</h1>
      <p class="credit"><a href="mailto:adrianparr@gmail.com">by Adrian Parr</a> on Friday 11th January 2019</p>
      <v-expansion-panel>
        <v-expansion-panel-content>
          <div slot="header">Filters</div>
          <v-card>
            <h4 class="filter-heading subheading font-weight-bold">Genres</h4>
            <div class="checkbox-container">
              <v-checkbox v-for="(genre, index) in onlyLoadedGenres" :key="index"
                :label="genre.name"
                :value="genre.id"
                v-model="genre.checked"
                color="primary"
              ></v-checkbox>
            </div>
            <h4 class="filter-heading subheading font-weight-bold">Minimum rating</h4>
            <div class="min-rating-slider-wrapper">
              <v-slider v-model="minimumRating" thumb-label="always" min="0" max="10" step="0.5" @input="onMinimumRatingInput($event)"></v-slider>
            </div>
          </v-card>
        </v-expansion-panel-content>
      </v-expansion-panel>
      <h6 class="subheading results-heading">Results: {{numFilteredMovies}}</h6>
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
      tmdbPosterBaseUrl: "http://image.tmdb.org/t/p/w300/",
      tmdbApiNowPlayingPage: 1,
      tmdbApiNowPlayingTotalPages: null,
      maxNumPages: 5,
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
    filteredMovies: function() {
      if (this.filteredGenreIds.length === 0) {
        return this.allMovies.filter(function(movie) {
          if (movie.vote_average >= this.minimumRating) {
            return movie
          }
        }, this);
      } else {
        return this.allMovies.filter(function(movie) {
          if (movie.vote_average >= this.minimumRating && _.difference(this.filteredGenreIds, movie.genre_ids).length === 0) {
            return movie
          }
        }, this);
      }
    },
    numFilteredMovies: function() {
      return this.filteredMovies.length
    },
    filteredGenreIds: function() {
      let filteredGenreIds = []
      filteredGenreIds = this.onlyLoadedGenres.map(function(genre) {
        if (genre.checked) {
          return genre.id
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
      } else {
        this.finishedLoadingMovies();
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
          'value': null,
          'checked': false
        }
      }, this);
      this.onlyLoadedGenres = _.sortBy(this.onlyLoadedGenres, ['name'])
    },
    finishedLoadingMovies: function() {
      this.onlyLoadedGenres.forEach(function (genre) {
        genre.checked = null;
      });
    }
  }
};
</script>

<style lang="scss" scoped>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.application {
  padding: 1rem;
}
.credit {
  text-align: center;
}
.credit a {
  color: inherit;
}
.main-heading {
  text-align: center;
}
.checkbox-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  padding: 0 1rem;
}
.filter-heading {
  padding: 0 1rem;
  margin-top: 1rem;
}
.min-rating-slider-wrapper {
  padding: 0 1rem;
  margin-top: 2.5rem;
}
.results-heading {
  margin-top: 1rem;
}
.movie-list {
  display: grid;
  grid-gap: 1rem;
  grid-template-columns: 1fr;
  list-style: none;
  margin-top: 1rem;
  padding: 0;
}
@media (min-width: 480px) {
  .movie-list {
    grid-template-columns: 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
@media (min-width: 690px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 930px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1160px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1380px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
@media (min-width: 1610px) {
  .movie-list {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
  .checkbox-container {
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
}
</style>