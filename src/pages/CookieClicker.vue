<script setup>
import { computed, ref, watch } from 'vue';

const cookies = ref(0);
const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0},
    { name: 'Grandma', price: 100, cps: 1, count: 0},
    { name: 'Farm', price: 1000, cps: 10, count: 0},
    { name: 'Factory', price: 10_000, cps: 100, count: 0},
]);


const achievements = ref({
  firstClick: false,
  firstUpgrade: false,
  goldenCookie: false
});


function buyBuilding(building) {
    cookies.value -= building.price;
    building.price += Math.ceil(building.price / 100 * 15);
    building.count++;
    if (!achievements.value.firstUpgrade) {
        achievements.value.firstUpgrade = true;
        alert("CONGRATS!NEW ACHIVEMENT");
    }
}


let cps = computed(() => {
    let cps = 0;
    buildings.value.forEach(building => {
        cps += building.cps * building.count;
    });
    return cps;
});


function onCookieClick() {
    cookies.value++;
    if (!achievements.value.firstClick) {
        achievements.value.firstClick = true;
        alert("CONGRATS!NEW ACHIVEMENT");
    }
}


setInterval(() => {
    cookies.value += cps.value;


    if (!achievements.value.goldenCookie && Math.random() < 0.001) {
        achievements.value.goldenCookie = true;
        alert("Поздравляем! Золотая печенька появилась! Получите бонус!");
        cookies.value += 1000;  
    }

    document.title = '🍪' + cookies.value.toFixed(1) + ' Cookies!';
}, 1000);
</script>

<template>
    <div class="columns">

        <div class="column is-4 has-background-primary has-text-centered">
            <h1 class="is-size-1">{{ +cookies.toFixed(1) }} cookies</h1>
            <h3 class="is-size-3">{{ +cps.toFixed(1) }} cps</h3>

            <figure class="image is-square" @click="onCookieClick">
                <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
            </figure>
        </div>


        <div class="column is-2 has-background-warning">
            <button v-for="building in buildings" 
                    :disabled="cookies<building.price" 
                    @click="buyBuilding(building)" 
                    class="button is-primary is-large is-fullwidth">
                {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
            </button>
        </div>
    </div>


    <div v-if="achievements.firstClick || achievements.firstUpgrade || achievements.goldenCookie" class="notification is-info">
        <h4 class="title">Your achievements:</h4>
        <ul>
            <li v-if="achievements.firstClick">first click</li>
            <li v-if="achievements.firstUpgrade">first upgrade</li>
            <li v-if="achievements.goldenCookie"> ITS GOOOLDEEEEN</li>
        </ul>
    </div>
</template>
