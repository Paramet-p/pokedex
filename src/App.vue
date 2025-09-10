<script setup>
import { ref } from 'vue'
import PokeballSpinner from './components/icons/PokeballSpinner.vue'
import IconSearch from './components/icons/IconSearch.vue'
import DropDown from './components/elements/DropDown.vue'

const allPokemons = ref([])
const startIndex = ref(0)
const endIndex = ref(12)
const pokemons = ref([])
const error = ref(null)
const searchTerm = ref('')
const sortOption = ref('lowest-number')
const sortOptions = [
  { name: 'Lowest Number (First)', value: 'lowest-number' },
  { name: 'Highest Number (First)', value: 'highest-number' },
  { name: 'A-Z', value: 'a-z' },
  { name: 'Z-A', value: 'z-a' }
]
const typeList = [
  'normal', 'fire', 'water', 'electric', 'grass', 'ice', 'fighting', 'poison',
  'ground', 'flying', 'psychic', 'bug', 'rock', 'ghost', 'dragon', 'dark',
  'steel', 'fairy'
]
const selectedTypes = ref([])
const showAdvancedSearch = ref(false)
const isLoadingPokemons = ref(true)
const allAbilities = ref([{ name: 'All', value: 'all' }])
const selectedAbility = ref('all')
const isLoadingAllAbilities = ref(false)
const numberRangeStart = ref(1)
const numberRangeEnd = ref(1025)
const isLoadingpokemonPopup = ref(true)
const isLoadingAbility = ref(true)
const isLoadingEvolutions = ref(true)
const showPopup = ref(false)
const selectedPokemon = ref(null)
const selectedPokemonSpecies = ref(null)
const selectedPokemonAbility = ref(null)
const selectedDetailsTab = ref('stats')
const selectedPokemonEvolutions = ref(null)
const selectedPokemonEvolutionsDetails = ref([])

const statNames = ['HP', 'Attack', 'Defense', 'Sp. Atk', 'Sp. Def', 'Speed']

