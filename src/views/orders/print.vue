```vue
<template>
    <div class="print-wrapper" v-if="checkPermission(['order_view'])">

        <!-- ================= TOOLBAR ================= -->
        <div class="print-toolbar">
            <div class="toolbar-content">

                <div class="toolbar-brand">
                    <div class="brand-mark overflow-hidden">
                        <img src="@/assets/images/logo.png" width="72" alt="">
                    </div>

                    <div>
                        <strong>حدیث اسکارف</strong>
                        <span>مرکز مدیریت سفارشات</span>
                    </div>
                </div>

                <div class="toolbar-meta">
                    <div class="toolbar-chip">
                        <i class="bi bi-receipt"></i>
                        <span>{{ orders.length }} سفارش</span>
                    </div>

                    <div class="toolbar-chip">
                        <i class="bi bi-printer"></i>
                        <span>{{ printType === 'full' ? 'فاکتور سفارش' : 'برچسب ارسال' }}</span>
                    </div>
                </div>

                <div class="toolbar-actions">
                    <button @click="handlePrint" class="btn-print">
                        <i class="bi bi-printer-fill"></i>
                        <span>چاپ</span>
                    </button>

                    <router-link to="/orders" class="btn-back">
                        <i class="bi bi-arrow-right"></i>
                        <span>بازگشت</span>
                    </router-link>
                </div>

            </div>
        </div>

        <!-- ================= LOADING ================= -->
        <div v-if="loading" class="loading-state">
            <div class="loading-box">
                <div class="loading-spinner"></div>
                <strong>در حال آماده‌سازی سفارش‌ها</strong>
                <span>لطفاً چند لحظه صبر کنید...</span>
            </div>
        </div>

        <!-- ================= EMPTY ================= -->
        <div v-else-if="orders.length === 0" class="empty-state">
            <div class="empty-icon">
                <i class="bi bi-inbox"></i>
            </div>
            <h3>سفارشی برای نمایش وجود ندارد</h3>
            <p>لطفاً سفارش‌های مورد نظر را انتخاب کنید.</p>

            <router-link to="/orders" class="empty-button">
                <i class="bi bi-arrow-right"></i>
                بازگشت به سفارش‌ها
            </router-link>
        </div>

        <!-- ================= PRINT ================= -->
        <div id="printSection">



            <div class="print-body">

                <!-- ================= FULL PRINT ================= -->
                <template v-if="printType === 'full'">

                    <div v-for="(order, index) in orders" :key="order.id" class="order-card">

                        <!-- ORDER HEADER -->
                        <div class="order-header">

                            <div class="order-main">

                                <div class="order-id-box">
                                    <small>شماره سفارش</small>
                                    <strong>#{{ String(order.id).padStart(5, '0') }}</strong>
                                </div>

                                <div class="order-statuses">
                                    <span class="status-badge" :class="statusClass(order.status)">
                                        <i class="bi bi-circle-fill"></i>
                                        {{ statusText(order.status) }}
                                    </span>

                                    <span class="payment-badge" :class="paymentClass(order.payment_status)">
                                        <i class="bi bi-credit-card"></i>
                                        {{ paymentStatusText(order.payment_status) }}
                                    </span>
                                </div>

                            </div>

                            <div class="order-date">
                                <span>تاریخ ثبت</span>
                                <strong>
                                    <i class="bi bi-calendar3"></i>
                                    {{ formatDate(order.created_at) }}
                                </strong>
                            </div>

                        </div>

                        <div class="order-body">

                            <!-- INFO -->
                            <div class="info-grid">

                                <!-- CUSTOMER -->
                                <div class="info-card">
                                    <div class="info-card-head">
                                        <div class="info-icon">
                                            <i class="bi bi-person-fill"></i>
                                        </div>
                                        <span>اطلاعات گیرنده</span>
                                    </div>

                                    <div class="info-card-content">
                                        <strong>
                                            {{ order.user?.full_name ?? '-' }}
                                        </strong>

                                        <div class="info-line">
                                            <i class="bi bi-phone"></i>
                                            <span>{{ order.user?.mobile ?? '-' }}</span>
                                        </div>
                                    </div>
                                </div>

                                <!-- ADDRESS -->
                                <div class="info-card address-card">
                                    <div class="info-card-head">
                                        <div class="info-icon">
                                            <i class="bi bi-geo-alt-fill"></i>
                                        </div>
                                        <span>آدرس تحویل</span>
                                    </div>

                                    <div class="info-card-content">
                                        <strong class="address-text">
                                            {{ order.address?.address_line ?? '-' }}
                                        </strong>

                                        <div class="info-line">
                                            <i class="bi bi-pin-map"></i>
                                            <span>
                                                {{ order.address?.province?.name ?? '-' }}
                                                -
                                                {{ order.address?.city?.name ?? '-' }}
                                            </span>
                                        </div>

                                        <div class="info-line">
                                            <i class="bi bi-upc-scan"></i>
                                            <span>
                                                کد پستی:
                                                {{ order.address?.postal_code ?? '-' }}
                                            </span>
                                        </div>
                                    </div>
                                </div>

                                <!-- SHIPPING -->
                                <div class="info-card">
                                    <div class="info-card-head">
                                        <div class="info-icon">
                                            <i class="bi bi-truck-front-fill"></i>
                                        </div>
                                        <span>ارسال و پرداخت</span>
                                    </div>

                                    <div class="info-card-content">
                                        <strong>
                                            {{ order.shipping?.title ?? '-' }}
                                        </strong>

                                        <div class="info-line">
                                            <i class="bi bi-box-seam"></i>
                                            <span>
                                                ارسال:
                                                {{ Number(order.shipping_cost).toLocaleString('fa-IR') }}
                                                تومان
                                            </span>
                                        </div>

                                        <div class="info-line">
                                            <i class="bi bi-wallet2"></i>
                                            <span>
                                                {{ paymentMethodText(order.payment_method) }}
                                            </span>
                                        </div>
                                    </div>
                                </div>

                            </div>

                            <!-- PRODUCTS -->
                            <div class="section-title">
                                <div>
                                    <span>جزئیات سفارش</span>
                                    <small>محصولات سفارش داده شده</small>
                                </div>
                            </div>

                            <div class="products-table">
                                <table>
                                    <thead>
                                        <tr>
                                            <th class="index-col">#</th>
                                            <th class="product-col">محصول</th>
                                            <th>مشخصات</th>
                                            <th class="qty-col">تعداد</th>
                                            <th>قیمت واحد</th>
                                            <th>مبلغ کل</th>
                                        </tr>
                                    </thead>

                                    <tbody>
                                        <tr v-for="(item, idx) in order.items" :key="item.id">
                                            <td class="index-cell">
                                                {{ String(idx + 1).padStart(2, '0') }}
                                            </td>

                                            <td class="product-name">
                                                {{ item.product?.title ?? '-' }}
                                            </td>

                                            <td class="variants">
                                                <template v-if="item.variant?.values?.length">
                                                    <span v-for="val in item.variant.values" :key="val.id"
                                                        class="variant-tag">
                                                        {{ val.attribute?val.attribute.name+" : ":"" }}
                                                        {{ val.value }}
                                                    </span>
                                                </template>
                                                <span v-else class="muted">بدون مشخصات</span>
                                            </td>

                                            <td class="quantity">
                                                <span>{{ item.quantity }}</span>
                                            </td>

                                            <td>
                                                {{ Number(item.price).toLocaleString('fa-IR') }}
                                                <small>تومان</small>
                                            </td>

                                            <td class="total-price">
                                                {{ Number(item.price * item.quantity).toLocaleString('fa-IR') }}
                                                <small>تومان</small>
                                            </td>
                                        </tr>
                                    </tbody>
                                </table>
                            </div>

                            <!-- SUMMARY -->
                            <div class="bottom-section">

                                <div class="thank-message">
                                    <div class="thank-icon">
                                        <i class="bi bi-heart-fill"></i>
                                    </div>

                                    <div>
                                        <strong>از خرید شما سپاسگزاریم</strong>
                                        <span>حدیث اسکارف؛ انتخابی برای سلیقه شما</span>
                                    </div>
                                </div>

                                <div class="summary">

                                    <div class="summary-row">
                                        <span>جمع محصولات</span>
                                        <strong>
                                            {{ Number(order.subtotal).toLocaleString('fa-IR') }}
                                            <small>تومان</small>
                                        </strong>
                                    </div>

                                    <div class="summary-row discount-row">
                                        <span>
                                            تخفیف
                                            <i class="bi bi-tag-fill"></i>
                                        </span>

                                        <strong>
                                            -
                                            {{ Number(order.discount_amount).toLocaleString('fa-IR') }}
                                            <small>تومان</small>
                                        </strong>
                                    </div>

                                    <div class="summary-row">
                                        <span>هزینه ارسال</span>
                                        <strong>
                                            {{ Number(order.shipping_cost).toLocaleString('fa-IR') }}
                                            <small>تومان</small>
                                        </strong>
                                    </div>

                                    <div class="summary-total">
                                        <div>
                                            <span>مبلغ نهایی</span>
                                            <small>قابل پرداخت</small>
                                        </div>

                                        <strong>
                                            {{ Number(order.total).toLocaleString('fa-IR') }}
                                            <small>تومان</small>
                                        </strong>
                                    </div>

                                </div>

                            </div>

                        </div>

                        <div v-if="index < orders.length - 1" class="page-separator">
                            <span></span>
                            <i class="bi bi-scissors"></i>
                            <span></span>
                        </div>

                    </div>

                </template>

                <!-- ================= LABEL ================= -->
                <template v-else-if="printType === 'label'">

                    <div class="labels-grid">
                        <template v-for="order in orders" :key="order.id">
                            <div class="label-card">

                                <div class="label-top">

                                    <div class="label-brand">
                                        <div class="mini-logo overflow-hidden">
                                            <img src="@/assets/images/logo.png" width="72" alt="">

                                        </div>

                                        <div>
                                            <strong>
                                                اطلاعات فرستنده:
                                            </strong>
                                            <span>
                                                حدیث اسکارف

                                            </span>
                                        </div>
                                    </div>

                                    <div class="label-number">
                                        <small>website</small>
                                        <strong>www.hadis-scarf.ir</strong>
                                    </div>

                                </div>

                                <div class="label-route">

                                    <div class="route-title">
                                        <span class="route-line"></span>
                                        <i class="bi bi-arrow-down-circle-fill"></i>
                                        <span>ارسال از</span>
                                    </div>

                                    <div class="receiver-name">
                                        فروشگاه حدیث اسکارف
                                    </div>

                                    <div class="receiver-phone">
                                        <i class="bi bi-telephone-fill"></i>
                                        ۰۹۳۳۵۸۱۴۴۷۵
                                    </div>

                                    <div class="receiver-address">
                                        آدرس: گلستان گرگان -سر سه راهی سلامتی
                                    </div>

                                    <div class="receiver-location">
                                        <span>
                                            گلستان
                                            -
                                            گرگان
                                        </span>

                                        <strong>
                                            کد پستی:
                                            ------
                                        </strong>
                                    </div>

                                </div>

                                <div class="label-bottom">

                                    <div>
                                        <small>Hadis Scarf</small>
                                        <strong>
                                            فروشگاه شال و روسی
                                        </strong>
                                    </div>

                                    <div class="label-price">
                                        <small>تاریخ سفارش</small>
                                        <strong>
                                            {{ new Date(order.created_at).toLocaleDateString('fa-IR') }}
                                        </strong>
                                    </div>

                                </div>

                            </div>
                            <div class="label-card">

                                <div class="label-top">

                                    <div class="label-brand">
                                        <div class="mini-logo">
                                            <i class="bi bi-bag-heart-fill"></i>
                                        </div>

                                        <div>
                                            <strong>اطلاعات گیرنده:</strong>
                                            <span>SHIPPING LABEL</span>
                                        </div>
                                    </div>

                                    <div class="label-number">
                                        <small>شناسه سفارش</small>
                                        <strong>#{{ String(order.id).padStart(5, '0') }}</strong>
                                    </div>

                                </div>

                                <div class="label-route">

                                    <div class="route-title">
                                        <span class="route-line"></span>
                                        <i class="bi bi-arrow-down-circle-fill"></i>
                                        <span>ارسال به</span>
                                    </div>

                                    <div class="receiver-name">
                                        {{ order.user?.full_name ?? '-' }}
                                    </div>

                                    <div class="receiver-phone">
                                        <i class="bi bi-telephone-fill"></i>
                                        {{ order.user?.mobile ?? '-' }}
                                    </div>

                                    <div class="receiver-address">
                                        {{ order.address?.address_line ?? '-' }}
                                    </div>

                                    <div class="receiver-location">
                                        <span>
                                            {{ order.address?.province?.name ?? '-' }}
                                            -
                                            {{ order.address?.city?.name ?? '-' }}
                                        </span>

                                        <strong>
                                            کد پستی:
                                            {{ order.address?.postal_code ?? '-' }}
                                        </strong>
                                    </div>

                                </div>

                                <div class="label-bottom">

                                    <div>
                                        <small>روش ارسال</small>
                                        <strong>
                                            {{ order.shipping?.title ?? '-' }}
                                        </strong>
                                    </div>

                                    <div class="label-price">
                                        <small>مبلغ سفارش</small>
                                        <strong>
                                            {{ Number(order.total).toLocaleString('fa-IR') }}
                                            <em>تومان</em>
                                        </strong>
                                    </div>

                                </div>

                            </div>
                        </template>

                    </div>

                </template>

            </div>

            <!-- PRINT FOOTER -->
            <div class="print-footer">
                <div class="footer-line"></div>
                <strong>حدیث اسکارف</strong>
                <span>این فاکتور توسط سیستم مدیریت سفارشات صادر شده است.</span>
                <small>www.hadis-scarf.ir</small>
            </div>

        </div>

    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import axios from "axios";
import { useAdmin } from '@/stores/modules/admin';

const route = useRoute();
const router = useRouter();
const store = useAdmin();
const checkPermission = store.checkPermission;

const orders = ref([]);
const loading = ref(false);
const orderIds = ref([]);
const printType = ref('full');

const handlePrint = () => {
    const printContents = document.getElementById('printSection').innerHTML;

    const iframe = document.createElement('iframe');

    iframe.style.position = 'fixed';
    iframe.style.right = '0';
    iframe.style.bottom = '0';
    iframe.style.width = '0';
    iframe.style.height = '0';
    iframe.style.border = 'none';

    document.body.appendChild(iframe);

    const doc = iframe.contentWindow.document;

    doc.open();

    doc.write(`
        <!DOCTYPE html>
        <html dir="rtl">
        <head>
            <meta charset="UTF-8">
            <title>پرینت سفارش‌ها</title>

            <link
                rel="stylesheet"
                href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.0/font/bootstrap-icons.css"
            >

            <style>

                * {
                    margin: 0;
                    padding: 0;
                    box-sizing: border-box;
                }

                @page {
                    size: A4;
                    margin: 12mm;
                }

                body {
                    direction: rtl;
                    font-family:
                        IRANSans,
                        "IRANSansX",
                        Tahoma,
                        Arial,
                        sans-serif;

                    background: #fff;
                    color: #18181b;
                    line-height: 1.7;
                    font-size: 13px;
                }

                .print-header {
                    display: flex !important;
                    align-items: center;
                    justify-content: space-between;
                    padding-bottom: 22px;
                    border-bottom: 1px solid #e4e4e7;
                    margin-bottom: 20px;
                }

                .invoice-brand {
                    display: flex;
                    align-items: center;
                    gap: 12px;
                }

                .invoice-logo {
                    width: 48px;
                    height: 48px;
                    border: 1px solid #18181b;
                    border-radius: 14px;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    font-size: 20px;
                }

                .invoice-brand h1 {
                    font-size: 21px;
                    margin: 0;
                    font-weight: 900;
                    color: #111113;
                }

                .invoice-brand p {
                    margin: 2px 0 0;
                    color: #71717a;
                    font-size: 10px;
                }

                .invoice-meta {
                    display: flex;
                    gap: 30px;
                    text-align: right;
                }

                .invoice-meta-item span {
                    display: block;
                    color: #a1a1aa;
                    font-size: 9px;
                    margin-bottom: 2px;
                }

                .invoice-meta-item strong {
                    font-size: 12px;
                    font-weight: 800;
                }

                .print-title {
                    display: flex;
                    align-items: center;
                    gap: 12px;
                    margin: 20px 0;
                }

                .print-title > span {
                    flex: 1;
                    height: 1px;
                    background: #e4e4e7;
                }

                .print-title div {
                    text-align: center;
                }

                .print-title small {
                    display: block;
                    color: #a1a1aa;
                    font-size: 8px;
                    letter-spacing: 2px;
                    direction: ltr;
                }

                .print-title strong {
                    display: block;
                    font-size: 15px;
                    font-weight: 900;
                }

                .order-card {
                    border: 1px solid #e4e4e7;
                    border-radius: 14px;
                    overflow: hidden;
                    margin-bottom: 25px;
                    page-break-inside: avoid;
                    break-inside: avoid;
                }

                .order-header {
                    display: flex;
                    align-items: center;
                    justify-content: space-between;
                    padding: 15px 18px;
                    background: #fafafa;
                    border-bottom: 1px solid #e4e4e7;
                }

                .order-main {
                    display: flex;
                    align-items: center;
                    gap: 18px;
                }

                .order-id-box small {
                    display: block;
                    font-size: 8px;
                    color: #a1a1aa;
                }

                .order-id-box strong {
                    font-size: 17px;
                    font-weight: 900;
                }

                .order-statuses {
                    display: flex;
                    gap: 5px;
                }

                .status-badge,
                .payment-badge {
                    display: inline-flex;
                    align-items: center;
                    gap: 5px;
                    border-radius: 999px;
                    padding: 4px 9px;
                    font-size: 9px;
                    font-weight: 700;
                    background: #f4f4f5;
                    color: #52525b;
                }

                .status-badge i {
                    font-size: 5px;
                }

                .status-completed,
                .payment-paid {
                    background: #ecfdf5;
                    color: #047857;
                }

                .status-pending,
                .payment-pending {
                    background: #fffbeb;
                    color: #b45309;
                }

                .status-processing {
                    background: #eff6ff;
                    color: #1d4ed8;
                }

                .status-shipped {
                    background: #f5f3ff;
                    color: #6d28d9;
                }

                .status-canceled,
                .payment-failed {
                    background: #fef2f2;
                    color: #b91c1c;
                }

                .status-returned,
                .payment-refunded {
                    background: #f4f4f5;
                    color: #52525b;
                }

                .order-date {
                    text-align: left;
                }

                .order-date span {
                    display: block;
                    color: #a1a1aa;
                    font-size: 8px;
                }

                .order-date strong {
                    display: block;
                    font-size: 10px;
                }

                .order-body {
                    padding: 18px;
                }

                .info-grid {
                    display: grid;
                    grid-template-columns: 1fr 1.4fr 1fr;
                    gap: 10px;
                    margin-bottom: 22px;
                }

                .info-card {
                    border: 1px solid #ededed;
                    border-radius: 10px;
                    padding: 12px;
                    background: #fff;
                    min-height: 105px;
                }

                .info-card-head {
                    display: flex;
                    align-items: center;
                    gap: 7px;
                    padding-bottom: 8px;
                    margin-bottom: 8px;
                    border-bottom: 1px solid #f1f1f1;
                    font-size: 9px;
                    color: #71717a;
                    font-weight: 700;
                }

                .info-icon {
                    width: 26px;
                    height: 26px;
                    border-radius: 8px;
                    background: #18181b;
                    color: #fff;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    font-size: 11px;
                }

                .info-card-content > strong {
                    display: block;
                    font-size: 11px;
                    font-weight: 800;
                    margin-bottom: 5px;
                }

                .info-line {
                    display: flex;
                    align-items: center;
                    gap: 5px;
                    color: #52525b;
                    font-size: 9px;
                    margin-top: 3px;
                }

                .info-line i {
                    color: #a1a1aa;
                }

                .address-text {
                    line-height: 1.8;
                }

                .section-title {
                    display: flex;
                    align-items: center;
                    gap: 10px;
                    margin-bottom: 9px;
                }

                .section-title:before {
                    content: "";
                    display: block;
                    width: 3px;
                    height: 25px;
                    background: #18181b;
                    border-radius: 5px;
                }

                .section-title span {
                    display: block;
                    font-size: 11px;
                    font-weight: 900;
                }

                .section-title small {
                    display: block;
                    color: #a1a1aa;
                    font-size: 8px;
                }

                .products-table {
                    border: 1px solid #e4e4e7;
                    border-radius: 9px;
                    overflow: hidden;
                }

                .products-table table {
                    width: 100%;
                    border-collapse: collapse;
                }

                .products-table thead {
                    background: #18181b;
                    color: #fff;
                }

                .products-table th {
                    padding: 8px 7px;
                    font-size: 8px;
                    font-weight: 700;
                    white-space: nowrap;
                }

                .products-table td {
                    padding: 9px 7px;
                    border-bottom: 1px solid #f0f0f0;
                    text-align: center;
                    font-size: 9px;
                }

                .products-table tr:last-child td {
                    border-bottom: 0;
                }

                .product-col {
                    width: 30%;
                }

                .product-name {
                    text-align: right !important;
                    font-weight: 800;
                }

                .index-cell {
                    color: #a1a1aa;
                    font-weight: 700;
                }

                .variant-tag {
                    display: inline-block;
                    background: #f4f4f5;
                    border: 1px solid #e4e4e7;
                    padding: 2px 6px;
                    border-radius: 5px;
                    font-size: 7px;
                    margin: 1px;
                }

                .muted {
                    color: #a1a1aa;
                }

                .quantity span {
                    display: inline-flex;
                    min-width: 23px;
                    height: 23px;
                    align-items: center;
                    justify-content: center;
                    border-radius: 6px;
                    background: #f4f4f5;
                    font-weight: 800;
                }

                .products-table td small {
                    color: #a1a1aa;
                    font-size: 7px;
                }

                .total-price {
                    font-weight: 900;
                }

                .bottom-section {
                    display: flex;
                    align-items: flex-end;
                    justify-content: space-between;
                    gap: 25px;
                    margin-top: 16px;
                }

                .thank-message {
                    display: flex;
                    align-items: center;
                    gap: 9px;
                    color: #52525b;
                }

                .thank-icon {
                    width: 34px;
                    height: 34px;
                    border-radius: 10px;
                    background: #18181b;
                    color: white;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                }

                .thank-message strong {
                    display: block;
                    font-size: 10px;
                    color: #18181b;
                }

                .thank-message span {
                    display: block;
                    font-size: 8px;
                    color: #a1a1aa;
                }

                .summary {
                    width: 300px;
                    border: 1px solid #e4e4e7;
                    border-radius: 10px;
                    overflow: hidden;
                }

                .summary-row {
                    display: flex;
                    align-items: center;
                    justify-content: space-between;
                    padding: 7px 11px;
                    font-size: 9px;
                    border-bottom: 1px solid #f1f1f1;
                }

                .summary-row strong {
                    font-weight: 800;
                }

                .summary-row small {
                    font-size: 7px;
                    color: #a1a1aa;
                }

                .discount-row strong {
                    color: #b91c1c;
                }

                .summary-total {
                    display: flex;
                    align-items: center;
                    justify-content: space-between;
                    padding: 12px;
                    background: #18181b;
                    color: #fff;
                }

                .summary-total span {
                    display: block;
                    font-size: 10px;
                    font-weight: 800;
                }

                .summary-total small {
                    display: block;
                    font-size: 7px;
                    color: #a1a1aa;
                }

                .summary-total strong {
                    font-size: 15px;
                    font-weight: 900;
                }

                .summary-total strong small {
                    display: inline;
                    color: #a1a1aa;
                }

                .page-separator {
                    display: flex;
                    align-items: center;
                    gap: 10px;
                    margin: 22px 0;
                    color: #a1a1aa;
                }

                .page-separator span {
                    height: 1px;
                    background: #e4e4e7;
                    flex: 1;
                }

                .page-separator i {
                    font-size: 12px;
                }

                .labels-grid {
                    display: grid;
                    grid-template-columns: repeat(2, 1fr);
                    gap: 12px;
                }

                .label-card {
                    border: 1px solid #18181b;
                    border-radius: 12px;
                    overflow: hidden;
                    page-break-inside: avoid;
                    break-inside: avoid;
                }

                .label-top {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    padding: 12px;
                    background: #18181b;
                    color: white;
                }

                .label-brand {
                    display: flex;
                    align-items: center;
                    gap: 7px;
                }

                .mini-logo {
                    width: 28px;
                    height: 28px;
                    border: 1px solid rgba(255,255,255,.35);
                    border-radius: 7px;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                }

                .label-brand strong {
                    display: block;
                    font-size: 11px;
                }

                .label-brand span {
                    display: block;
                    font-size: 6px;
                    letter-spacing: 1px;
                    color: #a1a1aa;
                    direction: ltr;
                }

                .label-number {
                    text-align: left;
                }

                .label-number small {
                    display: block;
                    font-size: 6px;
                    color: #a1a1aa;
                    direction: ltr;
                }

                .label-number strong {
                    font-size: 14px;
                }

                .label-route {
                    padding: 15px;
                }

                .route-title {
                    display: flex;
                    align-items: center;
                    gap: 7px;
                    color: #71717a;
                    font-size: 8px;
                    margin-bottom: 9px;
                }

                .route-title i {
                    font-size: 13px;
                    color: #18181b;
                }

                .route-line {
                    width: 22px;
                    height: 1px;
                    background: #d4d4d8;
                }

                .receiver-name {
                    font-size: 16px;
                    font-weight: 900;
                    margin-bottom: 3px;
                }

                .receiver-phone {
                    font-size: 9px;
                    color: #52525b;
                    margin-bottom: 10px;
                }

                .receiver-phone i {
                    margin-left: 4px;
                }

                .receiver-address {
                    font-size: 10px;
                    line-height: 1.8;
                    padding: 8px;
                    background: #fafafa;
                    border-radius: 7px;
                }

                .receiver-location {
                    display: flex;
                    justify-content: space-between;
                    gap: 10px;
                    margin-top: 8px;
                    font-size: 8px;
                    color: #52525b;
                }

                .receiver-location strong {
                    font-weight: 800;
                    color: #18181b;
                }

                .label-bottom {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    padding: 10px 14px;
                    border-top: 1px dashed #d4d4d8;
                    background: #fafafa;
                }

                .label-bottom small {
                    display: block;
                    color: #a1a1aa;
                    font-size: 7px;
                }

                .label-bottom strong {
                    display: block;
                    font-size: 9px;
                }

                .label-price {
                    text-align: left;
                }

                .label-price strong {
                    font-size: 11px;
                }

                .label-price em {
                    font-size: 7px;
                    font-style: normal;
                    color: #71717a;
                }

                .print-footer {
                    display: block !important;
                    text-align: center;
                    margin-top: 25px;
                    padding-top: 15px;
                    color: #a1a1aa;
                }

                .footer-line {
                    height: 1px;
                    background: #e4e4e7;
                    margin-bottom: 10px;
                }

                .print-footer strong {
                    display: block;
                    font-size: 10px;
                    color: #52525b;
                }

                .print-footer span,
                .print-footer small {
                    display: block;
                    font-size: 7px;
                    margin-top: 2px;
                }

                @media print {
                    .order-card {
                        box-shadow: none;
                    }
                }

            </style>
        </head>

        <body>
            ${printContents}
        </body>
        </html>
    `);

    doc.close();

    setTimeout(() => {
        iframe.contentWindow.print();

        setTimeout(() => {
            document.body.removeChild(iframe);
        }, 1500);

    }, 500);
};


// ================= HELPERS =================

const getCurrentDate = () => {
    return new Date().toLocaleDateString('fa-IR', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
    });
};


const fetchOrders = async () => {

    const ids = route.query.ids;
    const type = route.query.type || 'full';

    printType.value = type;

    if (!ids) {
        router.push('/orders');
        return;
    }

    orderIds.value = ids
        .split(',')
        .map(id => parseInt(id));

    loading.value = true;

    try {

        const response = await axios.post('/orders/print-data', {
            ids: orderIds.value
        });

        if (response.data.success) {

            orders.value = response.data.data;

        } else {

            alert('خطا در دریافت اطلاعات سفارش‌ها');
            router.push('/orders');

        }

    } catch (error) {

        console.error('Error:', error);

        alert('خطا در ارتباط با سرور');
        router.push('/orders');

    } finally {

        loading.value = false;

    }
};


const formatDate = (date) => {

    if (!date) return '-';

    try {

        const d = new Date(date);

        return d.toLocaleDateString('fa-IR') +
            ' - ' +
            d.toLocaleTimeString('fa-IR', {
                hour: '2-digit',
                minute: '2-digit'
            });

    } catch {

        return date;

    }
};


const statusText = (status) => {

    const map = {
        pending: "در انتظار",
        reserved: "رزرو شده",
        processing: "در حال پردازش",
        paid: "پرداخت شده",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        returned: "مرجوعی",
    };

    return map[status] ?? status;
};


const statusClass = (status) => {

    const map = {
        pending: "status-pending",
        reserved: "status-reserved",
        processing: "status-processing",
        paid: "status-completed",
        shipped: "status-shipped",
        completed: "status-completed",
        canceled: "status-canceled",
        returned: "status-returned",
    };

    return map[status] ?? "status-pending";
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


const paymentClass = (status) => {

    const map = {
        pending: "payment-pending",
        paid: "payment-paid",
        failed: "payment-failed",
        refunded: "payment-refunded",
    };

    return map[status] ?? "payment-pending";
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
    fetchOrders();
});
</script>


<style scoped>
/* =========================================================
   BASE
========================================================= */

.print-wrapper {
    min-height: 100vh;
    background:
        radial-gradient(circle at top right, rgba(24, 24, 27, .035), transparent 30%),
        #f6f6f5;
    color: #18181b;
    direction: rtl;
}


/* =========================================================
   TOOLBAR
========================================================= */

.print-toolbar {
    background: rgba(255, 255, 255, .92);
    backdrop-filter: blur(18px);
    border-bottom: 1px solid #e4e4e7;
}

.toolbar-content {
    max-width: 1400px;
    min-height: 76px;
    margin: auto;
    padding: 12px 28px;

    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 25px;
}

.toolbar-brand {
    display: flex;
    align-items: center;
    gap: 11px;
}

.brand-mark {
    width: 42px;
    height: 42px;

    display: flex;
    align-items: center;
    justify-content: center;

    background: #18181b;
    color: #fff;
    border-radius: 12px;
    font-size: 18px;
}

.toolbar-brand strong {
    display: block;
    font-size: 14px;
    font-weight: 900;
}

.toolbar-brand span {
    display: block;
    color: #a1a1aa;
    font-size: 10px;
    margin-top: 2px;
}

.toolbar-meta {
    display: flex;
    align-items: center;
    gap: 8px;
}

.toolbar-chip {
    display: flex;
    align-items: center;
    gap: 7px;

    padding: 9px 13px;

    background: #fafafa;
    border: 1px solid #e4e4e7;
    border-radius: 10px;

    color: #52525b;
    font-size: 11px;
    font-weight: 700;
}

.toolbar-chip i {
    color: #18181b;
}

.toolbar-actions {
    display: flex;
    align-items: center;
    gap: 8px;
}

.btn-print,
.btn-back {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 8px;

    height: 42px;
    padding: 0 18px;

    border-radius: 10px;

    font-size: 12px;
    font-weight: 800;

    cursor: pointer;
    text-decoration: none;

    transition: .2s ease;
}

.btn-print {
    border: 0;
    background: #18181b;
    color: #fff;
}

.btn-print:hover {
    background: #27272a;
    transform: translateY(-1px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, .12);
}

.btn-back {
    background: #fff;
    color: #52525b;
    border: 1px solid #e4e4e7;
}

.btn-back:hover {
    background: #f4f4f5;
    color: #18181b;
}


/* =========================================================
   PRINT SECTION
========================================================= */

#printSection {
    max-width: 1300px;
    margin: 0 auto;
    padding: 40px 30px 70px;
}

.print-header,
.print-footer {
    display: none;
}


/* =========================================================
   LOADING
========================================================= */

.loading-state {
    min-height: 65vh;

    display: flex;
    align-items: center;
    justify-content: center;
}

.loading-box {
    width: 260px;
    padding: 30px;

    display: flex;
    flex-direction: column;
    align-items: center;

    background: #fff;
    border: 1px solid #e4e4e7;
    border-radius: 18px;

    box-shadow: 0 15px 45px rgba(0, 0, 0, .05);
}

.loading-spinner {
    width: 42px;
    height: 42px;

    border: 3px solid #e4e4e7;
    border-top-color: #18181b;

    border-radius: 50%;

    animation: spin .8s linear infinite;
    margin-bottom: 18px;
}

.loading-box strong {
    font-size: 13px;
}

.loading-box span {
    color: #a1a1aa;
    font-size: 10px;
    margin-top: 5px;
}

@keyframes spin {
    to {
        transform: rotate(360deg);
    }
}


/* =========================================================
   EMPTY
========================================================= */

.empty-state {
    min-height: 65vh;

    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.empty-icon {
    width: 75px;
    height: 75px;

    display: flex;
    align-items: center;
    justify-content: center;

    border-radius: 22px;
    background: #fff;
    border: 1px solid #e4e4e7;

    color: #a1a1aa;
    font-size: 30px;

    margin-bottom: 20px;
}

.empty-state h3 {
    margin: 0;
    font-size: 17px;
    font-weight: 900;
}

.empty-state p {
    margin: 7px 0 20px;
    color: #a1a1aa;
    font-size: 12px;
}

.empty-button {
    display: inline-flex;
    align-items: center;
    gap: 7px;

    padding: 10px 16px;

    border-radius: 9px;

    background: #18181b;
    color: #fff;

    font-size: 11px;
    font-weight: 800;
    text-decoration: none;
}


/* =========================================================
   ORDER CARD
========================================================= */

.order-card {
    background: #fff;
    border: 1px solid #e4e4e7;
    border-radius: 18px;

    overflow: hidden;

    box-shadow: 0 12px 40px rgba(0, 0, 0, .035);

    margin-bottom: 30px;

    transition: .25s ease;
}

.order-card:hover {
    box-shadow: 0 18px 50px rgba(0, 0, 0, .06);
}

.order-header {
    padding: 18px 22px;

    display: flex;
    align-items: center;
    justify-content: space-between;

    background: #fff;

    border-bottom: 1px solid #e4e4e7;
}

.order-main {
    display: flex;
    align-items: center;
    gap: 18px;
}

.order-id-box small {
    display: block;
    color: #a1a1aa;
    font-size: 9px;
    margin-bottom: 2px;
}

.order-id-box strong {
    font-size: 18px;
    font-weight: 900;
}

.order-statuses {
    display: flex;
    gap: 6px;
}

.status-badge,
.payment-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;

    padding: 5px 10px;

    border-radius: 999px;

    font-size: 9px;
    font-weight: 800;
}

.status-badge i {
    font-size: 5px;
}

.status-completed,
.payment-paid {
    background: #ecfdf5;
    color: #047857;
}

.status-pending,
.payment-pending {
    background: #fffbeb;
    color: #b45309;
}

.status-processing {
    background: #eff6ff;
    color: #1d4ed8;
}

.status-shipped {
    background: #f5f3ff;
    color: #6d28d9;
}

.status-canceled,
.payment-failed {
    background: #fef2f2;
    color: #b91c1c;
}

.status-returned,
.payment-refunded {
    background: #f4f4f5;
    color: #52525b;
}

.order-date {
    text-align: left;
}

.order-date span {
    display: block;
    color: #a1a1aa;
    font-size: 9px;
}

.order-date strong {
    display: block;
    font-size: 10px;
    margin-top: 2px;
}


/* =========================================================
   ORDER BODY
========================================================= */

.order-body {
    padding: 22px;
}

.info-grid {
    display: grid;
    grid-template-columns: 1fr 1.4fr 1fr;
    gap: 12px;

    margin-bottom: 28px;
}

.info-card {
    padding: 15px;

    min-height: 125px;

    background: #fafafa;
    border: 1px solid #eeeeef;
    border-radius: 13px;
}

.info-card-head {
    display: flex;
    align-items: center;
    gap: 8px;

    padding-bottom: 10px;
    margin-bottom: 10px;

    border-bottom: 1px solid #e7e7e8;

    color: #71717a;
    font-size: 10px;
    font-weight: 800;
}

.info-icon {
    width: 30px;
    height: 30px;

    display: flex;
    align-items: center;
    justify-content: center;

    background: #18181b;
    color: #fff;

    border-radius: 9px;

    font-size: 12px;
}

.info-card-content>strong {
    display: block;

    color: #18181b;
    font-size: 12px;
    font-weight: 900;

    margin-bottom: 7px;
}

.info-line {
    display: flex;
    align-items: flex-start;
    gap: 7px;

    color: #52525b;
    font-size: 10px;

    margin-top: 5px;
}

.info-line i {
    color: #a1a1aa;
    margin-top: 3px;
}

.address-text {
    line-height: 1.8;
}


/* =========================================================
   SECTION TITLE
========================================================= */

.section-title {
    display: flex;
    align-items: center;
    gap: 10px;

    margin-bottom: 10px;
}

.section-title::before {
    content: "";

    width: 4px;
    height: 30px;

    background: #18181b;
    border-radius: 4px;
}

.section-title span {
    display: block;
    font-size: 12px;
    font-weight: 900;
}

.section-title small {
    display: block;
    color: #a1a1aa;
    font-size: 9px;
    margin-top: 2px;
}


/* =========================================================
   TABLE
========================================================= */

.products-table {
    overflow: hidden;

    border: 1px solid #e4e4e7;
    border-radius: 11px;
}

.products-table table {
    width: 100%;
    border-collapse: collapse;
}

.products-table thead {
    background: #18181b;
    color: #fff;
}

.products-table th {
    padding: 11px 9px;

    font-size: 9px;
    font-weight: 800;
}

.products-table td {
    padding: 11px 9px;

    border-bottom: 1px solid #f0f0f0;

    text-align: center;

    font-size: 10px;
}

.products-table tbody tr:last-child td {
    border-bottom: none;
}

.product-name {
    text-align: right !important;
    font-weight: 800;
}

.index-cell {
    color: #a1a1aa;
    font-weight: 800;
}

.variant-tag {
    display: inline-block;

    padding: 3px 7px;
    margin: 1px;

    border-radius: 6px;

    background: #f4f4f5;
    border: 1px solid #e4e4e7;

    font-size: 8px;
}

.muted {
    color: #a1a1aa;
}

.quantity span {
    display: inline-flex;

    min-width: 26px;
    height: 26px;

    align-items: center;
    justify-content: center;

    border-radius: 7px;

    background: #f4f4f5;

    font-weight: 900;
}

.products-table td small {
    color: #a1a1aa;
    font-size: 8px;
}

.total-price {
    font-weight: 900;
}


/* =========================================================
   SUMMARY
========================================================= */

.bottom-section {
    display: flex;
    align-items: flex-end;
    justify-content: space-between;

    gap: 30px;

    margin-top: 18px;
}

.thank-message {
    display: flex;
    align-items: center;
    gap: 10px;
}

.thank-icon {
    width: 38px;
    height: 38px;

    display: flex;
    align-items: center;
    justify-content: center;

    background: #18181b;
    color: #fff;

    border-radius: 11px;
}

.thank-message strong {
    display: block;
    font-size: 11px;
    font-weight: 900;
}

.thank-message span {
    display: block;
    color: #a1a1aa;
    font-size: 9px;
    margin-top: 2px;
}

.summary {
    width: 330px;

    overflow: hidden;

    border: 1px solid #e4e4e7;
    border-radius: 12px;
}

.summary-row {
    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 9px 12px;

    border-bottom: 1px solid #f0f0f0;

    color: #52525b;
    font-size: 10px;
}

.summary-row strong {
    color: #18181b;
    font-weight: 900;
}

.summary-row small {
    color: #a1a1aa;
    font-size: 8px;
}

.discount-row strong {
    color: #b91c1c;
}

.summary-total {
    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 14px;

    background: #18181b;
    color: #fff;
}

.summary-total span {
    display: block;
    font-size: 11px;
    font-weight: 900;
}

.summary-total small {
    display: block;
    color: #a1a1aa;
    font-size: 8px;
}

.summary-total strong {
    font-size: 17px;
    font-weight: 900;
}

.summary-total strong small {
    display: inline;
}


/* =========================================================
   SEPARATOR
========================================================= */

.page-separator {
    display: flex;
    align-items: center;
    gap: 12px;

    margin: 25px 0;

    color: #a1a1aa;
}

.page-separator span {
    flex: 1;
    height: 1px;
    background: #e4e4e7;
}

.page-separator i {
    font-size: 12px;
}


/* =========================================================
   LABELS
========================================================= */

.labels-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 18px;
}

.label-card {
    overflow: hidden;

    background: #fff;

    border: 1px solid #18181b;
    border-radius: 15px;

    box-shadow: 0 10px 35px rgba(0, 0, 0, .04);
}

.label-top {
    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 15px 17px;

    background: #18181b;
    color: #fff;
}

.label-brand {
    display: flex;
    align-items: center;
    gap: 9px;
}

.mini-logo {
    width: 34px;
    height: 34px;

    display: flex;
    align-items: center;
    justify-content: center;

    border: 1px solid rgba(255, 255, 255, .25);
    border-radius: 9px;

    font-size: 14px;
}

.label-brand strong {
    display: block;
    font-size: 12px;
    font-weight: 900;
}

.label-brand span {
    display: block;
    color: #a1a1aa;
    font-size: 7px;
    letter-spacing: 1px;
    direction: ltr;
}

.label-number {
    text-align: left;
}

.label-number small {
    display: block;
    color: #a1a1aa;
    font-size: 7px;
    direction: ltr;
}

.label-number strong {
    font-size: 15px;
}

.label-route {
    padding: 20px;
}

.route-title {
    display: flex;
    align-items: center;
    gap: 8px;

    color: #71717a;

    font-size: 9px;
    font-weight: 700;

    margin-bottom: 10px;
}

.route-title i {
    color: #18181b;
}

.route-line {
    width: 30px;
    height: 1px;
    background: #d4d4d8;
}

.receiver-name {
    font-size: 18px;
    font-weight: 900;

    margin-bottom: 4px;
}

.receiver-phone {
    color: #52525b;
    font-size: 10px;

    margin-bottom: 13px;
}

.receiver-phone i {
    margin-left: 5px;
}

.receiver-address {
    padding: 10px;

    background: #fafafa;
    border: 1px solid #f0f0f0;
    border-radius: 8px;

    font-size: 10px;
    line-height: 1.9;
}

.receiver-location {
    display: flex;
    justify-content: space-between;
    gap: 12px;

    margin-top: 9px;

    color: #52525b;
    font-size: 9px;
}

.receiver-location strong {
    color: #18181b;
    font-weight: 900;
}

.label-bottom {
    display: flex;
    align-items: center;
    justify-content: space-between;

    padding: 12px 17px;

    background: #fafafa;
    border-top: 1px dashed #d4d4d8;
}

.label-bottom small {
    display: block;
    color: #a1a1aa;
    font-size: 8px;
}

.label-bottom strong {
    display: block;
    font-size: 10px;
    font-weight: 900;
}

.label-price {
    text-align: left;
}

.label-price strong {
    font-size: 13px;
}

.label-price em {
    color: #71717a;
    font-size: 8px;
    font-style: normal;
}


/* =========================================================
   RESPONSIVE
========================================================= */

@media (max-width: 1000px) {

    .toolbar-content {
        padding: 12px 18px;
    }

    .toolbar-meta {
        display: none;
    }

    #printSection {
        padding: 25px 18px 50px;
    }

    .info-grid {
        grid-template-columns: 1fr;
    }

    .labels-grid {
        grid-template-columns: 1fr;
    }

}

@media (max-width: 650px) {

    .toolbar-brand {
        display: none;
    }

    .toolbar-content {
        justify-content: space-between;
    }

    .order-header {
        align-items: flex-start;
        gap: 12px;
        flex-direction: column;
    }

    .order-main {
        width: 100%;
        justify-content: space-between;
    }

    .order-date {
        text-align: right;
    }

    .bottom-section {
        flex-direction: column;
        align-items: stretch;
    }

    .summary {
        width: 100%;
    }

    .thank-message {
        padding: 5px 0;
    }

    .products-table {
        overflow-x: auto;
    }

    .products-table table {
        min-width: 700px;
    }

    .label-bottom,
    .receiver-location {
        flex-direction: column;
        align-items: flex-start;
    }

    .label-price {
        text-align: right;
    }

}


/* =========================================================
   PRINT SCREEN
========================================================= */

@media print {

    .print-wrapper {
        background: #fff;
    }

    .print-toolbar {
        display: none !important;
    }

    #printSection {
        max-width: 100%;
        padding: 0;
    }

    .print-header,
    .print-footer {
        display: block !important;
    }

    .order-card,
    .label-card {
        box-shadow: none !important;
    }

}
</style>
```
