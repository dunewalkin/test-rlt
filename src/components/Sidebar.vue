<template>
   <transition name="slide">
      <div v-if="isOpen" class="sidebar">
         <button class="close-button" @click="closeSidebar"></button>
         <div class="item-wrapper">
            <div class="item-image-wrapper">
               <img 
                  class="item-image" 
                  :src="selectedItem ? selectedItem.image : '../assets/images/Drag-element-1.svg'" 
                  alt="Selected item"
               />            
            </div>

            <div class="item-content">
               <img src="../assets/images/Skeleton.svg" alt="">
            </div>

            <div class="line"></div>

            <div class="form-wrapper">
               <template v-if="mode === 'edit'">
                  <input 
                     type="number" 
                     placeholder="Введите количество"
                     min="1" 
                     class="input-field"
                     v-model.number="count"
                  />
                  <div class="buttons">
                     <button class="cancel-button" @click="closeSidebar">Отмена</button>
                     <button class="confirm-button" @click="updateCount">Подтвердить</button>
                  </div>
               </template>

               <template v-else-if="mode === 'delete'">
                  <div class="buttons">
                     <button class="delete-button" @click="confirmDelete">Удалить предмет</button>
                  </div>
               </template>
            </div>
         </div>
      </div>
   </transition>
</template>

<script setup lang="ts">
   import { defineProps, defineEmits, ref, watch } from 'vue';

   const props = defineProps({
      isOpen: Boolean,
      selectedItem: Object as () => { image: string; count: number } | null,
      mode: String, // 

   });

   const count = ref<number>(1);

   watch(() => props.selectedItem, (newItem) => {
      if (newItem) count.value = newItem.count;
   }, { immediate: true });

   const emit = defineEmits(['close', 'update', 'delete']);

   const closeSidebar = () => {
      emit('close');
   };

   const updateCount = () => {
      emit('update', count.value);
   };

   const confirmDelete = () => {
      emit('delete', props.selectedItem);
      closeSidebar();
   };
</script>


<style scoped lang="scss">
   .sidebar {
      position: absolute;
      top: 0;
      right: 0;
      width: 50%;
      height: 100%;
      border-left: 1px solid var(--border-color);
      background: rgba(38, 38, 38, 0.50);
      backdrop-filter: blur(8px);
      padding-inline: 20px;
   }

   .close-button {
      position: absolute;
      width: 24px;
      height: 24px;
      top: 8px;
      right: 8px;
      background-image: url(../assets/images/Close-icon.svg);
      background-color: transparent;
      border: none;
   }

   .item-wrapper {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
   }

   .item-image-wrapper {
      width: 100%;
      display: flex;
      justify-content: center;
      border-bottom: 1px solid var(--border-color);
      padding-block: 55px 30px;
   }

   .item-image {
      width: 130px;
      height: 130px;
   }

   .item-content {
      width: 100%;
      padding-top: 16px;
      height: 150px;
      overflow-y: auto;
      padding-bottom: 10px; 
   }

   .item-content img {
      width: 100%;
   }

   .line {
      border-bottom: 1px solid var(--border-color); 
      width: calc(100% + 40px);
   }

   .form-wrapper {
      width: 100%;
      padding-block: 20px;
   }

   .form-wrapper input {
      width: 100%;
      border-radius: 4px;
      border: 1px solid var(--border-color);
      background: var(--secondary-background);
      color: var(--primary-white);
      padding: 12px;
   }

   .form-wrapper input::placeholder {
      color: var(--primary-white);
      opacity: 0.4;
   }

   .form-wrapper input:hover {
      border-color: var(--primary-white);
   }

   .form-wrapper input:focus {
      border-color: var(--primary-red);
      outline: none;
      box-shadow: 0 0 5px rgba(250, 114, 114, 0.5);
   }

   .form-wrapper input[type="number"]::-webkit-outer-spin-button,
   .form-wrapper input[type="number"]::-webkit-inner-spin-button {
      -webkit-appearance: none;
      margin: 0;
   }

   .buttons {
      width: 100%;
      margin-top: 20px;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
   }

   .cancel-button, 
   .confirm-button,
   .delete-button {
      border-radius: 8px;
      border: none;
      font-family: "SF Pro Display";
      font-size: 14px;
      font-style: normal;
      font-weight: 400;
      line-height: normal;
      padding: 8px 28px;
      transition: background 0.2s ease, transform 0.1s ease;
   }

   .cancel-button:hover {
      background: rgba(255, 255, 255, 0.8);
   }

   .confirm-button:hover {
      background: var(--secondary-red);
   }

   .cancel-button:active, 
   .confirm-button:active {
      transform: scale(0.95);
   }

   .cancel-button:focus, 
   .confirm-button:focus {
      outline: 2px solid var(--primary-red);
      outline-offset: 2px;
   }

   .cancel-button {
      background: var(--primary-white);
      color: var(--primary-black);
   }

   .confirm-button,
   .delete-button {
      background: var(--primary-red);
      color: var(--primary-white);
   }

   .delete-button {
      width: 100%;
   }

   .slide-enter-active, .slide-leave-active {
      transition: transform 0.3s ease-in-out;
   }

   .slide-enter-from, .slide-leave-to {
      transform: translateX(100%);
   }
</style>
