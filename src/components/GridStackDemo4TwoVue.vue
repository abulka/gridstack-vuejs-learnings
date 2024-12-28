<script setup lang="ts">
import { onMounted, ref, h, render } from 'vue'
import { GridStack, type GridStackNode, Utils } from 'gridstack'
import type { GridStackOptions, GridItemHTMLElement } from 'gridstack'
import type { GridStackWidget } from 'gridstack'
import DemoBlank from './DemoBlank1.vue'
import DemoImage from './DemoImage.vue'

/* Types/Interfaces 
    GridStackWidget has x, y, w, h, id, content + many others
    GridStackWidgetExt adds kind ⇦ for vue component use
    GridStackNode adds el, grid, subGrid
*/
interface GridStackWidgetExt extends GridStackWidget {
  kind?: any
}

let leftGrid: GridStack | null = null;
let rightGrid: GridStack | null = null;

const leftGridFloat = ref(true)
const rightGridFloat = ref(false)

// Set to true to use the renderCB technique Set to false to use the myClone &
// convertToVue technique Both techniques are shown for comparison
//
// Remember to disable the myClone helper function { helper: myClone }, if you
// want to use the renderCB technique, and enable it if you want to use the
// myClone + convertToVue technique
const useRenderCB: boolean = false

// Initial grid items
const items: GridStackNode[] = [
  { x: 0, y: 0, w: 2, h: 2, content: '0' },
  { x: 1, y: 1, h: 2, content: '1' },
  { x: 1, y: 1, content: '2' },
  { x: 3, y: 1, content: '3' },
  { x: 2, y: 3, w: 3, maxW: 3, content: 'has maxW=3' },
  { x: 2, y: 5, content: '5' }
]

// Grid options
const gridOptions: GridStackOptions = {
  // column: 6, // ❌ CHANGING THIS away from 12 BROKE ITEM VISIBILITY
  minRow: 1,
  cellHeight: 70,
  float: true,
  removable: '.trash',
  acceptWidgets: () => true,
  // children: items, // OPTIONAL - pre-populate with items
  margin: 8
}

/* 
Sidebar content for drop creation - the attributes (of type `GridStackWidget`)
to assign to each element on drop. 

You can also use `gridstacknode` attribute in the HTML to specify the attributes
of the node, but the HTML can't specify the `kind` as a non string, so must be
done via this technique.

Each item in this array corresponds to its item in the sidebar, based on order.
*/
const sidebarContent: GridStackWidgetExt[] = [
  { content: 'dropped', id: 'dup_id' },
  { content: 'max=3 ha ha', w: 2, h: 1, maxW: 3 },
  // andy
  { content: 'dropped', id: 'fred_id' },
  // you can supply content in gridstacknode attribute as well
  { content: 'dropped', kind: DemoImage, id: 'mary_id' },
  { content: 'dropped', kind: DemoBlank, id: 'sam_id', w: 4, h: 3 },
  { content: 'dropped', kind: DemoImage, id: 'jo_id' },
  {},
  {},
  {},
  {}
]

// Lifecycle
onMounted(() => {
  // Initialize grids
  const grids = GridStack.initAll(gridOptions)
  leftGrid = grids[0]
  rightGrid = grids[1]

  // Set right grid to non-floating
  rightGrid.float(false)

  // Setup drag-in functionality
  GridStack.setupDragIn(
    '.sidebar-item, .sidebar>.grid-stack-item',
    { helper: myClone }, // when using myClone + convertToVue, causes renderCB not to be called
    // {}, // when using renderCB, don't need myClone
    sidebarContent
  )

  // Add events to both grids
  grids.forEach((grid, i) => addGridEvents(grid, i))

  /*
  Callback to create the content of widgets so the app can control how to store
  and restore it By default this lib will do 'el.textContent = w.content'
  forcing text only support for avoiding potential XSS issues.

  🤯 You don't need `myClone` if you use this callback

  🤯 You don't need `convertToVue` if you use this callback

  📝 But... this callback is only called if 
    - you have an entry in the sidebarContent array
    - you have disabled the `myClone` helper function { helper: myClone },
  */
  GridStack.renderCB = function (el, w) {
    if (useRenderCB) {
      // Dynamically render a vue component
      const widgetId = w.id ? w.id : 'no_id'
      const widgetNode = h(DemoImage, { widgetId })
      render(widgetNode, el)
    }
    else
      el.innerHTML = w.content; // default behaviour

    console.log(`🌻 renderCB: el`, el, `w`, w)
  };
})

