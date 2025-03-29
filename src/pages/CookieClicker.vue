<script setup>
import { computed, ref } from 'vue';

const cookies = ref(0);
const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1000, cps: 10, count: 0 },
    { name: 'Factory', price: 0, cps: 1000000, count: 0 },
]);

function buyBuilding(building) {
    cookies.value -= building.price;
    building.price += Math.ceil(building.price / 100 * 15);
    building.count++;
}
let cps = computed(() => {
    let cps = 0;
    buildings.value.forEach(building => {
        cps += building.cps * building.count;
    });
    return cps;
});
setInterval(() => {
    cookies.value += cps.value;

    document.title = '🍪' + +cookies.value.toFixed(1) + ' Cookies!';
}, 1000);

</script>
<template>
    <div class="columns">
        <div class="column is-4 has-background-white has-text-centered">
            <h1 class="is-size-1 has-text-black">{{ +cookies.toFixed(1) }} cookies</h1>
            <h3 class="is-size-3 has-text-black">{{ +cps.toFixed(1) }} cps</h3>
            <figure class="image is-square cookie-move" @click="cookies++">
                <img
                    src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
            </figure>
        </div>

        <div class="column is-6 has-background-white has-text-black">
            <div v-for="building in buildings">
                <div v-if="building.count > 0" class="py-2">
                    you have {{ building.count }} {{ building.name }}'s
                    <div style="display: flex; flex-wrap: wrap; gap: 4px; max-width: 100%;">
                        <div v-for="(_, i) in Array.from({ length: building.count })" :key="i">
                            👵
                        </div>
                    </div>
                </div>
            </div>
        </div>





        <div class="column is-2 has-background-white">
            <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)"
                class="button is-primary is-large is-fullwidth">
                {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
            </button>
        </div>
    </div>
</template>

<style scoped>
.cookie-move {
    transition: transform 0.1s ease;
}

.cookie-move:active {
    transform: scale(0.9) rotate(5deg);
}
</style>