<template>
  <section>
    <div class="grid grid-cols-2 gap-4 mb-4">
      <div>
        <FormLabel class="block mb-2 text-sm" for="dataPeriod"
          name="Data Period" required />
        <Select v-model="dataPeriod" label-id="dataPeriod" label-class="text-sm"
          :options="dataPeridOptions" option-value="value" option-label="label"
          :option-disabled="option => option.disabled"
          fluid :pt="{ option: 'text-sm' }"
          @value-change="changePeriod"
        />
      </div>

      <div>
        <FormLabel class="block mb-2 text-sm" for="dataType"
          name="Data Type" required />
        <Select v-model="dataType" label-id="dataType" label-class="text-sm"
          :options="dataTypeOptions" option-value="value" option-label="label"
          :option-disabled="option => option.disabled"
          fluid :pt="{ option: 'text-sm' }"
          @value-change="changeType"
        />
      </div>

      <div v-show="isBiMonthly || isQuarterly">
        <FormLabel class="block mb-2 text-sm" for="startMonth"
          name="Start Month" required />
        <Select v-model="startMonth" label-id="startMonth" label-class="text-sm"
          :options="startMonthOptions" option-value="value" option-label="label"
          :option-disabled="option => option.disabled"
          fluid :pt="{ option: 'text-sm' }"
          @value-change="changeStartMonth"
        />
      </div>

      <Skeleton v-show="chartLoading && isAnnual && isConsumption"
        class="w-1/2" height="67px" :pt="{ root: 'bg-#f5f5f7' }" />
      <div v-show="!chartLoading && isAnnual && isConsumption" class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="annualUsageKwh" :name="annualLabel"
          required />
        <div class="flex gap-2 items-center">
          <InputNumber v-model="annualUsageKwh"
            input-id="annualUsageKwh" input-class="gs-inputnumber"
            fluid autocomplete="off" suffix=" kWh"
            :disabled="false"
            :min="0" :max="99999999"
            :min-fraction-digits="2" :max-fraction-digits="2"
            :locale="currencyInputLocale(prjCurrency)"
            @value-change="annualUsageKwhChange"
          />
        </div>
        <!-- <small class="mt-1 text-xs text-negative">{{ errors.customBillUsage }}</small> -->
      </div>

      <div v-show="isAnnual && isBill" class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="annualUsageBill" :name="annualLabel"
          required />
        <div class="flex gap-2 items-center">
          <InputNumber v-model="annualUsageBill"
            input-id="annualUsageBill" input-class="gs-inputnumber"
            fluid autocomplete="off"
            mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
            :disabled="false"
            :min="0" :max="99999999"
            :min-fraction-digits="2" :max-fraction-digits="2"
            @value-change="annualUsageBillChange"
          />
        </div>
        <!-- <small class="mt-1 text-xs text-negative">{{ errors.customBillUsage }}</small> -->
      </div>

      <div v-show="isCustom" class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="customUsageTime" :name="t('start_end_date')"
          required />
        <div class="flex gap-2 items-center">
          <DatePicker v-model="customUsageRangeTime" selection-mode="range"
            icon-display="input" input-class="datepicker-wrap"
            :pt="{ root: 'w-full!', inputIconContainer: 'left-2! w-6!' }"
            show-icon date-format="M. dd, yy"
            :max-date="setMaxDate()"
            :locale="useCurrentLocale()"
            @value-change="dateRangeChange">
            <template #inputicon>
              <Icon name="material-symbols:calendar-today" size="1rem" />
            </template>
          </DatePicker>
        </div>
        <small class="mt-1 text-xs text-negative">{{ dateErrorTips }}</small>
      </div>

      <div v-show="isCustom && isConsumption" class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="usageKwhForPeriod" :name="t('usage_for_period')"
          required />
        <div class="flex gap-2 items-center">
          <InputNumber v-model="usageKwhForPeriod"
            input-id="usageKwhForPeriod" input-class="gs-inputnumber"
            fluid autocomplete="off" suffix=" kWh"
            :disabled="false"
            :min="0" :max="99999999"
            :min-fraction-digits="2" :max-fraction-digits="2"
            :locale="useCurrentLocale()"
            @value-change="usagePeriodChange($event, 'usage')"
          />
        </div>
        <small class="mt-1 text-xs text-negative">{{ usageKwhForPeriodTips }}</small>
      </div>

      <div v-show="isCustom && isBill" class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="usageBillForPeriod" :name="t('usage_for_period')"
          required />
        <div class="flex gap-2 items-center">
          <InputNumber v-model="usageBillForPeriod"
            input-id="usageBillForPeriod" input-class="gs-inputnumber"
            fluid autocomplete="off"
            :disabled="false"
            :min="0" :max="99999999"
            :min-fraction-digits="2" :max-fraction-digits="2"
            mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
            @value-change="usagePeriodChange($event, 'bill')"
          />
        </div>
        <small class="mt-1 text-xs text-negative">{{ usageBillForPeriodTips }}</small>
      </div>

      <!-- <div v-show="isYearly && isConsumption" class="flex flex-col flex-1">
        <FormLabel class="block mb-1 text-sm" for="annualYearlyUsage" required
          :name="t('current_average_feed_in')" />
        <div class="flex gap-2 items-center">
          <InputNumber v-model.lazy="annualYearlyUsage"
            input-id="annualYearlyUsage" input-class="gs-inputnumber valid-item"
            fluid suffix=" kWh" autocomplete="off"
            :min="0" :max="10000000"
            :min-fraction-digits="2" :max-fraction-digits="2"
            :locale="useCurrentLocale()"
          />
          <span class="text-sm">/Year</span>
        </div>
        @value-change="onModelChange($event, 'estimateAnnualUsage')"
        <p class="mt-1 text-xs text-gray">
          <span class="text-negative">{{  }}</span>
          <small class="text-xs">{{ t('current_average_feed_in_tips') }}</small>
        </p>
      </div> -->

      <!-- <div v-show="isYearly && isBill" class="flex flex-col flex-1">
        <FormLabel class="block mb-1 text-sm" for="annualYearlyBill" required
          :name="t('current_average_feed_in')" />
        <div class="flex gap-2 items-center">
          <InputNumber v-model.lazy="annualYearlyBill"
            input-id="annualYearlyBill" input-class="gs-inputnumber valid-item"
            fluid autocomplete="off"
            :min="0" :max="10000000"
            :min-fraction-digits="2" :max-fraction-digits="2"
            mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
          />
          <span class="text-sm">/Year</span>
        </div>
        @value-change="onModelChange($event, 'estimateAnnualUsage')"
        <p class="mt-1 text-xs text-gray">
          <span class="text-negative">{{  }}</span>
          <small class="text-xs">{{ t('current_average_feed_in_tips') }}</small>
        </p>
      </div> -->

      <!-- Next version wait to do  -->
      <!-- <div v-show="isDaily && isConsumption" class="flex flex-col flex-1">
        <FormLabel class="block mb-1 text-sm" for="annualDailyUsage" required
          :name="t('current_average_feed_in')" />
        <div class="flex gap-2 items-center">
          <InputNumber v-model.lazy="annualDailyUsage"
            input-id="annualDailyUsage" input-class="gs-inputnumber valid-item"
            fluid suffix=" kWh" autocomplete="off"
            :min="0" :max="10000000"
            :min-fraction-digits="2" :max-fraction-digits="2"
            :locale="useCurrentLocale()"
          />
          <span class="text-sm">/Year</span>
        </div>
        <p class="mt-1 text-xs text-gray">
          <span class="text-negative">{{  }}</span>
          <small class="text-xs">{{ t('current_average_feed_in_tips') }}</small>
        </p>
      </div> -->

      <!-- Next version wait to do  -->
      <!-- <div v-show="isDaily && isBill" class="flex flex-col flex-1">
        <FormLabel class="block mb-1 text-sm" for="annualDailyBill" required
          :name="t('current_average_feed_in')" />
        <div class="flex gap-2 items-center">
          <InputNumber v-model.lazy="annualDailyBill"
            input-id="annualDailyBill" input-class="gs-inputnumber valid-item"
            fluid autocomplete="off"
            :min="0" :max="10000000"
            :min-fraction-digits="2" :max-fraction-digits="2"
            mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
          />
          <span class="text-sm">/Year</span>
        </div>
        @value-change="onModelChange($event, 'estimateAnnualUsage')"
        <p class="mt-1 text-xs text-gray">
          <span class="text-negative">{{  }}</span>
          <small class="text-xs">{{ t('current_average_feed_in_tips') }}</small>
        </p>
      </div> -->
    </div>

    <!-- 月度 -->
    <InputGroupFor12Months v-show="isMonthly && isConsumption" ref="For12MonthsKwhRef" type="usage"
      :max-fraction-digits="2"
      :use-placeholder="true" :chart-loading="chartLoading"
      suffix=" kWh"
      :details="monthlyUsage" :chart-data="usageChartData"
      :default-data="monthlyKwhData"
      @on-item-change="onMonthlyItemChange($event, 'usage')"
    />

    <InputGroupFor12Months v-show="isMonthly && isBill" ref="For12MonthsBillRef" type="bill"
      :max-fraction-digits="2"
      :use-placeholder="true" :chart-loading="chartLoading"
      mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
      :details="monthlyUsage" :chart-data="usageChartData"
      :default-data="monthBillData"
      @on-item-change="onMonthlyItemChange($event, 'bill')"
    />
    <!-- onMonthlyBillItemChange -->
    <!-- 双月 -->
    <InputGroupForBi2Months v-show="isBiMonthly && isConsumption" ref="ForBi2MonthsRef" type="usage"
      :default-data="biMonthKwhData" :start-month="startMonth"
      :details="biMonthlyUsage" :chart-data="usageChartData"
      :use-placeholder="true" :chart-loading="chartLoading"
      suffix=" kWh"
      @on-item-change="onBiMonthsItemChange($event, 'usage')"
    />

    <InputGroupForBi2Months v-show="isBiMonthly && isBill" ref="ForBi2MonthsBillRef" type="bill"
      :default-data="biMonthBillData" :start-month="startMonth"
      :details="biMonthlyUsage" :chart-data="usageChartData"
      :use-placeholder="true" :chart-loading="chartLoading"
      mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
      @on-item-change="onBiMonthsItemChange($event, 'bill')"
    />

    <!-- 季度 -->
    <InputGroupForQuarterly v-show="isQuarterly && isConsumption" ref="ForQuarterlyRef" type="usage"
      :default-data="quarterKwhData" :start-month="startMonth"
      :details="quarterlyUsage" :chart-data="usageChartData"
      :use-placeholder="true" suffix=" kWh" :chart-loading="chartLoading"
      @on-item-change="onQuarterItemChange($event, 'usage')"
    />

    <InputGroupForQuarterly v-show="isQuarterly && isBill" ref="ForQuarterlyBillRef" type="bill"
      :default-data="quarterBillData" :start-month="startMonth"
      :details="quarterlyUsage" :chart-data="usageChartData"
      :use-placeholder="true" :chart-loading="chartLoading"
      mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
      @on-item-change="onQuarterItemChange($event, 'bill')"
    />

    <div class="max-w-full mt-6">
      <div class="bg-white flex flex-col gap-6 relative border rounded-lg overflow-hidden">
        <div class="mb-2 pt-2 px-2 flex justify-between text-sm">
          <div>{{ t('estimated_average_monthly_usage') }}</div>
          <div class="text-grey-light">
            {{ t('estimate_annual_usage') }}:
            <span class="text-gsblack">{{ getNumberFormat(displayChartConsumption, ' kWh') }}</span>
          </div>
        </div>
        <EnergyChart ref="ChartRef" simple style="height: 160px"
          :loading="chartLoading" :chart-value="usageChartData" />
        <div v-if="false && showChartTips" class="absolute inset-0 flex items-center justify-center bg-white opacity-95">
          {{ showChartTips }}
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { promiseTimeout } from '@vueuse/core'
import InputGroupFor12Months from './InputGroupFor12Months.vue'
import InputGroupForBi2Months from './InputGroupForBi2Months.vue'
import InputGroupForQuarterly from './InputGroupForQuarterly.vue'
import FormLabel from '~/components/Forms/FormLabel.vue'
import EnergyChart from '~/components/Charts/EnergyChart.vue'
import { useCurrentLocale } from '~/composables/useCurrency'
import { calculateElectricityApi } from '~/apis/modules/project'

