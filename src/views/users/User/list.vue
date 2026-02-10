<template>
    <div class="users-page container mt-4">
        <!-- Header -->
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h4>مدیریت کاربران</h4>
            <router-link to="/users/create" class="btn btn-primary">
                افزودن کاربر
            </router-link>
        </div>

        <!-- Filter -->
        <div class="card mb-3">
            <div class="card-body">
                <input v-model="filters.search" @input="getUsers" type="text" class="form-control"
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
                                <th>کدملی</th>
                                <th>تاریخ تولد</th>
                                <th>عملیات</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="user in users.data" :key="user.id">
                                <td>{{ user.id }}</td>
                                <td>{{ user.full_name }}</td>
                                <td>{{ user.mobile }}</td>
                                <td>{{ user.national_code ?? '-' }}</td>
                                <td>{{ user.birth_date ?? '-' }}</td>
                                <td>
                                    <router-link :to="`/users/${user.id}/addresses`" class="btn btn-sm btn-success">
                                        آدرس ها
                                    </router-link>
                                    <router-link :to="`/users/${user.id}/edit`" class="btn btn-sm btn-info ms-2">
                                        ویرایش
                                    </router-link>
                                    <button class="btn btn-sm btn-danger ms-2" @click="confirmDelete(user.id)">
                                        حذف
                                    </button>
                                </td>
                            </tr>
                        </tbody>
                    </table>

                    <b-pagination v-model="currentPage" :total-rows="users.total" v-if="users.last_page != 1"
                        :per-page="users.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3"></b-pagination>

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
const router = useRouter();
const route = useRoute();
const users = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
const currentPage = ref(1);

const getUsers = async (page = 1) => {
    loading.value = true;
    try {
        const response = await axios.get("/users", {
            params: {
                page,
                search: filters.value.search,
            },
        });
        users.value = response.data;
        currentPage.value = response.data.current_page;
    } finally {
        loading.value = false;
    }
};

const changePage = (page) => {
    if (page) {
        router.replace({ name: route.name, query: { page: page } })
        getUsers(page)
    }
    else currentUrl = "/products"
};

const confirmDelete = (id) => {
    Swal.fire({
        title: "آیا مطمئن هستید؟",
        text: "این عملیات قابل بازگشت نیست!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "بله، حذف شود",
        cancelButtonText: "انصراف",
    }).then(async (result) => {
        if (result.isConfirmed) {
            await axios.delete(`/users/${id}`);
            getUsers(currentPage.value);
            Swal.fire("حذف شد!", "کاربر موردنظر با موفقیت حذف شد.", "success");
        }
    });
};

onMounted(() => {
    currentPage.value = route.query.page ?? 1;
    getUsers(currentPage.value);
});
</script>