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
        for="userPhone" name="Phone Number" />
      <div class="flex gap-2 items-center">
        <InputText v-model="userPhone" fluid />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.userPhone }}</small>
    </div>

    <div class="flex flex-col flex-1">
      <FormLabel class="block mb-1 text-sm" 
        for="userEmail" name="Email" />
      <div class="flex gap-2 items-center">
        <InputText v-model="userEmail" fluid
          :min="0" :max="200"
        />
      </div>
      <small class="mt-1 text-xs text-negative">{{ errors.userEmail }}</small>
    </div>
  </form>
</template>

<script setup lang="ts">
import z from 'zod'
import FormLabel from '~/components/Forms/FormLabel.vue'

const validationSchema = toTypedSchema(
  z.object({
    userPhone: z.optional(z.string()).nullable(),
    userEmail: z.optional(z.string().email()).nullable(),
  }),
)
const { defineField, handleSubmit, errors, handleReset } = useForm({
  validationSchema,
  initialValues: {
    userPhone: null,
    userEmail: null,
  }
})

const [userPhone] = defineField('userPhone')
const [userEmail] = defineField('userEmail')

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
