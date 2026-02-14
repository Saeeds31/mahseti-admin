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
      <div v-for="item in menuItems" :key="item.name" class="w-100">

        <b-nav-item v-if="!item.children && checkPermission(item.permissions)" :to="item.link"
          class="d-flex align-items-center">
          <i :class="item.icon + ' me-2'"></i>
          <span v-if="mobileOpen || windowWidth < 992">{{ item.name }}</span>
        </b-nav-item>

        <!-- آیتم با زیرمنو -->
        <div v-else-if="checkPermission(item.permissions, 'or')">
          <button class=" navItem btn w-100 text-start d-flex align-items-center" @click="toggleSubmenu(item)">
            <i :class="item.icon + ' me-2'"></i>
            <span v-if="mobileOpen || windowWidth < 992">{{ item.name }}</span>
            <i class="mr-auto bi" :class="openSubmenu === item.name ? 'bi-chevron-up' : 'bi-chevron-down'"></i>
          </button>
          <b-collapse v-model="openSubmenuStates[item.name]" class="ps-4">
            <b-nav vertical>
              <template v-for="child in item.children" :key="child.name">
                <b-nav-item v-if="checkPermission(child.permissions)" :to="child.link"
                  class="d-flex align-items-center">
                  <i :class="child.icon + ' me-2'"></i>
                  <span v-if="mobileOpen || windowWidth < 992">{{ child.name }}</span>
                </b-nav-item>
              </template>
            </b-nav>
          </b-collapse>
        </div>
      </div>
    </div>

    <!-- Toggle Button موبایل -->
    <b-button variant="primary" class="d-lg-none position-fixed top-3 start-3" @click="mobileOpen = !mobileOpen">
      <i class="bi-list"></i>
    </b-button>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { BNav, BNavItem, BButton, BCollapse } from "bootstrap-vue-3";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const mobileOpen = ref(true);
const windowWidth = ref(window.innerWidth);
const openSubmenuStates = ref({});
const openSubmenu = ref(null);
const toggleSubmenu = (item) => {
  openSubmenuStates.value[item.name] = !openSubmenuStates.value[item.name];
  openSubmenu.value = openSubmenuStates.value[item.name] ? item.name : null;
};

