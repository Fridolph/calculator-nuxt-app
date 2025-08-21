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
        for="username" name="Name" required />
      <div class="flex gap-2 items-center">
        <InputText v-model="username" fluid />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.username }}</small>
    </div>

    <div class="flex flex-col flex-1">
      <FormLabel class="block mb-1 text-sm" 
        for="age" name="Age" required />
      <div class="flex gap-2 items-center">
        <InputNumber v-model="age" fluid
          :min="0" :max="200"
        />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.username }}</small>
    </div>
  </form>
</template>

<script setup lang="ts">
import z from 'zod'
import FormLabel from '~/components/Forms/FormLabel.vue'

const validationSchema = toTypedSchema(
  z.object({
    username: z.string(),
    age: z.number().min(18).max(60),
  }),
)
const { defineField, handleSubmit, errors, handleReset } = useForm({
  validationSchema,
  initialValues: {
    username: undefined,
    age: undefined,
  }
})

const [username] = defineField('username')
const [age] = defineField('age')

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
