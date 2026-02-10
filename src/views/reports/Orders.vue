<template>
    <div class="container py-4">
        <!-- فیلترها -->
        <div class="card mb-4">
            <div class="card-body">
                <form @submit.prevent="getReport()" class="row g-3">
                    <div class="col-md-2">
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                            v-model="filters.start_date"></date-picker>

                    </div>
                    <div class="col-md-2">
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                            v-model="filters.end_date"></date-picker>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.status" class="form-select">
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
                        <select v-model="filters.payment_method" class="form-select">
                            <option value="">همه روش‌های پرداخت</option>
                            <option value="online">پرداخت آنلاین</option>
                            <option value="wallet">کیف پول</option>
                            <option value="cod">پرداخت در محل</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.payment_status" class="form-select">
                            <option value="">همه وضعیت‌های پرداخت</option>
                            <option value="pending">در انتظار پرداخت</option>
                            <option value="paid">پرداخت شده</option>
                            <option value="failed">ناموفق</option>
                            <option value="refunded">برگشت داده شده</option>
                        </select>
                    </div>
                    <div class="col-12">
                        <button type="submit" class="btn btn-primary w-100">
                            اعمال فیلتر
                        </button>
                    </div>
                </form>
            </div>
        </div>
        <div v-if="loading" class="text-center my-5">
            <div class="spinner-border" role="status"></div>
            <p class="mt-2">در حال بارگذاری...</p>
        </div>
        <div v-else class="card">
            <div class="card-body">
                <!-- عنوان + سوییچر -->
                <div class="d-flex justify-content-between align-items-center mb-3">
                    <h2 class="h5 mb-0">گزارش فروش</h2>
                    <div class="btn-group">
                        <button @click="viewMode = 'chart'"
                            :class="['btn', viewMode === 'chart' ? 'btn-primary' : 'btn-outline-primary']"
                            title="نمایش نمودار">
                            📊
                        </button>
                        <button @click="viewMode = 'table'"
                            :class="['btn', viewMode === 'table' ? 'btn-primary' : 'btn-outline-primary']"
                            title="نمایش جدول">
                            📋
                        </button>
                    </div>
                </div>

                <!-- نمودار -->
                <div v-if="viewMode === 'chart'">
                    <BarChart v-if="chartData" :chart-data="chartData" />
                </div>

                <!-- جدول -->
                <div v-else>
                    <div class="table-responsive">
                        <table class="table table-striped text-center align-middle">
                            <thead class="table-light">
                                <tr>
                                    <th>شناسه</th>
                                    <th>کاربر</th>
                                    <th>مبلغ کل</th>
                                    <th>وضعیت</th>
                                    <th>پرداخت</th>
                                    <th>تاریخ</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="order in tableData.data" :key="order.id">
                                    <td>{{ order.id }}</td>
                                    <td>{{ order.user?.name }}</td>
                                    <td>{{ order.total.toLocaleString() }} تومان</td>
                                    <td>{{ translateStatus(order.status) }}</td>
                                    <td>
                                        {{ translatePaymentMethod(order.payment_method) }}
                                        ({{ translatePaymentStatus(order.payment_status) }})
                                    </td>
                                    <td>{{ new Date(order.created_at).toLocaleDateString("fa-IR") }}</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                    <b-pagination v-model="currentPage" :total-rows="tableData.total" v-if="tableData.last_page != 1"
                        :per-page="tableData.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3"></b-pagination>

                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from "chart.js";
import { Bar } from "vue-chartjs";
const currentPage = ref(1)
const loading = ref(false)

ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);

const BarChart = {
    props: ["chartData"],
    components: { Bar },
    template: `<Bar :data="chartData" :options="{ responsive: true, plugins: { legend: { display: false } } }" />`,
};

const filters = ref({
    start_date: "",
    end_date: "",
    status: "",
    payment_method: "",
    payment_status: "",
});

const tableData = ref({ data: [] });
const chartData = ref(null);
const viewMode = ref("table"); // پیش‌فرض روی نمودار

const getReport = async (page = 1) => {
    loading.value = true;
    try {

        const { data } = await axios.get("/reports/sales?page=" + page, {
            params: { ...filters.value },
        });
        tableData.value = data.table;
        currentPage.value = data.table.current_page

        chartData.value = {
            labels: data.chart.map((c) => c.date),
            datasets: [
                {
                    label: "میزان فروش",
                    data: data.chart.map((c) => c.total_sales),
                    backgroundColor: "#3b82f6",
                },
            ],
        };
    } catch (error) {
        console.log(error);

    } finally {
        loading.value = false
    }

};

const changePage = (page) => {
    if (page > 0) {
        getReport(page);
    }
};

const translateStatus = (status) => {
    const map = {
        pending: "در انتظار",
        reserved: "رزرو شده",
        processing: "در حال پردازش",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        returned: "مرجوعی",
    };
    return map[status] || status;
};

const translatePaymentMethod = (method) => {
    const map = {
        online: "پرداخت آنلاین",
        wallet: "کیف پول",
        cod: "پرداخت در محل",
    };
    return map[method] || method;
};

const translatePaymentStatus = (status) => {
    const map = {
        pending: "در انتظار پرداخت",
        paid: "پرداخت شده",
        failed: "ناموفق",
        refunded: "برگشت داده شده",
    };
    return map[status] || status;
};

getReport();
</script>