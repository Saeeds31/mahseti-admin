<template>
  <div id="sidebar">
    <!-- Overlay موبایل -->
    <b-collapse v-model="mobileOpen" class="d-lg-none">
      <div class="position-fixed top-0 start-0 w-100 h-100 bg-dark bg-opacity-50" @click="mobileOpen = false"></div>
    </b-collapse>

    <!-- Sidebar -->
    <div class="sidebar d-flex flex-column bg-light" :class="{ 'sidebar-open': mobileOpen }">
      <!-- Logo -->
      <router-link to="/dashboard" class="p-3 d-flex justify-content-center border-bottom">
        <img src="../../assets/images/logo.png" alt="Logo" class="img-fluid" />
      </router-link>

      <!-- Menu Items -->
      <b-nav vertical class="flex-column mt-2">
        <div v-for="item in menuItems" :key="item.name" class="w-100">
          <!-- آیتم اصلی -->
          <b-nav-item v-if="!item.children" :to="item.link" class="d-flex align-items-center">
            <i :class="item.icon + ' me-2'"></i>
            <span v-if="mobileOpen || windowWidth < 992">{{ item.name }}</span>
          </b-nav-item>

          <!-- آیتم با زیرمنو -->
          <div v-else>
            <button class=" navItem btn w-100 text-start d-flex align-items-center" @click="toggleSubmenu(item)">
              <i :class="item.icon + ' me-2'"></i>
              <span v-if="mobileOpen || windowWidth < 992">{{ item.name }}</span>
              <i class="mr-auto bi" :class="openSubmenu === item.name ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
            </button>
            <b-collapse v-model="openSubmenuStates[item.name]" class="ps-4">
              <b-nav vertical>
                <b-nav-item v-for="child in item.children" :key="child.name" :to="child.link"
                  class="d-flex align-items-center">
                  <i :class="child.icon + ' me-2'"></i>
                  <span v-if="mobileOpen || windowWidth < 992">{{ child.name }}</span>
                </b-nav-item>
              </b-nav>
            </b-collapse>
          </div>
        </div>
      </b-nav>
    </div>

    <!-- Toggle Button موبایل -->
    <b-button variant="primary" class="d-lg-none position-fixed top-3 start-3" @click="mobileOpen = !mobileOpen">
      <i class="bi-list"></i>
    </b-button>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { BNav, BNavItem, BButton, BCollapse } from "bootstrap-vue-3";

export default {
  components: { BNav, BNavItem, BButton, BCollapse },
  setup() {
    const mobileOpen = ref(true);
    const windowWidth = ref(window.innerWidth);

    // ذخیره وضعیت باز/بسته بودن زیرمنوها
    const openSubmenuStates = ref({});
    const openSubmenu = ref(null);

    const toggleSubmenu = (item) => {
      openSubmenuStates.value[item.name] = !openSubmenuStates.value[item.name];
      openSubmenu.value = openSubmenuStates.value[item.name] ? item.name : null;
    };

    // منوها
    const menuItems = [
      { name: "داشبورد", link: "/", icon: "bi-house-fill" },

      {
        name: "کاربران",
        icon: "bi-people-fill",
        children: [
          { name: "لیست کاربران", link: "/users", icon: "bi-list-ul" },
          { name: "نقش ها", link: "/users/roles", icon: "bi-person-badge" },
          { name: "مدیران", link: "/users/managers", icon: "bi-person-check" },
        ],
      },

      {
        name: "کیف پول",
        icon: "bi-wallet2",
        children: [
          { name: "کیف پول", link: "/wallets", icon: "bi-list-ul" },
        ],
      },

      {
        name: "محصولات",
        icon: "bi-box-seam",
        children: [
          { name: "ویژگی‌ها", link: "/products/attributes", icon: "bi-sliders" },
          { name: "دسته‌بندی‌ها", link: "/products/categories", icon: "bi-tags" },
          { name: "محصولات", link: "/products", icon: "bi-bag" },
          { name: "افزودن محصول", link: "/products/create", icon: "bi-plus-square" },
          { name: "مشخصات", link: "/products/specification", icon: "bi-table" },
        ],
      },



      {
        name: "محتوا",
        icon: "bi-layout-text-window",
        children: [
          { name: "منو", link: "/content/menus", icon: "bi-list" },
          { name: "اسلایدر", link: "/content/sliders", icon: "bi-images" },
          { name: "بنر", link: "/content/banners", icon: "bi-collection" },
        ],
      },


      {
        name: "گزارشات",
        icon: "bi-bar-chart-line",
        children: [
          { name: "کاربران", link: "/reports/users", icon: "bi-people" },
          { name: "سفارشات", link: "/reports/orders", icon: "bi-basket" },
          { name: "محصولات", link: "/reports/products", icon: "bi-box2" },
        ],
      },

      {
        name: "فروشگاه",
        icon: "bi-shop",
        children: [
          { name: "تخفیفات", link: "/shop/coupons", icon: "bi-percent" },
          { name: "حمل و نقل", link: "/shop/shipping", icon: "bi-truck" },
        ],
      },
      {
        "name": "مکان‌ها",
        "icon": "bi-building",
        children: [
          { name: "استان‌ها", link: "/location/provinces/list", icon: "bi-geo-alt" },
          {
            name:
              "شهرها", link: "/location/cities/list", icon: "bi-geo"
          }
        ]
      },

      {
        name: "دیدگاه‌ها",
        icon: "bi-chat-dots",
        children: [
          { name: "محصولات", link: "/comments/products", icon: "bi-bag" },
          { name: "مقالات", link: "/comments/articles", icon: "bi-journal" },
        ],
      },
      {
        name: "سفارشات",
        icon: "bi-basket3",
        children: [
          { name: "سفارشات روز", link: "/orders/today", icon: "bi-calendar-day" },
          { name: "لیست", link: "/orders", icon: "bi-list-check" },
          { name: "افزودن", link: "/orders/create", icon: "bi-plus-square" },
        ],
      },

      {
        name: "مقالات",
        icon: "bi-journal",
        children: [
          { name: "دسته‌بندی‌ها", link: "/articles/categories", icon: "bi-tags" },
          { name: "لیست مقالات", link: "/articles/list", icon: "bi-file-text" },
        ],
      },

      {
        name: "تنظیمات",
        icon: "bi-gear",
        link: "/settings",
      },
    ];


    const updateWidth = () => (windowWidth.value = window.innerWidth);
    onMounted(() => window.addEventListener("resize", updateWidth));
    onBeforeUnmount(() => window.removeEventListener("resize", updateWidth));

    return {
      mobileOpen,
      windowWidth,
      menuItems,
      openSubmenu,
      openSubmenuStates,
      toggleSubmenu,
    };
  },
};
</script>

<style scoped>
.sidebar {
  width: 250px;
  transition: width 0.2s;
  height: 100vh;
  overflow: auto;
  position: fixed;
  right: 0;
  top: 0;
  z-index: 1000;
}

.sidebar-open {
  width: 250px;
  /* expanded */
}

.nav-link {
  padding: 0.5rem 1rem;
}

@media (max-width: 991px) {
  .sidebar {
    position: fixed;
    top: 0;
    right: 0;
    width: 250px;
    z-index: 1050;
    transform: translateX(-100%);
    transition: transform 0.3s ease;
  }

  .sidebar-open {
    transform: translateX(0);
  }
}
</style>
