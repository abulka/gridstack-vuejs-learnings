<!-- filepath: /Users/andy/Devel/toolback-project/gridstack.js/demo/GridItemComponent.vue -->
<template>
    <button @click="handleRemove">X</button>
    <p>Vue Grid Item {{ itemId }}</p>
  </template>
  
  <script setup>
  import { toRefs, onBeforeUnmount } from 'vue';
  
  const props = defineProps({
    itemId: {
      type: [String, Number],
      required: true,
    },
  });
  
  const emit = defineEmits(['remove']);
  const { itemId } = toRefs(props);
  
  onBeforeUnmount(() => {
    console.log(`In vue onBeforeUnmount for item ${itemId.value}`);
  });
  
  function handleRemove() {
    emit('remove');
  }
  </script>
  
  <style scoped>
  /* Add your styles here */
  </style>
  