<template>
  rowsHeight.value - {{ rowsHeight }}
  <div
    class="assembly-object-table no-drag"
    :style="{
      height: props.resizeHeightTable
        ? props.resizeHeightTable - 135 + 'px'
        : props.initialHeight
        ? props.initialHeight
        : '196px',
    }"
  >
    <div ref="table" class="assembly-object-table__table">
      <V2Table v-if="tableData.length" :id="props.tableId" class="table_base">
        <V2TableHead class="table_base__thead">
          <V2TableRow
class="table_base__tr-heading">
            <V2TableCol            
              is="th"
              v-for="(head, index) in props.headers"
              :id="`${props.tableId}_th_${index}`"
              :key="index + 'th'"
              :ref="(el: any) => {
                refStorage[head.code] = el as HTMLTableElement
              }
                "
              :width="colsWidthObj[head.code]"
              class="table_base__th-heading"
              style="position: relative"
              :draggable="isDraggableHeader"
              @dragstart="dragStartHeader(index)"
              @dragover="dragOverHeader(index)"
              @dragenter.prevent
              @drop="dragDropHeader"
            >
              <div class="cell">
                <span>{{ head.name }}</span>
              </div>
              <div
                class="resizer"
                :style="{ height: tableData.length ? heightTable : '50px'}"
                @mousedown.stop="startResizeHeader($event, head.code)"
              ></div>
            </V2TableCol>
          </V2TableRow>
        </V2TableHead>
        <V2TableBody
          v-for="(parametr, index) in tableData"
          :key="index"
          class="table_base__body p-relative"
          :draggable="isDraggableRow"
          :data-index="index"
          @dragstart="dragStart($event, index)"
          @dragover.prevent
          @dragenter.prevent
          @drop="dragDrop($event, index)"
        >
          <ScopeWorkStructureTableRow
            :data="parametr"
            :table-id="props.tableId"
            :height ="rowsHeight[index]"
            :expand="props.expand"
            :headers="props.headers"
            :index="index"
            :are-icons-blue="props.areIconsBlue"
            :are-icons-small="props.areIconsSmall"
            :cols-width="colsWidthObj"
            @delete-row="onDeleteClick"
            @delete-child-row="onDeleteChildRow"
            @change-row="emits('change-row')"
            @update-row="updateRow"
          />
    <div class="resizer-row" :style="{width: generalWidthTable + 'px'}" @mousedown.stop="startResizeRow($event, index)"></div>
        </V2TableBody>
      </V2Table>
      <div v-else class="assembly-object-table__default">нет данных</div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ITableData } from '@/components/library/LibraryRegistry/interfaceLibraryRegistry'
  import {
    IHeaders,
    IRefStore,
  } from '@/components/library/libraryComponents/interfaceLibraryComponents'
  import { IObject } from '@/components/common/interfaceCommon'

  interface IProps {
    tableId: string
    headers: IHeaders[]
    rowTable: ITableData[]
    expand?: boolean
    areIconsBlue?: boolean
    areIconsSmall?: boolean
    resizeHeightTable?: number | null
    initialHeight?: string
    isHideDelete?: boolean
    isDraggable?: boolean
  }
  interface IColsWidthObj {
  [index: string]: string;
}
  const props = withDefaults(defineProps<IProps>(), {
    expand: false,
    areIconsBlue: false,
    areIconsSmall: false,
    isDraggable: true,
  })

  const emits = defineEmits<IEmits>()

  interface IEmits {
    (e: 'delete-row', row: IObject): void
    (e: 'deleteChildRow', row: IObject, parentCode: string): void
    (e: 'change-row'): void
    (e: 'update-row', row: IObject): void
    (e: 'set-columns-width', data: IObject): void
    (e: 'set-columns-order', data: IHeaders[]): void


  }
  const refStorage = reactive<IRefStore>({
    code: null,
    name: null,
    unit: null,
    attrs: null,
    params: null,
    actions: null,
  })

  const table = ref<HTMLTableElement>()
  const tableData = ref<ITableData[]>([])

  const initialHeaders = ref([...props.headers]) 
  const colsWidth = ref(initialHeaders.value.map((header: IObject) => ({code: header.code, width: header.width})))
  const colsWidthObj = computed(() => colsWidth.value.reduce((acc, col) => ({...acc, [col.code]: col.width }), {})) as unknown as IColsWidthObj
  //Общая ширина таблицы
  const generalWidthTable = computed(() => (colsWidth.value.map((header) => ((Number(header?.width?.slice(0, -2))) ))).reduce((total, width) => total + width))
  //Заголовки таблицы с переводом из px в проценты от ширины таблицы
  const headersWidthPersent = computed(() => (colsWidth.value.map((header) => ({code: header.code, width: (Number(header?.width?.slice(0, -2)) * 100 / (generalWidthTable.value as number)).toFixed(2) }))))
  // Объект ширин колонок в процентах
  const colsWidthPersentObj = computed(() => headersWidthPersent.value.reduce((acc, col) => ({...acc, [col.code]: Number(col.width) }), {}))

  const heightTable = ref<string>('0')
  const isDraggableHeader = ref<boolean>(true)
  const isDraggableRow = ref<boolean>(true)

  const onDeleteClick = (row: IObject) => {
    emits('delete-row', row)
  }

  const onDeleteChildRow = (row: IObject, parentCode: string) => {
    emits('deleteChildRow', row, parentCode)
  }

  const dragStart = (e: DragEvent, idx: number) => {
    console.log('isDraggableRow.value', isDraggableRow.value);
    
    if(!isDraggableRow.value) return
    if(!e.dataTransfer) return 
    e.dataTransfer.dropEffect = 'move'
    e.dataTransfer.effectAllowed = 'move'    
    e.dataTransfer.setData('itemID', idx.toString())
  }

  const dragDrop = (e: DragEvent, idx: number): void => {
    if(!e.dataTransfer) return 
    e.stopPropagation()
    e.preventDefault()
    const itemIndex = Number(e.dataTransfer.getData('itemID'))
    const droppedIndex = idx
    changeTable(itemIndex, droppedIndex)
  }

  const changeTable = (itemIndex: number, droppedIndex: number) => {
    const movedItem = tableData.value[itemIndex]
    tableData.value.splice(itemIndex, 1)
    tableData.value.splice(droppedIndex, 0, movedItem)
  }
  const updateRow = (row: IObject) => {
    emits('update-row', row)
  }

  const x = ref(0)
  const widthCurrentColumn = ref(0)
  const newWidthCurrentColumn = ref(0)
  const currentColumn = ref()
  const currentColumnIdx = ref<number>(0)

  const setWidthCols = () => {
    const initialCol = colsWidth.value.map((col: IObject, idx: number) => {
      const curCol = document.querySelector(
        `#${props.tableId}-table-${props.tableId}_th_${idx.toString()}-th`
      )
      return {code: col.code, width: `${(curCol as HTMLElement).offsetWidth}px`}
    })
    const curTtable = document.querySelector(`#${props.tableId}-table`)
    ;(curTtable as HTMLTableElement).style.width = 'auto'
    colsWidth.value = [...initialCol]    

    
  }

  const startResizeHeader = (e: MouseEvent, code: string) => {
    isDraggableHeader.value = false

    const idx = colsWidth.value.findIndex(col => col.code === code)  
    
    currentColumnIdx.value = idx
    x.value = 0

    currentColumn.value = document.querySelector(
      `#${props.tableId}-table-${props.tableId}_th_${idx.toString()}-th`
    )
    if (currentColumn.value) {
      widthCurrentColumn.value = (
        currentColumn.value as HTMLElement
      ).offsetWidth
    }
    x.value = 0
    x.value = e.clientX

    document.addEventListener('mousemove', mouseMoveHandler)
    document.addEventListener('mouseup', mouseUpHandler)
  }

  // ======================
