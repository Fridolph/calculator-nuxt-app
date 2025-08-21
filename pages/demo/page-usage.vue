<template>
  <NuxtLayout name="empty">
    <div class="max-w-full min-h-100vh p-6 bg-white rounded overflow-hidden">
      <div class="btn-wrap flex items-center gap-2">
        <Button class="mr-2 h-38px" @click="onEditElectricityUsageClick">Usage Drawer</Button>

        <CreateProjectBtn @on-click="handleClick" />

      </div>

      <CreateProjectDrawer ref="CreateProjectDrawerRef" :project-type="projectType"
        header-title="Edit Electricity Usage" />

      <section class="mt-6 max-w-200">
        <InputGroupAndChartSwitcher />
      </section>

      <EditElectricityDrawer ref="editElectricityRef" @reload="onReload" />

      <!-- <div class="p-4 lg:max-w-200">
        <div class="bg-white p-2 flex flex-col gap-6 relative border rounded-lg overflow-hidden">
          <div class="mb-2 pt-2 px-2 flex justify-between text-sm">
            <div>{{ t('estimated_average_monthly_usage') }}</div>
            <div class="text-grey-light">
              {{ t('estimate_annual_usage') }}:
              <span class="text-gsblack">{{ formatNumber(1234, { precision: 2, pattern: '# !', symbol: 'kWh', negativePattern: '-# !' }) }}</span>
            </div>
          </div>
          <EnergyChart ref="ChartRef" simple style="height: 160px"
            :loading="chartLoading" :chart-value="usageChartData" />
          <div v-if="false && showChartTips" class="absolute inset-0 flex items-center justify-center bg-white opacity-95">
            {{ showChartTips }}
          </div>
        </div>
      </div> -->
    </div>
  </NuxtLayout>
</template>

<script setup lang="ts">
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
const { t } = useI18n()
const chartLoading = ref(false)
const usageChartData = ref([
  11, 22, 33, 44, 55, 66, 77, 88, 99, 110, 111, 122,
])
const showChartTips = computed(() => 'Please fill in the annual usage')
const projectType = ref<'systemDesign' | 'batteryOnly'>('systemDesign')
const CreateProjectDrawerRef = useTemplateRef('CreateProjectDrawerRef')

function handleClick(type: 'systemDesign' | 'batteryOnly') {
  console.log('🚀 ~ handleClick:', type)
  projectType.value = type
  CreateProjectDrawerRef.value?.open({
    aiAnalysisStatus: 20,
    googleKey: 'AIzaSyC05WHc4fcN52QHoBCEErGkYqBlz-1dDaQ',
    googleSession: 'AJVsH2wmSHCkoW_EdVmG94cuZmqsTC13E7PcTd_GTvG5Nakus61kpXrU865IlbflVRoThOsWCf0dKVVQBk2HGQUcbU3tf5e_MveesGEoAffAGZDjV0IMeOnYGb_QrLqUC7qo9Pzt9g3ji8E',
  })
}

const projectStore = useProjectStore()
const { detailData } = storeToRefs(projectStore)
const { fetchProjectDetail } = projectStore

function onReload() {
  // fetchProjectDetail({ params: { projectId } })
  fetchProjectDetail({ params: { projectId: 903 } })
}

const editElectricityRef = useTemplateRef('editElectricityRef')
function onEditElectricityUsageClick() {
  editElectricityRef.value?.open({ detailData: detailData.value })
}
</script>
