<script setup>
import { ref } from 'vue'
const pokemons = ref([])
const nextUrl = ref(null)
const error = ref(null)
const isLoading = ref(true)

const fetchPokemon = async (url) => {
  try {
    isLoading.value = true
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    nextUrl.value = data.next

    const pokemonFetch = data.results.map(pokemon => fetch(pokemon.url))
    const pokemonResponses = await Promise.all(pokemonFetch)

    for (const res of pokemonResponses) {
      if (!res.ok) {
        throw new Error('Network response was not ok')
      }
      const pokemonData = await res.json()
      pokemons.value.push({
        name: pokemonData.name,
        number: pokemonData.id,
        image: pokemonData.sprites.front_default,
        types: pokemonData.types.map(type => type.type.name)
      })
    }
  } catch (err) {
    error.value = err.message
  } finally {
    isLoading.value = false
  }
}

fetchPokemon('https://pokeapi.co/api/v2/pokemon?limit=12&offset=0')
</script>

<template>
  <div class="main-container">
    <h1>pokedex</h1>
    <div class="pokemon-list-wrapper">
      <div class="pokemon-card-container">
        <div class="pokemon-card" v-for="pokemon in pokemons" :key="pokemon.name">
          <div class="pokemon-image">
            <img :src="pokemon.image" />
          </div>
          <h4>#{{ pokemon.number.toString().padStart(4, '0') }}</h4>
          <h2>{{ pokemon.name }}</h2>
          <div class="pokemon-types">
            <h3 v-for="(type, index) in pokemon.types" :key="index">{{ type }}</h3>
          </div>
        </div>
      </div>
      <button class="load-more-button" @click="fetchPokemon(nextUrl)" :class="{ loading: isLoading }" :disabled="isLoading">
        <span v-if="isLoading">Loading...</span>
        <span v-else>Load more Pokemon</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
.main-container {
  padding: 20px;
}

.pokemon-list-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.pokemon-card-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  gap: 20px;
}

.pokemon-card {
  background-color: #222;
  border-radius: 8px;
  padding: 10px;
  width: 200px;
}

.pokemon-image {
  display: flex;
  background-color: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  justify-content: center;
  align-items: center;
  height: 150px;
}

.pokemon-types {
  display: flex;
  gap: 10px;
}

.load-more-button {
  background-color: #30a7d7;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}
.load-more-button:hover {
  background-color: #1b82b1;
}
.load-more-button.loading {
  background-color: #1b82b1;
  cursor: default;
}
</style>
