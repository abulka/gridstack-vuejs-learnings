<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, h, render } from 'vue';
import GridItemComponent from './DemoGridItemComponent.vue';
import DemoBlank from './DemoBlank1.vue'
import DemoImage from './DemoImage.vue'
import { GridStack } from 'gridstack';
import { type GridStackNode, Utils } from 'gridstack';
import type { GridStackOptions, GridItemHTMLElement } from 'gridstack'
import type { GridStackWidget } from 'gridstack'

/* Types/Interfaces 
    GridStackWidget has x, y, w, h, id, content + many others
    GridStackWidgetExt adds kind ⇦ for vue component use
    GridStackNode adds el, grid, subGrid
*/
interface GridStackWidgetExt extends GridStackWidget {
  kind?: any
}

const info = ref('');
let grid: GridStack | null = null;
const items = [
  { id: 1, x: 0, y: 1, h: 2, w: 2 },
  { id: 2, x: 2, y: 4, h: 2, w: 6 },
  { id: 3, x: 4, y: 2, h: 2, w: 2 },
  { id: 4, x: 2, y: 1, h: 2, w: 2 },
  { id: 5, x: 0, y: 6, w: 2, h: 2 },
];
let count = ref(items.length);
const shadowDom = {};

// Grid options
const gridOptions: GridStackOptions = {
  // column: 6, // ❌ CHANGING THIS away from 12 BROKE ITEM VISIBILITY
  minRow: 1,
  cellHeight: 70,
  float: true,
  removable: '.trash',
  acceptWidgets: () => true,
  // children: items, // OPTIONAL - pre-populate with items or call grid.load(items); later
  margin: 8
}

/* 
Sidebar content for drop creation - the attributes (of type `GridStackWidget`)
to assign to each element on drop. 

You can also use `gridstacknode` attribute in the HTML to specify the attributes
of the node, but the HTML can't specify the `kind` as a non string, so must be
done via this technique.

Each item in this array corresponds to its item in the sidebar, based on order.

You must define some content or the renderCB will not be called.
*/
const sidebarContent: GridStackWidgetExt[] = [
  { content: 'n/a', id: '111', kind: GridItemComponent, w: 2, h: 1 },
  { content: 'n/a', id: '222', kind: DemoImage, w: 2, h: 1, maxW: 4, maxH: 2 },
  { content: 'n/a', id: '333', kind: DemoBlank, w: 4, h: 3 },
  {}
]

onMounted(() => {
  grid = GridStack.init(gridOptions);

  // Setup drag-in functionality
  GridStack.setupDragIn(
    '.sidebar-item, .sidebar>.grid-stack-item',
    {}, // when using renderCB, don't need myClone or added event
    sidebarContent
  )

  // Listen for remove events to clean up Vue renders
  grid.on('removed', function (event, items) {
    items.forEach((item) => {
      if (shadowDom[item.id]) {
        render(null, shadowDom[item.id]);
        delete shadowDom[item.id];
      }
    });
  });

  GridStack.renderCB = function (el: HTMLElement, widget: GridStackWidgetExt) {
    // el: HTMLElement div.grid-stack-item-content
    // widget: GridStackWidget

    const gridItemEl = el.closest('.grid-stack-item'); // div.grid-stack-item (parent of el)

    // Create Vue component for the widget content
    const itemId = widget.id;
    const kind = widget.kind ? widget.kind : GridItemComponent
    const widgetNode = h(kind, {
      itemId: itemId, // TODO all components should have itemId prop and emit remove event
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

function compact(index: number) {
  grid?.compact()
  info.value = `Compact done`
}

function toggleFloat() {
  if (!grid) return

  const newFloat = !grid.getFloat()
  grid.float(newFloat)
  info.value = `Float ${newFloat ? 'enabled' : 'disabled'}`
}

</script>

<template>
  <main id="app">
    <h1>Vue3: Gridstack Drag Drop - Demo 6</h1>
    <p>
      GridStack handles widget creation and Vue handles rendering the content
      using the modern (since V11) GridStack.renderCB. Add <strong>kind</strong>
      prop to the widget data to render different components.
    </p>
    <button type="button"
      @click="addNewWidget">Add Widget</button> {{ info }}
    <button class="control-btn"
      @click="compact(0)">Compact</button>
    <button class="control-btn"
      @click="toggleFloat">Toggle Float</button>

    <div class="main-content">
      <div class="left-section">
        <div class="sidebar">

          <div class="sidebar-item">Item</div>
          <div class="sidebar-item">Image</div>
          <div class="sidebar-item">Blank</div>

        </div>
      </div>
      <div class="right-section">
        <div class="trash">
        </div>
      </div>
    </div>

    <div class="grid-stack"></div>
  </main>
</template>

<style>
/* cannot have scoped or else you see nothing 
alternatively import the styles in the script section. */
@import '../assets/demo.css';

.sidebar {
  /* override the demo.css style - looks better */
  height: auto;
}

.grid-demo {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.main-content {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.left-section {
  flex: 2;
}

.right-section {
  flex: 1;
}

.grids-container {
  display: flex;
  gap: 20px;
}

.grid-section {
  flex: 1;
}

.grid-controls {
  margin-bottom: 10px;
}

.control-btn {
  background: #4a90e2;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  margin-right: 8px;
  cursor: pointer;
}

.control-btn:hover {
  background: #357abd;
}

.float-active {
  background: #45a049;
}
</style>