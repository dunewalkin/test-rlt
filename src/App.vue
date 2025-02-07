<script setup lang="ts">
   import Inventory from './components/Inventory.vue';
   import FooterComponent from './components/Footer.vue';
   import MainCard from './components/MainCard.vue';
</script>

<template>
   <div class="main-page">
     <MainCard />
     <Inventory />
     <FooterComponent />
   </div>
 </template>


<style scoped lang="scss">
   .main-page {
      width: 100%;
      height: 100%;
      display: grid;
      grid-template-columns: 2fr 5fr; 
      grid-template-rows: 5fr 1fr;
      gap: 24px;
   }
</style>
 