const monthBillArr = ['janBill', 'febBill', 'marBill', 'aprBill', 'mayBill', 'junBill', 'julBill', 'augBill', 'sepBill', 'octBill', 'novBill', 'decBill'] as const
const monthCstArr = ['janConsumption', 'febConsumption', 'marConsumption', 'aprConsumption', 'mayConsumption', 'junConsumption', 'julConsumption', 'augConsumption', 'sepConsumption', 'octConsumption', 'novConsumption', 'decConsumption'] as const
const biBillArr = ['biMonth1Bill', 'biMonth2Bill', 'biMonth3Bill', 'biMonth4Bill', 'biMonth5Bill', 'biMonth6Bill'] as const
const biCstArr = ['biMonth1Consumption', 'biMonth2Consumption', 'biMonth3Consumption', 'biMonth4Consumption', 'biMonth5Consumption', 'biMonth6Consumption'] as const
const quarBillArr = ['quarter1Bill', 'quarter2Bill', 'quarter3Bill', 'quarter4Bill'] as const
const quarCstArr = ['quarter1Consumption', 'quarter2Consumption', 'quarter3Consumption', 'quarter4Consumption'] as const

const { t } = useI18n()
const $dayjs = useDayjs()
const props = withDefaults(defineProps<{
  isAU: boolean
  editable?: boolean
  defaultValues: UsageDetails & { siteState: string, siteRegion: string }
}>(), {
  editable: true,
})
const emits = defineEmits(['group-item-change'])
const projectStore = useProjectStore()
const { chartLoading, prjCurrency, currencySymbol } = storeToRefs(projectStore)
const { updateChartLoading, getNumberFormat, getMoney } = projectStore

