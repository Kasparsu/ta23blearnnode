<script setup>
import { computed, ref } from 'vue';

const cookies = ref(0);
const totalCookies = ref(0); // Track total cookies produced
const achievements = ref([]); // Track unlocked achievements

const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0 },
    { name: 'Grandma', price: 100, cps: 1, count: 0 },
    { name: 'Farm', price: 1000, cps: 10, count: 0 },
    { name: 'Factory', price: 10_000, cps: 100, count: 0 },
    { name: 'Mine', price: 50_000, cps: 50, count: 0 },
    { name: 'Bank', price: 200_000, cps: 200, count: 0 },
]);

const upgrades = ref([
    { name: 'Double Cursor CPS', price: 50, target: 'Cursor', multiplier: 2, purchased: false },
    { name: 'Double Grandma CPS', price: 500, target: 'Grandma', multiplier: 2, purchased: false },
    { name: 'Double Farm CPS', price: 5000, target: 'Farm', multiplier: 2, purchased: false },
    { name: 'Double Factory CPS', price: 50_000, target: 'Factory', multiplier: 2, purchased: false },
    { name: 'Double Mine CPS', price: 100_000, target: 'Mine', multiplier: 2, purchased: false }, // New Upgrade
    { name: 'Double Bank CPS', price: 500_000, target: 'Bank', multiplier: 2, purchased: false }, // New Upgrade
]);

const goldenCookie = ref({ active: false, position: { top: '50%', left: '50%' } });

function buyBuilding(building) {
    if (cookies.value >= building.price) {
        cookies.value -= building.price;
        building.price += Math.ceil(building.price / 100 * 15);
        building.count++;
        checkAchievements(); // Check for achievements after purchase
    }
}

function buyUpgrade(upgrade) {
    if (cookies.value >= upgrade.price && !upgrade.purchased) {
        cookies.value -= upgrade.price;
        const targetBuilding = buildings.value.find(b => b.name === upgrade.target);
        if (targetBuilding) {
            targetBuilding.cps *= upgrade.multiplier;
        }
        upgrade.purchased = true;
    }
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
    totalCookies.value += cps.value; // Track total cookies produced
    document.title = '🍪' + cookies.value.toFixed(1) + ' Cookies!';
}, 1000);

function spawnGoldenCookie() {
    goldenCookie.value.active = true;
    goldenCookie.value.position = {
        top: `${Math.random() * 80 + 10}%`,
        left: `${Math.random() * 80 + 10}%`,
    };
    setTimeout(() => {
        goldenCookie.value.active = false;
    }, 10000);
}

function clickGoldenCookie() {
    cookies.value += 1000;
    goldenCookie.value.active = false;
    checkAchievements();
    console.log(achievements.value); // Debugging
}

setInterval(() => {
    if (!goldenCookie.value.active) {
        spawnGoldenCookie();
    }
}, Math.random() * 30000 + 30000);

// Achievements Logic
function checkAchievements() {
    const newAchievements = [];

    if (totalCookies.value >= 1000 && !achievements.value.includes('First 1,000 Cookies')) {
        newAchievements.push('First 1,000 Cookies');
    }
    if (totalCookies.value >= 100_000 && !achievements.value.includes('100,000 Cookies Club')) {
        newAchievements.push('100,000 Cookies Club');
    }
    if (buildings.value.find(b => b.name === 'Mine')?.count >= 10 && !achievements.value.includes('10 Mines Built')) {
        newAchievements.push('10 Mines Built');
    }
    if (buildings.value.find(b => b.name === 'Bank')?.count >= 5 && !achievements.value.includes('5 Banks Built')) {
        newAchievements.push('5 Banks Built');
    }

    newAchievements.forEach(achievement => {
        achievements.value.push(achievement);
        console.log(`Achievement Unlocked: ${achievement}`); // Debugging
        alert(`Achievement Unlocked: ${achievement}`);
    });
}
</script>

