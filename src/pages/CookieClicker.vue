<script setup>
import { computed, ref, onMounted } from 'vue';
import AchievementBox from './AchievementBox.vue'


const cookies = ref(0);
const buildings = ref([
  { name: 'Cursor',     price: 15,        cps: 5,           count: 0, emoji: '🖱️' },
  { name: 'Grandma',    price: 100,       cps: 20,          count: 0, emoji: '👵' },
  { name: 'Farm',       price: 1100,      cps: 2000,        count: 0, emoji: '🌾' },
  { name: 'Factory',    price: 12000,     cps: 20000,       count: 0, emoji: '🏭' },
  { name: 'Mine',       price: 130000,    cps: 100000,      count: 0, emoji: '⛏️' },
  { name: 'Shipment',   price: 1400000,   cps: 500000,      count: 0, emoji: '🚀' },
  { name: 'Alchemy Lab',price: 20000000,  cps: 2000000,     count: 0, emoji: '⚗️' },
  { name: 'Portal',     price: 330000000, cps: 10000000,    count: 0, emoji: '🌀' },
  { name: 'Time Machine', price: 5100000000, cps: 40000000, count: 0, emoji: '⏰' },
  { name: 'Antimatter Condenser', price: 75000000000, cps: 160000000, count: 0, emoji: '💠' },
  { name: 'Prism',      price: 1000000000000, cps: 640000000, count: 0, emoji: '🌈' },
  { name: 'Chancemaker', price: 14000000000000, cps: 2560000000, count: 0, emoji: '🎲' },
  { name: 'Fractal Engine', price: 170000000000000, cps: 10240000000, count: 0, emoji: '🧩' },
  { name: 'Javascript Console', price: 2100000000000000, cps: 40960000000, count: 0, emoji: '💻' },
  { name: 'Idleverse',  price: 26000000000000000, cps: 163840000000, count: 0, emoji: '🌌' },
  { name: 'Cortex Baker', price: 310000000000000000, cps: 655360000000, count: 0, emoji: '🧠' }
]);



let userclicked = ref(0);
const clickAchievementsData = [
  [10, "Welcome to Cookie Clicker!", "10 clicks"],
  [50, "Click addict!", "50 clicks already?"],
  [100, "Click master!", "100 clicks. You serious?"],
  [500, "Click god!", "500 clicks. Seek help."],
  [10000, "Finger of steel", "10,000 clicks. What's wrong with you?"],
  [100, "Clicking Apprentice 🖱️", "You've clicked 100 times."],
];

const buildingAchievementsData = [
  ["Cursor", 10, "Starter Clicker 🖱️", "You own 10 Cursors."],
  ["Cursor", 20, "Cursor Collector 👆", "You own 20 Cursors."],
  ["Cursor", 50, "Click Swarm 💥", "You own 50 Cursors."],

  ["Grandma", 10, "Grandma's Bakery 👵", "You have 10 Grandmas."],
  ["Grandma", 25, "Cookie Coven 🧙‍♀️", "You have 25 Grandmas."],
  ["Grandma", 50, "Elder Council 🕯️", "You have 50 Grandmas."],

  ["Farm", 5, "Farming Tycoon 🌾", "You own 5 Farms."],
  ["Farm", 20, "Agricultural Empire 🚜", "You own 20 Farms."],
  ["Farm", 50, "Mega Farm 🏡", "You own 50 Farms."],

  ["Factory", 10, "Mass Production 🏭", "You have 10 Factories."],
  ["Factory", 50, "Factorio! ⚙️", "You have 50 Factories."],
  ["Factory", 100, "Industrial Overlord 🛠️", "You have 100 Factories."],

  ["Mine", 10, "Rock Digger ⛏️", "You have 10 Mines."],
  ["Mine", 25, "Tunnel Vision 🕳️", "You have 25 Mines."],
  ["Mine", 50, "Motherlode Miner 💎", "You have 50 Mines."],

  ["Shipment", 5, "Export Hustler 🚚", "You own 5 Shipments."],
  ["Shipment", 20, "Space Dealer 🚀", "You own 20 Shipments."],
  ["Shipment", 50, "Interstellar Trade 🪐", "You own 50 Shipments."],

  ["Alchemy Lab", 5, "Apprentice Alchemist ⚗️", "You own 5 Alchemy Labs."],
  ["Alchemy Lab", 15, "Golden Touch ✨", "You own 15 Alchemy Labs."],
  ["Alchemy Lab", 30, "Philosopher’s Stone 🧪", "You own 30 Alchemy Labs."],

  ["Portal", 1, "First Rift 🌀", "You opened a Portal."],
  ["Portal", 10, "Beyond the Veil 🌌", "You own 10 Portals."],
  ["Portal", 25, "Dimensional Master 🪞", "You own 25 Portals."],

  ["Time Machine", 1, "Time Beginner ⏰", "You built your first Time Machine."],
  ["Time Machine", 10, "Looper 🔁", "You own 10 Time Machines."],
  ["Time Machine", 30, "Temporal Lord 🕰️", "You own 30 Time Machines."],

  ["Antimatter Condenser", 5, "Quantum Tinkerer 💠", "You own 5 Antimatter Condensers."],
  ["Antimatter Condenser", 15, "Core Melter ⚛️", "You own 15 Antimatter Condensers."],

  ["Prism", 5, "Light Bender 🌈", "You own 5 Prisms."],
  ["Prism", 15, "Spectrum Shifter 🎇", "You own 15 Prisms."],

  ["Chancemaker", 5, "Probability Twister 🎲", "You own 5 Chancemakers."],
  ["Chancemaker", 15, "Luck Architect 🍀", "You own 15 Chancemakers."],

  ["Fractal Engine", 5, "Pattern Weaver 🧩", "You own 5 Fractal Engines."],
  ["Fractal Engine", 15, "Recursive Genius 🔁", "You own 15 Fractal Engines."],

  ["Javascript Console", 5, "Console Logger 💻", "You own 5 JavaScript Consoles."],
  ["Javascript Console", 15, "Infinite Loop 🖥️", "You own 15 JavaScript Consoles."],

  ["Idleverse", 1, "Meta Awakening 🌌", "You created the Idleverse."],
  ["Idleverse", 10, "Simulation Overlord 🧬", "You own 10 Idleverses."],

  ["Cortex Baker", 1, "Mind Over Batter 🧠", "You built your first Cortex Baker."],
  ["Cortex Baker", 5, "Neural Doughmaster 🧠🍪", "You own 5 Cortex Bakers."],
  ["Cortex Baker", 10, "Brainstorm Bakesplosion 🧠⚡", "You own 10 Cortex Bakers."]
];


