<template>
  <div class="min-h-screen bg-slate-50 p-3 md:p-4">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Property Zones..." />

    <div class="max-w-7xl mx-auto bg-white rounded-lg overflow-hidden border border-slate-200 shadow-sm">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-4 flex flex-col sm:flex-row justify-between items-center gap-4">
        <div class="flex items-center gap-3">
          <div class="bg-white/20 p-2 rounded-lg backdrop-blur-md">
            <i class="fas fa-map text-white text-lg"></i>
          </div>
          <div>
            <h1 class="text-xl font-black text-white tracking-tight uppercase">Property Zones</h1>
            <p class="text-white/70 text-xs uppercase tracking-widest font-semibold">Real Estate Portfolio</p>
          </div>
        </div>

        <button
          v-if="$hasPermission('pms.add_propertyzone')"
          @click="visible = true"
          class="bg-white text-primary hover:bg-slate-50 font-bold px-6 py-2.5 rounded-lg shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2 text-sm"
        >
          <i class="fas fa-plus"></i> Add Zone
        </button>
      </div>

      <div class="p-4 md:p-6">
        <div class="flex flex-col md:flex-row justify-between items-center mb-6 gap-4">
          <div class="relative w-full md:w-1/3">
            <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400">
              <i class="fas fa-search text-sm"></i>
            </span>
            <input
              v-model="searchTerm"
              @input="onSearch"
              type="search"
              placeholder="Search zones..."
              class="border-2 border-slate-200 rounded-lg pl-10 pr-4 py-2 text-sm w-full focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all"
            />
          </div>

          <div class="flex items-center gap-3">
            <div class="flex items-center gap-2 text-xs font-semibold text-slate-600">
              <label for="pageSize">Show</label>
              <select
                id="pageSize"
                v-model="pageSize"
                @change="fetchZones()"
                class="border-2 border-slate-200 rounded-lg px-3 py-2 text-sm bg-white focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all"
              >
                <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
              </select>
            </div>
            <div class="text-xs font-semibold text-slate-600">
              Total: <span class="text-primary font-black">{{ filteredAndSortedZones.length }}</span>
            </div>
          </div>
        </div>

        <div class="hidden lg:block overflow-hidden rounded-lg border border-slate-200 shadow-sm">
          <table class="min-w-full divide-y divide-slate-100">
            <thead class="bg-gradient-to-r from-dprimary to-primary">
              <tr>
                <th @click="sortBy('name')" class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider cursor-pointer hover:bg-primary/90 transition-colors">
                  <div class="flex items-center gap-1">
                    <i class="fas fa-map-pin text-white/70"></i>Zone Name
                    <SortIcon :field="'name'" :sort-key="sortKey" :sort-asc="sortAsc" />
                  </div>
                </th>
                <th class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-location-dot text-white/70 mr-2"></i>Location
                </th>
                <th class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-users text-white/70 mr-2"></i>Stakeholders
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-map text-white/70"></i> Map
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-cog text-white/70"></i> Actions
                </th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-slate-100">
              <tr v-for="zone in filteredAndSortedZones" :key="zone.id" class="hover:bg-slate-50 transition-colors group">
                <td class="px-4 py-2 whitespace-nowrap">
                  <div class="text-xs font-semibold text-slate-800 group-hover:text-primary transition-colors">{{ zone.name }}</div>
                  <div class="text-[9px] text-slate-400 font-mono">ID: {{ zone.id }}</div>
                </td>
                <td class="px-4 py-2 whitespace-nowrap">
                  <div class="text-xs text-slate-700 font-medium">{{ zone.address }}</div>
                  <div class="text-[10px] text-slate-400">{{ zone.city }}, {{ zone.state }}</div>
                </td>
                <td class="px-4 py-2">
                  <div class="flex flex-col gap-0.5 text-xs">
                    <div class="flex items-center gap-1">
                      <span class="text-[8px] font-black text-slate-400 uppercase">Owner:</span>
                      <button @click="goToDetail(zone.owner_id)" class="text-primary hover:underline font-semibold">{{ zone.ownerName }}</button>
                    </div>
                    <div class="flex items-center gap-1">
                      <span class="text-[8px] font-black text-slate-400 uppercase">Mgr:</span>
                      <button @click="goToDetail(zone.manager_id)" class="text-primary hover:underline font-semibold">{{ zone.managerName }}</button>
                    </div>
                  </div>
                </td>
                <td class="px-4 py-2 text-center">
                  <a :href="`https://www.google.com/maps/search/?api=1&query=${zone.latlong}`" target="_blank" 
                     class="inline-flex items-center gap-1 px-2 py-1 rounded-lg bg-red-50 text-red-600 text-[9px] font-bold hover:bg-red-600 hover:text-white transition-all">
                    <i class="fa-solid fa-location-dot"></i> View
                  </a>
                </td>
                <td class="px-4 py-2 whitespace-nowrap">
                  <div class="flex justify-center items-center gap-1">
                    <button @click="goToZoneDetail(zone.id)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all shadow-sm" title="View Detail">
                      <i class="fas fa-eye text-xs"></i>
                    </button>
                    <button v-if="$hasPermission('pms.change_propertyzone')" @click="editZone(zone)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all shadow-sm" title="Edit">
                      <i class="fas fa-edit text-xs"></i>
                    </button>
                    <button @click="properties(zone.id)" class="px-2 h-7 flex items-center justify-center rounded-lg bg-slate-100 text-slate-600 hover:bg-slate-800 hover:text-white transition-all text-[8px] font-black uppercase tracking-tighter" title="Units">
                      Units
                    </button>
                    <button @click="openSaleModal(zone.id)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-yellow-50 text-yellow-600 hover:bg-yellow-500 hover:text-white transition-all shadow-sm" title="List for Sale">
                      <i class="fas fa-tag text-xs"></i>
                    </button>
                    <button v-if="$hasPermission('pms.delete_propertyzone')" @click="askDeleteConfirmation(zone)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-red-50 text-red-600 hover:bg-red-600 hover:text-white transition-all shadow-sm" title="Delete">
                      <i class="fas fa-trash-alt text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="lg:hidden grid grid-cols-1 gap-3">
          <div v-for="zone in filteredAndSortedZones" :key="zone.id" class="bg-white rounded-lg border-l-4 border-l-primary shadow-sm p-4 border border-slate-200">
            <div class="flex justify-between items-start mb-3">
               <div>
                 <h3 class="font-bold text-slate-900 text-sm">{{ zone.name }}</h3>
                 <p class="text-xs text-slate-400 flex items-center gap-1 mt-1">
                   <i class="fas fa-map-marker-alt text-red-400"></i> {{ zone.city }}, {{ zone.address }}
                 </p>
               </div>
               <span class="px-2 py-1 rounded-md bg-primary/10 text-primary text-[9px] font-black uppercase">{{ zone.state }}</span>
            </div>

            <div class="grid grid-cols-2 gap-3 mb-4 p-2 bg-slate-50 rounded-lg text-xs">
              <div>
                <p class="text-[8px] font-black text-slate-400 uppercase tracking-widest">Owner</p>
                <p class="font-semibold text-slate-700 truncate">{{ zone.ownerName }}</p>
              </div>
              <div>
                <p class="text-[8px] font-black text-slate-400 uppercase tracking-widest">Manager</p>
                <p class="font-semibold text-slate-700 truncate">{{ zone.managerName }}</p>
              </div>
            </div>

            <div class="flex flex-wrap gap-2">
              <button @click="goToZoneDetail(zone.id)" class="flex-1 py-2 bg-primary/10 text-primary text-[9px] font-black uppercase rounded-lg hover:bg-primary hover:text-white transition-all">Info</button>
              <button @click="editZone(zone)" class="flex-1 py-2 bg-primary/10 text-primary text-[9px] font-black uppercase rounded-lg hover:bg-primary hover:text-white transition-all">Edit</button>
              <button @click="properties(zone.id)" class="flex-1 py-2 bg-slate-100 text-slate-600 text-[9px] font-black uppercase rounded-lg hover:bg-slate-800 hover:text-white transition-all">Units</button>
              <button @click="openSaleModal(zone.id)" class="w-10 flex items-center justify-center bg-yellow-50 text-yellow-600 rounded-lg hover:bg-yellow-500 hover:text-white transition-all"><i class="fas fa-tag text-xs"></i></button>
            </div>
          </div>
        </div>

        <div class="mt-6 flex flex-col sm:flex-row justify-between items-center gap-4 pt-4 border-t border-slate-200">
          <p class="text-xs font-semibold text-slate-600 uppercase tracking-wider">
            Page <span class="text-primary font-black">{{ currentPage }}</span> of {{ totalPages }}
          </p>
          <div class="flex gap-2">
            <button
              :disabled="currentPage <= 1"
              @click="changePage(currentPage - 1)"
              class="px-4 py-2 rounded-lg bg-slate-100 text-slate-600 font-bold text-xs uppercase tracking-wider hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm"
            >
              Previous
            </button>
            <button
              :disabled="currentPage >= totalPages"
              @click="changePage(currentPage + 1)"
              class="px-4 py-2 rounded-lg bg-slate-100 text-slate-600 font-bold text-xs uppercase tracking-wider hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm"
            >
              Next
            </button>
          </div>
        </div>
      </div>
    </div>

    <AddZone v-if="visible" :visible="visible" @close="visible = false" @refresh="fetchZones" />
    <UpdateZone v-if="updateVisible" :visible="updateVisible" :zone="zoneToEdit" @close="updateVisible = false" @refresh="fetchZones" />
    <ConfirmModal v-if="confirmVisible" :visible="confirmVisible" title="Delete Zone" message="Are you sure you want to remove this property zone from the system?" @confirm="confirmDelete" @cancel="confirmVisible = false" />
    <SaleModal v-if="saleVisible" :visible="saleVisible" :propertyZoneId="salePropertyId" sourceType="zone" @close="saleVisible = false" @refresh="fetchZones" />
  </div>