const dataPeridOptions = [
  { label: t('annual'), value: 'annualUsage' },
  { label: t('monthly'), value: 'monthlyUsage' },
  { label: t('bi_monthly'), value: 'biMonthlyUsage' },
  { label: t('quarterly'), value: 'quarterlyUsage' },
  // { label: 'Yearly', value: 'yearly' },
  // { label: 'Daily', value: 'daily' },
  { label: t('custom'), value: 'customPeriodUsage' },
]

const dataTypeOptions = computed(() => {
  const options = [
    { label: t('usage_kwh'), value: 'usage' },
    { label: `${t('bill')} (${currencySymbol.value})`, value: 'bill' },
  ]
  if (props.isAU && options[1]) options[1].label = `${t('bill')} (${currencySymbol.value}), incl.GST`
  return options
})

const dataPeriod = ref<UsageDataPeriod>('annualUsage')
const dataType = ref<UsageDataType>('usage')
const usageChartData = ref<number[]>([])
const showChartTips = computed(() => 'Please fill in the annual usage')

// const data_source = computed(() => dataPeriod.value + '_' + dataType.value)
// const annualYearlyUsage = ref<number | null>()
// const annualYearlyBill = ref<number | null>()
// const annualDailyUsage = ref<number | null>()
// const annualDailyBill = ref<number | null>()
// Annual Usage
const annualUsageKwh = ref<number>()
const annualUsageBill = ref<number>()
const displayChartConsumption = ref<number>()
const annualLabel = computed(() => {
  const usageArr = [t('annual'), ' kWh', ' (', t('default_for_location'), ': ', getNumberFormat(annualUsageKwh.value, ''), ' ', t('annual_consumption_unit'), ')']
  const billArr = [t('annual'), ' ', t('bill'), ' (', t('default_for_location'), ': ', getMoney(annualUsageBill.value), '/Year', ')']
  return isConsumption.value ? usageArr.join('') : billArr.join('')
})
const isAnnual = computed(() => dataPeriod.value === 'annualUsage')
const isMonthly = computed(() => dataPeriod.value === 'monthlyUsage')
const isBiMonthly = computed(() => dataPeriod.value === 'biMonthlyUsage')
const isQuarterly = computed(() => dataPeriod.value === 'quarterlyUsage')
const isCustom = computed(() => dataPeriod.value === 'customPeriodUsage')
const isConsumption = computed(() => dataType.value === 'usage')
const isBill = computed(() => dataType.value === 'bill')
// const isYearly = computed(() => dataPeriod.value === 'yearly')
// const isDaily = computed(() => dataPeriod.value === 'daily')
// const yearlyLabel = computed(() => {
//   if (isYearly.value && isConsumption.value) {
//     return `${t('current_average_feed_in')} kWh (${t('default_for_location')}: ${$number(estimateAnnualUsage.value, { precision: 2 })} kWh/year)`
//   }
//   return t('current_average_feed_in')
// })

