<template>
    <h1>Chained computed Reactivity and selection - GridStack.js with Vue.js</h1>
    <p>selectedId: {{ selectedId }} selectedGridStackNode.id: {{ selectedGridStackNode?.id }}</p>
    <p v-if="selectedGridStackNode">selectedGridStackNode.x: {{ selectedGridStackNode?.x }} y: {{
        selectedGridStackNode?.y }}</p>
    <p v-else>no node selected</p>
    <button type="button"
        @click="addNewWidget">Add Widget</button>
    <div class="grid-stack"
        id="demo1-grid"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, computed } from 'vue';
import { GridStack, GridStackWidget } from 'gridstack';
import { type GridStackNode } from 'gridstack'

let nextId = ref(0);
let grid: GridStack | null = null;
const items = [
    { x: 2, y: 1, h: 2, content: 'hi', id: '1' },
    { x: 2, y: 4, w: 3, id: '2' },
    { x: 4, y: 2, id: '3' },
    { x: 3, y: 1, h: 2, id: '4' },
    { x: 0, y: 6, w: 2, h: 2, id: '5' },
];
const selectedId = ref('');

const selectedGridStackNode = computed<GridStackNode | null>(() => {
    console.log(`  selectedGridStackNode being computed`);
    const currentSelectedId = selectedId.value;  // explicit dependency NEEDED!!
    return grid?.engine.nodes.find(node => node.id === currentSelectedId) || null;
});

onMounted(() => {
    grid = GridStack.init({
        minRow: 2,
        alwaysShowResizeHandle: false,
        cellHeight: 'auto',
        float: true,
        margin: 10,
    });
    nextId.value = items.length + 1;

    grid.on("change", function (event, items) {
        console.log("change", items);
        // ❌ this doesn't trigger the computed
    });
    grid.on("dragstop", function (event, element) {
        const node = element.gridstackNode;
    });
    GridStack.renderCB = function (el: HTMLElement, widget: GridStackWidget) {
        el.closest('.grid-stack-item')?.addEventListener('click', (event) => handleItemClick(event, el, widget));
    };
    document.querySelector('.grid-stack')?.addEventListener('click', handleGridClick);
    grid.load(items);
});

const handleItemClick = (event: Event, el: HTMLElement, widget: GridStackWidget) => {
    console.log(`Item click`, event);
    const parentEl: HTMLElement | null = el.closest('.grid-stack-item');
    if (widget.id && parentEl) {
        selectedId.value = widget.id;  // ✅ this is the key to reactivity, changing the selectedId triggers the computed to re-run
        deselectAll();
        parentEl.classList.add('selected');
    }
};

const handleGridClick = (event: Event) => {
    const target = event.target as HTMLElement;
    if (!target.closest('.grid-stack-item')) {
        selectedId.value = '';
        deselectAll();
    }
};

function addNewWidget() {
    const node = items[nextId.value] || {
        x: Math.round(12 * Math.random()),
        y: Math.round(5 * Math.random()),
        w: Math.round(1 + 3 * Math.random()),
        h: Math.round(1 + 3 * Math.random()),
    };
    node.id = String(nextId.value++)
    node.content = node.id
    grid?.addWidget(node);
    selectedId.value = node.id;
}

const deselectAll = () => {
    if (!grid) return;
    const els = grid.el.querySelectorAll('.grid-stack-item');
    els.forEach(el => el.classList.remove('selected'));
};
</script>

<style>
@import '../assets/demo.css';

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
