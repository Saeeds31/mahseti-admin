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
              <span class="badge bg-primary ms-2">#{{ order?.id || '---' }}</span>
            </h4>
            <small class="text-muted">تاریخ ثبت: {{ formatDate(order?.created_at) }}</small>
          </div>
          <div class="d-flex gap-2">
            <b-button variant="outline-secondary" size="sm" @click="$router.back()">
              <i class="bi bi-arrow-right me-1"></i> بازگشت
            </b-button>
          </div>
        </div>
      </b-col>
    </b-row>

    <b-row>
      <b-col cols="12" lg="8">
        <b-card class="shadow-sm h-100" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center justify-content-between">
              <div>
                <i class="fas fa-user-circle me-2 text-primary"></i>
                <span class="fw-bold">اطلاعات مشتری</span>
              </div>
              <span class="badge bg-secondary">{{ order?.user?.full_name || 'نامشخص' }}</span>
            </div>
          </template>

          <!-- آدرس با دکمه ویرایش -->
          <div class="info-item">
            <span class="info-label">آدرس</span>
            <span class="info-value" style="width: 65%;">
              {{ getAddressText(order?.address) }}
              <b-button v-if="canEdit" variant="outline-primary" size="sm" class="ms-1" @click="showAddressModalFunc()"
                title="ویرایش آدرس">
                <i class="bi bi-pencil"></i>
              </b-button>
            </span>
          </div>

          <!-- روش حمل با دکمه ویرایش -->
          <div class="info-item">
            <span class="info-label">روش حمل</span>
            <span class="info-value" style="width: 65%;">
              <i class="fas fa-truck me-1"></i>
              {{ order?.shipping?.title || 'نامشخص' }}
              <span class="text-muted ms-1">({{ formatPrice(order?.shipping_cost) }})</span>
              <b-button v-if="canEdit" variant="outline-primary" size="sm" class="ms-1" @click="showShippingModalFunc()"
                title="ویرایش روش حمل">
                <i class="bi bi-pencil"></i>
              </b-button>
            </span>
          </div>

          <!-- نوع سفارش (عادی/رزرو) -->
          <div class="info-item" v-if="canEdit">
            <span class="info-label">نوع سفارش</span>
            <span class="info-value" style="width: 65%;">
              <b-form-select v-model="order.reservation_type" :options="reservationTypeOptions" size="sm"
                :disabled="loadingReservation" />
              <b-button variant="outline-primary" size="sm" class="ms-1" @click="changeReservationType"
                :disabled="loadingReservation">
                <i v-if="loadingReservation" class="bi bi-three-dots"></i>
                <i v-else class="bi bi-save2"></i>
              </b-button>
            </span>
          </div>
          <div class="info-item" v-else>
            <span class="info-label">نوع سفارش</span>
            <span class="info-value">
              {{ getReservationTypeText(order?.reservation_type) }}
            </span>
          </div>

          <div class="info-item">
            <span class="info-label">روش پرداخت</span>
            <span class="info-value">
              <i class="fas fa-credit-card me-1"></i>
              {{ paymentMethods[order?.payment_method] || 'نامشخص' }}
            </span>
          </div>

          <div class="info-item">
            <span class="info-label">وضعیت</span>
            <span class="info-value">
              <span :class="getStatusClass(order?.status)">{{ getStatusText(order?.status) }}</span>
            </span>
          </div>

          <div v-if="order?.status == 'reserved'" class="alert alert-info mt-2">
            <i class="fas fa-clock me-1"></i>
            تاریخ اتمام رزرو: {{ formatDate(order?.reserved_until) }}
          </div>
        </b-card>
      </b-col>



      <b-col cols="12" lg="4">
        <b-card class="shadow-sm h-100" header-tag="header">
          <template #header>
            <div class="d-flex align-items-center">
              <i class="fas fa-cogs me-2 text-warning"></i>
              <span class="fw-bold">عملیات</span>
            </div>
          </template>

          <b-form-group label="تغییر وضعیت" label-for="order-status">
            <b-form-select id="order-status" v-model="order.status" :options="orderStatusOptions"
              :class="getStatusClass(order?.status)" @change="updateOrderStatus" :disabled="updating" />
          </b-form-group>

          <div class="mt-3 d-grid gap-2">
            <b-button variant="primary" @click="updateOrderStatus" :disabled="updating || !order?.id">
              <i v-if="updating" class="fas fa-spinner fa-spin me-1"></i>
              <i v-else class="fas fa-save me-1"></i>
              ذخیره تغییرات
            </b-button>

            <router-link variant="info" class="btn btn-info" :to="`/orders/print?ids=${order.id}&type=full`"
              target="_blank">
              <i class="fas fa-print me-1"></i>
              چاپ فاکتور
            </router-link>
          </div>
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
                <span class="badge bg-secondary ms-2">{{ order?.items?.length || 0 }} آیتم</span>
              </div>
              <div>
                <b-button v-if="canEdit" variant="outline-primary" size="sm" @click="showItemModal = true">
                  <i class="fas fa-plus me-1"></i>
                  افزودن محصول
                </b-button>
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
                  <th style="width: 120px;">تعداد</th>
                  <th style="width: 150px;">قیمت واحد</th>
                  <th style="width: 150px;">مجموع</th>
                  <th v-if="canEdit" style="width: 150px;">عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in order?.items || []" :key="item.id">
                  <td>{{ index + 1 }}</td>
                  <td>
                    <img width="50" height="50" class="rounded" :src="getProductImage(item)"
                      :alt="getProductTitle(item)" />
                  </td>
                  <td class="text-start">
                    <span class="fw-bold">{{ getProductTitle(item) }}</span>
                    <div class="text-muted small">{{ getVariantText(item) }}</div>
                  </td>
                  <td>
                    <template v-if="editingItem === item.id">
                      <input type="number" class="form-control form-control-sm text-center"
                        v-model.number="itemEditForm.quantity" min="1" />
                    </template>
                    <template v-else>
                      {{ item.quantity }}
                    </template>
                  </td>
                  <td>
                    <template v-if="editingItem === item.id">
                      <input type="number" class="form-control form-control-sm text-center"
                        v-model.number="itemEditForm.price" min="0" />
                    </template>
                    <template v-else>
                      {{ formatPrice(item.price) }}
                    </template>
                  </td>
                  <td class="fw-bold">{{ formatPrice((item.price || 0) * (item.quantity || 0)) }}</td>
                  <td v-if="canEdit">
                    <template v-if="editingItem === item.id">
                      <b-button variant="success" size="sm" @click="saveItemEdit(item)" :disabled="itemSaving">
                        <i v-if="itemSaving" class="bi bi-circle-fill"></i>
                        <i v-else class="bi bi-check"></i>
                      </b-button>
                      <b-button variant="secondary" size="sm" @click="editingItem = null">
                        <i class="bi bi-x-circle"></i>
                      </b-button>
                    </template>
                    <template v-else>
                      <b-button variant="outline-primary" size="sm" @click="startEditItem(item)" title="ویرایش آیتم">
                        <i class="bi bi-pencil"></i>
                      </b-button>
                      <b-button variant="outline-danger" size="sm" @click="removeItem(item.id)" title="حذف آیتم"
                        :disabled="itemRemoving === item.id">
                        <i v-if="itemRemoving === item.id" class="bi bi-dot"></i>
                        <i v-else class="bi bi-trash"></i>
                      </b-button>
                    </template>
                  </td>
                </tr>

                <tr v-if="!order?.items?.length">
                  <td colspan="7" class="text-center text-muted py-4">
                    <i class="fas fa-box-open fa-2x mb-2 d-block"></i>
                    هیچ محصولی در سفارش وجود ندارد
                  </td>
                </tr>

                <tr v-if="order?.items?.length" class="table-success fw-bold">
                  <td colspan="8" class="text-end">
                    <b-col cols="12" lg="12">
                      <b-card class="shadow-sm h-100" header-tag="header">
                        <template #header>
                          <div class="d-flex align-items-center">
                            <i class="fas fa-chart-pie me-2 text-success"></i>
                            <span class="fw-bold">خلاصه مالی</span>
                          </div>
                        </template>

                        <div class="info-item">
                          <span class="info-label">جمع جزء</span>
                          <span class="info-value">{{ formatPrice(order?.subtotal) }}</span>
                        </div>

                        <div class="info-item text-success">
                          <span class="info-label">
                            <i class="fas fa-tag me-1"></i> تخفیف
                          </span>
                          <span class="info-value">-{{ formatPrice(order?.discount_amount) }}</span>
                        </div>

                        <div class="info-item">
                          <span class="info-label">هزینه ارسال</span>
                          <span class="info-value">{{ formatPrice(order?.shipping_cost) }}</span>
                        </div>

                        <hr class="my-2">

                        <div class="info-item total-item">
                          <span class="info-label fw-bold">جمع کل</span>
                          <span class="info-value total-price">{{ formatPrice(order?.total) }}</span>
                        </div>
                      </b-card>
                    </b-col>
                  </td>

                </tr>
              </tbody>
            </table>
          </div>
        </b-card>
      </b-col>
    </b-row>

    <!-- مودال ویرایش آدرس -->
    <Modal v-if="showAddressModal" id="addressModal" @closeModal="() => { showAddressModal = false }"
      title="ویرایش آدرس">
      <div>
        <div class="mb-3">
          <label class="form-label">انتخاب آدرس جدید</label>
          <Treeselect v-model="addressForm.address_id" :multiple="false" :options="addressOptions"
            placeholder="انتخاب آدرس..." />
        </div>
        <div v-if="addressForm.current_address" class="text-muted small">
          آدرس فعلی: {{ getAddressText(addressForm.current_address) }}
        </div>
      </div>
      <div>
        <button class="btn btn-secondary" @click="() => { showAddressModal = false; resetAddressModal(); }">
          لغو
        </button>
        <button class="btn btn-primary" @click="saveAddress" :disabled="savingAddress">
          <i v-if="savingAddress" class="fas fa-spinner fa-spin me-1"></i>
          ذخیره تغییرات
        </button>
      </div>
    </Modal>

    <!-- مودال ویرایش روش حمل -->
    <Modal v-if="showShippingModal" id="shippingModal"
      @closeModal="() => { showShippingModal = false; resetShippingModal(); }" title="ویرایش روش حمل">
      <div>
        <div class="mb-3">
          <label class="form-label">انتخاب روش حمل</label>
          <Treeselect v-model="shippingForm.shipping_id" :multiple="false" :options="shippingOptions"
            placeholder="انتخاب روش حمل..." :normalizer="shippingNormalizer" />
        </div>
        <div v-if="shippingForm.current_shipping" class="text-muted small">
          روش فعلی: {{ shippingForm.current_shipping.title }} ({{ formatPrice(shippingForm.current_shipping.cost) }})
        </div>
      </div>
      <div>
        <button class="btn btn-secondary" @click="() => { showShippingModal = false; resetShippingModal(); }">
          لغو
        </button>
        <button class="btn btn-primary" @click="saveShipping" :disabled="savingShipping">
          <i v-if="savingShipping" class="fas fa-spinner fa-spin me-1"></i>
          ذخیره تغییرات
        </button>
      </div>
    </Modal>

    <!-- مودال افزودن آیتم -->
    <Modal v-if="showItemModal" id="itemModal" @closeModal="() => { showItemModal = false; resetItemModal(); }"
      title="افزودن محصول جدید" size="lg">
      <div>
        <div class="row g-3">
          <div class="col-12">
            <label class="form-label">انتخاب محصول</label>
            <multiselect @search-change="loadProducts" v-model="itemForm._selectedProduct" placeholder="جستجوی محصول..."
              open-direction="bottom" :options="productOptions" label="title" track-by="id" :searchable="true"
              :multiple="false" :close-on-select="true" :show-labels="false" @select="onProductSelect">
              <template slot="noOptions">جستجو کنید</template>
              <template slot="noResult">
                <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                <span v-else v-text="'موردی یافت نشد'"></span>
              </template>
            </multiselect>
          </div>
          <div class="col-md-6">
            <label class="form-label">تعداد</label>
            <input type="number" class="form-control" v-model.number="itemForm.quantity" min="1" />
          </div>
          <div class="col-md-6">
            <label class="form-label">قیمت واحد</label>
            <input type="number" class="form-control" v-model.number="itemForm.price" min="0" />
          </div>
        </div>
      </div>
      <div>
        <button class="btn btn-secondary" @click="() => { showItemModal = false; resetItemModal(); }">
          لغو
        </button>
        <button class="btn btn-success" @click="saveItem" :disabled="savingItem">
          <i v-if="savingItem" class="fas fa-spinner fa-spin me-1"></i>
          افزودن محصول
        </button>
      </div>
    </Modal>
  </b-container>
