<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-50 via-white to-tertiary/10">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading rent ledger..." />

    <!-- Header Section -->
    <div class="px-6 pt-6 pb-4 flex justify-between items-center">
      <button
        @click="$router.back()"
        class="group flex items-center gap-2 px-4 py-2 bg-white border-2 border-slate-200 rounded-xl text-xs font-black uppercase tracking-widest text-slate-600 hover:text-primary hover:border-primary transition-all"
      >
        <i class="fas fa-arrow-left group-hover:-translate-x-1 transition-transform"></i>
        Back
      </button>
      
      <div class="flex items-center gap-2 bg-white px-3 py-1.5 rounded-lg border-2 border-slate-200">
        <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
        <span class="text-[10px] font-black text-slate-500 uppercase tracking-widest">Live Billing System</span>
      </div>
    </div>

    <!-- Main Content -->
    <div class="px-6 pb-6">
      <div class="bg-white rounded-[2rem] border-2 border-slate-200 overflow-hidden">
        
        <!-- Header Banner -->
        <div class="bg-gradient-to-r from-primary to-dprimary px-8 py-6">
          <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
              <h1 class="text-2xl font-black text-white tracking-tight">Lease & Rent Ledger</h1>
              <p class="text-white/70 text-xs font-bold uppercase tracking-widest mt-1">Tenant Occupancy & Financials</p>
            </div>
            
            <button
              @click="visible = true"
              class="bg-white text-primary px-6 py-3 rounded-xl font-black text-xs uppercase tracking-widest transition-all hover:scale-105 active:scale-95 flex items-center justify-center gap-2 shadow-lg"
            >
              <i class="fas fa-plus"></i>
              Create Lease
            </button>
          </div>
        </div>

        <!-- Controls Section -->
        <div class="px-8 py-5 border-b-2 border-slate-100 flex flex-col sm:flex-row justify-between items-center gap-4">
          <div class="relative w-full sm:max-w-xs">
            <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-primary"></i>
            <input
              v-model="searchTerm"
              @input="onSearch"
              type="search"
              placeholder="Search tenant or property..."
              class="w-full bg-slate-50 border-2 border-slate-200 rounded-xl py-2.5 pl-11 pr-4 text-sm font-bold focus:bg-white focus:ring-2 focus:ring-primary/50 focus:border-primary outline-none transition-all"
            />
          </div>

          <div class="flex items-center gap-3 bg-slate-50 px-4 py-2.5 rounded-xl border-2 border-slate-200">
            <label class="text-xs font-black text-slate-600 uppercase tracking-wider">Show</label>
            <select v-model="pageSize" @change="fetchRents()" class="bg-transparent font-black text-primary text-xs outline-none cursor-pointer">
              <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
            </select>
            <span class="text-xs font-black text-slate-600 uppercase tracking-wider">entries</span>
          </div>
        </div>

        <!-- Desktop Table -->
        <div class="hidden lg:block overflow-x-auto">
          <table class="w-full">
            <thead>
              <tr class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 border-b-2 border-slate-200">
                <th class="px-6 py-3 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-file-contract text-primary mr-2"></i>Agreement
                </th>
                <th class="px-6 py-3 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-calendar text-primary mr-2"></i>Timeline
                </th>
                <th class="px-6 py-3 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-dollar-sign text-primary mr-2"></i>Finances
                </th>
                <th class="px-6 py-3 text-center text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-circle-info text-primary mr-2"></i>Status
                </th>
                <th class="px-6 py-3 text-right text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-cog text-primary mr-2"></i>Actions
                </th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-100">
              <tr v-for="rent in rents" :key="rent.id" class="hover:bg-slate-50 transition-colors duration-150">
                <td class="px-6 py-3">
                  <div class="space-y-1">
                    <p @click="goToPropertyDetail(rent.property_id.id)" class="text-sm font-black text-slate-800 hover:text-primary cursor-pointer transition-colors">
                      {{ rent.property_id?.name || 'N/A' }}
                    </p>
                    <div @click="goToUserDetail(rent.user_id.id)" class="flex items-center gap-2 text-xs font-bold text-slate-500 hover:text-primary cursor-pointer transition-colors">
                      <i class="fas fa-user text-xs"></i>
                      {{ rent.user_id?.first_name }} {{ rent.user_id?.last_name }}
                    </div>
                  </div>
                </td>

                <td class="px-6 py-3">
                  <div class="flex flex-col gap-1">
                    <span class="text-xs font-black text-slate-700">{{ new Date(rent.start_date).toLocaleDateString() }}</span>
                    <span class="text-xs font-bold text-slate-500">→ {{ new Date(rent.end_date).toLocaleDateString() }}</span>
                  </div>
                </td>

                <td class="px-6 py-3">
                  <div class="flex flex-col gap-1">
                    <span class="text-sm font-black text-primary">${{ rent.rent_amount }}</span>
                    <span class="text-xs font-bold text-slate-500 uppercase">{{ rent.payment_cycle }}</span>
                  </div>
                </td>

                <td class="px-6 py-3 text-center">
                  <span
                    class="px-3 py-1.5 rounded-lg text-xs font-black uppercase tracking-widest inline-block"
                    :class="rent.status === 'active' ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'"
                  >
                    {{ rent.status }}
                  </span>
                </td>

                <td class="px-6 py-3 text-right">
                  <div class="flex items-center justify-end gap-2">
                    <button @click="selectedRentId = rent.id; showModal = true" class="w-8 h-8 bg-emerald-100 text-emerald-600 rounded-lg hover:bg-emerald-600 hover:text-white transition-all flex items-center justify-center text-xs" title="Pay Rent">
                      <i class="fas fa-credit-card"></i>
                    </button>
                    <button @click="rentDetail(rent.id)" class="w-8 h-8 bg-primary/10 text-primary rounded-lg hover:bg-primary hover:text-white transition-all flex items-center justify-center text-xs">
                      <i class="fas fa-eye"></i>
                    </button>
                    <button @click="goToPayments(rent.id)" class="px-3 py-2 bg-slate-100 text-slate-600 text-xs font-black uppercase rounded-lg hover:bg-slate-800 hover:text-white transition-all">
                      Ledger
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="rents.length === 0">
                <td colspan="5" class="text-center py-8">
                  <div class="flex flex-col items-center gap-2">
                    <i class="fas fa-inbox text-4xl text-slate-300"></i>
                    <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No rent records found</p>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Mobile Cards -->
        <div class="lg:hidden px-6 py-4 space-y-3">
          <div v-for="rent in rents" :key="rent.id" class="bg-slate-50 border-2 border-slate-200 rounded-xl p-4 space-y-3">
            <div class="flex justify-between items-start gap-2">
              <div @click="goToPropertyDetail(rent.property_id.id)" class="flex-1">
                <h3 class="font-black text-slate-800 text-sm">{{ rent.property_id?.name || 'N/A' }}</h3>
                <p class="text-xs font-bold text-slate-500 mt-1">{{ rent.user_id?.first_name }} {{ rent.user_id?.last_name }}</p>
              </div>
              <span class="px-2 py-1 rounded-lg text-xs font-black uppercase" :class="rent.status === 'active' ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'">
                {{ rent.status }}
              </span>
            </div>
            
            <div class="grid grid-cols-3 gap-2 bg-white p-3 rounded-lg border border-slate-200">
              <div>
                <p class="text-xs font-bold text-slate-500 uppercase mb-1">Rent</p>
                <p class="text-sm font-black text-primary">${{ rent.rent_amount }}</p>
              </div>
              <div>
                <p class="text-xs font-bold text-slate-500 uppercase mb-1">Cycle</p>
                <p class="text-xs font-bold text-slate-700">{{ rent.payment_cycle }}</p>
              </div>
              <div>
                <p class="text-xs font-bold text-slate-500 uppercase mb-1">Ends</p>
                <p class="text-xs font-bold text-slate-700">{{ new Date(rent.end_date).toLocaleDateString('en-US', { month: 'short', day: 'numeric' }) }}</p>
              </div>
            </div>

            <div class="flex gap-2">
              <button @click="selectedRentId = rent.id; showModal = true" class="flex-1 bg-emerald-600 text-white py-2 rounded-lg text-xs font-black uppercase">Pay</button>
              <button @click="rentDetail(rent.id)" class="flex-1 bg-primary text-white py-2 rounded-lg text-xs font-black uppercase">View</button>
              <button @click="goToPayments(rent.id)" class="flex-1 bg-slate-700 text-white py-2 rounded-lg text-xs font-black uppercase">Ledger</button>
            </div>
          </div>

          <div v-if="rents.length === 0" class="text-center py-8">
            <i class="fas fa-inbox text-4xl text-slate-300 mb-2"></i>
            <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No rent records found</p>
          </div>
        </div>

        <!-- Pagination -->
        <div class="px-8 py-4 border-t-2 border-slate-100 flex justify-between items-center">
          <div class="flex items-center gap-2">
            <button :disabled="!previous" @click="fetchRents(previous)" class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
              <i class="fas fa-chevron-left text-xs"></i>
            </button>
            <div class="px-4 py-2 bg-gradient-to-r from-primary to-dprimary text-white rounded-lg">
              <span class="text-xs font-black uppercase">{{ currentPage }} / {{ totalPages }}</span>
            </div>
            <button :disabled="!next" @click="fetchRents(next)" class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
              <i class="fas fa-chevron-right text-xs"></i>
            </button>
          </div>
          <span class="text-xs font-bold text-slate-500 uppercase">Total: <span class="text-primary font-black">{{ rents.length }}</span> records</span>
        </div>
      </div>
    </div>

    <!-- Modals -->
    <AddPictureModal v-if="addPictureVisible" :visible="addPictureVisible" :rentId="selectedRentId" @close="addPictureVisible = false" @refresh="fetchRents" />
    <MakePaymentModal v-if="showModal" :visible="showModal" :rentId="selectedRentId" @close="showModal = false" @success="handlePaymentSuccess" />
    <AddRent v-if="visible" :propertyId="$route.params.id" :visible="visible" @close="visible = false" @refresh="fetchRents" />
    <UpdateRent v-if="updateVisible" :visible="updateVisible" :rent="rentToEdit" @close="updateVisible = false" @refresh="fetchRents" />
    <ConfirmModal v-if="confirmVisible" :visible="confirmVisible" title="Confirm Deletion" message="Are you sure you want to delete this rent?" @confirm="confirmDelete" @cancel="confirmVisible = false" />
  </div>
