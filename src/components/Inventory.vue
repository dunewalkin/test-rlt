<template>
   <main class="inventory">
      <div class="inventory-item" v-for="(_, cellIndex) in inventoryCells" :key="cellIndex">
         <div class="draggable-container" v-if="getDraggableInCell(cellIndex)">
            <div
               v-if="draggingItemId === getDraggableInCell(cellIndex)!.id && dragging"
               class="dragging-item"
               :style="draggingStyles"
               >
                  <img :src="getDraggableInCell(cellIndex)!.image" alt="Drag element">
            </div>
            <div
               v-else
               class="centered-item"
               @mousedown="startDrag(getDraggableInCell(cellIndex)!, $event)"
               >
                  <img :src="getDraggableInCell(cellIndex)!.image" alt="Drag element">
            </div>
            <button v-if="!(dragging && draggingItemId === getDraggableInCell(cellIndex)!.id)" class="count-button">
               {{ getDraggableInCell(cellIndex)!.count }}
            </button>
         </div>
      </div>
      </main>
   </template>
   
<script setup lang="ts">
   import { ref, onMounted, watch } from 'vue';
   import dragElement1 from '../assets/images/Drag-element-1.svg';
   import dragElement2 from '../assets/images/Drag-element-2.svg';
   import dragElement3 from '../assets/images/Drag-element-3.svg';
   
   interface Draggable {
      id: number;
      cell: number;
      image: string;
      count: number;
   }
   
   // Генерируем 25 ячеек для сетки
   const inventoryCells = ref(Array.from({ length: 25 }, (_, i) => i));

   // Значения по умолчанию
   const defaultDraggables: Draggable[] = [
      { id: 1, cell: 0, image: dragElement1, count: 2 },
      { id: 2, cell: 1, image: dragElement2, count: 1 },
      { id: 3, cell: 2, image: dragElement3, count: 6 }
   ];

   // Массив с перемещаемыми элементами
   const draggables = ref<Draggable[]>(defaultDraggables);

   // Состояние перетаскивания
   const draggingItemId = ref<number | null>(null);
   const dragging = ref(false);
   const draggingStyles = ref<Record<string, string>>({});
   
   // Функция для получения элемента, находящегося в заданной ячейке
   const getDraggableInCell = (cellIndex: number): Draggable | undefined => {
      return draggables.value.find(item => item.cell === cellIndex);
   };

   // При монтировании компонента загружаем сохраненные данные, если они есть
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

   // Отслеживаем изменения массива и сохраняем его в localStorage
   watch(draggables, (newValue) => {
      localStorage.setItem('draggables', JSON.stringify(newValue));
   }, { deep: true });
   
   const startDrag = (draggable: Draggable, event: MouseEvent) => {
      draggingItemId.value = draggable.id;
      dragging.value = true;
      
      // Запоминаем начальные координаты курсора
      const startX = event.clientX;
      const startY = event.clientY;
      
      // Определяем размеры и положение ячейки, в которой находится элемент
      const target = event.currentTarget as HTMLElement;
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
         height: `${targetRect.height}px`
      };
      };
   
      const onMouseUp = (e: MouseEvent) => {
      window.removeEventListener('mousemove', onMouseMove);
      window.removeEventListener('mouseup', onMouseUp);
      
      // Вычисляем новую ячейку, в которую попал элемент
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
      
      // Проверка: если ячейка уже занята другим элементом, не обновляем позицию
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
</script>

<style scoped lang="scss">
   .inventory {
      background-color: var(--secondary-background);
      border-radius: var(--border-radius);
      border: 1px solid var(--border-color);

      grid-column: 2;
      grid-row: 1;

      display: grid;
      grid-template-columns: repeat(5, 1fr);
      grid-template-rows: repeat(5, 1fr);
   }

   .inventory-item {
      border: 0.5px solid var(--border-color);
      box-sizing: border-box;
      height: 100%;
      position: relative;
   }

   .inventory-item:nth-child(-n+5) {
      border-top: none;
   }

   .inventory-item:nth-child(n+21) {
      border-bottom: none;
   }

   .inventory-item:nth-child(5n) {
      border-right: none;
   }

   .inventory-item:nth-child(5n+1) {
      border-left: none;
   }

   .centered-item {
      padding: 23px; 
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100%;
      width: 100%;
      cursor: grab;
   }

   .dragging-item {
      padding: 23px;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: var(--secondary-background); 
      border: 1px solid var(--border-color);
      border-radius: 24px;
      cursor: grabbing;
      user-select: none;
   }

   .count-button {
      position: absolute;
      bottom: 0;
      right: 0;
      width: 16px;
      height: 16px;
      border-radius: 6px 0px 0px 0px;
      border: 1px solid var(--border-color);
      background-color: var(--secondary-background);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      padding: 0;

      color:  var(--primary-white);
      opacity: 0.4;
      text-align: center;
      font-family: Inter;
      font-size: 10px;
      font-style: normal;
      font-weight: 500;
      line-height: normal;
   }
</style>
 