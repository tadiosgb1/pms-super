<template>
  <div class="min-h-screen bg-slate-50 p-3 md:p-4">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Co-working Spaces..." />

    <div class="max-w-7xl mx-auto bg-white rounded-lg overflow-hidden border border-slate-200 shadow-sm">
      
      <!-- Header -->
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-4 flex flex-col sm:flex-row justify-between items-center gap-4">
        <div class="flex items-center gap-3">
          <div class="bg-white/20 p-2 rounded-lg backdrop-blur-md">
            <i class="fas fa-laptop-house text-white text-lg"></i>
          </div>
          <div>
            <h2 class="text-xl font-black text-white tracking-tight uppercase">Co-Working Spaces</h2>
            <p class="text-white/70 text-xs uppercase tracking-wider font-semibold">Manage Shared Office Inventory</p>
          </div>
        </div>
        <button
          @click="showAddSpace = true"
          class="bg-white text-primary hover:bg-slate-50 font-bold px-6 py-2.5 rounded-lg shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2 text-sm"
        >
          <i class="fas fa-plus"></i> Add Space
        </button>
      </div>

      <!-- Search & Filter -->
      <div class="p-4 md:p-6 border-b border-slate-200 flex flex-col md:flex-row justify-between items-center gap-4 bg-slate-50/50">
        <div class="relative w-full md:w-1/3">
          <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400">
            <i class="fas fa-search text-sm"></i>
          </span>
          <input
            v-model="searchTerm"
            type="search"
            placeholder="Search spaces..."
            class="border-2 border-slate-200 rounded-lg pl-10 pr-4 py-2 text-sm w-full focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all"
          />
        </div>

        <div class="flex items-center gap-3 text-xs font-semibold text-slate-600">
          <label>Show</label>
          <select
            v-model="pageSize"
            @change="fetchSpaces()"
            class="border-2 border-slate-200 rounded-lg px-3 py-2 text-sm bg-white focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all"
          >
            <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
          </select>
          <span>entries</span>
        </div>
      </div>

      <!-- Desktop Table -->
      <div class="hidden md:block overflow-hidden rounded-lg border border-slate-200 shadow-sm">
        <div class="overflow-x-auto">
          <table class="w-full text-sm divide-y divide-slate-100">
            <thead class="bg-gradient-to-r from-dprimary to-primary">
              <tr>
                <th class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider cursor-pointer hover:bg-primary/90 transition-colors" @click="sortBy('name')">
                  <div class="flex items-center gap-1">
                    <i class="fas fa-building text-white/70"></i>Space Name
                    <SortIcon :field="'name'" :sort-key="sortKey" :sort-asc="sortAsc" />
                  </div>
                </th>
                <th class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-map-marker-alt text-white/70 mr-2"></i>Location
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-users text-white/70 mr-2"></i>Capacity
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-dollar-sign text-white/70 mr-2"></i>Daily Price
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-calendar text-white/70 mr-2"></i>Monthly Price
                </th>
                <th class="px-4 py-2.5 text-left text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-map-pin text-white/70 mr-2"></i>Zone
                </th>
                <th class="px-4 py-2.5 text-center text-xs font-black text-white uppercase tracking-wider">
                  <i class="fas fa-cog text-white/70"></i> Actions
                </th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-slate-100">
              <tr v-for="space in filteredAndSortedSpaces" :key="space.id" class="hover:bg-slate-50 transition-colors group">
                <td class="px-4 py-2 whitespace-nowrap">
                  <span class="text-xs font-semibold text-slate-800 group-hover:text-primary transition-colors">{{ space.name }}</span>
                </td>
                <td class="px-4 py-2 whitespace-nowrap text-xs text-slate-600">{{ space.location }}</td>
                <td class="px-4 py-2 whitespace-nowrap text-center">
                  <span class="inline-flex items-center px-2 py-1 rounded-md bg-primary/10 text-primary text-xs font-black">
                    {{ space.capacity }}
                  </span>
                </td>
                <td class="px-4 py-2 whitespace-nowrap text-center text-xs font-semibold text-slate-800">{{ space.price_daily }}</td>
                <td class="px-4 py-2 whitespace-nowrap text-center text-xs font-semibold text-slate-800">{{ space.price_monthly }}</td>
                <td class="px-4 py-2 whitespace-nowrap text-xs text-slate-600 truncate max-w-xs">{{ space.zone?.name }}</td>
                <td class="px-4 py-2 whitespace-nowrap text-center">
                  <div class="flex justify-center gap-1">
                    <button 
                      @click="goDetail(space.id)" 
                      class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all shadow-sm" 
                      title="View Detail"
                    >
                      <i class="fas fa-eye text-xs"></i>
                    </button>
                    <button 
                      @click="editSpace(space)" 
                      class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all shadow-sm" 
                      title="Edit"
                    >
                      <i class="fas fa-edit text-xs"></i>
                    </button>
                    <button 
                      @click="goToRentals(space.id)" 
                      class="w-7 h-7 flex items-center justify-center rounded-lg bg-green-50 text-green-600 hover:bg-green-600 hover:text-white transition-all shadow-sm" 
                      title="Rentals"
                    >
                      <i class="fas fa-plus text-xs"></i>
                    </button>
                    <button 
                      @click="askDeleteConfirmation(space)" 
                      class="w-7 h-7 flex items-center justify-center rounded-lg bg-red-50 text-red-600 hover:bg-red-600 hover:text-white transition-all shadow-sm" 
                      title="Delete"
                    >
                      <i class="fas fa-trash-alt text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredAndSortedSpaces.length === 0">
                <td colspan="7" class="text-center py-8 text-slate-400">
                  <div class="flex flex-col items-center">
                    <i class="fas fa-inbox text-3xl mb-2"></i>
                    <p class="text-xs font-medium">No co-working spaces found</p>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Mobile Cards -->
      <div class="md:hidden grid grid-cols-1 gap-3 p-4">
        <div v-for="space in filteredAndSortedSpaces" :key="space.id" class="bg-white border-l-4 border-l-primary rounded-lg shadow-sm p-4 border border-slate-200">
          <div class="flex justify-between items-start mb-3">
            <div>
              <p class="font-bold text-slate-800 text-sm">{{ space.name }}</p>
              <p class="text-xs text-slate-400 flex items-center gap-1 mt-1">
                <i class="fas fa-map-marker-alt text-[9px]"></i> {{ space.location }}
              </p>
            </div>
            <span class="bg-primary/10 text-primary text-[9px] font-black px-2 py-1 rounded-md">{{ space.capacity }} Seats</span>
          </div>
          
          <div class="grid grid-cols-2 gap-2 mb-3 text-xs">
            <div class="bg-slate-50 p-2 rounded-lg border border-slate-200">
              <p class="text-slate-400 font-bold uppercase tracking-tighter text-[9px]">Daily</p>
              <p class="font-black text-slate-700">{{ space.price_daily }}</p>
            </div>
            <div class="bg-slate-50 p-2 rounded-lg border border-slate-200">
              <p class="text-slate-400 font-bold uppercase tracking-tighter text-[9px]">Monthly</p>
              <p class="font-black text-slate-700">{{ space.price_monthly }}</p>
            </div>
          </div>

          <div class="flex justify-between items-center pt-3 border-t border-slate-200">
            <p class="text-xs font-semibold text-slate-600">{{ space.zone?.name }}</p>
            <div class="flex gap-2">
              <button @click="goDetail(space.id)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all text-xs">
                <i class="fas fa-eye"></i>
              </button>
              <button @click="editSpace(space)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all text-xs">
                <i class="fas fa-edit"></i>
              </button>
              <button @click="askDeleteConfirmation(space)" class="w-7 h-7 flex items-center justify-center rounded-lg bg-red-50 text-red-600 hover:bg-red-600 hover:text-white transition-all text-xs">
                <i class="fas fa-trash-alt"></i>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Pagination -->
      <div class="px-6 py-4 bg-slate-50 border-t border-slate-200 flex flex-col sm:flex-row justify-between items-center gap-4">
        <p class="text-xs font-semibold text-slate-600 uppercase tracking-wider">
          Page <span class="text-primary font-black">{{ currentPage }}</span> of {{ totalPages }}
        </p>
        <div class="flex gap-2">
          <button
            :disabled="!previous"
            @click="fetchSpaces(previous)"
            class="px-4 py-2 rounded-lg text-xs font-black uppercase tracking-wider transition-all bg-slate-100 text-slate-600 hover:bg-slate-200 disabled:opacity-30 disabled:cursor-not-allowed shadow-sm"
          >
            <i class="fas fa-chevron-left mr-2"></i> Previous
          </button>
          <button
            :disabled="!next"
            @click="fetchSpaces(next)"
            class="px-4 py-2 rounded-lg text-xs font-black uppercase tracking-wider transition-all bg-gradient-to-r from-primary to-dprimary text-white hover:shadow-lg disabled:opacity-30 disabled:cursor-not-allowed shadow-md"
          >
            Next <i class="fas fa-chevron-right ml-2"></i>
          </button>
        </div>
      </div>
    </div>

    <AddSpace :visible="showAddSpace" @close="showAddSpace = false" @success="fetchSpaces" />
    <UpdateCoworkspace :visible="updateVisible" :space="spaceToEdit" @close="updateVisible = false" @refresh="fetchSpaces" />
    <ConfirmModal
      v-if="confirmVisible"
      :visible="confirmVisible"
      title="Confirm Deletion"
      message="This action will permanently remove this co-working space and all associated booking configurations. Proceed?"
      @confirm="confirmDelete"
      @cancel="confirmVisible = false"
    />
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddSpace from "./Add.vue";
import UpdateCoworkspace from "./update.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Loading from "@/components/Loading.vue";