const annualUsageKwhChange = useDebounceFn(async (evt: number) => {
  // console.log('🚀 ~ annualUsageKwhChange:', evt)
  try {
    annualUsageKwh.value = evt
    const ret = await calculateElectricity()
    await promiseTimeout(100)
    if (!ret) return
    await promiseTimeout(100)
    emits('group-item-change', { ...ret, ...baseParams.value })
  }
  catch (err) {
    console.error(err)
  }
}, 300)

const annualUsageBillChange = useDebounceFn(async (evt: number) => {
  // console.log('🚀 ~ annualUsageBillChange:', evt)
  try {
    annualUsageBill.value = evt
    const ret = await calculateElectricity()
    if (!ret) return
    await promiseTimeout(100)
    emits('group-item-change', { ...ret, ...baseParams.value })
  }
  catch (err) {
    console.error(err)
  }
}, 300)

async function changePeriod(evt: UsageDataPeriod) {
  try {
    dataPeriod.value = evt
    const ret = await calculateElectricity()
    if (!ret) return
    await promiseTimeout(100)
    emits('group-item-change', { ...ret, ...baseParams.value })
  }
  catch (err) {
    console.error(err)
  }
  // console.log('🚀 ~ changePeriod: 2222', evt, baseParams.value)
}

async function changeType(evt: UsageDataType) {
  try {
    dataType.value = evt
    const ret = await calculateElectricity()
    if (!ret) return
    await promiseTimeout(100)
    emits('group-item-change', { ...ret, ...baseParams.value })
  }
  catch (err) {
    console.error(err)
  }
}

async function changeStartMonth(evt: MonthValue) {
  try {
    startMonth.value = evt
    const ret = await calculateElectricity()
    if (!ret) return
    await promiseTimeout(100)
    emits('group-item-change', { ...ret, ...baseParams.value })
  }
  catch (err) {
    console.error(err)
  }
}

const ChartRef = useTemplateRef('ChartRef')

const initWc = watch(() => props.defaultValues, async (vals) => {
  await promiseTimeout(100)
  // dataPeriod.value = vals?.usageElectricDataSource
  // dataType.value = vals?.usageElectricDataType
  if (vals?.annualUsage?.annualConsumption) {
    annualUsageKwh.value = vals.annualUsage?.annualConsumption as number
    annualUsageBill.value = vals.annualUsage?.annualBill as number
  }

  if (isAnnual.value) displayChartConsumption.value = (vals.annualUsage?.annualConsumption || vals.estimateAnnualUsage) as number
  else if (isMonthly.value) displayChartConsumption.value = (vals.monthlyUsage?.annualConsumption || vals.estimateAnnualUsage) as number
  else if (isBiMonthly.value) displayChartConsumption.value = (vals.biMonthlyUsage?.annualConsumption || vals.estimateAnnualUsage) as number
  else if (isQuarterly.value) displayChartConsumption.value = (vals.quarterlyUsage?.annualConsumption || vals.estimateAnnualUsage) as number
  else if (isCustom.value) displayChartConsumption.value = (vals.customPeriodUsage?.annualConsumption || vals.estimateAnnualUsage) as number

  if (vals?.monthlyUsage) {
    monthlyUsage.value.consumptionIndex = vals?.monthlyUsage?.consumptionIndex as number[]
    monthlyUsage.value.billIndex = vals?.monthlyUsage?.billIndex as number[]
    monthCstArr.forEach((v, i: number) => {
      monthlyUsage.value[monthCstArr[i]] = vals?.monthlyUsage?.[v]
    })
    monthBillArr.forEach((v, i) => {
      monthlyUsage.value[monthBillArr[i]] = vals?.monthlyUsage?.[v]
    })
  }
  if (vals?.biMonthlyUsage) {
    biMonthlyUsage.value.consumptionIndex = vals?.biMonthlyUsage?.consumptionIndex as number[]
    biMonthlyUsage.value.billIndex = vals?.biMonthlyUsage?.billIndex as number[]
    biBillArr.forEach((v, i) => {
      biMonthlyUsage.value[biBillArr[i]] = vals?.biMonthlyUsage?.[v]
    })
    biCstArr.forEach((v, i) => {
      biMonthlyUsage.value[biCstArr[i]] = vals?.biMonthlyUsage?.[v]
    })
  }
  if (vals?.quarterlyUsage) {
    quarterlyUsage.value.consumptionIndex = vals?.quarterlyUsage?.consumptionIndex as number[]
    quarterlyUsage.value.billIndex = vals?.quarterlyUsage?.billIndex as number[]
    quarBillArr.forEach((v, i) => {
      quarterlyUsage.value[quarBillArr[i]] = vals?.quarterlyUsage?.[v]
    })
    quarCstArr.forEach((v, i) => {
      quarterlyUsage.value[quarCstArr[i]] = vals?.quarterlyUsage?.[v]
    })
  }
  if (vals.customPeriodUsage) {
    const cusUsage = vals?.customPeriodUsage
    customUsageStartTime.value = cusUsage?.customUsageStartTime
    customUsageEndTime.value = cusUsage?.customUsageEndTime
    usageKwhForPeriod.value = cusUsage?.customConsumption ?? null
    usageBillForPeriod.value = cusUsage?.customBill ?? null
    if (cusUsage?.customUsageStartTime && cusUsage?.customUsageEndTime) {
      customUsageRangeTime.value = [
        new Date($dayjs(cusUsage?.customUsageStartTime).format('YYYY-MM-DD')),
        new Date($dayjs(cusUsage?.customUsageEndTime).format('YYYY-MM-DD')),
      ]
    }
  }
  if (vals.monthElectricityUsage?.id) {
    const months = ['jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct', 'nov', 'dec'] as const
    const chartData = months.map((month: MonthKey) => (vals?.monthElectricityUsage as MonthElectricityUsage)?.[month])
    // console.log('接口请求后', res, chartData)
    usageChartData.value = chartData
    ChartRef.value?.updateChartShow()
    return
  }
  await calculateElectricity()
}, { immediate: true })

