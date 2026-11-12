<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3 shipping-methods" v-if="checkPermission(['shipping_view'])">

    <!-- فیلتر -->
    <div class="card mb-2 header-card">
      <div class="card-body p-2 p-md-3">
        <div class="row g-2">
          <div class="col-12 col-sm-8 col-md-4">
            <input v-model="filters.search" @keyup.enter="fetchData" type="text" class="form-control search-input"
              placeholder="جستجو بر اساس نام" />
          </div>
          <div class="col-12 col-sm-4 col-md-2">
            <button @click="fetchData" class="btn btn-primary w-100 search-btn">
              <i class="bi bi-search"></i>
              <span>جستجو</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- نوار ابزار: افزودن + شروع مرتب‌سازی + تایید / انصراف -->
    <div class="mb-2 toolbar-wrapper">
      <template v-if="!isSorting">
        <router-link to="/shop/shipping/create" class="btn btn-success add-btn">
          <i class="bi bi-plus"></i>
          <span>افزودن روش ارسال</span>
        </router-link>
        <button class="btn btn-outline-primary sort-start-btn" @click="startSorting"
          :disabled="shippingMethods.length < 2">
          <i class="bi bi-arrow-down-up"></i>
          <span>شروع مرتب‌سازی</span>
        </button>
      </template>
      <template v-else>
        <div class="sort-hint text-muted small me-auto">
          <i class="bi bi-info-circle"></i>
          با دکمه‌های <i class="bi bi-arrow-up"></i> / <i class="bi bi-arrow-down"></i> ترتیب را تغییر دهید، سپس تأیید
          کنید.
        </div>
        <button class="btn btn-success" @click="confirmSorting" :disabled="savingOrder">
          <i class="bi bi-check2-circle"></i>
          <span>{{ savingOrder ? 'در حال ذخیره...' : 'تأیید ترتیب' }}</span>
        </button>
        <button class="btn btn-outline-secondary" @click="cancelSorting" :disabled="savingOrder">
          <i class="bi bi-x-circle"></i>
          <span>انصراف</span>
        </button>
      </template>
    </div>

    <!-- جدول -->
    <div class="card">
      <div class="card-body p-2 p-md-3">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status"></div>
        </div>

        <div v-else>
          <div v-if="!shippingMethods || shippingMethods.length === 0" class="text-center py-5 text-muted">
            <i class="bi bi-inbox fs-1 d-block mb-2"></i>
            <p>روش ارسالی یافت نشد</p>
          </div>

          <template v-else>
            <!-- ===== جدول دسکتاپ ===== -->
            <div class="table-responsive d-none d-md-block">
              <table class="table table-bordered table-hover mb-0">
                <thead>
                  <tr>
                    <th v-if="isSorting" style="width: 48px;" class="text-center">ترتیب</th>
                    <th>نام</th>
                    <th>توضیحات</th>
                    <th class="text-center">هزینه</th>
                    <th class="text-center">وضعیت</th>
                    <th class="text-center">عملیات</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(method, index) in shippingMethods" :key="method.id" :class="{ 'sorting-row': isSorting }">
                    <td v-if="isSorting" class="text-center">
                      <div class="sort-buttons">
                        <button class="btn btn-sm btn-light sort-btn" @click="moveUp(index)" :disabled="index === 0"
                          title="بالا">
                          <i class="bi bi-arrow-up"></i>
                        </button>
                        <button class="btn btn-sm btn-light sort-btn" @click="moveDown(index)"
                          :disabled="index === shippingMethods.length - 1" title="پایین">
                          <i class="bi bi-arrow-down"></i>
                        </button>
                      </div>
                    </td>
                    <td class="method-title">{{ method.title }}</td>
                    <td class="method-desc">{{ method.description || '—' }}</td>
                    <td class="text-center">
                      <span class="cost-badge">
                        <i class="bi bi-cash-coin"></i>
                        {{ method.cost }} تومان
                      </span>
                    </td>
                    <td class="text-center">
                      <span class="status-badge" :class="method.status ? 'status-active' : 'status-inactive'">
                        <i class="bi" :class="method.status ? 'bi-check-circle-fill' : 'bi-x-circle-fill'"></i>
                        {{ method.status ? 'فعال' : 'غیرفعال' }}
                      </span>
                    </td>
                    <td>
                      <div class="d-flex flex-wrap gap-1 justify-content-center">
                        <router-link :to="`/shop/shipping/${method.id}/edit`" class="btn btn-sm btn-primary">
                          <i class="bi bi-pen"></i><span>ویرایش</span>
                        </router-link>
                        <button @click="deleteMethod(method.id)" class="btn btn-sm btn-danger" :disabled="isSorting">
                          <i class="bi bi-trash3-fill"></i><span>حذف</span>
                        </button>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>

            <!-- ===== کارت موبایل ===== -->
            <div class="d-md-none method-cards">
              <div v-for="(method, index) in shippingMethods" :key="method.id" class="method-card"
                :class="{ 'inactive-card': !method.status, 'sorting-card': isSorting }">
                <div class="method-card-header">
                  <div class="method-icon">
                    <i class="bi bi-truck"></i>
                  </div>
                  <div class="method-info">
                    <div class="method-name">{{ method.title }}</div>
                    <div class="method-id">شناسه: #{{ method.id }}</div>
                  </div>
                  <span class="status-badge-sm" :class="method.status ? 'status-active' : 'status-inactive'">
                    <i class="bi" :class="method.status ? 'bi-check-circle-fill' : 'bi-x-circle-fill'"></i>
                    {{ method.status ? 'فعال' : 'غیرفعال' }}
                  </span>
                </div>

                <div class="method-card-body">
                  <div class="info-row">
                    <i class="bi bi-cash-coin"></i>
                    <span class="info-label">هزینه:</span>
                    <span class="info-value cost-value">{{ method.cost }} تومان</span>
                  </div>
                  <div class="info-row desc-row" v-if="method.description">
                    <i class="bi bi-card-text"></i>
                    <span class="info-label">توضیحات:</span>
                    <span class="info-value desc-value">{{ method.description }}</span>
                  </div>
                </div>

                <!-- دکمه‌های مرتب‌سازی موبایل -->
                <div v-if="isSorting" class="mobile-sort-row">
                  <button class="btn btn-sm btn-outline-primary flex-fill" @click="moveUp(index)"
                    :disabled="index === 0">
                    <i class="bi bi-arrow-up"></i> بالا
                  </button>
                  <button class="btn btn-sm btn-outline-primary flex-fill" @click="moveDown(index)"
                    :disabled="index === shippingMethods.length - 1">
                    <i class="bi bi-arrow-down"></i> پایین
                  </button>
                </div>

                <div class="method-card-actions" v-else>
                  <router-link :to="`/shop/shipping/${method.id}/edit`" class="btn btn-sm btn-primary flex-fill">
                    <i class="bi bi-pen"></i><span>ویرایش</span>
                  </router-link>
                  <button @click="deleteMethod(method.id)" class="btn btn-sm btn-danger flex-fill">
                    <i class="bi bi-trash3-fill"></i><span>حذف</span>
                  </button>
                </div>
              </div>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import Swal from "sweetalert2";
