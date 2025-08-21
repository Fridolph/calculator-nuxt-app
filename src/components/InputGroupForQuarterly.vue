<template>
  <SkeletonWrap :loading="chartLoading">
    <template #load>
      <Skeleton height="60px" :pt="{ root: '!bg-#f5f5f7' }" />
    </template>
    <div class="flex flex-col lg:flex-row w-full">
      <div class="inline-flex mb-2 md:mb-0 items-start">
        <div class="item min-w-48">
          <FormLabel for="Annual" name="Annual" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model="annual" name="Annual" class="min-w-48"
              :disabled="true"
              :prefix="props.prefix" :suffix="props.suffix"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="99999999"
              :locale="currencyInputLocale(prjCurrency)"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model="annual" name="Annual" class="min-w-48"
              :disabled="true"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="99999999"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
            />
          </template>
        </div>
        <div class="item min-w-6 self-center">
          <label>&nbsp;</label>
          <span>=</span>
        </div>
      </div>
      <div class="flex-1 grid gap-4 grid-cols-2 md:grid-cols-4">
        <div v-for="quarter in quarters" :key="quarter.key" class="item">
          <FormLabel :for="props.type + quarter.key" :name="quarterValues[quarter.key].name" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model.lazy="quarterValues[quarter.key].value" :input-id="props.type + quarter.key" fluid
              :placeholder="quarterValues[quarter.key].placeholder" input-class="placeholder:text-gray"
              :prefix="props.prefix" :suffix="props.suffix"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="9999999" :disabled="quarterValues[quarter.key].disabled"
              :invalid="useInvalid && quarterValues[quarter.key].invalid"
              :locale="currencyInputLocale(prjCurrency)"
              @value-change="onItemChange($event, 'q1')"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model.lazy="quarterValues[quarter.key].value" :input-id="props.type + quarter.key" fluid
              :placeholder="quarterValues[quarter.key].placeholder" input-class="placeholder:text-gray"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="9999999" :disabled="quarterValues[quarter.key].disabled"
              :invalid="useInvalid && quarterValues[quarter.key].invalid"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
              @value-change="onItemChange($event, 'q1')"
            />
          </template>
        </div>
      </div>
    </div>
  </SkeletonWrap>
</template>

<script setup lang="ts">
import FormLabel from '~/components/Forms/FormLabel.vue'

type StartMonth = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
type QuarterKey = 'q1' | 'q2' | 'q3' | 'q4'
type LabelValue = 1 | 2 | 3 | 4
type CompProps = {
  prefix?: string
  suffix?: string
  maxFractionDigits?: number
  editableQuarter?: number[]
  chartLoading: boolean
  defaultData: {
    q1: number | null
    q2: number | null
    q3: number | null
    q4: number | null
    editedNumArr: LabelValue[]
    chartData: number[]
  }
  startMonth?: StartMonth
  useInvalid?: boolean
  usePlaceholder?: boolean
  type?: 'usage' | 'bill'
}
const props = withDefaults(defineProps<CompProps>(), {
  prefix: '',
  suffix: '',
  useInvalid: false,
  usePlaceholder: false,
  maxFractionDigits: 2,
  startMonth: 1,
  type: 'usage',
  editableQuarter: () => [1, 2, 3, 4],
})
const emits = defineEmits(['on-item-change'])
const projectStore = useProjectStore()
const { getNumberFormat, getMoney } = useProjectStore()
const { prjCurrency } = storeToRefs(projectStore)

const annual = defineModel<number | null>('annual', { default: null })
const quarters = ref<{ key: QuarterKey, number: LabelValue }[]>([
  { key: 'q1', number: 1 },
  { key: 'q2', number: 2 },
  { key: 'q3', number: 3 },
  { key: 'q4', number: 4 },
])
const quarterValues = ref<Record<QuarterKey, { value: null | number, phValue: number, placeholder: string | undefined, name: ComputedRef<string>, disabled: ComputedRef<boolean>, invalid: boolean }>>({
  q1: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth)), disabled: computed(() => !props.editableQuarter?.includes(1)), invalid: false },
  q2: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 3)), disabled: computed(() => !props.editableQuarter?.includes(2)), invalid: false },
  q3: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 6)), disabled: computed(() => !props.editableQuarter?.includes(3)), invalid: false },
  q4: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 9)), disabled: computed(() => !props.editableQuarter?.includes(4)), invalid: false },
})

function generateLabel(startMonth: number) {
  const monthNames = [
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec',
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec',
  ]
  return `${monthNames[startMonth - 1]}-${monthNames[startMonth + 1]}`
}

async function onItemChange($event: any, key: QuarterKey) {
  // emits('on-item-blur', { key, value: data[key], data, edited_month })
  await nextTick()
  emits('on-item-change', {
    annual: annual.value,
    key,
    value: $event,
    ...getCurrentValues(),
  })
}

function computeTotal(): number {
  let total: number = 0
  if (!props.usePlaceholder) {
    total = quarters.value.reduce((acc, quarter) => {
      return $number(acc).add(quarterValues.value[quarter.key].value || 0).value
    }, 0)
  }
  else {
    total = quarters.value.reduce((acc, quarter) => {
      const waitAddNum = quarterValues.value[quarter.key].value == null
        // ? convertToNumber(quarterValues.value[quarter.key].placeholder as string)
        ? quarterValues.value[quarter.key].phValue
        : quarterValues.value[quarter.key].value
      return $number(acc).add(waitAddNum as number).value
    }, 0)
  }
  annual.value = total
  return total
}

function clearValues() {
  quarters.value.forEach((qItem) => {
    quarterValues.value[qItem.key].value = null
    quarterValues.value[qItem.key].placeholder = undefined
  })
}

function getCurrentValues() {
  const inputValues = quarters.value.map(quarter => quarterValues.value[quarter.key as QuarterKey].value) ?? []
  const inputIndexArr = getNonNullIndices(inputValues)
  const pdValues = quarters.value.map(quarter => quarterValues.value[quarter.key as QuarterKey].placeholder) ?? []
  const userValues = mergeUsageArrays(inputValues, pdValues.map(v => convertToNumber(v)))
  return {
    chartData: props.defaultData.chartData ?? [],
    pdValues,
    inputValues,
    userValues,
    inputIndexArr,
  }
}

function initialize() {
  const defData = props.defaultData
  if (!props.usePlaceholder) {
    quarters.value.forEach((qItem) => {
      quarterValues.value[qItem.key].value = defData?.[qItem.key]
    })
  }
  else {
    quarters.value.forEach((qItem) => {
      quarterValues.value[qItem.key].phValue = defData?.[qItem.key] as number
      quarterValues.value[qItem.key].placeholder
        = `${props.type === 'usage' ? getNumberFormat(defData?.[qItem.key] as number, ' kWh') : getMoney(defData?.[qItem.key] as number)}`
        // = `${props.prefix ?? ''}${defData?.[qItem.key]?.toString()}${props.suffix ?? ''}`
    })

    if (defData?.editedNumArr.length > 0) {
      defData?.editedNumArr.forEach((qNum) => {
        const quarterKey = quarters.value.find(v => qNum === v.number)!.key
        quarterValues.value[quarterKey].value = defData?.[quarterKey]
      })
    }
  }
}

const watcher = watchEffect(() => {
  computeTotal()
})

watch(() => props.defaultData, (vals) => {
  initialize()
}, { deep: true })

onMounted(() => initialize)
onUnmounted(() => {
  watcher()
})

defineExpose({
  initialize,
  computeTotal,
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