onUnmounted(() => {
  initWc()
})

const baseParams = computed(() => {
  const vals = props.defaultValues
  const params: any = {
    id: vals.id,
    countryCode: vals?.siteState,
    siteRegion: vals?.siteRegion,
    projectType: vals?.projectType,
    usageElectricDataSource: dataPeriod.value,
    usageElectricDataType: dataType.value,
    electricityPricePerKwh: vals?.electricityPricePerKwh,
    fixedChargeDay: vals?.fixedChargeDay,
    netMetering: vals?.netMetering,
    feedInTariff: vals?.feedInTariff,
    taxRate: $number(vals?.taxRate as number, { precision: 4 }).divide(100).value, // totalTax: vals?.totalTax,
    totalTax: $number(vals?.totalTax as number, { precision: 4 }).divide(100).value, // taxRate: vals?.taxRate,
    currentAverageFeedIn: vals?.currentAverageFeedIn,
  }
  return params
})

async function calculateElectricity() {
  try {
    updateChartLoading(true)
    const calcAnnualData = () => {
      const body = { ...baseParams.value, annualUsage: {} }
      if (isBill.value) body.annualUsage.annualBill = annualUsageBill.value
      if (isConsumption.value) body.annualUsage.annualConsumption = annualUsageKwh.value || props.defaultValues?.estimateAnnualUsage
      return body
    }
    const calcMonthlyData = () => {
      const body = { ...baseParams.value, monthlyUsage: {} }
      if (isBill.value) {
        body.monthlyUsage.billIndex = (monthlyUsage.value.billIndex || []) as number[]
        monthBillArr.forEach((key, idx) => {
          if (body.monthlyUsage.billIndex.includes(idx + 1)) body.monthlyUsage[key] = monthlyUsage.value[key]
          else body.monthlyUsage[key] = null
        })
        if (body.monthlyUsage.billIndex.length > 0) body.monthlyUsage.annualBill = monthlyUsage.value.annualBill
        else body.monthlyUsage.annualBill = annualUsageBill.value
      }
      if (isConsumption.value) {
        body.monthlyUsage.consumptionIndex = (monthlyUsage.value.consumptionIndex || []) as number[]
        if (body.monthlyUsage.consumptionIndex.length > 0) body.monthlyUsage.annualConsumption = monthlyUsage.value.annualConsumption
        else body.monthlyUsage.annualConsumption = annualUsageKwh.value || props.defaultValues?.estimateAnnualUsage
        monthCstArr.forEach((key, idx) => {
          if (body.monthlyUsage.consumptionIndex.includes(idx + 1)) body.monthlyUsage[key] = monthlyUsage.value[key]
          else body.monthlyUsage[key] = null
        })
      }
      return body
    }
    const calcBiMonthlyData = () => {
      const body = { ...baseParams.value, biMonthlyUsage: {} }
      body.biMonthlyUsage.startMonth = startMonth.value
      if (isBill.value) {
        body.biMonthlyUsage.billIndex = (biMonthlyUsage.value.billIndex || []) as number[]
        biBillArr.forEach((key, idx) => {
          if (body.biMonthlyUsage.billIndex.includes(idx + 1)) body.biMonthlyUsage[key] = biMonthlyUsage.value[key]
          else body.biMonthlyUsage[key] = null
        })
        if (body.biMonthlyUsage.billIndex.length > 0) body.biMonthlyUsage.annualBill = biMonthlyUsage.value.annualBill
        else body.biMonthlyUsage.annualBill = annualUsageBill.value
      }
      if (isConsumption.value) {
        body.biMonthlyUsage.consumptionIndex = (biMonthlyUsage.value.consumptionIndex || []) as number[]
        biCstArr.forEach((key, idx) => {
          if (body.biMonthlyUsage.consumptionIndex.includes(idx + 1)) body.biMonthlyUsage[key] = biMonthlyUsage.value[key]
          else body.biMonthlyUsage[key] = null
        })
        if (body.biMonthlyUsage.consumptionIndex.length > 0) body.biMonthlyUsage.annualConsumption = biMonthlyUsage.value.annualConsumption
        else body.biMonthlyUsage.annualConsumption = annualUsageKwh.value || props.defaultValues?.estimateAnnualUsage
      }
      return body
    }
    const calcQuarterlyData = () => {
      const body = { ...baseParams.value, quarterlyUsage: {} }
      body.quarterlyUsage.startMonth = startMonth.value
      if (isBill.value) {
        body.quarterlyUsage.billIndex = (quarterlyUsage.value.billIndex || []) as number[]
        quarBillArr.forEach((key, idx) => {
          if (body.quarterlyUsage.billIndex.includes(idx + 1)) body.quarterlyUsage[key] = quarterlyUsage.value[key]
          body.quarterlyUsage[key] = quarterlyUsage.value[key]
        })
        if (body.quarterlyUsage.billIndex.length > 0) body.quarterlyUsage.annualBill = quarterlyUsage.value.annualBill
        else body.quarterlyUsage.annualBill = annualUsageBill.value
      }
      if (isConsumption.value) {
        body.quarterlyUsage.consumptionIndex = (quarterlyUsage.value.consumptionIndex || []) as number[]
        quarCstArr.forEach((key, idx) => {
          if (body.quarterlyUsage.consumptionIndex.includes(idx + 1)) body.quarterlyUsage[key] = quarterlyUsage.value[key]
          body.quarterlyUsage[key] = quarterlyUsage.value[key]
        })
        if (body.quarterlyUsage.consumptionIndex.length > 0) body.quarterlyUsage.annualConsumption = quarterlyUsage.value.annualConsumption
        else body.quarterlyUsage.annualConsumption = annualUsageKwh.value || props.defaultValues?.estimateAnnualUsage
      }
      return body
    }
    const calcCustomData = () => {
      const body = { ...baseParams.value, customPeriodUsage: {} }
      if (customUsageRangeTime.value.some(v => [null, '', undefined].includes(v))) return false

      const startTime = `${$dayjs(customUsageRangeTime.value[0]).utc(true).format('YYYY-MM-DD 00:00:00')}`
      const endTime = `${$dayjs(customUsageRangeTime.value[1]).utc(true).format('YYYY-MM-DD 00:00:00')}`
      // customUsageStartTime.value = $dayjs(startTime).utc(true).valueOf()
      // customUsageEndTime.value = $dayjs(endTime).utc(true).valueOf()

      body.customPeriodUsage.customUsageStartTime = $dayjs(startTime).utc(true).valueOf()
      body.customPeriodUsage.customUsageEndTime = $dayjs(endTime).utc(true).valueOf()
      if (isBill.value) {
        if (!usageBillForPeriod.value) return false
        body.customPeriodUsage.customBill = usageBillForPeriod.value
      }
      if (isConsumption.value) {
        if (!usageKwhForPeriod.value) return false
        body.customPeriodUsage.customConsumption = usageKwhForPeriod.value
      }
      return body
    }

    let body
    if (isAnnual.value) body = calcAnnualData()
    else if (isMonthly.value) body = calcMonthlyData()
    else if (isBiMonthly.value) body = calcBiMonthlyData()
    else if (isQuarterly.value) body = calcQuarterlyData()
    else if (isCustom.value) body = calcCustomData()
    if (!body) return false
    const res: any = await calculateElectricityApi(body)
    // 图表数据公用
    const months = ['jan', 'feb', 'mar', 'apr', 'may', 'jun', 'jul', 'aug', 'sep', 'oct', 'nov', 'dec'] as const
    const chartData = months.map((monthKey: MonthKey) => (res!.monthElectricityUsage as MonthElectricityUsage)?.[monthKey])
    // console.log('接口请求后', res, chartData)
    usageChartData.value = chartData
    // 过滤选择类型的对应数据，传给上层本地缓存
    const extractFields = () => {
      let result: any = {}
      if (isAnnual.value) {
        const { annualUsage }: any = res
        displayChartConsumption.value = annualUsage?.annualConsumption
        annualUsageBill.value = annualUsage?.annualBill
        annualUsageKwh.value = annualUsage?.annualConsumption
        result = { annualUsage: null }
        result.annualUsage = annualUsage
      }
      if (isMonthly.value) {
        const { monthlyUsage: mRes }: any = res
        displayChartConsumption.value = mRes?.annualConsumption
        monthlyUsage.value = { ...mRes }
        result = { monthlyUsage: null }
        result.monthlyUsage = { ...mRes }
      }
      if (isBiMonthly.value) {
        const { biMonthlyUsage: biRes }: any = res
        displayChartConsumption.value = biRes?.annualConsumption
        biMonthlyUsage.value = { ...biRes }
        result = { biMonthlyUsage: null }
        result.biMonthlyUsage = { ...biRes }
      }
      if (isQuarterly.value) {
        const { quarterlyUsage: qRes }: any = res
        displayChartConsumption.value = qRes?.annualConsumption
        quarterlyUsage.value = { ...qRes }
        result = { quarterlyUsage: null }
        result.quarterlyUsage = { ...qRes }
      }
      // custom 的数据单独处理
      if (isCustom.value) {
        const { customPeriodUsage: customRes }: any = res
        if (customRes?.customUsageStartTime) customUsageStartTime.value = customRes.customUsageStartTime
        if (customRes?.customUsageEndTime) customUsageEndTime.value = customRes.customUsageEndTime
        // usageKwhForPeriod.value = customRes?.customConsumption
        // usageBillForPeriod.value = customRes?.customBill
        displayChartConsumption.value = customRes?.annualConsumption
        result = { customPeriodUsage: null }
        result.customPeriodUsage = res.customPeriodUsage
      }
      result.monthElectricityUsage = res?.monthElectricityUsage
      result.usageElectricDataSource = res?.usageElectricDataSource
      result.usageElectricDataType = res?.usageElectricDataType
      return result
    }

    const filterdData = extractFields()
    console.log('接口 calculateElectricityApi >> 过滤后', filterdData)
    return filterdData
  }
  catch (err) {
    console.error('calculateElectricity', err)
  }
  finally {
    updateChartLoading(false)
  }
}

