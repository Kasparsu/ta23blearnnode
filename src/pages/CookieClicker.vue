<script setup>
import { computed, ref } from 'vue';

const cookies = ref(0);
const buildings = ref([
  { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
  { name: 'Grandma', price: 100, cps: 1, count: 0 },
  { name: 'Farm', price: 1000, cps: 10, count: 0 },
  { name: 'Factory', price: 10000, cps: 100, count: 0 },
]);

const achievements = ref([]);
const bonusTimer = ref(30);
const isRunningAway = ref(false);
const runAwayCooldown = ref(45);
const runAwayTimer = ref(0);
const cookieStyle = ref({ top: '0px', left: '0px', position: 'relative' });
const scale = ref(1);

function buyBuilding(building) {
  if (cookies.value >= building.price) {
    cookies.value -= building.price;
    building.count++;
    building.price = Math.ceil(building.price * 1.15);
  }
}

const cps = computed(() =>
  buildings.value.reduce((sum, b) => sum + b.count * b.cps, 0)
);

setInterval(() => {
  cookies.value += cps.value;
  document.title = '🍪 ' + cookies.value.toFixed(1) + ' Cookies!';
  checkAchievements();
}, 1000);

function checkAchievements() {
  if (cookies.value >= 100 && !achievements.value.includes('100 cookies')) {
    achievements.value.push('100 cookies');
    showToast('🏆 Achievement: 100 cookies!');
  }
  if (buildings.value[1].count >= 1 && !achievements.value.includes('First Grandma')) {
    achievements.value.push('First Grandma');
    showToast('👵 Achievement: First Grandma!');
  }
}

function showToast(message) {
  const toast = document.createElement('div');
  toast.innerText = message;
  toast.style.position = 'fixed';
  toast.style.top = '20px';
  toast.style.left = '50%';
  toast.style.transform = 'translateX(-50%)';
  toast.style.background = 'white';
  toast.style.padding = '10px 20px';
  toast.style.borderRadius = '8px';
  toast.style.boxShadow = '0 0 10px rgba(0,0,0,0.2)';
  toast.style.zIndex = '9999';
  document.body.appendChild(toast);
  setTimeout(() => toast.remove(), 3000);
}

// Bonus every 30 seconds
setInterval(() => {
  bonusTimer.value--;
  if (bonusTimer.value <= 0) {
    cookies.value += 100;
    showToast('🎁 Bonus: +100 cookies!');
    bonusTimer.value = 25;
  }
}, 1000);

// "Runaway Cookie" event
setInterval(() => {
  if (!isRunningAway.value) {
    runAwayCooldown.value--;
    if (runAwayCooldown.value <= 0) {
      startRunningCookie();
    }
  } else {
    runAwayTimer.value--;
    if (runAwayTimer.value <= 0) {
      stopRunningCookie();
    } else {
      moveCookieRandomly();
    }
  }
}, 1000);

function startRunningCookie() {
  isRunningAway.value = true;
  runAwayTimer.value = 10;
  scale.value = 0.5; // shrink cookie
  showToast('😈 Cookie is running away!');
}

function stopRunningCookie() {
  isRunningAway.value = false;
  runAwayCooldown.value = Math.floor(Math.random() * 60 + 30);
  cookieStyle.value = { top: '0px', left: '0px', position: 'relative' };
  scale.value = 1; // restore size
  showToast('🍪 Cookie stopped running!');
}

function moveCookieRandomly() {
  const x = Math.floor(Math.random() * 200 - 100);
  const y = Math.floor(Math.random() * 200 - 100);
  cookieStyle.value = {
    position: 'relative',
    left: `${x}px`,
    top: `${y}px`,
    transition: 'all 0.3s ease',
  };
}

function clickCookie() {
  cookies.value += 1;
  scale.value = isRunningAway.value ? 0.55 : 1.1; // keep smaller if running
  const audio = new Audio('https://www.myinstants.com/media/sounds/minecraft_click.mp3');
  audio.play();
  setTimeout(() => (scale.value = isRunningAway.value ? 0.5 : 1), 100);
}
</script>

<template>
  <div class="columns">
    <div class="column is-4 has-background-primary has-text-centered">
      <h1 class="is-size-1">{{ cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3">{{ cps.toFixed(1) }} cps</h3>

      <figure
        class="image is-square"
        :style="[cookieStyle, { transform: `scale(${scale})`, transition: 'transform 0.1s' }]"
        @click="clickCookie"
      >
        <img
          src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511"
          alt="cookie"
        />
      </figure>

      <p v-if="isRunningAway" class="mt-2" style="color: #b71c1c;">😈 Cookie is escaping!</p>
      <p v-else class="mt-2" style="color: #1a237e;">🕓 Next escape in: {{ runAwayCooldown }}s</p>
      <p class="mt-2" style="color: #2e7d32;">🎁 Bonus in: {{ bonusTimer }}s</p>
    </div>

    <div class="column is-6 has-background-link has-text-white p-4">
      <h2 class="is-size-4">Achievements:</h2>
      <ul>
        <li v-for="ach in achievements" :key="ach">✅ {{ ach }}</li>
      </ul>
    </div>

    <div class="column is-2 has-background-warning">
      <button
        v-for="building in buildings"
        :key="building.name"
        :disabled="cookies < building.price"
        @click="buyBuilding(building)"
        class="button is-primary is-large is-fullwidth my-2"
      >
        {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
      </button>
    </div>
  </div>
</template>

