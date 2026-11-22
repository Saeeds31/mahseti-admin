<template>
  <div class="container mt-4" v-if="checkPermission([isEdit ? 'campaign_update' : 'campaign_create'])">
    <h3 class="p-2"><i class="bi bi-megaphone"></i> {{ isEdit ? 'ویرایش کمپین' : 'ایجاد کمپین' }}</h3>

    <div v-if="hasOrders" class="alert alert-warning">
      این کمپین سفارش ثبت‌شده دارد؛ تغییر پله‌ها ذخیره نخواهد شد (نام، تاریخ پایان و وضعیت قابل ویرایش است).
    </div>

    <form @submit.prevent="submit" class="bg-light p-3 rounded row g-3">
      <div class="col-md-12">
        <label class="form-label">نام کمپین (مثلاً بلک فرایدی)</label>
        <input v-model="form.name" class="form-control" required maxlength="255" />
      </div>

      <div class="col-md-6">
        <label class="form-label">شروع</label>
        <date-picker type="datetime" display-format="jYYYY/jMM/jDD HH:mm" format="YYYY-MM-DD HH:mm:ss"
          v-model="form.starts_at" />
      </div>
      <div class="col-md-6">
        <label class="form-label">پایان</label>
        <date-picker type="datetime" display-format="jYYYY/jMM/jDD HH:mm" format="YYYY-MM-DD HH:mm:ss"
          v-model="form.ends_at" />
      </div>

      <div class="col-12 form-check form-switch ms-3">
        <input id="active" type="checkbox" class="form-check-input" v-model="form.is_active" />
        <label for="active" class="form-check-label">فعال</label>
      </div>

      <div class="col-12">
        <h5>پله‌های کمپین <small class="text-muted">(بر اساس جمع سبد، بدون هزینه ارسال)</small></h5>

        <div v-for="(t, i) in form.tiers" :key="i" class="border rounded p-3 mb-2 bg-white">
          <div class="row g-2 align-items-end">
            <div class="col-md-3">
              <label class="form-label">از (تومان)</label>
              <input type="number" class="form-control" :value="t.min_amount" disabled />
            </div>
            <div class="col-md-3">
              <label class="form-label">تا (تومان)</label>
              <input v-if="i < form.tiers.length - 1" type="number" min="1" class="form-control"
                v-model.number="t.max_amount" @input="relink" required />
              <input v-else class="form-control" value="بدون سقف" disabled />
            </div>
            <div class="col-md-2">
              <label class="form-label">درصد تخفیف</label>
              <input type="number" min="0" max="100" step="0.5" class="form-control" v-model.number="t.discount_percent" />
            </div>
            <div class="col-md-2">
              <div class="form-check mt-4">
                <input :id="`fs${i}`" type="checkbox" class="form-check-input" v-model="t.free_shipping" />
                <label :for="`fs${i}`" class="form-check-label">ارسال رایگان</label>
              </div>
            </div>
            <div class="col-md-2 text-end">
              <button type="button" class="btn btn-sm btn-outline-danger" v-if="form.tiers.length > 1"
                @click="removeTier(i)"><i class="bi bi-trash3"></i></button>
            </div>
          </div>
          <small class="text-muted">
            {{ Number(t.min_amount).toLocaleString('fa-IR') }}
            تا {{ t.max_amount ? Number(t.max_amount).toLocaleString('fa-IR') : '∞' }} تومان
          </small>
        </div>

        <button type="button" class="btn btn-sm btn-outline-primary" @click="addTier">
          <i class="bi bi-plus"></i> افزودن پله
        </button>
        <div v-if="errors.tiers" class="text-danger mt-2">{{ errors.tiers[0] }}</div>
      </div>

      <div class="col-12">
        <button class="btn btn-success" :disabled="loading">
          <i class="bi bi-save2"></i>
          <span class="mx-2">{{ loading ? 'در حال ذخیره...' : 'ذخیره' }}</span>
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import { useAdmin } from '@/stores/modules/admin'

const checkPermission = useAdmin().checkPermission
const route = useRoute()
const router = useRouter()
const isEdit = computed(() => !!route.params.id)
const loading = ref(false)
const hasOrders = ref(false)
const errors = reactive({})

const form = ref({
  name: '', starts_at: '', ends_at: '', is_active: true,
  tiers: [
    { min_amount: 0, max_amount: 2000000, free_shipping: true, discount_percent: 0 },
    { min_amount: 2000000, max_amount: 4000000, free_shipping: true, discount_percent: 5 },
    { min_amount: 4000000, max_amount: null, free_shipping: true, discount_percent: 10 },
  ],
})

// شروع هر پله = سقف پله قبلی
const relink = () => {
  form.value.tiers.forEach((t, i) => {
    t.min_amount = i === 0 ? 0 : form.value.tiers[i - 1].max_amount ?? 0
    if (i === form.value.tiers.length - 1) t.max_amount = null
  })
}
const addTier = () => {
  const tiers = form.value.tiers
  const last = tiers[tiers.length - 1]
  last.max_amount = last.min_amount + 2000000
  tiers.push({ min_amount: last.max_amount, max_amount: null, free_shipping: false, discount_percent: 0 })
}
const removeTier = (i) => { form.value.tiers.splice(i, 1); relink() }

const fetchData = async () => {
  if (!isEdit.value) return
  const { data } = await axios.get(`/campaigns/${route.params.id}`)
  const c = data.data
  hasOrders.value = c.orders_count > 0
  form.value = {
    name: c.name,
    starts_at: c.starts_at,
    ends_at: c.ends_at,
    is_active: !!c.is_active,
    tiers: c.tiers.map(t => ({
      min_amount: t.min_amount, max_amount: t.max_amount,
      free_shipping: !!t.free_shipping, discount_percent: t.discount_percent,
    })),
  }
}

const submit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  loading.value = true
  try {
    relink()
    const url = isEdit.value ? `/campaigns/${route.params.id}` : '/campaigns'
    const { data } = await (isEdit.value ? axios.put(url, form.value) : axios.post(url, form.value))
    toast.success(data.message)
    router.push('/campaigns')
  } catch (e) {
    Object.assign(errors, e.response?.data?.errors || {})
    toast.error(e.response?.data?.message || 'خطا در ذخیره کمپین')
  } finally { loading.value = false }
}

onMounted(fetchData)
</script>