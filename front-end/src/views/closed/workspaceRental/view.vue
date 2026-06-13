<template>
  <div class="bg-slate-50/50 min-h-screen pb-10">
    <Toast ref="toast" />

    <div class="max-w-[1600px] mx-auto p-4 sm:p-2">
      <Loading :visible="loading" message="Syncing rental records..." />

      <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-7 flex justify-between items-center">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center text-white backdrop-blur-md border border-white/20">
              <i class="fas fa-key text-xl"></i>
            </div>
            <div>
              <h1 class="text-xl font-black text-white tracking-tight uppercase leading-none">Workspace Rentals</h1>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Guest Management & Occupancy</p>
            </div>
          </div>
          
          <button
            @click="showAddRental = true"
            class="group bg-white text-primary px-6 py-3 rounded-xl shadow-xl shadow-black/10 hover:shadow-white/20 transition-all duration-300 flex items-center gap-3 active:scale-95"
          >
            <i class="fas fa-plus text-xs group-hover:rotate-90 transition-transform"></i>
            <span class="text-[10px] font-black uppercase tracking-widest">New Rental</span>
          </button>
        </div>

        <div class="p-6 border-b border-gray-50 flex flex-col lg:flex-row justify-between items-center gap-6 bg-white">
          <div class="relative w-full lg:max-w-md">
            <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
            <input
              v-model="searchTerm"
              type="search"
              placeholder="Search by guest or space..."
              class="w-full pl-10 pr-4 py-3 bg-gray-50 border border-transparent rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all shadow-inner"
            />
          </div>

          <div class="flex items-center gap-4">
            <div class="flex items-center gap-3 bg-gray-50 px-4 py-2 rounded-xl border border-gray-100">
              <span class="text-[10px] font-black uppercase text-gray-400">View</span>
              <select
                v-model="pageSize"
                @change="fetchRentals()"
                class="bg-transparent text-xs font-black text-gray-700 outline-none cursor-pointer"
              >
                <option v-for="size in pageSizes" :key="size" :value="size">{{ size }} Entries</option>
              </select>
            </div>
          </div>
        </div>

        <div class="hidden md:block overflow-x-auto">
          <table class="w-full border-collapse">
            <thead>
              <tr class="bg-gray-50/50 text-[10px] font-black uppercase tracking-widest text-gray-400 border-b border-gray-50">
                <th class="px-6 py-5 text-left cursor-pointer hover:text-primary transition-colors" @click="sortBy('guest_name')">
                  Guest Info <SortIcon :field="'guest_name'" :sort-key="sortKey" :sort-asc="sortAsc" />
                </th>
                <th class="px-6 py-5 text-left">Contact Details</th>
                <th class="px-6 py-5 text-left">Session Details</th>
                <th class="px-6 py-5 text-left">Assigned Space</th>
                <th class="px-6 py-5 text-left">Status</th>
                <th class="px-6 py-5 text-right">Actions</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-50">
              <tr v-for="rental in filteredAndSortedRentals" :key="rental.id" class="group hover:bg-slate-50/80 transition-all">
                <td class="px-6 py-5">
                  <div class="text-sm font-black text-gray-800">{{ rental.guest_name }}</div>
                  <div class="text-[9px] font-bold text-gray-400 uppercase tracking-tighter">By {{ rental.user.first_name }}</div>
                </td>
                <td class="px-6 py-5">
                  <div class="flex flex-col gap-1">
                    <div class="flex items-center gap-2 text-xs font-bold text-gray-600">
                      <i class="fas fa-envelope text-[10px] text-gray-300 w-3"></i> {{ rental.guest_email }}
                    </div>
                    <div class="flex items-center gap-2 text-xs font-bold text-gray-600">
                      <i class="fas fa-phone text-[10px] text-gray-300 w-3"></i> {{ rental.guest_phone }}
                    </div>
                  </div>
                </td>
                <td class="px-6 py-5">
                  <div class="text-xs font-black text-gray-700 uppercase">{{ rental.cycle }}</div>
                  <div class="text-[10px] font-bold text-secondary mt-0.5">Started: {{ rental.start_date }}</div>
                </td>
                <td class="px-6 py-5">
                  <div class="inline-flex items-center gap-2 px-3 py-1.5 bg-slate-100 rounded-lg group-hover:bg-white transition-colors">
                    <span class="text-[11px] font-black text-gray-700">{{ rental.space.name || rental.space }}</span>
                    <button @click="goToSpaceDetail(rental.space.id)" class="text-primary hover:scale-110 transition-transform">
                      <i class="fas fa-external-link-alt text-[9px]"></i>
                    </button>
                  </div>
                </td>
                <td class="px-6 py-5">
                  <span
                    :class="rental.is_active ? 'bg-emerald-100 text-emerald-600' : 'bg-gray-100 text-gray-400'"
                    class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-block"
                  >
                    {{ rental.is_active ? "Active" : "Inactive" }}
                  </span>
                </td>
                <td class="px-6 py-5">
                  <div class="flex justify-end items-center gap-2">
                    <button v-if="rental.is_active" @click="openPaymentModal(rental.id)" class="px-4 py-2 bg-emerald-600 text-white rounded-xl text-[10px] font-black uppercase tracking-widest shadow-md shadow-emerald-100 hover:bg-emerald-700 transition-all active:scale-95">
                      Pay
                    </button>
                    <button @click="goToPayments(rental.id)" class="w-9 h-9 flex items-center justify-center rounded-xl bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white transition-all" title="View History">
                      <i class="fas fa-history text-xs"></i>
                    </button>
                    <button @click="editRental(rental)" class="w-9 h-9 flex items-center justify-center rounded-xl bg-slate-100 text-slate-600 hover:bg-slate-200 transition-all">
                      <i class="fas fa-edit text-xs"></i>
                    </button>
                    <button @click="askDeleteConfirmation(rental)" class="w-9 h-9 flex items-center justify-center rounded-xl bg-rose-50 text-rose-500 hover:bg-rose-500 hover:text-white transition-all">
                      <i class="fas fa-trash-alt text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredAndSortedRentals.length === 0">
                <td colspan="6" class="py-20 text-center">
                  <i class="fas fa-user-friends text-gray-100 text-5xl mb-4"></i>
                  <p class="text-[10px] font-black text-gray-300 uppercase tracking-[0.2em]">No rental contracts found</p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden p-4 space-y-4">
          <div v-for="rental in filteredAndSortedRentals" :key="rental.id" class="bg-white border border-gray-100 rounded-[2rem] p-6 shadow-sm">
            <div class="flex justify-between items-start mb-4">
              <div>
                <h3 class="font-black text-gray-800 uppercase tracking-tight">{{ rental.guest_name }}</h3>
                <p class="text-[10px] font-bold text-gray-400 tracking-widest">{{ rental.space.name || rental.space }}</p>
              </div>
              <span :class="rental.is_active ? 'text-emerald-500' : 'text-gray-300'" class="text-[9px] font-black uppercase tracking-[0.2em]">
                {{ rental.is_active ? "● Active" : "○ Inactive" }}
              </span>
            </div>

            <div class="grid grid-cols-2 gap-4 mb-6">
              <div class="p-3 bg-gray-50 rounded-2xl">
                <p class="text-[8px] font-black text-gray-400 uppercase mb-1">Cycle</p>
                <p class="text-[10px] font-bold text-gray-700 uppercase">{{ rental.cycle }}</p>
              </div>
              <div class="p-3 bg-gray-50 rounded-2xl">
                <p class="text-[8px] font-black text-gray-400 uppercase mb-1">Started</p>
                <p class="text-[10px] font-bold text-gray-700">{{ rental.start_date }}</p>
              </div>
            </div>

            <div class="flex flex-wrap gap-2">
              <button v-if="rental.is_active" @click="openPaymentModal(rental.id)" class="flex-1 py-3 bg-emerald-600 text-white rounded-xl text-[9px] font-black uppercase tracking-widest">Pay Now</button>
              <button @click="goToPayments(rental.id)" class="w-12 h-12 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center"><i class="fas fa-history"></i></button>
              <button @click="editRental(rental)" class="w-12 h-12 bg-gray-100 text-gray-500 rounded-xl flex items-center justify-center"><i class="fas fa-edit"></i></button>
              <button @click="askDeleteConfirmation(rental)" class="w-12 h-12 bg-rose-50 text-rose-500 rounded-xl flex items-center justify-center"><i class="fas fa-trash"></i></button>
            </div>
          </div>
        </div>

        <div class="p-8 border-t border-gray-50 flex items-center justify-between bg-white">
          <button @click="fetchRentals(previous)" :disabled="!previous" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-left text-xs"></i>
          </button>
          <span class="text-[10px] font-black text-gray-300 uppercase tracking-[0.3em]">Page {{ currentPage }} / {{ totalPages }}</span>
          <button @click="fetchRentals(next)" :disabled="!next" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-right text-xs"></i>
          </button>
        </div>
      </div>

      <AddRental :visible="showAddRental" @close="showAddRental = false" @success="fetchRentals" />
      <WorkspaceRentalUpdate :visible="updateVisible" :rental="rentalToEdit" @close="updateVisible = false" @refresh="fetchRentals" />
      <ConfirmModal :visible="confirmVisible" title="Confirm Deletion" message="Are you sure you want to delete this rental contract?" @confirm="confirmDelete" @cancel="confirmVisible = false" />
      <WorkspaceRentalPay v-if="paymentVisible" :visible="paymentVisible" :rentalId="selectedRentalId" @close="paymentVisible = false" @success="fetchRentals" />
    </div>
  </div>
