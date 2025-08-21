<template>
  <form class="grid grid-cols-2 gap-4" @submit="onSubmit">
    <!-- <InputNumber v-model.lazy="username"
      input-id="electricityPricePerKwh" input-class="gs-inputnumber valid-item"
      fluid autocomplete="off"
      :disabled="false" :invalid="!!errors.electricityPricePerKwh"
      :min="0" :max="99999999"
      :min-fraction-digits="2" :max-fraction-digits="2"
      mode="currency" :currency="prjCurrency" :locale="currencyInputLocale(prjCurrency)"
      @value-change="onModelChange($event, 'electricityPricePerKwh')"
    /> -->
    <div class="flex flex-col flex-1">
      <FormLabel class="block mb-1 text-sm" 
        for="description" name="Description" />
      <div class="flex gap-2 items-center">
        <InputText v-model="description" fluid />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.description }}</small>
    </div>

    <div class="flex flex-col flex-1">
      <FormLabel class="block mb-1 text-sm" 
        for="percent" name="percent" />
      <div class="flex gap-2 items-center">
        <InputNumber v-model="percent" fluid
          :min="0" :max="100"
        />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.percent }}</small>
    </div>
  </form>
</template>

<script setup lang="ts">
import z from 'zod'
import FormLabel from '~/components/Forms/FormLabel.vue'

const validationSchema = toTypedSchema(
  z.object({
    description: z.string().optional().nullable(),
    percent: z.number().min(1).max(99).nullable(),
  }),
)
const { defineField, handleSubmit, errors, handleReset } = useForm({
  validationSchema,
  initialValues: {
    description: null,
    percent: null,
  }
})

const [description] = defineField('description')
const [percent] = defineField('percent')

const onSubmit = handleSubmit(values => {
  console.log(values)
  return Promise.resolve(values)
}, err => {
  console.error(err)
  return Promise.reject(err)
})

defineExpose({
  onSubmit,
  handleReset,
})
</script>