// ------------------------ MonthlyInputGroup.vue 用电量 ------------------------
const monthlyUsage = ref({
  startMonth: 1,
  annualBill: null,
  billIndex: [] as number[],
  janBill: null,
  febBill: null,
  marBill: null,
  aprBill: null,
  mayBill: null,
  junBill: null,
  julBill: null,
  augBill: null,
  sepBill: null,
  octBill: null,
  novBill: null,
  decBill: null,
  annualConsumption: null,
  consumptionIndex: [] as number[],
  janConsumption: null,
  febConsumption: null,
  marConsumption: null,
  aprConsumption: null,
  mayConsumption: null,
  junConsumption: null,
  julConsumption: null,
  augConsumption: null,
  sepConsumption: null,
  octConsumption: null,
  novConsumption: null,
  decConsumption: null,
})

const monthlyKwhData = computed<{ editedNumArr: unknown[], chartData: unknown[] }>(() => ({
  editedNumArr: monthlyUsage.value?.consumptionIndex || [],
  chartData: usageChartData.value,
}))

// ------------------------ MonthlyInputGroup.vue 账单 ------------------------
const monthBillData = computed<{ editedNumArr: unknown[], chartData: unknown[] }>(() => ({
  editedNumArr: monthlyUsage.value?.billIndex || [],
  chartData: usageChartData.value,
}))

