<template>
  <dialog id="modal_mobile_menu" class="modal px-6" :open="navigation">
    <div class="modal-box">
      <HeaderMobileLogo />
      <HeaderMobileMenu />
      <div class="modal-action">
        <form method="dialog">
          <!-- if there is a button in form, it will close the modal -->
          <button @click="navigation = false" class="btn">{{ $t('btn.close') }}</button>
        </form>
      </div>
    </div>
    <label class="modal-backdrop" @click="navigation = false" for="modal_mobile_menu">Close</label>
  </dialog>
</template>

<script setup lang="ts">
import {useLocalStorage} from "@vueuse/core";
const navigation = useLocalStorage('navigation', false)
onMounted(()=>{
  navigation.value = false;
});
</script>
