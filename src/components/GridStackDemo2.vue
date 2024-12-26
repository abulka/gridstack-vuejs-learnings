<script setup lang="ts">
import { h, onMounted, render, watchEffect } from 'vue'
import { GridStack } from 'gridstack';
import DemoBlank from './DemoBlank1.vue'
import { type GridStackNode } from 'gridstack';

let grid: GridStack | null = null;

onMounted(() => {
    grid = GridStack.init({
        margin: 12,
        cellHeight: 100,
        float: true,
        // disableOneColumnMode: true,
        acceptWidgets: true,
        minRow: 1,
    })

    grid.on('added', function (event: Event, items: GridStackNode[]) {
        for (const item of items) {
            const itemEl = item.el as HTMLElement
            const itemElContent = itemEl.querySelector('.grid-stack-item-content') as HTMLElement

            const widgetId = item.id

            if (typeof widgetId === 'undefined') {
                continue
            }

            // dynamically render a vue component, and append it to the grid stack item content
            // https://vuejs.org/guide/extras/render-function.html
            const widgetNode = h(DemoBlank, { widgetId })
            render(widgetNode, itemElContent)
        }
    });

    grid.on('removed', function (event, items) {
        for (const item of items) {
            const itemEl = item.el
            if (!itemEl) continue // ANDY added

            const itemElContent = itemEl.querySelector('.grid-stack-item-content')
            if (!itemElContent) continue // ANDY added

            // Unmount the vue node from the item element
            // Calling render with null will allow vue to clean up the DOM, and trigger lifecycle hooks
            render(null, itemElContent)
        }
    });

    watchEffect(() => {
        console.log('watchEffect')
    })
    
});
</script>
<template>
    <div class="grid-stack grow shrink-0 w-full h-full"></div>
</template>