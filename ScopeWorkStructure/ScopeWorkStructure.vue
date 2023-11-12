<template>
  <div class="scope-work-structure">
    <div v-if="props.isUndoChanges" class="scope-work-structure__controls">
      <div class="scope-work-structure__controls-btns">
        <Button
          :id="`${pageId}_btn_cancel`"
          class="mr-10"
          view="smoke"
          size="small"
          :disabled="!props.historyExist"
          text="Отменить все изменения"
          @click="emits('clear-history')"
        />
        <button
          :id="`${pageId}_btn_prev`"
          :disabled="!props.historyHasPrevious"
          class="scope-work-structure__wrapper-btn"
          @click="emits('undo-operation')"
        >
          <IconSprite
            class="scope-work-structure__controls-btn"
            :class="{ disabled: !props.historyHasPrevious }"
            name="arrow_right"
          />
        </button>
        <button
          :id="`${pageId}_btn_next`"
          :disabled="!props.historyHasNext"
          class="scope-work-structure__wrapper-btn"
          @click="emits('redo-operation')"
        >
          <IconSprite
            class="scope-work-structure__controls-btn"
            :class="{ disabled: !props.historyHasNext }"
            name="arrow_left"
          />
        </button>
      </div>
    </div>
    <div
      v-if="props.setCheckbox.title.status"
      class="scope-work-structure__controls"
    >
      <span>{{ props.objectName }}</span>
    </div>
    <div
      class="scope-work-structure__table"
      :class="[
        { 'print-nine-elements': isNineElements },
        { 'print-five-elements': isFiveElements },
        { 'print-four-elements': isFourElements },
        { 'print-three-elements': isThreeElements },
      ]"
      :style="{ height: resizeHeightStructureTable - 140 + 'px' }"
    >
      <ScopeWorkStructureTable
        :table-id="`${pageId}_table`"
        :row-table="tableData"
        :headers="setHeaders"
        :expand="expand"
        :are-icons-blue="props.areIconsBlue"
        :are-icons-small="props.areIconsSmall"
        :resize-height-table="resizeHeightStructureTable"
        :initial-height="props.initialHeight ? props.initialHeight : '225px'"
        @delete-row="onDeleteRow"
        @delete-child-row="onDeleteChildRow"
        @change-row="onChangeRow"
        @set-columns-width="setColumnsWidth"
        @set-columns-order="setColumnsOrder"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { IHeaders } from '@/screens/Administration/interfaceAdministration'
  import { IObject } from '@/components/common/interfaceCommon'
  import { ICheckboxes } from '@/components/scopeWork/scopeWorkComponents/interfaceScopeWorkComponents'
  import { checkboxData } from '@/components/scopeWork/scopeWorkComponents/dataScopeWorkComponents'

  interface IProps {
    areIconsBlue?: boolean
    areIconsSmall?: boolean
    expand?: boolean
    buildElementGroupsData?: IObject
    // ESLINT-DISABLE: Старая реализация
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    resizeHeightStructureTable: any
    initialHeight?: string
    isUndoChanges?: boolean
    objectName?: string
    setCheckbox?: ICheckboxes
    isPrint?: boolean
    historyExist?: boolean
    historyHasNext?: boolean
    historyHasPrevious?: boolean
    isExtended?: boolean
  }

  const props = withDefaults(defineProps<IProps>(), {
    isUndoChanges: true,
    objectName: '',
    setCheckbox: () => checkboxData,
    isPrint: false,
    isExtended: false,
  })
  const emits = defineEmits<IEmits>()

  interface IEmits {
    (e: 'delete-element', row: IObject): void
    (e: 'deleteChildRow', row: IObject, parentCode: string): void
    (e: 'change-element'): void
    (e: 'clear-history'): void
    (e: 'undo-operation'): void
    (e: 'redo-operation'): void
    (e: 'set-columns-width', data: IObject): void
  }

  const pageId = 'scope_work_structure'

  const tableData = computed(() => props.buildElementGroupsData?.map((group: IObject) => {    
      // ESLINT-ERROR: Старая реализация
      // eslint-disable-next-line no-param-reassign
      group.actions = true
      // eslint-disable-next-line no-param-reassign
      group.height = 50
      
      return group
    }))

  const headers: IHeaders[] = [
    { name: '№ пп', code: 'row_number' },
    { name: 'Шифр', code: 'code' },
    { name: 'Наименование', code: 'name' },
    { name: 'Ед.изм', code: 'unit' },
    { name: 'Количество', code: 'count', type: 'input' },
    { name: '', code: 'actions' },
  ]

  const extendedHeaders = ref<IHeaders[]>([
    { name: '№ пп', code: 'row_number', width: '70px' },
    { name: '№ ЛСР', code: 'calc_number', type: 'input', width: '75px' },
    { name: 'Шифр', code: 'code', width: '90px' },
    { name: 'Наименование', code: 'name', width: '280px' },
    { name: 'Ед.изм', code: 'unit', width: '75px' },
    { name: 'Количество', code: 'count', type: 'input', width: '115px' },
    { name: 'Ссылка', code: 'references', type: 'input', width: '190px' },
    { name: 'Формула', code: 'formula', type: 'input', width: '95px' },
    { name: '', code: 'actions', width: '50px' },
  ])

  const settingsTable = reactive(
    {
      columns_width: {},
      columns_order: {}
    }
  )  
  
  const setHeaders = computed(() => {
    let header
    if (props.isPrint) {
      const fullCode = props.setCheckbox.code.status ? '' : 'code'
      const number = props.setCheckbox.number.status ? '' : 'row_number'
      header = headers.filter(
        (el) =>
          el.code !== fullCode && el.code !== number && el.code !== 'actions'
      )
    } else if (props.isExtended) {
      header = extendedHeaders.value
    } else {
      header = headers
    }
    return header
  })

  const isNineElements = computed(() => setHeaders.value.length === 9)
  const isFiveElements = computed(
    () => props.isPrint && setHeaders.value.length === 5
  )
  const isFourElements = computed(
    () => props.isPrint && setHeaders.value.length === 4
  )
  const isThreeElements = computed(
    () => props.isPrint && setHeaders.value.length === 3
  )

  const onDeleteRow = (row: IObject) => {
    emits('delete-element', row)
  }

  const onDeleteChildRow = (row: IObject, parentCode: string) => {
    emits('deleteChildRow', row, parentCode)
  }

  const onChangeRow = () => {
    emits('change-element')
  }

  const setColumnsWidth = (columnsWidth: IObject) => {
    settingsTable.columns_width = columnsWidth
    emits('set-columns-width', columnsWidth)
    
  }
  const setColumnsOrder = (data: IHeaders[]) => {
    extendedHeaders.value = data
    settingsTable.columns_order = data.reduce((acc, header: IObject, idx: number) => ({...acc, [header.code]: idx + 1}), {})

  }
</script>

<style scoped lang="sass">
  @import "ScopeWorkStructure"
</style>