<template>
  <div class="container mt-4 sliders">

    <!-- فیلتر -->
    <div class="card mb-3">
      <div class="card-body row g-2 align-items-center">
        <div class="col-md-4">
          <input v-model="filters.search" @keyup.enter="fetchData" type="text" class="form-control"
            placeholder="جستجو بر اساس عنوان" />
        </div>
        <div class="col-md-2">
          <button class="btn btn-primary w-100" @click="fetchData">جستجو</button>
        </div>
      </div>
    </div>

    <!-- افزودن -->
    <div class="d-flex justify-content-end mb-3">
      <router-link to="/content/sliders/create" class="btn btn-success">
        افزودن اسلایدر
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
              <th>عنوان</th>
              <th>نوع</th>
              <th>تصویر</th>
              <th>عملیات</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="slider in sliders" :key="slider.id">
              <td>{{ slider.title }}</td>
              <td>{{ slider.type == "desktop" ? "دسکتاپ" : "موبایل" }}</td>
              <td>
                <img v-if="slider.image" :src="slider.image" alt="" class="img-thumbnail" style="max-width:100px;">
              </td>
              <td>
                <router-link :to="`/content/sliders/${slider.id}/edit`"
                  class="btn btn-sm btn-primary me-2">ویرایش</router-link>
                <button @click="deleteSlider(slider.id)" class="btn btn-sm btn-danger">حذف</button>
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

const sliders = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
let currentUrl = "/sliders";

const fetchData = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    sliders.value = data.data.map(slide => ({ id: slide.id, title: slide.title, type: slide.type, image: baseImageAddress + slide.image, }));

  } finally {
    loading.value = false;
  }
};


const deleteSlider = async (id) => {
  const result = await Swal.fire({
    title: "آیا مطمئن هستید؟",
    text: "این عملیات قابل بازگشت نیست!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });

  if (result.isConfirmed) {
    await axios.delete(`/sliders/${id}`);
    Swal.fire("حذف شد!", "اسلایدر با موفقیت حذف شد.", "success");
    fetchData();
  }
};

onMounted(() => {
  fetchData();
});
</script>