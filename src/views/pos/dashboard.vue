<template>
  <div class="container mt-4" v-if="checkPermission(['pos_view'])">
    <!-- کارت‌های آماری -->
    <div v-if="summary" class="row g-3 mb-4">
      <div class="col-md-3">
        <div class="card bg-primary text-white">
          <div class="card-body">
            <h5 class="card-title">
              <i class="bi bi-wallet2"></i>
              فروش امروز
            </h5>
            <h2 class="mb-0">{{ formatPrice(summary.today_sales) }}</h2>
            <small>{{ summary.today_orders }} سفارش</small>
          </div>
        </div>
      </div>

      <div class="col-md-3">
        <div class="card bg-success text-white">
          <div class="card-body">
            <h5 class="card-title">
              <i class="bi bi-cash-coin"></i>
              موجودی صندوق
            </h5>
            <h2 class="mb-0">{{ formatPrice(summary.current_cash) }}</h2>
            <small>{{ summary.open_sessions }} شیفت باز</small>
          </div>
        </div>
      </div>

      <div class="col-md-3">
        <div class="card bg-info text-white">
          <div class="card-body">
            <h5 class="card-title">
              <i class="bi bi-cart-check"></i>
              سفارشات امروز
            </h5>
            <h2 class="mb-0">{{ summary.today_orders }}</h2>
            <small>میانگین: {{ formatPrice(summary.average_order) }}</small>
          </div>
        </div>
      </div>

      <div class="col-md-3">
        <div class="card bg-warning text-white">
          <div class="card-body">
            <h5 class="card-title">
              <i class="bi bi-arrow-left-right"></i>
              برگشتی امروز
            </h5>
            <h2 class="mb-0">{{ formatPrice(summary.today_refunds) }}</h2>
            <small>{{ summary.refund_count }} مورد</small>
          </div>
        </div>
      </div>
    </div>

    <!-- وضعیت شیفت فعلی -->
    <div class="card mb-4" v-if="currentSession">
      <div class="card-header d-flex justify-content-between align-items-center">
        <h5>
          <i class="bi bi-clock-history"></i>
          شیفت فعال شما
        </h5>
        <button class="btn btn-danger btn-sm" @click="closeSession">
          <i class="bi bi-x-circle"></i>
          بستن شیفت
        </button>
      </div>
      <div class="card-body">
        <div class="row">
          <div class="col-md-3">
            <strong>شروع شیفت:</strong>
            {{ formatDate(currentSession.opened_at) }}
          </div>
          <div class="col-md-3">
            <strong>موجودی اولیه:</strong>
            {{ formatPrice(currentSession.opening_balance) }}
          </div>
          <div class="col-md-3">
            <strong>موجودی فعلی:</strong>
            {{ formatPrice(currentSession.current_balance) }}
          </div>
          <div class="col-md-3">
            <strong>تعداد سفارشات:</strong>
            {{ currentSession.orders_count || 0 }}
          </div>
        </div>
      </div>
    </div>

    <div class="card mb-4" v-else>
      <div class="card-body text-center py-4">
        <h5 class="text-muted">
          <i class="bi bi-clock"></i>
          شما شیفت فعالی ندارید
        </h5>
        <button class="btn btn-success mt-2" @click="openSession">
          <i class="bi bi-play-circle"></i>
          باز کردن شیفت جدید
        </button>
      </div>
    </div>

    <!-- نمودار فروش -->
    <div class="card mb-4">
      <div class="card-header">
        <div class="d-flex justify-content-between align-items-center">
          <h5>
            <i class="bi bi-graph-up"></i>
            نمودار فروش
          </h5>
          <div class="d-flex gap-2">
            <select v-model="chartFilter" class="form-select form-select-sm" style="width: auto;">
              <option value="daily">روزانه</option>
              <option value="weekly">هفتگی</option>
              <option value="monthly">ماهانه</option>
            </select>
            <button class="btn btn-primary btn-sm" @click="loadChartData">
              <i class="bi bi-arrow-repeat"></i>
            </button>
          </div>
        </div>
      </div>
      <div class="card-body">
        <div v-if="chartLoading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>
        <canvas ref="chartCanvas" v-else></canvas>
      </div>
    </div>

    <!-- آخرین سفارشات -->
    <div class="card">
      <div class="card-header d-flex justify-content-between align-items-center">
        <h5>
          <i class="bi bi-list-ul"></i>
          آخرین سفارشات
        </h5>
        <router-link to="/pos/orders" class="btn btn-sm btn-outline-primary">
          مشاهده همه
        </router-link>
      </div>
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>مشتری</th>
                <th>مبلغ</th>
                <th>فروشنده</th>
                <th>تاریخ</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="order in orders.data" :key="order.id">
                <td>#{{ order.id }}</td>
                <td>{{ order.user?.full_name || 'مشتری' }}</td>
                <td>{{ formatPrice(order.total_amount) }}</td>
                <td>{{ order.cashier?.full_name || 'نامشخص' }}</td>
                <td>{{ formatDateTime(order.created_at) }}</td>
                <td>
                  <span class="badge" :class="{
                    'bg-success': order.status === 'paid',
                    'bg-danger': order.status === 'cancelled',
                    'bg-warning': order.status === 'returned',
                    'bg-secondary': order.status === 'pending'
                  }">
                    {{ getStatusLabel(order.status) }}
                  </span>
                </td>
                <td>
                  <router-link :to="`/pos/order/${order.id}`" class="btn btn-sm btn-info">
                    <i class="bi bi-eye"></i>
                  </router-link>
                </td>
              </tr>
            </tbody>
          </table>

          <b-pagination v-model="currentPage" :total-rows="orders.total" v-if="orders.last_page != 1"
            :per-page="orders.per_page" @Update:modelValue="changePage" align="center" class="mt-3" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from "vue";
