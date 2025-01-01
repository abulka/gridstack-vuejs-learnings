<script setup lang="ts">
import { ref } from 'vue';
const someValue = ref(202);
</script>

<template>
  <div>
    <label>
      Widget Editor:
      <input v-model="someValue" />
    </label>
    {{ someValue }}
  </div>
</template>