<template>
    <div class="container">
        <!-- Cookie Display -->
        <div class="cookie-display">
            <h1 class="title">{{ +cookies.toFixed(1) }} cookies</h1>
            <h3 class="subtitle">{{ +cps.toFixed(1) }} cps</h3>
            <figure class="cookie" @click="cookies++">
                <img src="https://sweetlorens.com/cdn/shop/products/Copy-of-Chocolate-Chunk-Full-Cookie-transparent-background.png?v=1687811511" />
            </figure>
        </div>

        <!-- Shop and Upgrades -->
        <div class="shop-upgrades">
            <!-- Upgrades Section -->
            <div class="upgrades">
                <h2 class="section-title">Upgrades</h2>
                <button
                    v-for="upgrade in upgrades"
                    :key="upgrade.name"
                    :disabled="cookies < upgrade.price || upgrade.purchased"
                    @click="buyUpgrade(upgrade)"
                    class="button upgrade-button"
                >
                    {{ upgrade.name }} 🍪{{ upgrade.price }} <span v-if="upgrade.purchased">(Purchased)</span>
                </button>
            </div>

            <!-- Buildings Section -->
            <div class="buildings">
                <h2 class="section-title">Shop</h2>
                <button
                    v-for="building in buildings"
                    :key="building.name"
                    :disabled="cookies < building.price"
                    @click="buyBuilding(building)"
                    class="button building-button"
                >
                    {{ building.name }} 🍪{{ building.price }} #{{ building.count }}
                </button>
            </div>
        </div>

        <!-- Achievements -->
        <div class="achievements">
            <h2 class="section-title">Achievements</h2>
            <ul>
                <li v-for="achievement in achievements" :key="achievement">{{ achievement }}</li>
            </ul>
        </div>

        <!-- Golden Cookie -->
        <div
            v-if="goldenCookie.active"
            @click="clickGoldenCookie"
            class="golden-cookie"
            :style="{ top: goldenCookie.position.top, left: goldenCookie.position.left }"
        >
            🍪
        </div>
    </div>
</template>

<style>
/* General Layout */
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    font-family: Arial, sans-serif;
}

/* Cookie Display */
.cookie-display {
    text-align: center;
    margin-bottom: 40px;
}

.cookie-display .title {
    font-size: 3rem;
    color: #4CAF50;
}

.cookie-display .subtitle {
    font-size: 1.5rem;
    color: #555;
}

.cookie {
    cursor: pointer;
    margin-top: 20px;
}

.cookie img {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

/* Shop and Upgrades */
.shop-upgrades {
    display: flex;
    justify-content: space-between;
    width: 100%;
    max-width: 800px;
}

.upgrades, .buildings {
    flex: 1;
    margin: 0 10px;
}

.section-title {
    font-size: 1.5rem;
    margin-bottom: 20px;
    text-align: center;
    color: #333;
}

/* Buttons */
.button {
    display: block;
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    font-size: 1rem;
    text-align: center;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.upgrade-button {
    background-color: #FFC107;
    color: #fff;
}

.upgrade-button:disabled {
    background-color: #FFD54F;
    cursor: not-allowed;
}

.building-button {
    background-color: #4CAF50;
    color: #fff;
}

.building-button:disabled {
    background-color: #81C784;
    cursor: not-allowed;
}

/* Golden Cookie */
.golden-cookie {
    position: absolute;
    width: 50px;
    height: 50px;
    background-color: gold;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 24px;
    font-weight: bold;
    cursor: pointer;
    z-index: 1000;
    box-shadow: 0 0 10px 5px rgba(255, 215, 0, 0.8);
}

/* Achievements Section */
.achievements {
    margin-top: 20px;
    text-align: center;
}

.achievements ul {
    list-style: none;
    padding: 0;
}

.achievements li {
    background-color: #FFD700;
    color: #333;
    padding: 10px;
    margin: 5px 0;
    border-radius: 5px;
    font-weight: bold;
}
</style>