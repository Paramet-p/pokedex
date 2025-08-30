<script setup>
import { ref } from 'vue'
import PokeballSpinner from './components/icons/PokeballSpinner.vue'

const pokemons = ref([])
const nextUrl = ref(null)
const error = ref(null)
const isLoadingPokemons = ref(true)
const isLoadingpokemonPopup = ref(true)
const isLoadingAbility = ref(true)
const showPopup = ref(false)
const selectedPokemon = ref(null)
const selectedPokemonSpecies = ref(null)
const selectedPokemonAbility = ref(null)
const selectedDetailsTab = ref('stats')

const statNames = ['HP', 'Attack', 'Defense', 'Sp. Atk', 'Sp. Def', 'Speed']

const showPokemonDetails = (pokemon) => {
  selectedPokemon.value = pokemon
  showPopup.value = true
  fetchPokemonSpecies()
}

const closePokemonDetails = () => {
  showPopup.value = false
  selectedPokemon.value = null
  selectedPokemonSpecies.value = null
  selectedPokemonAbility.value = null
  selectedDetailsTab.value = 'stats'
}

const fetchPokemonSpecies = async () => {
  try {
    isLoadingpokemonPopup.value = true
    const response = await fetch(selectedPokemon.value.species.url)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    selectedPokemonSpecies.value = data
  } catch (err) {
    error.value = err.message
    throw new Error(err.message)
  } finally {
    isLoadingpokemonPopup.value = false
  }
}

const fetchAbility = async () => {
  try {
    isLoadingAbility.value = true
    const response = await fetch(selectedPokemon.value.abilities[0].ability.url)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    selectedPokemonAbility.value = data
  } catch (err) {
    error.value = err.message
    throw new Error(err.message)
  } finally {
    isLoadingAbility.value = false
  }
}

const firstLetterUpperCase = (str) => {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

const fetchPokemon = async (url) => {
  try {
    isLoadingPokemons.value = true
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
    isLoadingPokemons.value = false
  }
}

fetchPokemon('https://pokeapi.co/api/v2/pokemon?limit=12&offset=0')
</script>

<template>
  <div class="main-container">
    <h1>Pokedex</h1>
    <div class="pokemon-list-wrapper">
      <div class="pokemon-card-container">
        <div class="pokemon-card" v-for="pokemon in pokemons" :key="pokemon.id" @click="showPokemonDetails(pokemon)">
          <div class="pokemon-image">
            <img :src="pokemon.sprites.other['official-artwork'].front_default" :alt="pokemon.name" height="140px"
              width="140px" />
          </div>
          <h5>#{{ pokemon.id.toString().padStart(4, '0') }}</h5>
          <h2>{{ firstLetterUpperCase(pokemon.name) }}</h2>
          <div class="pokemon-types">
            <h4 v-for="(type, index) in pokemon.types" :key="index" :class="`pokemon-type-${type.type.name}`">{{
              firstLetterUpperCase(type.type.name) }}</h4>
          </div>
        </div>
      </div>
      <button class="load-more-button" @click="fetchPokemon(nextUrl)" :class="{ loading: isLoadingPokemons }"
        :disabled="isLoadingPokemons">
        <span v-if="isLoadingPokemons">Loading...</span>
        <span v-else>Load more Pokemon</span>
      </button>
    </div>
  </div>
  <!-- popup -->
  <div v-if="showPopup" class="pokemon-popup-overlay">
    <div class="pokemon-popup">
      <button class="close-button" @click="closePokemonDetails">X</button>
      <!-- <div v-if="isLoadingpokemonPopup" style="color: white; font-size: 1.5rem;">
        Loading...
      </div> -->
      <div v-if="isLoadingpokemonPopup" style="display: flex; justify-content: center; align-items: center; height: 100%;">
        <PokeballSpinner />
      </div>
      <div class="popup-header" v-if="selectedPokemon && !isLoadingpokemonPopup">
        <h1>{{ firstLetterUpperCase(selectedPokemon.name) }}</h1>
        <h1 style="color: #616161">#{{ selectedPokemon.id.toString().padStart(4, '0') }}</h1>
      </div>
      <div v-if="selectedPokemon && !isLoadingpokemonPopup">
        <div class="popup-content">
          <div class="popup-image">
            <img :src="selectedPokemon.sprites.other['official-artwork'].front_default" :alt="selectedPokemon.name"
              width="250px" />
          </div>
          <div style="display: flex; flex-direction: column; gap: 20px;">
            <div class="popup-about">
              <div>
                <div style="margin-bottom: 20px;">
                  <h4>Height</h4>
                  <h3>{{ (selectedPokemon.height / 10).toFixed(1) }} m</h3>
                </div>
                <div>
                  <h4>Weight</h4>
                  <h3>{{ (selectedPokemon.weight / 10).toFixed(1) }} kg</h3>
                </div>
              </div>
              <div>
                <div style="margin-bottom: 20px;">
                  <h4>Category</h4>
                  <h3>{{ selectedPokemonSpecies.genera[7].genus.slice(0, -7) }}</h3>
                </div>
                <div>
                  <h4>Abilities</h4>
                  <div style="display: flex; align-items: baseline; gap: 10px;">
                    <h3 style="cursor: pointer" @click="fetchAbility">
                      {{ firstLetterUpperCase(selectedPokemon.abilities[0].ability.name) }}
                    </h3>
                    <div class="circle-icon" @click="fetchAbility">?</div>
                  </div>
                </div>
              </div>
              <!-- Ability Info -->
              <div class="popup-ability-info" :class="{ 'show': selectedPokemonAbility && !isLoadingAbility }">
                <button @click="selectedPokemonAbility = null">
                  Close
                </button>
                <h5>Ability Info</h5>
                <h3>
                  {{ firstLetterUpperCase(selectedPokemon.abilities[0].ability.name) }}
                </h3>
                <h4>
                  {{ selectedPokemonAbility?.flavor_text_entries?.[selectedPokemonAbility.flavor_text_entries.length -
                    1]?.flavor_text ?? 'No Info' }}
                </h4>
              </div>
            </div>
            <h3 style="text-align: start;">Type</h3>
            <div class="pokemon-types">
              <h4 v-for="(type, index) in selectedPokemon.types" :key="index" :class="`pokemon-type-${type.type.name}`">
                {{ firstLetterUpperCase(type.type.name) }}
              </h4>
            </div>
          </div>
        </div>
      </div>
      <!-- Details -->
      <div v-if="selectedPokemon && !isLoadingpokemonPopup">
        <!-- Details Tabs -->
        <div class="details-tabs">
          <button :class="{'active': selectedDetailsTab === 'stats'}" @click="selectedDetailsTab = 'stats'" >
            Stats
          </button>
          <button :class="{'active': selectedDetailsTab === 'evolutions'}" @click="selectedDetailsTab = 'evolutions'">
            Evolutions
          </button>
        </div>
        <!-- Stats Content -->
        <div class="stats-content" v-if="selectedDetailsTab === 'stats'">
          <div style="display: flex;">
            <div v-for="stat, index in statNames" :key="stat">
              <div class="stat-bar" v-for="value in (15 - Math.ceil(selectedPokemon.stats[index].base_stat / 17))"
                :key="value"></div>
              <div class="stat-bar" style="background-color: #30a7d7;"
                v-for="value in Math.ceil(selectedPokemon.stats[index].base_stat / 17)" :key="value"></div>
              <span style="font-weight: 500;">{{ stat }}</span>
            </div>
          </div>
        </div>
        <!-- Evolutions Content -->
        <div class="evolutions-content" v-else-if="selectedDetailsTab === 'evolutions'">
          evolutions content
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  gap: 20px;
}

