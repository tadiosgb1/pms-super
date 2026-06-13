<template>
  <div>
    <transition name="slide">
      <aside
        class="w-64 flex flex-col fixed md:relative z-15 h-full bg-white border-r border-slate-100"
      >
        <!-- Header -->
        <div
          v-if="showTitle"
          class="flex items-center space-x-4 p-4 font-black text-lg text-white bg-primary sticky top-0 z-10 shadow-md"
        >
          <div class="w-10 h-10 bg-white rounded-xl flex items-center justify-center">
            <img src="../../assets/img/logo1.jpg" class="h-8 w-8 rounded-lg" />
          </div>
          <p>Alpha PMS</p>
        </div>

        <!-- Menu -->
        <div class="flex-1 overflow-y-auto pt-4">
          <ul class="px-3 space-y-1">
            <li v-for="(item, index) in filteredMenuItems" :key="item.name">

              <!-- Parent Menu -->
              <div v-if="item.children">
                <button
                  @click="toggleMenu(index)"
                  class="w-full flex items-center justify-between px-4 py-2.5 rounded-xl text-sm font-bold text-primary hover:bg-slate-50"
                >
                  <div class="flex items-center">
                    <i :class="[item.icon, 'mr-3']"></i>
                    <span>{{ item.name }}</span>
                  </div>

                  <i
                    :class="openMenuIndex === index ? 'fas fa-chevron-down' : 'fas fa-chevron-right'"
                  ></i>
                </button>

                <!-- Submenu -->
                <transition name="fade">
                  <ul
                    v-if="openMenuIndex === index"
                    class="ml-8 mt-1 space-y-1"
                  >
                    <li v-for="child in item.children" :key="child.route">
                      <router-link
                        :to="{ name: child.route }"
                        class="block px-3 py-2 rounded-lg text-sm font-semibold transition"
                        :class="
                          $route.name === child.route
                            ? 'bg-primary text-white'
                            : 'text-gray-600 hover:bg-gray-100'
                        "
                      >
                        {{ child.name }}
                      </router-link>
                    </li>
                  </ul>
                </transition>
              </div>

              <!-- Single Menu -->
              <router-link
                v-else
                :to="{ name: item.route }"
                class="flex items-center px-4 py-2.5 rounded-xl text-sm font-bold transition"
                :class="
                  $route.name === item.route
                    ? 'bg-primary text-white'
                    : 'text-primary hover:bg-slate-50'
                "
              >
                <i :class="[item.icon, 'mr-3']"></i>
                {{ item.name }}
              </router-link>

            </li>
          </ul>

          <div class="h-10"></div>
        </div>
      </aside>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showTitle: false,
      is_superuser: false,
      openMenuIndex: null,

      menuItems: [

  


   




  
   
        { name: "Dashboard", route: "first-dash", icon: "fas fa-gauge" },

        {
          name: "Administration",
          icon: "fas fa-user-shield",
          children: [
            { name: "Users", route: "user_view", permission: "pms.view_user" },
            { name: "Groups", route: "groups", permission: "auth.view_group" },
            { name: "Permissions", route: "permissions_view", permission: "auth.view_permission" },
          ]
        },

      { name: "Property Types", route: "PropertyTypes-view", icon: "fas fa-gauge" },


        {
          name: "Property Management",
          icon: "fas fa-building",
          children: [
            { name: "All Properties", route: "properties", permission: "pms.view_property" },
            // { name: "Owner/Broker Properties", route: "owner-properties" },
          
            //{ name: "For Sale", route: "propertiesListForSale" },
          ]
        },

        {
          name: "Rentals",
          icon: "fas fa-file-contract",
          children: [
            { name: "Property Rentals", route: "rents", permission: "pms.view_rent" },
            { name: "Property for rent", route: "rents", permission: "pms.view_rent" },
            { name: "Shared Rentals", route: "coworking-space-rentals" },
            { name: "Overdue Rentals",  route: "rents",},
          ]
        },

         {
          name: "Sales",
          icon: "fas fa-money-bill-wave",
          children: [
            { name: "For Sale", route: "propertiesListForSale" },
            { name: "Sold", route: "propertiesListForSale" },
            {
            name: "Broker for sale",
            route: "BrokerListSales-view",
            icon: "fas fa-folder",
            color: "#22c55e"
          },
          { name: "Tenant for sale", route: "TenatProperty-view" },

          ]
        },

        {
          name: "Co-work Spaces",
          icon: "fas fa-people-roof",
          children: [
            { name: "View", route: "coworking-spaces" },
            // { name: "Tenants Property", route: "TenatProperty-view" },
          ]
        },

        {
          name: "Finance",
          icon: "fas fa-money-bill-wave",
          children: [
            { name: "Payments", route: "payments" },
            { name: "Transactions", route: "transactions" },
            { name: "Subscriptions", route: "subscriptions_view", permission: "pms.view_subscription" },
          ]
        },

        {
          name: "Operations",
          icon: "fas fa-screwdriver-wrench",
          children: [
            { name: "Maintenance", route: "maintenance-requests", permission: "pms.view_maintenancerequest" },
            { name: "Zones", route: "zones", permission: "pms.view_propertyzone" },
          ]
        },

        {
          name: "Reports & Notifications",
          icon: "fas fa-chart-pie",
          children: [
            { name: "Reports", route: "reports" },
            { name: "Notifications", route: "notifications" },
          ]
        },

        {
          name: "System",
          icon: "fas fa-gear",
          children: [
            { name: "Configurations", route: "configurations", is_superuser: true },
            //{ name: "Commission Configurations", route: "SystemConfigurations-view" },
          ]
        },

        // { name: "Contacts", route: "contacts", icon: "fas fa-address-book" },
      ],
    };
  },

  computed: {
    filteredMenuItems() {
      return this.menuItems
        .map(item => {
          if (item.children) {
            const children = item.children.filter(child => {
              if (child.is_superuser && !this.is_superuser) return false;
              if (child.permission) return this.$hasPermission(child.permission);
              return true;
            });
            return { ...item, children };
          }
          return item;
        })
        .filter(item => {
          if (item.children) return item.children.length > 0;
          if (item.is_superuser && !this.is_superuser) return false;
          if (item.permission) return this.$hasPermission(item.permission);
          return true;
        });
    },
  },

  methods: {
    toggleMenu(index) {
      this.openMenuIndex =
        this.openMenuIndex === index ? null : index;
    }
  },

  mounted() {
    this.is_superuser =
      localStorage.getItem("is_superuser") === "true";
    this.showTitle = window.innerWidth < 1024;

    // Auto-open active menu
    this.menuItems.forEach((item, index) => {
      if (item.children) {
        if (item.children.some(child => child.route === this.$route.name)) {
          this.openMenuIndex = index;
        }
      }
    });
  },
};
</script>

<style scoped>
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}
.slide-enter-from,
.slide-leave-to {
  transform: translateX(-100%);
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.2s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-5px);
}
</style>