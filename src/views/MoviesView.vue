<script setup>
import { ref, onMounted, watch } from 'vue';
import api from '../plugins/axios';

const genres = ref([]);
const mostrarLista = ref(false);  // Estado para controlar a exibição da lista de gêneros
const searchQuery = ref('');  // Variável para armazenar o termo de busca

onMounted(async () => {
  const response = await api.get('genre/movie/list?language=pt-BR');
  genres.value = response.data.genres;
});

const movies = ref([]);
const allMovies = ref([]);  // Armazenar todos os filmes inicialmente

const listMovies = async (genreId) => {
  const response = await api.get('discover/movie', {
    params: {
      with_genres: genreId,
      language: 'pt-BR'
    }
  });
  movies.value = response.data.results;
  allMovies.value = response.data.results;  // Armazenar todos os filmes para pesquisa
};

// Método para alternar a exibição da lista
const toggleLista = () => {
  mostrarLista.value = !mostrarLista.value;
};

// Método para filtrar os filmes com base na pesquisa
const filterMovies = () => {
  if (!searchQuery.value.trim()) {
    movies.value = allMovies.value;  // Se não houver pesquisa, mostra todos os filmes
  } else {
    movies.value = allMovies.value.filter(movie =>
      movie.title.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  }
};

// Assista à pesquisa e filtre os filmes sempre que o usuário digitar
watch(searchQuery, filterMovies);

</script>

<template>
  <div class="container">
    <div class="title">
      <h1 class="page-title">Gêneros de Filmes</h1>
    </div>

    <div class="content">
      <!-- Container de Gêneros -->
      <div class="GenerosContainer">
        <button @click="toggleLista" class="toggle-button">
          {{ mostrarLista ? 'Esconder Gêneros' : 'Mostrar Gêneros' }}
        </button>

        <!-- Barra de pesquisa -->
        <div v-if="mostrarLista">
          <input 
            v-model="searchQuery" 
            type="text" 
            class="search-input" 
            placeholder="Pesquise um filme..."
          />
        </div>

        <!-- Lista de gêneros de filmes que só aparece quando 'mostrarLista' é verdadeiro -->
        <ul v-if="mostrarLista" class="genre-list">
          <li v-for="genre in genres" :key="genre.id" @click="listMovies(genre.id)" class="genre-item">
            {{ genre.name }}
          </li>
        </ul>
      </div>

      <!-- Container de Filmes -->
      <div class="movie-list">
        <div v-for="movie in movies" :key="movie.id" class="movie-card">
          <img :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`" :alt="movie.title" />
          <div class="movie-details">
            <p class="movie-title">{{ movie.title }}</p>
            <p class="movie-release-date">{{ movie.release_date }}</p>
            <p class="movie-genres">{{ movie.genre_ids.join(', ') }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  font-family: 'Arial', sans-serif;
  background-color: #121212;
  color: #fff;
  padding: 20px;
}

.title {
  text-align: center;
  font-size: 1rem;
  font-weight: bold;
  margin-top: 30px;
  padding: 20px;
  margin-bottom: 1.5rem;
  color: white;
}

/* Flexbox para organizar GenerosContainer e Movie-List lado a lado */
.content {
  display: flex;
  justify-content: space-between;
  gap: 2rem;
  flex-wrap: wrap;
}

/* Estilo do botão para alternar a lista */
.toggle-button {
  background-color: #3261e4;
  color: white;
  font-size: 1rem;
  padding: 10px 20px;
  border-radius: 25px;
  border: none;
  cursor: pointer;
  margin-bottom: 1rem;
  transition: background-color 0.3s ease;
}

.toggle-button:hover {
  background-color: #2a4eb3;
}

/* Estilo da barra de pesquisa */
.search-input {
  width: 100%;
  padding: 10px;
  margin-top: 1rem;
  margin-bottom: 1rem;
  border-radius: 5px;
  border: 1px solid #ccc; /* Borda padrão */
  background-color: #1f1f1f;
  color: #fff;
  font-size: 1rem;
  outline: none;
}

.search-input::placeholder {
  color: #bbb;
}

/* Estilo da lista de gêneros com itens mais compactos */
.genre-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  margin-top: 1rem;
}

.genre-item {
  background-color: #1f1f1f;
  padding: 8px 12px;
  margin: 5px 0;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  text-align: center;
  text-transform: uppercase;
  font-size: 0.9rem;
}

.genre-item:hover {
  background-color: #3261e4;
}

/* Estilo da lista de filmes */
.movie-list {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  justify-content: center;
  flex: 1;
}

/* Estilo dos cartões de filmes */
.movie-card {
  width: 15rem;
  height: 30rem;
  border-radius: 0.5rem;
  overflow: hidden;
  box-shadow: 0 0 0.5rem #000;
  background-color: #1f1f1f;
  transition: transform 0.3s ease;
}

.movie-card:hover {
  transform: scale(1.05);
}

.movie-card img {
  width: 100%;
  height: 20rem;
  object-fit: cover;
  border-radius: 0.5rem;
  box-shadow: 0 0 0.5rem #000;
}

.movie-details {
  padding: 0.5rem;
}

.movie-title {
  font-size: 1.1rem;
  font-weight: bold;
  margin: 0.5rem 0;
  color: #fff;
  height: 3.2rem;
  line-height: 1.3rem;
}

.movie-release-date {
  font-size: 0.9rem;
  color: #aaa;
}

.movie-genres {
  font-size: 0.9rem;
  color: #bbb;
}

@media (max-width: 768px) {
  .movie-card {
    width: 12rem;
    height: 25rem;
  }

  .page-title {
    font-size: 1.5rem;
  }

  .genre-item {
    font-size: 0.8rem;
  }

  /* Ajuste para telas menores */
  .content {
    flex-direction: column;
    align-items: center;
  }

  .GenerosContainer {
    width: 100%;
  }

  .movie-list {
    width: 100%;
  }
}
</style>
