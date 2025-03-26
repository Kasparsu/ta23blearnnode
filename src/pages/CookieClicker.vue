<script setup>
import { computed, ref, onMounted } from 'vue'
import { toast } from 'bulma-toast' // Added bulma-toast import

// Main game state (объединение всех ref в один)
const gameState = ref({
  cookies: 99999,
  goldenCookieClicked: ref(false),
  goldenCookieVisible: false,
  goldenCookieBoost: false,
  goldenCookieTimeout: null,
  goldenCookiePosition: { top: '50%', left: '50%' },
  buildings: [
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1000, cps: 10, count: 0 },
    { name: 'Factory', price: 10000, cps: 100, count: 0 },
  ],
  achievements: [],
  upgrades: [],
})

// Define achievements with a check function and unique ID
const milestones = [
  {
    id: 1,
    name: "Cookie Enthusiast 🍪",
    description: "Reach 1000 cookies.",
    check: (cookies) => cookies >= 1000,
    effect: () => {},  // No effect
    isAchievement: true  // Add this property
  },
  {
    id: 2,
    name: "Grandma's Favorite 👵",
    description: "Have 10 Grandmas.",
    check: (cookies, buildings) => buildings.find(b => b.name === 'Grandma')?.count >= 10,
    effect: () => {},  // No effect
    isAchievement: true
  },
  {
    id: 3,
    name: "Fast Fingers ⚡",
    description: "Have 5 Cursors.",
    check: (cookies, buildings) => buildings.find(b => b.name === 'Cursor')?.count >= 5,
    effect: () => {
      if (!gameState.value.upgrades.includes('Fast Fingers')) {
        gameState.value.upgrades.push('Fast Fingers');
        showToast("⚡ Upgrade unlocked: Fast Fingers (x2 CPS)");
      }
    },
    isAchievement: false  // Set to false to exclude from achievements
  },
  {
    id: 4,
    name: "Industrialist 🏭",
    description: "Own 1 Factory.",
    check: (cookies, buildings) => buildings.find(b => b.name === 'Factory')?.count >= 1,
    effect: () => {},  // No effect
    isAchievement: true
  },
  // Add more as needed
];


// Buy a building
function buyBuilding(building) {
  gameState.value.cookies -= building.price
  building.price += Math.ceil(building.price * 0.15)
  building.count++
  checkForUpgradesAndAchievements()  // Call only once
}

// Total cookies per second
let cps = computed(() => {
  let total = 0
  gameState.value.buildings.forEach(b => total += b.cps * b.count)
  if (gameState.value.upgrades.includes('Fast Fingers')) total *= 2
  return total
})

// Cookie generation loop
setInterval(() => {
  gameState.value.cookies += cps.value * (gameState.value.goldenCookieBoost ? 10 : 1)

  // Update page title
  if (gameState.value.goldenCookieVisible) {
    document.title = '🌟 GOLDEN COOKIE! 🍪'
  } else if (gameState.value.goldenCookieBoost) {
    document.title = '🚀 Boosted! 🍪 ' + gameState.value.cookies.toFixed(1)
  } else {
    document.title = '🍪 ' + gameState.value.cookies.toFixed(1) + ' Cookies!'
  }

  checkForUpgradesAndAchievements()  // Check for achievements after cookies are updated
}, 1000)

// Random golden cookie spawns
onMounted(() => {
  setInterval(() => {
    if (!gameState.value.goldenCookieVisible && Math.random() < 0.6) {
      spawnGoldenCookie()
    }
  }, 15000)
})

// Show toast message using bulma-toast
function showToast(message) {
  toast({
    message: message,
    type: 'is-success',
    position: 'top-center',
    duration: 3000,       
    dismissible: true,   
    closeOnClick: true,
    pauseOnHover: false,
    opacity: 0.9
  })
}
// Show golden cookie at random position
function spawnGoldenCookie() {
  gameState.value.goldenCookieVisible = true
  gameState.value.goldenCookiePosition = {
    top: `${Math.floor(Math.random() * 80) + 5}%`,
    left: `${Math.floor(Math.random() * 80) + 5}%`,
  }

  gameState.value.goldenCookieTimeout = setTimeout(() => {
    gameState.value.goldenCookieVisible = false
  }, 5000)
}

