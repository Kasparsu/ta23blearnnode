<script setup>
import { computed, ref, onMounted } from 'vue'

// Main game state
const cookies = ref(0) // Total cookies
const goldenCookieVisible = ref(false) // Show golden cookie
const goldenCookieBoost = ref(false) // Boost active
const goldenCookieTimeout = ref(null) // Timeout handler
const goldenCookiePosition = ref({ top: '50%', left: '50%' }) // Cookie position
const toastMessage = ref('') // Toast text
const showToast = ref(false) // Show toast

// Buildings (auto CPS generators)
const buildings = ref([
  { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
  { name: 'Grandma', price: 100, cps: 1, count: 0 },
  { name: 'Farm', price: 1000, cps: 10, count: 0 },
  { name: 'Factory', price: 10000, cps: 100, count: 0 },
])

const achievements = ref([]) // Unlocked achievements
const upgrades = ref([]) // Earned upgrades

// Buy a building
function buyBuilding(building) {
  cookies.value -= building.price
  building.price += Math.ceil(building.price * 0.15)
  building.count++
  checkForUpgrades()
  checkAchievements()
}

// Total cookies per second
let cps = computed(() => {
  let total = 0
  buildings.value.forEach(b => total += b.cps * b.count)
  if (upgrades.value.includes('Fast Fingers')) total *= 2
  return total
})

// Cookie generation loop
setInterval(() => {
  cookies.value += cps.value * (goldenCookieBoost.value ? 10 : 1)

  // Update page title
  if (goldenCookieVisible.value) {
    document.title = '🌟 GOLDEN COOKIE! 🍪'
  } else if (goldenCookieBoost.value) {
    document.title = '🚀 Boosted! 🍪 ' + cookies.value.toFixed(1)
  } else {
    document.title = '🍪 ' + cookies.value.toFixed(1) + ' Cookies!'
  }
}, 1000)

// Random golden cookie spawns
onMounted(() => {
  setInterval(() => {
    if (!goldenCookieVisible.value && Math.random() < 0.6) {
      spawnGoldenCookie()
    }
  }, 15000)
})

// Show golden cookie at random position
function spawnGoldenCookie() {
  goldenCookieVisible.value = true
  goldenCookiePosition.value = {
    top: `${Math.floor(Math.random() * 80) + 5}%`,
    left: `${Math.floor(Math.random() * 80) + 5}%`,
  }

  goldenCookieTimeout.value = setTimeout(() => {
    goldenCookieVisible.value = false
  }, 5000)
}

// Handle golden cookie click
function clickGoldenCookie() {
  const el = document.querySelector('.golden-cookie img')
  if (el) {
    el.classList.add('golden-clicked')
    setTimeout(() => el.classList.remove('golden-clicked'), 800)
  }

  goldenCookieVisible.value = false
  goldenCookieBoost.value = true
  show("🌟 Golden Cookie activated! x10 CPS")
  clearTimeout(goldenCookieTimeout.value)

  setTimeout(() => {
    goldenCookieBoost.value = false
  }, 10000)
}

// Show toast message
function show(message) {
  toastMessage.value = message
  showToast.value = true
  setTimeout(() => {
    showToast.value = false
  }, 3000)
}

// Unlock achievements
function checkAchievements() {
  const unlocked = []

  if (cookies.value >= 1000) unlocked.push("Cookie Enthusiast 🍪")
  if (buildings.value.find(b => b.name === 'Grandma')?.count >= 10) unlocked.push("Grandma's Favorite 👵")
  if (buildings.value.find(b => b.name === 'Factory')?.count >= 1) unlocked.push("Industrialist 🏭")

  unlocked.forEach(name => {
    if (!achievements.value.includes(name)) {
      achievements.value.push(name)
      show("🏆 Achievement unlocked: " + name)
    }
  })
}

// Unlock upgrades
function checkForUpgrades() {
  const cursor = buildings.value.find(b => b.name === 'Cursor')
  if (cursor.count >= 5 && !upgrades.value.includes('Fast Fingers')) {
    upgrades.value.push('Fast Fingers')
    show("⚡ Upgrade unlocked: Fast Fingers (x2 CPS)")
  }
}
</script>

<template>
  <div class="columns is-gapless main-container">
    <!-- Left Column -->
    <div class="column is-4 has-text-centered p-5 left-panel">
      <h1 class="is-size-1 has-text-white">{{ cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3 has-text-light">{{ cps.toFixed(1) }} cps</h3>

      <div class="cookie-container" @click="cookies++">
        <img
          class="cookie-img"
          src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511"
          alt="Main Cookie"
        />
      </div>
    </div>

    <!-- Middle Column -->
    <div class="column is-6 middle-panel">
      <div class="has-text-centered p-6 is-size-4 has-text-light">
        <p>Welcome to Cookie Clicker Deluxe 🍪</p>
        <p>Golden Cookies appear randomly – click fast!</p>

        <div class="mt-6">
          <h3 class="has-text-white is-size-5 mb-2">🏆 Achievements</h3>
          <ul>
            <li v-for="a in achievements" :key="a" class="has-text-warning">
              {{ a }}
            </li>
          </ul>

          <h3 v-if="upgrades.length" class="has-text-white is-size-5 mt-5 mb-2">🧠 Upgrades</h3>
          <ul>
            <li v-for="u in upgrades" :key="u" class="has-text-info">
              {{ u }}
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Right Column -->
    <div class="column is-2 building-panel p-3">
      <button
        v-for="building in buildings"
        :key="building.name"
        :disabled="cookies < building.price"
        @click="buyBuilding(building)"
        class="button is-success is-fullwidth mb-3 building-button"
      >
        {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
      </button>
    </div>

    <!-- Golden Cookie -->
    <div
      v-if="goldenCookieVisible"
      :style="{
        position: 'absolute',
        top: goldenCookiePosition.top,
        left: goldenCookiePosition.left,
        zIndex: 9999,
      }"
      @click="clickGoldenCookie"
      class="golden-cookie"
    >
      <img
        src="https://pngimg.com/uploads/cookie/cookie_PNG13657.png"
        alt="Golden Cookie"
        class="golden-cookie-img"
      />
    </div>

    <!-- Toast Notification -->
    <transition name="toast-fade">
      <div v-if="showToast" class="toast">
        {{ toastMessage }}
      </div>
    </transition>
  </div>
</template>

<style scoped>
.main-container {
  height: 100vh;
  position: relative;
  overflow: hidden;
}

.left-panel {
  background: linear-gradient(135deg, #00c9ff, #92fe9d);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.middle-panel {
  background: linear-gradient(135deg, #667eea, #764ba2);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.building-panel {
  background: linear-gradient(135deg, #f7971e, #ffd200);
  overflow-y: auto;
}

.cookie-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 300px;
  width: 100%;
}

.cookie-img {
  width: 250px;
  height: 250px;
  transition: transform 0.1s ease-in-out;
}
.cookie-img:hover {
  transform: scale(1.05);
}
.cookie-img:active {
  animation: shake 0.2s;
}

@keyframes shake {
  0% { transform: scale(1.05) rotate(0deg); }
  25% { transform: scale(1.05) rotate(2deg); }
  50% { transform: scale(1.05) rotate(-2deg); }
  75% { transform: scale(1.05) rotate(2deg); }
  100% { transform: scale(1.05) rotate(0deg); }
}

.golden-cookie-img {
  width: 60px;
  transition: transform 0.2s;
}
.golden-clicked {
  animation: explode 0.6s ease-out;
}

@keyframes explode {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.8) rotate(15deg);
    opacity: 0.8;
  }
  100% {
    transform: scale(0.1) rotate(-45deg);
    opacity: 0;
  }
}

.building-button {
  font-weight: bold;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
}

.toast {
  position: absolute;
  top: 20%;
  left: 50%;
  transform: translateX(-50%);
  background: #222;
  color: white;
  padding: 1rem 2rem;
  border-radius: 10px;
  font-size: 1.25rem;
  font-weight: bold;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
  z-index: 10000;
  animation: popIn 0.3s ease-out;
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: opacity 0.5s;
}
.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
}

@keyframes popIn {
  from {
    opacity: 0;
    transform: translateX(-50%) scale(0.8);
  }
  to {
    opacity: 1;
    transform: translateX(-50%) scale(1);
  }
}
</style>
