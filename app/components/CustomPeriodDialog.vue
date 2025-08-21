<template>
  <TitleDialog v-model:visible="visible" :title="t('custom_period_usage')" class="w-115">
    <section class="flex flex-col gap-4">
      <!-- <div class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="customUsageStartTime" name="Start Date"
          required />
        <div class="flex gap-2 items-center">
          <DatePicker v-model="customUsageStartTime" show-icon date-format="M. dd, yy"
            icon-display="input" input-class="datepicker-wrap"
            :pt="{ root: 'w-full!', inputIconContainer: 'left-2! w-6!' }"
            @value-change="dateStartChange">
            <template #inputicon>
              <Icon name="material-symbols:calendar-today" size="1rem" />
            </template>
          </DatePicker>
        </div>
      </div> -->
      <div class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="customUsageTime" :name="t('bill_date_range')"
          required />
        <div class="flex gap-2 items-center">
          <DatePicker v-model="customUsageRangeTime" selection-mode="range"
            :max-date="setMaxDate()"
            show-icon date-format="M. dd, yy"
            :locale="useCurrentLocale()"
            icon-display="input" input-class="datepicker-wrap"
            :pt="{ root: 'w-full!', inputIconContainer: 'left-2! w-6!' }"
            @value-change="dateRangeChange">
            <template #inputicon>
              <Icon name="material-symbols:calendar-today" size="1rem" />
            </template>
          </DatePicker>
        </div>
      </div>
      <div class="flex flex-col flex-1">
        <FormLabel class="block mb-2 text-sm" for="customBillUsage" :name="t('bill_usage')"
          required />
        <div class="flex gap-2 items-center">
          <InputNumber v-model="customBillUsage"
            input-id="customBillUsage"
            autocomplete="off"
            :disabled="false"
            :min="0" :max="99999999"
            input-class="gs-inputnumber"
            suffix=" kWh" fluid
            :locale="useCurrentLocale()"
            :min-fraction-digits="2" :max-fraction-digits="2"
            @value-change="$event => customBillUsage"
          />
        </div>
      </div>
    </section>

    <template #footer>
      <div class="flex justify-end gap-2">
        <Button :label="t('cancel_button')" size="small" class="min-w-20"
          outlined @click="onCancel" />
        <Button :label="t('update')" size="small" class="min-w-20"
          :disabled="isLoading || isDisabled"
          :loading="isLoading"
          @click="onUpdate" />
      </div>
    </template>
  </TitleDialog>
</template>

<script setup lang="ts">
import TitleDialog from '~/components/Dialogs/TitleDialog.vue'
import FormLabel from '~/components/Forms/FormLabel.vue'
import { useCurrentLocale } from '~/composables/useCurrency'

const props = defineProps<{
  defaultValues: {
    customUsageStartTime: string
    customUsageEndTime: string
    customBillUsage: number | null
  }
}>()
const { t } = useI18n()
const emits = defineEmits(['update', 'close'])
const $dayjs = useDayjs()
const visible = defineModel<boolean>('visible', { default: false })
const customUsageStartTime = ref<any>('')
const customUsageEndTime = ref<any>('')
const customUsageRangeTime = ref<any>('')
const customBillUsage = ref<number | null>(null)

const isDisabled = computed(() => {
  return !customUsageStartTime.value || !customUsageEndTime.value || !customBillUsage.value
})

const openWc = watch(() => visible.value, (val) => {
  if (!val) return
  const { defaultValues } = props
  customUsageStartTime.value = defaultValues.customUsageStartTime
  customUsageEndTime.value = defaultValues.customUsageEndTime
  customUsageRangeTime.value = [
    new Date($dayjs(defaultValues.customUsageStartTime).valueOf()),
    new Date($dayjs(defaultValues.customUsageEndTime).valueOf()),
  ]
  customBillUsage.value = defaultValues.customBillUsage
}, { immediate: true })

onUnmounted(() => {
  openWc()
})

function onCancel() {
  emits('close')
}

function dateRangeChange(evt: [Date | null, Date | null]) {
  const isValid = !evt.some(v => ['', null, undefined].includes(v))
  if (isValid) {
    const sTime = `${$dayjs(evt[0]).format('YYYY-MM-DD')} 00:00:00`
    customUsageStartTime.value = sTime
    const eTime = `${$dayjs(evt[1]).format('YYYY-MM-DD')} 00:00:00`
    customUsageEndTime.value = eTime
    console.log('🚀 ~ dateRangeChange:', evt)
    console.log('sTime', sTime, $dayjs(sTime).utc().valueOf())
    console.log('eTime', eTime, $dayjs(eTime).utc().valueOf())
  }
}

const isLoading = ref<boolean>(false)
const updateLoading = (status: boolean) => isLoading.value = status

function onUpdate() {
  isLoading.value = true
  emits('update', {
    customUsageStartTime: $dayjs(customUsageStartTime.value).utc().valueOf(),
    customUsageEndTime: $dayjs(customUsageEndTime.value).utc().valueOf(),
    customBillUsage: customBillUsage.value,
  })
}

function setMaxDate() {
  const today = new Date()
  const month = today.getMonth()
  const year = today.getFullYear()
  // const prevMonth = (month === 0) ? 11 : month - 1
  // const prevYear = (prevMonth === 11) ? year - 1 : year
  // const nextYear = (nextMonth === 0) ? year + 1 : year
  const date = today.getDate()
  const maxDate = new Date()

  maxDate.setMonth(month)
  maxDate.setFullYear(year)
  maxDate.setDate(date)
  return maxDate
}

defineExpose({
  updateLoading,
})
</script>