const loadPokemons = async () => {
  try {
    isLoadingPokemons.value = true
    // filter data by search term
    let filteredPokemons = allPokemons.value
    if (searchTerm.value.trim() !== '') {
      const search = searchTerm.value.toLowerCase().trim()
      filteredPokemons = allPokemons.value.filter(pokemon => pokemon.name.includes(search) || pokemon.id.toString().includes(search))
    }
    // sort data
    switch (sortOption.value) {
      case 'lowest-number':
        filteredPokemons.sort((a, b) => a.id - b.id)
        break
      case 'highest-number':
        filteredPokemons.sort((a, b) => b.id - a.id)
        break
      case 'a-z':
        filteredPokemons.sort((a, b) => a.name.localeCompare(b.name))
        break
      case 'z-a':
        filteredPokemons.sort((a, b) => b.name.localeCompare(a.name))
        break
      default:
        break
    }
    // fetch 12 pokemons details per time
    const pokemonToDisplay = filteredPokemons.slice(startIndex.value, endIndex.value)
    const pokemonPromises = pokemonToDisplay.map(pokemon => fetch(pokemon.url))
    const pokemonResponses = await Promise.all(pokemonPromises)
    // push pokemons data to pokemons array to display
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

const fetchAllPokemons = async () => {
  try {
    isLoadingPokemons.value = true
    const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=1025&offset=0')
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    allPokemons.value = data.results.map((pokemon, index) => {
      return { name: pokemon.name, id: index + 1, url: pokemon.url }
    })
    loadPokemons()
  } catch (err) {
    error.value = err.message
  } finally {
    isLoadingPokemons.value = false
  }
}

const fetchAllAbilities = async () => {
  try {
    isLoadingAllAbilities.value = true
    const response = await fetch('https://pokeapi.co/api/v2/ability?limit=10000&offset=0')
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    let fetchedAbilities = data.results.map((ability) => {
      return { name: capitalizedString(ability.name), value: ability.name }
    })
    fetchedAbilities.sort((a, b) => a.name.localeCompare(b.name))
    allAbilities.value.push(...fetchedAbilities)
  } catch (err) {
    error.value = err.message
  } finally {
    isLoadingAllAbilities.value = false
  }
}

const resetAdvancedSearch = () => {
  selectedTypes.value = []
  selectedAbility.value = 'all'
  numberRangeStart.value = 1
  numberRangeEnd.value = 1025
}

const reSetIndex = () => {
  pokemons.value = []
  startIndex.value = 0
  endIndex.value = 12
}

const searchPokemons = () => {
  reSetIndex()
  loadPokemons()
}

const sortPokemons = () => {
  reSetIndex()
  loadPokemons()
}

const loadMorePokemons = () => {
  startIndex.value += 12
  endIndex.value += 12
  loadPokemons()
}

const selectType = (type) => {
  if (selectedTypes.value.includes(type)) {
    selectedTypes.value = selectedTypes.value.filter(t => t !== type)
  } else {
    selectedTypes.value.push(type)
  }
}

const showPokemonDetails = (pokemon) => {
  selectedPokemon.value = pokemon
  showPopup.value = true
  selectedDetailsTab.value = 'stats'
  selectedPokemonEvolutions.value = null
  selectedPokemonEvolutionsDetails.value = []
  selectedPokemonAbility.value = null
  fetchPokemonSpecies()
}

const closePokemonDetails = () => {
  showPopup.value = false
  selectedPokemon.value = null
  selectedPokemonSpecies.value = null
  selectedPokemonAbility.value = null
  selectedDetailsTab.value = 'stats'
  selectedPokemonEvolutions.value = null
  selectedPokemonEvolutionsDetails.value = []
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

const fetchAbility = async (abilityUrl) => {
  try {
    isLoadingAbility.value = true
    const response = await fetch(abilityUrl)
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

const fetchEvolutions = async () => {
  try {
    isLoadingEvolutions.value = true
    selectedDetailsTab.value = 'evolutions'
    const response = await fetch(selectedPokemonSpecies.value.evolution_chain.url)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    const data = await response.json()
    selectedPokemonEvolutions.value = data

    const evolutionUrls = []
    const chain = data.chain

    evolutionUrls.push(chain.species.name)
    if (chain.evolves_to.length > 0) {
      evolutionUrls.push(chain.evolves_to[0].species.name)
      if (chain.evolves_to[0].evolves_to.length > 0) {
        evolutionUrls.push(chain.evolves_to[0].evolves_to[0].species.name)
      }
    }

    const evolutionPromises = evolutionUrls.map(name => fetch(`https://pokeapi.co/api/v2/pokemon/${name}`))
    const evolutionResponses = await Promise.all(evolutionPromises)

    selectedPokemonEvolutionsDetails.value = []
    for (const res of evolutionResponses) {
      if (!res.ok) {
        throw new Error('Network response was not ok')
      }
      const evoData = await res.json()
      selectedPokemonEvolutionsDetails.value.push(evoData)
    }
  } catch (err) {
    error.value = err.message
    throw new Error(err.message)
  } finally {
    isLoadingEvolutions.value = false
  }
}

const selectDetailTabs = (tab) => {
  selectedDetailsTab.value = tab
  selectedPokemonEvolutionsDetails.value = []
  selectedPokemonEvolutionsDetails.value = []
  if (tab === 'evolutions') {
    fetchEvolutions()
  }
}

const capitalizedString = (str) => {
  if (str) {
    // str have single word
    if (!str.includes('-')) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }
    // str have multiple words
    const words = str.split('-')
    const capitalizedWords = words.map(word => word.charAt(0).toUpperCase() + word.slice(1))
    return capitalizedWords.join(' ')
  }
}

const getEnGenus = (genera) => {
  const enGenus = genera.find(genus => genus.language.name === 'en')
  return enGenus ? enGenus.genus.slice(0, -7) : ''
}

fetchAllPokemons()
</script>

<template>
  <div class="main-container">
    <h1>Pokedex</h1>
    <!-- Search -->
    <div class="search-filter-container">
      <div class="search-container">
        <div class="search-wrapper">
          <h2>Name or Number</h2>
          <div class="search-bar">
            <input v-model="searchTerm" @keypress.enter="searchPokemons" />
            <button @click="searchPokemons">
              <IconSearch />
            </button>
          </div>
          <h4>Use the Advanced Search to explore Pokemon by type</h4>
        </div>
      </div>
      <!-- advanced search -->
      <div class="search-advanced" :class="{ active: showAdvancedSearch }">
        <div class="search-advanced-wrapper">
          <!-- type filter -->
          <div class="search-advanced-type">
            <span style="margin-bottom: 0px;">Type</span>
            <h4 v-for="(type, index) in typeList" :key="index" :class="`pokemon-type-${type}`"
              style="position: relative; width: 140px; height: 30px;">
              <div class="search-advanced-type-selector" :class="{ active: selectedTypes.includes(type) }"
                @click="selectType(type)"></div>
              {{ capitalizedString(type) }}
            </h4>
          </div>
          <div>
            <!-- ability filter -->
            <div style="display: flex; flex-direction: column; margin-bottom: 20px;">
              <span>Ability</span>
              <DropDown :options="allAbilities" width="100%" v-model:selected="selectedAbility" />
            </div>
            <!-- number range filter -->
            <div style="display: flex; flex-direction: column;">
              <span>Number Range</span>
              <div class="number-range-input">
                <input v-model="numberRangeStart" />
                <span> - </span>
                <input v-model="numberRangeEnd" />
              </div>
            </div>
          </div>
          <div class="group-button">
            <button class="button-reset" @click="resetAdvancedSearch()">Reset</button>
            <button class="button-search">Search</button>
          </div>
        </div>
      </div>
      <div class="search-advanced-footer" @click="showAdvancedSearch = !showAdvancedSearch; fetchAllAbilities()">
        <span>
          Show Advanced Search
        </span>
      </div>
    </div>
    <!-- Sort Button -->
    <div class="sort-container">
      Sort By
      <DropDown :options="sortOptions" v-model:selected="sortOption"
        @update:selected="sortPokemons" />
    </div>
    <!-- Pokemon List -->
    <div class="pokemon-list-wrapper">
      <div class="pokemon-card-container">
        <div class="pokemon-card" v-for="pokemon in pokemons" :key="pokemon.id" @click="showPokemonDetails(pokemon)">
          <div class="pokemon-image">
            <img :src="pokemon.sprites.other['official-artwork'].front_default" :alt="pokemon.name" height="140px"
              width="140px" />
          </div>
          <h5>#{{ pokemon.id.toString().padStart(4, '0') }}</h5>
          <h2>{{ capitalizedString(pokemon.name) }}</h2>
          <div class="pokemon-types">
            <h4 v-for="(type, index) in pokemon.types" :key="index" :class="`pokemon-type-${type.type.name}`">{{
              capitalizedString(type.type.name) }}</h4>
          </div>
        </div>
      </div>
      <!-- Load More Button -->
      <button class="load-more-button" @click="loadMorePokemons()" :class="{ loading: isLoadingPokemons }"
        :disabled="isLoadingPokemons" v-if="pokemons.length < allPokemons.length">
        <span class="spinner-wrapper" v-if="isLoadingPokemons">
          <PokeballSpinner primaryColor="#dddddd" secondaryColor="#1b82b1" size="20px" />
        </span>
        <span v-else>Load more Pokemon</span>
      </button>
    </div>
  </div>
  <!-- popup -->
  <div v-if="showPopup" class="pokemon-popup-overlay">
    <div class="pokemon-popup">
      <button class="close-button" @click="closePokemonDetails">X</button>
      <div v-if="isLoadingpokemonPopup"
        style="display: flex; justify-content: center; align-items: center; height: 100%;">
        <PokeballSpinner />
      </div>
      <!-- popup header -->
      <div class="popup-header" v-if="selectedPokemon && !isLoadingpokemonPopup">
        <h1>{{ capitalizedString(selectedPokemon.name) }}</h1>
        <h1 style="color: #616161">#{{ selectedPokemon.id.toString().padStart(4, '0') }}</h1>
      </div>
      <!-- popup content -->
      <div v-if="selectedPokemon && !isLoadingpokemonPopup">
        <div class="popup-content">
          <!-- pokemon image -->
          <div class="popup-image">
            <img :src="selectedPokemon.sprites.other['official-artwork'].front_default" :alt="selectedPokemon.name"
              width="250px" />
          </div>
          <!-- about pokemon -->
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
                  <h3>{{ getEnGenus(selectedPokemonSpecies.genera) }}</h3>
                </div>
                <div>
                  <h4>Abilities</h4>
                  <div v-for="ability in selectedPokemon.abilities" :key="ability.ability.name"
                    style="display: flex; align-items: baseline; gap: 10px;">
                    <h3 v-if="!ability.is_hidden" style="cursor: pointer" @click="fetchAbility(ability.ability.url)">
                      {{ capitalizedString(ability.ability.name) }}
                    </h3>
                    <div v-if="!ability.is_hidden" class="circle-icon" @click="fetchAbility(ability.ability.url)">?
                    </div>
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
                  {{ capitalizedString(selectedPokemonAbility?.name) }}
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
                {{ capitalizedString(type.type.name) }}
              </h4>
            </div>
          </div>
        </div>
      </div>
      <!-- Details -->
      <div v-if="selectedPokemon && !isLoadingpokemonPopup">
        <!-- Details Tabs -->
        <div class="details-tabs">
          <!-- Stats Tab -->
          <button :class="{ 'active': selectedDetailsTab === 'stats' }" @click="selectDetailTabs('stats')">
            Stats
          </button>
          <!-- Evolutions Tab -->
          <button :class="{ 'active': selectedDetailsTab === 'evolutions' }" @click="selectDetailTabs('evolutions')">
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
              <span style="font-weight: 500; color: #ffffff">{{ stat }}</span>
            </div>
          </div>
        </div>
        <!-- Evolutions Content -->
        <div class="evolutions-content" v-else-if="selectedDetailsTab === 'evolutions'">
          <div v-if="isLoadingEvolutions">
            <PokeballSpinner primaryColor="#9f9f9f" secondaryColor="#595959" />
          </div>
          <div class="evolutions-container" v-if="selectedPokemonEvolutionsDetails && !isLoadingEvolutions">
            <div class="evolutions-pokemon" v-for="pokemon, index in selectedPokemonEvolutionsDetails"
              :key="pokemon.id">
              <div class="evolutions-pokemon-details" @click="showPokemonDetails(pokemon)">
                <div class="evolutions-pokemon-frame">
                  <img :src="pokemon.sprites.other['official-artwork'].front_default" :alt="pokemon.name"
                    height="130px" />
                </div>
                <div class="evolutions-pokemon-name">
                  <h3>{{ capitalizedString(pokemon.name) }}</h3>
                  <h3 style="color: #a4acaf;">#{{ pokemon.id.toString().padStart(4, '0') }}</h3>
                </div>
                <div class="pokemon-types" style="gap: 2px;">
                  <div v-for="(type, index) in pokemon.types" :key="index" :class="`pokemon-type-${type.type.name}`"
                    style="width: 70px; font-size: 0.8rem;">
                    {{ capitalizedString(type.type.name) }}
                  </div>
                </div>
              </div>
              <div v-if="index < selectedPokemonEvolutionsDetails.length - 1" style="font-size: 2rem;">
                >
              </div>
            </div>
          </div>
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
  gap: 20px;
  width: 100%;
}

.main-container h1 {
  color: #616161;
  font-weight: 500;
  font-size: 2.5rem;
}

.search-filter-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.search-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: #313131;
  width: 100%;
  padding: 20px 0;
}

.search-container h2 {
  color: #ffffff;
  margin-bottom: 10px;
}

.search-container h4 {
  color: #ffffff;
  margin-top: 10px;
}

.search-wrapper {
  display: flex;
  flex-direction: column;
  width: 860px;
}

.search-bar {
  display: flex;
  gap: 10px;
}

.search-bar input {
  border: 2px solid #616161;
  border-radius: 5px;
  height: 42px;
  width: 250px;
  padding-left: 10px;
  font-size: 1rem;
}

.search-bar button {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ee6b2f;
  border: none;
  height: 42px;
  width: 48px;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease-out;
}

.search-bar button:hover {
  background-color: #da471b;
}

.search-advanced {
  display: flex;
  justify-content: center;
  background-color: #616161;
  width: 100%;
  height: 40px;
  transition: height 0.3s ease-out;
  padding-top: 40px;
  color: #ffffff;
  overflow: hidden;
  user-select: none;
}

.search-advanced.active {
  height: 400px;
  transition: height 0.3s ease-out;
}

.search-advanced-wrapper {
  width: 860px;
  display: grid;
  grid-template-columns: 2fr 1fr;
}

.search-advanced-wrapper span {
  grid-column: span 3;
  font-weight: 500;
  font-size: 1.2rem;
  margin-bottom: 10px;
}

.search-advanced-type {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 10px;
}

.search-advanced-type-selector {
  position: absolute;
  background-color: rgba(0, 0, 0, 0.6);
  width: 100%;
  height: 100%;
  border-radius: 5px;
  cursor: pointer;
}

.search-advanced-type-selector.active {
  background-color: transparent;
  box-shadow: 0 0 6px 2px rgba(255, 255, 255, 0.7);
}

.number-range-input {
  display: flex;
  align-items: center;
  gap: 10px;
}

.number-range-input input {
  border: none;
  border-radius: 5px;
  height: 42px;
  width: 90px;
  padding-left: 10px;
  font-size: 1rem;
}

.group-button {
  display: flex;
  justify-content: flex-end;
  grid-column: span 2;
  margin-bottom: 20px;
}

.button-reset {
  background-color: #a4a4a4;
  border: none;
  color: white;
  height: 46px;
  padding: 0 26px;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 20px;
  margin-right: 10px;
  transition: background-color 0.2s ease-out;
  font-weight: 500;
  font-size: 1rem;
}

.button-reset:hover {
  background-color: #8b8b8b;
}

.button-search {
  background-color: #ee6b2f;
  border: none;
  color: white;
  height: 46px;
  padding: 0 26px;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 20px;
  transition: background-color 0.2s ease-out;
  font-weight: 500;
  font-size: 1rem;
}

.button-search:hover {
  background-color: #da471b;
}

.search-advanced-footer {
  display: flex;
  justify-content: center;
  position: relative;
  background-color: #616161;
  width: 400px;
  height: 20px;
  cursor: pointer;
}

.search-advanced-footer span {
  position: absolute;
  top: -10px;
  color: #ffffff;
  font-weight: 500;
  user-select: none;
}

.search-advanced-footer::before {
  content: '';
  position: absolute;
  left: -20px;
  width: 20px;
  height: 20px;
  background-color: #616161;
  border-radius: 0 0 0 20px;
}

.search-advanced-footer::after {
  content: '';
  position: absolute;
  right: -20px;
  width: 20px;
  height: 20px;
  background-color: #616161;
  border-radius: 0 0 20px 0;
}

.sort-container {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  width: 860px;
  gap: 10px;
  color: #a4a4a4;
  font-weight: 500;
}

.sort-container select {
  border: none;
  border-radius: 5px;
  background-color: #313131;
  color: #ffffff;
  height: 36px;
  width: 200px;
  padding: 0 10px;
  font-size: 1rem;
}

.sort-container select:hover {
  cursor: pointer;
}

.sort-container option {
  background-color: #616161;
}

.sort-container option:hover {
  background-color: #719f3f;
}

.pokemon-list-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  margin-bottom: 20px;
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
  user-select: none;
}

.pokemon-card:hover {
  transform: translateY(-2px);
  transition: background-color 0.3s ease-out, transform 0.3s ease-out;
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
  align-items: center;
  background-color: #719f3f;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-dragon {
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(#53a4cf 50%, #f16e57 50%);
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-fairy {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #fdb9e9;
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-fire {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #fd7d24;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-ghost {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #7b62a3;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-ground {
  display: flex;
  justify-content: center;
  align-items: center;
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
  align-items: center;
  background-color: #f366b9;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-steel {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #9eb7b8;
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-dark {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #707070;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-electric {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #eed535;
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-fighting {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #d56723;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-flying {
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(#3dc7ef 50%, #bdb9b8 50%);
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-grass {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #9bcc50;
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-ice {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #51c4e7;
  color: black;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-poison {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #b97fc8;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-rock {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #a38c21;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.pokemon-type-water {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #4592c4;
  color: white;
  border-radius: 5px;
  width: 85px;
}

.load-more-button {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #30a7d7;
  color: white;
  border: none;
  height: 36px;
  width: 164px;
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
  width: 360px;
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
  background-color: #c0c0c0;
  color: #515151;
  transition: background-color 0.3s ease-out, color 0.3s ease-out;
}

.details-tabs button:hover {
  background-color: #a4a4a4;
  color: #000000;
}

.details-tabs button.active {
  background-color: #616161;
  color: #ffffff;
}

.stats-content {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #616161;
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
  background-color: #616161;
  border-radius: 0 10px 10px 10px;
  height: 300px;
}

.evolutions-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  width: 100%;
}

.evolutions-pokemon {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  font-weight: 500;
  color: #ffffff;
}

.evolutions-pokemon-details {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.evolutions-pokemon-details:hover {
  cursor: pointer;
}

.evolutions-pokemon-frame {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #717171;
  border-radius: 50%;
  width: 140px;
  height: 140px;
  border: #ffffff 4px solid;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.4);
  overflow: hidden;
}

.evolutions-pokemon-frame img {
  height: 115px;
}

.evolutions-pokemon-name {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  font-weight: 500;
}

.evolutions-pokemon-name h3 {
  font-weight: 500;
}
</style>