import axios from "axios";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;

const shippingMethods = ref([]);
const loading = ref(false);
const filters = ref({ search: "" });

const isSorting = ref(false);
const savingOrder = ref(false);
// نسخه اصلی ترتیب برای انصراف
let originalOrderIds = [];

/* ===== دریافت داده ===== */
const fetchData = async (url = "/shipping-methods") => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    shippingMethods.value = data.data ?? [];
    // اگر در حال مرتب‌سازی هستیم، بعد از fetch خارجش نکنیم
    if (!isSorting.value) {
      originalOrderIds = shippingMethods.value.map(m => m.id);
    }
  } finally {
    loading.value = false;
  }
};

/* ===== شروع مرتب‌سازی ===== */
const startSorting = () => {
  originalOrderIds = shippingMethods.value.map(m => m.id);
  isSorting.value = true;
};

/* ===== حرکت بالا ===== */
const moveUp = (index) => {
  if (index <= 0) return;
  const arr = shippingMethods.value;
  [arr[index - 1], arr[index]] = [arr[index], arr[index - 1]];
};

/* ===== حرکت پایین ===== */
const moveDown = (index) => {
  const arr = shippingMethods.value;
  if (index >= arr.length - 1) return;
  [arr[index + 1], arr[index]] = [arr[index], arr[index + 1]];
};

