<script setup>
import { ref } from 'vue'
const pokemons = ref([])
const nextUrl = ref(null)
const error = ref(null)
const isLoading = ref(true)

const firstLetterUpperCase = (str) => {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

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
      pokemons.value.push(pokemonData)
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
        <div class="pokemon-card" v-for="pokemon in pokemons" :key="pokemon.id">
          <div class="pokemon-image">
            <img :src="pokemon.sprites.other['official-artwork'].front_default" height="140px" width="140px" />
          </div>
          <h5>#{{ pokemon.id.toString().padStart(4, '0') }}</h5>
          <h2>{{ firstLetterUpperCase(pokemon.name) }}</h2>
          <div class="pokemon-types">
            <h4 v-for="(type, index) in pokemon.types" :key="index" :class="`pokemon-type-${type.type.name}`">{{ firstLetterUpperCase(type.type.name) }}</h4>
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
.pokemon-card:hover {
  background-color: #333;
  transform: translateY(-2px);
  transition: background-color 0.3s, box-shadow 0.3s, transform 0.3s;
  color: white;
  cursor: pointer;
}
.pokemon-card h5 {
  color: #707070;
}
.pokemon-card h2 {
  font-weight: 500;
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

.pokemon-type-bug {
  display: flex;
  justify-content: center;
  background-color: #719f3f;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-dragon {
  display: flex;
  justify-content: center;
  background: linear-gradient(#53a4cf 50%, #f16e57 50%);
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-fairy {
  display: flex;
  justify-content: center;
  background-color: #fdb9e9;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-fire {
  display: flex;
  justify-content: center;
  background-color: #fd7d24;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-ghost {
  display: flex;
  justify-content: center;
  background-color: #7b62a3;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-ground {
  display: flex;
  justify-content: center;
  background: linear-gradient(#f7de3f 50%, #ab9842 50%);
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-normal {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #a4acaf;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-psychic {
  display: flex;
  justify-content: center;
  background-color: #f366b9;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-steel {
  display: flex;
  justify-content: center;
  background-color: #9eb7b8;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-dark {
  display: flex;
  justify-content: center;
  background-color: #707070;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-electric {
  display: flex;
  justify-content: center;
  background-color: #eed535;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-fighting {
  display: flex;
  justify-content: center;
  background-color: #d56723;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-flying {
  display: flex;
  justify-content: center;
  background: linear-gradient(#3dc7ef 50%, #bdb9b8 50%);
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-grass {
  display: flex;
  justify-content: center;
  background-color: #9bcc50;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-ice {
  display: flex;
  justify-content: center;
  background-color: #51c4e7;
  color: black;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-poison {
  display: flex;
  justify-content: center;
  background-color: #b97fc8;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-rock {
  display: flex;
  justify-content: center;
  background-color: #a38c21;
  color: white;
  border-radius: 5px;
  width: 85px;
}
.pokemon-type-water {
  display: flex;
  justify-content: center;
  background-color: #4592c4;
  color: white;
  border-radius: 5px;
  width: 85px;
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
