<template>
  <div class="bg-slate-50/50 min-h-screen pb-10">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Fetching ledger data..." />

    <div class="max-w-[1600px] mx-auto p-4 sm:p-2">
      <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center text-white backdrop-blur-md border border-white/20">
              <i class="fas fa-file-invoice-dollar text-xl"></i>
            </div>
            <div>
              <h1 class="text-xl font-black text-white tracking-tight uppercase leading-none">Rent Payments</h1>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Financial Transaction History</p>
            </div>
          </div>
        </div>

        <div v-if="!rentId" class="p-6 border-b border-gray-50 flex flex-wrap items-center justify-between gap-6">
          <div class="flex flex-wrap items-center gap-4">
            <div class="relative">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
              <input
                v-model="searchTerm"
                @input="fetchPayments(1)"
                type="search"
                placeholder="Search ledger..."
                class="pl-10 pr-4 py-2.5 bg-gray-50 border border-transparent rounded-xl text-sm font-bold focus:bg-white focus:border-primary focus:ring-4 focus:ring-primary/5 outline-none transition-all w-full sm:w-64"
              />
            </div>

            <div class="flex items-center gap-2 bg-gray-50 p-1.5 rounded-xl border border-gray-100">
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Status</span>
                <select
                  @change="fetchPayments(1)"
                  v-model="statusFilter"
                  class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer"
                >
                  <option value="all">All Records</option>
                  <option value="complete">Completed</option>
                  <option value="pending">Pending</option>
                  <option value="cancelled">Cancelled</option>
                </select>
              </div>
              <div class="w-px h-4 bg-gray-200"></div>
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Method</span>
                <select
                  @change="fetchPayments(1)"
                  v-model="payment_method"
                  class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer"
                >
                  <option value="">All</option>
                  <option value="tellebirr">Tellebirr</option>
                  <option value="cash">Cash</option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <span class="text-[10px] font-black uppercase text-gray-400">Show</span>
            <div class="flex gap-1">
              <button 
                v-for="n in [10, 20, 50]" 
                :key="n"
                @click="perPage = n; fetchPayments(1)"
                class="px-3 py-1 text-[10px] font-black rounded-lg transition-all"
                :class="perPage === n ? 'bg-primary text-white shadow-lg shadow-primary/20' : 'bg-gray-50 text-gray-400 hover:bg-gray-100'"
              >
                {{ n }}
              </button>
            </div>
          </div>
        </div>

        <div class="hidden lg:block overflow-x-auto">
          <table class="w-full border-collapse">
            <thead>
              <tr class="bg-gray-50/50 text-[10px] font-black uppercase tracking-[0.15em] text-gray-400 border-b border-gray-50">
                <th class="px-6 py-4 text-left">Ref</th>
                <th class="px-6 py-4 text-left">Financial Detail</th>
                <th class="px-6 py-4 text-left">Logistics</th>
                <th class="px-6 py-4 text-left">Status</th>
                <th class="px-6 py-4 text-left">Timeline</th>
                <th class="px-6 py-4 text-left">Relations</th>
                <th class="px-6 py-4 text-right">Actions</th>
              </tr>
            </thead>

            <tbody class="divide-y divide-gray-50">
              <tr
                v-for="(payment, index) in filteredPayments"
                :key="payment.id"
                class="group hover:bg-slate-50/80 transition-all duration-200"
              >
                <td class="px-6 py-4 text-[11px] font-bold text-gray-300">#{{ index + 1 }}</td>
                <td class="px-6 py-4">
                  <div class="text-sm font-black text-gray-800">{{ payment.amount }} ETB</div>
                  <div class="text-[10px] font-bold text-secondary uppercase tracking-tighter">{{ payment.payment_method }}</div>
                </td>
                <td class="px-6 py-4">
                  <div class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">Due Date</div>
                  <div class="text-xs font-bold text-gray-700">{{ formatDate(payment.due_date) }}</div>
                  <div class="text-[9px] font-mono text-gray-400 mt-1">{{ payment.transaction_id }}</div>
                </td>
                <td class="px-6 py-4">
                  <span
                    class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-block"
                    :class="{
                      'bg-emerald-100 text-emerald-700': payment.status === 'paid' || payment.status === 'complete',
                      'bg-amber-100 text-amber-700': payment.status === 'pending',
                      'bg-rose-100 text-rose-700': payment.status === 'cancelled'
                    }"
                  >
                    {{ payment.status }}
                  </span>
                </td>
                <td class="px-6 py-4">
                  <div class="text-[9px] font-black text-gray-300 uppercase tracking-widest">Modified</div>
                  <div class="text-[11px] font-bold text-gray-500">
                    {{ payment.updated_at ? formatDate(payment.updated_at) : formatDate(payment.created_at) }}
                  </div>
                </td>
                <td class="px-6 py-4 space-x-2">
                  <button @click="goToRentDetail(payment.rent_id)" class="px-3 py-1.5 bg-gray-100 text-gray-600 rounded-lg text-[10px] font-black uppercase tracking-widest hover:bg-primary hover:text-white transition-all">Rent</button>
                  <button @click="goToUserDetail(payment.user_id)" class="px-3 py-1.5 bg-gray-100 text-gray-600 rounded-lg text-[10px] font-black uppercase tracking-widest hover:bg-secondary hover:text-white transition-all">User</button>
                </td>
                <td class="px-6 py-4 text-right">
                  <div class="flex justify-end gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                    <button
                      v-if="payment.status == 'pending' || payment.status == 'cancelled'"
                      @click="approve(payment)"
                      class="w-8 h-8 rounded-lg bg-emerald-50 text-emerald-600 hover:bg-emerald-600 hover:text-white transition-all"
                      title="Approve"
                    >
                      <i class="fas fa-check text-xs"></i>
                    </button>
                    <button
                      v-if="payment.status == 'pending' || payment.status == 'complete'"
                      @click="reject(payment)"
                      class="w-8 h-8 rounded-lg bg-rose-50 text-rose-600 hover:bg-rose-600 hover:text-white transition-all"
                      title="Reject"
                    >
                      <i class="fas fa-times text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
          <div v-if="filteredPayments.length === 0" class="py-20 text-center">
            <i class="fas fa-folder-open text-gray-100 text-5xl mb-4"></i>
            <p class="text-xs font-black text-gray-300 uppercase tracking-[0.2em]">No financial records discovered</p>
          </div>
        </div>

        <div class="lg:hidden p-4 space-y-4">
          <div
            v-for="payment in filteredPayments"
            :key="payment.id"
            class="bg-white border border-gray-100 rounded-3xl p-5 shadow-sm space-y-4"
          >
            <div class="flex justify-between items-start">
              <div>
                <div class="text-lg font-black text-gray-800">{{ payment.amount }} ETB</div>
                <div class="text-[10px] font-black text-secondary uppercase tracking-widest">{{ payment.payment_method }}</div>
              </div>
              <span
                class="px-3 py-1 rounded-lg text-[9px] font-black uppercase tracking-widest"
                :class="{
                  'bg-emerald-50 text-emerald-600': payment.status === 'paid' || payment.status === 'complete',
                  'bg-amber-50 text-amber-600': payment.status === 'pending',
                  'bg-rose-50 text-rose-600': payment.status === 'cancelled'
                }"
              >
                {{ payment.status }}
              </span>
            </div>

            <div class="grid grid-cols-2 gap-4 text-[11px] font-bold text-gray-500">
              <div class="bg-gray-50 p-3 rounded-2xl">
                <p class="text-[8px] font-black text-gray-300 uppercase mb-1">Due Date</p>
                {{ formatDate(payment.due_date) }}
              </div>
              <div class="bg-gray-50 p-3 rounded-2xl">
                <p class="text-[8px] font-black text-gray-300 uppercase mb-1">Transaction</p>
                <span class="font-mono">{{ payment.transaction_id || 'N/A' }}</span>
              </div>
            </div>

            <div class="flex gap-2 pt-2">
              <button @click="approve(payment)" v-if="payment.status !== 'complete' && payment.status !== 'paid'" class="flex-1 py-3 bg-emerald-600 text-white rounded-xl text-[10px] font-black uppercase tracking-widest">Approve</button>
              <button @click="reject(payment)" v-if="payment.status !== 'cancelled'" class="flex-1 py-3 bg-rose-50 text-rose-600 rounded-xl text-[10px] font-black uppercase tracking-widest">Cancel</button>
            </div>
          </div>
        </div>

        <div v-if="!rentId" class="p-6 border-t border-gray-50 flex items-center justify-between">
          <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">
            Page {{ currentPage }} <span class="mx-1 text-gray-200">/</span> {{ totalPages }}
          </p>
          <div class="flex gap-2">
            <button
              @click="fetchUrl(previous)"
              :disabled="!previous"
              class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20"
            >
              <i class="fas fa-chevron-left text-xs"></i>
            </button>
            <button
              @click="fetchUrl(next)"
              :disabled="!next"
              class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20"
            >
              <i class="fas fa-chevron-right text-xs"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import Loading from "@/components/Loading.vue"; // <-- Added Loading
