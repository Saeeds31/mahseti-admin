<template>
  <div class="container mt-4">
    <h3>ایجاد اسلایدر جدید</h3>
    <form @submit.prevent="submitForm" class="row g-3">
      <!-- عنوان -->
      <div class="col-md-6">
        <label class="form-label">عنوان</label>
        <input v-model="form.title" type="text" class="form-control" required maxlength="255" />
      </div>

      <!-- لینک -->
      <div class="col-md-6">
        <label class="form-label">لینک (اختیاری)</label>
        <input v-model="form.link" type="text" class="form-control" maxlength="255" />
      </div>

      <!-- توضیحات -->
      <div class="col-12 col-md-8">
        <label class="form-label">توضیحات (اختیاری)</label>
        <Editor v-model="form.description" />
      </div>

      <!-- تصویر -->
      <div class="col-12 col-md-4">
        <label class="form-label">انتخاب تصویر</label>
        <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png" theme="grid" deletable sortable />
      </div>

      <!-- نوع -->
      <div class="col-md-6">
        <label class="form-label">نوع اسلایدر</label>
        <select v-model="form.type" class="form-select">
          <option value="">انتخاب کنید</option>
          <option value="desktop">دسکتاپ</option>
          <option value="mobile">موبایل</option>
        </select>
      </div>

      <!-- متن دکمه -->
      <div class="col-md-6">
        <label class="form-label">متن دکمه (اختیاری)</label>
        <input v-model="form.button_text" type="text" class="form-control" maxlength="100" />
      </div>

      <div class="col-12">
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ loading ? 'در حال ذخیره...' : 'ذخیره' }}
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Editor from '@/components/shared/editor.vue'

const form = ref({
  title: '',
  link: '',
  description: '',
  image: null,
  type: '',
  button_text: '',
})
function imageLoaded(files) {
  form.value.image = files[0].file
}
const loading = ref(false)

const submitForm = async () => {
  try {
    loading.value = true
    const formData = new FormData()
    for (let key in form.value) {
      if (form.value[key]) formData.append(key, form.value[key])
    }
    await axios.post('/sliders', formData)
    toast.success('اسلایدر با موفقیت ایجاد شد')
    form.value = {
      title: '',
      link: '',
      description: '',
      image: null,
      type: '',
      button_text: '',
    }
  } catch (e) {
    toast.error('خطا در ایجاد اسلایدر')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* استایل دلخواه */
</style>