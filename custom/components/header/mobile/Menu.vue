<template>
  <div class="divider divider-primary">Menu</div>
  <div class="grid grid-cols-6 gap-3">
    <div v-for="category in categories" class="col-span-3">
      <a href="/de/category/bowling" class="btn btn-primary btn-block">{{category.name}}</a>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { useLocalStorage } from "@vueuse/core";
import { usePocketBase } from "~/utils/pocketbase";

const pb = usePocketBase();
const lang = useLocalStorage("lang", "de", {});
const currency = useLocalStorage("currency", "euro", {});

const categories = ref([]);

const load = async  ()=>{
  categories.value = await pb.collection('categories').getFullList(25);
}

onMounted(load);
</script>
