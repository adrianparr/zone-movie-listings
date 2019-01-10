<template>
  <v-card class="movie-list-item">
    <img :src="getFullPosterUrl" :alt="movieData.title">
    <h2>{{movieData.title}}</h2>
    <ul>
      <li v-for="(genre, index) in this.getGenres" :key="index">{{genre}}</li>
    </ul>
    <p>Vote average: {{movieData.vote_average}}</p>
    <p>Popularity: {{movieData.popularity}}</p>
  </v-card>
</template>

<script>
import _ from 'lodash';

export default {
  name: 'MovieListItem',
  props: {
    movieData: Object,
    genreIds: Array,
    posterBaseUrl: String
  },
  computed: {
    getFullPosterUrl: function() {
      return `${this.posterBaseUrl}${this.movieData.poster_path}`
    },
    getGenres: function() {
      let genres = this.movieData.genre_ids
      genres = genres.map(function(id) {
        return _.find(this.genreIds, {'id': id}).name
      }, this);
      genres.sort()
      return genres
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.movie-list-item {
  height: 100%;
  padding: 1rem;
}
</style>