const SortIcon = {
  props: ["field", "sortKey", "sortAsc"],
  template: `
    <span class="inline-block ml-1 text-white/70">
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
  name: "CoworkingSpacesView",
  components: { SortIcon, Toast, AddSpace, UpdateCoworkspace, ConfirmModal, Loading },
  data() {
    return {
      searchTerm: "",
      sortKey: "name",
      sortAsc: true,
      spaces: [],
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      pageSize: 10,
      pageSizes: [5, 10, 20, 50, 100],
      showAddSpace: false,
      updateVisible: false,
      spaceToEdit: null,
      confirmVisible: false,
      spaceToDelete: null,
      loading: false,
    };
  },
  computed: {
    filteredAndSortedSpaces() {
      const term = this.searchTerm.toLowerCase();
      let filtered = this.spaces.filter(
        (space) =>
          space.name.toLowerCase().includes(term) ||
          space.location.toLowerCase().includes(term) ||
          space.description.toLowerCase().includes(term) ||
          String(space.capacity).includes(term) ||
          String(space.price_daily).includes(term) ||
          String(space.price_monthly).includes(term)
      );

      filtered.sort((a, b) => {
        let aVal = a[this.sortKey];
        let bVal = b[this.sortKey];

        if (typeof aVal === "string") aVal = aVal.toLowerCase();
        if (typeof bVal === "string") bVal = bVal.toLowerCase();

        if (aVal < bVal) return this.sortAsc ? -1 : 1;
        if (aVal > bVal) return this.sortAsc ? 1 : -1;
        return 0;
      });

      return filtered;
    },
  },
  mounted() {
    this.fetchSpaces();
  },
  methods: {
    goDetail(id) {
      this.$router.push({
        name: "co-work-detail",
        params: { id: id },
      });
    },
    async fetchSpaces(url = null) {
      this.loading = true;
      try {
        const response = await this.$getCoworkingSpaces(url, this.pageSize);
        this.spaces = response.spaces || [];
        this.currentPage = response.currentPage || 1;
        this.totalPages = response.totalPages || 1;
        this.next = response.next;
        this.previous = response.previous;
      } catch (err) {
        console.error("Failed to fetch coworking spaces:", err);
        this.spaces = [];
      } finally {
        this.loading = false;
      }
    },
    sortBy(key) {
      if (this.sortKey === key) this.sortAsc = !this.sortAsc;
      else this.sortKey = key;
    },
    editSpace(space) {
      this.spaceToEdit = space;
      this.updateVisible = true;
    },
    askDeleteConfirmation(space) {
      this.spaceToDelete = space;
      this.confirmVisible = true;
    },
    async confirmDelete() {
      this.confirmVisible = false;
      try {
        await this.$apiDelete(`/delete_coworking_space/${this.spaceToDelete.id}`);
        this.$root.$refs.toast.showToast("Co-Working Space deleted successfully", "success");
        this.fetchSpaces();
      } catch (err) {
        console.error(err);
        this.$refs.toast.showToast("Failed to delete co-working space", "error");
      } finally {
        this.spaceToDelete = null;
      }
    },
    goToRentals(workspace_id) {
      this.$router.push({
        path: "/coworking-space-rentals",
        query: { workspace_id },
      });
    },
  },
};
</script>