</template>

<script setup>
import { ref, computed, onMounted } from "vue"
import axios from "axios"
import { toast } from "vue3-toastify"
import "vue3-toastify/dist/index.css"
import { useRoute, useRouter } from "vue-router"
import { useAdmin } from '@/stores/modules/admin'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import Multiselect from 'vue-multiselect'
import 'vue-multiselect/dist/vue-multiselect.css'
import Modal from '@/components/shared/modal.vue'

const store = useAdmin()
const checkPermission = store.checkPermission
const route = useRoute()
const router = useRouter()

// State
const order = ref({ items: [], user: {}, address: {}, shipping: {} })
const updating = ref(false)
const isRequesting = ref(false)
const baseImageAddress = ''

// مودال‌ها
const showAddressModal = ref(false)
const showShippingModal = ref(false)
const showItemModal = ref(false)
const savingAddress = ref(false)
const savingShipping = ref(false)
const savingItem = ref(false)
// فرم‌ها
const addressForm = ref({ address_id: null, current_address: null })
const shippingForm = ref({ shipping_id: null, current_shipping: null })
const itemForm = ref({ product_id: null, product_variant_id: null, quantity: 1, price: 0, _selectedProduct: null })

// ویرایش آیتم
const editingItem = ref(null)
const itemEditForm = ref({ quantity: 1, price: 0 })
const itemSaving = ref(false)
const itemRemoving = ref(null)
const loadingReservation = ref(false)

