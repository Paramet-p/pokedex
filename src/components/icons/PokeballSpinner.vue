<template>
    <div class="pokeball">
        <div class="pokeball-line">
            <div class="pokeball-circle">
                <div class="pokeball-circle-inner"></div>
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