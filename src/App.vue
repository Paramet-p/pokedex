<script setup>
import { ref } from 'vue'
const pokemons = ref([])
const error = ref(null)
const isLoading = ref(true)

const fetchPokemon = async () => {
  try {
    isLoading.value = true
    const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=12&offset=0')
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()

    console.log(data)
    for (const pokemon of data.results) {
      // pokemons.value.push(pokemon)
      const pokemonResponse = await fetch(pokemon.url)
      if (!pokemonResponse.ok) {
        throw new Error('Network response was not ok for ' + pokemon.name)
      }
      const pokemonData = await pokemonResponse.json()
      pokemons.value.push({
        name: pokemon.name,
        number: pokemonData.id,
        image: pokemonData.sprites.front_default,
        types: pokemonData.types.map(type => type.type.name)
      })
    }
    console.log(pokemons.value)
  } catch (err) {
    error.value = err.message
  } finally {
    isLoading.value = false
  }
}
fetchPokemon()
</script>

<template>
  <div>
    <h1>pokedex</h1>
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
  </div>
</template>

<style scoped>
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
}

.pokemon-types {
  display: flex;
  gap: 10px;
}
</style>
