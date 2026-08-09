<template>
    <div class="container mt-4 orders-page" v-if="checkPermission(['order_today'])">
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-list-check"></i>
                    <span>
                        مدیریت سفارش‌ها
                    </span>
                </h3>
                <div class="d-flex align-items-center gap-2">
                    <!-- انتخاب نوع پرینت -->
                    <select v-model="printType" class="form-select form-select-sm" style="width: 200px;">
                        <option value="full">پرینت کامل (جزئیات سفارش)</option>
                        <option value="label">پرینت برچسب (فرستنده و گیرنده)</option>
                    </select>
                    
                    <!-- دکمه پرینت گروهی -->
                    <button 
                        v-if="selectedOrders.length > 0" 
                        @click="goToPrint" 
                        class="btn btn-success"
                    >
                        <i class="bi bi-printer"></i>
                        پرینت ({{ selectedOrders.length }})
                    </button>
                    <router-link to="/orders/create" class="btn btn-primary">
                        <i class="bi bi-plus"></i>
                        <span>
                            افزودن سفارش
                        </span>
                    </router-link>
                </div>
            </div>
            <div class="card-body row g-2">
                <div class="col-md-3">
                    <input v-model="filters.search" @input="getOrders" type="text" class="form-control"
                        placeholder="جستجو (کاربر یا شماره سفارش)" />
                </div>
                <div class="col-md-2">
                    <select v-model="filters.status" @change="getOrders" class="form-select">
                        <option value="">همه وضعیت‌ها</option>
                        <option value="pending">در انتظار</option>
                        <option value="reserved">رزرو شده</option>
                        <option value="processing">در حال پردازش</option>
                        <option value="shipped">ارسال شده</option>
                        <option value="completed">تکمیل شده</option>
                        <option value="canceled">لغو شده</option>
                        <option value="returned">مرجوعی</option>
                    </select>
                </div>
                <div class="col-md-2">
                    <select v-model="filters.payment_status" @change="getOrders" class="form-select">
                        <option value="">همه پرداخت‌ها</option>
                        <option value="pending">در انتظار پرداخت</option>
                        <option value="paid">پرداخت شده</option>
                        <option value="failed">ناموفق</option>
                        <option value="refunded">برگشت داده شده</option>
                    </select>
                </div>
                <div class="col-md-2">
                    <select v-model="filters.payment_method" @change="getOrders" class="form-select">
                        <option value="">روش پرداخت</option>
                        <option value="online">پرداخت آنلاین</option>
                        <option value="wallet">کیف پول</option>
                        <option value="cod">پرداخت در محل</option>
                    </select>
                </div>
            </div>
        </div>

        <!-- Table -->
        <div class="card">
            <div class="card-body table-responsive">
                <table class="table table-bordered align-middle text-center">
                    <thead>
                        <tr>
                            <th style="width: 50px;">
                                <input 
                                    type="checkbox" 
                                    :checked="allSelected" 
                                    @change="toggleAll"
                                    :disabled="orders.length === 0"
                                />
                            </th>
                            <th>#</th>
                            <th>کاربر</th>
                            <th>آدرس</th>
                            <th>روش ارسال</th>
                            <th>مبلغ کل</th>
                            <th>وضعیت سفارش</th>
                            <th>وضعیت پرداخت</th>
                            <th>روش پرداخت</th>
                            <th style="width: 120px;">عملیات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="loading">
                            <td colspan="10" class="text-center">
                                <div class="spinner-border" role="status"></div>
                            </td>
                        </tr>
                        <tr v-else v-for="order in orders" :key="order.id">
                            <td>
                                <input 
                                    type="checkbox" 
                                    :value="order.id" 
                                    v-model="selectedOrders"
                                />
                            </td>
                            <td>{{ order.id }}</td>
                            <td>{{ order.user?.full_name ?? "-" }}</td>
                            <td>{{ order.address?.address_line ?? "-" }}</td>
                            <td>{{ order.shipping?.title ?? "-" }}</td>
                            <td>{{ Number(order.total).toLocaleString('fa-IR') }} تومان</td>
                            <td>
                                <span class="badge" :class="statusBadge(order.status)">
                                    {{ statusText(order.status) }}
                                </span>
                            </td>
                            <td>
                                <span class="badge" :class="paymentStatusBadge(order.payment_status)">
                                    {{ paymentStatusText(order.payment_status) }}
                                </span>
                            </td>
                            <td>{{ paymentMethodText(order.payment_method) }}</td>
                            <td>
                                <router-link :to="`/orders/${order.id}`" class="btn btn-sm btn-info">
                                    <i class="bi bi-eye"></i>
                                </router-link>
                                <!-- دکمه پرینت تکی -->
                                <button @click="singlePrint(order.id)" class="btn btn-sm btn-secondary">
                                    <i class="bi bi-printer"></i>
                                </button>
                            </td>
                        </tr>
                        <tr v-if="!loading && orders.length === 0">
                            <td colspan="10" class="text-center">هیچ سفارشی یافت نشد</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import { useAdmin } from '@/stores/modules/admin';

