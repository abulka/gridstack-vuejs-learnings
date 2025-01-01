<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, h, render, watch, type Component, shallowRef, markRaw, toRaw } from 'vue';
import ProblemWidget1 from './widgets/ProblemWidget1.vue'
import ProblemWidget1Editor from './widgets/ProblemWidget1Editor.vue'

const specs = [
  { id: '1', kind: ProblemWidget1, kindEditor: ProblemWidget1Editor },
];
const componentInstances = [
  h(ProblemWidget1, { itemId: '101' }),
];
</script>

<template>
    <!-- <Component
    v-if="selectedWidget.kindEditor && selectedWidget.id && componentInstances[selectedWidget.id]"
    :is="currentKind"
    :componentInstance="componentInstances[selectedWidget.id]"
    /> -->

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
</template>
