<template>
  <div class="min-h-screen bg-background p-4 md:p-2">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Processing Maintenance Records..." />

    <div class="max-w-7xl mx-auto bg-white  rounded-xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md text-white">
            <i class="fas fa-tools text-2xl"></i>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Maintenance Requests</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Facility & Operations Management</p>
          </div>
        </div>

        <button
          v-if="!propertyId"
          @click="visible = true"
          class="bg-white text-primary hover:bg-gray-50 font-bold px-6 py-2.5 rounded-xl shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2"
        >
          <i class="fas fa-plus-circle"></i> New Request
        </button>
      </div>

      <div class="p-6 bg-gray-50/50 border-b border-gray-100">
        <div class="flex flex-col lg:flex-row justify-between gap-6">
          <div class="grid grid-cols-1 sm:grid-cols-4 gap-4 flex-1">
            <div class="space-y-1">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Start Date</label>
              <input v-model="startDate" type="date" class="custom-input h-10 !mt-0 shadow-sm" />
            </div>
            <div class="space-y-1">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">End Date</label>
              <input v-model="endDate" type="date" class="custom-input h-10 !mt-0 shadow-sm" />
            </div>
            <div class="space-y-1">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Date Type</label>
              <select v-model="filter_by" class="custom-input h-10 !mt-0 shadow-sm">
                <option value="" disabled>Filter By...</option>
                <option value="requested_at">Requested At</option>
                <option value="resolved_at">Resolved At</option>
              </select>
            </div>
            <div class="flex items-end">
              <button
                @click="applyDateFilter()"
                :disabled="!filter_by"
                class="w-full h-10 bg-primary text-white font-bold rounded-lg hover:bg-dprimary disabled:opacity-30 transition-all flex items-center justify-center gap-2 shadow-md"
              >
                <i class="fas fa-filter text-xs"></i> Apply
              </button>
            </div>
          </div>

          <div class="flex items-center gap-3 self-end lg:self-center bg-white px-4 py-2 rounded-xl border border-gray-100 shadow-sm">
            <label class="text-xs font-bold text-gray-400 uppercase tracking-wider">Show</label>
            <select v-model.number="pageSize" class="border-none text-sm font-bold text-primary focus:ring-0 cursor-pointer">
              <option v-for="size in [5, 10, 50, 100]" :key="size" :value="size">{{ size }}</option>
            </select>
            <span class="text-xs font-bold text-gray-400 uppercase tracking-wider">Rows</span>
          </div>
        </div>
      </div>

      <div class="hidden md:block p-6">
        <div class="overflow-hidden rounded-xl border border-gray-100 shadow-sm">
          <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr class="text-left text-[11px] font-black text-gray-400 uppercase tracking-widest">
                <th class="px-6 py-4 cursor-pointer hover:text-primary transition-colors" @click="sortBy('description')">
                  Description <i class="fas fa-sort ml-1 opacity-30"></i>
                </th>
                <th class="px-6 py-4 cursor-pointer hover:text-primary transition-colors" @click="sortBy('status')">
                  Status <i class="fas fa-sort ml-1 opacity-30"></i>
                </th>
                <th class="px-6 py-4">Timeline</th>
                <th class="px-6 py-4">Property & User</th>
                <th class="px-6 py-4 text-center">Action</th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-50">
              <tr v-for="maint in paginatedData" :key="maint.id" class="hover:bg-primary/5 transition-all group">
                <td class="px-6 py-4 whitespace-nowrap">
                  <p class="text-sm font-bold text-gray-700 max-w-xs truncate" :title="maint.description">
                    {{ maint.description }}
                  </p>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span 
                    :class="maint.status === 'pending' ? 'bg-amber-100 text-amber-700 border-amber-200' : 'bg-green-100 text-green-700 border-green-200'"
                    class="px-3 py-1 rounded-full text-[10px] font-black uppercase border tracking-tighter"
                  >
                    {{ maint.status }}
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex flex-col gap-1">
                    <span class="text-[10px] text-gray-400 font-bold"><i class="far fa-clock mr-1"></i>REQ: {{ formatDate(maint.requested_at) }}</span>
                    <span v-if="maint.resolved_at" class="text-[10px] text-green-500 font-bold"><i class="fas fa-check-double mr-1"></i>RES: {{ formatDate(maint.resolved_at) }}</span>
                    <span v-else class="text-[10px] text-amber-500 font-bold italic">Awaiting Resolution</span>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex flex-col gap-1">
                    <router-link v-if="maint.property_id" :to="`/dashboard/properties/${maint.property_id.id}`" class="text-xs font-bold text-primary hover:underline">
                      <i class="fas fa-building mr-1 opacity-50"></i> {{ maint.property_id.name }}
                    </router-link>
                    <router-link v-if="maint.user_id" :to="`/user_detail/${maint.user_id.id}`" class="text-[11px] font-medium text-gray-500 hover:text-primary">
                      <i class="fas fa-user mr-1 opacity-50"></i> {{ fullName(maint.user_id) }}
                    </router-link>
                  </div>
                </td>
                <td class="px-6 py-4 text-center">
                  <button
                    v-if="maint.status === 'pending'"
                    @click="confirm = true; selectedId = maint.id"
                    class="px-4 py-1.5 bg-primary text-white text-xs font-bold rounded-lg hover:bg-dprimary shadow-md transition-all active:scale-95"
                  >
                    Resolve
                  </button>
                  <span v-else class="text-green-500">
                    <i class="fas fa-check-circle text-xl"></i>
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div class="md:hidden grid gap-4 p-4">
        <div v-for="maint in paginatedData" :key="maint.id" class="bg-white rounded-2xl border border-gray-100 shadow-md p-5 relative overflow-hidden">
          <div class="absolute top-0 right-0 p-2">
            <span :class="maint.status === 'pending' ? 'text-amber-500' : 'text-green-500'">
              <i :class="maint.status === 'pending' ? 'fas fa-clock' : 'fas fa-check-circle'"></i>
            </span>
          </div>
          
          <h3 class="font-bold text-gray-800 mb-2 pr-6">{{ maint.description }}</h3>
          
          <div class="space-y-2 mb-4">
            <div class="flex items-center gap-2 text-[11px] text-gray-500">
              <i class="fas fa-building w-4 text-primary opacity-50"></i>
              <span class="font-bold tracking-tight">{{ maint.property_id ? maint.property_id.name : 'N/A' }}</span>
            </div>
            <div class="flex items-center gap-2 text-[11px] text-gray-500">
              <i class="fas fa-user w-4 text-primary opacity-50"></i>
              <span>{{ maint.user_id ? fullName(maint.user_id) : 'N/A' }}</span>
            </div>
          </div>

          <div class="flex justify-between items-center pt-4 border-t border-gray-50">
            <div class="text-[10px] font-bold text-gray-400">
              REQ: {{ formatDate(maint.requested_at) }}
            </div>
            <button
              v-if="maint.status === 'pending'"
              @click="confirm = true; selectedId = maint.id"
              class="px-4 py-2 bg-primary text-white text-xs font-black rounded-lg uppercase tracking-widest shadow-lg shadow-primary/20"
            >
              Resolve
            </button>
          </div>
        </div>
      </div>

      <div v-if="filteredData.length > 0" class="p-6 bg-gray-50/50 border-t border-gray-100 flex flex-col sm:flex-row justify-between items-center gap-4">
        <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">
          Showing Page <span class="text-primary">{{ currentPage }}</span> of {{ totalPages }}
        </p>
        <div class="flex gap-2">
          <button
            @click="prevPage"
            :disabled="currentPage === 1"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm"
          >
            <i class="fas fa-chevron-left mr-2"></i> Previous
          </button>
          <button
            @click="nextPage"
            :disabled="currentPage === totalPages"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm"
          >
            Next <i class="fas fa-chevron-right ml-2"></i>
          </button>
        </div>
      </div>
    </div>

    <div v-if="confirm" class="fixed inset-0 z-[60] flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm">
      <div class="bg-white rounded-3xl shadow-2xl max-w-sm w-full p-8 text-center border border-gray-100 transform animate-in zoom-in duration-200">
        <div class="w-20 h-20 bg-amber-50 rounded-full flex items-center justify-center mx-auto mb-6">
          <i class="fas fa-exclamation-triangle text-3xl text-amber-500"></i>
        </div>
        <h3 class="text-xl font-bold text-gray-800 mb-2">Resolve Request?</h3>
        <p class="text-gray-500 text-sm mb-8">Are you sure this maintenance issue has been completed? This action cannot be undone.</p>
        <div class="flex gap-3">
          <button @click="confirm = false" class="flex-1 px-4 py-3 rounded-xl bg-gray-100 text-gray-600 font-bold hover:bg-gray-200 transition-colors">
            Cancel
          </button>
          <button @click="resolve()" class="flex-1 px-4 py-3 rounded-xl bg-primary text-white font-bold hover:bg-dprimary shadow-lg shadow-primary/30 transition-all">
            Yes, Resolve
          </button>
        </div>
      </div>
    </div>

    <MaintenanceRequestAdd
      v-if="visible"
      :visible="visible"
      @close="visible = false; fetchMaintenance()"
    />
  </div>