// گزینه‌ها
const addressOptions = ref([])
const shippingOptions = ref([])
const productOptions = ref([])

// قابلیت ویرایش
const canEdit = computed(() => {
  if (!order.value) return false
  return !['completed', 'canceled', 'shipped'].includes(order.value.status)
})

const orderStatusOptions = [
  { value: "pending", text: "در انتظار" },
  { value: "reserved", text: "رزرو شده" },
  { value: "processing", text: "در حال پردازش" },
  { value: "shipped", text: "ارسال شده" },
  { value: "paid", text: "پرداخت شده" },
  { value: "completed", text: "تکمیل شده" },
  { value: "canceled", text: "لغو شده" },
  { value: "returned", text: "مرجوع شده" },
]

const reservationTypeOptions = [
  { value: "none", text: "عادی" },
  { value: "three_days", text: "رزرو سه روزه" },
  { value: "seven_days", text: "رزرو هفت روزه" },
]

const paymentMethods = {
  online: "پرداخت آنلاین",
  wallet: "کیف پول",
  cod: "پرداخت در محل",
}
function showAddressModalFunc() {
  addressForm.value.current_address = order.value.address
  showAddressModal.value = true
}
function showShippingModalFunc() {
  showShippingModal.value = true;
  shippingForm.value.current_shipping = {
    id: order.value.shipping_id,
    title: order.value.shipping?.title,
    cost: order.value.shipping_cost
  }
}
// ==================== آدرس ====================
const loadAddresses = async () => {
  try {
    const response = await axios.get(`/orders/${order.value.id}/addresses`)
    addressOptions.value = response.data.data.map(a => ({
      id: a.id,
      label: `${a.receiver_name} - ${a.address_line}`
    }))
    addressForm.value.current_address = order.value.address
  } catch (error) {
    console.error('Error loading addresses:', error)
  }
}

