<template>
  <div class="container mt-4" v-if="checkPermission(['campaign_view'])">
    <h3 class="p-2"><i class="bi bi-bar-chart"></i> گزارش کمپین: {{ report.campaign?.name }}</h3>

    <div class="row g-3 mb-3">
      <div class="col-6 col-md-3" v-for="s in cards" :key="s.label">
        <div class="card text-center"><div class="card-body">
          <div class="text-muted small">{{ s.label }}</div>
          <div class="fs-5 fw-bold">{{ Number(s.value || 0).toLocaleString('fa-IR') }}</div>
        </div></div>
      </div>
    </div>

    <div class="card mb-3">
      <div class="card-header">عملکرد هر پله</div>
      <div class="card-body table-responsive">
        <table class="table table-bordered mb-0">
          <thead><tr><th>بازه (تومان)</th><th>مزایا</th><th>تعداد سفارش</th><th>جمع فروش</th><th>جمع تخفیف</th></tr></thead>
          <tbody>
            <tr v-for="t in report.tiers" :key="t.id">
              <td>{{ t.min_amount.toLocaleString('fa-IR') }} تا {{ t.max_amount ? t.max_amount.toLocaleString('fa-IR') : '∞' }}</td>
              <td>{{ t.free_shipping ? 'ارسال رایگان' : '' }} {{ t.discount_percent > 0 ? `${t.discount_percent}٪ تخفیف` : '' }}</td>
              <td>{{ t.orders_count }}</td>
              <td>{{ t.total_sales.toLocaleString('fa-IR') }}</td>
              <td>{{ t.total_discount.toLocaleString('fa-IR') }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div class="card">
      <div class="card-header d-flex gap-2 flex-wrap">
        <input v-model="search" @input="onSearch" class="form-control w-auto" placeholder="شماره سفارش / نام / موبایل" />
        <select v-model="tierId" @change="load(1)" class="form-select w-auto">
          <option value="">همه پله‌ها</option>
          <option v-for="t in report.tiers" :key="t.id" :value="t.id">
            {{ t.min_amount.toLocaleString('fa-IR') }} - {{ t.max_amount ? t.max_amount.toLocaleString('fa-IR') : '∞' }}
          </option>
        </select>
      </div>
      <div class="card-body table-responsive">
        <table class="table table-bordered table-striped mb-0">
          <thead><tr><th>سفارش</th><th>مشتری</th><th>جمع کالا</th><th>تخفیف کمپین</th><th>ارسال رایگان</th><th>مبلغ نهایی</th><th>تاریخ</th></tr></thead>
          <tbody>
            <tr v-for="o in report.orders?.data" :key="o.id">
              <td>#{{ o.order_id }}</td>
              <td>{{ o.order?.user?.full_name }} <small class="text-muted">{{ o.order?.user?.mobile }}</small></td>
              <td>{{ o.order_subtotal.toLocaleString('fa-IR') }}</td>
              <td>{{ o.discount_amount.toLocaleString('fa-IR') }}</td>
              <td>{{ o.free_shipping ? '✓' : '-' }}</td>
              <td>{{ o.order_total.toLocaleString('fa-IR') }}</td>
              <td>{{ moment(o.created_at).format('jYYYY/jMM/jDD HH:mm') }}</td>
            </tr>
            <tr v-if="!report.orders?.data?.length"><td colspan="7" class="text-center text-muted">سفارشی یافت نشد</td></tr>
          </tbody>
        </table>
        <b-pagination v-if="report.orders?.last_page > 1" v-model="page" :total-rows="report.orders.total"
          :per-page="report.orders.per_page" align="center" class="mt-3" @update:modelValue="load" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'
import moment from 'moment-jalaali'
import { useAdmin } from '@/stores/modules/admin'

const checkPermission = useAdmin().checkPermission
const route = useRoute()
const report = ref({})
const page = ref(1)
const search = ref('')
const tierId = ref('')
let debounce = null

const cards = computed(() => [
  { label: 'تعداد سفارش', value: report.value.stats?.orders_count },
  { label: 'جمع فروش (تومان)', value: report.value.stats?.total_sales },
  { label: 'جمع تخفیف داده‌شده', value: report.value.stats?.total_discount },
  { label: 'سفارش با ارسال رایگان', value: report.value.stats?.free_shipping_count },
])

const load = async (p = 1) => {
  page.value = p
  const { data } = await axios.get(`/campaigns/${route.params.id}/report`, {
    params: { page: p, search: search.value, tier_id: tierId.value || undefined },
  })
  report.value = data
}
const onSearch = () => { clearTimeout(debounce); debounce = setTimeout(() => load(1), 400) }

onMounted(() => load())
</script>