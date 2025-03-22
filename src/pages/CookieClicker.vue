<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

const cookies = ref(0);
const buildings = ref([
    { name: 'Cursor', price: 15, cps: 0.1, count: 0},
    { name: 'Grandma', price: 100, cps: 1, count: 0},
    { name: 'Farm', price: 1_000, cps: 10, count: 0},
    { name: 'Factory', price: 10_000, cps: 30, count: 0},
]);

// AutoSave
const saveGameState = () => {
  const gameState = {
    cookies: cookies.value,
    buildings: buildings.value.map(b => ({
      name: b.name,
      price: b.price,
      cps: b.cps,
      count: b.count
    }))
  };
  
  try {
    localStorage.setItem('cookieGameSave', JSON.stringify(gameState));
    console.log('Game saved:', gameState);
  } catch (error) {
    console.error('Save error:', error);
  }
};

// Loading the save
const loadGameState = () => {
  try {
    const savedData = localStorage.getItem('cookieGameSave');
    if (savedData) {
      const parsedData = JSON.parse(savedData);
      cookies.value = parsedData.cookies || 0;
      buildings.value = buildings.value.map(b => {
        const savedBuilding = parsedData.buildings.find(sb => sb.name === b.name);
        return savedBuilding ? {...b, ...savedBuilding} : b;
      });
      console.log('Game loaded:', parsedData);
    }
  } catch (error) {
    console.error('Load error:', error);
  }
};

let saveInterval;

onMounted(() => {
  loadGameState();
  saveInterval = setInterval(saveGameState, 1000);
});

onBeforeUnmount(() => {
  clearInterval(saveInterval);
});

function buyBuilding(building) {
  cookies.value -= building.price;
  building.price += Math.ceil(building.price / 100 * 15);
  building.count++;
}

const cps = computed(() => 
  buildings.value.reduce((acc, building) => 
    acc + (building.cps * building.count), 0)
);

setInterval(() => {
  cookies.value += cps.value;
  document.title = `🍪${cookies.value.toFixed(1)} Cookies!`;
}, 1000);

</script>

<template>
    <div class="columns">
        <div class="column is-4 has-background-grey has-text-centered" >
            <h1 class="is-size-1"> {{ +cookies.toFixed(1) }} cookies </h1>
            <h3 class="is-size-3"> {{ +cps.toFixed(1) }} cps </h3>
            <figure class="image is-square " @click="cookies++">
                <img src="https://www.freeiconspng.com/uploads/download-biscuit-cookie-monster-clipart-24.png">
            </figure>
        </div>

        <div class="column is-6 has-background-grey-light">
          <div class="columns">

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Grandma' ).count >=1">
              <img src="https://png.pngtree.com/png-clipart/20230914/original/pngtree-cute-grandma-clipart-the-cartoon-old-lady-character-has-a-bouquet-png-image_11242831.png"> 
            </figure>

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Grandma' ).count >=10">
                <img src="https://png.pngtree.com/png-clipart/20230914/original/pngtree-cute-grandma-clipart-the-cartoon-old-lady-character-has-a-bouquet-png-image_11242831.png">
            </figure>

          </div>
          <div class="columns">

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Farm' ).count >=1">
                <img src="https://pics.clipartpng.com/Corn_PNG_Clipart-466.png">
            </figure>

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Farm' ).count >=10">
                <img src="https://pics.clipartpng.com/Corn_PNG_Clipart-466.png">
            </figure>

          </div>  
          <div class="columns">

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Factory' ).count >=1">
                <img src="https://www.freeiconspng.com/uploads/gear-icon-11.png">
            </figure>

            <figure class="image is-square image is-96x96" v-if="buildings.find(b => b.name === 'Factory' ).count >=10">
                <img src="https://www.freeiconspng.com/uploads/gear-icon-11.png">
            </figure>
            
          </div>
        </div>

        <div class="column is-2 has-background-white-ter">
            <button v-for="building in buildings" :disabled="cookies<building.price" @click="buyBuilding(building)" class="button is-primary is-large is-fullwidth">
                {{ building.name }} 🍪 {{ building.price }} #{{ building.count }} 
            </button>
        </div>
    </div>
    
</template>