const resizing = ref(false);
    const rowIndex = ref(-1);
    const initialY = ref(0);
    const initialHeight = ref(0);
    const rowsHeight = ref()

  const startResizeRow = (e: MouseEvent, idx: number) => {
    isDraggableRow.value = false
    resizing.value = true
    rowIndex.value = idx
    initialY.value = e.clientY
    initialHeight.value = rowsHeight.value[idx]
    console.log('initialY.value', initialY.value)    
    // initialHeight.value = props.data[index].height;

    document.addEventListener('mousemove', handleResizeRow)
    document.addEventListener('mouseup', stopResizeRow);    
  }

  // const newHeight = ref(50)

   const handleResizeRow = (event: MouseEvent) => {
      if (resizing.value) {        
        const newHeight = initialHeight.value + (event.clientY - initialY.value)
        if (newHeight > 49) {
        rowsHeight.value[rowIndex.value] = newHeight
        }
      }
    }

    const stopResizeRow = () => {
      isDraggableRow.value = true
      resizing.value = false;
      document.removeEventListener('mousemove', handleResizeRow)
      document.removeEventListener('mouseup', stopResizeRow)
    };

  

    // const startResize = (index) => {
    //   resizing.value = true;
    //   rowIndex.value = index;
    //   initialY.value = event.clientY;
    //   initialHeight.value = props.data[index].height;

    //   document.addEventListener('mousemove', handleResize);
    //   document.addEventListener('mouseup', stopResize);
    // };

    // const handleResize = (event) => {
    //   if (resizing.value) {
    //     const newHeight = initialHeight.value + (event.clientY - initialY.value);
    //     if (newHeight > 0) {
    //       props.data[rowIndex.value].height = newHeight;
    //     }
    //   }
    // };

    // const stopResize = () => {
    //   resizing.value = false;
    //   document.removeEventListener('mousemove', handleResize);
    //   document.removeEventListener('mouseup', stopResize);
    // };
  // ======================

  

  const mouseMoveHandler = function (e: MouseEvent) {
    const dx = e.clientX - x.value
    newWidthCurrentColumn.value = widthCurrentColumn.value + dx
    colsWidth.value[currentColumnIdx.value].width = newWidthCurrentColumn.value + 'px'
    emits('set-columns-width', colsWidthPersentObj.value)

  }

  const mouseUpHandler = function () {
    isDraggableHeader.value = true
    document.removeEventListener('mousemove', mouseMoveHandler)
    document.removeEventListener('mouseup', mouseUpHandler)
  }

// start: работа с перемещением столбцов
const draggedIndex = ref<number | null>(null)

  const dragStartHeader = (idx: number) => {
    if(!isDraggableHeader.value) return
    draggedIndex.value = idx    
  }
const dragOverHeader = (idx: number) => {
  if(draggedIndex.value !== null){
    const columns = [...initialHeaders.value]
    const [draggedColumn] = columns.splice(draggedIndex.value, 1)
    columns.splice(idx, 0, draggedColumn)    
    emits('set-columns-order', columns)    
  }  
}
const dragDropHeader = () => {
  draggedIndex.value = null
}
// end: работа с перемещением столбцов

  watch(
    () => props.rowTable,
    (value) => {
      console.log('value', value);
      
      tableData.value = value
      rowsHeight.value = value.map((it: IObject) => it.height)
      nextTick(() => {
        heightTable.value = `${table?.value?.offsetHeight}px`
        setWidthCols()
      })     
    }
  )

  onMounted(() => {
    document.removeEventListener('mouseup', mouseUpHandler)
  })
</script>

<style scoped lang="sass">
  @import "scopeWorkStructureTable"
</style>