<script setup>
import { ref, reactive, computed } from 'vue';
import axios from 'axios';

const searchInput = ref('');
const searchResults = ref([]);
const selectedMovieData = ref(null);
const releaseData = ref(null);
const loading = ref(false);

const omdbApiKey = 'YOUR_OMDB_API_KEY'; // Replace with your actual API key
const discogsApiKey = 'YOUR_DISCOGS_API_KEY'

const searchMovies = async () => {
  if (searchInput.value.trim() === '') {
    movies.value = [];
    return;
  }
  loading.value = true;
  try {
    const response = await axios.get(
      `https://www.omdbapi.com/?apikey=${apiKey}&s=${searchInput.value}`
        //`https://www.omdbapi.com/?apikey=${omdbApiKey}&s=${searchInput.value}`
    );
    if (response.data.Search) {
      searchResults.value = response.data.Search;
    } else {
        searchResults.value = [];
    }
  } catch (error) {
    console.error('Error fetching movies:', error);
    searchResults.value = [];
  } finally {
    loading.value = false;
  }
};

const fetchMovieDetails = async (imdbID) => {
    
    try {
    const response = await axios.get(
      `https://www.omdbapi.com/?apikey=${apiKey}&i=${imdbID}&plot=full`
    );
    selectedMovie.value = response.data;
    
  } catch (error) {
    console.error('Error fetching movie details:', error);
    selectedMovie.value = null;
  } finally {
    loading.value = false;
  }
};

const fetchReleaseData = async (movieTitle, movieYear) => {
    releaseData.value = [];
  try {
    const discogsSearchResponse = await axios.get(
            `https://api.discogs.com/database/search?q=${encodeURIComponent(movieTitle)}&year=${movieYear}&format=Blu-ray&type=release&key=${discogsApiKey}&secret=YOUR_DISCOGS_API_SECRET`
          );
    

    if (discogsSearchResponse.data.results && discogsSearchResponse.data.results.length > 0) {
        const releases = discogsSearchResponse.data.results.map((release) => ({
        id: release.id,
        title: release.title,
        year: release.year,
        cover_image: release.cover_image,
        uri: release.uri
        }));
        releaseData.value = releases;
        
    }
    else{
        console.log('no results')
        releaseData.value = [];
    }
  } catch (error) {
    console.error('Error fetching release data:', error);
    releaseData.value = [];
  } 
};


const selectMovie = async (movie) => {
  loading.value = true;
  await fetchMovieDetails(movie.imdbID);
  await fetchReleaseData(movie.Title, movie.Year);
  loading.value = false;
};

const clearSearch = () => {
    searchResults.value = [];
    searchInput.value = '';
    selectedMovieData.value = null;
    releaseData.value = null;
}
</script>
<template>
  <div>
    <h1>Physical Media DB</h1>
    <div>
      <input type="text" v-model="searchInput" placeholder="Search for a movie" />
      <button @click="searchMovies">Search</button>
      <button @click="clearSearch">Clear</button>
      <div v-if="loading">Loading...</div>
    </div>
    <div v-if="searchResults.length > 0">
      <h2>Search Results</h2>
      <ul>
        <li v-for="movie in searchResults" :key="movie.imdbID">
          <button @click="selectMovie(movie)">
            {{ movie.Title }} ({{ movie.Year }})
          </button>
        </li>
      </ul>
    </div>
    <div v-if="selectedMovieData">
      <h2>{{ selectedMovieData.Title }} ({{ selectedMovieData.Year }})</h2>
      <img :src="selectedMovieData.Poster" v-if="selectedMovieData.Poster && selectedMovieData.Poster !== 'N/A'" :alt="`${selectedMovieData.Title} poster`" style="max-width: 200px;">
      <p v-if="selectedMovieData.Plot">{{ selectedMovieData.Plot }}</p>
      <p v-if="selectedMovieData.Released">Released: {{ selectedMovieData.Released }}</p>
      <p v-if="selectedMovieData.DVD">DVD Release: {{ selectedMovieData.DVD }}</p>
      <p v-if="selectedMovieData.Ratings">Ratings:
        <span v-for="rating in selectedMovieData.Ratings" :key="rating.Source">
            {{ rating.Source }}: {{ rating.Value }}
            <template v-if="rating !== selectedMovieData.Ratings[selectedMovieData.Ratings.length - 1]">, </template>
        </span>
      </p>
      <h3>Physical Media Releases:</h3>
       <div v-if="releaseData && releaseData.length > 0">
            <ul>
                <li v-for="release in releaseData" :key="release.id">
                    <a :href="release.uri" target="_blank" rel="noopener noreferrer">
                        <img :src="release.cover_image" :alt="release.title" style="max-width: 100px; vertical-align: middle;">
                        <span>{{ release.title }} ({{ release.year }})</span>
                    </a>
                </li>
            </ul>
        </div>
        <div v-else>
            <p>No physical media releases found on Discogs.</p>
        </div>


      <ul>
        <h3>Other Places to Check for Releases:</h3>
        <li>Check boutique distributors such as:</li>
        <li>Arrow Video</li>
        <li>Criterion Collection</li>
        <li>Shout! Factory</li>
        <li>Severin Films</li>
        <li>Vinegar Syndrome</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.logo{
    padding:1.5em;
}
</style>
