<script setup>
import { computed, ref, onMounted } from 'vue';
import AchievementBox from './AchievementBox.vue'


const cookies = ref(0);
const buildings = ref([
  { name: 'Cursor',  price: 15,     cps: 5,       count: 0 },
  { name: 'Grandma', price: 100,    cps: 20,         count: 0 },
  { name: 'Farm',    price: 1100,   cps: 2000,         count: 0 },
  { name: 'Factory', price: 12000,  cps: 20000,        count: 0 },
]);


let userclicked = ref(0);
let achievements = computed(() => [
  {
    name: "Click more!",
    desc: "You clicked the cookie 10 times!",
    get condition() {
      return userclicked.value == 10
    }
  },
  {
    name: "Click addict!",
    desc: "50 clicks already?",
    get condition() {
      return userclicked.value == 50
    }
  },
  {
    name: "Click master!",
    desc: "100 clicks. You serious?",
    get condition() {
      return userclicked.value == 100
    }
  },
  {
    name: "Click god!",
    desc: "500 clicks. Seek help.",
    get condition() {
      return userclicked.value == 500
    }
  },
  {
    name: "Endless clicker",
    desc: "1,000 clicks. No turning back.",
    get condition() {
      return userclicked.value == 1000
    }
  },
  {
    name: "Finger of steel",
    desc: "10,000 clicks. What's wrong with you?",
    get condition() {
      return userclicked.value == 10000
    }
  },
  {
    name: "Endless clicker",
    desc: "1,000 clicks. No turning back.",
    get condition() {
      return userclicked.value === 1000;
    }
  },
  {
    name: "Finger of steel",
    desc: "10,000 clicks. What's wrong with you?",
    get condition() {
      return userclicked.value === 10000;
    }
  },
  {
    name: "Factorio! ⚙️",
    desc: "you have 100 factories",
    get condition() {
      for (const item of buildings.value) {
        if (item.name === "Factory" && item.count >= 100) {
          return true;
        }
      }
      return false;
    }
  },
  {
    name: "Grandma's Bakery 👵",
    desc: "You have 10 Grandmas.",
    get condition() {
      for (const item of buildings.value) {
        if (item.name === "Grandma" && item.count === 10) {
          return true;
        }
      }
      return false;
    }
  },
  {
    name: "Farming Tycoon 🌾",
    desc: "You own 5 Farms.",
    get condition() {
      for (const item of buildings.value) {
        if (item.name === "Farm" && item.count === 5) {
          return true;
        }
      }
      return false;
    }
  },
  {
    name: "Clicking Apprentice 🖱️",
    desc: "You've clicked 100 times.",
    get condition() {
      return userclicked.value === 100;
    }
  },
  {
    name: "Cursor Collector 👆",
    desc: "You own 20 Cursors.",
    get condition() {
      for (const item of buildings.value) {
        if (item.name === "Cursor" && item.count === 20) {
          return true;
        }
      }
      return false;
    }
  },
])

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
  cookies.value += 10000;
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
                👵
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