<template>
  <div class="bg-slate-50/50 min-h-screen pb-10">
    <Toast ref="toast" />

    <div class="max-w-[1600px] mx-auto p-4 sm:p-2">
      <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-7 flex justify-between items-center">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center text-white backdrop-blur-md border border-white/20">
              <i class="fas fa-file-invoice-dollar text-xl"></i>
            </div>
            <div>
              <h1 class="text-xl font-black text-white tracking-tight uppercase leading-none">Workspace Payments</h1>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Rental Ledger & Transaction History</p>
            </div>
          </div>
          
          <button
            v-if="!rentalId"
            @click="showModal = true"
            class="group bg-white text-primary px-6 py-3 rounded-xl shadow-xl shadow-black/10 hover:shadow-white/20 transition-all duration-300 flex items-center gap-3 active:scale-95"
          >
            <i class="fas fa-plus text-xs group-hover:rotate-90 transition-transform"></i>
            <span class="text-[10px] font-black uppercase tracking-widest">Add Payment</span>
          </button>
        </div>

        <div class="p-6 border-b border-gray-50 flex flex-wrap items-center justify-between gap-6 bg-white">
          <div class="flex flex-wrap items-center gap-4">
            <div class="relative">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
              <input
                v-model="searchTerm"
                @input="fetchPayments(1)"
                type="search"
                placeholder="Search rentals..."
                class="pl-10 pr-4 py-2.5 bg-gray-50 border border-transparent rounded-xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all w-full sm:w-64"
              />
            </div>

            <div class="flex items-center gap-2 bg-gray-50 p-1.5 rounded-xl border border-gray-100">
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Status</span>
                <select v-model="status" @change="fetchPayments()" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All Status</option>
                  <option value="complete">Paid</option>
                  <option value="pending">Pending</option>
                  <option value="cancelled">Cancelled</option>
                </select>
              </div>
              <div class="w-px h-4 bg-gray-200"></div>
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Method</span>
                <select v-model="payment_method" @change="fetchPayments(1)" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All Methods</option>
                  <option value="tellebirr">Telebirr</option>
                  <option value="cash">Cash</option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <span class="text-[10px] font-black uppercase text-gray-400">Limit</span>
            <select v-model="perPage" @change="fetchPayments(1)" class="bg-gray-50 px-3 py-1.5 rounded-lg text-xs font-bold text-gray-600 border-none outline-none cursor-pointer">
              <option v-for="size in [10, 20, 50]" :key="size" :value="size">{{ size }}</option>
            </select>
          </div>
        </div>

        <div class="hidden md:block overflow-x-auto">
          <table class="w-full border-collapse">
            <thead>
              <tr class="bg-gray-50/50 text-[10px] font-black uppercase tracking-widest text-gray-400 border-b border-gray-50">
                <th class="px-6 py-5 text-left font-black">Ref/Rental</th>
                <th class="px-6 py-5 text-left font-black">Financials</th>
                <th class="px-6 py-5 text-left font-black">Billing Cycle</th>
                <th class="px-6 py-5 text-left font-black">Evidence</th>
                <th class="px-6 py-5 text-left font-black">Status</th>
                <th class="px-6 py-5 text-right font-black">Process</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-50">
              <tr v-for="payment in filteredAndSortedPayments" :key="payment.id" class="group hover:bg-slate-50/80 transition-all">
                <td class="px-6 py-5">
                  <div class="text-[10px] font-black text-gray-400 uppercase tracking-tighter mb-1">ID #{{ payment.id }}</div>
                  <button @click="goToRentalDetail(payment.rental)" class="text-xs font-black text-primary hover:text-dprimary transition-colors flex items-center gap-2 italic">
                    Rental #{{ payment.rental_id }}
                    <i class="fas fa-external-link-alt text-[8px]"></i>
                  </button>
                </td>
                <td class="px-6 py-5">
                  <div class="text-sm font-black text-gray-800">{{ payment.amount }} ETB</div>
                  <div class="text-[10px] font-mono text-gray-400 uppercase tracking-tighter">{{ payment.transaction_id || 'No Transaction ID' }}</div>
                </td>
                <td class="px-6 py-5">
                  <div class="flex items-center gap-2">
                    <div class="text-[11px] font-bold text-gray-600">{{ formatDate(payment.cycle_start) }}</div>
                    <i class="fas fa-arrow-right text-[8px] text-gray-300"></i>
                    <div class="text-[11px] font-bold text-gray-600">{{ formatDate(payment.cycle_end) }}</div>
                  </div>
                  <div class="text-[9px] font-black text-secondary uppercase tracking-widest mt-1">{{ payment.payment_method }}</div>
                </td>
                <td class="px-6 py-5">
                  <button
                    v-if="payment.slip_picture"
                    @click="viewSlip(payment.slip_picture)"
                    class="group/slip flex items-center gap-2 px-3 py-1.5 bg-gray-100 rounded-lg text-[10px] font-black uppercase text-gray-500 hover:bg-primary hover:text-white transition-all"
                  >
                    <i class="fas fa-image group-hover/slip:scale-110 transition-transform"></i>
                    View Slip
                  </button>
                  <span v-else class="text-[10px] font-black text-gray-300 uppercase italic">Missing</span>
                </td>
                <td class="px-6 py-5">
                  <span
                    class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-block shadow-sm"
                    :class="{
                      'bg-emerald-100 text-emerald-700': payment.status === 'complete',
                      'bg-amber-100 text-amber-700': payment.status === 'pending',
                      'bg-rose-100 text-rose-700': payment.status === 'cancelled'
                    }"
                  >
                    {{ payment.status }}
                  </span>
                </td>
                <td class="px-6 py-5">
                  <div class="flex justify-end gap-2">
                    <button
                      v-if="payment.status !== 'complete'"
                      @click="approve(payment)"
                      class="px-4 py-2 bg-emerald-600 text-white rounded-xl text-[10px] font-black uppercase tracking-widest shadow-md shadow-emerald-200 hover:bg-emerald-700 transition-all active:scale-95"
                    >
                      Approve
                    </button>
                    <button
                      v-else
                      @click="disApprove(payment)"
                      class="px-4 py-2 bg-white border border-rose-200 text-rose-600 rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-rose-50 transition-all"
                    >
                      Revoke
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredAndSortedPayments.length === 0">
                <td colspan="6" class="py-20 text-center">
                  <i class="fas fa-folder-open text-gray-100 text-5xl mb-4"></i>
                  <p class="text-[10px] font-black text-gray-300 uppercase tracking-[0.2em]">No financial records found</p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden p-4 space-y-4">
          <div v-for="payment in filteredAndSortedPayments" :key="payment.id" class="bg-white border border-gray-100 rounded-[2rem] p-6 shadow-sm space-y-5">
            <div class="flex justify-between items-start">
              <div>
                <div class="text-xs font-black text-gray-400 uppercase tracking-widest mb-1">Rental #{{ payment.rental_id }}</div>
                <div class="text-xl font-black text-gray-800">{{ payment.amount }} ETB</div>
              </div>
              <span class="px-3 py-1 bg-gray-50 text-gray-400 rounded-lg text-[9px] font-black uppercase tracking-widest">
                {{ payment.status }}
              </span>
            </div>

            <div class="p-4 bg-gray-50 rounded-2xl space-y-2">
              <div class="flex justify-between text-[10px] font-bold">
                <span class="text-gray-400">CYCLE</span>
                <span class="text-gray-700">{{ formatDate(payment.cycle_start) }} - {{ formatDate(payment.cycle_end) }}</span>
              </div>
              <div class="flex justify-between text-[10px] font-bold">
                <span class="text-gray-400">METHOD</span>
                <span class="text-gray-700 uppercase">{{ payment.payment_method }}</span>
              </div>
            </div>

            <div v-if="payment.slip_picture" class="rounded-2xl overflow-hidden border border-gray-100">
               <img :src="payment.slip_picture" alt="Slip" class="w-full h-40 object-cover" />
            </div>

            <div class="flex gap-2">
              <button @click="goToRentalDetail(payment.rental)" class="flex-1 py-3 bg-gray-100 rounded-xl text-[9px] font-black uppercase tracking-widest text-gray-500">Details</button>
              <button v-if="payment.status !== 'complete'" @click="approve(payment)" class="flex-1 py-3 bg-emerald-600 text-white rounded-xl text-[9px] font-black uppercase tracking-widest shadow-lg shadow-emerald-200">Approve</button>
              <button v-else @click="disApprove(payment)" class="flex-1 py-3 bg-rose-50 text-rose-600 rounded-xl text-[9px] font-black uppercase tracking-widest">Revoke</button>
            </div>
          </div>
        </div>

        <div v-if="!rentalId" class="p-8 border-t border-gray-50 flex items-center justify-between">
          <button @click="fetchPayments(previous)" :disabled="!previous" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-left text-xs"></i>
          </button>
          <span class="text-[10px] font-black text-gray-300 uppercase tracking-[0.3em]">Page {{ currentPage }} / {{ totalPages }}</span>
          <button @click="fetchPayments(next)" :disabled="!next" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-right text-xs"></i>
          </button>
        </div>
      </div>

      <AddRentalPayment
        v-if="!rentalId"
        :visible="showModal"
        @close="showModal = false"
        @success="fetchPayments"
      />
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddRentalPayment from "./add.vue";

