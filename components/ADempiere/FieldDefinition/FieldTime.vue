<!--
 ADempiere-Vue (Frontend) for ADempiere ERP & CRM Smart Business Solution
 Copyright (C) 2017-Present E.R.P. Consultores y Asociados, C.A.
 Contributor(s): Yamel Senih ysenih@erpya.com www.erpya.com
 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <https:www.gnu.org/licenses/>.
-->
<template>
  <el-time-picker
    v-model="value"
    v-bind="commonsProperties"
    :picker-options="{
      minTime: minValue,
      maxTime: maxValue
    }"
    :is-range="isPickerRange"
    range-separator="-"
    @change="preHandleChange"
    @blur="focusLost"
    @focus="focusGained"
    @keydown.native="keyPressed"
    @keyup.native="keyReleased"
  />
</template>

<script>
// components and mixins
import fieldMixin from '@theme/components/ADempiere/FieldDefinition/mixin/mixinField.js'

export default {
  name: 'FieldTime',

  mixins: [
    fieldMixin
  ],

  computed: {
    isPickerRange() {
      if (this.metadata.isRange && !this.metadata.inTable) {
        return true
      }
      return false
    },
    maxValue() {
      if (!this.isEmptyValue(this.metadata.valueMax)) {
        return Number(this.metadata.valueMax)
      }
      return Infinity
    },
    minValue() {
      if (!this.isEmptyValue(this.metadata.valueMin)) {
        return Number(this.metadata.valueMin)
      }
      return -Infinity
    },
    cssClassCustomField() {
      return ' custom-field-time '
    }
  },

  methods: {
    parseValue(value) {
      if (typeof value === 'number') {
        value = new Date(value).toUTCString()
      }
      if (this.isEmptyValue(value)) {
        value = undefined
      }
      return value
    }
  }
}
</script>

<style scoped>
  .custom-field-time {
    width: 100% !important;
  }
</style>
