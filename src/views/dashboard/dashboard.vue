<template>
    <b-container fluid class="py-4" v-if="checkPermission(['dashboard_view'])">
        <!-- هدر داشبورد -->
        <b-row class="mb-4">
            <b-col cols="12">
                <div class="dashboard-header">
                    <h2 class="mb-0">📊 داشبورد مدیریت</h2>
                    <small class="text-muted">آخرین به‌روزرسانی: {{ currentTime }}</small>
                </div>
            </b-col>
        </b-row>

        <!-- کارت‌های آماری اصلی -->
        <b-row class="mb-4">
            <b-col cols="6" md="3" class="mb-3" v-for="stat in mainStats" :key="stat.label">
                <b-card class="stat-card h-100" :class="stat.color">
                    <div class="d-flex align-items-center">
                        <div class="stat-icon">
                            <i :class="stat.icon"></i>
                        </div>
                        <div class="stat-content ms-3">
                            <div class="stat-number">{{ stat.value }}</div>
                            <div class="stat-label">{{ stat.label }}</div>
                        </div>
                    </div>
                </b-card>
            </b-col>
        </b-row>

        <!-- ردیف اول: سفارش‌ها و محصولات -->
        <b-row>
            <!-- سفارش‌ها -->
            <b-col cols="12" lg="6" class="mb-4">
                <b-card class="dashboard-card h-100">
                    <template #header>
                        <div class="d-flex justify-content-between align-items-center">
                            <h5 class="mb-0">📦 سفارش‌ها</h5>
                            <span class="badge bg-primary">{{ dashboard.orders.total_orders }} سفارش</span>
                        </div>
                    </template>

                    <b-row class="g-2 mb-3">
                        <b-col cols="4" sm="3" v-for="(value, key) in orderStats" :key="key">
                            <div class="stat-item">
                                <div class="stat-item-label">{{ orderLabels[key] }}</div>
                                <div class="stat-item-value">{{ formatNumber(value) }}</div>
                            </div>
                        </b-col>
                    </b-row>

                    <div class="chart-wrapper">
                        <ApexChart 
                            type="line" 
                            height="200" 
                            :options="orderChartOptions" 
                            :series="orderSeries" 
                        />
                    </div>
                </b-card>
            </b-col>

            <!-- محصولات -->
            <b-col cols="12" lg="6" class="mb-4">
                <b-card class="dashboard-card h-100">
                    <template #header>
                        <div class="d-flex justify-content-between align-items-center">
                            <h5 class="mb-0">🛒 محصولات</h5>
                            <span class="badge bg-success">{{ dashboard.products.total_products }} محصول</span>
                        </div>
                    </template>

                    <b-row class="g-2 mb-3">
                        <b-col cols="4" sm="3" v-for="(value, key) in productStats" :key="key">
                            <div class="stat-item">
                                <div class="stat-item-label">{{ productLabels[key] }}</div>
                                <div class="stat-item-value">{{ formatNumber(value) }}</div>
                            </div>
                        </b-col>
                    </b-row>

                    <div class="chart-wrapper">
                        <ApexChart 
                            type="pie" 
                            height="200" 
                            :options="productChartOptions" 
                            :series="productSeries" 
                        />
                    </div>
                </b-card>
            </b-col>
        </b-row>

        <!-- ردیف دوم: کاربران و دیدگاه‌ها -->
        <b-row>
            <!-- کاربران -->
            <b-col cols="12" lg="6" class="mb-4">
                <b-card class="dashboard-card h-100">
                    <template #header>
                        <div class="d-flex justify-content-between align-items-center">
                            <h5 class="mb-0">👤 کاربران</h5>
                            <span class="badge bg-info">{{ dashboard.users.total_users }} کاربر</span>
                        </div>
                    </template>

                    <b-row class="g-2 mb-3">
                        <b-col cols="4" sm="3" v-for="(value, key) in userStats" :key="key">
                            <div class="stat-item">
                                <div class="stat-item-label">{{ userLabels[key] }}</div>
                                <div class="stat-item-value">{{ formatNumber(value) }}</div>
                            </div>
                        </b-col>
                    </b-row>

                    <div class="chart-wrapper">
                        <ApexChart 
                            type="donut" 
                            height="200" 
                            :options="userChartOptions" 
                            :series="userSeries" 
                        />
                    </div>
                </b-card>
            </b-col>

            <!-- دیدگاه‌ها -->
            <b-col cols="12" lg="6" class="mb-4">
                <b-card class="dashboard-card h-100">
                    <template #header>
                        <div class="d-flex justify-content-between align-items-center">
                            <h5 class="mb-0">💬 دیدگاه‌ها</h5>
                            <span class="badge bg-warning">{{ dashboard.comments.total_comments }} دیدگاه</span>
                        </div>
                    </template>

                    <b-row class="g-2 mb-3">
                        <b-col cols="4" sm="3" v-for="(value, key) in commentStats" :key="key">
                            <div class="stat-item">
                                <div class="stat-item-label">{{ commentLabels[key] }}</div>
                                <div class="stat-item-value">{{ formatNumber(value) }}</div>
                            </div>
                        </b-col>
                    </b-row>

                    <div class="chart-wrapper">
                        <ApexChart 
                            type="bar" 
                            height="200" 
                            :options="commentChartOptions" 
                            :series="commentSeries" 
                        />
                    </div>
                </b-card>
            </b-col>
        </b-row>
    </b-container>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import ApexChart from "vue3-apexcharts";
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
const dashboard = ref({
    orders: {},
    products: {},
    users: {},
    comments: {},
});

