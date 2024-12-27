<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { GridStack, type GridStackNode, Utils } from 'gridstack'
import type { GridStackOptions, GridItemHTMLElement } from 'gridstack'

// Types
interface SidebarItem extends GridStackNode {
  content: string
  id?: string
}

let leftGrid: GridStack | null = null;
let rightGrid: GridStack | null = null;

const leftGridFloat = ref(true)
const rightGridFloat = ref(false)

// Initial grid items
const items: GridStackNode[] = [
  { x: 0, y: 0, w: 2, h: 2, content: '0' },
  { x: 1, y: 1, h: 2, content: '1' },
  { x: 1, y: 1, content: '2' },
  { x: 3, y: 1, content: '3' },
  { x: 2, y: 3, w: 3, maxW: 3, content: 'has maxW=3' },
  { x: 2, y: 5, content: '5' }
]

// Sidebar content for drop creation
const sidebarContent: SidebarItem[] = [
  { content: 'dropped', id: 'dup_id' },
  { content: 'max=3', w: 2, h: 1, maxW: 3 }
]

// Grid options
const gridOptions: GridStackOptions = {
  // column: 6, // ❌ CHANGING THIS away from 12 BROKE ITEM VISIBILITY
  minRow: 1,
  cellHeight: 70,
  float: true,
  removable: '.trash',
  acceptWidgets: () => true,
  children: items,
  margin: 8
}

// Helper function for cloning sidebar items
function myClone(el: HTMLElement): HTMLElement {
  console.log('myClone', el)
  if (el.getAttribute('gs-id') === 'manual') {
    return Utils.createWidgetDivs(undefined, { w: 2, content: 'manual' })
  }
  return el.cloneNode(true) as HTMLElement
}

// Comprehensive event handler
function addGridEvents(grid: GridStack, id: number) {
  const g = id !== undefined ? `grid${id} ` : ''

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
    { helper: myClone },
    sidebarContent
  )

  // Add events to both grids
  grids.forEach((grid, i) => addGridEvents(grid, i))
})
</script>

<template>
  <div class="grid-demo">
    <h1>Two Grids Demo</h1>
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
          <div class="sidebar-item"
            gridstacknode='{ "w": 3, "content": "w:3" }'>w:3</div>
          <div class="sidebar-item"
            gs-id="manual">gs-id case</div>
          <div class="grid-stack-item"
            gs-w="3">
            <div class="grid-stack-item-content">DOM gs-w:3</div>
          </div>
          <div class="grid-stack-item"
            gridstacknode='{ "w": 2 }'>
            <div class="grid-stack-item-content">DOM w:2</div>
          </div>
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