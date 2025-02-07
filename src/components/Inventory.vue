<template>
   <main class="inventory">
   <div class="inventory-item" v-for="(_, cellIndex) in inventoryCells" :key="cellIndex">
      <div class="draggable-container" v-if="getDraggableInCell(cellIndex)">
         <!-- Режим перетаскивания: отображаем элемент, который двигается -->
         <div
         v-if="draggingItemId === getDraggableInCell(cellIndex)!.id && dragging"
         class="dragging-item"
         :style="draggingStyles"
         >  
         <img :src="getDraggableInCell(cellIndex)!.image" alt="Drag element" />
         </div>
         <!-- Режим покоя: отображаем элемент, с обработчиками для начала перетаскивания и открытия окна удаления -->
         <div
         v-else
         class="centered-item"
         @click="openDeleteBar(getDraggableInCell(cellIndex)!)"
         >
         <div class="centered-image-wrapper" @mousedown="startDrag(getDraggableInCell(cellIndex)!, $event)">
            <img :src="getDraggableInCell(cellIndex)!.image" alt="Drag element" />
         </div>
         </div>
         <!-- Кнопка изменения количества (скрыта, если элемент перетаскивается) -->
         <button
         v-if="!(dragging && draggingItemId === getDraggableInCell(cellIndex)!.id)"
         @click="openSidebar(getDraggableInCell(cellIndex)!)"
         class="count-button"
         >
         <span>{{ getDraggableInCell(cellIndex)!.count }}</span>
         </button>
      </div>
   </div>
   
   <!-- Sidebar для редактирования/удаления элемента -->
   <Sidebar 
      :isOpen="isSidebarOpen" 
      :selectedItem="selectedDraggable" 
      :mode="sidebarMode"
      @close="closeSidebar"
      @update="updateCount"
      @delete="deleteItem"
   />
   </main>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import Sidebar from './Sidebar.vue';
import dragElement1 from '../assets/images/Drag-element-1.svg';
import dragElement2 from '../assets/images/Drag-element-2.svg';
import dragElement3 from '../assets/images/Drag-element-3.svg';

interface Draggable {
   id: number;
   cell: number;
   image: string;
   count: number;
}

// Создаем 25 ячеек для сетки
const inventoryCells = ref(Array.from({ length: 25 }, (_, i) => i));

// Элементы инвентаря (дефолтные значения)
const defaultDraggables: Draggable[] = [
   { id: 1, cell: 0, image: dragElement1, count: 4 },
   { id: 2, cell: 1, image: dragElement2, count: 2 },
   { id: 3, cell: 2, image: dragElement3, count: 5 }
];
const draggables = ref<Draggable[]>(defaultDraggables);

// Загрузка сохраненных данных из localStorage при монтировании
onMounted(() => {
   const savedData = localStorage.getItem('draggables');
   if (savedData) {
   try {
      draggables.value = JSON.parse(savedData);
   } catch (error) {
      console.error('Ошибка парсинга сохранённых данных:', error);
   }
   }
});

// Сохраняем изменения в localStorage (глубокий watch)
watch(draggables, (newValue) => {
   localStorage.setItem('draggables', JSON.stringify(newValue));
}, { deep: true });

// Состояние перетаскивания и стили для двигаемого элемента
const draggingItemId = ref<number | null>(null);
const dragging = ref(false);
const draggingStyles = ref<Record<string, string>>({});

// Возвращает draggable-элемент, находящийся в указанной ячейке
const getDraggableInCell = (cellIndex: number): Draggable | undefined => {
   return draggables.value.find(item => item.cell === cellIndex);
};

/**
 * Начало перетаскивания элемента.
 * Определяет позицию и устанавливает необходимые стили.
 */