export default {
  name: "RentPayments",
  components:{Loading},
  data() {
    return {
      payments: [],
      searchTerm: "",
      perPage: 10,
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      statusFilter: "all",
      payment_method: "",
      loading:false
    };
  },
  computed: {
    rentId() {
      return this.$route.params.id || null;
    },
    filteredPayments() {
      const term = this.searchTerm.toLowerCase();
      return this.payments.filter((p) =>
        Object.values(p).some((val) =>
          String(val || "").toLowerCase().includes(term)
        )
      );
    },
  },
  mounted() {
    this.fetchPayments(1);
  },
  watch: {
    "$route.params.id"() {
      this.fetchPayments(1);
    },
  },
  methods: {
    goToUserDetail(id) {
      this.$router.push(`/user_detail/${id}`);
    },
    goToRentDetail(id) {
      this.$router.push(`/rent-detail/${id}`);
    },
    buildRoleParams(params = {}) {
      const isSuperUser =
        localStorage.getItem("is_superuser") === "1" ||
        localStorage.getItem("is_superuser") === "true";
      const groups = JSON.parse(localStorage.getItem("groups") || "[]");
      const email = localStorage.getItem("email");
      const id = localStorage.getItem("userId");

      if (!isSuperUser) {
        if (groups.includes("manager")) {
          params = {
            ...params,
            "rent_id__property_id__property_zone_id__manager_id__email": email,
            payment_method: this.payment_method,
          };
        } else if (groups.includes("owner")) {
          params = {
            ...params,
            "rent_id__property_id__property_zone_id__owner_id__email": email,
            payment_method: this.payment_method,
          };
        } else if (groups.includes("staff")) {
          params = {
            ...params,
            "rent_id__property_id__property_zone_id__staff_id__id": id,
            payment_method: this.payment_method,
          };
        } else if (groups.includes("super_staff")) {
          params = { ...params };
        }
      } else {
        params = { payment_method: this.payment_method };
      }
      return params;
    },
    async fetchPayments(page = 1) {
      this.loading=true
      try {
        let params = this.buildRoleParams();
        if (this.rentId) params = { ...params, rent_id: this.rentId };
        else {
          params = { ...params, page, page_size: this.perPage };
          if (this.statusFilter !== "all") params.status = this.statusFilter;
        }

        const res = await this.$apiGet("/get_payments", params);
        this.payments = Array.isArray(res.data) ? res.data : [];
        this.currentPage = res.current_page || 1;
        this.totalPages = res.total_pages || 1;
        this.next = res.next || null;
        this.previous = res.previous || null;
      } catch (error) {
        console.error(error);
        this.payments = [];
      }

      finally {
        this.loading=false
      }
    },
    async fetchUrl(url) {
      if (!url) return;
      try {
        const res = await this.$apiGet(url);
        this.payments = Array.isArray(res.data) ? res.data : [];
        this.currentPage = res.current_page || 1;
        this.totalPages = res.total_pages || 1;
        this.next = res.next || null;
        this.previous = res.previous || null;
      } catch (error) {
        console.error(error);
      }
    },
    async approve(payment) {
      const payload = { id: payment.id, status: "paid" };
      const res = await this.$apiPatch(`/update_payment`, payment.id, payload);
      if (res) {
        this.$root.$refs.toast.showToast("Payment Approved Successfully", "success");
        this.fetchPayments(this.currentPage);
      }
    },
    async reject(payment) {
      const payload = { id: payment.id, status: "pending" };
      const res = await this.$apiPatch(`/update_payment`, payment.id, payload);
      if (res) {
        this.$root.$refs.toast.showToast("Payment Disapproved Successfully", "success");
        this.fetchPayments(this.currentPage);
      }
    },
    formatDate(dateString) {
      if (!dateString) return "-";
      return new Date(dateString).toLocaleDateString();
    },
  },
};
</script>
