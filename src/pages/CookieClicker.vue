<script setup>
import { computed, ref, onMounted } from 'vue';

const cookies = ref(0);
const goldenCookies = ref(0);

const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1000, cps: 10, count: 0 },
    { name: 'Factory', price: 10_000, cps: 100, count: 0 },
]);

const upgrades = ref([
    { name: 'Double Cookies', price: 5000, effect: () => cps.value *= 2, owned: false },
]);

const achievements = ref([
    { name: 'First Click', goal: 1, reached: false },
    { name: 'First Building', goal: 1, reached: false },
    { name: 'First 100 Cookies', goal: 100, reached: false },
]);

function buyBuilding(building) {
    if (cookies.value >= building.price) {
        cookies.value -= building.price;
        building.price += Math.ceil(building.price / 100 * 15);
    }
}

let cps = computed(() => {
    let cpsValue = 0;
    buildings.value.forEach(building => {
        cpsValue += building.cps * building.count;
    });
    return cpsValue;
});

let goldenCookieTimer = setInterval(() => {
    if (Math.random() < 0.01) {
        goldenCookies.value++;
        alert('Golden Cookie! +500 Cookies');
        cookies.value += 500;
    }
}, 1000);

function checkAchievements() {
    achievements.value.forEach(achievement => {
        if (cookies.value >= achievement.goal && !achievement.reached) {
            achievement.reached = true;
            alert(`Achievement Unlocked: ${achievement.name}`);
        }
    });
}

setInterval(() => {
    cookies.value += cps.value;
    checkAchievements();
    document.title = `🍪 ${cookies.value.toFixed(1)} Cookies!`;
}, 1000);

function buyUpgrade(upgrade) {
    if (cookies.value >= upgrade.price && !upgrade.owned) {
        cookies.value -= upgrade.price;
        upgrade.owned = true;
        upgrade.effect();
        alert(`${upgrade.name} Purchased!`);
    }
}

onMounted(() => {
    const savedState = JSON.parse(localStorage.getItem('cookie-clicker-save'));
    if (savedState) {
        cookies.value = savedState.cookies;
        buildings.value = savedState.buildings;
        upgrades.value = savedState.upgrades;
    }
});

setInterval(() => {
    localStorage.setItem('cookie-clicker-save', JSON.stringify({
        cookies: cookies.value,
        buildings: buildings.value,
        upgrades: upgrades.value,
    }));
}, 10000);

</script>

<template>
  <div class="columns">
    <div class="column is-4 has-background-primary has-text-centered">
      <h1 class="is-size-1">{{ +cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3">{{ +cps.toFixed(1) }} cps</h3>
      <figure class="image is-square" @click="cookies++">
        <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
      </figure>
    </div>

    <div class="column is-6 has-background-link">
      <h2 class="is-size-4">Upgrades</h2>
      <button v-for="upgrade in upgrades" :disabled="cookies < upgrade.price || upgrade.owned" @click="buyUpgrade(upgrade)" class="button is-primary is-large is-fullwidth">
        {{ upgrade.name }} 🍪{{ upgrade.price }}
      </button>
      <h2 class="is-size-4">Achievements</h2>
      <ul>
        <li v-for="achievement in achievements" :key="achievement.name">
          {{ achievement.name }} - {{ achievement.reached ? 'Unlocked' : `Goal: ${achievement.goal}` }}
        </li>
      </ul>
    </div>
    <div class="column is-2 has-background-warning">
      <button v-for="building in buildings" :disabled="cookies < building.price" @click="buyBuilding(building)" class="button is-primary is-large is-fullwidth">
        {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
      </button>
    </div>
  </div>
  <div v-if="goldenCookies > 0" class="golden-cookie" @click="goldenCookies--">
    <img src="https://static.wikia.nocookie.net/ccbincc/images/e/ec/Golden_cookie.png/revision/latest/smart/width/250/height/250?cb=20220520155214" alt="Golden Cookie">
  </div>
</template>

<style scoped>
  .golden-cookie {
    position: absolute;
    top: 20%;
    left: 50%;
    transform: translateX(-50%);
    cursor: pointer;
    z-index: 1000;
  }
</style>