const currentTime = ref("");

// برچسب‌ها فارسی
const orderLabels = {
    total_orders: "کل سفارش‌ها",
    total_sales: "فروش کل",
    today_orders: "امروز",
    month_orders: "این ماه",
    average_order_value: "میانگین",
    max_order_value: "بیشترین",
    min_order_value: "کمترین",
};

const productLabels = {
    total_products: "کل محصولات",
    active_products: "فعال",
    inactive_products: "غیرفعال",
    out_of_stock: "ناموجود",
    average_price: "میانگین قیمت",
    max_price: "بیشترین قیمت",
    min_price: "کمترین قیمت",
};

const userLabels = {
    total_users: "کل کاربران",
    with_addresses: "با آدرس",
    with_wallet: "کیف پول دارد",
    without_wallet: "بدون کیف پول",
    today_registered: "ثبت‌نام امروز",
};

const commentLabels = {
    total_comments: "کل دیدگاه‌ها",
    approved: "تأیید شده",
    pending: "در انتظار",
    rejected: "رد شده",
    with_rating: "با امتیاز",
    average_rating: "میانگین امتیاز",
    today_comments: "امروز",
    this_month: "این ماه",
};

// محاسبه آمارها
const orderStats = computed(() => {
    const o = dashboard.value.orders;
    return {
        total_orders: o.total_orders || 0,
        total_sales: o.total_sales || 0,
        today_orders: o.today_orders || 0,
        month_orders: o.month_orders || 0,
        average_order_value: o.average_order_value || 0,
        max_order_value: o.max_order_value || 0,
        min_order_value: o.min_order_value || 0,
    };
});

const productStats = computed(() => {
    const p = dashboard.value.products;
    return {
        total_products: p.total_products || 0,
        active_products: p.active_products || 0,
        inactive_products: p.inactive_products || 0,
        out_of_stock: p.out_of_stock || 0,
        average_price: p.average_price || 0,
        max_price: p.max_price || 0,
        min_price: p.min_price || 0,
    };
});

const userStats = computed(() => {
    const u = dashboard.value.users;
    return {
        total_users: u.total_users || 0,
        with_addresses: u.with_addresses || 0,
        with_wallet: u.with_wallet || 0,
        without_wallet: u.without_wallet || 0,
        today_registered: u.today_registered || 0,
    };
});