const saveAddress = async () => {
  if (!addressForm.value.address_id) {
    toast.error('لطفاً یک آدرس انتخاب کنید')
    return
  }

  savingAddress.value = true
  try {
    const response = await axios.put(`/orders/${order.value.id}/change-address`, {
      address_id: addressForm.value.address_id
    })

    if (response.data.success) {
      toast.success('آدرس با موفقیت تغییر کرد')
      await fetchOrder()
      showAddressModal.value = false
      resetAddressModal()
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در تغییر آدرس')
  } finally {
    savingAddress.value = false
  }
}


const resetAddressModal = () => {
  addressForm.value = { address_id: null, current_address: null }
}

// ==================== روش حمل ====================
const loadShippings = async () => {
  try {
    const response = await axios.get(`/orders/${order.value.id}/available-shippings`)
    shippingOptions.value = response.data.data.map(s => ({
      id: s.id,
      name: s.name,
      cost: s.cost,
      is_current: s.is_current
    }))
    shippingForm.value.current_shipping = {
      id: order.value.shipping_id,
      title: order.value.shipping?.title,
      cost: order.value.shipping_cost
    }
  } catch (error) {
    console.error('Error loading shippings:', error)
  }
}

const saveShipping = async () => {
  if (!shippingForm.value.shipping_id) {
    toast.error('لطفاً یک روش حمل انتخاب کنید')
    return
  }

  savingShipping.value = true
  try {
    const response = await axios.put(`/orders/${order.value.id}/change-shipping`, {
      shipping_id: shippingForm.value.shipping_id
    })

    if (response.data.success) {
      toast.success('روش حمل با موفقیت تغییر کرد')
      await fetchOrder()
      showShippingModal.value = false
      resetShippingModal()
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در تغییر روش حمل')
  } finally {
    savingShipping.value = false
  }
}

const resetShippingModal = () => {
  shippingForm.value = { shipping_id: null, current_shipping: null }
}

// ==================== نوع سفارش ====================
const changeReservationType = async () => {
  loadingReservation.value = true
  try {
    const response = await axios.put(`/orders/${order.value.id}/change-reservation-type`, {
      reservation_type: order.value.reservation_type
    })

    if (response.data.success) {
      toast.success('نوع سفارش با موفقیت تغییر کرد')
      await fetchOrder()
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در تغییر نوع سفارش')
  } finally {
    loadingReservation.value = false
  }
}

const getReservationTypeText = (type) => {
  const map = {
    three_days: 'رزرو سه روزه',
    seven_days: 'رزرو هفت روزه'
  }
  return map[type] || 'عادی'
}

// ==================== آیتم‌ها ====================
const startEditItem = (item) => {
  editingItem.value = item.id
  itemEditForm.value = {
    quantity: item.quantity,
    price: item.price
  }
}

const saveItemEdit = async (item) => {
  if (itemEditForm.value.quantity < 1) {
    toast.error('تعداد باید حداقل 1 باشد')
    return
  }
  if (itemEditForm.value.price < 0) {
    toast.error('قیمت نمی‌تواند منفی باشد')
    return
  }

  itemSaving.value = true
  try {
    const response = await axios.put(`/orders/${order.value.id}/items/${item.id}`, {
      quantity: itemEditForm.value.quantity,
      price: itemEditForm.value.price
    })

    if (response.data.success) {
      toast.success('آیتم با موفقیت به‌روزرسانی شد')
      await fetchOrder()
      editingItem.value = null
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در به‌روزرسانی آیتم')
  } finally {
    itemSaving.value = false
  }
}

const removeItem = async (itemId) => {
  if (!confirm('آیا از حذف این آیتم مطمئن هستید؟')) return

  itemRemoving.value = itemId
  try {
    const response = await axios.delete(`/orders/${order.value.id}/items/${itemId}`)

    if (response.data.success) {
      toast.success('آیتم با موفقیت حذف شد')
      await fetchOrder()
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در حذف آیتم')
  } finally {
    itemRemoving.value = null
  }
}

// ==================== افزودن آیتم ====================
const onProductSelect = (product) => {
  if (product) {
    itemForm.value.product_id = product.product_id
    itemForm.value.product_variant_id = product.id
    itemForm.value.price = product.price
    itemForm.value._selectedProduct = product
  }
}

const saveItem = async () => {
  if (!itemForm.value._selectedProduct) {
    toast.error('لطفاً یک محصول انتخاب کنید')
    return
  }
  if (itemForm.value.quantity < 1) {
    toast.error('تعداد باید حداقل 1 باشد')
    return
  }
  if (itemForm.value.price < 0) {
    toast.error('قیمت نمی‌تواند منفی باشد')
    return
  }

  savingItem.value = true
  try {
    const response = await axios.post(`/orders/${order.value.id}/items`, {
      product_id: itemForm.value.product_id,
      product_variant_id: itemForm.value.product_variant_id,
      quantity: itemForm.value.quantity,
      price: itemForm.value.price
    })

    if (response.data.success) {
      toast.success('آیتم با موفقیت اضافه شد')
      await fetchOrder()
      showItemModal.value = false
      resetItemModal()
    }
  } catch (error) {
    toast.error(error.response?.data?.message || 'خطا در افزودن آیتم')
  } finally {
    savingItem.value = false
  }
}
const resetItemModal = () => {
  itemForm.value = {
    product_id: null,
    product_variant_id: null,
    quantity: 1,
    price: 0,
    _selectedProduct: null
  }
}

// ==================== لود محصولات ====================
const loadProducts = async (search) => {
  if (!search || search.length < 2) return
  isRequesting.value = true
  try {
    const { data } = await axios.get('/products', { params: { search } })
    productOptions.value = await convertToSelectableProduct(data.data)
  } catch (error) {
    console.error('Error loading products:', error)
  } finally {
    isRequesting.value = false
  }
}

async function convertToSelectableProduct(productList) {
  let finalList = []
  productList.forEach(product => {
    if (product.variants.length > 1) {
      product.variants.forEach((variant) => {
        let obj = {
          id: variant.id,
          product_id: product.id,
          price: variant.price,
          title: `${product.title} || ${variant.values.map((att) => att.value).join("-")} || موجودی: ${variant.stock}`,
          isDisabled: variant.stock > 0 ? false : true
        }
        finalList.push(obj)
      })
    } else {
      let obj = {
        isDisabled: product.variants[0].stock > 0 ? false : true,
        id: product.variants[0].id,
        title: product.title,
        price: product.price,
        product_id: product.id
      }
      finalList.push(obj)
    }
  })
  return finalList
}

const shippingNormalizer = (node) => {
  return {
    id: node.id,
    label: node.name + (node.cost !== undefined ? ` (${node.cost.toLocaleString()} تومان)` : ''),
  }
}

// ==================== وضعیت ====================
const updateOrderStatus = async () => {
  if (!order.value?.id) {
    toast.error("شناسه سفارش نامعتبر")
    return
  }

  updating.value = true
  try {
    let fd = new FormData()
    fd.append("status", order.value.status)
    await axios.post(`/orders-change-status/${order.value.id}`, fd)
    toast.success("وضعیت سفارش با موفقیت بروزرسانی شد")
    await fetchOrder()
  } catch (e) {
    toast.error("خطا در بروزرسانی وضعیت سفارش")
    console.error(e)
  } finally {
    updating.value = false
  }
}

// ==================== دریافت سفارش ====================
const fetchOrder = async () => {
  try {
    const res = await axios.get(`/orders/${route.params.id}`)
    order.value = res.data.data || {}

    if (!order.value.items) order.value.items = []
    if (!order.value.user) order.value.user = {}
    if (!order.value.address) order.value.address = {}
    if (!order.value.shipping) order.value.shipping = {}

    // بارگذاری داده‌های مودال‌ها
    if (canEdit.value) {
      await loadAddresses()
      await loadShippings()
    }
  } catch (e) {
    toast.error("خطا در گرفتن اطلاعات سفارش")
    console.error(e)
  }
}

// ==================== توابع کمکی ====================
const getAddressText = (address) => {
  if (!address) return '---'
  return `${address.province?.name || ''} - ${address.city?.name || ''} - ${address.address_line || ''}`
}

const getStatusText = (status) => {
  const map = {
    pending: 'در انتظار',
    reserved: 'رزرو شده',
    processing: 'در حال پردازش',
    shipped: 'ارسال شده',
    paid: 'پرداخت شده',
    completed: 'تکمیل شده',
    canceled: 'لغو شده',
    returned: 'مرجوع شده'
  }
  return map[status] || status || 'نامشخص'
}

const getStatusClass = (status) => {
  const map = {
    pending: 'badge bg-warning text-dark',
    reserved: 'badge bg-info text-white',
    processing: 'badge bg-primary text-white',
    shipped: 'badge bg-purple text-white',
    completed: 'badge bg-success text-white',
    canceled: 'badge bg-danger text-white',
    returned: 'badge bg-secondary text-white'
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
  if (!item?.product) return 'نامشخص'
  let title = item.product.title || 'نامشخص'
  if (item.variant?.values?.length) {
    title += ` [${item.variant.values.map(v => decodeURIComponent(v.value)).join(' - ')}]`
  }
  return title
}

const getVariantText = (item) => {
  if (!item?.variant?.values?.length) return ''
  return item.variant.values
    .map(v => {
      const name = v.attribute_id === 1 ? 'سایز' : v.attribute_id === 2 ? 'رنگ' : 'ویژگی'
      return `${name}: ${decodeURIComponent(v.value)}`
    })
    .join(' - ')
}

const getProductImage = (item) => {
  if (!item?.product) return ''
  if (item.product.main_image && item.product.main_image.includes('http'))
    return item.product.main_image
  return baseImageAddress + item.product.main_image
}

const handleImageError = (e) => {
  e.target.src = '/images/no-image.png'
}

onMounted(fetchOrder)
</script>

<style scoped>
.info-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #f0f0f0;
  align-items: center;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  color: #6c757d;
  font-size: 0.9rem;
  min-width: 70px;
}

.info-value {
  font-weight: 500;
  text-align: left;
  direction: ltr;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.total-item {
  padding: 12px 0;
  border-bottom: 2px solid #dee2e6;
}

.total-price {
  font-size: 1.2rem;
  color: #28a745;
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