<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { GridStack, type GridStackNode, Utils } from 'gridstack'
import type { GridStackOptions } from 'gridstack'

// Types
interface SidebarItem extends GridStackNode {
  content: string
  id?: string
}

// State
const leftGrid = ref<GridStack | null>(null)
const rightGrid = ref<GridStack | null>(null)
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
  column: 6,
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
  if (el.getAttribute('gs-id') === 'manual') {
    return Utils.createWidgetDivs(undefined, { w: 2, content: 'manual' })
  }
  return el.cloneNode(true) as HTMLElement
}

// Grid event handler
function addGridEvents(grid: GridStack, index: number) {
  grid.on('added removed change', function(event: Event, items: GridStackNode[]) {
    let str = ''
    items.forEach(function(item: GridStackNode) {
      str += ' (x,y)=' + item.x + ',' + item.y
    })
    console.log(`${event.type} ${index} items:${str}`)
  })
}

// Methods
function toggleFloat(index: number) {
  const grid = index === 0 ? leftGrid.value : rightGrid.value
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
  const grid = index === 0 ? leftGrid.value : rightGrid.value
  grid?.compact()
}

// Lifecycle
onMounted(() => {
  // Initialize grids
  const grids = GridStack.initAll(gridOptions)
  leftGrid.value = grids[0]
  rightGrid.value = grids[1]
  
  // Set right grid to non-floating
  rightGrid.value.float(false)
  
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
      Two grids, one floating one not, showing drag&drop from sidebar and between grids.
      <br>
      Use 'Esc' to cancel any move/resize. Use 'r' to rotate as you drag.
    </p>

    <div class="main-content">
      <div class="left-section">
        <div class="sidebar">
          <div class="sidebar-item">Drag me</div>
          <div class="sidebar-item">2x1, max=3</div>
          <div class="sidebar-item" :gridstacknode='{"w":3, "content":"w:3"}'>w:3</div>
          <div class="sidebar-item" gs-id="manual">gs-id case</div>
          <div class="grid-stack-item" gs-w="3">
            <div class="grid-stack-item-content">DOM gs-w:3</div>
          </div>
          <div class="grid-stack-item" :gridstacknode='{"w":2}'>
            <div class="grid-stack-item-content">DOM w:2</div>
          </div>
        </div>
      </div>
      <div class="right-section">
        <div class="trash">
          <div class="trash-icon">🗑️</div>
        </div>
      </div>
    </div>

    <div class="grids-container">
      <div class="grid-section">
        <div class="grid-controls">
          <button 
            class="control-btn" 
            :class="{ 'float-active': leftGridFloat }"
            @click="toggleFloat(0)"
          >
            float: {{ leftGridFloat }}
          </button>
          <button class="control-btn" @click="compact(0)">Compact</button>
        </div>
        <div class="grid-stack" id="left_grid"></div>
      </div>

      <div class="grid-section">
        <div class="grid-controls">
          <button 
            class="control-btn" 
            :class="{ 'float-active': rightGridFloat }"
            @click="toggleFloat(1)"
          >
            float: {{ rightGridFloat }}
          </button>
          <button class="control-btn" @click="compact(1)">Compact</button>
        </div>
        <div class="grid-stack" id="right_grid"></div>
      </div>
    </div>
  </div>
</template>

<style>
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

.sidebar {
  background: rgba(144, 238, 144, 0.2);
  padding: 15px;
  border-radius: 4px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.sidebar-item {
  background: rgb(102, 187, 152);
  color: white;
  padding: 10px;
  border-radius: 4px;
  cursor: move;
  min-width: 100px;
  text-align: center;
  border: 2px dashed rgba(0, 0, 0, 0.1);
}

.trash {
  background: rgba(255, 192, 192, 0.2);
  min-height: 100px;
  border: 2px dashed rgba(255, 0, 0, 0.3);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.trash-icon {
  font-size: 24px;
  opacity: 0.5;
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

.grid-stack {
  background: rgba(255, 255, 224, 0.6);
  border-radius: 4px;
  min-height: 400px;
}

.grid-stack-item-content {
  background: rgb(102, 187, 152);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  font-weight: 500;
}

/* GridStack specific styles */
.grid-stack > .grid-stack-item > .grid-stack-item-content {
  inset: 4px;
}

.grid-stack > .grid-stack-item > .ui-resizable-se {
  right: 2px;
  bottom: 2px;
}
</style>