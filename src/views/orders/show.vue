<template>
  <b-container fluid class="py-4" v-if="checkPermission(['order_view'])">
    <!-- هدر -->
    <b-row class="mb-4">
      <b-col>
        <div class="d-flex align-items-center justify-content-between">
          <div>
            <h4 class="mb-0 text-primary">
              <i class="fas fa-shopping-bag me-2"></i>
              جزئیات سفارش
              <span class="badge bg-primary ms-2">#{{ order.id }}</span>
            </h4>
            <small class="text-muted">تاریخ ثبت: {{ formatDate(order.created_at) }}</small>
          </div>
          <div>
            <b-button variant="outline-secondary" size="sm" @click="$router.back()">
              <i class="fas fa-arrow-right me-1"></i> بازگشت
            </b-button>
          </div>
        </div>
      </b-col>
    </b-row>

    <b-row>
      <!-- ستون اول: اطلاعات مشتری و آدرس -->
      <b-col cols="12" lg="4">
        <b-card class="shadow-sm h-100" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center">
              <i class="fas fa-user-circle me-2 text-primary"></i>
              <span class="fw-bold">اطلاعات مشتری</span>
            </div>
          </template>
          
          <div class="info-item">
            <span class="info-label">نام مشتری</span>
            <span class="info-value">{{ order.user?.full_name || 'نامشخص' }}</span>
          </div>
          
          <template v-if="order.address">
            <div class="info-item">
              <span class="info-label">گیرنده</span>
              <span class="info-value">{{ order.address.receiver_name }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">آدرس</span>
              <span class="info-value">
                {{ order.address.province?.name }} - {{ order.address.city?.name }} - 
                {{ order.address.address_line }}
              </span>
            </div>
            <div class="info-item">
              <span class="info-label">کدپستی</span>
              <span class="info-value">{{ order.address.postal_code }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">شماره تماس</span>
              <span class="info-value">{{ order.address.phone }}</span>
            </div>
          </template>
          
          <div class="info-item">
            <span class="info-label">روش حمل</span>
            <span class="info-value">
              <i class="fas fa-truck me-1"></i>
              {{ order.shipping?.title || 'نامشخص' }}
            </span>
          </div>
          <div class="info-item">
            <span class="info-label">روش پرداخت</span>
            <span class="info-value">
              <i class="fas fa-credit-card me-1"></i>
              {{ paymentMethods[order.payment_method] || 'نامشخص' }}
            </span>
          </div>
        </b-card>
      </b-col>

      <!-- ستون دوم: خلاصه مالی -->
      <b-col cols="12" lg="4">
        <b-card class="shadow-sm h-100" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center">
              <i class="fas fa-chart-pie me-2 text-success"></i>
              <span class="fw-bold">خلاصه مالی</span>
            </div>
          </template>

          <div class="info-item">
            <span class="info-label">جمع جزء</span>
            <span class="info-value">{{ formatPrice(order.subtotal) }}</span>
          </div>
          
          <div v-if="order.discount_amount > 0" class="info-item text-success">
            <span class="info-label">
              <i class="fas fa-tag me-1"></i> تخفیف
            </span>
            <span class="info-value">-{{ formatPrice(order.discount_amount) }}</span>
          </div>
          
          <div class="info-item">
            <span class="info-label">هزینه ارسال</span>
            <span class="info-value">{{ formatPrice(order.shipping_cost) }}</span>
          </div>

          <hr class="my-2">

          <div class="info-item total-item">
            <span class="info-label fw-bold">جمع کل</span>
            <span class="info-value total-price">{{ formatPrice(order.total) }}</span>
          </div>

          <!-- جمع کل با احتساب سفارش‌های فرزند -->
          <div v-if="order.child_orders?.length" class="info-item child-total">
            <span class="info-label text-muted">+ سفارش‌های مرتبط</span>
            <span class="info-value text-muted">{{ formatPrice(childOrdersTotal) }}</span>
          </div>

          <div v-if="order.child_orders?.length" class="info-item grand-total">
            <span class="info-label fw-bold">جمع نهایی</span>
            <span class="info-value grand-total-price">{{ formatPrice(grandTotal) }}</span>
          </div>

          <div v-if="order.child_orders?.length" class="text-muted small mt-2">
            <i class="fas fa-info-circle me-1"></i>
            شامل {{ order.child_orders.length }} سفارش مرتبط
          </div>
        </b-card>
      </b-col>

      <!-- ستون سوم: وضعیت‌ها و عملیات -->
      <b-col cols="12" lg="4">
        <b-card class="shadow-sm h-100" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center">
              <i class="fas fa-cogs me-2 text-warning"></i>
              <span class="fw-bold">وضعیت‌ها و عملیات</span>
            </div>
          </template>

          <b-form-group label="وضعیت سفارش" label-for="order-status">
            <b-form-select 
              id="order-status" 
              v-model="order.status" 
              :options="orderStatusOptions"
              :class="getStatusClass(order.status)"
            />
          </b-form-group>

          <div v-if="order.status == 'reserved'" class="alert alert-info mt-2">
            <i class="fas fa-clock me-1"></i>
            تاریخ اتمام رزرو: {{ formatDate(order.reserved_until) }}
          </div>

          <b-form-group label="وضعیت پرداخت" label-for="payment-status" class="mt-3">
            <b-form-select 
              id="payment-status" 
              disabled 
              v-model="order.payment_status"
              :options="paymentStatusOptions"
              :class="getPaymentStatusClass(order.payment_status)"
            />
          </b-form-group>

          <b-button 
            variant="primary" 
            block 
            class="mt-3"
            @click="updateOrder"
            :disabled="updating"
          >
            <i v-if="updating" class="fas fa-spinner fa-spin me-1"></i>
            <i v-else class="fas fa-save me-1"></i>
            ذخیره تغییرات
          </b-button>

          <b-button 
            variant="outline-info" 
            block 
            class="mt-2"
            @click="printOrder"
          >
            <i class="fas fa-print me-1"></i>
            چاپ فاکتور
          </b-button>
        </b-card>
      </b-col>
    </b-row>

    <!-- جدول آیتم‌های سفارش -->
    <b-row class="mt-4">
      <b-col cols="12">
        <b-card class="shadow-sm" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center justify-content-between">
              <div>
                <i class="fas fa-list me-2 text-primary"></i>
                <span class="fw-bold">آیتم‌های سفارش</span>
                <span class="badge bg-secondary ms-2">{{ totalItems }} آیتم</span>
              </div>
              <div v-if="order.child_orders?.length" class="text-muted small">
                <i class="fas fa-copy me-1"></i>
                شامل {{ order.child_orders.length }} سفارش فرزند
              </div>
            </div>
          </template>

          <div class="table-responsive">
            <table class="table table-bordered table-hover align-middle text-center">
              <thead class="table-light">
                <tr>
                  <th style="width: 50px;">#</th>
                  <th style="width: 40px;">تصویر</th>
                  <th class="text-start">محصول</th>
                  <th style="width: 80px;">تعداد</th>
                  <th style="width: 150px;">قیمت واحد</th>
                  <th style="width: 150px;">مجموع</th>
                </tr>
              </thead>
              <tbody>
                <!-- آیتم‌های سفارش اصلی -->
                <tr v-for="(item, index) in order.items" :key="item.id" class="main-item">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <img 
                      width="50" 
                      height="50" 
                      class="rounded"
                      :src="getProductImage(item)" 
                      :alt="getProductTitle(item)"
                      @error="handleImageError"
                    >
                  </td>
                  <td class="text-start">
                    <span class="fw-bold">{{ getProductTitle(item) }}</span>
                    <div class="text-muted small">{{ getVariantText(item) }}</div>
                  </td>
                  <td>{{ item.quantity }}</td>
                  <td>{{ formatPrice(item.price) }}</td>
                  <td class="fw-bold">{{ formatPrice(item.quantity * item.price) }}</td>
                </tr>

                <!-- آیتم‌های سفارش‌های فرزند -->
                <template v-for="(childOrder, childIndex) in order.child_orders" :key="childOrder.id">
                  <tr class="child-header">
                    <td colspan="6" class="bg-light text-primary">
                      <i class="fas fa-copy me-1"></i>
                      سفارش فرزند #{{ childOrder.id }}
                      <span class="badge bg-secondary ms-2">{{ getStatusText(childOrder.status) }}</span>
                      <span class="badge ms-1" :class="getPaymentStatusClass(childOrder.payment_status)">
                        {{ getPaymentStatusText(childOrder.payment_status) }}
                      </span>
                      <span class="float-end text-muted small">
                        {{ formatDate(childOrder.created_at) }}
                      </span>
                    </td>
                  </tr>
                  <tr 
                    v-for="(item, itemIndex) in childOrder.items" 
                    :key="item.id"
                    class="child-item"
                  >
                    <td>{{ itemIndex + 1 }}</td>
                    <td>
                      <img 
                        width="50" 
                        height="50" 
                        class="rounded"
                        :src="getProductImage(item)" 
                        :alt="getProductTitle(item)"
                        @error="handleImageError"
                      >
                    </td>
                    <td class="text-start">
                      <span class="fw-bold">{{ getProductTitle(item) }}</span>
                      <div class="text-muted small">{{ getVariantText(item) }}</div>
                    </td>
                    <td>{{ item.quantity }}</td>
                    <td>{{ formatPrice(item.price) }}</td>
                    <td class="fw-bold">{{ formatPrice(item.quantity * item.price) }}</td>
                  </tr>
                  <tr class="child-summary">
                    <td colspan="4" class="text-end fw-bold text-muted">
                      جمع سفارش فرزند #{{ childOrder.id }}
                    </td>
                    <td colspan="2" class="fw-bold text-primary">
                      {{ formatPrice(childOrder.total) }}
                    </td>
                  </tr>
                </template>

                <!-- ردیف جمع کل -->
                <tr class="table-success fw-bold">
                  <td colspan="5" class="text-end">جمع کل سفارشات</td>
                  <td>{{ formatPrice(grandTotal) }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- خلاصه سریع سفارش‌های فرزند -->
          <div v-if="order.child_orders?.length" class="mt-3 p-3 bg-light rounded">
            <div class="d-flex flex-wrap gap-3">
              <div v-for="child in order.child_orders" :key="child.id" class="child-order-summary">
                <span class="fw-bold">سفارش #{{ child.id }}</span>
                <span class="badge ms-1" :class="getStatusClass(child.status)">
                  {{ getStatusText(child.status) }}
                </span>
                <span class="text-muted ms-2">
                  {{ child.items?.length || 0 }} آیتم
                </span>
                <span class="fw-bold text-primary ms-2">
                  {{ formatPrice(child.total) }}
                </span>
              </div>
            </div>
          </div>
        </b-card>
      </b-col>
    </b-row>
  </b-container>
</template>

<script setup>
import { ref, computed, onMounted } from "vue"
import axios from "axios"
import { toast } from "vue3-toastify"
import "vue3-toastify/dist/index.css"
import { useRoute, useRouter } from "vue-router"
import { useAdmin } from '@/stores/modules/admin'

const store = useAdmin()
const checkPermission = store.checkPermission
const route = useRoute()
const router = useRouter()

const order = ref({ items: [], child_orders: [] })
const updating = ref(false)
const baseImageAddress = ''

// محاسبه جمع سفارش‌های فرزند
const childOrdersTotal = computed(() => {
  if (!order.value.child_orders?.length) return 0
  return order.value.child_orders.reduce((sum, child) => sum + (child.total || 0), 0)
})

// محاسبه جمع نهایی
const grandTotal = computed(() => {
  const mainTotal = order.value.total || 0
  return mainTotal + childOrdersTotal.value
})

// تعداد کل آیتم‌ها
const totalItems = computed(() => {
  let count = order.value.items?.length || 0
  if (order.value.child_orders?.length) {
    order.value.child_orders.forEach(child => {
      count += child.items?.length || 0
    })
  }
  return count
})

const orderStatusOptions = [
  { value: "pending", text: "در انتظار" },
  { value: "reserved", text: "رزرو شده" },
  { value: "processing", text: "در حال پردازش" },
  { value: "shipped", text: "ارسال شده" },
  { value: "completed", text: "تکمیل شده" },
  { value: "canceled", text: "لغو شده" },
  { value: "returned", text: "مرجوع شده" },
]

const paymentMethods = {
  online: "پرداخت آنلاین",
  wallet: "کیف پول",
  cod: "پرداخت در محل",
}

const paymentStatusOptions = [
  { value: "pending", text: "در انتظار پرداخت" },
  { value: "paid", text: "پرداخت شده" },
  { value: "failed", text: "ناموفق" },
  { value: "refunded", text: "برگشت داده شده" },
]

const fetchOrder = async () => {
  try {
    const res = await axios.get(`/orders/${route.params.id}`)
    order.value = res.data.data.order
  } catch (e) {
    toast.error("خطا در گرفتن اطلاعات سفارش")
  }
}

const updateOrder = async () => {
  updating.value = true
  try {
    let fd = new FormData()
    fd.append("status", order.value.status)
    await axios.post(`/orders-change-status/${order.value.id}`, fd)
    toast.success("سفارش با موفقیت بروزرسانی شد")
  } catch (e) {
    toast.error("خطا در بروزرسانی سفارش")
  } finally {
    updating.value = false
  }
}

const getStatusText = (status) => {
  const map = {
    pending: 'در انتظار',
    reserved: 'رزرو شده',
    processing: 'در حال پردازش',
    shipped: 'ارسال شده',
    completed: 'تکمیل شده',
    canceled: 'لغو شده',
    returned: 'مرجوع شده'
  }
  return map[status] || status
}

const getStatusClass = (status) => {
  const map = {
    pending: 'bg-warning text-dark',
    reserved: 'bg-info text-white',
    processing: 'bg-primary text-white',
    shipped: 'bg-purple text-white',
    completed: 'bg-success text-white',
    canceled: 'bg-danger text-white',
    returned: 'bg-secondary text-white'
  }
  return map[status] || ''
}

const getPaymentStatusText = (status) => {
  const map = {
    pending: 'در انتظار پرداخت',
    paid: 'پرداخت شده',
    failed: 'ناموفق',
    refunded: 'برگشت داده شده'
  }
  return map[status] || status
}

const getPaymentStatusClass = (status) => {
  const map = {
    pending: 'bg-warning text-dark',
    paid: 'bg-success text-white',
    failed: 'bg-danger text-white',
    refunded: 'bg-secondary text-white'
  }
  return map[status] || ''
}

const formatDate = (date) => {
  if (!date) return '---'
  try {
    return new Date(date).toLocaleDateString('fa-IR', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  } catch {
    return date
  }
}

const formatPrice = (val) => {
  if (!val && val !== 0) return "۰"
  return new Intl.NumberFormat("fa-IR").format(val) + " تومان"
}

const getProductTitle = (item) => {
  let title = item.product?.title || 'نامشخص'
  if (item.variant?.values?.length) {
    title += ` [${item.variant.values.map(v => decodeURIComponent(v.value)).join(' - ')}]`
  }
  return title
}

const getVariantText = (item) => {
  if (!item.variant?.values?.length) return ''
  return item.variant.values
    .map(v => {
      const name = v.attribute_id === 1 ? 'سایز' : v.attribute_id === 2 ? 'رنگ' : 'ویژگی'
      return `${name}: ${decodeURIComponent(v.value)}`
    })
    .join(' - ')
}

const getProductImage = (item) => {
  if (!item.product) return ''
  if (item.product.main_image && item.product.main_image.includes('http'))
    return item.product.main_image
  return baseImageAddress + item.product.main_image
}

const handleImageError = (e) => {
  e.target.src = '/images/no-image.png'
}

const printOrder = () => {
  window.print()
}

onMounted(fetchOrder)
</script>

<style scoped>
.info-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #f0f0f0;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  color: #6c757d;
  font-size: 0.9rem;
}

.info-value {
  font-weight: 500;
  text-align: left;
  direction: ltr;
}

.total-item {
  padding: 12px 0;
  border-bottom: 2px solid #dee2e6;
}

.total-price {
  font-size: 1.2rem;
  color: #28a745;
}

.grand-total {
  padding: 12px 0;
  border-top: 2px solid #007bff;
  background-color: #f8f9ff;
  margin-top: 4px;
  border-radius: 4px;
}

.grand-total-price {
  font-size: 1.3rem;
  color: #007bff;
}

.child-total {
  padding: 6px 0;
}

.child-header td {
  background-color: #f8f9fa !important;
  font-weight: 600;
}

.child-item {
  background-color: #fafbfc;
}

.child-item td:first-child {
  color: #6c757d;
}

.child-summary td {
  background-color: #f0f2f5 !important;
  border-top: 2px solid #dee2e6;
}

.main-item {
  background-color: #ffffff;
}

.table-hover tbody tr:hover {
  background-color: rgba(0, 123, 255, 0.05);
}

.child-order-summary {
  background: white;
  padding: 6px 12px;
  border-radius: 6px;
  border: 1px solid #dee2e6;
}

.bg-purple {
  background-color: #6f42c1;
}

@media print {
  .btn,
  .alert {
    display: none !important;
  }
  
  .card {
    border: 1px solid #ddd !important;
    box-shadow: none !important;
  }
  
  .table {
    font-size: 12px !important;
  }
}
</style>