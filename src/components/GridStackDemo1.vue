<!-- filepath: /Users/andy/Devel/toolback-project/gridstack.js/demo/Vue3Gridstack.vue -->
<template>
    <!-- <main id="app"> -->
        <h1>How to integrate GridStack.js with Vue.js</h1>
        <p>
            As with any virtual DOM based framework, you need to check if Vue has
            rendered the DOM (or any updates to it) <strong>before</strong> you
            initialize GridStack or call its methods. As a basic example, check this
            component's <code>mounted</code> hook.
        </p>
        <p>
            If your app requires more complex render logic than the inline template
            in `addWidget`, consider
            <a href="https://github.com/gridstack/gridstack.js/tree/master/doc#makewidgetel">makeWidget</a>
            to let Vue deal with DOM rendering.
        </p>
        <button type="button"
            @click="addNewWidget">Add Widget</button> <span class="info">{{ info }}</span>
        <p>demo 1 gridstack goes here:</p>
        <div class="grid-stack" id="demo1-grid"></div>
    <!-- </main> -->
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import { GridStack } from 'gridstack';
// import '../assets/demo.css' // or import it in the style section below

let count = ref(0);
let info = ref("");
let timerId = null;
let grid = null; // DO NOT use ref(null) as proxies GS will break all logic when comparing structures... see https://github.com/gridstack/gridstack.js/issues/2115
const items = [
    { x: 2, y: 1, h: 2, content: 'hi' },
    { x: 2, y: 4, w: 3 },
    { x: 4, y: 2 },
    { x: 3, y: 1, h: 2 },
    { x: 0, y: 6, w: 2, h: 2 },
];

onMounted(() => {
    grid = GridStack.init({ // DO NOT use grid.value = GridStack.init(), see above
        minRow: 2, // how big the initial canvas is
        alwaysShowResizeHandle: false,
        cellHeight: 'auto',
        float: false,  // false means float everything to the top
        margin: 10,
        children: items, // if want to add items on init OPTIONAL
    });

    grid.on("dragstop", function (event, element) {
        const node = element.gridstackNode;
        info.value = `you just dragged node #${node.id} to ${node.x},${node.y} – good job!`;
    });
    count.value = items.length + 1; // start id counter at the length of the items array
});

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

watch(info, (newVal) => {
    if (newVal.length === 0) return;

    window.clearTimeout(timerId);
    timerId = window.setTimeout(() => {
        info.value = "";
    }, 2000);
});
</script>

<style>
/* cannot have scoped or else you see nothing 
alternatively import the styles in the script section. */
@import '../assets/demo.css';
.info {
    background-color: pink;
    color: black;
    font-size: 0.8em;
    margin-left: 1em;
}
</style>
