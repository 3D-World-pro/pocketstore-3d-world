<template>
  <section class="header rounded-lg bg-white mb-3 px-3 py-3 mx-3 xl:mx-auto max-w-6xl">
    <div class="flex justify-between">
      <section class="logo">
        <a href="/">
          <img src="/3d-world-pro-black.svg" alt="" class="w-2/3 max-h-16">
        </a>
      </section>
      <section>
        <nav>
          <Bars />
        </nav>
      </section>
    </div>
  </section>
  <Context />
</template>
<script setup lang="ts">
import Bars from "@/components/header/mobile/Bars.vue";
import Context from "@/components/header/mobile/Context.vue";
</script>