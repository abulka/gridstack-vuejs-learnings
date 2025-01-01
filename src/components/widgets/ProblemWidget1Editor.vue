<script setup lang="ts">
import { defineProps, defineEmits, ref, toRefs, watch } from 'vue';

const props = defineProps({
  componentInstance: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['update:componentInstance']);

const { componentInstance } = toRefs(props);

const localProps = ref({ ...componentInstance.value.props });

watch(localProps, (newVal) => {
  Object.assign(componentInstance.value.props, newVal);
  emit('update:componentInstance', componentInstance.value);
}, { deep: true });


</script>

<template>
  <div>
    <label>
      Widget ID:
      <input v-model="localProps.itemId" />
    </label>
  </div>
</template>
