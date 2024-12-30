<template>
    <main id="app">
      <a href="./index.html">Back to All Demos</a>
      <h1>Vue3: Gridstack Controls Vue Rendering Grid Items</h1>
      <p>
        <strong>Use Vue3 render functions with GridStack.renderCB</strong><br />
        GridStack handles widget creation and Vue handles rendering the content
        using the modern (since V11) GridStack.renderCB.
      </p>
      <p>Helpful Resources:</p>
      <ul>
        <li>
          <a
            href="https://vuejs.org/guide/extras/render-function.html#render-functions-jsx"
            target="_blank"
            >Vue Render Functions</a
          >
        </li>
        <li><a href="https://github.com/gridstack/gridstack.js/pull/2905" target="_blank">Gridstack pull request associated with this demo</a></li>
      </ul>
      <button type="button" @click="addNewWidget">Add Widget</button> {{ info }}
      <div class="grid-stack"></div>
    </main>
  </template>
  
  <script setup>
  import { ref, onMounted, onBeforeUnmount, h, render } from 'vue';
  import GridItemComponent from './widgets/DemoGridItemComponent.vue';
  
  import { GridStack } from 'gridstack';
  //import { GridStack, type GridStackNode, Utils } from 'gridstack';
  //import type { GridStackOptions, GridItemHTMLElement } from 'gridstack'
  //import type { GridStackWidget } from 'gridstack'
  
  const info = ref('');
  let grid = null;
  const items = [
    { id: 1, x: 2, y: 1, h: 2 },
    { id: 2, x: 2, y: 4, w: 3 },
    { id: 3, x: 4, y: 2 },
    { id: 4, x: 3, y: 1, h: 2 },
    { id: 5, x: 0, y: 6, w: 2, h: 2 },
  ];
  let count = ref(items.length);
  const shadowDom = {};
  
  onMounted(() => {
    grid = GridStack.init({
      float: true,
      cellHeight: '70px',
      minRow: 1,
    });
  
    // Listen for remove events to clean up Vue renders
    grid.on('removed', function (event, items) {
      items.forEach((item) => {
        if (shadowDom[item.id]) {
          render(null, shadowDom[item.id]);
          delete shadowDom[item.id];
        }
      });
    });
  
    GridStack.renderCB = function (el, widget) {
      // el: HTMLElement div.grid-stack-item-content
      // widget: GridStackWidget
  
      const gridItemEl = el.closest('.grid-stack-item'); // div.grid-stack-item (parent of el)
  
      // Create Vue component for the widget content
      const itemId = widget.id;
      const widgetNode = h(GridItemComponent, {
        itemId: itemId,
        onRemove: () => {
          // Catch the remove event from the Vue component
          grid.removeWidget(gridItemEl); // div.grid-stack-item
          info.value = `Widget ${itemId} removed`;
        },
      });
      shadowDom[itemId] = el;
      render(widgetNode, el); // Render Vue component into the GridStack-created element
    };
  
    grid.load(items);
  });
  
  onBeforeUnmount(() => {
    // Clean up Vue renders
    Object.values(shadowDom).forEach((el) => {
      render(null, el);
    });
  });
  
  function addNewWidget() {
    const node = items[count.value] || {
      x: Math.round(12 * Math.random()),
      y: Math.round(5 * Math.random()),
      w: Math.round(1 + 3 * Math.random()),
      h: Math.round(1 + 3 * Math.random()),
    };
    node.id = String(count.value++);
    grid.addWidget(node);
    info.value = `Widget ${node.id} added`;
  }
  </script>
  
  <style>
  /* cannot have scoped or else you see nothing 
  alternatively import the styles in the script section. */
  @import '../assets/demo.css';
  </style>
  