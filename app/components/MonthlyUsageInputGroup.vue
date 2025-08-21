<template>
  <div class="flex flex-col lg:flex-row w-full">
    <div v-show="props.useAnnualTotal" class="inline-flex mb-2 md:mb-0 items-start">
      <div class="item min-w-20 self-center">
        <FormLabel for="Annual" :name="t('annual')" required />
        <InputNumber v-model="annualValue" class="min-w-23"
          :name="t('annual')"
          :locale="currencyInputLocale(prjCurrency)"
          :min-fraction-digits="0" :max-fraction-digits="props.maxFractionDigits"
          :disabled="true"
          :min="0" :max="9999999999"
          suffix=" kWh" />
      </div>
      <div class="item min-w-6 self-center">
        <label>&nbsp;</label>
        <span>=</span>
      </div>
    </div>

    <div class="grid gap-4 grid-cols-4 md:grid-cols-[_1fr_1fr_1fr_1fr_1fr_1fr] xl:grid-cols-[_1fr_1fr_1fr_1fr_1fr_1fr]">
      <div v-for="month in months" :key="month.key" class="item">
        <FormLabel :for="monthNames[month.key]" :name="monthNames[month.key]" required />
        <InputNumber v-model="monthValues[month.key].value" class="min-w-20"
          :name="monthNames[month.key]"
          :locale="currencyInputLocale(prjCurrency)"
          :placeholder="monthValues[month.key].placeholder"
          input-class="placeholder:text-gray"
          :min-fraction-digits="0" :max-fraction-digits="props.maxFractionDigits"
          :min="0" :max="9999999999"
          :disabled="isMonthDisabled(month.index)"
          suffix=" kWh"
          :invalid="useInvalid && isMonthInvalid(month.key)"
          @value-change="onItemChange($event, month.key)" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, watchEffect, onUnmounted } from 'vue'
import { promiseTimeout, useDebounceFn } from '@vueuse/core'
import InputNumber from 'primevue/inputnumber'
import { useI18n } from 'vue-i18n'
import FormLabel from '~/components/Forms/FormLabel.vue'
import { useCurrentLocale } from '~/composables/useCurrency'

type MonthKey = 'Jan' | 'Feb' | 'Mar' | 'Apr' | 'May' | 'Jun' | 'Jul' | 'Aug' | 'Sep' | 'Oct' | 'Nov' | 'Dec'
interface MonthConfig {
  key: MonthKey
  index: number
}
interface MonthValue {
  value: number | null
  placeholder: string | undefined
}

const { t } = useI18n()
const months: MonthConfig[] = [
  { key: 'Jan', index: 1 },
  { key: 'Feb', index: 2 },
  { key: 'Mar', index: 3 },
  { key: 'Apr', index: 4 },
  { key: 'May', index: 5 },
  { key: 'Jun', index: 6 },
  { key: 'Jul', index: 7 },
  { key: 'Aug', index: 8 },
  { key: 'Sep', index: 9 },
  { key: 'Oct', index: 10 },
  { key: 'Nov', index: 11 },
  { key: 'Dec', index: 12 },
]

const props = withDefaults(defineProps<{
  useAnnualTotal?: boolean
  maxFractionDigits?: number
  editableMonths?: number[]
  defaultData?: number[]
  formErrors?: any
  useInvalid?: boolean
  usePlaceholderValue?: boolean
  computeValues?: number[]
  autoValues?: number[]
}>(), {
  // 是否支持 Annual 总计
  useAnnualTotal: false,
  // AU 兼容用，是否支持清空并展示错误提示
  useInvalid: false,
  // 是否支持占位符值（后端返 当有placeholder时也参与计算）
  usePlaceholderValue: true,
  maxFractionDigits: 2,
  // 可编辑月份
  editableMonths: () => [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12],
})
const projectStore = useProjectStore()
const { prjCurrency } = storeToRefs(projectStore)
const emits = defineEmits(['on-item-change', 'on-total-change', 'on-item-blur'])

// 集中管理月份数据
const monthValues = ref<Record<MonthKey, MonthValue>>({
  Jan: { value: null, placeholder: undefined },
  Feb: { value: null, placeholder: undefined },
  Mar: { value: null, placeholder: undefined },
  Apr: { value: null, placeholder: undefined },
  May: { value: null, placeholder: undefined },
  Jun: { value: null, placeholder: undefined },
  Jul: { value: null, placeholder: undefined },
  Aug: { value: null, placeholder: undefined },
  Sep: { value: null, placeholder: undefined },
  Oct: { value: null, placeholder: undefined },
  Nov: { value: null, placeholder: undefined },
  Dec: { value: null, placeholder: undefined },
})

const annualValue = ref<number | null>(null)