</template>

<script>
import Toast from "../../../components/Toast.vue";
import AddZone from "./add.vue";
import UpdateZone from "./update.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import SaleModal from "../propertiesListForSale/add.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

const SortIcon = {
  props: ["field", "sortKey", "sortAsc"],
  template: `
    <span class="inline-block ml-1 text-gray-500">
      <svg v-if="sortKey !== field" xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/>
      </svg>
      <svg v-else-if="sortAsc" xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 13l4 4 4-4m0-6l-4-4-4 4"/>
      </svg>
      <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/>
      </svg>
    </span>
  `,
};

export default {
  components: { AddZone, UpdateZone, ConfirmModal, SortIcon, Toast, SaleModal, Loading },
  data() {
    return {
      globalZones: [],
      visible: false,
      updateVisible: false,
      confirmVisible: false,
      zoneToEdit: null,
      zoneToDelete: null,
      searchTerm: "",
      sortKey: "name",
      sortAsc: true,
      currentPage: 1,
      totalPages: 1,
      pageSize: 10,
      pageSizes: [1, 5, 10, 20, 50, 100],
      saleVisible: false,
      salePropertyId: null,
      loading: false,
    };
  },
  computed: {
    filteredAndSortedZones() {
      let sorted = [...this.globalZones];
      sorted.sort((a, b) => {
        let res = 0;
        if (a[this.sortKey] < b[this.sortKey]) res = -1;
        if (a[this.sortKey] > b[this.sortKey]) res = 1;
        return this.sortAsc ? res : -res;
      });
      return sorted;
    },
  },
  async mounted() {
    await this.fetchZones();
  },
  methods: {
    openSaleModal(propertyZoneId) {
      this.salePropertyId = propertyZoneId;
      this.saleVisible = true;
    },
    async fetchZones() {
      this.loading = true;
      try {
        const params = { ordering: "-id" };
        const url = `/get_property_zones?page=${this.currentPage}&page_size=${this.pageSize}&search=${this.searchTerm}`;
        const result = await this.$getZones(url, params);
        this.globalZones = result.zones;
        this.currentPage = result.currentPage;
        this.totalPages = result.totalPages;
      } catch (err) {
        console.error("Error fetching zones:", err);
        this.globalZones = [];
      } finally {
        this.loading = false;
      }
    },
    onSearch() {
      this.currentPage = 1;
      this.fetchZones();
    },
    changePage(page) {
      this.currentPage = page;
      this.fetchZones();
    },
    editZone(zone) {
      this.zoneToEdit = zone;
      this.updateVisible = true;
    },
    askDeleteConfirmation(zone) {
      this.zoneToDelete = zone;
      this.confirmVisible = true;
    },
    async confirmDelete() {
      this.confirmVisible = false;
      try {
        const response = await this.$apiDelete(
          `/delete_property_zone/${this.zoneToDelete.id}`
        );
        if (response && response.error) {
          this.$root.$refs.toast.showToast(
            response.error || "Failed to delete zone",
            "error"
          );
        } else {
          this.$root.$refs.toast.showToast(
            "Zone deleted successfully",
            "success"
          );
          await this.fetchZones();
        }
      } catch (err) {
        this.$root.$refs.toast.showToast("Failed to delete zone", "error");
        console.error(err);
      }
      this.zoneToDelete = null;
    },


    goToZoneDetail(propertyZoneId) {
      if (propertyZoneId)
        this.$router.push(`/zones/${propertyZoneId}`);
    },


    properties(zoneId) {
      this.$router.push(`/properties?zone=${zoneId}`);
    },
    goToDetail(id) {
      if (id) this.$router.push(`/user_detail/${id}`);
    },
    sortBy(field) {
      if (this.sortKey === field) this.sortAsc = !this.sortAsc;
      else {
        this.sortKey = field;
        this.sortAsc = true;
      }
    },
  },
};
</script>
