<script setup>
import { computed, ref } from 'vue';

const cookies = ref(0);

const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1000, cps: 10, count: 0 },
    { name: 'Factory', price: 10_000, cps: 100, count: 0 },
]);

const upgrades = ref([
    { name: 'Better Ovens', effect: () => multiplyCps(2), price: 500, bought: false },
    { name: 'Fertilized Fields', effect: () => multiplyCps(2), price: 5000, bought: false },
]);

// Achievements list
const achievements = ref([]);
const achieved = ref([]);

// Shiny cookie state
const shinyCookie = ref({ visible: false, x: 0, y: 0 });

// Function to buy a building
function buyBuilding(building) {
    if (cookies.value >= building.price) {
        cookies.value -= building.price;
        building.price = Math.ceil(building.price * 1.15);
        building.count++;
        checkAchievements();  // Check achievements after purchasing
    }
}

// Function to buy an upgrade
function buyUpgrade(upgrade) {
    if (cookies.value >= upgrade.price && !upgrade.bought) {
        cookies.value -= upgrade.price;
        upgrade.effect();
        upgrade.bought = true;
    }
}

// Function to multiply CPS (used for upgrades)
function multiplyCps(factor) {
    buildings.value.forEach(building => {
        building.cps *= factor;
    });
}

// Computed total CPS
let cps = computed(() => {
    let totalCps = 0;
    buildings.value.forEach(building => {
        totalCps += building.cps * building.count;
    });
    return totalCps;
});

// Function to check achievements
function checkAchievements() {
    const newAchievements = [
        { name: "100 Cookies!", condition: () => cookies.value >= 100 },
        { name: "10,000 Cookies!", condition: () => cookies.value >= 10_000 },
        { name: "1 Million Cookies!", condition: () => cookies.value >= 1_000_000 },
        { name: "First Cursor!", condition: () => buildings.value[0].count >= 1 },
        { name: "10 Grandmas!", condition: () => buildings.value[1].count >= 10 },
        { name: "25 Farms!", condition: () => buildings.value[2].count >= 25 },
        { name: "50 Factories!", condition: () => buildings.value[3].count >= 50 }
    ];

    newAchievements.forEach(achievement => {
        if (achievement.condition() && !achieved.value.includes(achievement.name)) {
            achieved.value.push(achievement.name);
        }
    });
}

// Auto-generate cookies based on CPS every second
setInterval(() => {
    cookies.value += cps.value;
    document.title = '🍪 ' + cookies.value.toFixed(1) + ' Cookies!';
    checkAchievements();  // Check achievements regularly as cookies accumulate
}, 1000);

// Function to spawn a shiny cookie randomly
function spawnShinyCookie() {
    shinyCookie.value.x = Math.random() * 80 + 10; // Random X position (10% to 90% of screen)
    shinyCookie.value.y = Math.random() * 80 + 10; // Random Y position (10% to 90% of screen)
    shinyCookie.value.visible = true;

    setTimeout(() => {
        shinyCookie.value.visible = false;
        spawnNextShiny();
    }, 5000);
}

// Function to collect shiny cookie
function collectShinyCookie() {
    cookies.value += cps.value * 10;
    shinyCookie.value.visible = false;
    spawnNextShiny();
}

// Function to schedule the next shiny cookie spawn
function spawnNextShiny() {
    const delay = Math.random() * 20000 + 10000; // Random delay between 10-30 seconds
    setTimeout(spawnShinyCookie, delay);
}

// Start first shiny cookie spawn
spawnNextShiny();
</script>

<template>
    <div class="columns">
        <!-- Main cookie counter -->
        <div class="column is-4 has-background-primary has-text-centered">
            <h1 class="is-size-1">{{ cookies.toFixed(1) }} cookies</h1>
            <h3 class="is-size-3">{{ cps.toFixed(1) }} cps</h3>
            <figure class="image is-square" @click="cookies++">
                <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
            </figure>
        </div>

        <!-- Upgrades and Buildings section -->
        <div class="column is-4 has-background-info">
            <h2 class="is-size-2">Upgrades</h2>
            <button v-for="upgrade in upgrades" 
                    :disabled="cookies < upgrade.price || upgrade.bought" 
                    @click="buyUpgrade(upgrade)" 
                    class="button is-success is-fullwidth">
                {{ upgrade.name }} 🍪{{ upgrade.price }} 
                <span v-if="upgrade.bought">✅</span>
            </button>

            <h2 class="is-size-2">Buildings</h2>
            <button v-for="building in buildings" 
                    :disabled="cookies < building.price" 
                    @click="buyBuilding(building)" 
                    class="button is-primary is-fullwidth">
                {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
            </button>
        </div>

        <!-- Achievements section moved to the bottom -->
        <div class="column is-4 has-background-warning">
            <h2 class="is-size-2">Achievements</h2>
            <ul>
                <li v-for="achievement in achieved" class="has-text-white">
                    🎉 {{ achievement }}
                </li>
            </ul>
        </div>
    </div>

    <!-- Shiny Cookie -->
    <div v-if="shinyCookie.visible"
         class="shiny-cookie"
         :style="{ top: shinyCookie.y + 'vh', left: shinyCookie.x + 'vw' }"
         @click="collectShinyCookie">
        ✨🍪✨
    </div>
</template>

<style scoped>
.columns {
    margin: 1.5rem;
}

/* Shiny cookie styles */
.shiny-cookie {
    position: fixed;
    font-size: 2.5rem;
    cursor: pointer;
    animation: glow 1s infinite alternate;
}

@keyframes glow {
    from {
        text-shadow: 0 0 0.625rem gold;
    }
    to {
        text-shadow: 0 0 1.25rem orange, 0 0 1.875rem yellow;
    }
}
</style>