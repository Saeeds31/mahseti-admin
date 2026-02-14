<template>
  <div class="container shipping-rates" v-if="checkPermission(['shipping_list'])">

    <!-- فیلتر -->
    <div class="card mb-3">
      <div class="card-body row g-2 align-items-center">

        <div class="col-md-4">
          <select v-model="filters.province_id" class="form-select">
            <option value="">همه استان‌ها</option>
            <option v-for="province in provinces" :key="province.id" :value="province.id">
              {{ province.name }}
            </option>
          </select>
        </div>
        <div class="col-md-4">
          <select v-model="filters.city_id" class="form-select">
            <option value="">همه شهرها</option>
            <option v-for="city in cities" :key="city.id" :value="city.id">
              {{ city.name }}
            </option>
          </select>
        </div>
        <div class="col-md-4">
          <button class="btn btn-primary w-100" @click="fetchData">جستجو</button>
        </div>
      </div>
    </div>

    <!-- افزودن -->
    <div class="d-flex justify-content-end mb-3">
      <router-link :to="`/shop/shipping/${route.params.method}/range/create`" class="btn btn-success">
        افزودن بازه قیمت
      </router-link>
    </div>

    <!-- جدول -->
    <div class="card">
      <div class="card-body table-responsive">
        <div v-if="loading" class="text-center p-4">
          <div class="spinner-border text-primary"></div>
        </div>

        <table v-else class="table table-bordered table-hover">
          <thead>
            <tr>
              <th>روش ارسال</th>
              <th>استان</th>
              <th>شهر</th>
              <th>هزینه</th>
              <th>حداقل سفارش</th>
              <th>حداکثر سفارش</th>
              <th>عملیات</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="rate in shippingRates" :key="rate.id">
              <td>{{ rate.method.name }}</td>
              <td>{{ rate.province?.name }}</td>
              <td>{{ rate.city?.name }}</td>
              <td>{{ rate.cost }} تومان</td>
              <td>{{ rate.min_order_amount }} تومان</td>
              <td>{{ rate.max_order_amount }} تومان</td>
              <td>
                <router-link :to="`/shop/shipping/${route.params.method}/range/${rate.id}/edit`" class="btn btn-sm btn-primary me-2">
                  ویرایش
                </router-link>
                <button class="btn btn-sm btn-danger" @click="deleteRate(rate.id)">
                  حذف
                </button>
              </td>
            </tr>
          </tbody>
        </table>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute } from "vue-router";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const shippingRates = ref([]);
const shippingMethods = ref([]);
const provinces = ref([]);
const cities = ref([]);
const filters = ref({ province_id: "", city_id: "" });
const loading = ref(false);
const route = useRoute();
let currentUrl = `/shipping-ranges/${route.params.method}`;

const fetchData = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    shippingRates.value = data.data;
    console.log(data);
    
  } finally {
    loading.value = false;
  }
};

const fetchRelations = async () => {
  try {
    const [methodsRes, provincesRes, citiesRes] = await Promise.all([
      axios.get("/provinces"),
      axios.get("/cities/list"),
    ]);
    shippingMethods.value = methodsRes.data;
    provinces.value = provincesRes.data;
    cities.value = citiesRes.data;
  } catch (err) {
    console.error(err);
  }
};


const changePage = (page) => {
  if (page) fetchData(`${currentUrl}?page=${page}`);
  else currentUrl = "/shipping-rates"
};
const deleteRate = async (id) => {
  const result = await Swal.fire({
    title: "آیا مطمئن هستید؟",
    text: "این عملیات قابل بازگشت نیست!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });

  if (result.isConfirmed) {
    await axios.delete(`/shipping-ranges/${route.params.method}/${id}`);
    Swal.fire("حذف شد!", "نرخ ارسال با موفقیت حذف شد.", "success");
    fetchData();
  }
};

onMounted(() => {
  // fetchRelations();
  fetchData();
});
</script>