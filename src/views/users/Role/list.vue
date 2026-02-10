<template>
  <div class="container mt-4">

    <!-- دکمه افزودن نقش -->
    <div class="mb-3 text-end">
      <router-link to="/users/roles/create" class="btn btn-success">
        افزودن نقش
      </router-link>
    </div>

    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>نام نقش</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="role in roles" :key="role.id">
                <td>{{ role.id }}</td>
                <td>{{ role.name }}</td>
                <td v-if="!role.is_system">
                  <router-link :to="`/users/roles/${role.id}/edit`" class="btn btn-sm btn-warning me-2">
                    ویرایش
                  </router-link>

                  <button class="btn btn-sm btn-danger" @click="deleteRole(role.id)">
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

const roles = ref([]);
const loading = ref(false);
let currentUrl = "/roles";

const getroles = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    roles.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};



const deleteRole = (id) => {
  Swal.fire({
    title: "حذف نقش",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/roles/${id}`);
        Swal.fire("موفق", "نقش حذف شد", "success");
        getroles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getroles();
});
</script>