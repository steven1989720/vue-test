<template>
  <div id="app">
    <!-- Full-page loading overlay -->
    <div v-show="loading" class="loading-overlay">Loading...</div>

    <!-- Your app's content -->
    <div v-if="!loading">
      <div v-for="row in verticalList" :key="row.id" :id="row.id" class="vertical-row">
        <OneCell v-for="cell in row.cells" :key="cell.id" :id="cell.id" :number="cell.number"
        :blinking="cell.blinking"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted, onBeforeUnmount } from 'vue';
  import _ from 'lodash';
  import OneCell from './OneCell.vue';

  let bRenderUpdated = false;
  const MAX_COLUMN_COUNT = 100, FORMAL_ROW_COUNT = 100, UPDATE_MILLISECONDS = 1000;
  
  // v-observe-visibility="visibilityChanged"
  // const visibilityChanged = (isVisible, entry) => {
  //   const element = verticalList.value.find((element) => element.id === id)
  //   if (!element) return
  //     element.isVisible = isVisible
  //   console.log(isVisible, entry);
  // }

  function isElementInViewport(el) {
    const rect = el.getBoundingClientRect();
    return (
      rect.top >= 0 &&
      rect.left >= 0 &&
      rect.top <= (window.innerHeight || document.documentElement.clientHeight) &&
      rect.left <= (window.innerWidth || document.documentElement.clientWidth)
    ) ||
    (
      rect.bottom >= 0 &&
      rect.right >= 0 &&
      rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
      rect.right <= (window.innerWidth || document.documentElement.clientWidth)
    );
  }

  const verticalList = ref([]);
    // const renderList = [];
    let visibleFirstRow = -1, visibleLastRow = -1;

    const loading = ref(true);

    const generateRandomNumber = () => Math.floor(Math.random() * 100);

    const createCell = (rowId, index) => ({
      id: rowId * MAX_COLUMN_COUNT + index,
      number: generateRandomNumber(),
      blinking: false,
      transform: 'scale(1)'
    });

    const createRow = (_, rowIndex) => {
      const rowId = rowIndex + 1;
      return {
        id: rowId,
        cells: Array.from({ length: 10 + Math.floor(Math.random() * 5) }, (_, cellIndex) => createCell(rowId, cellIndex + 1))
      };
    };

    function markVisibleCellFunction() {
      bRenderUpdated = true;
      // renderList.length = 0;
      // let bFound = false;
      visibleFirstRow = visibleLastRow = -1;

      const rows = document.querySelectorAll('.vertical-row');
      for (let i = 0; i < rows.length; i++) {
        const row = rows[i];
        // if (isElementInViewport(row)) {
        //   bFound = true;
        //   const vueRow = verticalList.value.find(r => r.id == row.id);
        //   if (vueRow){
        //     const cells = row.querySelectorAll('.cell');
        //     cells.forEach(cell => {
        //       if (cell && isElementInViewport(cell)){
        //         const vueCell = vueRow.cells.find(c => c.id == cell.id);
        //         if (vueCell) {
        //           renderList.push(vueCell);
        //         }
        //       }
        //     });
        //   }
        // }else if (bFound){
        //   break;
        // }

        if (isElementInViewport(row)){
          if (visibleFirstRow < 0)
            visibleFirstRow = i;
          visibleLastRow = i;
        }else if (visibleFirstRow >= 0)
          break;
      }
    }

    var updateCellVisibility = _.debounce(markVisibleCellFunction, 200);

    onMounted(() => {
      verticalList.value = Array.from({ length: FORMAL_ROW_COUNT + Math.floor(Math.random() * FORMAL_ROW_COUNT / 2) }, createRow);
      // let totalCellCount = 0;

      function updateRenderCells(){
        if (!bRenderUpdated)
          markVisibleCellFunction();

        // let visibleCellCount = 0;

        // if (renderList && renderList.length){
        //   renderList.forEach(vueCell => {
        //     if (vueCell) {
        //       vueCell.number = generateRandomNumber();
        //       // visibleCellCount++;
        //     }
        //   });
        // }
        // console.log(`update ${visibleCellCount}`);

        if (visibleFirstRow >= 0){
          for (let index = visibleFirstRow; index <= visibleLastRow; index++) {
            const row = verticalList.value[index];
            const cellIndex = Math.floor(Math.random() * row.cells.length);
            const cell = row.cells[cellIndex];
            cell.number = generateRandomNumber();
            cell.blinking = true;
            setTimeout(() => {cell.blinking = false;}, 1000); // Blink duration
          }
        }

        setTimeout(updateRenderCells, UPDATE_MILLISECONDS);
      }

      setTimeout(updateRenderCells, UPDATE_MILLISECONDS);

      loading.value = false;

      window.addEventListener('scroll', updateCellVisibility);
      window.addEventListener('resize', updateCellVisibility);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('scroll', updateCellVisibility);
      window.removeEventListener('resize', updateCellVisibility);
    });
</script>

<style>
  .vertical-row {
    display: flex;
  }
  .loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(255, 255, 255, 1);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000; /* Ensure it's above other content */
  }
</style>