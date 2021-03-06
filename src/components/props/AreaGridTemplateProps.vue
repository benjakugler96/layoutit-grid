<template>
  <div class="grid-settings-container">
    <div class="items columns">
      <div class="items-header">
        <h2>grid-template-columns</h2>
      </div>
      <div v-for="column in colsNumber" :key="column" class="area-size">
        <label>{{ column }}:</label>
        <div
          :data-col="column"
          class="input-container"
          @mouseover="currentHover = { on: 'track', grid, type: 'col', track: column }"
          @mouseleave="currentHover = null"
        >
          <input
            :style="{ visibility: unitHasValue(getColUnit(grid)) ? 'visible' : 'hidden' }"
            :value="getColValue(grid, column - 1)"
            :class="{
              active: isFocused('col', column),
              dragging: dragging && (dragging.colLine === column || dragging.colLine === column + 1),
            }"
            :type="getColUnit(grid, column - 1) === 'minmax' ? 'text' : 'number'"
            :aria-label="`column ${column} size`"
            min="0"
            step="0.5"
            @focus="currentFocus = { on: 'track', grid, type: 'col', track: column }"
            @blur="currentFocus = null"
            @input="onSizeValueInput('col', column - 1, $event.target.value)"
          />
          <UnitSelect
            :value="getColUnit(grid, column - 1)"
            type="grid"
            :focused="
              isFocused('col', column) || (dragging && (dragging.colLine === column || dragging.colLine === column + 1))
            "
            class="unit-select"
            :aria-label="`column ${column} unit`"
            @input="onColUnitInput($event.target.value, column - 1)"
          />
          <OptionsButton
            class="remove-button"
            :aria-label="`remove column ${column}`"
            :disabled="grid.col.sizes.length === 1"
            @click="removeCol(area, column - 1)"
            @mouseover.stop="currentHover = { on: 'track', grid, type: 'col', track: column, action: 'remove' }"
            @mouseleave="currentHover = null"
          >
            <IconRemove />
          </OptionsButton>
        </div>
      </div>
      <div class="area-size">
        <OptionsButton class="add-button" @click="addCol(grid, '1fr')">add +</OptionsButton>
      </div>
    </div>
    <div class="items rows">
      <div class="items-header">
        <h2>grid-template-rows</h2>
      </div>
      <div v-for="row in rowsNumber" :key="row" class="area-size">
        <label>{{ row }}:</label>

        <div
          :data-row="row"
          class="input-container"
          @mouseover="currentHover = { on: 'track', grid, type: 'row', track: row }"
          @mouseleave="currentHover = null"
        >
          <input
            :style="{ visibility: unitHasValue(getRowUnit(grid, row - 1)) ? 'visible' : 'hidden' }"
            :value="getRowValue(grid, row - 1)"
            :class="{
              active: isFocused('row', row),
              dragging: dragging && (dragging.rowLine === row || dragging.rowLine === row + 1),
            }"
            :type="getRowUnit(grid, row - 1) === 'minmax' ? 'text' : 'number'"
            :aria-label="`row ${row} size`"
            min="0"
            step="0.5"
            @focus="currentFocus = { on: 'track', grid, type: 'row', track: row }"
            @blur="currentFocus = null"
            @input="onSizeValueInput('row', row - 1, $event.target.value)"
          />
          <UnitSelect
            :value="getRowUnit(grid, row - 1)"
            type="grid"
            :focused="isFocused('row', row) || (dragging && (dragging.rowLine === row || dragging.rowLine === row + 1))"
            :aria-label="`row ${row} unit`"
            @input="onRowUnitInput($event.target.value, row - 1)"
          />
          <OptionsButton
            class="remove-button"
            :aria-label="`remove row ${row}`"
            :disabled="grid.row.sizes.length === 1"
            @click="removeRow(area, row - 1)"
            @mouseover.stop="currentHover = { on: 'track', grid, type: 'row', track: row, action: 'remove' }"
            @mouseleave="currentHover = null"
          >
            <IconRemove />
          </OptionsButton>
        </div>
      </div>
      <div class="area-size">
        <OptionsButton class="add-button" @click="addRow(grid, '1fr')">add +</OptionsButton>
      </div>
    </div>
  </div>
</template>

<script setup>
import IconRemove from '../icons/IconRemove.vue'
import IconAdd from '../icons/IconAdd.vue'
import UnitSelect from '../common/UnitSelect.vue'
import OptionsButton from '../basic/OptionsButton.vue'

import { defineProps, computed } from 'vue'

const props = defineProps({
  area: { type: Object, required: true },
})
const grid = computed(() => props.area.grid)

