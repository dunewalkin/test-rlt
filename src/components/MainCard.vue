<template>
      <aside class="sidebar">
         <img class="sidebar__avatar" src="../assets/images/Blur.png" alt="Product image" />
         <img src="../assets/images/Skeleton.svg" alt="Skeleton">   
      </aside>
   </template>
   
<style scoped lang="scss">
   .sidebar {
      background-color: var(--secondary-background);
      border-radius: var(--border-radius);
      border: 1px solid var(--border-color);
   
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center; 
      gap: 20px;
   
      padding: 18px 14px 24px;
   
      grid-column: 1;
      grid-row: 1;
   
      width: 100%; 
      height: 100%;
   }
   
   .sidebar img {
      max-width: 100%; 
      height: auto; 
   }
   
   .skeleton {
      max-width: 100%; 
      height: auto;
   }
</style>

