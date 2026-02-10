<template>
  <div class="container mt-4">
    <h3>ویرایش روش حمل و نقل</h3>
    <form @submit.prevent="updateForm" class="row g-3">
      <!-- Name -->
      <div class="col-md-6">
        <label class="form-label">عنوان</label>
        <b-form-input v-model="form.name" type="text" class="form-control" required />

      </div>

      <!-- Default Cost -->
      <div class="col-md-6">
        <label class="form-label">مبلغ پیش فرض (تومان)</label>
        <b-form-input v-model="form.default_cost" type="text" class="form-control" required />

      </div>

      <!-- Description -->
      <div class="col-12">
        <label class="form-label">توضیحات</label>
        <b-form-textarea v-model="form.description"></b-form-textarea>
      </div>

      <!-- Status -->
      <div class="col-md-6">
        <div class="form-check mt-4">
          <b-form-checkbox id="status" v-model="form.status" :true-value="1" :false-value="0">
            فعال
          </b-form-checkbox>
        </div>
      </div>

      <div class="col-12">
        <button type="submit" class="btn btn-success" :disabled="loading">
          {{ loading ? 'در حال بروز رسانی ...' : 'بروز رسانی' }}
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import { BFormTextarea, BFormInput, BFormCheckbox } from "bootstrap-vue-3"

import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'

const route = useRoute()
const router = useRouter()
const form = ref({
  name: '',
  description: '',
  default_cost: null,
  status: false,
})
const loading = ref(false)

const fetchData = async () => {
  try {
    const { data } = await axios.get(`/shipping-methods/${route.params.id}`)
    form.value = data.data
    form.value.status=Number(data.data.status)
  } catch (e) {
    toast.error('Failed to load shipping data')
  }
}

const updateForm = async () => {
  try {
    loading.value = true
    await axios.put(`/shipping-methods/${route.params.id}`, form.value)
    toast.success('روش حمل و نقل ویرایش شد')
  } catch (e) {
    toast.error('خطایی در ویرایش روش به وجود آمده است')
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
</script>

<style scoped>
/* می‌تونی اینجا استایل دلخواه اضافه کنی */
</style>