const For12MonthsKwhRef = useTemplateRef('For12MonthsKwhRef')
// const clearMonthlyKwh = () => For12MonthsKwhRef.value?.clearValues()
// const initMonthlyKwh = () => For12MonthsKwhRef.value?.initialize()
const For12MonthsBillRef = useTemplateRef('For12MonthsBillRef')

async function onMonthlyItemChange(evt, type: 'usage' | 'bill') {
  console.log('🚀 ~ onMonthlyItemChange:', evt, type)
  const { annual, inputIndexArr, userValues } = evt
  if (type === 'usage') {
    monthlyUsage.value.annualConsumption = annual
    monthlyUsage.value.consumptionIndex = inputIndexArr
    userValues.forEach((v, i) => {
      monthlyUsage.value[monthCstArr[i]] = v
    })
  }
  else if (type === 'bill') {
    monthlyUsage.value.annualBill = annual
    monthlyUsage.value.billIndex = inputIndexArr
    userValues.forEach((v, i) => {
      monthlyUsage.value[monthBillArr[i]] = v
    })
  }
  const ret = await calculateElectricity()
  emits('group-item-change', ret)
}

// ------------------------ InputGroupForBi2Months.vue Kwh ------------------------
/* global MonthLabel MonthValue */
const startMonthOptions: { label: MonthLabel, value: MonthValue }[] = [
  { label: 'Jan', value: 1 },
  { label: 'Feb', value: 2 },
  { label: 'Mar', value: 3 },
  { label: 'Apr', value: 4 },
  { label: 'May', value: 5 },
  { label: 'Jun', value: 6 },
  { label: 'Jul', value: 7 },
  { label: 'Aug', value: 8 },
  { label: 'Sep', value: 9 },
  { label: 'Oct', value: 10 },
  { label: 'Nov', value: 11 },
  { label: 'Dec', value: 12 },
]
const startMonth = ref<MonthValue>(1)
const biMonthlyUsage = ref({
  startMonth: 1,
  annualConsumption: null,
  consumptionIndex: [] as number[],
  biMonth1Consumption: null,
  biMonth2Consumption: null,
  biMonth3Consumption: null,
  biMonth4Consumption: null,
  biMonth5Consumption: null,
  biMonth6Consumption: null,

  annualBill: null,
  billIndex: [] as number[],
  biMonth1Bill: null,
  biMonth2Bill: null,
  biMonth3Bill: null,
  biMonth4Bill: null,
  biMonth5Bill: null,
  biMonth6Bill: null,
})
const biMonthKwhData = computed(() => ({
  editedNumArr: biMonthlyUsage.value.consumptionIndex || [],
  b1: biMonthlyUsage.value.biMonth1Consumption,
  b2: biMonthlyUsage.value.biMonth2Consumption,
  b3: biMonthlyUsage.value.biMonth3Consumption,
  b4: biMonthlyUsage.value.biMonth4Consumption,
  b5: biMonthlyUsage.value.biMonth5Consumption,
  b6: biMonthlyUsage.value.biMonth6Consumption,
  chartData: usageChartData.value,
}))

const ForBi2MonthsRef = useTemplateRef('ForBi2MonthsRef')

async function onBiMonthsItemChange(evt, type: 'bill' | 'usage') {
  console.log('onBiMonthsItemChange', evt, type)
  const { annual, inputIndexArr, userValues } = evt

  if (type === 'bill') {
    biMonthlyUsage.value.annualBill = annual
    biMonthlyUsage.value.billIndex = inputIndexArr
    userValues.forEach((v, i) => {
      biMonthlyUsage.value[biBillArr[i]] = v
    })
  }
  else if (type === 'usage') {
    biMonthlyUsage.value.annualConsumption = annual
    biMonthlyUsage.value.consumptionIndex = inputIndexArr
    userValues.forEach((v, i) => {
      biMonthlyUsage.value[biCstArr[i]] = v
    })
  }
  const ret = await calculateElectricity()
  emits('group-item-change', ret)
}

// ------------------------ InputGroupForBi2Months.vue Bill ------------------------
const biMonthBillData = computed(() => {
  const usage = biMonthlyUsage.value
  return {
    editedNumArr: usage?.billIndex || [],
    b1: usage?.biMonth1Bill,
    b2: usage?.biMonth2Bill,
    b3: usage?.biMonth3Bill,
    b4: usage?.biMonth4Bill,
    b5: usage?.biMonth5Bill,
    b6: usage?.biMonth6Bill,
    chartData: usageChartData.value,
  }
})

const ForBi2MonthsBillRef = useTemplateRef('ForBi2MonthsBillRef')
// ------------------------ InputGroupForQuarterly.vue kWh ------------------------
const quarterlyUsage = ref({
  startMonth: 1,
  consumptionIndex: [] as number[],
  annualConsumption: null,
  quarter1Consumption: null,
  quarter2Consumption: null,
  quarter3Consumption: null,
  quarter4Consumption: null,
  annualBill: null,
  quarter1Bill: null,
  quarter2Bill: null,
  quarter3Bill: null,
  quarter4Bill: null,
  billIndex: [] as number[],
})

const quarterKwhData = computed(() => {
  const usage = quarterlyUsage.value
  return {
    editedNumArr: usage?.consumptionIndex || [],
    q1: usage?.quarter1Consumption,
    q2: usage?.quarter2Consumption,
    q3: usage?.quarter3Consumption,
    q4: usage?.quarter4Consumption,
    chartData: usageChartData.value,
  }
})