/* ===== تأیید ترتیب ===== */
const confirmSorting = async () => {
  savingOrder.value = true;
  try {
    await axios.post("/shipping-methods/reorder", {
      ids: shippingMethods.value.map(m => m.id),
    });
    originalOrderIds = shippingMethods.value.map(m => m.id);
    isSorting.value = false;
    Swal.fire({
      toast: true, position: "top-end", icon: "success",
      title: "ترتیب ذخیره شد", showConfirmButton: false, timer: 1500,
    });
  } catch (e) {
    Swal.fire("خطا", "ذخیره ترتیب ناموفق بود", "error");
  } finally {
    savingOrder.value = false;
  }
};

/* ===== انصراف ===== */
const cancelSorting = () => {
  // بازگردانی به ترتیب اصلی
  const map = new Map(shippingMethods.value.map(m => [m.id, m]));
  shippingMethods.value = originalOrderIds
    .map(id => map.get(id))
    .filter(Boolean);
  isSorting.value = false;
};

/* ===== حذف ===== */
const deleteMethod = async (id) => {
  if (isSorting.value) return;
  const result = await Swal.fire({
    title: "آیا مطمئن هستید؟",
    text: "این عملیات قابل بازگشت نیست!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });
  if (result.isConfirmed) {
    await axios.delete(`/shipping-methods/${id}`);
    Swal.fire("حذف شد!", "روش ارسال با موفقیت حذف شد.", "success");
    fetchData();
  }
};

onMounted(() => fetchData());
</script>


<style scoped>
/* ===== فیلتر ===== */
.header-card .card-body {
  padding: 14px 18px;
}

.search-input {
  border-radius: 10px;
  padding: 10px 14px;
  border: 1px solid #e0e0e0;
  transition: all 0.2s ease;
}

.search-input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.search-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 10px 16px;
  border-radius: 10px;
  font-weight: 600;
}

/* ===== نوار ابزار ===== */
.toolbar-wrapper {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  justify-content: flex-end;
}

.add-btn,
.sort-start-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  white-space: nowrap;
}

/* ===== جدول ===== */
.table {
  margin-bottom: 0;
}

.table thead th {
  background: #f8f9fa;
  font-weight: 600;
  color: #2d3436;
  white-space: nowrap;
  font-size: 0.9rem;
}

.table tbody td {
  vertical-align: middle;
  font-size: 0.9rem;
}

.method-title {
  font-weight: 600;
  max-width: 200px;
  word-break: break-word;
}

.method-desc {
  max-width: 300px;
  color: #6c757d;
  font-size: 0.85rem;
  word-break: break-word;
}

.cost-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  background: #f0fdf4;
  color: #16a34a;
  font-size: 0.8rem;
  font-weight: 700;
  padding: 4px 12px;
  border-radius: 20px;
  white-space: nowrap;
}

.status-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.78rem;
  font-weight: 700;
  padding: 4px 12px;
  border-radius: 20px;
}

.status-active {
  background: #dcfce7;
  color: #16a34a;
}

.status-inactive {
  background: #fee2e2;
  color: #dc2626;
}

.status-badge-sm {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  font-size: 0.68rem;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 20px;
  flex-shrink: 0;
}