const achievements = computed(() => [
  ...clickAchievementsData.map(([count, name, desc]) => ({
    name,
    desc,
    get condition() {
      return userclicked.value == count;
    },
  })),
  ...buildingAchievementsData.map(([buildingName, targetCount, name, desc]) => ({
    name,
    desc,
    get condition() {
      return buildings.value.some(
        (item) => item.name == buildingName && item.count == targetCount
      );
    },
  })),
]);


// Stores the max spawn positions for golden cookies inside the container
const goldenCookieSpawnLimits = {
  maxX: 0,
  maxY: 0,
};


//let unlocked_achievement = ref(-1); // if achievement index then show the echievement for 3 seconds

function whenCookieClicked() {
  cookies.value++;
  userclicked.value++;
}

function buyBuilding(building) {
  cookies.value -= building.price;
  building.count++;

  // Make price increase more gently, e.g. 10%
  building.price = Math.ceil(building.price * 1.1);
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

function updateSpawnLimits() {
  const container = document.querySelector('.golden-cookie-container');
  if (container) {
    const rect = container.getBoundingClientRect();
    goldenCookieSpawnLimits.maxX = rect.width;
    goldenCookieSpawnLimits.maxY = rect.height;
  }
}


function showGoldenCookie() {
  updateSpawnLimits();
  const cookie = document.querySelector('.golden-cookie');
  if (!cookie) return;

  const x = Math.random() * (goldenCookieSpawnLimits.maxX - 100);
  const y = Math.random() * (goldenCookieSpawnLimits.maxY - 100);

  cookie.style.position = 'fixed';
  cookie.style.left = `${x}px`;
  cookie.style.top = `${y}px`;
  cookie.style.display = "block";

}


onMounted(() => {
  if (typeof window !== 'undefined') {
    const div = document.querySelector('.golden-cookie');
    if (div) {
      showGoldenCookie();
      window.addEventListener('resize', () => {
        showGoldenCookie();
      });

    }
  }
});


function onGoldenCookieClick(){
  cookies.value += cookies.value * 2;
  const cookie = document.querySelector('.golden-cookie');
  cookie.style.display = "none";
  setTimeout(() => {
    showGoldenCookie();
  }, 1000);
}


</script>
<template>
  <div class="golden-cookie is-size-1" @click="onGoldenCookieClick">🍪</div>

  <div class="columns golden-cookie-container">
    <div class="column is-4 has-background-white has-text-centered">
      <h1 class="is-size-1 has-text-black">{{ +cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3 has-text-black">{{ +cps.toFixed(1) }} cps</h3>
      <figure class="image is-square cookie-move" @click="whenCookieClicked">
        <img
          src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
      </figure>
    </div>

    <div class="column is-6 has-background-white has-text-black is-flex is-flex-direction-column">
      <div style="flex:1;">
        <div v-for="building in buildings">
          <div v-if="building.count > 0" class="py-2">
            you have {{ building.count }} {{ building.name }}'s
            <div style="display: flex; flex-wrap: wrap; gap: 4px; max-width: 100%;">
              <div v-for="(_, i) in Array.from({ length: building.count })" :key="i">
                {{building.emoji }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="achievements-box d-block">
        <div class="achievement is-gapless">
          <AchievementBox :achievements="achievements"></AchievementBox>
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

.golden-cookie {
  position: fixed; /* stays in place even on scroll */
  top: 100px;       /* or use JS to randomize */
  left: 300px;
  z-index: 9999;    /* ensure it's above everything */
  pointer-events: pointer; /* clickable */
  display: 'block';
}


</style>