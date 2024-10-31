<script setup lang="ts">
</script>

<template>
  <div class="h-screen w-screen flex items-center justify-center bg-white">
    <div>
      <h1 class="text-3xl">
        Gsap Animation Demo
      </h1>
      <ul>
        <li>
          <NuxtLink class="link" to="/fireworks_animation">
            fireworks_animation
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="link" to="/fireworks_animation_1">
            fireworks_animation_1
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="link" to="/fireworks_animation_2">
            fireworks_animation_2
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="link" to="/fireworks_animation_3">
            fireworks_animation_3
          </NuxtLink>
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="css" scoped>
.link {
  @apply text-blue-500 underline underline-blue hover:text-blue-700;
}
</style>
