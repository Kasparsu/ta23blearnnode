<script setup>
import { ref } from 'vue';
import RouterTabs from './components/RouterTabs.vue';


</script>
<template>
    <RouterTabs />
    <Suspense>
      <template #default>
        <div v-if="$route.meta.container === undefined" class="container">
          <section class="section">
            <RouterView />
          </section>
        </div>
        <template v-else>
          <RouterView />
        </template>
      </template>
    </Suspense>
  </template>
  