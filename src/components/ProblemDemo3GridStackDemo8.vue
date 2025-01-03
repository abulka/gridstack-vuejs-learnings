<template>
    <h3>ProblemDemo3GridStackDemo8.vue</h3>
    <p>Chained computed Reactivity and selection - GridStack.js with Vue.js.</p>
    <p>Works ok now - remember tips in Mac Notes "Reactivity computed debacle" and comments in this code <strong>ProblemDemo3GridStackDemo8.vue</strong></p>
    <hr>
    <p>selectedId: {{ selectedId }} selectedGridStackNode.id: {{ selectedGridStackNode?.id }}</p>
    <p v-if="selectedGridStackNode">selectedGridStackNode.x: {{ selectedGridStackNode?.x }} y: {{
        selectedGridStackNode?.y }}</p>
    <p v-else>no node selected</p>
    <!-- <p>updateTrigger {{ updateTrigger }}</p> -->
    <p>computed3 {{ computed3?.x }} {{ computed3?.y  }}</p>
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
const updateTrigger = ref(0)

/**
 * This computed is the key to reactivity. It is triggered by changing the
 * selectedId value. It returns a new object to ensure the UI is re-rendered.
 *
 * Three things to watch out for:
 * 1. When using javascript 'find' that refers to a reactive property, a reactivity change
 *    is not detected, even if you return a new object. Need to have an explicit
 *    dependency outside the find() call e.g. const currentSelectedId =
 *    selectedId.value. No need to return a new object.
 * 2. Using a fake trigger e.g. updateTrigger.value to cause the computed to
 *    re-run (partial solution)
 * 3. Even with a fake trigger, the computed must return a new object to trigger
 *    the UI to re-render P.S. A slight workaround is to actually render the
 *    updateTrigger value in the template (in which case everything re-renders,
 *    an incidentally the selectedGridStackNode IS re-rendered - but that is a
 *    bit of luck and a hack).
 */

// Solution 1
// const selectedGridStackNode = computed<GridStackNode | null>(() => {
//     console.log(`  selectedGridStackNode being computed`);
//     const currentSelectedId = selectedId.value;  // explicit dependency NEEDED due to use of find()
//     const result = grid?.engine.nodes.find(node => node.id === currentSelectedId) || null;
//     return result // no need to return a new object
// });

// Solution 2
const selectedGridStackNode = computed<GridStackNode | null>(() => {
    console.log(`  selectedGridStackNode being computed`);

    updateTrigger.value;  // explicit dependency on fake trigger NEEDED!!
    const currentSelectedId = selectedId.value;  // explicit dependency NEEDED due to use of find()

    const result = grid?.engine.nodes.find(node => node.id === currentSelectedId) || null;

    // even though this computed may finally trigger ok, its not actually
    // re-rendering the UI unless we return a new object
    return { x: result?.x, y: result?.y, id: result?.id };  // this is a new object
});

// Question - Will this also be triggered because selectedGridStackNode is triggered? YES
// Question - Do we need to return a new object here? NO
const computed3 = computed<GridStackNode | null>(() => {
    console.log(`  computed3 being computed`);
    return selectedGridStackNode.value;
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

    // The Problem: ❌ The User interaction of moving a widget, or calling update() to programmatically 
    // move a widget, doesn't trigger the computed selectedGridStackNode to re-run because the gridstack
    // object is not reactive. Thus we don't see the x, y, id of the selectedGridStackNode update in the UI.
    // 
    // So we intercept the 'change' event and do something here to trigger the computed.

    // solution 1 - to trigger the computed - WORKS
    // grid.on("change", function (event, items) {
    //     console.log("change", items);
    //     // solution 1 - WORKS
    //     selectedId.value = '';
    //     selectedId.value = items[0].id || '';
    // });

    // solution 1A - fails
    // grid.on("change", function (event, items) {
    //     console.log("change", items);
    //     // Attempt to set selectedId to a new 'version' of the same value, like a new object - but
    //     // for a string? DOESN'T WORK
    //     selectedId.value = items[0].id ? `${items[0].id}` : '';
    // });

    // solution 2 - fake trigger - WORKS
    grid.on("change", function (event, items) {
        console.log("change", items);
        // The computed must ALSO return a new object to trigger the UI to re-render.
        updateTrigger.value++;
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
