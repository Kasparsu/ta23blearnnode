<script setup>
import { computed, ref, onUnmounted } from 'vue';

const cookies = ref(0);
const goldenCookies = ref([]);
const achievements = ref([]);
const achievementNotifications = ref([]);
const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0},
    { name: 'Grandma', price: 100, cps: 1, count: 0},
    { name: 'Farm', price: 1000, cps: 10, count: 0},
    { name: 'Factory', price: 10_000, cps: 100, count: 0},
]);

const upgrades = ref([
    { name: 'Better Cursors', price: 500, effect: () => buildings.value[0].cps *= 2, bought: false },
    { name: 'Cheaper Buildings', price: 1000, effect: () => buildings.value.forEach(b => b.price *= 0.9), bought: false }
]);

const achievementList = [
    { name: '100 Cookies!', condition: () => cookies.value >= 100 },
    { name: '1,000 Cookies!', condition: () => cookies.value >= 1000 },
    { name: '10,000 Cookies!', condition: () => cookies.value >= 10000 },
    { name: 'First Building!', condition: () => buildings.value.some(b => b.count > 0) },
    { name: 'Golden Collector!', condition: () => goldenCookies.value.length >= 5 }
];

function buyBuilding(building) {
    if (cookies.value < building.price) return;
    cookies.value -= building.price;
    building.price += Math.ceil(building.price * 0.15);
    building.count++;
    checkAchievements();
}

function buyUpgrade(upgrade) {
    if (cookies.value < upgrade.price || upgrade.bought) return;
    cookies.value -= upgrade.price;
    upgrade.effect();
    upgrade.bought = true;
    checkAchievements();
}

let cps = computed(() => {
    return buildings.value.reduce((total, b) => total + b.cps * b.count, 0);
});

let interval = setInterval(() => {
    cookies.value += cps.value;
    document.title = `🍪 ${cookies.value.toFixed(1)} Cookies!`;
    checkAchievements();
}, 1000);

onUnmounted(() => clearInterval(interval));

function spawnGoldenCookie() {
    goldenCookies.value.push({ id: Date.now(), x: Math.random() * 80 + 10, y: Math.random() * 80 + 10 });
    setTimeout(() => goldenCookies.value.shift(), 5000);
    checkAchievements();
}

setInterval(() => {
    if (Math.random() < 0.05) spawnGoldenCookie();
}, 10000);

function collectGoldenCookie(id) {
    cookies.value += Math.ceil(cookies.value * 0.1) + 50;
    goldenCookies.value = goldenCookies.value.filter(gc => gc.id !== id);
    checkAchievements();
}

function checkAchievements() {
    achievementList.forEach(ach => {
        if (ach.condition() && !achievements.value.includes(ach.name)) {
            achievements.value.push(ach.name);
            achievementNotifications.value.push(ach.name);
            setTimeout(() => {
                achievementNotifications.value.shift();
            }, 3000);
        }
    });
}
</script>

<template>
    <div class="columns">
        <div class="column is-4 has-background-primary has-text-centered">
            <h1 class="is-size-1">{{ cookies.toFixed(1) }} cookies</h1>
            <h3 class="is-size-3">{{ cps.toFixed(1) }} cps</h3>
            <figure class="image is-square" @click="cookies++">
                <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
            </figure>
            <div v-for="gc in goldenCookies" :key="gc.id" class="golden-cookie" :style="{ top: gc.y + '%', left: gc.x + '%' }" @click="collectGoldenCookie(gc.id)">✨</div>
        </div>
        <div class="column is-4 has-background-link">
            <h2 class="is-size-3">Upgrades</h2>
            <button v-for="upgrade in upgrades" :disabled="cookies < upgrade.price || upgrade.bought" @click="buyUpgrade(upgrade)" class="button is-warning is-fullwidth">
                {{ upgrade.name }} - 🍪{{ upgrade.price }}
            </button>
            <h2 class="is-size-3">Achievements</h2>
            <ul>
                <li v-for="ach in achievements" :key="ach">🏆 {{ ach }}</li>
            </ul>
            <div class="achievement-popup" v-for="notif in achievementNotifications" :key="notif">🏆 Achievement Unlocked: {{ notif }}</div>
        </div>
        <div class="column is-4 has-background-warning">
            <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)" class="button is-primary is-large is-fullwidth">
                {{ building.name }} ({{ building.count }}) - 🍪{{ building.price }} | +{{ building.cps }} cps
            </button>
        </div>
    </div>
</template>

<style>
.golden-cookie {
    position: absolute;
    background: gold;
    border-radius: 50%;
    padding: 10px;
    cursor: pointer;
    font-size: 20px;
    animation: blink 0.5s infinite alternate;
}
@keyframes blink {
    0% { opacity: 1; }
    100% { opacity: 0.5; }
}
.achievement-popup {
    position: fixed;
    top: 10px;
    left: 50%;
    transform: translateX(-50%);
    background: #ffdd57;
    padding: 10px 20px;
    border-radius: 10px;
    font-weight: bold;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    animation: fadeOut 3s forwards;
}
@keyframes fadeOut {
    0% { opacity: 1; }
    100% { opacity: 0; }
}
</style>
