<template>
    <div class="container mt-4" v-if="checkPermission(['campaign_view'])">
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h3><i class="bi bi-megaphone"></i> مدیریت کمپین‌ها</h3>
            <router-link v-if="checkPermission(['campaign_store'])" to="/campaigns/create" class="btn btn-primary">
                <i class="bi bi-plus"></i> افزودن
            </router-link>
        </div>

        <div class="card">
            <div class="card-body table-responsive">
                <div v-if="loading" class="text-center my-4">
                    <div class="spinner-border"></div>
                </div>
                <table v-else class="table table-bordered table-striped mb-0">
                    <thead>
                        <tr>
                            <th>نام</th>
                            <th>شروع</th>
                            <th>پایان</th>
                            <th>وضعیت</th>
                            <th>تعداد سفارش</th>
                            <th>جمع فروش</th>
                            <th>عملیات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="c in campaigns.data" :key="c.id">
                            <td>{{ c.name }}</td>
                            <td>{{ fa(c.starts_at) }}</td>
                            <td>{{ fa(c.ends_at) }}</td>
                            <td><span class="badge" :class="phase[c.phase].cls">{{ phase[c.phase].label }}</span></td>
                            <td>{{ c.orders_count }}</td>
                            <td>{{ Number(c.sales_sum || 0).toLocaleString('fa-IR') }}</td>
                            <td>
                                <div class="d-flex flex-wrap gap-1">
                                    <router-link :to="`/campaigns/${c.id}/report`" class="btn btn-sm btn-success">
                                        <i class="bi bi-bar-chart"></i> گزارش
                                    </router-link>
                                    <router-link v-if="checkPermission(['campaign_update'])"
                                        :to="`/campaigns/${c.id}/edit`" class="btn btn-sm btn-info">
                                        <i class="bi bi-pen"></i> ویرایش
                                    </router-link>
                                    <button v-if="checkPermission(['campaign_update'])" class="btn btn-sm btn-warning"
                                        @click="toggle(c)">
                                        {{ c.is_active ? 'غیرفعال' : 'فعال' }}
                                    </button>
                                    <button v-if="checkPermission(['campaign_delete'])" class="btn btn-sm btn-danger"
                                        @click="remove(c)">
                                        <i class="bi bi-trash3-fill"></i> حذف
                                    </button>
                                </div>
                            </td>
                        </tr>
                        <tr v-if="!campaigns.data?.length">
                            <td colspan="7" class="text-center text-muted">کمپینی یافت نشد</td>
                        </tr>
                    </tbody>
                </table>

                <b-pagination v-if="campaigns.last_page > 1" v-model="page" :total-rows="campaigns.total"
                    :per-page="campaigns.per_page" align="center" class="mt-3" @update:modelValue="load" />
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import Swal from 'sweetalert2'
import { toast } from 'vue3-toastify'
import moment from 'moment-jalaali'
import { useAdmin } from '@/stores/modules/admin'

const checkPermission = useAdmin().checkPermission
const campaigns = ref({ data: [] })
const loading = ref(false)
const page = ref(1)

const phase = {
    running: { label: 'در حال اجرا', cls: 'bg-success' },
    upcoming: { label: 'شروع نشده', cls: 'bg-info' },
    ended: { label: 'پایان یافته', cls: 'bg-secondary' },
    inactive: { label: 'غیرفعال', cls: 'bg-warning text-dark' },
}
const fa = (d) => moment(d).format('jYYYY/jMM/jDD HH:mm')

const load = async (p = 1) => {
    loading.value = true
    try {
        const { data } = await axios.get('/campaigns', { params: { page: p } })
        campaigns.value = data.data
    } finally { loading.value = false }
}

const toggle = async (c) => {
    try {
        const { data } = await axios.patch(`/campaigns/${c.id}/toggle`)
        toast.success(data.message)
        load(page.value)
    } catch (e) { toast.error(e.response?.data?.message || 'خطا') }
}

const remove = (c) => {
    Swal.fire({
        title: 'آیا مطمئن هستید؟', text: 'این عملیات قابل بازگشت نیست!', icon: 'warning',
        showCancelButton: true, confirmButtonText: 'بله، حذف شود', cancelButtonText: 'انصراف',
    }).then(async (r) => {
        if (!r.isConfirmed) return
        try {
            await axios.delete(`/campaigns/${c.id}`)
            toast.success('کمپین حذف شد')
            load(page.value)
        } catch (e) { toast.error(e.response?.data?.message || 'خطا در حذف') }
    })
}

onMounted(() => load())
</script>