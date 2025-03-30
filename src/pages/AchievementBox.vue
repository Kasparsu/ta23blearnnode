<script setup>
import { ref, watchEffect, reactive } from 'vue';
const props = defineProps(['achievements'])
const achievements_count = ref(props.achievements.length)

// makes the all the objects reactive as well, not just the array.
let localAchievements = reactive(
  props.achievements.map(a => ({ ...a,
    get condition(){
      return a.condition
    },
    shown: false
   }))
)

console.log(localAchievements)
watchEffect(() => {
  localAchievements.forEach(element => {    
    if(element.condition){
      element.shown = true; 
      setTimeout(() => {  
        element.shown = false; 
      }, 3000)
    }
  });
})




</script>

<template>

<div v-for="(achievement, index) in localAchievements" :key="index">
  <div class="card" :class="{'is-hidden': !achievement.shown}">
    <div class="card-content">
      <div class="is-flex is-justify-content-space-between">
        <h1 class="has-text-white is-size-4">{{ achievement.name }}</h1>
        <div>{{ index + 1 }}/{{ achievements_count }}</div>
      </div>
      <div class="has-text-white">{{ achievement.desc }}</div>
    </div>
  </div>
</div>

</template>