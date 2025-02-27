<!--
 ADempiere-Vue (Frontend) for ADempiere ERP & CRM Smart Business Solution
 Copyright (C) 2018-Present E.R.P. Consultores y Asociados, C.A.
 Contributor(s): Yamel Senih ysenih@erpya.com www.erpya.com
 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program. If not, see <https:www.gnu.org/licenses/>.
-->

<template>
  <el-autocomplete
    v-model="displayedValue"
    v-bind="commonsProperties"
    :fetch-suggestions="localSearch"
    :trigger-on-focus="true"
    clearable
    value-key="name"
    style="width: 100%;"
    popper-class="custom-field-bpartner-info"
    @focus="isFocus = true"
    @blur="isFocus = false"
    @select="handleSelect"
  >
    <template
      slot="prefix"
    >
      <i
        :class="metadata.icon"
      />
    </template>
    <template slot="suffix">
      <i
        class="el-icon-arrow-down el-input__icon"
      />
    </template>
    <template slot-scope="props">
      <div class="header">
        <b>{{ props.item.name }} </b>
      </div>
      <span class="info">
        {{ props.item.value }}
      </span>
    </template>
  </el-autocomplete>
</template>

<script>
// Components and Mixins
import fieldMixin from '@theme/components/ADempiere/FieldDefinition/mixin/mixinField.js'
import fieldWithDisplayColumn from '@theme/components/ADempiere/FieldDefinition/mixin/mixinWithDisplayColumn.js'
import selectMixin from '@theme/components/ADempiere/FieldDefinition/mixin/mixinFieldSelect.js'

export default {
  name: 'FieldAutocomplete',

  mixins: [
    fieldMixin,
    fieldWithDisplayColumn,
    selectMixin
  ],

  data() {
    return {
      recordsBusinessPartners: [],
      controlDisplayed: this.displayedValue,
      isFocus: false
    }
  },

  computed: {
    cssClassCustomField() {
      let styleClass = ' custom-field-select'
      if (this.isSelectMultiple) {
        styleClass += ' custom-field-select-multiple'
      }
      return styleClass
    },
    placeholder() {
      if (this.isFocus) {
        return this.displayedValue
      }
      return this.$t('quickAccess.searchWithEnter')
    },

    value: {
      get() {
        const value = this.$store.getters.getValueOfFieldOnContainer({
          parentUuid: this.metadata.parentUuid,
          containerUuid: this.metadata.containerUuid,
          columnName: this.metadata.columnName
        })
        if (this.isEmptyValue(value)) {
          /* eslint-disable */
          this.displayedValue = undefined
          /* eslint-disable */
          return value
        }

        let label = this.findLabel(value)
        if (!label) {
          label = this.displayedValue
          /* eslint-disable */
          this.optionsList.push({
            // TODO: Add uuid
            id: value,
            label
          })
          /* eslint-disable */
        }
        return value
      },
      set(value) {
        this.$store.commit('updateValueOfField', {
          parentUuid: this.metadata.parentUuid,
          containerUuid: this.metadata.containerUuid,
          columnName: this.metadata.columnName,
          value
        })
      }
    }
  },

  methods: {
    setNewDisplayedValue() {
      this.isFocus = true
      const displayValue = this.displayedValue
      if (this.controlDisplayed !== displayValue) {
        this.controlDisplayed = displayValue
      }
    },
    localSearch(stringToMatch, callBack) {
      const localListSearch = this.metadata.loadAll
      const results = stringToMatch ? localListSearch.filter(this.createFilter(stringToMatch)) : localListSearch
      // call callback function to return suggestions
      if (this.isEmptyValue(results) && stringToMatch.length > 3) {
        clearTimeout(this.timeOut)
        this.timeOut = setTimeout(() => {
          this.remoteSearch(stringToMatch)
            .then(remoteResponse => {
              callBack(remoteResponse)
            })
        }, 3000)
        return
      }
      callBack(results)
    },
    createFilter(stringToMatch) {
      return (find) => {
        return (find.name.toLowerCase().indexOf(stringToMatch.toLowerCase()) === 0)
      }
    },
    remoteSearch(searchValue) {
      return new Promise(resolve => {
        const message = {
          message: this.$t('notifications.searchWithOutRecords'),
          type: 'info',
          showClose: true
        }

        this.$store.dispatch(this.metadata.searchServer, {
          pageNumber: 1,
          searchValue
        })
          .then((response) => {
            const recordsList = this.metadata.loadAll
            if (this.isEmptyValue(recordsList)) {
              this.$message(message)
            }
            return response
          })
          .catch(error => {
            console.warn(error.message)

            this.$message(message)
            return []
          })
      })
    },
    handleSelect(item) {
      this.value = item.id
      this.displayValue = item.name
      this.uuidValue = item.uuid
    }
  }
}
</script>
