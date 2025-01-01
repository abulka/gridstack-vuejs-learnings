<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, h, render, watch, type Component, shallowRef, markRaw, toRaw } from 'vue';
import ProblemWidget1 from './widgets/ProblemWidget1.vue'
import ProblemWidget1Editor from './widgets/ProblemWidget1Editor.vue'
import { computed } from 'vue';

const specs = [
  { id: '1', kind: ProblemWidget1, kindEditor: ProblemWidget1Editor },
];
const componentInstances = [
  h(specs[0].kind, { itemId: specs[0].id }),
];

const selectedWidget = ref(specs[0]);

const currentKind = computed(() => { // OK
  return ProblemWidget1Editor
});

const currentKind1 = computed(() => {  // introduces the problem
  return selectedWidget.value?.kindEditor || null;
});

const currentKind2 = computed(() => { // markRaw doesn't help the problem
  return markRaw(selectedWidget.value?.kindEditor) || null;
});
</script>

<template>

    <!-- these all work without warnings  -->
    <ProblemWidget1 itemId="100" />

    <Component
        :is="ProblemWidget1"
        itemId="200"
    />

    <Component
        :is="specs[0].kind"
        itemId="200"
    />

    <ProblemWidget1Editor
        :componentInstance="componentInstances[0]"
    />

    <Component
        :is="ProblemWidget1Editor"
        :componentInstance="componentInstances[0]"
    />

    <Component
        :is="specs[0].kindEditor"
        :componentInstance="componentInstances[0]"
    />

    <!-- this introduces the problem 
    Vue received a Component that was made a reactive object. 
    This can lead to unnecessary performance overhead and should be avoided by marking the component with `markRaw` or using `shallowRef` instead of `ref`. 
    Component that was made reactive:  {__name: 'ProblemWidget1Editor',
    -->
    <Component
        :is="currentKind2"
        :componentInstance="componentInstances[0]"
    />
</template>
