<script setup>
defineProps({
    primaryColor: {
        type: String,
        default: '#4f4f4f'
    },
    secondaryColor: {
        type: String,
        default: '#ffffff'
    },
    size: {
        type: String,
        default: '51px'
    }
});
</script>

<template>
    <div 
        class="pokeball" 
        :style="{ 
            backgroundColor: primaryColor, 
            width: size, 
            height: size 
        }"
    >
        <div 
            class="pokeball-line" 
            :style="{ 
                backgroundColor: secondaryColor,
                width: size,
                height: `calc(${size} * 0.137)`
            }"
        >
            <div 
                class="pokeball-circle" 
                :style="{ 
                    backgroundColor: secondaryColor,
                    width: `calc(${size} * 0.41)`,
                    height: `calc(${size} * 0.41)`
                }"
            >
                <div 
                    class="pokeball-circle-inner" 
                    :style="{ 
                        backgroundColor: primaryColor,
                        width: `calc(${size} * 0.215)`,
                        height: `calc(${size} * 0.215)`
                    }"
                ></div>
            </div>
        </div>
    </div>
</template>

<style scoped>
@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(180deg);
    }
}

.pokeball {
    display: flex;
    justify-content: center;
    align-items: center;
    animation: spin 1.4s ease-in-out infinite;
    width: 51px;
    height: 51px;
    background-color: #4f4f4f;
    border-radius: 50%;
}

.pokeball-line {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 51px;
    height: 7px;
    background-color: #ffffff;
}

.pokeball-circle {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 21px;
    height: 21px;
    background-color: #ffffff;
    border-radius: 50%;
}

.pokeball-circle-inner {
    width: 11px;
    height: 11px;
    background-color: #4f4f4f;
    border-radius: 50%;
}
</style>