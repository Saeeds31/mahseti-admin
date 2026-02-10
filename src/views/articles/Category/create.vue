<template>
  <div class="p-4">
    <b-card title="ایجاد دسته‌بندی مقاله">
      <b-form class="row" @submit.prevent="submitForm">
        <!-- Title -->
        <b-form-group label="عنوان" class="col-md-6" label-for="title" description="عنوان دسته‌بندی">
          <b-form-input id="title" v-model="form.title" :state="!errors.title"
            placeholder="عنوان را وارد کنید"></b-form-input>
          <b-form-invalid-feedback v-if="errors.title">{{ errors.title }}</b-form-invalid-feedback>
        </b-form-group>

        <!-- Slug -->
        <b-form-group label="Slug" label-for="slug" class="col-md-6" description="آدرس یکتا">
          <b-form-input id="slug" v-model="form.slug" :state="!errors.slug" placeholder="slug دسته‌بندی"></b-form-input>
          <b-form-invalid-feedback v-if="errors.slug">{{ errors.slug }}</b-form-invalid-feedback>
        </b-form-group>

        <!-- Parent -->
        <b-form-group label="دسته‌بندی والد" label-for="parent_id" class="col-md-6">

          <Treeselect id="parent_id" v-model="form.parent_id" :normalizer="normalizer" :options="parentOptions"
            placeholder="انتخاب دسته‌بندی والد" :clearable="true" />
          <b-form-invalid-feedback v-if="errors.parent_id">{{ errors.parent_id }}</b-form-invalid-feedback>
        </b-form-group>

        <!-- Meta Title -->
        <b-form-group label="Meta Title" label-for="meta_title" class="col-md-6">
          <b-form-input id="meta_title" v-model="form.meta_title" :state="!errors.meta_title"></b-form-input>
          <b-form-invalid-feedback v-if="errors.meta_title">{{ errors.meta_title }}</b-form-invalid-feedback>
        </b-form-group>

        <!-- Meta Description -->
        <b-form-group label="Meta Description" label-for="meta_description" class="col-md-6">
          <b-form-textarea id="meta_description" v-model="form.meta_description" :state="!errors.meta_description"
            rows="3"></b-form-textarea>
          <b-form-invalid-feedback v-if="errors.meta_description">{{ errors.meta_description
          }}</b-form-invalid-feedback>
        </b-form-group>

        <!-- Description -->
        <b-form-group label="توضیحات" label-for="description" class="col-md-6">
          <b-form-textarea id="description" v-model="form.description" :state="!errors.description"
            rows="5"></b-form-textarea>
          <b-form-invalid-feedback v-if="errors.description">{{ errors.description }}</b-form-invalid-feedback>
        </b-form-group>

        <b-button type="submit" :disabled="loading" variant="primary">ذخیره</b-button>
      </b-form>
    </b-card>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { BForm, BFormGroup, BFormInput, BFormSelect, BFormTextarea, BButton, BCard, BFormInvalidFeedback } from 'bootstrap-vue-3'

const form = reactive({
  title: '',
  slug: '',
  parent_id: null,
  meta_title: '',
  meta_description: '',
  description: '',
})
let loading = ref(false);

const errors = reactive({})
const normalizer = (node) => {
  // تبدیل کلیدها به فرمت استاندارد کامپوننت
  return {
    id: node.id,
    label: node.title,
    children: node.children
  }
}
const parentOptions = ref([])
function getParentOption() {
  axios.get("/article-categories-by-child").then((res) => {
    parentOptions.value = res.data.data
  })
}
getParentOption()
const submitForm = async () => {
  errors.value = {}
  loading.value = true;
  try {
    const formData = new FormData()
    for (const key in form) {
      formData.append(key, form[key] ?? '')
    }

    const response = await axios.post('/article-categories', formData)
    toast.success('دسته‌بندی با موفقیت ذخیره شد ✅')
    // ریست فرم بعد از موفقیت
    Object.keys(form).forEach(key => form[key] = key === 'parent_id' ? null : '')
  } catch (err) {
    if (err.response && err.response.status === 422) {
      // خطاهای ولیدیشن لاراول
      const respErrors = err.response.data.errors
      Object.keys(respErrors).forEach(key => {
        if (Array.isArray(respErrors[key])) {
          errors[key] = respErrors[key].join(" - ");
        } else {
          errors[key] = respErrors[key];
        }
      })
      toast.error('لطفا خطاهای فرم را بررسی کنید ❌')
    } else {
      toast.error('خطا در ارسال اطلاعات ❌')
      console.error(err)
    }
  } finally {
    loading.value = false;
  }
}
</script>