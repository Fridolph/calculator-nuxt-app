<template>
  <NuxtLayout name="project" layout-class="bg-grey-bg">
    <div class="m-6 p-6 bg-white rounded">
      <section class="bg-white rounded p-4 flex flex-col gap-6">
        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            InputGroupFor12Months.vue - 用电量
          </h2>
          <InputGroupFor12Months ref="For12MonthsKwhRef" type="consumption"
            :max-fraction-digits="2"
            :use-placeholder="true"
            suffix=" kWh"
            :default-data="monthlyKwhData"
            :editable-months="[2, 3, 4, 5, 6, 6, 7, 8, 9, 10, 11]"
            @on-item-change="onMonthKwhChange"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="initMonthlyKwh">
              Init
            </Button>
            <Button @click="clearMonthlyKwh">
              ClearAll
            </Button>
            <Button @click="getMonthKwhValues">
              getMonthKwhValues
            </Button>
          </div>
          <p class="break-all">
            {{ monthlyDataString }}
          </p>
        </section>

        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            InputGroupFor12Months.vue - 账单
          </h2>
          <InputGroupFor12Months ref="For12MonthsBillRef" type="bill"
            :max-fraction-digits="2"
            :use-placeholder="true"
            :editable-months="[6, 7, 8, 9, 10, 11, 12]"
            prefix="$"
            :default-data="defaultMonthBillData"
            data-source="monthly_consumption"
            @on-item-change="onItemChange2"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="handleInit2">
              Init
            </Button>
            <Button @click="clearAll2">
              ClearAll
            </Button>
            <Button @click="getCurrentValues2">
              getCurrentValues2
            </Button>
          </div>
          <p class="break-all">
            {{ monthlyBillString }}
          </p>
        </section>

        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            Kwh > ForBi2Months.vue >>> startMonth: {{ startMonth }}
          </h2>
          <Select v-model="startMonth" :options="startMonthOptions"
            option-label="label" option-value="value" />
          <InputGroupForBi2Months ref="ForBi2MonthsRef" type="consumption"
            :default-data="biMonthKwhData" :start-month="startMonth"
            :use-placeholder="true"
            suffix=" kWh"
            @on-item-change="onBiMonthsItemChange"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="initBiMonthKwh">
              Init
            </Button>
            <Button @click="clearBiMonthKwh">
              ClearAll
            </Button>
            <Button @click="getBiMonthKwhValues">
              getCurrentValues
            </Button>
          </div>
          <p class="break-all">
            {{ ForBi2MonthsDataString }}
          </p>
        </section>

        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            Bill > ForBi2Months.vue >>> startMonth: {{ startMonth }}
          </h2>
          <InputGroupForBi2Months ref="ForBi2MonthsBillRef"
            :default-data="biMonthBillData" :start-month="startMonth"
            :use-placeholder="true"
            type="consumption"
            prefix="$"
            data-source="bimonthly_consumption"
            @on-item-change="onBiMonthsBillItemChange"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="initBiMonthBill">
              Init
            </Button>
            <Button @click="clearBiMonthBill">
              ClearAll
            </Button>
            <Button @click="getBiMonthBillValues">
              getCurrentValues
            </Button>
          </div>
          <p class="break-all">
            {{ ForBi2MonthsBillDataString }}
          </p>
        </section>

        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            kWh > ForQuarterly.vue >>> startMonth: {{ startMonth }}
          </h2>
          <InputGroupForQuarterly ref="ForQuarterlyRef"
            :default-data="quarterData" :start-month="startMonth"
            :use-placeholder="true" suffix="kWh"
            @on-item-change="onQuarterItemChange"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="initQuarter">
              Init
            </Button>
            <Button @click="clearQuater">
              ClearAll
            </Button>
            <Button @click="getQuarterValues">
              getCurrentValues
            </Button>
          </div>
          <p class="break-all">
            {{ ForQuarterDataString }}
          </p>
        </section>

        <section>
          <h2 class="proposal-model-title mb-4" data-borderbar="static">
            Bill > ForQuarterly.vue >>> startMonth: {{ startMonth }}
          </h2>
          <InputGroupForQuarterly ref="ForQuarterlyBillRef" type="bill"
            :default-data="quarterBillData" :start-month="startMonth"
            :use-placeholder="true" prefix="$"
            @on-item-change="onQuarterBillItemChange"
          />
          <div class="flex gap-2 mt-4">
            <Button @click="initQuarterBill">
              Init
            </Button>
            <Button @click="clearQuaterBill">
              ClearAll
            </Button>
            <Button @click="getQuarterBillValues">
              getCurrentValues
            </Button>
          </div>
          <p class="break-all">
            {{ ForQuarterBillString }}
          </p>
        </section>
      </section>
    </div>
  </NuxtLayout>
</template>

<script setup lang="ts">
import InputGroupFor12Months from '../projects/[projectId]/@components/analysis/InputGroupFor12Months.vue'
import InputGroupForBi2Months from '../projects/[projectId]/@components/analysis/InputGroupForBi2Months.vue'
import InputGroupForQuarterly from '../projects/[projectId]/@components/analysis/InputGroupForQuarterly.vue'

definePageMeta({
  layout: false,
})

withDefaults(defineProps<{
  dataLoading?: boolean
  canEdit?: boolean
}>(), {
  dataLoading: false,
  canEdit: true,
})

// ------------------------ MonthlyInputGroup.vue 用电量 ------------------------
const monthlyKwhData = ref({
  editedNumArr: [2, 11],
  chartData: [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200],
})

const For12MonthsKwhRef = useTemplateRef('For12MonthsKwhRef')
const clearMonthlyKwh = () => For12MonthsKwhRef.value?.clearValues()
const initMonthlyKwh = () => For12MonthsKwhRef.value?.initialize()