.main-container h1 {
  color: #919191;
  font-weight: 500;
  font-size: 2.5rem;
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
  background-color: #ffffff;
  border-radius: 8px;
  padding: 10px;
  width: 200px;
}

.pokemon-card:hover {
  /* background-color: rgba(0, 0, 0, 0.1); */
  transform: translateY(-2px);
  transition: background-color 0.3s ease-out, transform 0.3s ease-out;
  /* color: white; */
  cursor: pointer;
}

.pokemon-card h5 {
  color: #919191;
  font-weight: 500;
}

.pokemon-card h2 {
  color: #313131;
  font-weight: 500;
}

.pokemon-image {
  display: flex;
  background-color: #f2f2f2;
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

.pokemon-popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.pokemon-popup {
  background: #ffffff;
  color: #212121;
  padding: 20px;
  border-radius: 10px;
  position: relative;
  text-align: center;
  width: 660px;
  height: 704px;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: transparent;
  border: none;
  color: #313131;
  font-size: 1.2rem;
  cursor: pointer;
}

.popup-header {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
  gap: 15px;
}

.popup-header h1 {
  font-weight: 500;
}

.popup-content {
  display: flex;
  gap: 20px;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.popup-image {
  background-color: #f2f2f2;
  border-radius: 6px;
  width: 250px;
  height: 250px;
}

.popup-about {
  position: relative;
  display: grid;
  grid-template-columns: 1fr 1fr;
  text-align: left;
  background-color: #30a7d7;
  border-radius: 10px;
  padding: 15px;
  width: 350px;
}

.popup-about h4 {
  color: white;
}

.popup-about h3 {
  color: black;
}

.popup-ability-info {
  position: absolute;
  top: 0;
  left: 0;
  background-color: #313131;
  width: 100%;
  height: 100%;
  border-radius: 10px;
  padding: 15px;
  padding-top: 10px;
  transition: opacity 0.3s ease-out;
  opacity: 0;
  pointer-events: none;
}

.popup-ability-info.show {
  opacity: 1;
  pointer-events: auto;
}

.popup-ability-info h5 {
  color: #616161;
  margin-bottom: 5px;
  font-weight: 500;
}

.popup-ability-info h3 {
  color: #ffffff;
}

.popup-ability-info h4 {
  color: #f2f2f2;
}

.popup-ability-info button {
  position: absolute;
  top: 0;
  right: 0;
  background-color: #000000;
  border: none;
  color: #ffffff;
  border-radius: 0 10px 0 10px;
  padding: 10px;
  padding-left: 15px;
  padding-right: 15px;
  cursor: pointer;
}

.circle-icon {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ffffff;
  width: fit-content;
  border-radius: 100%;
  font-size: 12px;
  font-weight: 500;
  color: #30a7d7;
  width: 15px;
  height: 15px;
  cursor: pointer;
}

.details-tabs {
  display: flex;
  gap: 1px;
}

.details-tabs button {
  border: none;
  border-radius: 10px 10px 0 0;
  padding: 10px 20px;
  cursor: pointer;
  background-color: #d0d0d0;
  color: #515151;
}

.details-tabs button:hover {
  background-color: #c4c4c4;
  color: #000000;
  transition: background-color 0.3s ease-out, color 0.3s ease-out;
}

.details-tabs button.active {
  background-color: #a4a4a4;
  color: #000000;
}

.stats-content {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #a4a4a4;
  border-radius: 0 10px 10px 10px;
  padding: 5px;
  height: 300px;
}

.stat-bar {
  background-color: #ffffff;
  height: 10px;
  width: 80px;
  margin: 5px;
}

.evolutions-content {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #a4a4a4;
  border-radius: 0 10px 10px 10px;
  height: 300px;
}
</style>