export default {
  name: "WorkspacePaymentsView",
  components: { Toast, AddRentalPayment },
  data() {
    return {
      payments: [],
      searchTerm: "",
      perPage: 10,
      pageSizes: [5, 10, 20, 50, 100],
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      showModal: false,
      sortKey: "rental_info",
      sortAsc: true,
      payment_method: "",
      status: "",
    };
  },
  computed: {
    rentalId() {
      return this.$route.params.id || null;
    },
    filteredAndSortedPayments() {
      const term = this.searchTerm.toLowerCase();
      return this.payments
        .filter(
          (p) =>
            String(p.rental_info).toLowerCase().includes(term) ||
            String(p.status).toLowerCase().includes(term) ||
            String(p.amount).toLowerCase().includes(term)
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
    this.fetchPayments();
  },
  methods: {
    goToRentalDetail(id) {
      this.$router.push(`/co-work-rental-detail/${id}`);
    },
    viewSlip(url) {
      window.open(url, "_blank");
    },
    async fetchPayments(urlOrPage = null) {
      try {
        let params = {};
        if (this.rentalId) {
          params = { rental__id: this.rentalId, payment_method: this.payment_method, status: this.status };
        } else {
          params = { page: urlOrPage || 1, payment_method: this.payment_method, status: this.status, page_size: this.perPage };
        }

        const res = await this.$getWorkspacePayments(null, params);
        this.payments = res.payments || [];
        this.currentPage = res.currentPage || 1;
        this.totalPages = res.totalPages || 1;
        this.next = res.next || null;
        this.previous = res.previous || null;
      } catch (err) {
        console.error("Failed to fetch payments:", err);
        this.payments = [];
      }
    },
    approve(payment) {
      this.updatePaymentStatus(payment, "complete", "Payment Approved Successfully");
    },
    disApprove(payment) {
      this.updatePaymentStatus(payment, "pending", "Payment Disapproved Successfully");
    },
    async updatePaymentStatus(payment, status, message) {
      try {
        const res1 = await this.$apiPatch("/update_rental_payment", payment.id, { status });
        if (res1) {
          let payloadWorkspaceRental = { is_active: status === "pending" };
          await this.$apiPatch("/update_workspace_rental", payment.rental, payloadWorkspaceRental);
          this.$root.$refs.toast.showToast(message, "success");
          this.fetchPayments(this.currentPage);
        } else {
          this.$refs.toast.showToast("Failed to update rental payment", "error");
        }
      } catch (err) {
        console.error(err);
        this.$refs.toast.showToast("Something went wrong while updating payment", "error");
      }
    },
    formatDate(dateString) {
      if (!dateString) return "-";
      return new Date(dateString).toLocaleDateString();
    },
  },
};
</script>