const monthlyDataString = ref('')
const getMonthKwhValues = () => {
  const data = For12MonthsKwhRef.value?.getCurrentValues()
  monthlyDataString.value = JSON.stringify(data)
}

function onMonthKwhChange(evt) {
  console.log('🚀 ~ onItemChange:', evt)
  getMonthKwhValues()
}

// ------------------------ MonthlyInputGroup.vue 账单 ------------------------
const defaultMonthBillData = ref<{ editedNumArr: unknown[], chartData: unknown[] }>({
  editedNumArr: [2, 11],
  chartData: [11.1, 22.2, 33.3, 44.4, 55.5, 66.6, 77.7, 88.8, 99.9, 100.10, 111.11, 122.22],
})

const For12MonthsBillRef = useTemplateRef('For12MonthsBillRef')
const handleInit2 = () => For12MonthsBillRef.value?.initialize()
const clearAll2 = () => For12MonthsBillRef.value?.clearValues()

const monthlyBillString = ref('')
const getCurrentValues2 = () => {
  const data = For12MonthsBillRef.value?.getCurrentValues()
  monthlyBillString.value = JSON.stringify(data)
}

function onItemChange2(evt) {
  console.log('🚀 ~ onItemChange:', evt)
  getCurrentValues2()
}

// ------------------------ InputGroupForBi2Months.vue Kwh ------------------------
/* global MonthKey MonthValue */
const startMonthOptions = ref<{ label: MonthKey, value: MonthValue }[]>([
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
])
type StartMonth = keyof typeof startMonthOptions['value']
const startMonth = ref<StartMonth>(5)
const biMonthKwhData = ref({
  editedNumArr: [2, 5],
  b1: 222,
  b2: 444,
  b3: 666,
  b4: 888,
  b5: 777,
  b6: 555,
  chartData: [100, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 1200],
})

const ForBi2MonthsRef = useTemplateRef('ForBi2MonthsRef')
const ForBi2MonthsDataString = ref('')
const initBiMonthKwh = () => {
  ForBi2MonthsDataString.value = ''
  ForBi2MonthsRef.value?.initialize()
}
const clearBiMonthKwh = () => {
  ForBi2MonthsRef.value?.clearValues()
}

const getBiMonthKwhValues = () => {
  const data = ForBi2MonthsRef.value?.getCurrentValues()
  ForBi2MonthsDataString.value = JSON.stringify(data)
}

function onBiMonthsItemChange(evt) {
  console.log('🚀 ~ onItemChange:', evt)
  getBiMonthKwhValues()
}

// ------------------------ InputGroupForBi2Months.vue Bill ------------------------
const biMonthBillData = ref({
  editedNumArr: [1, 3],
  b1: 33.3,
  b2: 55.5,
  b3: 77.7,
  b4: 66.6,
  b5: 22.22,
  b6: 11.11,
  chartData: [11, 22, 33, 44, 55, 66, 77, 88, 99, 77, 55, 33],
})

const ForBi2MonthsBillRef = useTemplateRef('ForBi2MonthsBillRef')
const ForBi2MonthsBillDataString = ref('')
const initBiMonthBill = () => {
  ForBi2MonthsBillDataString.value = ''
  ForBi2MonthsBillRef.value?.initialize()
}
const clearBiMonthBill = () => {
  ForBi2MonthsBillRef.value?.clearValues()
}

const getBiMonthBillValues = () => {
  const data = ForBi2MonthsBillRef.value?.getCurrentValues()
  ForBi2MonthsBillDataString.value = JSON.stringify(data)
}

function onBiMonthsBillItemChange(evt) {
  console.log('🚀 ~ onBiMonthsBillItemChange:', evt)
  getBiMonthBillValues()
}

// ------------------------ InputGroupForQuarterly.vue kWh ------------------------
const quarterData = ref({
  editedNumArr: [3],
  q1: 331.3,
  q2: 553.5,
  q3: 775.7,
  q4: 663.6,
  chartData: [11, 22, 33, 44, 55, 66, 77, 88, 99, 77, 55, 33],
})

const ForQuarterlyRef = useTemplateRef('ForQuarterlyRef')
const ForQuarterDataString = ref('')
const initQuarter = () => {
  ForQuarterDataString.value = ''
  ForQuarterlyRef.value?.initialize()
}
const clearQuater = () => {
  ForQuarterlyRef.value?.clearValues()
}

const getQuarterValues = () => {
  const data = ForQuarterlyRef.value?.getCurrentValues()
  ForQuarterDataString.value = JSON.stringify(data)
}

function onQuarterItemChange(evt) {
  console.log('🚀 ~ onQuarterItemChange:', evt)
  getQuarterValues()
}

// ------------------------ InputGroupForQuarterly.vue Bill ------------------------
const quarterBillData = ref({
  editedNumArr: [3],
  q1: 1331.3,
  q2: 2553.5,
  q3: 1775.7,
  q4: 2663.6,
  chartData: [111, 222, 333, 444, 555, 666, 777, 888, 999, 777, 555, 333],
})

const ForQuarterlyBillRef = useTemplateRef('ForQuarterlyBillRef')
const ForQuarterBillString = ref('')
const initQuarterBill = () => {
  ForQuarterBillString.value = ''
  ForQuarterlyBillRef.value?.initialize()
}
const clearQuaterBill = () => {
  ForQuarterlyRef.value?.clearValues()
}

const getQuarterBillValues = () => {
  const data = ForQuarterlyBillRef.value?.getCurrentValues()
  ForQuarterBillString.value = JSON.stringify(data)
}

function onQuarterBillItemChange(evt) {
  console.log('🚀 ~ onQuarterBillItemChange:', evt)
  getQuarterBillValues()
}
</script>
