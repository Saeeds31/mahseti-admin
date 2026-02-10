<template>
  <div class="container py-4">
    <b-card>
      <h5 class="mb-3">ایجاد استان</h5>
      <b-form @submit.prevent="handleSubmit">
        <b-row>
          <b-col cols="12" md="6">
            <b-form-group label="نام استان" label-for="name">
              <b-form-input id="name" v-model="form.name" :state="errors.name ? false : null" />
              <small v-if="errors.name" class="text-danger">{{ errors.name[0] }}</small>
            </b-form-group>
          </b-col>
        </b-row>

        <div class="mt-3">
          <b-button type="submit" :disabled="loading" variant="success">ایجاد استان</b-button>
        </div>
      </b-form>
    </b-card>
  </div>
</template>

<script setup>
import { reactive } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'

const form = reactive({
  name: '',
})

const errors = reactive({})
let loading = ref(false);

const handleSubmit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  loading.value = true;
  try {
    await axios.post('/provinces', form)
    toast.success('استان با موفقیت ایجاد شد ✅')
    Object.keys(form).forEach(key => form[key] = '')

  } catch (err) {
    if (err.response?.status === 422) {
      Object.assign(errors, err.response.data.errors)
      toast.error('خطاهای فرم را بررسی کنید ❌')
    } else {
      toast.error('خطا در ارسال اطلاعات ❌')
    }
  } finally {
    loading.value = false;
  }
}
</script>