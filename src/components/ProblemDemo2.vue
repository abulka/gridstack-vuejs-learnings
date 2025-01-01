<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, h, render, watch, type Component, shallowRef, markRaw, toRaw } from 'vue';
import ProblemWidget from './widgets/ProblemWidget2.vue'
import ProblemWidgetEditor from './widgets/ProblemWidget2Editor.vue'
import { computed } from 'vue';

const specs = [
  { id: '1', kind: ProblemWidget, kindEditor: ProblemWidgetEditor },
  { id: '2', kind: markRaw(ProblemWidget), kindEditor: markRaw(ProblemWidgetEditor) },
];
const componentInstances = [
  h(specs[0].kind, { itemId: specs[0].id }),
  h(specs[1].kind, { itemId: specs[1].id }),
];

const selectedWidget = ref(specs[0]);
const selectedWidget2 = ref(specs[1]);

const getKindEditor = computed(() => { 
  // works without warnings
  return ProblemWidgetEditor
});

const getKindEditor1 = computed(() => {  
  // introduces the problem
  // causes Vue received a Component that was made a reactive object. 
  return selectedWidget.value?.kindEditor || null;
});

const getKindEditor2 = computed(() => { 
  // markRaw doesn't help the problem
  // still causes Vue received a Component that was made a reactive object. 
  return markRaw(selectedWidget.value?.kindEditor) || null;
});

const getKindEditor3 = computed(() => { 
  // shallowRef doesn't help the problem
  // causes main.ts:7 [Vue warn]: Component is missing template or render function:  
  return shallowRef(selectedWidget.value?.kindEditor) || null;
});

// the fix

const getKindEditor4 = computed(() => {  
  // FIXES the problem 🎉
  // using markRaw in the specs seems to be the solution
  return selectedWidget.value?.kindEditor || null;
});

</script>

<template>

    <!-- no warnings  -->
    <Component :is="ProblemWidgetEditor" />

    <!-- no warnings  -->
    <Component :is="specs[0].kindEditor" />

    <!-- these all introduces the problem   -->
    <!-- <Component :is="getKindEditor1" /> -->
    <!-- <Component :is="getKindEditor2" /> -->
    <!-- <Component :is="getKindEditor3" /> -->

    <!-- fix -->
    <Component :is="getKindEditor4" />

</template>
