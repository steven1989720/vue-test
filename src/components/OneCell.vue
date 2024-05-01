<template>
    <div
      class="cell"
      :style="{ transform: isHovered ? 'scale(0.8)' : 'scale(1)' }"
      @mouseover="isHovered = true"
      @mouseleave="isHovered = false"
    >
      {{ number }}
    </div>
  </template>
  
  <script>
  export default {
    props: {
      number: Number
    },
    data() {
      return {
        isHovered: false
      };
    }
  };
  </script>
  
  <style scoped>
  .cell {
    width: 50px;
    height: 50px;
    border: 1px solid black;
    border-radius: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2px;
    transition: transform 0.3s ease;
  }
  </style>
  