/*
Helper function for cloning sidebar items. Is passed the full HTML element
dragged `el`.

Special case for fun: If the element has a gs-id of 'manual', create a widget
with a w of 2 and content of 'manually created'. Otherwise, clone the element.
*/
function myClone(el: HTMLElement): HTMLElement {
  let newEl: HTMLElement;

  if (useRenderCB) {
    console.log(`💋💋💋 helper myClone: renderCB of sidebar div - ignoring and letting renderCB do it all`, el);
    return el
  }

  if (el.getAttribute('gs-id') === 'manual') {
    console.log(`💋💋 helper myClone: manual of sidebar div`, el);
    /** create the default grid item divs, and content possibly lazy loaded calling GridStack.renderCB */
    const class_: string = '';
    const n: GridStackNode = { w: 2, content: 'manually created' };
    newEl = Utils.createWidgetDivs(class_, n);
  } else {
    console.log(`💋 helper myClone: pure clone of sidebar div`, el);
    newEl = el.cloneNode(true) as HTMLElement; // also removes the id attribute
  }

  return newEl;
}

function convertToVue(items: GridStackNode[]) {
  for (const item of items) {
    console.log(`✅ convertToVue for item`, item)
    if (!item.el) {
      console.error('  Element is undefined')
      continue
    }
    const itemEl: HTMLElement = item.el
    const itemElContent = itemEl.querySelector('.grid-stack-item-content') as HTMLElement

    const widgetId = item.id

    if (typeof widgetId === 'undefined') {
      console.error('  Widget ID is undefined')
      continue
    }
    console.log(`  Widget ID ${widgetId} found`)

    const kind = item.kind ? item.kind : DemoBlank

    // // dynamically render a vue component, and append it to the grid stack item content
    // // https://vuejs.org/guide/extras/render-function.html
    const widgetNode = h(kind, { widgetId })
    console.log(`widgetNode`, widgetNode)
    render(widgetNode, itemElContent)
  }
}

// Comprehensive event handler
function addGridEvents(grid: GridStack, id: number) {
  const g = id !== undefined ? `grid${id} ` : ''

  grid.on('added', (event: Event, items: GridStackNode[]) => {
    // dumpItems(0)
    if (useRenderCB) return
    convertToVue(items)
    // dumpItems(0)
  })

  grid.on('added removed change', (event: Event, items: GridStackNode[]) => {
    let str = ''
    items.forEach((item) => {
      str += ` (${item.x},${item.y} ${item.w}x${item.h})`
    })
    console.log(`${g}${event.type} ${items.length} items (x,y w h):${str}`)
  })

  grid.on('enable', (event: Event) => {
    console.log(`${g}enable`)
  })

  grid.on('disable', (event: Event) => {
    console.log(`${g}disable`)
  })

  grid.on('dragstart', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const x = el.getAttribute('gs-x')
    const y = el.getAttribute('gs-y')
    console.log(`${g}dragstart ${n.content || ''} pos: (${n.x},${n.y}) = (${x},${y})`)
  })

  grid.on('drag', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const x = el.getAttribute('gs-x')
    const y = el.getAttribute('gs-y')
    // console.log(`${g}drag ${n.content || ''} pos: (${n.x},${n.y}) = (${x},${y})`)
  })

  grid.on('dragstop', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const x = el.getAttribute('gs-x')
    const y = el.getAttribute('gs-y')
    console.log(`${g}dragstop ${n.content || ''} pos: (${n.x},${n.y}) = (${x},${y})`)
  })

  grid.on('dropped', (event: Event, previousNode: GridStackNode | undefined, newNode: GridStackNode | undefined) => {
    if (previousNode) {
      console.log(`${g}dropped - Removed widget from grid:`, previousNode)
    }
    if (newNode) {
      console.log(`${g}dropped - Added widget in grid:`, newNode)
    }
  })

  grid.on('resizestart', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const rec = el.getBoundingClientRect()
    console.log(`${g}resizestart ${n.content || ''} size: (${n.w}x${n.h}) = (${Math.round(rec.width)}x${Math.round(rec.height)})px`)
  })

  grid.on('resize', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const rec = el.getBoundingClientRect()
    console.log(`${g}resize ${n.content || ''} size: (${n.w}x${n.h}) = (${Math.round(rec.width)}x${Math.round(rec.height)})px`)
  })

  grid.on('resizestop', (event: Event, el: GridItemHTMLElement) => {
    const n = el.gridstackNode
    const rec = el.getBoundingClientRect()
    console.log(`${g}resizestop ${n.content || ''} size: (${n.w}x${n.h}) = (${Math.round(rec.width)}x${Math.round(rec.height)})px`)
  })
}

