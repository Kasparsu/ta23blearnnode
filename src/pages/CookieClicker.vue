<script setup>
import { computed, ref, watch } from 'vue';

const cookies = ref(0);
const clickPower = ref(1);
const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0},
    { name: 'Grandma', price: 100, cps: 1, count: 0},
    { name: 'Farm', price: 1000, cps: 10, count: 0},
    { name: 'Factory', price: 10000, cps: 100, count: 0},
]);

const upgrades = ref([
    { name: 'Better Oven', price: 100, bonus: 1, bought: false },
    { name: 'Tastier Dough', price: 200, bonus: 2, bought: false },
    { name: 'Improved Rolling Pins', price: 500, bonus: 5, bought: false },
]);

const achievements = ref({
  firstClick: false,
  firstUpgrade: false,
  firstUpgradeBought: false,
  goldenCookie: false
});

const isClicked = ref(false);

function buyBuilding(building) {
    if (cookies.value >= building.price) {
        cookies.value -= building.price;
        building.price += Math.ceil(building.price * 0.15);
        building.count++;
    }
}

function buyUpgrade(upgrade) {
    if (cookies.value >= upgrade.price && !upgrade.bought) {
        cookies.value -= upgrade.price;
        clickPower.value += upgrade.bonus;
        upgrade.bought = true;
        if (!achievements.value.firstUpgradeBought) {
            achievements.value.firstUpgradeBought = true;
            alert("CONGRATS! FIRST UPGRADE ACHIEVEMENT");
        }
    }
}

let cps = computed(() => {
    return buildings.value.reduce((total, building) => total + building.cps * building.count, 0);
});

function onCookieClick() {
    cookies.value += clickPower.value;
    if (!achievements.value.firstClick) {
        achievements.value.firstClick = true;
        alert("CONGRATS! NEW ACHIEVEMENT");
    }
    isClicked.value = true;
    setTimeout(() => isClicked.value = false, 300);
}

setInterval(() => {
    cookies.value += cps.value;
    if (!achievements.value.goldenCookie && Math.random() < 0.1) {
        achievements.value.goldenCookie = true;
        alert("GOLDEN COOKIE! WHAT, HOW?!");
        cookies.value += 1000;
    }
}, 1000);
</script>

<template>
    <div class="columns">
        <div class="column is-5 has-background-primary has-text-centered">
            <h1 class="is-size-1">{{ cookies.toFixed(1) }} cookies</h1>
            <h3 class="is-size-1">{{ cps.toFixed(1) }} cps</h3>
            <figure class="image is-4by3" @click="onCookieClick">
                <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" 
                     :class="{'clicked': isClicked}" />
            </figure>
        </div>

        <div class="column is-4 has-background-warning">
            <h2 class="title">Upgrades & Buildings</h2>
            <div>
                <h3 class="subtitle">Upgrades</h3>
                <button v-for="upgrade in upgrades" 
                        :disabled="cookies < upgrade.price || upgrade.bought" 
                        @click="buyUpgrade(upgrade)" 
                        class="button is-success is-large is-fullwidth">
                    {{ upgrade.name }} 🍪{{ upgrade.price }} (Click +{{ upgrade.bonus }})
                </button>
            </div>
            <div>
                <h3 class="subtitle">Buildings</h3>
                <button v-for="building in buildings" 
                        :disabled="cookies < building.price" 
                        @click="buyBuilding(building)" 
                        class="button is-primary is-large is-fullwidth">
                    {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
                </button>
            </div>
        </div>

        <div class="column is-3 has-background-info">
            <h2 class="title">Achievements</h2>
            <div v-if="achievements.firstClick || achievements.firstUpgradeBought || achievements.goldenCookie" class="notification is-info">
                <ul>
                    <li v-if="achievements.firstClick">First Click</li>
                    <li v-if="achievements.firstUpgradeBought">First Upgrade Bought</li>
                    <li v-if="achievements.goldenCookie">GOLDEN COOKIE!</li>
                </ul>
            </div>
        </div>
    </div>
</template>

<style scoped>
.image img {
    max-width: 500px;
    max-height: 500px;
    width: auto;
    transition: transform 0.3s ease;
}

.image img.clicked {
    transform: scale(0.8);
}
</style>