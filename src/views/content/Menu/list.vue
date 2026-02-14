<template>
  <div class="container mt-4" v-if="checkPermission(['menu_view'])">
    <!-- دکمه افزودن -->
    <div class="mb-3 text-end">
      <router-link to="/content/menus/create" class="btn btn-success">
        افزودن منو
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
                <th>عنوان</th>
                <th>منو والد</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="menu in flattenedMenus" :key="menu.id">
                <td>{{ menu.id }}</td>
                <td>
                  <span :style="{ 'padding-right': `${menu.level * 20}px` }">
                    {{ menu.title }}
                  </span>
                </td>
                <td>{{ menu.parent?.title || '---' }}</td>
                <td>
                  <router-link
                    :to="`/content/menus/${menu.id}/edit`"
                    class="btn btn-sm btn-warning me-2"
                  >
                    ویرایش
                  </router-link>
                  <button
                    class="btn btn-sm btn-danger"
                    @click="deleteMenu(menu.id)"
                  >
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
const menus = ref([]);
const flattenedMenus = ref([]);
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/menus";

// تابع برای مسطح کردن منوهای سلسله‌مراتبی
const flattenMenus = (menuItems, level = 0, parent = null) => {
  let result = [];
  menuItems.forEach((menu) => {
    // اضافه کردن منوی فعلی به لیست
    result.push({
      ...menu,
      level, // برای نمایش فاصله‌گذاری (indentation)
      parent, // برای دسترسی به والد
    });
    // اگر فرزندان وجود دارند، آن‌ها را هم به‌صورت بازگشتی اضافه کن
    if (menu.children && menu.children.length > 0) {
      result = result.concat(
        flattenMenus(menu.children, level + 1, menu)
      );
    }
  });
  return result;
};

// دریافت منوها
const getMenus = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    menus.value = data.data;
    // تبدیل داده‌های سلسله‌مراتبی به لیست مسطح
    flattenedMenus.value = flattenMenus(data.data);
  } catch (err) {
    console.error(err);
    Swal.fire("خطا", "مشکلی در دریافت منوها پیش آمد", "error");
  } finally {
    loading.value = false;
  }
};

// حذف منو
const deleteMenu = (id) => {
  Swal.fire({
    title: "حذف منو",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/menus/${id}`);
        Swal.fire("موفق", "منو حذف شد", "success");
        getMenus(); // به‌روزرسانی لیست منوها
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getMenus();
});
</script>

<style scoped>
/* برای بهبود نمایش فاصله‌گذاری در منوهای فرزند */
td span {
  display: inline-block;
}
</style>