// 计算月份名称
const monthNames = computed(() => {
  const keys: Record<MonthKey, string> = {
    Jan: 'january_short',
    Feb: 'february_short',
    Mar: 'march_short',
    Apr: 'april_short',
    May: 'may_short',
    Jun: 'june_short',
    Jul: 'july_short',
    Aug: 'august_short',
    Sep: 'september_short',
    Oct: 'october_short',
    Nov: 'november_short',
    Dec: 'december_short',
  }

  return Object.fromEntries(
    Object.entries(keys).map(([key, value]) => [
      key as MonthKey,
      t(value).replace('.', ''),
    ]),
  ) as Record<MonthKey, string>
})

// 月份禁用状态
const isMonthDisabled = (index: number) =>
  props.editableMonths && !props.editableMonths.includes(index)

// 月份验证状态
const isMonthInvalid = (key: MonthKey) =>
  monthValues.value[key].value == null

// 辅助函数: 自动填充单位
const fillUnit = (val?: number | string) => {
  if (val === undefined || val === null) return ''
  const num = typeof val === 'string' ? parseFloat(val) : val
  return !isNaN(num) ? `${num} kWh` : ''
}

// 辅助函数: 从占位符提取数值
const getPhValue = (str?: string) => {
  if (!str) return 0
  if (str.includes(' kWh')) {
    const num = str.split(' kWh')[0]
    return parseFloat(num) || 0
  }
  return 0
}

const autoWc = watch(() => props.autoValues, async (val) => {
  if (!Array.isArray(val)) return

  await promiseTimeout(200)

  months.forEach((month, index) => {
    const value = val[index]
    if (value !== undefined) {
      monthValues.value[month.key].value = value
      monthValues.value[month.key].placeholder = fillUnit(value)
    }
  })
}, { immediate: true, deep: true })

const computeWc = watch(() => props.computeValues, (val) => {
  if (!props.usePlaceholderValue || !Array.isArray(val)) return

  months.forEach((month, index) => {
    const num = val[index]
    if (num !== undefined) {
      monthValues.value[month.key].placeholder
        = $number(num, { precision: props.maxFractionDigits }).value.toString()
    }
  })
}, { immediate: true })

const computeTotal = () => {
  let total = 0

  months.forEach((month) => {
    const { value, placeholder } = monthValues.value[month.key]
    const numValue = props.usePlaceholderValue
      ? value ?? (placeholder ? getPhValue(placeholder) : 0)
      : value ?? 0

    total = $number(total, { precision: props.maxFractionDigits })
      .add(numValue)
      .value
  })

  annualValue.value = total
  return total
}

const onItemChange = useDebounceFn(async ($event: any, key: MonthKey) => {
  // 需求如此：若手动清空输入框，则使用占位符的值重新赋上
  if ($event == null) {
    const placeholder = monthValues.value[key].placeholder
    monthValues.value[key].value = placeholder ? getPhValue(placeholder) : null
    return
  }
  monthValues.value[key].value = $event
  const total = computeTotal()

  const data = months.reduce((acc, month) => {
    acc[month.key] = monthValues.value[month.key].value
    return acc
  }, {} as Record<MonthKey, number | null>)

  emits('on-item-change', {
    key,
    value: $event,
    data,
    annual: total,
  })
}, 666)

const clearValues = () => {
  months.forEach((month) => {
    monthValues.value[month.key].value = null
    monthValues.value[month.key].placeholder = undefined
  })
  annualValue.value = null
}

const getCurrentValues = () => {
  const pdValues: Record<MonthKey, string | undefined> = {}
  const inputValues: Record<MonthKey, number | null> = {}
  const chartData: number[] = []

  months.forEach((month) => {
    const key = month.key
    pdValues[key] = monthValues.value[key].placeholder
    inputValues[key] = monthValues.value[key].value

    chartData.push(inputValues[key] !== null ? Number(inputValues[key]) : Number(pdValues[key] || 0))
  })

  return { pdValues, inputValues, chartData }
}

const totalWatcher = watchEffect(() => {
  computeTotal()
})

onUnmounted(() => {
  autoWc()
  computeWc()
  totalWatcher()
})

defineExpose({
  getCurrentValues,
  clearValues,
  Annual: annualValue,
  ...months.reduce((acc, month) => {
    acc[month.key] = computed(() => monthValues.value[month.key].value)
    acc[`${month.key}Ph`] = computed(() => monthValues.value[month.key].placeholder)
    return acc
  }, {} as Record<string, any>),
})
</script>

<style scoped lang="scss">
.item {
  @apply flex flex-col text-sm;
  :deep(.form-label) {
    @apply text-typo-main mb-1;
  }
  :deep(.p-inputnumber-input) {
    @apply w-full h-9 text-sm;
  }
}
</style>