import axios from "axios";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
import Swal from "sweetalert2";
import Chart from 'chart.js/auto';

const store = useAdmin();
const checkPermission = store.checkPermission;

const route = useRoute();
const router = useRouter();

// State
const loading = ref(false);
const chartLoading = ref(false);
const currentPage = ref(1);
const chartFilter = ref('daily');

const summary = ref({
  today_sales: 0,
  today_orders: 0,
  current_cash: 0,
  open_sessions: 0,
  average_order: 0,
  today_refunds: 0,
  refund_count: 0
});

const currentSession = ref(null);
const orders = ref({ data: [], total: 0, per_page: 20, last_page: 1 });
const chartData = ref([]);
const chartCanvas = ref(null);
let chartInstance = null;

// Methods
async function getDashboardData() {
  loading.value = true;
  try {
    const res = await axios.get('/pos/dashboard');


    summary.value = res.data.data.summary;
    currentSession.value = res.data.data.current_session;
    orders.value = res.data.data.recent_orders;
    currentPage.value = res.data.data.recent_orders.current_page || 1;
  } catch (err) {
    console.error('Error loading dashboard:', err);
    Swal.fire('خطا', 'مشکلی در بارگذاری داشبورد پیش آمد', 'error');
  } finally {
    loading.value = false;
  }
}

async function loadChartData() {
  chartLoading.value = true;
  try {
    const { data } = await axios.get('/pos/chart-data', {
      params: { filter: chartFilter.value }
    });
    chartData.value = data;
    await nextTick();
    renderChart();
  } catch (err) {
    console.error('Error loading chart:', err);
  } finally {
    chartLoading.value = false;
  }
}

function renderChart() {
  if (chartInstance) {
    chartInstance.destroy();
  }

  if (!chartCanvas.value) return;

  const labels = chartData.value.map(item => item.label);
  const sales = chartData.value.map(item => item.sales);
  const ordersCount = chartData.value.map(item => item.orders);

  chartInstance = new Chart(chartCanvas.value, {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [
        {
          label: 'فروش (تومان)',
          data: sales,
          backgroundColor: 'rgba(13, 110, 253, 0.6)',
          borderColor: 'rgba(13, 110, 253, 1)',
          borderWidth: 1,
          yAxisID: 'y'
        },
        {
          label: 'تعداد سفارشات',
          data: ordersCount,
          type: 'line',
          backgroundColor: 'rgba(40, 167, 69, 0.2)',
          borderColor: 'rgba(40, 167, 69, 1)',
          borderWidth: 2,
          yAxisID: 'y1'
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          position: 'top'
        }
      },
      scales: {
        y: {
          beginAtZero: true,
          ticks: {
            callback: function (value) {
              return value.toLocaleString('fa-IR');
            }
          }
        },
        y1: {
          beginAtZero: true,
          position: 'right',
          grid: {
            drawOnChartArea: false
          }
        }
      }
    }
  });
}

