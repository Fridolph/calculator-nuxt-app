<template>
  <SkeletonWrap :loading="chartLoading">
    <template #load>
      <Skeleton height="60px" :pt="{ root: '!bg-#f5f5f7' }" />
    </template>
    <div class="flex flex-col lg:flex-row w-full">
      <div class="inline-flex mb-2 md:mb-0 items-start">
        <div class="item min-w-36">
          <FormLabel :for="`annual${type === 'bill' ? '_bill' : ''}`" name="Annual" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model.lazy="annual" class="min-w-38"
              name="annual"
              :locale="currencyInputLocale(prjCurrency)" :disabled="true"
              :prefix="props.prefix" :suffix="props.suffix"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="-9999999999" :max="9999999999"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model.lazy="annual" class="min-w-38"
              name="annual_bill"
              :disabled="true"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="-9999999999" :max="9999999999"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
            />
          </template>
        </div>
        <div class="item min-w-6 self-center">
          <label>&nbsp;</label>
          <span>=</span>
        </div>
      </div>
      <div class="grid gap-2 grid-cols-2 md:grid-cols-6">
        <div v-for="bItem in biMonths" :key="props.type + bItem.key" class="item">
          <FormLabel :for="props.type + biMonthsValues[bItem.key].name" :name="biMonthsValues[bItem.key].name" class="text-center" />
          <template v-if="type === 'usage'">
            <InputNumber v-model.lazy="biMonthsValues[bItem.key].value" :placeholder="biMonthsValues[bItem.key].placeholder"
              :input-id="props.type + biMonthsValues[bItem.key].name" fluid
              input-class="placeholder:text-gray"
              :prefix="props.prefix" :suffix="props.suffix"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="99999999"
              :locale="currencyInputLocale(prjCurrency)"
              :disabled="biMonthsValues[bItem.key].disabled"
              :invalid="useInvalid && biMonthsValues[bItem.key].invalid"
              @value-change="onItemChange($event, bItem.key)"
            />
          </template>
          <template v-else-if="type === 'bill'">
            <InputNumber v-model.lazy="biMonthsValues[bItem.key].value" :placeholder="biMonthsValues[bItem.key].placeholder"
              :input-id="props.type + biMonthsValues[bItem.key].name" fluid
              input-class="placeholder:text-gray"
              :min-fraction-digits="2" :max-fraction-digits="2"
              :min="0" :max="99999999"
              mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
              :disabled="biMonthsValues[bItem.key].disabled"
              :invalid="useInvalid && biMonthsValues[bItem.key].invalid"
              @value-change="onItemChange($event, bItem.key)"
            />
          </template>
        </div>
        <!-- <div class="item">
          <FormLabel :for="b1Name" :name="b1Name" class="text-center" />
          <InputNumber
            v-model="b1" input-id="b1" fluid
            locale="en-US" :placeholder="b1Ph" input-class="placeholder:text-gray"
            :prefix="props.prefix" :suffix="props.suffix"
            :min-fraction-digits="0" :max-fraction-digits="props.maxFractionDigits"
            :min="-9999999999" :max="9999999999" :disabled="b1Disabled"
            :invalid="useInvalid && b1Invalid"
            @blur="onItemChange($event, 'b1')"
          />
        </div> -->
      </div>
    </div>
  </SkeletonWrap>
</template>

<script setup lang="ts">
import FormLabel from '~/components/Forms/FormLabel.vue'

