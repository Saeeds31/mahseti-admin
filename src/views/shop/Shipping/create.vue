<template>
    <div class="container mt-4" v-if="checkPermission(['shipping_store'])">
        <h3>ایجاد روش حمل و نقل</h3>
        <form @submit.prevent="submitForm" class="row g-3">
            <!-- Name -->
            <div class="col-md-6">
                <label class="form-label">عنوان روش</label>
                <b-form-input v-model="form.name" type="text" class="form-control" required />
            </div>

            <!-- Default Cost -->
            <div class="col-md-6">
                <label class="form-label">مقدار پیش فرض (تومان)</label>
                <b-form-input v-model="form.default_cost" type="number" min="0" class="form-control" />
            </div>

            <!-- Description -->
            <div class="col-12">
                <label class="form-label">توضیحات روش</label>
                <b-form-textarea v-model="form.description"></b-form-textarea>
            </div>

            <!-- Status -->
            <div class="col-md-6">
                <div class="form-check mt-4">
                    <b-form-checkbox id="status" v-model="form.status" :true-value="1" :false-value="0">
                        فعال
                    </b-form-checkbox>
                </div>
            </div>

            <div class="col-12">
                <button type="submit" class="btn btn-primary" :disabled="loading">
                    {{ loading ? 'در حال ثبت ...' : 'ثبت' }}
                </button>
            </div>
        </form>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { BFormTextarea, BFormInput, BFormCheckbox } from "bootstrap-vue-3"
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;

const form = ref({
    name: '',
    description: '',
    default_cost: null,
    status: false,
})

const loading = ref(false)

const submitForm = async () => {
    const formData = new FormData()
    for (const key in form.value) {
        formData.append(key, key == 'status' ? Number(form.value[key]) : form.value[key])
    }
    try {
        loading.value = true
        await axios.post('/shipping-methods', formData)
        toast.success('روش حمل و نقل با موفقیت ایجاد شد')
        form.value = { name: '', description: '', default_cost: null, status: false }
    } catch (e) {
        toast.error('خطایی در ثبت داده به وجود آمده است')
    } finally {
        loading.value = false
    }
}
</script>

<style scoped>
/* می‌تونی اینجا استایل دلخواه اضافه کنی */
</style>