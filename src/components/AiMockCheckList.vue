<template>
  <div class="scroll-container custom-border-wrap">
    <div v-for="item in list" :key="item.id">
      <div class="flex h-6 items-center gap-1">
        <Icon v-if="item.status === CheckStatus.Waiting"
          name="material-symbols:check-rounded" size="1.5rem" class="shink-0 invisible text-grey-line" />
        <i v-else-if="item.status === CheckStatus.Checking" class="shink-0 gs-loader w-14px! h-14px!" />
        <Icon v-else-if="item.status === CheckStatus.Completed"
          name="material-symbols:circle-outline" size="1.2rem" class="shink-0 text-green" />
        <Icon v-else-if="item.status === CheckStatus.Error"
          name="gs-basic:error-warning" size="20px" class="shink-0 text-negative" />
        <strong class="text-sm"
          :class="{
            'text-negative': item.status === CheckStatus.Error,
            'text-grey-light': item.status === CheckStatus.Waiting,
          }">
          {{ item.title }}
        </strong>
        <Icon v-if="item.status === CheckStatus.Checking" name="svg-spinners:3-dots-fade" size="1rem" />
      </div>
      <section v-if="item.children && item.children.length > 0" class="flex flex-col mt-3 gap-3">
        <div v-for="li in item.children" :key="li.id"
          class="flex h-6 items-center gap-1 pl-6">
          <Icon v-if="li.status === CheckStatus.Waiting"
            name="material-symbols:check-rounded" size="1.5rem" class="shrink-0 invisible text-grey-line" />
          <i v-else-if="li.status === CheckStatus.Checking" class="shrink-0 gs-loader w-14px! h-14px!" />
          <Icon v-else-if="li.status === CheckStatus.Completed"
            name="material-symbols:circle-outline" size="1.2rem" class="shrink-0 text-green" />
          <span :class="{
            'text-sm': true,
            'text-negative': li.status === CheckStatus.Error,
            'text-grey-light': li.status === CheckStatus.Waiting,
          }">{{ li.title }}</span>
          <Icon v-if="li.status === CheckStatus.Checking" name="svg-spinners:3-dots-fade" size="1rem" />
        </div>
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { CheckStatus } from '~/composables/projects/useAnalysis'
import type { CheckItem } from '~/composables/projects/useAnalysis'

const { list } = defineProps<{
  list: CheckItem[]
}>()
</script>