// Methods
function toggleFloat(index: number) {
  const grid = index === 0 ? leftGrid : rightGrid
  if (!grid) return

  const newFloat = !grid.getFloat()
  grid.float(newFloat)

  if (index === 0) {
    leftGridFloat.value = newFloat
  } else {
    rightGridFloat.value = newFloat
  }
}

function compact(index: number) {
  const grid = index === 0 ? leftGrid : rightGrid
  grid?.compact()
}

function dumpItems(index: number) {
  const grid = index === 0 ? leftGrid : rightGrid
  const loadedNodes = grid?.engine.nodes || []; // Get loaded nodes
  console.log('----- dumpItems', loadedNodes);
}

</script>

<template>
  <div class="grid-demo">
    <h1>Two Grids Demo - vue components</h1>
    <p>
      Two grids, one floating one not, showing drag &amp; drop from sidebar and between grids.
      <br>
      Use 'Esc' to cancel any move/resize. Use 'r' to rotate as you drag.
    </p>

    <div class="main-content">
      <div class="left-section">
        <div class="sidebar">

          <div class="sidebar-item">Drag me</div>

          <div class="sidebar-item">2x1, max=3</div>

          <!-- andy palette -->
          <div class="sidebar-item">Drag me2</div>

          <div class="sidebar-item"
            gs-id="mary">Drag me3
          </div>

          <div class="sidebar-item"
            gridstacknode='{ "id": "meee", "h": 3, "w": 4, "content": "me4" }'>
            Drag me4
          </div>
          <!-- andy ends -->

          <div class="sidebar-item"
            gridstacknode='{ "w": 3, "content": "w:3" }'>
            w:3
          </div>

          <div class="sidebar-item"
            gs-id="manual">
            gs-id = manual
          </div>

          <div class="grid-stack-item"
            gs-w="3">
            <div class="grid-stack-item-content">
              DOM gs-w:3
            </div>
          </div>

          <div class="grid-stack-item"
            gridstacknode='{ "w": 2 }'>
            <div class="grid-stack-item-content">
              DOM w:2
            </div>
          </div>

          <div class="grid-stack-item"
            gs-w="3"
            gs-h="3">
            <div class="grid-stack-item-content">
              🎄DOM
            </div>
          </div>

          <div class="grid-stack-item"
            gridstacknode='{ "w": 4, "h": 1 }'>
            <div class="grid-stack-item-content">
              👉 DOM
            </div>
          </div>

          <div class="sidebar-item"
            gridstacknode='{ "w": 4, "h": 1 }'>
            👉 Drag me
          </div>

          <div class="sidebar-item">Drag me!!</div>

        </div>
      </div>
      <div class="right-section">
        <div class="trash">
        </div>
      </div>
    </div>

    <div class="grids-container">
      <div class="grid-section">
        <div class="grid-controls">
          <button class="control-btn"
            :class="{ 'float-active': leftGridFloat }"
            @click="toggleFloat(0)">
            float: {{ leftGridFloat }}
          </button>
          <button class="control-btn"
            @click="compact(0)">Compact</button>
        </div>
        <div class="grid-stack"
          id="left_grid"></div>
      </div>

      <div class="grid-section">
        <div class="grid-controls">
          <button class="control-btn"
            :class="{ 'float-active': rightGridFloat }"
            @click="toggleFloat(1)">
            float: {{ rightGridFloat }}
          </button>
          <button class="control-btn"
            @click="compact(1)">Compact</button>
        </div>
        <div class="grid-stack"
          id="right_grid"></div>
      </div>
    </div>
  </div>
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