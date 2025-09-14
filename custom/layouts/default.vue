<template>
  <section class="page min-h-screen" data-theme="light">
    <header class="pt-3">
      <Header/>
    </header>
    <main class="mx-auto max-w-6xl min-h-screen px-3 2xl:px-0">
      <Messages/>
      <Breadcrumb/>
      <CookieBanner/>
      <BackToTop/>
      <slot/>
    </main>
    <footer>
      <Footer/>
    </footer>
    <div id="particles-js"></div>
  </section>
</template>
<script setup lang="ts">
import "../main.css";
import Footer from "~/components/Footer.vue";
import Header from "~/components/Header.vue";
</script>

<style scoped>
.page {
  background: #2A7B9B;
  background: linear-gradient(90deg, rgba(42, 123, 155, 1) 0%, rgba(87, 199, 133, 1) 50%, rgba(237, 221, 83, 1) 100%);
}
</style>


