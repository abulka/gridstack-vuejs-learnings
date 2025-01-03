<template>
    <h1>Reactivity and selection GridStack.js with Vue.js</h1>
    <p>selected: {{ selectedGridStackNode?.id }}</p>
    <p v-if="selectedGridStackNode">selectedGridStackNode x: {{ selectedGridStackNode?.x }} y: {{ selectedGridStackNode?.y }}</p>
    <p v-else>no node selected</p>
    <button type="button"
        @click="addNewWidget">Add Widget</button> <span class="info">{{ info }}</span>
    <div class="grid-stack"
        id="demo1-grid"></div>
    <!-- </main> -->
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue';
import { GridStack } from 'gridstack';
import { type GridStackNode } from 'gridstack'

let count = ref(0);
let info = ref("");
let timerId = null;
let grid: GridStack | null = null; // DO NOT use ref(null) as proxies GS will break all logic when comparing structures... see https://github.com/gridstack/gridstack.js/issues/2115
const items = [
    { x: 2, y: 1, h: 2, content: 'hi', id: '1' },
    { x: 2, y: 4, w: 3, id: '2' },
    { x: 4, y: 2, id: '3' },
    { x: 3, y: 1, h: 2, id: '4' },
    { x: 0, y: 6, w: 2, h: 2, id: '5' },
];
const selectedId = ref('');

const selectedGridStackNode = computed < GridStackNode | null > (() => {
    const currentSelectedId = selectedId.value;  // explicit dependency NEEDED!!
    console.log(`  selectedGridStackNode being computed`);
    return grid?.engine.nodes.find(node => node.id === currentSelectedId) || null;
});

onMounted(() => {
    grid = GridStack.init({ // DO NOT use grid.value = GridStack.init(), see above
        minRow: 2, // how big the initial canvas is
        alwaysShowResizeHandle: false,
        cellHeight: 'auto',
        float: false,  // false means float everything to the top
        margin: 10,
        // children: items, // if want to add items on init OPTIONAL
    });

    grid.on("change", function (event, items) {
        console.log("change", items);
    });
    grid.on("dragstop", function (event, element) {
        const node = element.gridstackNode;
        info.value = `you just dragged node #${node.id} to ${node.x},${node.y} good job!`;
    });
    count.value = items.length + 1; // start id counter at the length of the items array

    GridStack.renderCB = function (el: HTMLElement, widget: GridStackWidgetExt) {
        // gridItemEl  div.grid-stack-item            (parent of el)
        // el          div.grid-stack-item-content
        console.log(`GridStack.renderCB`, el, widget);

        const parentEl: HTMLElement | null = el.closest('.grid-stack-item');
        if (parentEl) {
            parentEl.addEventListener('click', (event) => {
                console.log(`parentEl click`, event);
                selectedId.value = widget.id;
                deselectAll();
                parentEl.classList.add('selected');
                info.value = `Widget ${widget.id} selected`;
            });
        }
    };

    const gridElement = document.querySelector('.grid-stack');
    if (gridElement) {
        gridElement.addEventListener('click', handleGridClick);
    }

    grid.load(items);
});

const handleGridClick = (event: Event) => {
    const target = event.target as HTMLElement;
    if (!target.closest('.grid-stack-item')) {
        selectedId.value = '';
        deselectAll();
    }
};

function addNewWidget() {
    const node = items[count.value] || {
        x: Math.round(12 * Math.random()),
        y: Math.round(5 * Math.random()),
        w: Math.round(1 + 3 * Math.random()),
        h: Math.round(1 + 3 * Math.random()),
    };
    node.id = String(count.value++)
    node.content = node.id
    grid.addWidget(node);
    selectedId.value = node.id;
}

const deselectAll = () => {
    if (!grid) return;
    const els = grid.el.querySelectorAll('.grid-stack-item');
    els.forEach(el => el.classList.remove('selected'));
};

watch(info, (newVal) => {
    if (newVal.length === 0) return;

    window.clearTimeout(timerId);
    timerId = window.setTimeout(() => {
        info.value = "";
    }, 2000);
});
</script>

<style>
@import '../assets/demo.css';

.info {
    background-color: pink;
    color: black;
    font-size: 0.8em;
    margin-left: 1em;
}


.grid-stack-item .grid-stack-item-content {
    border: 2px solid transparent;
    /* Base style with transparent border */
    /* Override default style so that it always has a border, to avoid size changes */
}

.grid-stack-item.selected .grid-stack-item-content {
    border: 2px solid blue;
    /* Outline style for selected widget */
}
</style>