</template>

<script>
import AddRent from "@/views/closed/rent/add.vue";
import UpdateRent from "@/views/closed/rent/update.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Toast from "@/components/Toast.vue";
import AddPictureModal from "@/views/closed/rent/addRentPicture.vue";
import MakePaymentModal from "@/views/closed/rent/addRentPayment.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  name: "RentsView",
  components: {
    AddRent,
    UpdateRent,
    ConfirmModal,
    Toast,
    AddPictureModal,
    MakePaymentModal,
    Loading
  },
  data() {
    return {
      selectedRentId: "",
      addPictureVisible: false,
      searchTerm: "",
      visible: false,
      updateVisible: false,
      confirmVisible: false,
      rentToEdit: null,
      rentToDelete: null,
      sortKey: "property_id",
      sortAsc: true,
      rents: [],
      showModal: false,
      pageSize: 10,
      pageSizes: [5, 10, 20, 50, 100],
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
     loading:false
    };
  },
  computed: {
    filteredAndSortedRents() {
      let sorted = [...this.rents];
      sorted.sort((a, b) => {
        let valA = a[this.sortKey];
        let valB = b[this.sortKey];
        if (this.sortKey === "property_id") {
          valA = a.property_id?.name || "";
          valB = b.property_id?.name || "";
        }
        if (this.sortKey === "user_id") {
          valA = a.user_id?.first_name || "";
          valB = b.user_id?.first_name || "";
        }
        if (valA < valB) return this.sortAsc ? -1 : 1;
        if (valA > valB) return this.sortAsc ? 1 : -1;
        return 0;
      });
      return sorted;
    },
  },
  mounted() {
    this.fetchRents();
  },
  methods: {
    goToPayments(rentId) {
      this.$router.push({ name: "rents_payment_detail", params: { id: rentId } });
    },
    rentDetail(rentId) {
      this.$router.push({ name: "rent-detail", params: { id: rentId } });
    },
    buildRoleParams(params = {}) {
      const isSuperUser =
        localStorage.getItem("is_superuser") === "1" ||
        localStorage.getItem("is_superuser") === "true";
      const groups = JSON.parse(localStorage.getItem("groups") || "[]");
      const email = localStorage.getItem("email");

      if (!isSuperUser) {
        if (groups.includes("manager")) {
          params["property_id__property_zone_id__manager_id__email"] = email;
         
        } else if (groups.includes("owner")) {
          params["property_id__property_zone_id__owner_id__email"] = email;
             
        } else if (groups.includes("staff")) {
          params["property_id__property_zone_id__staff_id__email"] = email;
            
        }
      }
      return params;
    },
    async fetchRents(url = `/get_rents?search=${this.searchTerm}&page_size=${this.pageSize}`) {
      this.loading=true
      try {
        const params = this.buildRoleParams();
        const response = await this.$apiGet(url, params);
       console.log("response rents",response);

        if (response && response.data) {
          this.rents = response.data || [];
          this.next = response.next;
          this.previous = response.previous;
          this.currentPage = response.current_page;
          this.totalPages = response.total_pages;
        }
      } catch (error) {
        console.error("Failed to fetch rents:", error);
        this.rents = [];
      }
      finally {
        this.loading=false
      }
    },
    goToPropertyDetail(propertyId) {
      if (propertyId)
        this.$router.push({
          name: "PropertyDetail",
          params: { id: propertyId },
        });
    },
    goToUserDetail(id) {
      this.$router.push(`/user_detail/${id}`);
    },
    onSearch() {
      this.currentPage = 1;
      this.fetchRents();
    },
    sortBy(key) {
      if (this.sortKey === key) this.sortAsc = !this.sortAsc;
      else {
        this.sortKey = key;
        this.sortAsc = true;
      }
    },
  },
};
</script>

<style scoped>
@media (max-width: 768px) {
  .card {
    transition: box-shadow 0.2s ease-in-out;
  }
  .card:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
}
</style>
