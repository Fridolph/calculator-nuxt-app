<template>
  <NuxtLayout name="project" layout-class="bg-grey-bg">
    
    <section class="max-w-200 p-4">
      <FormA ref="compARef"></FormA>
      <FormB ref="compBRef"></FormB>
      <FormC ref="compCRef"></FormC>

      <div class="flex gap-4 mt-4">
        <Button @click="validAndSubmit">submit</Button>
        <Button @click="resetAll">reset</Button>
      </div>
    </section>
  </NuxtLayout>
</template>

<script setup lang="ts">
import z from 'zod'
import FormA from './components/FormA.vue'
import FormB from './components/FormB.vue'
import FormC from './components/FormC.vue'

const compARef = useTemplateRef('compARef')
const compBRef = useTemplateRef('compBRef')
const compCRef = useTemplateRef('compCRef')

async function validAndSubmit() {
  const res = await Promise.allSettled([
    compARef.value?.onSubmit(),
    compBRef.value?.onSubmit(),
    compCRef.value?.onSubmit(),
  ])
  console.log('🚀 ~ validAndSubmit ~ res:', res)
}

function resetAll() {
  [compARef, compBRef, compCRef].forEach(v => v.value?.handleReset())
}
</script>