<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

const cookies = ref(0);
const buildings = ref([
  { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
  { name: 'Grandma', price: 100, cps: 1, count: 0 },
  { name: 'Farm', price: 1000, cps: 10, count: 0 },
  { name: 'Factory', price: 10000, cps: 30, count: 0 },
]);

const upgrades = ref([
  { id: 1, name: 'Better Cursors', description: 'Cursors give 2x more cookies', cost: 100, bought: false, apply: () => { multiplyBuildingCPS('Cursor', 2) }},
  { id: 2, name: 'Efficient Farms', description: 'Farms give 2x more cookies', cost: 5000, bought: false, apply: () => { multiplyBuildingCPS('Farm', 2) }},
  { id: 3, name: 'Grandma Helpers', description: 'Grandmas give 2x more cookies', cost: 2000, bought: false, apply: () => { multiplyBuildingCPS('Grandma', 2) }},
]);

const achievements = ref([
  { id: 1, name: 'First Click', description: 'Click the cookie once', achieved: false, condition: () => cookies.value >= 1 },
  { id: 2, name: '100 Cookies', description: 'Bake 100 cookies', achieved: false, condition: () => cookies.value >= 100 },
  { id: 3, name: '10 Buildings', description: 'Buy 10 buildings', achieved: false, condition: () => totalBuildings() >= 10 },
  { id: 4, name: 'Upgrade Buyer', description: 'Buy 2 upgrades', achieved: false, condition: () => upgrades.value.filter(u => u.bought).length >= 2 },
]);

const goldenCookieVisible = ref(false);
const goldenCookieMultiplier = ref(1);
let goldenCookieTimeout = null;
let goldenCookieEffectTimeout = null;

function multiplyBuildingCPS(buildingName, factor) {
  const b = buildings.value.find(b => b.name === buildingName);
  if (b) b.cps *= factor;
}

function totalBuildings() {
  return buildings.value.reduce((acc, b) => acc + b.count, 0);
}

function buyBuilding(building) {
  if (cookies.value >= building.price) {
    cookies.value -= building.price;
    building.price = Math.ceil(building.price * 1.15);
    building.count++;
    checkAchievements();
  }
}

function buyUpgrade(upgrade) {
  if (cookies.value >= upgrade.cost && !upgrade.bought) {
    cookies.value -= upgrade.cost;
    upgrade.bought = true;
    upgrade.apply();
    checkAchievements();
  }
}

function clickGoldenCookie() {
  if (!goldenCookieVisible.value) return;
  goldenCookieVisible.value = false;
  goldenCookieMultiplier.value = 7;
  if (goldenCookieEffectTimeout) clearTimeout(goldenCookieEffectTimeout);
  goldenCookieEffectTimeout = setTimeout(() => {
    goldenCookieMultiplier.value = 1;
  }, 10000);
}

function scheduleGoldenCookie() {
  if (goldenCookieTimeout) clearTimeout(goldenCookieTimeout);
  const nextTime = 30000 + Math.random() * 30000;
  goldenCookieTimeout = setTimeout(() => {
    goldenCookieVisible.value = true;
    scheduleGoldenCookie();
  }, nextTime);
}

function checkAchievements() {
  achievements.value.forEach(a => {
    if (!a.achieved && a.condition()) {
      a.achieved = true;
    }
  });
}

const saveGameState = () => {
  const gameState = {
    cookies: cookies.value,
    buildings: buildings.value.map(b => ({
      name: b.name,
      price: b.price,
      cps: b.cps,
      count: b.count
    })),
    upgrades: upgrades.value.map(u => ({
      id: u.id,
      bought: u.bought,
    })),
    achievements: achievements.value.map(a => ({
      id: a.id,
      achieved: a.achieved,
    })),
  };
  try {
    localStorage.setItem('cookieGameSave', JSON.stringify(gameState));
  } catch (error) {
    console.error('Save error:', error);
  }
};

const loadGameState = () => {
  try {
    const savedData = localStorage.getItem('cookieGameSave');
    if (savedData) {
      const parsed = JSON.parse(savedData);
      cookies.value = parsed.cookies || 0;
      buildings.value = buildings.value.map(b => {
        const savedB = parsed.buildings?.find(sb => sb.name === b.name);
        return savedB ? {...b, ...savedB} : b;
      });
      if (parsed.upgrades) {
        upgrades.value.forEach(u => {
          const savedU = parsed.upgrades.find(su => su.id === u.id);
          if (savedU && savedU.bought) {
            u.bought = true;
            u.apply();
          }
        });
      }
      if (parsed.achievements) {
        achievements.value.forEach(a => {
          const savedA = parsed.achievements.find(sa => sa.id === a.id);
          if (savedA) a.achieved = savedA.achieved;
        });
      }
    }
  } catch (error) {
    console.error('Load error:', error);
  }
};

const cps = computed(() =>
  buildings.value.reduce((acc, b) => acc + b.cps * b.count, 0) * goldenCookieMultiplier.value
);

let saveInterval;
let cookieInterval;

onMounted(() => {
  loadGameState();
  scheduleGoldenCookie();

  saveInterval = setInterval(saveGameState, 1000);

  cookieInterval = setInterval(() => {
    cookies.value += cps.value / 10;
    document.title = `🍪${cookies.value.toFixed(1)} Cookies!`;
    checkAchievements();
  }, 100);

});

onBeforeUnmount(() => {
  clearInterval(saveInterval);
  clearInterval(cookieInterval);
  clearTimeout(goldenCookieTimeout);
  clearTimeout(goldenCookieEffectTimeout);
});
</script>

<template>
  <div class="columns" style="position: relative;">
    <div class="column is-4 has-background-grey has-text-centered">
      <h1 class="is-size-1">{{ cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3">{{ cps.toFixed(1) }} per second</h3>
      <figure class="image is-square" @click="cookies++" style="cursor:pointer;">
        <img src="https://www.freeiconspng.com/uploads/download-biscuit-cookie-monster-clipart-24.png" />
      </figure>

      <figure
        v-if="goldenCookieVisible"
        class="image is-64x64"
        @click="clickGoldenCookie"
        style="position: absolute; top: 10px; right: 10px; cursor: pointer;"
        title="Golden cookie! Click for x7 boost for 10 seconds"
      >
        <img src="https://upload.wikimedia.org/wikipedia/commons/9/99/Gold_Coin_icon.png" />
      </figure>

      <div class="mt-5 has-text-left" style="color: white;">
        <h4 class="is-size-4">Achievements</h4>
        <ul>
          <li
            v-for="a in achievements"
            :key="a.id"
            :style="{ textDecoration: a.achieved ? 'line-through' : 'none', color: a.achieved ? 'gold' : 'white' }"
          >
            {{ a.name }} - <small>{{ a.description }}</small>
          </li>
        </ul>
      </div>
    </div>

    <div class="column is-6 has-background-grey-dark">
      <!-- Grandma Images -->
      <div class="columns is-flex is-flex-wrap-wrap">
        <figure
          v-for="n in Math.floor(buildings.find(b => b.name === 'Grandma').count / 10) + 1"
          v-if="buildings.find(b => b.name === 'Grandma').count >= 1"
          class="image is-96x96 m-2"
          :key="'grandma-' + n"
        >
          <img src="https://png.pngtree.com/png-clipart/20230914/original/pngtree-cute-grandma-clipart-the-cartoon-old-lady-character-has-a-bouquet-png-image_11242831.png" />
        </figure>
      </div>

      <!-- Farm Images -->
      <div class="columns is-flex is-flex-wrap-wrap">
        <figure
          v-for="n in Math.floor(buildings.find(b => b.name === 'Farm').count / 10) + 1"
          v-if="buildings.find(b => b.name === 'Farm').count >= 1"
          class="image is-96x96 m-2"
          :key="'farm-' + n"
        >
          <img src="https://pics.clipartpng.com/Corn_PNG_Clipart-466.png" />
        </figure>
      </div>

      <!-- Factory Images -->
      <div class="columns is-flex is-flex-wrap-wrap">
        <figure
          v-for="n in Math.floor(buildings.find(b => b.name === 'Factory').count / 10) + 1"
          v-if="buildings.find(b => b.name === 'Factory').count >= 1"
          class="image is-96x96 m-2"
          :key="'factory-' + n"
        >
          <img src="https://www.freeiconspng.com/uploads/gear-icon-11.png" />
        </figure>
      </div>
    </div>

    <div class="column is-2 has-background-grey-darker">
      <button
        v-for="building in buildings"
        :key="building.name"
        :disabled="cookies < building.price"
        @click="buyBuilding(building)"
        class="button is-primary is-large is-fullwidth mb-3"
      >
        {{ building.name }} 🍪 {{ building.price }} #{{ building.count }}
      </button>

      <hr />

      <button
        v-for="upgrade in upgrades"
        :key="upgrade.id"
        :disabled="cookies < upgrade.cost || upgrade.bought"
        @click="buyUpgrade(upgrade)"
        class="button is-warning is-fullwidth mb-2"
        :class="{ 'is-light': upgrade.bought }"
        :title="upgrade.description"
      >
        {{ upgrade.name }} - 🍪 {{ upgrade.cost }}
        <span v-if="upgrade.bought">✓</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
button[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