// منوها
const menuItems = ref(
  [
    {
      name: "داشبورد",
      permissions: ['dashboard_view'],
      link: "/", icon: "bi-house-fill"
    },

    {
      name: "کاربران",
      permissions: ['user_view', 'role_view', 'manager_view'],
      icon: "bi-people-fill",
      children: [
        {
          name: "لیست کاربران",
          permissions: ['user_view'],
          link: "/users", icon: "bi-list-ul"
        },
        {
          name: "نقش ها",
          permissions: ['role_view'],
          link: "/users/roles", icon: "bi-person-badge"
        },
        {
          name: "مدیران",
          permissions: ['manager_view'],
          link: "/users/managers", icon: "bi-person-check"
        },
      ],
    },

    {
      name: "کیف پول",
      icon: "bi-wallet2",
      permissions: ['wallet_view'],

      children: [
        {
          name: "کیف پول",
          permissions: ['wallet_view'],
          link: "/wallets", icon: "bi-list-ul"
        },
      ],
    },

    {
      name: "محصولات",
      permissions: ['wallet_view', 'attributes_view', 'category_view', 'product_view', 'product_store', 'specifications_view'],
      icon: "bi-box-seam",
      children: [
        {
          name: "ویژگی‌ها",
          permissions: ['attributes_view'],
          link: "/products/attributes", icon: "bi-sliders"
        },
        {
          name: "دسته‌بندی‌ها",
          permissions: ['category_view'],
          link: "/products/categories", icon: "bi-tags"
        },
        {
          name: "محصولات",
          permissions: ['product_view'],
          link: "/products", icon: "bi-bag"
        },
        {
          name: "افزودن محصول",
          permissions: ['product_store'],
          link: "/products/create", icon: "bi-plus-square"
        },
        {
          name: "مشخصات",
          permissions: ['specifications'],
          link: "/products/specification", icon: "bi-table"
        },
      ],
    },



    {
      name: "محتوا",
      permissions: ['menu_view', 'slider_view', 'banner_view'],

      icon: "bi-layout-text-window",
      children: [
        {
          name: "منو",
          permissions: ['menu_view'],
          link: "/content/menus", icon: "bi-list"
        },
        {
          name: "اسلایدر",
          permissions: ['slider_view'],
          link: "/content/sliders", icon: "bi-images"
        },
        {
          name: "بنر",
          permissions: ['banner_view'],
          link: "/content/banners", icon: "bi-collection"
        },
      ],
    },


    {
      name: "گزارشات",
      permissions: ['report_users', 'report_orders', 'report_products'],

      icon: "bi-bar-chart-line",
      children: [
        {
          name: "کاربران",
          permissions: ['report_users'],
          link: "/reports/users", icon: "bi-people"
        },
        {
          name: "سفارشات",
          permissions: ['report_orders'],
          link: "/reports/orders", icon: "bi-basket"
        },
        {
          name: "محصولات",
          permissions: ['report_products'],
          link: "/reports/products", icon: "bi-box2"
        },
      ],
    },

    {
      name: "فروشگاه",
      icon: "bi-shop",
      permissions: ['coupon_view', 'shipping_view'],


      children: [
        {
          name: "تخفیفات",
          permissions: ['coupon_view'],
          link: "/shop/coupons", icon: "bi-percent"
        },
        {
          name: "حمل و نقل",
          permissions: ['shipping_view'],
          link: "/shop/shipping", icon: "bi-truck"
        },
      ],
    },
    {
      "name": "مکان‌ها",
      permissions: ['city_view', 'province_view'],

      "icon": "bi-building",
      children: [
        {
          name: "استان‌ها",
          permissions: ['province_view'],
          link: "/location/provinces/list", icon: "bi-geo-alt"
        },
        {
          name:
            "شهرها",
          permissions: ['city_view'],
          link: "/location/cities/list", icon: "bi-geo"
        }
      ]
    },

    {
      name: "دیدگاه‌ها",
      permissions: ['comment_products', 'comment_blogs'],
      icon: "bi-chat-dots",
      children: [
        {
          name: "محصولات",
          permissions: ['comment_product'],
          link: "/comments/products", icon: "bi-bag"
        },
        {
          name: "مقالات",
          permissions: ['comment_blogs'],
          link: "/comments/articles", icon: "bi-journal"
        },
      ],
    },
    {
      name: "سفارشات",
      permissions: ['order_view', 'order_today', 'order_store'],
      icon: "bi-basket3",
      children: [
        {
          name: "سفارشات روز",
          permissions: ['order_today'],
          link: "/orders/today", icon: "bi-calendar-day"
        },
        {
          name: "لیست",
          permissions: ['order_view'],
          link: "/orders", icon: "bi-list-check"
        },
        {
          name: "افزودن",
          permissions: ['order_store'],
          link: "/orders/create", icon: "bi-plus-square"
        },
      ],
    },

    {
      name: "مقالات",
      permissions: ['articlecategory_view', 'article_view'],

      icon: "bi-journal",
      children: [
        {
          name: "دسته‌بندی‌ها",
          permissions: ['articlecategory_view'],
          link: "/articles/categories", icon: "bi-tags"
        },
        {
          name: "لیست مقالات",
          permissions: ['article_view'],
          link: "/articles/list", icon: "bi-file-text"
        },
      ],
    },

    {
      name: "تنظیمات",
      permissions: ['settings_view'],
      icon: "bi-gear",
      link: "/settings",
    },
  ]
);
const updateWidth = () => (windowWidth.value = window.innerWidth);
onMounted(() => window.addEventListener("resize", updateWidth));
onBeforeUnmount(() => window.removeEventListener("resize", updateWidth));
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
