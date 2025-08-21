<template>
  <SkeletonWrap :loading="chartLoading">
    <template #load>
      <Skeleton height="130px" :pt="{ root: '!bg-#f5f5f7' }" />
    </template>
    <div class="flex flex-col lg:flex-row w-full">
      <div class="inline-flex mb-2 items-start">
        <div class="item min-w-20">
          <FormLabel :for="`annual${type === 'bill' ? '_bill' : ''}`" name="Annual" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model.lazy="Annual" class="min-w-23"
              name="annual"
              :prefix="props.prefix" :suffix="props.suffix"
              :disabled="true"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="9999999999"
              :locale="currencyInputLocale(prjCurrency)"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model.lazy="Annual" class="min-w-23"
              name="annual_bill"
              :disabled="true"
              :min-fraction-digits="0" :max-fraction-digits="2"
              :min="0" :max="9999999999"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
            />
          </template>
        </div>
        <div class="item min-w-6">
          <label>&nbsp;</label>
          <span class="py-1 leading-8">=</span>
        </div>
      </div>
      <div class="grid gap-2 grid-cols-4 md:grid-cols-6">
        <div v-for="month in months" :key="props.type + '-' + month.key" class="item">
          <FormLabel :for="`${props.type === 'bill' ? 'bill_' : ''}${month.key}`" :name="toUpperFirstLetter(month.key)" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model.lazy="monthValues[month.key as MonthKey].value"
              :name="month.key"
              input-class="placeholder:text-gray" class="min-w-20"
              :placeholder="monthValues[month.key as MonthKey].placeholder"
              :prefix="props.prefix" :suffix="props.suffix"
              :locale="currencyInputLocale(prjCurrency)"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="99999999" :disabled="monthValues[month.key as MonthKey].disabled"
              :invalid="useInvalid && monthValues[month.key as MonthKey].invalid"
              @value-change="onItemChange($event, month.key as MonthKey)"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model.lazy="monthValues[month.key as MonthKey].value"
              :name="`bill_${month.key}`"
              input-class="placeholder:text-gray" class="min-w-20"
              :placeholder="monthValues[month.key as MonthKey].placeholder"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
              :min-fraction-digits="0" :max-fraction-digits="2"
              :min="0" :max="99999999" :disabled="monthValues[month.key as MonthKey].disabled"
              :invalid="useInvalid && monthValues[month.key as MonthKey].invalid"
              @value-change="onItemChange($event, month.key as MonthKey)"
            />
          </template>
        </div>
      </div>
    </div>
  </SkeletonWrap>
</template>

<script setup lang="ts">
import FormLabel from '~/components/Forms/FormLabel.vue'

type MonthKey = 'jan' | 'feb' | 'mar' | 'apr' | 'may' | 'jun' | 'jul' | 'aug' | 'sep' | 'oct' | 'nov' | 'dec'
type MonthNumber = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12

type CompProps = {
  maxFractionDigits?: number
  editableMonths?: number[]
  details: any
  chartData: number[]
  useInvalid?: boolean
  usePlaceholder?: boolean
  type: 'usage' | 'bill'
  chartLoading: boolean
  prefix?: string | undefined
  suffix?: string | undefined
}
const props = withDefaults(defineProps<CompProps>(), {
  useInvalid: false,
  usePlaceholder: false,
  maxFractionDigits: 2,
  editableMonths: () => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
})
const emits = defineEmits<{
  'on-item-change': [
    data: {
      annual: number | null
      key: MonthKey
      value: number | null
      inputValues: (number | null)[]
      pdValues: (string | undefined)[]
      chartData: (number | null)[]
    },
  ]
}>()
const projectStore = useProjectStore()
const { getNumberFormat, getMoney } = useProjectStore()
const { prjCurrency } = storeToRefs(projectStore)
const isBill = computed(() => props.type === 'bill')
const editedNumArr = computed(() => isBill.value ? props.details?.billIndex : props.details?.consumptionIndex)
const chartData = computed(() => props.chartData)
// const monthlyKwhData = computed<{ editedNumArr: unknown[], chartData: unknown[] }>(() => ({
//   editedNumArr: monthlyUsage.value?.consumptionIndex || [],
//   chartData: usageChartData.value,
// }))
// const isUsage = computed(() => props.type === 'usage')
const phMaps = computed(() => {
  const data = props?.details ?? {}
  return {
    jan: isBill.value ? data?.janBill : data?.janConsumption,
    feb: isBill.value ? data?.febBill : data?.febConsumption,
    mar: isBill.value ? data?.marBill : data?.marConsumption,
    apr: isBill.value ? data?.aprBill : data?.aprConsumption,
    may: isBill.value ? data?.mayBill : data?.mayConsumption,
    jun: isBill.value ? data?.junBill : data?.junConsumption,
    jul: isBill.value ? data?.julBill : data?.julConsumption,
    aug: isBill.value ? data?.augBill : data?.augConsumption,
    sep: isBill.value ? data?.sepBill : data?.sepConsumption,
    oct: isBill.value ? data?.octBill : data?.octConsumption,
    nov: isBill.value ? data?.novBill : data?.novConsumption,
    dec: isBill.value ? data?.decBill : data?.decConsumption,
  }
})

