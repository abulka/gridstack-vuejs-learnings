<script setup lang="ts">
import { h, onMounted, render, watchEffect, ref } from 'vue'
import { GridStack } from 'gridstack';
import { type GridStackNode } from 'gridstack';
import DemoBlank from './widgets/DemoBlank1.vue'
import DemoImage from './widgets/DemoImage1.vue';

let grid: GridStack | null = null;

const items = [
    { x: 5, y: 1, w: 2, h: 2, id: '0', kind: DemoBlank },
    { x: 2, y: 4, w: 3, h: 2, id: '1', kind: 'div' },
    { x: 4, y: 2, w: 2, h: 2, id: '2', kind: DemoImage },
    { x: 3, y: 1, h: 2, id: '3' },
    { x: 0, y: 0, w: 2, h: 2, id: '4' },
];
for (let i = 0; i < items.length; i++) {
    items[i].content = items[i].id;
}
let count = ref(items.length);

onMounted(() => {
    grid = GridStack.init({
        margin: 12,
        cellHeight: 100,
        float: true,
        // disableOneColumnMode: true,
        acceptWidgets: true,
        minRow: 1,
    },
        '.grid-stack'
    )
    grid.load(items); // doesn't trigger 'added, sadly - so manually trigger it

    // Manually trigger the 'added' callback
    const loadedNodes = grid.engine.nodes || []; // Get loaded nodes
    if (loadedNodes.length) {
        addHandler(loadedNodes)
    }

    grid.on('added', function (event: Event, items: GridStackNode[]) {
        addHandler(items)
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
function addHandler(items: GridStackNode[]) {
    console.log(`addHandler, items`, items)
    for (const item of items) {
        // console.log(`is this a node?`, item, item.content)
        const itemEl = item.el as HTMLElement
        const itemElContent = itemEl.querySelector('.grid-stack-item-content') as HTMLElement

        const widgetId = item.id

        if (typeof widgetId === 'undefined') {
            console.error('Widget ID is undefined')
            continue
        }
        console.log(`${widgetId} added`)

        const kind = item.kind ? item.kind : DemoBlank

        // dynamically render a vue component, and append it to the grid stack item content
        // https://vuejs.org/guide/extras/render-function.html
        const widgetNode = h(kind, { widgetId })
        render(widgetNode, itemElContent)
    }
}

function addNewWidget() {
    const node = items[count.value] || {
        x: Math.round(12 * Math.random()),
        y: Math.round(5 * Math.random()),
        w: Math.round(1 + 3 * Math.random()),
        h: Math.round(1 + 3 * Math.random()),
    };
    node.id = String(count.value++)
    node.content = node.id
    // node.content = '<button type="button" @click="addNewWidget">Add Widget</button>'
    // node.content = "<b>hooray</b>"; // why doesnt this work? https://github.com/gridstack/gridstack.js/tree/master/doc#item-attributes 
    grid.addWidget(node);
}
</script>
<template>
    <h1>GridStack hahahahahah</h1>
    <button type="button"
        @click="addNewWidget">Add Widget</button>

    <!-- <div class="grid-stack grow shrink-0 w-full h-full" id="demo2-grid"></div> -->
    <div class="grid-stack"></div>
    <DemoImage imageUrl="/flowers.png" widgetId="flowers-big"/>
</template>

<style>
/* cannot have scoped or else you see nothing */

.grid-stack {
    background: blue;
}

.grid-stack-item {
    /* background: cyan; */
}

.grid-stack-item-content {
    background: green;
}
</style>