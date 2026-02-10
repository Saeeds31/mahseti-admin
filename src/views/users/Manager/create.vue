<template>
    <div class="container mt-4">
        <h3>تخصیص نقش به کاربر</h3>
        <form @submit.prevent="submitForm" class="row g-3">

            <!-- انتخاب کاربر با سرچ لحظه‌ای -->
            <div class="col-md-12">
                <label class="form-label">انتخاب کاربر</label>
                <Treeselect :options="options" v-model="selectedUser" :multiple="false" @search-change="loadUsers"
                    placeholder="جستجوی کاربر..." :searchable="true" />
            </div>

            <!-- انتخاب نقش‌ها -->
            <div class="col-md-12">
                <label class="form-label">نقش‌ها</label>
                <Treeselect v-model="selectedRoles" :multiple="true" :options="roles" placeholder="انتخاب نقش‌ها"
                    label="label" value="id" />
            </div>

            <div class="col-12">
                <button type="submit" class="btn btn-primary" :disabled="loading">
                    {{ loading ? 'در حال ذخیره...' : 'ذخیره نقش‌ها' }}
                </button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
const selectedUser = ref(null)
const selectedRoles = ref([])

const roles = ref([]);
const options = ref([]);
const loading = ref(false)

// بارگذاری نقش‌ها فقط یک بار
const fetchRoles = async () => {
    try {
        const { data } = await axios.get('/roles')
        roles.value = data.data.map(r => ({ id: r.id, label: r.name }))
    } catch (e) {
        console.log(e);

        toast.error('خطا در دریافت نقش‌ها')
    }
}

// سرچ کاربران به صورت async
const loadUsers = async (searchQuery) => {
    console.log(searchQuery);
    try {
        const { data } = await axios.get('/users?search=' + searchQuery ?? '')
        const ops = data.data.map(u => ({ id: u.id, label: `${u.full_name} (${u.mobile})` }))
        options.value = ops
    } catch (e) {
        console.log(e);
        toast.error('خطا در جستجوی کاربران')
    }
}

const submitForm = async () => {
    if (!selectedUser.value) {
        toast.error('لطفاً یک کاربر انتخاب کنید')
        return
    }
    try {
        loading.value = true
        const formData = new FormData()
        formData.append('user_id', selectedUser.value)
        selectedRoles.value.forEach(roleId => formData.append('roles[]', roleId))
        await axios.post('/user-managers/assign-roles', formData)
        toast.success('نقش‌ها با موفقیت تخصیص داده شد')
    } catch (e) {
        toast.error('خطا در ذخیره نقش‌ها')
    } finally {
        loading.value = false
    }
}

onMounted(fetchRoles)
</script>

<style scoped>
/* استایل دلخواه */
</style>