</template>

<script>
import Toast from "../../../components/Toast.vue";
import MaintenanceRequestAdd from "@/views/closed/maintenanceRequests/add.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  name: "MaintenanceRequestView",
  components: { Toast, MaintenanceRequestAdd ,Loading},
   props: {
    propertyId: Number,
  },
  data() {
    return {
      maintenance: [],
      filter_by: "",
      searchTerm: "",
      startDate: "",
      endDate: "",
      currentPage: 1,
      pageSize: 5,
      sortKey: "requested_at",
      sortAsc: true,
      visible: false,
      confirm: false,
      selectedId: "",
      loading:false,
    };
  },
  computed: {
    filteredData() {
      return this.maintenance.sort((a, b) => {
        let res = 0;
        if (a[this.sortKey] < b[this.sortKey]) res = -1;
        if (a[this.sortKey] > b[this.sortKey]) res = 1;
        return this.sortAsc ? res : -res;
      });
    },
    totalPages() {
      return Math.ceil(this.filteredData.length / this.pageSize) || 1;
    },
    paginatedData() {
      const start = (this.currentPage - 1) * this.pageSize;
      return this.filteredData.slice(start, start + this.pageSize);
    },
  },
  mounted() {
    this.fetchMaintenance();
  },
  methods: {
    // helper to build params based on role
    buildRoleParams(params = {}) {
      const isSuperUser =
        localStorage.getItem("is_superuser") === "1" ||
        localStorage.getItem("is_superuser") === "true";

      const groups = JSON.parse(localStorage.getItem("groups") || "[]");
      const email = localStorage.getItem("email");

      if (!isSuperUser) {
        if (groups.includes("manager")) {
          params = { ...params, "property_id__property_zone_id__manager_id__email": email };
        } else if (groups.includes("owner")) {
          params = { ...params, "property_id__property_zone_id__owner_id__email": email };
        } else if (groups.includes("staff")) {
          params = { ...params, "staff_id__email": email };
        } else if (groups.includes("super_staff")) {
          params = { ...params };
        }
      }
       // ✅ Always filter by propertyId if available
  if (this.propertyId) {
    params = { ...params, "property_id__id": this.propertyId };
  }
      return params;
    },

    async fetchMaintenance() {
      this.loading=true;
      try {
        let params = this.buildRoleParams();
        const response = await this.$apiGet("get_maintenance_requests", params);
        this.maintenance = response.data || [];
        console.log("maintenance",this.maintenance);
      } catch (err) {
        console.error("Failed to fetch maintenance", err);
        this.maintenance = [];
        this.$refs.toast.showToast("Failed to load requests", "error");
      }finally{
        this.loading=false;
      }
    },

    async applyDateFilter() {
      try {
        let params = this.buildRoleParams();

        if (this.filter_by === "requested_at") {
          params = {
            ...params,
            requested_at__gt: this.startDate,
            requested_at__lt: this.endDate,
          };
        } else if (this.filter_by === "resolved_at") {
          params = {
            ...params,
            resolved_at__gt: this.startDate,
            resolved_at__lt: this.endDate,
          };
        }

        const res = await this.$apiGet("get_maintenance_requests", params);
        this.maintenance = res.data;
      } catch (error) {
        console.error("Failed to fetch maintenance", error);
        this.maintenance = [];
        this.$refs.toast.showToast("Failed to load requests", "error");
      }
    },

    async resolve() {
      try {
        const payload = { id: this.selectedId };
        await this.$apiPost("/resolve_maintenance_request", payload);
        this.confirm = false;
        this.fetchMaintenance(); // refetch after resolve
      } catch (error) {
        console.error("Failed to resolve request", error);
        this.$refs.toast.showToast("Failed to resolve request", "error");
      }
    },

    sortBy(field) {
      if (this.sortKey === field) {
        this.sortAsc = !this.sortAsc;
      } else {
        this.sortKey = field;
        this.sortAsc = true;
      }
    },

    prevPage() {
      if (this.currentPage > 1) this.currentPage--;
    },
    nextPage() {
      if (this.currentPage < this.totalPages) this.currentPage++;
    },
    fullName(user) {
      return [user.first_name, user.middle_name, user.last_name].filter(Boolean).join(" ");
    },
    formatDate(dateStr) {
      if (!dateStr) return "";
      return new Date(dateStr).toLocaleDateString();
    },
  },
};
</script>
