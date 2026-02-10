<template>
  <div class="container mt-4">
    <h3>ساخت بازه قیمتی</h3>
    <form @submit.prevent="submitForm" class="row g-3">
      <!-- Province -->
      <div class="col-md-6">
        <label class="form-label">استان</label>
        <Treeselect id="category_ids" :multiple="false" v-model="province_id" :options="provinces"
          placeholder="انتخاب استان " :clearable="true" :valueConsistsOf="'ALL'" />
      </div>
      <!-- City -->
      <div class="col-md-6">
        <label class="form-label">شهر(اختیار)</label>
        <Treeselect id="category_ids" :multiple="false" v-model="form.city_id" :options="cities"
          placeholder="انتخاب شهر " :clearable="true" :valueConsistsOf="'ALL'" />
      </div>

      <!-- Cost -->
      <div class="col-md-6">
        <label class="form-label">قیمت</label>
        <input v-model="form.cost" type="number" min="0" class="form-control" required />
      </div>

      <!-- Min Order Amount -->
      <div class="col-md-6">
        <label class="form-label">حداقل مبلغ سفارش (اختیار)</label>
        <input v-model="form.min_order_amount" type="number" min="0" class="form-control" />
      </div>

      <!-- Max Order Amount -->
      <div class="col-md-6">
        <label class="form-label">حداکثر مبلغ سفارش (اختیار)</label>
        <input v-model="form.max_order_amount" type="number" min="0" class="form-control" />
      </div>

      <div class="col-12">
        <button type="submit" class="btn btn-primary" :disabled="loading">
          {{ loading ? 'در حال ذخیره سازی' : 'ذخیره سازی' }}
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, reactive } from 'vue'
import axios from 'axios'

import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { useRoute } from 'vue-router'


const form = reactive({
  city_id: null,
  cost: null,
  min_order_amount: null,
  max_order_amount: null,
})
let province_id = ref(null);
const route = useRoute()
const provinces = ref([])
const cities = ref([])
const loading = ref(false)
const loadInitialData = async () => {
  try {
    const { data } = await axios.get('/provinces')
    provinces.value = data.data.map(p => ({ id: p.id, label: p.name, value: p.id, cities: p.cities }))
  } catch (e) {
    toast.error('خطایی در دریافت اطلاعات به وجودآمده است')
  }
}

watch(() => province_id.value, (newSelect) => {
  provinceChanged(newSelect)
})
const provinceChanged = async (province_id) => {
  form.city_id = null;
  if (!province_id) {
    cities.value = []
    form.city_id = null
    return
  }
  try {
    let province = provinces.value.find((prov) => prov.id === province_id)
    cities.value = province.cities.map(c => ({ id: c.id, label: c.name, value: c.id }))

  } catch (e) {
    console.log(e);
    toast.error('مشکلی در لود شهرها به وجود آمد')
  }
}

const submitForm = async () => {
  try {
    loading.value = true
    const formData = new FormData()
    for (let key in form) {
      if (form[key]) formData.append(key, form[key])
    }
    formData.append("province_id", province_id.value)
    await axios.post(`/shipping-ranges/${route.params.method}`, formData)
    toast.success('بازه قیمتی با موفقیت ایجاد شد')
    Object.keys(form).forEach(key => (form[key] = null))
    province_id.value = null;
  } catch (e) {
    toast.error('خطایی در ثبت به وجود آمده است')
  } finally {
    loading.value = false
  }
}

onMounted(loadInitialData)
</script>

<style scoped>
/* می‌تونی استایل دلخواه اضافه کنی */
</style>