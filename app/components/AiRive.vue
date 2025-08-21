<template>
  <ClientOnly>
    <NuxtRive id="rive" ref="riveComp"
      :class="[sizeClass]"
      :rive-params="riveParams"
      :options="riveOptions" />
  </ClientOnly>
</template>

<script setup lang="ts">
import { Layout, Fit, Alignment } from '@rive-app/webgl'

withDefaults(defineProps<{
  sizeClass?: string
}>(), {
  sizeClass: 'size-45px',
})

const riveComp = useTemplateRef('riveComp')

const riveOptions = {
  fitCanvasToArtboardHeight: false,
  useOffscreenRenderer: true,
}

const riveParams = {
  src: '/rives/ai.riv',
  autoplay: true,
  layout: new Layout({
    fit: Fit.Cover,
    alignment: Alignment.Center,
  }),
  isAnimationStateMachine: true,
  // stateMachines: mode.value === 'power' ? 'Charging' : 'Power Selling',
  // artboard: 'Artboard',
}
</script>
