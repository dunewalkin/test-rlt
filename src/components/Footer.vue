<template>
   <footer class="footer">
      <div class="skeleton-wrapper">
         <div class="skeleton"></div>
      </div>
      <button class="close-wrapper"></button>
   </footer>
 </template>

<style scoped lang="scss">

.footer {
   background-color: var(--secondary-background);
   width: 100%;
   height: 100%;
   border-radius: var(--border-radius);
   border: 1px solid var(--border-color);

   display: flex;
   flex-direction: row;
   justify-content: space-between;

   grid-column: 1 / span 2; 
   grid-row: 2;
}

.skeleton-wrapper {
   width: 100%;
   height: 100%;
   display: flex;
   align-items: center;
}

.skeleton {
   display: flex;
   align-items: center;
   width: 90%;
   height: 36px;
   border-radius: 12px;
   background: linear-gradient(90deg, #3C3C3C 0%, #444 51.04%, #333 100%);
   margin-left: 18px;
   background-size: 200% 100%;
   animation: skeleton-loading 1.5s infinite linear;
}

@keyframes skeleton-loading {
   0% {
      background-position: -200% 0;
   }
   100% {
      background-position: 200% 0;
   }
}

.close-wrapper {
   width: 24px;
   height: 24px;
   margin: 8px;
   cursor: pointer;
   background-image: url(../assets/images/Close-icon.svg);
   background-color: transparent;
   background-position: center;
   border: none;
   background-repeat: no-repeat;
}

</style>