const router = useRouter();
const store = useAdmin();
const checkPermission = store.checkPermission;

const orders = ref([]);
const loading = ref(false);
const selectedOrders = ref([]);
const printType = ref('full');

const filters = ref({
    search: "",
    status: "",
    payment_status: "",
    payment_method: "",
});
const currentPage = ref(1);
let abortController = null;

// محاسبه انتخاب همه
const allSelected = computed(() => {
    return orders.value.length > 0 && 
           orders.value.every(order => selectedOrders.value.includes(order.id));
});

// انتخاب/لغو انتخاب همه
const toggleAll = (event) => {
    if (event.target.checked) {
        selectedOrders.value = orders.value.map(order => order.id);
    } else {
        selectedOrders.value = [];
    }
};

// رفتن به صفحه پرینت با انتخاب‌های گروهی
const goToPrint = () => {
    if (selectedOrders.value.length === 0) {
        alert('لطفاً حداقل یک سفارش را انتخاب کنید.');
        return;
    }
    router.push({
        path: '/orders/print',
        query: { 
            ids: selectedOrders.value.join(','),
            type: printType.value
        }
    });
};

// پرینت تکی
const singlePrint = (orderId) => {
    router.push({
        path: '/orders/print',
        query: { 
            ids: orderId.toString(),
            type: printType.value
        }
    });
};

const getOrders = async (page = 1) => {
    loading.value = true;
    // پاک کردن انتخاب‌ها
    selectedOrders.value = [];
    
    if (abortController) {
        abortController.abort();
    }

    abortController = new AbortController();

    try {
        const response = await axios.get("/orders-todays-orders", {
            params: {
                ...filters.value,
            },
            signal: abortController.signal,
        });
        orders.value = response.data.data;
        currentPage.value = page;
    } catch (error) {
        if (error.name !== 'AbortError') {
            console.error('Error fetching orders:', error);
        }
    } finally {
        loading.value = false;
    }
};

const statusText = (status) => {
    const map = {
        pending: "در انتظار",
        reserved: "رزرو شده",
        processing: "در حال پردازش",
        paid: "پرداخت  شده",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        returned: "مرجوعی",
    };
    return map[status] ?? status;
};

const statusBadge = (status) => {
    const map = {
        pending: "bg-secondary",
        reserved: "bg-warning text-dark",
        processing: "bg-info",
        shipped: "bg-primary",
        paid: "bg-success",
        completed: "bg-success",
        canceled: "bg-danger",
        returned: "bg-dark",
    };
    return map[status] ?? "bg-secondary";
};

const paymentStatusText = (status) => {
    const map = {
        pending: "در انتظار پرداخت",
        paid: "پرداخت شده",
        failed: "ناموفق",
        refunded: "برگشت داده شده",
    };
    return map[status] ?? status;
};

const paymentStatusBadge = (status) => {
    const map = {
        pending: "bg-warning text-dark",
        paid: "bg-success",
        failed: "bg-danger",
        refunded: "bg-secondary",
    };
    return map[status] ?? "bg-secondary";
};

const paymentMethodText = (method) => {
    const map = {
        online: "پرداخت آنلاین",
        wallet: "کیف پول",
        cod: "پرداخت در محل",
    };
    return map[method] ?? method;
};

onMounted(() => {
    getOrders();
});
</script>

<style scoped>
/* استایل‌های اضافی برای بهبود ظاهر */
.orders-page .btn-sm {
    margin: 0 2px;
}

.orders-page .btn-sm i {
    font-size: 14px;
}

/* استایل چک‌باکس‌ها */
.orders-page input[type="checkbox"] {
    width: 18px;
    height: 18px;
    cursor: pointer;
    accent-color: #667eea;
}

/* استایل هدر جدول */
.orders-page thead th {
    background-color: #f8f9fa;
    font-weight: 600;
    white-space: nowrap;
}

/* استایل ردیف‌های جدول */
.orders-page tbody tr {
    transition: background-color 0.2s;
}

.orders-page tbody tr:hover {
    background-color: #f8f9ff;
}

/* استایل دکمه‌ها */
.orders-page .btn-success {
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    border: none;
    transition: all 0.3s;
}

.orders-page .btn-success:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(16, 185, 129, 0.4);
}

.orders-page .btn-primary {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border: none;
    transition: all 0.3s;
}

.orders-page .btn-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}

/* استایل سلکت پرینت */
.orders-page .form-select-sm {
    border: 2px solid #e5e7eb;
    border-radius: 8px;
    padding: 6px 12px;
    font-size: 13px;
    transition: all 0.3s;
}

.orders-page .form-select-sm:focus {
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}
</style>