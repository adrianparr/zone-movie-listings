<template>
  <v-card class="movie-list-item">
    <v-img :src="getFullPosterUrl" :alt="movieData.title">
      <v-layout align-end fill-height pa-3 white--text class="movie-list-item-img-layout">
        <h3 class="title font-weight-light" fluid>{{movieData.title}}</h3>
      </v-layout>
    </v-img>
    <v-divider></v-divider>
    <ul class="movie-genre-list">
      <li v-for="(genre, index) in this.getGenres" :key="index" class="movie-genre-list-item">
        <v-chip class="movie-genre-chip" small>{{genre}}</v-chip>
      </li>
    </ul>
    <p class="vote-average">{{movieData.vote_average}}</p>
    <!-- <p>Popularity: {{movieData.popularity}}</p> -->
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
<style scoped>
.movie-list-item {
  height: 100%;
  padding: 1rem;
}
.movie-list-item-img-layout {
  background: linear-gradient(to bottom, hsla(0,0%,30%,0) 0%, hsla(0,0%,23%,0) 29%, hsla(0,0%,7%,0.81) 100%);
  width: 100%;
}
.movie-genre-list {
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  margin-top: 1rem;
}
.movie-genre-list-item {
  display: inline-block;
}
.vote-average {
  align-items: center;
  background-color: hsla(0, 0%, 33%, 0.9);
  border-radius: 50% !important;
  color: #fff;
  display: flex;
  font-size: 0.8rem;
  font-weight: bold;
  height: 2rem;
  justify-content: center;
  left: 1.5rem;
  line-height: 1.2rem;
  margin-bottom: 0;
  padding: 0;
  position: absolute;
  top: 1.5rem;
  width: 2rem;
}
</style>
<style>
  .movie-genre-chip .v-chip__content {
    padding: 2px 8px 0 8px;
    height: 17px;
    line-height: 1;
  }
  .movie-genre-chip.v-chip--small {
    height: auto !important;
  }
</style>