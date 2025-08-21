<template>
  <TitleDialog v-model:visible="visible" title="Please select the project type">
    <p>Note: The type cannot be changed once selected. Please fill in the information correctly.</p>

    <div class="select-wrap flex gap-2 mt-4">
      <div class="btn-wrap" :class="{ 'is-selected': curSelected === 'systemDesign' }"
        @click="onSelect('systemDesign')">
        <i class="icon-wrap bg-blue">
          <Icon name="gs-basic:add-home" size="16" />
        </i>
        <span>System Design</span>
      </div>
      <div class="btn-wrap" :class="{ 'is-selected': curSelected === 'batteryOnly' }"
        @click="onSelect('batteryOnly')">
        <i class="icon-wrap bg-green">
          <Icon name="gs-basic:battery-plus" size="16" />
        </i>
        <span>Battery Only</span>
      </div>
    </div>

    <template #footer>
      <div class="flex justify-end gap-4">
        <Button size="small" label="Save" class="min-w-20"
          :loading="saveLoading" :disabled="saveLoading"
          @click="onSave" />
      </div>
    </template>
  </TitleDialog>
</template>

<script setup lang="ts">
import TitleDialog from '~/components/Dialogs/TitleDialog.vue'

const emits = defineEmits(['on-save'])

const visible = defineModel<boolean>('visible', { default: false })
const curSelected = ref<ProjectDesignType>('systemDesign')

function onSelect(type: ProjectDesignType) {
  curSelected.value = type
}

const saveLoading = ref(false)

function stopLoading() {
  saveLoading.value = false
}
function onSave() {
  emits('on-save', curSelected.value)
  saveLoading.value = true
}

defineExpose({
  stopLoading,
})
</script>

<style scoped lang="scss">
.btn-wrap {
  @apply flex-1 p-3 rounded flex items-center gap-2 border border-grey-line cursor-pointer hover:border-blue;
  &.is-selected {
    @apply border-blue;
  }
}
.icon-wrap {
  @apply flex justify-center items-center size-6 rounded text-white;
}
</style>