const commentStats = computed(() => {
    const c = dashboard.value.comments;
    return {
        total_comments: c.total_comments || 0,
        approved: c.approved || 0,
        pending: c.pending || 0,
        rejected: c.rejected || 0,
        with_rating: c.with_rating || 0,
        average_rating: c.average_rating || 0,
        today_comments: c.today_comments || 0,
        this_month: c.this_month || 0,
    };
});

// کارت‌های آماری اصلی
const mainStats = computed(() => [
    { 
        label: "کل سفارش‌ها", 
        value: formatNumber(dashboard.value.orders.total_orders), 
        icon: "bi bi-bag",  // آیکون کیف/سفارش
        color: "stat-primary" 
    },
    { 
        label: "کل فروش", 
        value: formatPrice(dashboard.value.orders.total_sales), 
        icon: "bi bi-currency-dollar",  // آیکون دلار/پول
        color: "stat-success" 
    },
    { 
        label: "کل کاربران", 
        value: formatNumber(dashboard.value.users.total_users), 
        icon: "bi bi-people",  // آیکون گروه کاربران
        color: "stat-info" 
    },
    { 
        label: "کل محصولات", 
        value: formatNumber(dashboard.value.products.total_products), 
        icon: "bi bi-box-seam",  // آیکون جعبه/محصول
        color: "stat-warning" 
    },
]);

// نمودار سفارش‌ها
const orderSeries = ref([{ name: "سفارش‌ها", data: [] }]);
const orderChartOptions = ref({
    chart: { 
        id: "orders",
        toolbar: { show: false },
        sparkline: { enabled: false }
    },
    xaxis: { 
        categories: [],
        labels: { rotate: -45 }
    },
    yaxis: {
        labels: {
            formatter: (val) => val.toLocaleString()
        }
    },
    stroke: { curve: 'smooth', width: 3 },
    colors: ['#6c5ce7'],
    grid: { show: false },
    tooltip: {
        y: {
            formatter: (val) => val.toLocaleString()
        }
    }
});

// نمودار محصولات
const productSeries = ref([]);
const productChartOptions = ref({
    labels: ["فعال", "غیرفعال", "ناموجود"],
    colors: ['#00b894', '#fdcb6e', '#e17055'],
    legend: { position: 'bottom' },
    dataLabels: { enabled: false }
});

// نمودار کاربران
const userSeries = ref([]);
const userChartOptions = ref({
    labels: ["دارای کیف پول", "بدون کیف پول"],
    colors: ['#0984e3', '#dfe6e9'],
    legend: { position: 'bottom' },
    dataLabels: { enabled: false }
});

// نمودار دیدگاه‌ها
const commentSeries = ref([{ name: "دیدگاه‌ها", data: [] }]);
const commentChartOptions = ref({
    chart: { 
        id: "comments",
        toolbar: { show: false }
    },
    xaxis: { 
        categories: ["تأیید شده", "در انتظار", "رد شده"] 
    },
    colors: ['#00b894', '#fdcb6e', '#e17055'],
    grid: { show: false },
    plotOptions: {
        bar: { borderRadius: 4 }
    }
});

// توابع کمکی
function formatNumber(value) {
    if (!value && value !== 0) return "0";
    return Number(value).toLocaleString('fa-IR');
}

function formatPrice(value) {
    if (!value && value !== 0) return "0";
    return Number(value).toLocaleString('fa-IR') + " تومان";
}

