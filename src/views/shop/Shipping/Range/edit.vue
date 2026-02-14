<template>
  <div class="container mt-4" v-if="checkPermission(['shipping_update'])">
    <h3>ویرایش بازه حمل و نقل</h3>
    <form @submit.prevent="updateForm" class="row g-3">
      <!-- Province -->
      <div class="col-md-6">
        <label class="form-label">استان</label>
        <Treeselect id="category_ids" :multiple="false" v-model="province_id" v-if="provinces.length"
          :options="provinces" placeholder="انتخاب استان " :clearable="true" :valueConsistsOf="'ALL'" />
      </div>
      <!-- City -->
      <div class="col-md-6">
        <label class="form-label">شهر(اختیار)</label>
        <Treeselect id="category_ids" :multiple="false" v-if="cities.length" v-model="form.city_id" :options="cities"
          placeholder="انتخاب شهر " :clearable="true" :valueConsistsOf="'ALL'" />
      </div>
      <!-- Cost -->
      <div class="col-md-6">
        <label class="form-label">مبلغ</label>
        <input v-model="form.cost" type="number" min="0" class="form-control" required />
      </div>

      <!-- Min Order Amount -->
      <div class="col-md-6">
        <label class="form-label">حداقل سفارش (اختیاری)</label>
        <input v-model="form.min_order_amount" type="number" min="0" class="form-control" />
      </div>

      <!-- Max Order Amount -->
      <div class="col-md-6">
        <label class="form-label">حداکثر سفارش (اختیاری)</label>
        <input v-model="form.max_order_amount" type="number" min="0" class="form-control" />
      </div>

      <div class="col-12">
        <button type="submit" class="btn btn-success" :disabled="loading">
          {{ loading ? 'درحال بروزرسانی' : 'بروزرسانی' }}
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute()
const router = useRouter()

const form = ref({
  city_id: null,
  cost: null,
  min_order_amount: null,
  max_order_amount: null,
})
let province_id = ref(null);
const provinces = ref([])
const cities = ref([])
const loading = ref(false)
const loadInitialData = async (setCities = false) => {
  try {
    const { data } = await axios.get('/provinces')
    provinces.value = data.data.map(p => ({ id: p.id, label: p.name, value: p.id, cities: p.cities }))
    if (setCities) {
      provinceChanged(province_id.value);
    }
  } catch (e) {
    toast.error('مشکلی در دریافت اطلاعات به وجود آمده')
  }
}
const fetchData = async () => {
  try {
    const { data } = await axios.get(`/shipping-ranges/${route.params.method}/${route.params.id}`)
    form.value = data.data
    province_id.value = data.data.province_id;
    await loadInitialData(true);
  } catch (e) {
    toast.error('Failed to load shipping range data')
  }
}
const updateForm = async () => {
  try {
    loading.value = true
    const formData = new FormData()
    for (let key in form.value) {
      if (form.value[key] && typeof form.value[key] !== Object) formData.append(key, form.value[key])
    }
    formData.append("_method", "PUT")
    formData.append("province_id", province_id.value)
    await axios.post(`/shipping-ranges/${route.params.method}/${route.params.id}`, formData)
    toast.success('Shipping range updated successfully')
  } catch (e) {
    toast.error('Error while updating shipping range')
  } finally {
    loading.value = false
  }
}
onMounted(async () => {
  await fetchData()
})
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
    if (province) {
      console.log(province);

      cities.value = province.cities.map(c => ({ id: c.id, label: c.name, value: c.id }))
      console.log(form.value.city_id);

    }

  } catch (e) {
    console.log(e);
    toast.error('مشکلی در لود شهرها به وجود آمد')
  }
}

</script>

<style scoped>
/* می‌تونی استایل دلخواه اضافه کنی */
</style>