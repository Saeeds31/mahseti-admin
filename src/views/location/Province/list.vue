<template>
  <div class="container mt-4" v-if="checkPermission(['province_view'])">

    <!-- باکس فیلتر -->
    <div class="card mb-3">
      <div class="card-body">
        <form @submit.prevent="getProvinces">
          <div class="row g-2">
            <div class="col-md-4">
              <input v-model="filters.name" type="text" class="form-control" placeholder="جستجو بر اساس نام استان" />
            </div>
            <div class="col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <!-- دکمه افزودن -->
    <div class="mb-3 text-end">
      <router-link to="/location/provinces/create" class="btn btn-success">
        افزودن استان
      </router-link>
    </div>

    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>نام استان</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="province in provinces" :key="province.id">
                <td>{{ province.id }}</td>
                <td>{{ province.name }}</td>
                <td>
                  <router-link :to="`/location/provinces/${province.id}/edit`" class="btn btn-sm btn-warning me-2">
                    ویرایش
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteProvince(province.id)">
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

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const provinces = ref([]);
const loading = ref(false);
const filters = ref({ name: "" });
let currentUrl = "/provinces";

const getProvinces = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    provinces.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const deleteProvince = (id) => {
  Swal.fire({
    title: "حذف استان",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/provinces/${id}`);
        Swal.fire("موفق", "استان حذف شد", "success");
        getProvinces();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getProvinces();
});
</script>