const startDrag = (draggable: Draggable, event: MouseEvent) => {
   draggingItemId.value = draggable.id;
   dragging.value = true;

   // Добавляем класс для кастомного курсора на контейнере
   document.querySelector('.inventory')?.classList.add('custom-cursor');

   const startX = event.clientX;
   const startY = event.clientY;
   
   // Используем ближайший родительский элемент с классом .centered-item для корректного расчета позиции
   const target = (event.currentTarget as HTMLElement).closest('.centered-item') as HTMLElement;
   if (!target) return;
   
   const targetRect = target.getBoundingClientRect();
   const offsetX = startX - targetRect.left;
   const offsetY = startY - targetRect.top;

   const onMouseMove = (e: MouseEvent) => {
   draggingStyles.value = {
      position: 'fixed',
      left: `${e.clientX - offsetX}px`,
      top: `${e.clientY - offsetY}px`,
      zIndex: '1000',
      pointerEvents: 'none',
      width: `${targetRect.width}px`,
      height: `${targetRect.height}px`,
   };
   };

   const onMouseUp = (e: MouseEvent) => {
   window.removeEventListener('mousemove', onMouseMove);
   window.removeEventListener('mouseup', onMouseUp);

   // Убираем класс кастомного курсора
   document.querySelector('.inventory')?.classList.remove('custom-cursor');

   // Рассчитываем новую позицию в сетке
   const grid = document.querySelector('.inventory') as HTMLElement;
   const gridRect = grid.getBoundingClientRect();
   const cellWidth = gridRect.width / 5;
   const cellHeight = gridRect.height / 5;
   
   const relativeX = e.clientX - gridRect.left;
   const relativeY = e.clientY - gridRect.top;
   let newColumn = Math.floor(relativeX / cellWidth);
   let newRow = Math.floor(relativeY / cellHeight);
   newColumn = Math.max(0, Math.min(4, newColumn));
   newRow = Math.max(0, Math.min(4, newRow));
   const newCell = newRow * 5 + newColumn;

   // Обновляем позицию, если ячейка свободна
   const cellOccupied = draggables.value.some(
      item => item.cell === newCell && item.id !== draggable.id
   );
   if (!cellOccupied) {
      const index = draggables.value.findIndex(item => item.id === draggable.id);
      if (index !== -1) {
         draggables.value[index].cell = newCell;
      }
   }

   draggingItemId.value = null;
   dragging.value = false;
   draggingStyles.value = {};
   };

   window.addEventListener('mousemove', onMouseMove);
   window.addEventListener('mouseup', onMouseUp);
};

// Управление отображением Sidebar (для редактирования или удаления элемента)
const isSidebarOpen = ref(false);
const selectedDraggable = ref<Draggable | null>(null);
const newCount = ref<number>(1);
const sidebarMode = ref<'edit' | 'delete'>('edit');

/**
 * Открывает Sidebar для редактирования количества.
 */
const openSidebar = (draggable: Draggable) => {
   selectedDraggable.value = draggable;
   newCount.value = draggable.count;
   sidebarMode.value = 'edit';
   isSidebarOpen.value = true;
};

/**
 * Открывает Sidebar для удаления элемента.
 */
const openDeleteBar = (draggable: Draggable) => {
   selectedDraggable.value = draggable;
   sidebarMode.value = 'delete';
   isSidebarOpen.value = true;
};

/**
 * Удаляет выбранный элемент.
 */
const deleteItem = (item: Draggable) => {
   draggables.value = draggables.value.filter(d => d.id !== item.id);
};

/**
 * Закрывает Sidebar.
 */
const closeSidebar = () => {
   isSidebarOpen.value = false;
};

/**
 * Обновляет количество выбранного элемента.
 */
const updateCount = (newCountValue: number) => {
   if (selectedDraggable.value) {
   const index = draggables.value.findIndex(item => item.id === selectedDraggable.value!.id);
   if (index !== -1) {
      draggables.value[index].count = newCountValue;
   }
   }
   closeSidebar();
};
</script>

<style scoped lang="scss">
   .inventory {
      position: relative;
      background-color: var(--secondary-background);
      border-radius: var(--border-radius);
      border: 1px solid var(--border-color);

      grid-column: 2;
      grid-row: 1;

      display: grid;
      grid-template-columns: repeat(5, 1fr);
      grid-template-rows: repeat(5, 1fr);
   }

   .custom-cursor {
      cursor: url('@/assets/images/Grabbing-cursor.svg') 12 12, grab;
   }

   .inventory-item {
      border: 0.5px solid var(--border-color);
      box-sizing: border-box;
      height: 100%;
      position: relative;
   }

   .inventory-item:nth-child(-n+5) { border-top: none; }
   .inventory-item:nth-child(n+21) { border-bottom: none; }
   .inventory-item:nth-child(5n) { border-right: none; }
   .inventory-item:nth-child(5n+1) { border-left: none; }

   .draggable-container {
      height: 100%;
   }

   .centered-item {
      padding: 23px; 
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100%;
      width: 100%;
      cursor: url('../assets/images/Hover-cursor.svg') 12 12, auto;
   }

   .centered-item:hover {
      background-color: var(--hover-background); 
      cursor: url('../assets/images/Hover-cursor.svg') 12 12, pointer;
   }

   .centered-image-wrapper {
      cursor: url('../assets/images/Grabbing-cursor.svg') 12 12, grab;
   }

   .dragging-item {
      padding: 23px;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: var(--secondary-background); 
      border: 1px solid var(--border-color);
      border-radius: 24px;
      cursor: url('@/assets/images/Grabbing-cursor.svg') 12 12, grab;
   }

   .count-button {
      position: absolute;
      bottom: 0;
      right: 0;
      border-radius: 6px 0px 0px 0px;
      border: 1px solid var(--border-color);
      background-color: var(--secondary-background);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      padding: 0;
   }

   .count-button span {
      color:  var(--primary-white);
      opacity: 0.4;
      text-align: center;
      font-family: Inter;
      font-size: 10px;
      font-style: normal;
      font-weight: 500;
      line-height: normal;
      padding: 2px 4px;
   }

   
</style>