// دریافت داده از API
onMounted(async () => {
    try {
        const { data } = await axios.get("/dashboard");
        dashboard.value = data.data;

        // زمان جاری
        const now = new Date();
        currentTime.value = now.toLocaleString('fa-IR');

        // بروزرسانی نمودار سفارش‌ها با داده‌های ماهانه
        const monthlyData = dashboard.value.orders.monthly_daily_breakdown || [];
        const dates = monthlyData.map(item => item.date);
        const counts = monthlyData.map(item => item.count);
        const sales = monthlyData.map(item => Number(item.total_sales));

        orderSeries.value = [
            { name: "تعداد سفارش‌ها", data: counts },
            { name: "فروش (تومان)", data: sales }
        ];
        orderChartOptions.value.xaxis.categories = dates;

        // نمودار محصولات
        productSeries.value = [
            dashboard.value.products.active_products || 0,
            dashboard.value.products.inactive_products || 0,
            dashboard.value.products.out_of_stock || 0,
        ];

        // نمودار کاربران
        userSeries.value = [
            dashboard.value.users.with_wallet || 0,
            dashboard.value.users.without_wallet || 0,
        ];

        // نمودار دیدگاه‌ها
        commentSeries.value[0].data = [
            dashboard.value.comments.approved || 0,
            dashboard.value.comments.pending || 0,
            dashboard.value.comments.rejected || 0,
        ];

    } catch (error) {
        console.error("خطا در دریافت داده‌های داشبورد:", error);
    }
});
</script>

<style scoped>
/* هدر داشبورد */
.dashboard-header {
    padding: 10px 0;
    border-bottom: 2px solid #f0f0f0;
}

.dashboard-header h2 {
    font-weight: 700;
    color: #2d3436;
}

/* کارت‌های آماری اصلی */
.stat-card {
    border: none;
    border-radius: 12px;
    transition: all 0.3s ease;
    cursor: default;
}

.stat-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0,0,0,0.1);
}

.stat-primary {
    background: linear-gradient(135deg, #6c5ce7, #a29bfe);
    color: white;
}

.stat-success {
    background: linear-gradient(135deg, #00b894, #55efc4);
    color: white;
}

.stat-info {
    background: linear-gradient(135deg, #0984e3, #74b9ff);
    color: white;
}

.stat-warning {
    background: linear-gradient(135deg, #fdcb6e, #f39c12);
    color: white;
}

.stat-icon {
    width: 50px;
    height: 50px;
    background: rgba(255,255,255,0.2);
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 24px;
}

.stat-content {
    flex: 1;
}

.stat-number {
    font-size: 22px;
    font-weight: 700;
    line-height: 1.2;
}

.stat-label {
    font-size: 13px;
    opacity: 0.9;
}

/* کارت‌های داشبورد */
.dashboard-card {
    border: none;
    border-radius: 16px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.06);
    transition: all 0.3s ease;
}

.dashboard-card:hover {
    box-shadow: 0 8px 30px rgba(0,0,0,0.1);
}

.dashboard-card .card-header {
    background: transparent;
    border-bottom: 2px solid #f8f9fa;
    padding: 16px 20px;
}

.dashboard-card .card-header h5 {
    font-weight: 600;
    color: #2d3436;
}

.dashboard-card .card-body {
    padding: 20px;
}

/* آیتم‌های آماری */
.stat-item {
    background: #f8f9fa;
    border-radius: 8px;
    padding: 8px 6px;
    text-align: center;
    transition: all 0.2s ease;
}

.stat-item:hover {
    background: #e9ecef;
}

.stat-item-label {
    font-size: 10px;
    color: #6c757d;
    margin-bottom: 2px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.stat-item-value {
    font-size: 13px;
    font-weight: 700;
    color: #2d3436;
}

/* محفظه نمودار */
.chart-wrapper {
    margin-top: 8px;
}

/* Badge‌های هدر */
.badge {
    font-size: 12px;
    padding: 6px 14px;
    border-radius: 20px;
    font-weight: 500;
}

/* واکنش‌گرایی */
@media (max-width: 768px) {
    .stat-number {
        font-size: 18px;
    }
    
    .stat-icon {
        width: 40px;
        height: 40px;
        font-size: 18px;
    }
    
    .dashboard-card .card-body {
        padding: 12px;
    }
    
    .stat-item-value {
        font-size: 11px;
    }
    
    .stat-item-label {
        font-size: 9px;
    }
}
</style>