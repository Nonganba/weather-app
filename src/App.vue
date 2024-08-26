<template>
  <div
    class="flex flex-col min-h-screen font-Roboto bg-custom-image">
    <SiteNavigation />
    <RouterView class="flex-1" v-slot="{ Component }">
      <Transition name="page">
        <component :is="Component" />
      </Transition>
    </RouterView>
  </div>
</template>

<script setup>
import { RouterView } from "vue-router";
import SiteNavigation from "./components/SiteNavigation.vue";
</script>

<style>
.page-enter-active,
.page-leave-active {
  transition: 600ms ease all;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
}

.bg-custom-image {
  background-size: cover;
  background-position: center;
  background-image: linear-gradient(rgba(255, 255, 255, 0.5), rgba(255, 255, 255, 0.5)), url('@/assets/weather.png');
}
</style>
