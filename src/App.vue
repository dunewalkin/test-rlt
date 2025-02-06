<template>
   <div class="main-page">
     <div class="component component--first"></div>
     <div class="component component--second"></div>
     <div class="component component--third"></div>
   </div>
</template>

<style scoped lang="scss">

   .main-page {
      width: 100%;
      height: 100%;
      background-color: #1D1D1D;
      display: grid;
      grid-template-columns: 2fr 5fr; 
      grid-template-rows: 5fr 1fr;
      gap: 24px;
   }

   .component {
      border-radius: 12px;
      border: 1px solid #4D4D4D;
   }

   .component--first {
      grid-column: 1;
      grid-row: 1;
   }

   .component--second {
      grid-column: 2;
      grid-row: 1;
   }

   .component--third {
      grid-column: 1 / span 2; 
      grid-row: 2;
   }
</style>
 