type StartMonthNumber = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12
type BMonthKey = 'b1' | 'b2' | 'b3' | 'b4' | 'b5' | 'b6'
type BMonthValue = 1 | 2 | 3 | 4 | 5 | 6
type CompProps = {
  prefix?: undefined | string
  suffix?: undefined | string
  maxFractionDigits?: number
  editableMonths?: number[]
  details: any
  chartLoading: boolean
  chartData: number[]
  // defaultData: {
  //   b1: number | null
  //   b2: number | null
  //   b3: number | null
  //   b4: number | null
  //   b5: number | null
  //   b6: number | null
  //   editedNumArr: BMonthValue[]
  //   chartData: number[]
  // }
  startMonth?: StartMonthNumber
  useInvalid?: boolean
  usePlaceholder?: boolean
  // dataSource?: 'bimonthly_consumption' | 'bimonthly_bill'
  type?: 'bill' | 'usage'
  computeValues?: number[]
}
const props = withDefaults(defineProps<CompProps>(), {
  useInvalid: false,
  usePlaceholder: false,
  maxFractionDigits: 2,
  startMonth: 1,
  editableMonths: () => [1, 2, 3, 4, 5, 6],
  type: 'usage',
})
const emits = defineEmits<{
  'on-item-change': [
    data: {
      annual: number | null
      key: BMonthKey
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
// const isUsage = computed(() => props.type === 'usage')
const chartData = computed(() => props.chartData)
const editedNumArr = computed(() => isBill.value ? props.details?.billIndex : props.details?.consumptionIndex)
const phMaps = computed<Record<BMonthKey, number>>(() => {
  const data = props.details
  if (isBill.value) {
    return {
      b1: data?.biMonth1Bill,
      b2: data?.biMonth2Bill,
      b3: data?.biMonth3Bill,
      b4: data?.biMonth4Bill,
      b5: data?.biMonth5Bill,
      b6: data?.biMonth6Bill,
    }
  }
  return {
    b1: data?.biMonth1Consumption,
    b2: data?.biMonth2Consumption,
    b3: data?.biMonth3Consumption,
    b4: data?.biMonth4Consumption,
    b5: data?.biMonth5Consumption,
    b6: data?.biMonth6Consumption,
  }
})

const annual = ref<number | null>(null)
const biMonths = ref<{ key: BMonthKey, number: BMonthValue }[]>([
  { key: 'b1', number: 1 },
  { key: 'b2', number: 2 },
  { key: 'b3', number: 3 },
  { key: 'b4', number: 4 },
  { key: 'b5', number: 5 },
  { key: 'b6', number: 6 },
])

const biMonthsValues = ref<Record<BMonthKey, { value: null | number, phValue: number, placeholder: undefined | string, name: ComputedRef<string>, disabled: ComputedRef<boolean>, invalid: boolean }>>({
  b1: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth)), disabled: computed(() => !props.editableMonths?.includes(1)), invalid: false },
  b2: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 2)), disabled: computed(() => !props.editableMonths?.includes(2)), invalid: false },
  b3: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 4)), disabled: computed(() => !props.editableMonths?.includes(3)), invalid: false },
  b4: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 6)), disabled: computed(() => !props.editableMonths?.includes(4)), invalid: false },
  b5: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 8)), disabled: computed(() => !props.editableMonths?.includes(5)), invalid: false },
  b6: { value: null, phValue: 0, placeholder: undefined, name: computed(() => generateLabel(props.startMonth + 10)), disabled: computed(() => !props.editableMonths?.includes(6)), invalid: false },
})

function generateLabel(startMonth: number) {
  const monthNames = [
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec',
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec',
  ]
  return `${monthNames[startMonth - 1]}-${monthNames[startMonth]}`
}

async function onItemChange($event: number | null, key: BMonthKey) {
  // console.log('onItemChange', $event, key)
  biMonthsValues.value[key].value = $event
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
    total = biMonths.value.reduce((acc, bmonth) => {
      return $number(acc).add(biMonthsValues.value[bmonth.key].value || 0).value
    }, 0)
  }
  else {
    total = biMonths.value.reduce((acc, bmonth) => {
      const waitAddNum = biMonthsValues.value[bmonth.key].value == null
        // ? convertToNumber(biMonthsValues.value[bmonth.key].placeholder as string)
        ? biMonthsValues.value[bmonth.key].phValue
        : biMonthsValues.value[bmonth.key].value
      return $number(acc).add(waitAddNum as number).value
    }, 0)
  }
  annual.value = total
  return total
}

function clearValues() {
  biMonths.value.forEach((bItem) => {
    biMonthsValues.value[bItem.key].value = null
    biMonthsValues.value[bItem.key].placeholder = undefined
  })
}

function getCurrentValues() {
  const inputValues = biMonths.value.map(bMonth => biMonthsValues.value[bMonth.key as BMonthKey].value) ?? []
  const inputIndexArr = getNonNullIndices(inputValues)
  const pdValues = biMonths.value.map(bMonth => biMonthsValues.value[bMonth.key as BMonthKey].placeholder) ?? []
  const userValues = mergeUsageArrays(inputValues, pdValues.map(v => convertToNumber(v)))
  return {
    chartData: chartData.value ?? [],
    pdValues,
    inputValues,
    userValues,
    inputIndexArr,
  }
}

function initialize() {
  if (!props.usePlaceholder) {
    biMonths.value.forEach((bItem) => {
      biMonthsValues.value[bItem.key].value = phMaps.value?.[bItem.key]
    })
  }
  else {
    biMonths.value.forEach((bItem) => {
      biMonthsValues.value[bItem.key].phValue = phMaps.value?.[bItem.key]
      biMonthsValues.value[bItem.key].placeholder
        = `${props.type === 'usage' ? getNumberFormat(phMaps.value?.[bItem.key], ' kWh') : getMoney(phMaps.value?.[bItem.key])}`
        // = `${props.prefix ?? ''}${phMaps.value?.[bItem.key]?.toString()}${props.suffix ?? ''}`
    })

    if (editedNumArr.value && editedNumArr.value.length > 0) {
      editedNumArr.value.forEach((bNum: BMonthValue) => {
        const bmonthKey = biMonths.value.find(v => bNum === v.number)!.key
        biMonthsValues.value[bmonthKey].value = phMaps.value?.[bmonthKey]
      })
    }
  }
}

const watcher = watchEffect(() => {
  computeTotal()
})

watch([() => props.details, () => props.chartData], (vals) => {
  initialize()
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