import {
  addCol,
  addRow,
  getRowValue,
  getRowUnit,
  setRowValue,
  removeRow,
  getColValue,
  getColUnit,
  setColValue,
  removeCol,
  parseUnit,
  parseValue,
  dragging,
  withChangedValue,
  withChangedUnit,
  currentFocus,
  currentHover,
  setRowValueUnit,
  setColValueUnit,
} from '../../store.js'

import { useGridDimensions } from '../../composables/area.js'
import { unitMeasureMap } from '../../utils.js'
import { debounce } from '../../composables'

const { colsNumber, rowsNumber } = useGridDimensions(grid)

function unitHasValue(unit) {
  return !(unit === 'initial' || unit === 'auto' || unit === 'min-content' || unit === 'max-content')
}

// TODO: compute new value using previous size

function defaultValueForUnit(unit) {
  return unitMeasureMap[unit] || 1
}

function onRowUnitInput(unit, row) {
  setRowValueUnit(props.area.grid, row, { value: defaultValueForUnit(unit), unit })
}

function onColUnitInput(unit, col) {
  setColValueUnit(props.area.grid, col, { value: defaultValueForUnit(unit), unit })
}

const onSizeValueInput = debounce((type, track, value) => {
  if (type === 'row') {
    setRowValue(grid.value, track, value)
  } else {
    setColValue(grid.value, track, value)
  }
})
const onAutoSizeValueInput = debounce((type, value) => {
  grid.value[type].auto = withChangedValue(grid.value[type].auto, value)
})

function isFocused(type, track) {
  const tf = currentFocus.value
  return tf && tf.on === 'track' && tf.grid === grid.value && tf.type === type && tf.track === track
}
</script>

<style scoped lang="scss">
h2 {
  display: block;
  flex: 1;
  max-width: max-content;
  color: rgb(156, 220, 254);
  font-size: 13px;
  text-shadow: none;
  font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  direction: ltr;
  text-align: left;
  white-space: pre;
  word-spacing: normal;
  margin: 0;
  font-weight: normal;
  padding-left: 10px;
  margin-bottom: 5px;
}

.items {
  background: rgb(35, 36, 31);
  padding: 10px 0 0;
  margin-bottom: 10px;
  &:last-of-type {
    margin-bottom: 0;
  }
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type='number'] {
  -moz-appearance: textfield;
}
.area-size {
  display: flex;
  align-items: center;
  padding-left: 10px;
  border-bottom: 1px solid rgba(68, 68, 68, 0.5);
  &:hover {
    background: #1c1d19;
    .input-container .unit-select,
    .input-container input {
      background: #1c1d19;
    }
  }
  &:last-of-type {
    border: 0;
  }
  label {
    display: block;
    flex: 1;
    max-width: max-content;
    color: rgb(215, 186, 125);
    font-size: 13px;
    text-shadow: none;
    font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
    direction: ltr;
    text-align: left;
    white-space: pre;
    word-spacing: normal;
    opacity: 0.8;
  }
  .input-container {
    display: grid;
    grid-template-columns: 1fr 1fr 30px;
    flex: 1;
    input {
      font-size: 14px;
      border: 0;
      width: 100%;
      background: rgb(35, 36, 31);
      color: rgb(206, 145, 120);
      height: 35px;
      text-align: right;
      flex: 1;
      font-size: 13px;
      text-shadow: none;
      font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
      direction: ltr;
      padding: 0;
      &:focus,
      &:hover {
        color: #eee;
      }
    }
    .unit-select {
      appearance: none;
      height: 35px;
      background: rgb(35, 36, 31);
      color: rgb(206, 145, 120);
      flex: 1;
      font-size: 13px;
      text-shadow: none;
      font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
      direction: ltr;
      cursor: text;
      text-align: center;
      padding-left: 3px;
      &:focus,
      &:hover {
        color: #eee;
      }
    }
  }
}

button,
.add-button {
  background: transparent;
  margin: 0;
  padding: 0;
  height: auto;
  font-size: 13px;
  text-shadow: none;
  font-family: Consolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
  direction: ltr;
  text-align: left;
  white-space: pre;
  word-spacing: normal;
  color: rgb(215, 186, 125);
  color: rgb(156, 220, 254);

  height: 35px;
  display: flex;
  align-items: center;
  &.remove-button[disabled] {
    opacity: 0.5;
  }
  &.remove-button {
    opacity: 0.8;
  }
  svg {
    width: 10px;
    stroke: rgb(206, 145, 120);
  }
  &:hover {
    background: transparent;
    opacity: 1;
    color: #eee;
    svg {
      stroke: #fff;
    }
  }
}

.add-button {
  height: 30px;
}
</style>
