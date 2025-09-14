<template>
  <section class="page">
    <div class="mx-auto max-w-6xl grid grid-cols-6 gap-3">
      <HomepageHero />
      <CatalogProductGrid identifier="bowling" />
    </div>
  </section>
</template>

<script setup lang="ts">
useHead({
    title: '3D-World.pro',
    meta: [
      {
        name: 'description',
        content: 'PocketStore.io is a Open Source Shop Framework which is based on PocketBase and made with Nuxt/VueJs by Jonathan Martz.'
      }
    ]
});
</script>