async function openSession() {
  const result = await Swal.fire({
    title: 'باز کردن شیفت جدید',
    html: `
      <div class="text-end">
        <label class="form-label">موجودی اولیه صندوق (تومان)</label>
        <input id="openingBalance" class="form-control" type="number" value="0">
        <label class="form-label mt-2">توضیحات (اختیاری)</label>
        <input id="notes" class="form-control" type="text" placeholder="توضیحات...">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'باز کردن شیفت',
    cancelButtonText: 'انصراف',
    preConfirm: () => {
      const openingBalance = document.getElementById('openingBalance').value;
      const notes = document.getElementById('notes').value;
      return { opening_balance: parseInt(openingBalance) || 0, notes };
    }
  });

  if (result.isConfirmed) {
    try {
      const { data } = await axios.post('/pos/cashier/open', result.value);
      Swal.fire('موفق', 'شیفت با موفقیت باز شد', 'success');
      await getDashboardData();
    } catch (err) {
      Swal.fire('خطا', err.response?.data?.message || 'مشکلی در باز کردن شیفت پیش آمد', 'error');
    }
  }
}

async function closeSession() {
  const result = await Swal.fire({
    title: 'بستن شیفت',
    html: `
      <div class="text-end">
        <label class="form-label">موجودی نهایی صندوق (تومان)</label>
        <input id="closingBalance" class="form-control" type="number" placeholder="موجودی نهایی را وارد کنید">
        <label class="form-label mt-2">توضیحات (اختیاری)</label>
        <input id="notes" class="form-control" type="text" placeholder="توضیحات...">
      </div>
    `,
    showCancelButton: true,
    confirmButtonText: 'بستن شیفت',
    cancelButtonText: 'انصراف',
    preConfirm: () => {
      const closingBalance = document.getElementById('closingBalance').value;
      if (!closingBalance) {
        Swal.showValidationMessage('لطفاً موجودی نهایی را وارد کنید');
        return false;
      }
      const notes = document.getElementById('notes').value;
      return { closing_balance: parseInt(closingBalance), notes };
    }
  });

  if (result.isConfirmed) {
    try {
      const sessionId = currentSession.value.id;
      await axios.post(`/pos/cashier/close/${sessionId}`, result.value);
      Swal.fire('موفق', 'شیفت با موفقیت بسته شد', 'success');
      await getDashboardData();
    } catch (err) {
      Swal.fire('خطا', err.response?.data?.message || 'مشکلی در بستن شیفت پیش آمد', 'error');
    }
  }
}

function changePage(page) {
  if (page) {
    router.replace({ name: route.name, query: { page: page } });
    getDashboardData();
  }
}

function formatPrice(price) {
  return new Intl.NumberFormat('fa-IR').format(price || 0) + ' تومان';
}

function formatDate(date) {
  if (!date) return '-';
  return new Date(date).toLocaleDateString('fa-IR');
}

function formatDateTime(date) {
  if (!date) return '-';
  return new Date(date).toLocaleString('fa-IR');
}

function getPaymentMethodLabel(method) {
  const labels = {
    cash: 'نقدی',
    card: 'کارت',
    transfer: 'انتقال'
  };
  return labels[method] || method;
}

function getStatusLabel(status) {
  const labels = {
    paid: 'پرداخت شده',
    pending: 'در انتظار',
    cancelled: 'لغو شده',
    returned: 'برگشت خورده'
  };
  return labels[status] || status;
}

onMounted(() => {
  getDashboardData();
  loadChartData();
});
</script>

<style scoped>
.card {
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.card-header {
  background: #f8f9fa;
  border-bottom: 2px solid #e9ecef;
}

.table th {
  background: #f8f9fa;
}

canvas {
  height: 300px !important;
  width: 100% !important;
}
</style>