.status-badge-sm.status-active {
  background: #dcfce7;
  color: #16a34a;
}

.status-badge-sm.status-inactive {
  background: #fee2e2;
  color: #dc2626;
}

/* ===== حالت مرتب‌سازی ===== */
.sorting-row {
  background: #f0f9ff !important;
  transition: background 0.2s ease;
}

.sort-buttons {
  display: inline-flex;
  flex-direction: column;
  gap: 4px;
  align-items: center;
}

.sort-btn {
  padding: 2px 8px;
  border: 1px solid #dee2e6;
  background: #fff;
  line-height: 1;
  font-size: 0.85rem;
  color: #495057;
  transition: all 0.15s ease;
}

.sort-btn:hover:not(:disabled) {
  background: #3b82f6;
  color: #fff;
  border-color: #3b82f6;
}

.sort-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.sorting-card {
  border: 2px dashed #3b82f6 !important;
  background: #f0f9ff !important;
}

.mobile-sort-row {
  display: flex;
  gap: 8px;
  padding-top: 12px;
  border-top: 1px solid #f0f0f0;
  margin-top: 4px;
}

/* ===== کارت‌های موبایل ===== */
.method-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.method-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 14px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.method-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.method-card.inactive-card {
  background: #fefefe;
  border-right: 4px solid #ef4444;
}

.method-card.inactive-card .method-icon {
  background: linear-gradient(135deg, #94a3b8, #cbd5e1);
  box-shadow: 0 4px 12px rgba(148, 163, 184, 0.25);
}

.method-card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding-bottom: 12px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 12px;
}

.method-icon {
  width: 44px;
  height: 44px;
  background: linear-gradient(135deg, #3b82f6, #2563eb);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 20px;
  flex-shrink: 0;
  box-shadow: 0 4px 12px rgba(59, 130, 246, 0.25);
}

.method-info {
  flex: 1;
  min-width: 0;
}

.method-name {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.method-id {
  font-size: 0.72rem;
  color: #6c757d;
  margin-top: 2px;
}

.method-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.info-row>i {
  color: #6c5ce7;
  font-size: 0.95rem;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
}

.info-label {
  color: #6c757d;
  font-weight: 500;
}

.info-value {
  color: #2d3436;
  font-weight: 600;
  margin-right: auto;
}

.cost-value {
  color: #16a34a;
  background: #f0fdf4;
  padding: 3px 10px;
  border-radius: 20px;
  font-size: 0.78rem;
  font-weight: 700;
}

.desc-row {
  align-items: flex-start;
}

.desc-row>i {
  margin-top: 3px;
}

.desc-value {
  color: #6c757d;
  font-weight: 500;
  font-size: 0.8rem;
  text-align: right;
  line-height: 1.45;
  word-break: break-word;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.method-card-actions {
  display: flex;
  gap: 8px;
  padding-top: 12px;
  border-top: 1px solid #f0f0f0;
}

.method-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.8rem;
  padding: 8px 10px;
  font-weight: 600;
}

/* ========================================= */
@media (max-width: 767.98px) {
  .toolbar-wrapper {
    justify-content: stretch;
  }

  .add-btn,
  .sort-start-btn {
    width: 100%;
    justify-content: center;
    padding: 10px 16px;
  }

  .sort-hint {
    width: 100%;
    margin-bottom: 6px;
  }
}

@media (max-width: 399.98px) {
  .method-card {
    padding: 12px;
  }

  .method-icon {
    width: 38px;
    height: 38px;
    font-size: 17px;
  }

  .method-name {
    font-size: 0.9rem;
  }

  .info-row {
    font-size: 0.78rem;
  }

  .method-card-actions .btn {
    font-size: 0.72rem;
    padding: 6px 8px;
  }

  .status-badge-sm {
    font-size: 0.62rem;
    padding: 2px 6px;
  }
}

@media (min-width: 768px) {
  .method-cards {
    display: none;
  }
}
</style>