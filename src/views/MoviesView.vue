<template>
  <div>
    <div class="banner">
      <video src="../assets/img/movie/YouCut_20241220_130412694.mp4" autoplay loop muted></video>
      <div class="mainHome">
        <div class="titleHome">
          <h1>BENSA STREETWEAR</h1>
        </div>
        <div class="descriptionHome">
          <p>
            Lorem ipsum dolor sit amet consectetur adipisicing elit. Laboriosam eaque, praesentium rerum cupiditate saepe, blanditiis nulla aut ut perferendis dignissimos dolorum voluptate? Omnis odio inventore consectetur, architecto aut quam fugit.
          </p>
        </div>
        <div class="buttonsHome">
          <button><img src="../assets/img/botao-play.png" alt="Play" />Assistir</button>
          <button><img src="../assets/img/more-information.png" alt="Mais Info" />Mais Info</button>
        </div>
      </div>
    </div>
    <div class="container">
      <div class="layout">
        <section class="filter-section">
          <label for="genreSelect">Filtrar por Gênero:</label>
          <div class="select-wrapper">
            <select id="genreSelect" v-model="selectedGenre" @change="onGenreChange">
              <option value="" disabled>Selecione um gênero</option>
              <option v-for="genre in genres" :key="genre.id" :value="genre.id">
                {{ genre.name }}
              </option>
            </select>
          </div>
        </section>

    
        <header class="explore-header">
          <h1> {{ selectedGenreName ? `${selectedGenreName}` : 'Filmes' }}</h1>
        </header>
      </div>

      <div v-if="isLoading && movies.length === 0" class="loading">
        <p>Carregando...</p>
      </div>
      <section v-else class="movie-section">
        <div class="movie-grid">
          <div v-for="movie in movies" :key="movie.id" class="movie-card">
            <button @click="visualizar('movie', movie.id)">
            <img
              :src="movie.poster_path ? `https://image.tmdb.org/t/p/w780${movie.poster_path}` : 'https://via.placeholder.com/342x513?text=Sem+Imagem'"
              :alt="movie.title"
              loading="lazy"
            />
            <div class="movie-overlay">
              <h3>{{ movie.title }}</h3>
              <p>Data de Lançamento: {{ formatDate(movie.release_date) }}</p>
              <p>Avaliação: {{ movie.vote_average.toFixed(1) }}/10</p>
            </div>
            </button>
          </div>
        </div>
        <div v-if="isLoadingMore" class="loading-more">
          <p>Carregando mais filmes...</p>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import api from '../plugins/axios';
import { useRouter } from 'vue-router' // Para navegar entre as páginas

const genres = ref([]);
const movies = ref([]);
const selectedGenre = ref(null);
const selectedGenreName = ref('');
const page = ref(1);
const isLoading = ref(false);
const isLoadingMore = ref(false);
let isFetching = false;

const router = useRouter()

function visualizar(type, id) {
  router.push(`/${type}/${id}`); // Navega para a página com o tipo e ID
}

onMounted(async () => {
  try {
    const response = await api.get('genre/movie/list', {
      params: { language: 'pt-BR' },
    });
    genres.value = response.data.genres;
  } catch (error) {
    console.error('Erro ao carregar os gêneros:', error);
  }
});

const loadMovies = async (genreId, reset = false) => {
  if (isFetching) return;
  isFetching = true;

  if (reset) {
    movies.value = [];
    page.value = 1;
    isLoading.value = true;
  } else {
    isLoadingMore.value = true;
  }

  try {
    const response = await api.get('discover/movie', {
      params: {
        with_genres: genreId,
        language: 'pt-BR',
        page: page.value,
      },
    });
    movies.value = reset ? response.data.results : [...movies.value, ...response.data.results];
    page.value++;
  } catch (error) {
    console.error('Erro ao carregar filmes:', error);
  } finally {
    isLoading.value = false;
    isLoadingMore.value = false;
    isFetching = false;
  }
};

const onGenreChange = () => {
  const selected = genres.value.find((g) => g.id === selectedGenre.value);
  selectedGenreName.value = selected ? selected.name : '';
  loadMovies(selectedGenre.value, true);
};

const formatDate = (date) => (date ? new Date(date).toLocaleDateString('pt-BR') : 'N/A');

window.addEventListener('scroll', () => {
  if (window.innerHeight + window.scrollY >= document.body.offsetHeight - 100) {
    loadMovies(selectedGenre.value);
  }
});
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Arial', sans-serif;
}

.banner {
  position: relative;
  width: 100%;
  height: 80dvh;
  overflow: hidden;
}
.divider {
  height: 2px;
  background-color: #444;
  top: 50%;
  border-radius: 1px;
}


.banner video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.mainHome {
  position: absolute;
  top: 50%;
  left: 20%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  text-align: center;
  z-index: 10;
  color: white;
}

.titleHome h1 {
  font-size: 5rem;
}

.descriptionHome p {
  font-size: 1rem;
  color: #ffffffb3;
  max-width: 600px;
  line-height: 2.0;
}

.buttonsHome {
  display: flex;
  gap: 1rem;
}

.buttonsHome button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  background-color: transparent;
  color: #fff;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;
  transition: transform 0.3s;
}

.buttonsHome button:hover {
  transform: translateY(-3px);
}

.container {
  padding: 3rem;
  background-color: #121212;
  color: #fff;
}

.layout {
  display: flex;
  justify-content: space-between; 
  align-items: center;
  gap: 1rem; 
}


.explore-header {
  position: absolute;
  top: 85%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  font-size: 2.5rem;
  color: #fff;
  z-index: 2;
}


.filter-section {
  position: absolute;
  top: 83%;
  left: 5%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 0.5rem;
}

.select-wrapper select {
  padding: 0.8rem;
  font-size: 1rem;
  border-radius: 0.5rem;
  background-color: #1c1c1c;
  color: #fff;
  cursor: pointer;
  width: 250px;
  transition: all 0.3s;
  appearance: none;
}
.select-wrapper {
  position: relative;
}

.select-wrapper:after {
  content: '▼'; 
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: #fff;
  pointer-events: none;
}

select:focus {
  overflow: visible; 
}

select optgroup, select option {
  direction: ltr; 
}

.explore-header {
  text-align: center;
  font-size: 3rem;
  color: #fff;
  margin-top: 2rem;
}


.movie-grid {
  display: grid;
  grid-template-columns: repeat(4, 342px);
  justify-content: center;
  gap: 1rem;
  margin-top: 130px;
}

.movie-card {
  position: relative;
  overflow: hidden;
  border-radius: 0.5rem;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transition: all 0.5s ease-in-out;
}

.movie-card img {
  width: 342px;
  height: 500px;
  object-fit: cover;
}

.movie-card:hover {
  opacity: .5;
}

.movie-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  opacity: 0;
  transition: opacity 0.3s;
  color: #fff;
}

.movie-card:hover .movie-overlay {
  opacity: 1;
}

.loading,
.loading-more {
  text-align: center;
  color: #fff;
  margin-top: 1.5rem;
}

button{
  background: none;
  cursor: pointer;
}
</style>
