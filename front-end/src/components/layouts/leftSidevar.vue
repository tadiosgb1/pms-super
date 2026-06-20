<template>
  <div>
    <transition name="slide">
      <aside class="w-52 flex flex-col fixed md:relative z-15 h-full bg-white border-r border-slate-100">

        <!-- Mobile header (only shows when sidebar is overlaid) -->
        <div v-if="showTitle"
          class="flex items-center gap-3 px-4 py-3.5 bg-white border-b border-slate-100 sticky top-0 z-10">
          <div class="w-8 h-8 rounded-lg overflow-hidden shrink-0 shadow-sm">
            <img src="../../assets/img/logo1.jpg" class="h-full w-full object-cover" />
          </div>
          <div>
            <p class="text-sm font-black text-slate-800 leading-none">Alpha PMS</p>
            <p class="text-[9px] text-slate-400 font-semibold uppercase tracking-widest mt-0.5">Super Admin</p>
          </div>
        </div>

        <!-- Menu -->
        <div class="flex-1 overflow-y-auto py-3 sidebar-inner">
          <ul class="px-2 space-y-0.5">
            <li v-for="(item, index) in filteredMenuItems" :key="item.name">

              <!-- Parent/Group Menu -->
              <div v-if="item.children">
                <button
                  @click="toggleMenu(index)"
                  class="w-full flex items-center justify-between px-3 py-2.5 rounded-lg transition-all group"
                  :class="openMenuIndex === index
                    ? 'bg-primary/10 text-primary'
                    : 'text-slate-800 hover:bg-slate-100 hover:text-slate-900'"
                >
                  <div class="flex items-center gap-2.5">
                    <div class="w-5 h-5 flex items-center justify-center rounded shrink-0"
                      :class="openMenuIndex === index ? 'text-primary' : 'text-slate-500 group-hover:text-slate-700'">
                      <i :class="[item.icon, 'text-xs']"></i>
                    </div>
                    <span class="text-xs font-bold tracking-tight">{{ item.name }}</span>
                  </div>
                  <i class="fas text-[10px] transition-transform duration-200 text-slate-400"
                    :class="openMenuIndex === index ? 'fa-chevron-down' : 'fa-chevron-right'">
                  </i>
                </button>

                <!-- Submenu -->
                <transition name="fade">
                  <ul v-if="openMenuIndex === index" class="mt-0.5 mb-1 ml-3 pl-3 space-y-0.5 border-l-2 border-slate-200">
                    <li v-for="child in item.children" :key="child.route">
                      <router-link
                        :to="{ name: child.route }"
                        class="flex items-center gap-2 px-2.5 py-2 rounded-md text-xs font-semibold transition-all"
                        :class="$route.name === child.route
                          ? 'bg-primary text-white shadow-sm'
                          : 'text-slate-700 hover:bg-primary/10 hover:text-primary'"
                      >
                        <span class="w-1.5 h-1.5 rounded-full shrink-0 transition-colors"
                          :class="$route.name === child.route ? 'bg-white' : 'bg-slate-300'">
                        </span>
                        {{ child.name }}
                      </router-link>
                    </li>
                  </ul>
                </transition>
              </div>

              <!-- Single Item -->
              <router-link
                v-else
                :to="{ name: item.route }"
                class="flex items-center gap-2.5 px-3 py-2.5 rounded-lg text-xs font-bold transition-all group"
                :class="$route.name === item.route
                  ? 'bg-primary text-white shadow-sm'
                  : 'text-slate-800 hover:bg-slate-100 hover:text-slate-900'"
              >
                <div class="w-5 h-5 flex items-center justify-center shrink-0">
                  <i :class="[item.icon, 'text-xs',
                    $route.name === item.route ? 'text-white' : 'text-slate-500 group-hover:text-slate-700']">
                  </i>
                </div>
                {{ item.name }}
              </router-link>

            </li>
          </ul>

          <!-- Section divider label helper (visual spacing) -->
          <div class="h-6"></div>
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
      userRole: null,       // 'superuser' | 'super_staff'
      openMenuIndex: null,

      menuItems: [
        // ── Dashboard (everyone) ────────────────────────────────────────
        {
          name: "Dashboard",
          route: "first-dash",
          icon: "fas fa-gauge",
        },

        // ── SaaS Administration (superuser only) ────────────────────────
        {
          name: "Administration",
          icon: "fas fa-user-shield",
          roles: ["superuser"],
          children: [
            { name: "Users",       route: "user_view",        permission: "pms.view_user" },
            { name: "Groups",      route: "groups",           permission: "auth.view_group" },
            { name: "Permissions", route: "permissions_view", permission: "auth.view_permission" },
          ],
        },

        // ── Client Management (superuser only) ──────────────────────────
        {
          name: "Client Management",
          icon: "fas fa-users-gear",
          roles: ["superuser"],
          children: [
            { name: "Owners",   route: "owners" },
            { name: "Brokers",  route: "brokers" },
          ],
        },

        // ── Plans & Subscriptions (superuser only) ──────────────────────
        {
          name: "Plans & Subscriptions",
          icon: "fas fa-clipboard-list",
          roles: ["superuser"],
          children: [
            { name: "Plans",         route: "plans_view" },
            { name: "Subscriptions", route: "subscriptions_view", permission: "pms.view_subscription" },
          ],
        },

        // ── Property Types (superuser + super_staff) ────────────────────
        {
          name: "Property Types",
          route: "PropertyTypes-view",
          icon: "fas fa-list-check",
          roles: ["superuser", "super_staff"],
        },

        // ── Property Management (superuser + super_staff) ───────────────
        {
          name: "Property Management",
          icon: "fas fa-building",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "All Properties", route: "properties", permission: "pms.view_property" },
          ],
        },

        // ── Rentals (superuser + super_staff) ───────────────────────────
        {
          name: "Rentals",
          icon: "fas fa-file-contract",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "Property Rentals", route: "rents",                  permission: "pms.view_rent" },
            { name: "Shared Rentals",   route: "coworking-space-rentals" },
            { name: "Overdue Rentals",  route: "rents" },
          ],
        },

        // ── Sales (superuser + super_staff) ─────────────────────────────
        {
          name: "Sales",
          icon: "fas fa-money-bill-wave",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "For Sale",       route: "propertiesListForSale" },
            { name: "Sold",           route: "propertiesListForSale" },
            { name: "Broker for sale", route: "BrokerListSales-view" },
            { name: "Tenant for sale", route: "TenatProperty-view" },
          ],
        },

        // ── Co-work Spaces (superuser + super_staff) ─────────────────────
        {
          name: "Co-work Spaces",
          icon: "fas fa-people-roof",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "Spaces",  route: "coworking-spaces" },
            { name: "Rentals", route: "coworking-space-rentals" },
          ],
        },

        // ── Finance (superuser + super_staff) ───────────────────────────
        {
          name: "Finance",
          icon: "fas fa-coins",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "Payments",     route: "payments" },
            { name: "Transactions", route: "transactions" },
            { name: "Commissions — Sale", route: "sale-commissions" },
            { name: "Commissions — Rent", route: "rent-commissions" },
          ],
        },

        // ── Operations (superuser + super_staff) ────────────────────────
        {
          name: "Operations",
          icon: "fas fa-screwdriver-wrench",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "Maintenance", route: "maintenance-requests", permission: "pms.view_maintenancerequest" },
            { name: "Zones",       route: "zones",                permission: "pms.view_propertyzone" },
          ],
        },

        // ── Reports & Notifications (everyone) ──────────────────────────
        {
          name: "Reports & Notifications",
          icon: "fas fa-chart-pie",
          roles: ["superuser", "super_staff"],
          children: [
            { name: "Reports",       route: "reports" },
            { name: "Notifications", route: "notifications" },
          ],
        },

        // ── System (superuser only) ──────────────────────────────────────
        {
          name: "System",
          icon: "fas fa-gear",
          roles: ["superuser"],
          children: [
            { name: "Configurations",        route: "configurations" },
            { name: "System Configurations", route: "SystemConfigurations-view" },
          ],
        },
      ],
    };
  },

  computed: {
    filteredMenuItems() {
      return this.menuItems
        .map(item => {
          if (item.children) {
            const children = item.children.filter(child => {
              // Role check on individual child
              if (child.roles && !child.roles.includes(this.userRole)) return false;
              // Permission check
              if (child.permission) return this.$hasPermission(child.permission);
              return true;
            });
            return { ...item, children };
          }
          return item;
        })
        .filter(item => {
          // Role check at group/item level
          if (item.roles && !item.roles.includes(this.userRole)) return false;
          // Hide groups whose children were all filtered out
          if (item.children) return item.children.length > 0;
          // Permission check for standalone items
          if (item.permission) return this.$hasPermission(item.permission);
          return true;
        });
    },
  },

  methods: {
    toggleMenu(index) {
      this.openMenuIndex = this.openMenuIndex === index ? null : index;
    },
  },

  mounted() {
    // Read role from localStorage — expected values: 'superuser' | 'super_staff'
    const isSuperuser = localStorage.getItem("is_superuser") === "true";
    const storedRole  = localStorage.getItem("role");

    if (isSuperuser) {
      this.userRole = "superuser";
    } else if (storedRole === "super_staff") {
      this.userRole = "super_staff";
    } else {
      // Fallback: treat any authenticated super-portal session as super_staff
      this.userRole = storedRole || "super_staff";
    }

    this.showTitle = window.innerWidth < 1024;

    // Auto-open the group that contains the current route
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
  transition: all 0.18s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-4px);
}

/* Thin scrollbar for sidebar inner */
.sidebar-inner {
  scrollbar-width: thin;
  scrollbar-color: #e2e8f0 transparent;
}
.sidebar-inner::-webkit-scrollbar {
  width: 3px;
}
.sidebar-inner::-webkit-scrollbar-track {
  background: transparent;
}
.sidebar-inner::-webkit-scrollbar-thumb {
  background-color: #e2e8f0;
  border-radius: 99px;
}
.sidebar-inner:hover::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
}
</style>