type MonthItemProps = { key: MonthKey, number: MonthNumber }
const months: MonthItemProps[] = [
  { key: 'jan', number: 1 },
  { key: 'feb', number: 2 },
  { key: 'mar', number: 3 },
  { key: 'apr', number: 4 },
  { key: 'may', number: 5 },
  { key: 'jun', number: 6 },
  { key: 'jul', number: 7 },
  { key: 'aug', number: 8 },
  { key: 'sep', number: 9 },
  { key: 'oct', number: 10 },
  { key: 'nov', number: 11 },
  { key: 'dec', number: 12 },
]

function initialize() {
  const chartsValues = Object.values(phMaps.value)
  if (!props.usePlaceholder) {
    months.forEach((month, index) => {
      monthValues.value[month.key as MonthKey].value = chartsValues[index] as number
    })
  }
  else {
    months.forEach((month, index) => {
      monthValues.value[month.key as MonthKey].phValue = chartsValues[index]
      monthValues.value[month.key as MonthKey].placeholder
        = `${props.type === 'usage' ? getNumberFormat(chartsValues[index], ' kWh') : getMoney(chartsValues[index])}`
    })
    if (editedNumArr.value.length > 0) {
      editedNumArr.value.forEach((mNum: MonthNumber) => {
        const monthKey = months.find(v => mNum === v.number)!.key
        monthValues.value[monthKey].value = chartsValues[mNum - 1] as number
      })
    }
  }
}

watch([() => props.details, () => props.chartData], (vals) => {
  initialize()
})

function toUpperFirstLetter(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1)
}

type MonthProps = {
  value: null | number
  placeholder: undefined | string
  phValue: number // 为兼容国际化数字计算
  disabled: ComputedRef<boolean>
  invalid: boolean
}
const monthValues = ref<Record<MonthKey, MonthProps>>({
  jan: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(1)), invalid: false },
  feb: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(2)), invalid: false },
  mar: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(3)), invalid: false },
  apr: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(4)), invalid: false },
  may: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(5)), invalid: false },
  jun: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(6)), invalid: false },
  jul: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(7)), invalid: false },
  aug: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(8)), invalid: false },
  sep: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(9)), invalid: false },
  oct: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(10)), invalid: false },
  nov: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(11)), invalid: false },
  dec: { value: null, placeholder: undefined, phValue: 0, disabled: computed(() => !props.editableMonths.includes(12)), invalid: false },
})

const Annual = defineModel<number | null>('Annual', { default: null })

async function onItemChange($event: any, key: MonthKey) {
  // console.log('🚀 ~ onItemChange:', $event, key)
  if (key) monthValues.value[key].value = $event
  await nextTick()
  emits('on-item-change', {
    annual: Annual.value,
    key,
    value: $event,
    ...getCurrentValues(),
  })
}

function computeTotal(): number {
  let total: number = 0
  if (!props.usePlaceholder) {
    total = months.reduce((acc, month) => {
      return $number(acc).add(monthValues.value[month.key as MonthKey].value || 0).value
    }, 0)
  }
  else {
    total = months.reduce((acc, month) => {
      const waitAddNum = monthValues.value[month.key as MonthKey].value == null
        ? monthValues.value[month.key as MonthKey].phValue
        : monthValues.value[month.key as MonthKey].value
      return $number(acc).add(waitAddNum as number).value
    }, 0)
  }
  Annual.value = total
  return total
}

function clearValues() {
  months.forEach((month) => {
    monthValues.value[month.key as MonthKey].value = null
    monthValues.value[month.key as MonthKey].placeholder = undefined
  })
  Annual.value = null
}

function getCurrentValues() {
  const inputValues = months.map(month => monthValues.value[month.key as MonthKey].value) ?? []
  const inputIndexArr = getNonNullIndices(inputValues)
  const pdValues = months.map(month => monthValues.value[month.key as MonthKey].placeholder) ?? []
  const userValues = mergeUsageArrays(inputValues, pdValues.map(v => convertToNumber(v)))
  // const chartData = months.map(month => monthValues.value[month.key as MonthKey].value ?? convertToNumber(monthValues.value[month.key as MonthKey].placeholder)) ?? []
  return {
    chartData: chartData.value ?? [],
    pdValues,
    inputValues,
    userValues,
    inputIndexArr,
  }
}

const watcher = watchEffect(() => {
  computeTotal()
})

onMounted(() => initialize)
onUnmounted(() => {
  watcher()
})

defineExpose({
  initialize,
  getCurrentValues,
  clearValues,
})
</script>

<style scoped lang="scss">
.item {
  @apply flex flex-col items-center justify-center text-sm;
  :deep(.form-label) {
    @apply text-typo-main mb-1;
  }
  :deep(.p-inputnumber-input) {
    @apply w-full p-1 h-9 text-center text-sm;
  }
}
</style>