</template>


<script>
import Toast from "@/components/Toast.vue";
import AddRental from "./add.vue";
import WorkspaceRentalUpdate from "./update.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import WorkspaceRentalPay from "./workspacerentalpay.vue";
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
  name: "WorkspaceRentalView",
  components: { Toast, AddRental, WorkspaceRentalUpdate, ConfirmModal, WorkspaceRentalPay, SortIcon,Loading },
  data() {
    return {
      searchTerm: "",
      rentals: [],
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      pageSize: 10,
      pageSizes: [5, 10, 20, 50, 100],
      showAddRental: false,
      updateVisible: false,
      rentalToEdit: null,
      confirmVisible: false,
      rentalToDelete: null,
      paymentVisible: false,
      selectedRentalId: null,
      sortKey: "guest_name",
      sortAsc: true,
    };
  },
  computed: {
    filteredAndSortedRentals() {
      const term = this.searchTerm.toLowerCase();
      return this.rentals
        .filter(
          (r) =>
            r.guest_name.toLowerCase().includes(term) ||
            r.guest_email.toLowerCase().includes(term) ||
            r.guest_phone.toLowerCase().includes(term) ||
            r.cycle.toLowerCase().includes(term) ||
            r.start_date.toLowerCase().includes(term) ||
            String(r.is_active).toLowerCase().includes(term) ||
            String(r.space?.name || r.space).toLowerCase().includes(term)
        )
        .sort((a, b) => {
          let aVal = a[this.sortKey];
          let bVal = b[this.sortKey];
          if (typeof aVal === "string") aVal = aVal.toLowerCase();
          if (typeof bVal === "string") bVal = bVal.toLowerCase();
          if (aVal < bVal) return this.sortAsc ? -1 : 1;
          if (aVal > bVal) return this.sortAsc ? 1 : -1;
          return 0;
        });
    },
  },
  mounted() {
    this.fetchRentals();
   
  },
  methods: {
    goToSpaceDetail(id) {
      this.$router.push(`/co-work-detail/${id}`);
    },
    async fetchRentals(url = null) {
      this.loading=true
      try {
        const response = await this.$getWorkspaceRentals(url, this.pageSize);
        this.rentals = response.rentals || [];
        this.currentPage = response.currentPage || 1;
        this.totalPages = response.totalPages || 1;
        this.next = response.next;
        this.previous = response.previous;
      } catch (err) {
        console.error("Failed to fetch rentals:", err);
        this.rentals = [];
      }
      finally {
        this.loading=false
      }
    },
    sortBy(key) {
      if (this.sortKey === key) this.sortAsc = !this.sortAsc;
      else this.sortKey = key;
    },
    editRental(rental) {
      this.rentalToEdit = rental;
      this.updateVisible = true;
    },
    askDeleteConfirmation(rental) {
      this.rentalToDelete = rental;
      this.confirmVisible = true;
    },
    async confirmDelete() {
      this.confirmVisible = false;
      try {
        await this.$apiDelete(`/delete_workspace_rental/${this.rentalToDelete.id}`);
        this.$root.$refs.toast.showToast("Rental deleted successfully", "success");
        this.fetchRentals();
      } catch (err) {
        console.error(err);
        this.$refs.toast.showToast("Failed to delete rental", "error");
      } finally {
        this.rentalToDelete = null;
      }
    },
    goToPayments(rental_id) {
      this.$router.push({
        name: "coworking-payments-detail",
        params: { id: rental_id },
      });
    },
    openPaymentModal(rentalId) {
      this.selectedRentalId = rentalId;
      this.paymentVisible = true;
    },
  },
};
</script>