// Handle golden cookie click
function clickGoldenCookie() {
  gameState.value.goldenCookieClicked = true;
  gameState.value.goldenCookieVisible = false;
  gameState.value.goldenCookieBoost = true;
  showToast("🌟 Golden Cookie activated! x10 CPS");
  clearTimeout(gameState.value.goldenCookieTimeout);

  setTimeout(() => {
    gameState.value.goldenCookieBoost = false;
  }, 10000);

  gameState.value.goldenCookieClicked = false;
}

// Combined function to check both upgrades and achievements
function checkForUpgradesAndAchievements() {
  // Check for achievements
  milestones.forEach(milestone => {
    // Check if the milestone should be unlocked
    if (milestone.check(gameState.value.cookies, gameState.value.buildings)) {
      // Apply the milestone's effect whether it's an achievement or not
      milestone.effect();
      
      // Only add to achievements if isAchievement is true and it's not already in the list
      if (milestone.isAchievement && !gameState.value.achievements.some(a => a.id === milestone.id)) {
        gameState.value.achievements.push({
          id: milestone.id,
          name: milestone.name,
          description: milestone.description
        });
        showToast(`🏆 Achievement/unlock: ${milestone.name}`);
      }
    }
  });
}

</script>

<template>
  <div class="columns is-gapless main-container">
    <!-- Left Column -->
    <div class="column is-4 has-text-centered p-5 left-panel">
      <h1 class="is-size-1 has-text-white">{{ gameState.cookies.toFixed(1) }} cookies</h1>
      <h3 class="is-size-3 has-text-light">{{ cps.toFixed(1) }} cps</h3>

      <div class="cookie-container" @click="gameState.cookies++">
        <img class="cookie-img"
          src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511"
          alt="Main Cookie" />
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
            <li v-for="a in gameState.achievements" :key="a.id" class="has-text-warning">
              {{ a.name }}
            </li>
          </ul>

          <h3 v-if="gameState.upgrades.length" class="has-text-white is-size-5 mt-5 mb-2">🧠 Upgrades</h3>
          <ul>
            <li v-for="(u, index) in gameState.upgrades" :key="index" class="has-text-info">
              {{ u }}
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Right Column -->
    <div class="column is-2 building-panel p-3">
      <button v-for="building in gameState.buildings" :key="building.name"
        :disabled="gameState.cookies < building.price" @click="buyBuilding(building)"
        class="button is-success is-fullwidth mb-3 building-button">
        {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
      </button>
    </div>

    <!-- Golden Cookie - moved position and z-index to CSS -->
    <div v-if="gameState.goldenCookieVisible" 
         :style="{
           top: gameState.goldenCookiePosition.top,
           left: gameState.goldenCookiePosition.left
         }" 
         @click="clickGoldenCookie" 
         class="golden-cookie">
      <img src="https://pngimg.com/uploads/cookie/cookie_PNG13657.png" 
           alt="Golden Cookie" 
           class="golden-cookie-img"
           :class="{ 'golden-clicked': gameState.goldenCookieClicked }" 
           @animationend="onAnimationEnd" />
    </div>

    <!-- Toast notifications are now handled by bulma-toast -->
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

/* Golden cookie styling - position and z-index moved here */
.golden-cookie {
  position: absolute;
  z-index: 9999;
}

@keyframes shake {
  0% {
    transform: scale(1.05) rotate(0deg);
  }

  25% {
    transform: scale(1.05) rotate(2deg);
  }

  50% {
    transform: scale(1.05) rotate(-2deg);
  }

  75% {
    transform: scale(1.05) rotate(2deg);
  }

  100% {
    transform: scale(1.05) rotate(0deg);
  }
}

.golden-cookie-img {
  width: 60px;
  transition: transform 0.2s;
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

.golden-clicked {
  animation: explode 0.3s ease-out;
}

.building-button {
  font-weight: bold;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
}

/* Custom toast styles removed as we're using bulma-toast */
</style>