const ForQuarterlyRef = useTemplateRef('ForQuarterlyRef')
async function onQuarterItemChange(evt, type: 'bill' | 'usage') {
  console.log('🚀 ~ onQuarterItemChange:', evt, type)

  const { annual, inputIndexArr, userValues } = evt

  if (type === 'bill') {
    quarterlyUsage.value.annualBill = annual
    quarterlyUsage.value.billIndex = inputIndexArr
    userValues.forEach((v, i) => {
      quarterlyUsage.value[quarBillArr[i]] = v
    })
  }
  else if (type === 'usage') {
    quarterlyUsage.value.annualConsumption = annual
    quarterlyUsage.value.consumptionIndex = inputIndexArr
    userValues.forEach((v, i) => {
      quarterlyUsage.value[quarCstArr[i]] = v
    })
  }
  const ret = await calculateElectricity()
  emits('group-item-change', ret)
}

// ------------------------ InputGroupForQuarterly.vue Bill ------------------------
const quarterBillData = computed(() => {
  const usage = quarterlyUsage.value
  return {
    editedNumArr: usage?.billIndex,
    q1: usage?.quarter1Bill,
    q2: usage?.quarter2Bill,
    q3: usage?.quarter3Bill,
    q4: usage?.quarter4Bill,
    chartData: usageChartData.value,
  }
})

const ForQuarterlyBillRef = useTemplateRef('ForQuarterlyBillRef')

// ------------------------ Custom Date and Period ------------------------
const customUsageRangeTime = ref<any[]>([])
const customUsageStartTime = ref()
const customUsageEndTime = ref()
const dateErrorTips = computed(() => {
  if (!customUsageRangeTime.value || !customUsageRangeTime.value.length) {
    return t('general_input_note', { field_name: t('start_end_date') })
  }
  else if (customUsageRangeTime.value.some(date => !date)) {
    return t('general_input_note', { field_name: t('start_end_date') })
  }
  return ''
})

const usageKwhForPeriod = ref<number | null>(null)
const usageKwhForPeriodTips = computed(() => (isConsumption.value && !usageKwhForPeriod.value) ? `${t('general_input_note', { field_name: t('usage_for_period') })}` : '')
const usageBillForPeriod = ref<number | null>(null)
const usageBillForPeriodTips = computed(() => isBill.value && !usageBillForPeriod.value ? `${t('general_input_note', { field_name: t('usage_for_period') })}` : '')

function setMaxDate() {
  const today = new Date()
  const month = today.getMonth()
  const year = today.getFullYear()
  // const prevMonth = (month === 0) ? 11 : month - 1  // const prevYear = (prevMonth === 11) ? year - 1 : year  // const nextYear = (nextMonth === 0) ? year + 1 : year
  const date = today.getDate()
  const maxDate = new Date()

  maxDate.setMonth(month)
  maxDate.setFullYear(year)
  maxDate.setDate(date)
  return maxDate
}

async function usagePeriodChange(event: number, type: 'bill' | 'usage') {
  // console.log('🚀 ~ usagePeriodChange:', event, type)
  if (type === 'bill') usageBillForPeriod.value = event
  else if (type === 'usage') usageKwhForPeriod.value = event
  try {
    await promiseTimeout(100)
    const ret = await calculateElectricity()
    if (!ret) return
    emits('group-item-change', {
      ...baseParams.value,
      ...ret,
      usageElectricDataSource: ret.usageElectricDataSource,
      usageElectricDataType: ret.usageElectricDataType,
    })
  }
  catch (err) {
    console.error('usagePeriodChange', err)
  }
}

async function dateRangeChange(evt) {
  console.log('🚀 ~ dateRangeChange ~ dateRangeChange:', evt)
  customUsageRangeTime.value = evt
  if (dateErrorTips.value) return
  const startTime = `${$dayjs(evt[0]).utc(true).format('YYYY-MM-DD 00:00:00')}`
  const endTime = `${$dayjs(evt[1]).utc(true).format('YYYY-MM-DD 00:00:00')}`
  customUsageStartTime.value = $dayjs(startTime).utc(true).valueOf()
  customUsageEndTime.value = $dayjs(endTime).utc(true).valueOf()

  try {
    const ret = await calculateElectricity()
    if (!ret) return false
    emits('group-item-change', {
      ...baseParams.value,
      ...ret,
      usageElectricDataSource: ret.usageElectricDataSource,
      usageElectricDataType: ret.usageElectricDataType,
    })
  }
  catch (err) {
    console.error(err)
  }
}

function handleValid() {
  return new Promise((resolve, reject) => {
    if (dataPeriod.value === 'annualUsage') {
      if (dataType.value === 'usage' && !annualUsageKwh.value) reject(t('general_input_note', { field_name: t('usage_for_period') }))
      if (dataType.value === 'bill' && !annualUsageBill.value) reject(t('general_input_note', { field_name: t('usage_for_period') }))
    }
    if (dataPeriod.value === 'customPeriodUsage') {
      if (dateErrorTips.value) reject(t('general_input_note', { field_name: t('start_end_date') }))
      if (customUsageRangeTime.value.some(v => [null, '', undefined].includes(v))) reject(t('general_input_note', { field_name: t('start_end_date') }))
      if (dataType.value === 'usage' && !usageKwhForPeriod.value) reject(t('general_input_note', { field_name: t('usage_for_period') }))
      if (dataType.value === 'bill' && !usageBillForPeriod.value) reject(t('general_input_note', { field_name: t('usage_for_period') }))
    }
    resolve(true)
  })
}

const updateDataPeriod = (period: UsageDataPeriod) => dataPeriod.value = period
const updateDataType = (type: UsageDataType) => dataType.value = type

defineExpose({
  handleValid,
  updateDataPeriod,
  updateDataType,
  calculateElectricity,
})
</script>
