<!--
 ADempiere-Vue (Frontend) for ADempiere ERP & CRM Smart Business Solution
 Copyright (C) 2017-Present E.R.P. Consultores y Asociados, C.A.
 Contributor(s): Elsio Sanchez elsiosanches@gmail.com www.erpya.com
 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See thelistPaymentsRefund() {
      const listServer = this.currentOrder.listPayments
      if (!this.isEmptyValue(listServer)) {
        return listServer.payments.filter(payment => payment.isRefund)
      }
      return []
    },
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <https:www.gnu.org/licenses/>.
-->
<template>
  <div>
    <el-form
      label-position="top"
      label-width="10px"
      @submit.native.prevent="notSubmitForm"
    >
      <el-row :gutter="12">
        <el-col
          v-for="field in fieldsList"
          :key="field.sequence"
          :span="6"
        >
          <field-definition
            :key="field.columnName"
            :metadata-field="field"
          />
        </el-col>
        <el-col :span="12">
          <el-form-item :label="$t('form.pos.collect.Currency')" class="from-field">
            <el-select v-model="referenceCurrency" :disabled="!isEmptyValue(referenceCurrency)">
              <el-option
                v-for="item in listCurrency"
                :key="item.id"
                :label="item.iso_code + '(' + item.currency_symbol + ')'"
                :value="item.iso_code"
              />
            </el-select>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
  </div>
</template>

<script>
import { formatPrice } from '@/utils/ADempiere/valueFormat.js'
import formMixin from '@theme/components/ADempiere/Form/formMixin'
import posMixin from '@theme/components/ADempiere/Form/VPOS/posMixin.js'
import fieldsListCash from './fieldsListCash.js'

export default {
  name: 'Cash',
  mixins: [
    formMixin,
    posMixin
  ],
  props: {
    change: {
      type: Number,
      default: 0
    },
    typeRefund: {
      type: Object,
      default: () => {
        return {}
      }
    },
    defaultCurrency: {
      type: Object,
      default: () => {
        return {}
      }
    },
    referenceCurrency: {
      type: String,
      default: ''
    },
    metadata: {
      type: Object,
      default: () => {
        return {
          uuid: 'Cash',
          containerUuid: 'Cash'
        }
      }
    }
  },
  data() {
    return {
      option: 1,
      typePay: 0,
      value: '',
      fieldsList: fieldsListCash,
      currentFieldCurrency: '',
      currentPaymentType: ''
    }
  },
  computed: {
    amountRefund() {
      return this.formatPrice(this.change / this.dayRate.divideRate, this.currencyReference.iso_code)
    },
    currencyReference() {
      const reference = this.isEmptyValue(this.typeRefund.refund_reference_currency) ? this.defaultCurrency.id : this.typeRefund.refund_reference_currency.id
      const currency = this.listCurrency.find(currency => {
        if (currency.id === reference) {
          return currency
        }
      })
      return currency
    },
    showDialogo() {
      return this.$store.state['pointOfSales/payments/index'].dialogoInvoce.show
    },
    caseOrder() {
      return this.$store.state['pointOfSales/payments/index'].dialogoInvoce.type
    },
    isoCode() {
      return this.$store.getters.posAttributes.currentPointOfSales.displayCurrency.iso_code
    },
    maximumDailyRefundAllowed() {
      return this.$store.getters.posAttributes.currentPointOfSales.maximumDailyRefundAllowed
    },
    maximumRefundAllowed() {
      return this.$store.getters.posAttributes.currentPointOfSales.maximumRefundAllowed
    },
    displayeCurrency() {
      const tenderType = this.$store.getters.getValueOfField({
        containerUuid: 'OverdrawnInvoice',
        columnName: 'TenderType'
      })
      if (tenderType === 'D') {
        return true
      }
      return false
    },
    primaryFieldsList() {
      return this.fieldsList.filter(field => field.sequence <= 2)
    },
    hiddenFieldsList() {
      return this.fieldsList.filter(field => field.sequence >= 3)
    },
    listCurrency() {
      return this.$store.getters.getCurrenciesList
    },
    emptyFieldGiftCard() {
      const empty = this.fieldsList.filter(field => {
        if (field.sequence < 3 && this.isEmptyValue(
          this.$store.getters.getValueOfField({
            containerUuid: 'OverdrawnInvoice',
            columnName: field.columnName
          })
        )) {
          return field
        }
      })
      return empty.map(empty => empty.name)
    },
    emptyMandatoryFields() {
      return this.$store.getters.getFieldsListEmptyMandatory({ containerUuid: 'OverdrawnInvoice', formatReturn: 'name' })
    },
    paymentTypeList() {
      return this.$store.getters.getPaymentTypeList
    },
    convertionsList() {
      return this.$store.state['pointOfSales/point/index'].conversionsList
    },
    currentConvertion() {
      if (this.isEmptyValue(this.currentPointOfSales.displayCurrency)) {
        return {}
      }
      const convert = this.convertionsList.find(convert => {
        if (!this.isEmptyValue(convert.currencyTo) && !this.isEmptyValue(this.currentPointOfSales.displayCurrency) && convert.currencyTo.id === this.currentPointOfSales.displayCurrency.id) {
          return convert
        }
      })
      if (convert) {
        return convert
      }
      return {}
    },
    dayRate() {
      const currency = this.listCurrency.find(currency => currency.iso_code === this.currencyReference.iso_code)
      const convert = this.convertionsList.find(convert => {
        if (!this.isEmptyValue(currency) && !this.isEmptyValue(convert.currencyTo) && currency.id === convert.currencyTo.id && this.currentPointOfSales.currentPriceList.currency.id !== currency.id) {
          return convert
        }
      })
      if (!this.isEmptyValue(convert)) {
        return convert
      }
      return {
        currencyTo: this.currentPointOfSales.currentPriceList.currency,
        divideRate: 1,
        iSOCode: this.currentPointOfSales.currentPriceList.currency.iSOCode
      }
    }
  },
  mounted() {
    this.$store.commit('updateValueOfField', {
      containerUuid: this.metadata.containerUuid,
      columnName: 'PayAmt',
      value: this.currentPointOfSales.currentOrder.refundAmount
    })
  },
  methods: {
    formatPrice,
    close() {
      this.$store.commit('dialogoInvoce', { show: false })
    },
    changeCurrency(value) {
      this.currentFieldCurrency = value
      const currency = this.listCurrency.find(currency => currency.iso_code === value)
      this.$store.dispatch('currencyRedund', currency)
    },
    changePaymentType(value) {
      this.$store.commit('currentTenderChange', value)
      this.currentPaymentType = value
      this.$store.commit('updateValueOfField', {
        containerUuid: 'OverdrawnInvoice',
        columnName: 'TenderType',
        value: value
      })
    }
  }
}
</script>

<style scoped>
  .el-image {
    display: inline-block;
    overflow: hidden;
  }
  .el-card__header {
    padding: 18px 20px;
    border-bottom: 1px solid #e6ebf5;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    background-color: rgb(245, 247, 250);
  }
  .el-card__body {
    padding-top: 0px !important;
    padding-right: 0px!important;
    padding-bottom: 20px;
    padding-left: 10px!important;
    height: 100%!important;
  }

  .bottom {
    margin-top: 0px!important;
    line-height: 1px;
  }

  .button {
    padding: 0;
    float: right;
  }

  .image {
    width: 100%;
    display: block;
    height: 9vh;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both
  }
  .el-header {
    background: 'white';
    color: #333;
    line-height: 10px;
  }
  .el-aside {
    color: #333;
  }
  .el-row {
    margin: 0px!important;
  }
</style>
