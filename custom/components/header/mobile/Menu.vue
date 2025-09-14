<template>
  <div class="divider divider-primary">Menu</div>
  <div class="grid grid-cols-6 gap-3">
    <div class="col-span-3">
      <select v-model="lang" class="select select-primary w-full">
        <option value="de">{{ $t('lang.de') }}</option>
        <option value="en">{{ $t('lang.en') }}</option>
      </select>
    </div>
    <div class="col-span-3">
      <select v-model="currency" class="select select-primary w-full">
        <option value="€">{{ $t('checkout.currency.euro') }}</option>
        <option value="$">{{ $t('checkout.currency.dollar') }}</option>
      </select>
    </div>
    <div class="col-span-3">
      <a href="/de/category/bowling" class="btn btn-primary btn-block">{{ $t('catalog.category.bowling') }}</a>
    </div>
    <div class="col-span-3">
      <a href="/de/category/nerd" class="btn btn-primary btn-block">{{ $t('catalog.category.nerdy') }}</a>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { useLocalStorage } from "@vueuse/core";
import { usePocketBase } from "~/util/pocketbase";

const pb = usePocketBase();
const lang = useLocalStorage("lang", "de", {});
const currency = useLocalStorage("currency", "euro", {});
</script>
