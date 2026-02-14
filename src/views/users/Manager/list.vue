<template>
    <div class="users-page container mt-4" v-if="checkPermission(['manager_view'])">
        <!-- Header -->
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h4>مدیریت مدیران</h4>
            <router-link to="/users/managers/create" class="btn btn-primary">
                افزودن مدیر
            </router-link>
        </div>

        <!-- Filter -->
        <div class="card mb-3">
            <div class="card-body">
                <input v-model="filters.search" @input="getManagers" type="text" class="form-control"
                    placeholder="جستجو بر اساس نام یا موبایل" />
            </div>
        </div>

        <!-- Table -->
        <div class="card">
            <div class="card-body">
                <div v-if="loading" class="text-center my-5">
                    <div class="spinner-border" role="status"></div>
                    <p class="mt-2">در حال بارگذاری...</p>
                </div>

                <div v-else>
                    <table class="table table-bordered table-striped">
                        <thead>
                            <tr>
                                <th>شناسه</th>
                                <th>نام کامل</th>
                                <th>موبایل</th>
                                <th>عملیات</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{ user.id }}</td>
                                <td>{{ user.full_name }}</td>
                                <td>{{ user.mobile }}</td>
                                <td>
                                    <router-link :to="`/users/managers/${user.id}/edit`" class="btn btn-sm btn-info">
                                        ویرایش
                                    </router-link>
                                    <button class="btn btn-sm btn-danger ms-2" @click="confirmDelete(user.id)">
                                        حذف
                                    </button>
                                </td>
                            </tr>
                        </tbody>
                    </table>


                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const route = useRoute();
const users = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });

const getManagers = async () => {
    loading.value = true;
    try {
        const response = await axios.get("/user-managers");
        users.value = response.data;
    } finally {
        loading.value = false;
    }
};
onMounted(() => {
    getManagers();
});
</script>