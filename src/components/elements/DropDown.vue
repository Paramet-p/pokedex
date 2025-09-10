<script setup>
import { ref, onUnmounted } from 'vue';

defineProps({
    width: {
        type: String,
        default: '240px'
    },
    height: {
        type: String,
        default: '36px'
    },
    options: {
        type: Array,
        default: []
    },
    selected: {
        type: String,
        default: ''
    }
})

const emit = defineEmits(['update:selected']);

const showOptions = ref(false);
const dropdownRef = ref(null);

const selectOption = (option) => {
    emit('update:selected', option.value);
    showOptions.value = false;
}

const closeDropdown = (event) => {
    if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
        showOptions.value = false;
    }
}

document.addEventListener('click', closeDropdown);
onUnmounted(() => {
    document.removeEventListener('click', closeDropdown);
});
</script>

<template>
    <div class="dropdown" ref="dropdownRef" @click="showOptions = !showOptions">
        <div>
            <span>{{ options.find(option => option.value === selected)?.name || options[0]?.name }}</span>
            <span class="dropdown-arrow" :class="{ 'active': showOptions }">V</span>
        </div>
        <ul v-if="showOptions">
            <li v-for="item, index in options" :key="index" @click.stop="selectOption(item)">{{ item.name }}</li>
        </ul>
    </div>
</template>

<style scoped>
.dropdown {
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: relative;
    border: none;
    border-radius: 5px;
    background-color: #313131;
    color: #ffffff;
    height: v-bind(height);
    width: v-bind(width);
    padding: 0 10px;
    font-size: 1rem;
    cursor: pointer;
    user-select: none;
}

.dropdown:hover {
    background-color: #282828;
}

.dropdown-arrow {
    transition: transform 0.2s ease-out;
}

.dropdown-arrow.active {
    transform: rotate(180deg);
}

.dropdown div {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
}

.dropdown ul {
    position: absolute;
    top: 100%;
    left: 0;
    width: v-bind(width);
    max-height: 274px;
    background-color: #313131;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
    list-style: none;
    padding: 0;
    overflow-y: auto;
    z-index: 1000;
}

.dropdown li {
    padding: 10px;
    cursor: pointer;
    background-color: #616161;
}

.dropdown li:hover {
    background-color: #313131;
}

::-webkit-scrollbar {
    width: 10px;
    height: 10px;
}

::-webkit-scrollbar-track {
    background: #616161;
}

::-webkit-scrollbar-thumb {
    background: #313131;
    border-radius: 5px;
}

::-webkit-scrollbar-thumb:hover {
    background: #282828;
}
</style>