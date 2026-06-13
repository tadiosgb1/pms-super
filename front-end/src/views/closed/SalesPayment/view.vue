<template>
  <div class="bg-slate-50/50 min-h-screen pb-10">
    <Toast ref="toast" />

    <div class="max-w-[1600px] mx-auto p-4 sm:p-2">
      <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-7 flex justify-between items-center">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-xl flex items-center justify-center text-white backdrop-blur-md border border-white/20">
              <i class="fas fa-hand-holding-usd text-xl"></i>
            </div>
            <div>
              <h1 class="text-xl font-black text-white tracking-tight uppercase leading-none">Sale Payments</h1>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Record for Sale #{{ saleId }}</p>
            </div>
          </div>
          
          <router-link
            to="/propertiesListForSale"
            class="group flex items-center gap-2 bg-white/10 hover:bg-white text-white hover:text-primary px-5 py-2.5 rounded-xl transition-all duration-300 border border-white/20 text-[10px] font-black uppercase tracking-widest"
          >
            <i class="fas fa-arrow-left transition-transform group-hover:-translate-x-1"></i>
            Back to Sales
          </router-link>
        </div>

        <div class="p-6 border-b border-gray-50 flex flex-wrap items-center justify-between gap-6">
          <div class="flex flex-wrap items-center gap-4">
            <div class="relative">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
              <input
                v-model="searchTerm"
                @input="fetchPayments(1)"
                type="search"
                placeholder="Search ledger..."
                class="pl-10 pr-4 py-2.5 bg-gray-50 border border-transparent rounded-xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all w-full sm:w-64"
              />
            </div>

            <div class="flex items-center gap-2 bg-gray-50 p-1.5 rounded-xl border border-gray-100">
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Status</span>
                <select v-model="statusFilter" @change="fetchPayments(1)" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All Status</option>
                  <option value="complete">Completed</option>
                  <option value="pending">Pending</option>
                  <option value="cancelled">Cancelled</option>
                </select>
              </div>
              <div class="w-px h-4 bg-gray-200"></div>
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Method</span>
                <select v-model="payment_method" @change="fetchPayments(1)" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All Methods</option>
                  <option value="Telle">Telebirr</option>
                  <option value="cbe">CBE</option>
                  <option value="Manual">Manual</option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <span class="text-[10px] font-black uppercase text-gray-400">View</span>
            <select v-model.number="perPage" @change="fetchPayments(1)" class="bg-gray-50 px-3 py-1.5 rounded-lg text-xs font-bold text-gray-600 border-none outline-none cursor-pointer">
              <option v-for="n in [10, 20, 50]" :key="n" :value="n">{{ n }}</option>
            </select>
          </div>
        </div>

        <div class="hidden md:block overflow-x-auto">
          <table class="w-full border-collapse">
            <thead>
              <tr class="bg-gray-50/50 text-[10px] font-black uppercase tracking-widest text-gray-400 border-b border-gray-50">
                <th class="px-6 py-4 text-left">Financial Detail</th>
                <th class="px-6 py-4 text-left">Transaction Ref</th>
                <th class="px-6 py-4 text-left">Status</th>
                <th class="px-6 py-4 text-left">Timeline</th>
                <th class="px-6 py-4 text-left">Parties</th>
                <th class="px-6 py-4 text-right">Review Actions</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-50">
              <tr v-for="p in filteredPayments" :key="p.id" class="hover:bg-slate-50/80 transition-all">
                <td class="px-6 py-4">
                  <div class="text-sm font-black text-gray-800">{{ p.amount | currency }}</div>
                  <div class="text-[9px] font-black text-secondary uppercase tracking-widest">{{ p.payment_method }}</div>
                </td>
                <td class="px-6 py-4">
                  <div class="text-[10px] font-mono font-bold text-gray-500 uppercase tracking-tighter bg-gray-100 px-2 py-1 rounded-md inline-block">
                    {{ p.transaction_id || 'NO-REF' }}
                  </div>
                </td>
                <td class="px-6 py-4">
                  <span
                    class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-block"
                    :class="{
                      'bg-emerald-100 text-emerald-700': p.status === 'complete',
                      'bg-amber-100 text-amber-700': p.status === 'pending',
                      'bg-rose-100 text-rose-700': p.status === 'cancelled'
                    }"
                  >
                    {{ p.status }}
                  </span>
                </td>
                <td class="px-6 py-4">
                  <div class="text-[9px] font-black text-gray-300 uppercase tracking-tighter mb-1">Created</div>
                  <div class="text-[11px] font-bold text-gray-500">{{ p.created_at }}</div>
                </td>
                <td class="px-6 py-4 space-x-2">
                  <button @click="goToSaleDetail(p.property_zone_sale_id)" class="px-3 py-1.5 bg-gray-100 text-gray-600 rounded-lg text-[10px] font-black uppercase tracking-widest hover:bg-primary hover:text-white transition-all">Sale</button>
                  <button @click="goToUserDetail(p.buyer_id)" class="px-3 py-1.5 bg-gray-100 text-gray-600 rounded-lg text-[10px] font-black uppercase tracking-widest hover:bg-secondary hover:text-white transition-all">Buyer</button>
                </td>
                <td class="px-6 py-4">
                  <div class="flex justify-end gap-2">
                    <button
                      v-if="p.status=='pending' || p.status=='cancelled'"
                      @click="approve(p)"
                      class="px-4 py-2 bg-emerald-600 text-white rounded-xl text-[10px] font-black uppercase tracking-widest shadow-md shadow-emerald-200 hover:bg-emerald-700 transition-all active:scale-95"
                    >
                      Approve
                    </button>
                    <button
                      v-if="p.status=='pending' || p.status=='complete'"
                      @click="reject(p)"
                      class="px-4 py-2 bg-white border border-rose-200 text-rose-600 rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-rose-50 transition-all active:scale-95"
                    >
                      Cancel
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredPayments.length === 0">
                <td colspan="6" class="py-20 text-center">
                  <i class="fas fa-receipt text-gray-100 text-5xl mb-4"></i>
                  <p class="text-[10px] font-black text-gray-300 uppercase tracking-[0.2em]">No payment history found</p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden p-4 space-y-4">
          <div v-for="p in filteredPayments" :key="p.id" class="bg-white border border-gray-100 rounded-[2rem] p-6 shadow-sm space-y-4">
            <div class="flex justify-between items-start">
              <div>
                <div class="text-xl font-black text-gray-800">{{ p.amount | currency }}</div>
                <div class="text-[10px] font-black text-secondary uppercase tracking-widest">{{ p.payment_method }}</div>
              </div>
              <span class="px-3 py-1 bg-gray-50 text-gray-400 rounded-lg text-[9px] font-black uppercase tracking-widest">
                {{ p.status }}
              </span>
            </div>
            
            <div class="grid grid-cols-2 gap-2 py-4 border-y border-gray-50">
              <button @click="goToSaleDetail(p.property_zone_sale_id)" class="py-2 bg-gray-50 rounded-xl text-[9px] font-black uppercase tracking-widest text-gray-500 italic">View Sale</button>
              <button @click="goToUserDetail(p.buyer_id)" class="py-2 bg-gray-50 rounded-xl text-[9px] font-black uppercase tracking-widest text-gray-500 italic">View Buyer</button>
            </div>

            <div class="flex gap-2 pt-2">
              <button v-if="p.status !== 'complete'" @click="approve(p)" class="flex-1 py-3 bg-emerald-600 text-white rounded-xl text-[10px] font-black uppercase tracking-widest">Approve</button>
              <button v-if="p.status !== 'cancelled'" @click="reject(p)" class="flex-1 py-3 bg-rose-50 text-rose-600 rounded-xl text-[10px] font-black uppercase tracking-widest">Cancel</button>
            </div>
          </div>
        </div>

        <div class="p-8 border-t border-gray-50 flex items-center justify-between">
          <button @click="fetchUrl(previous)" :disabled="!previous" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-left text-xs"></i>
          </button>
          <span class="text-[10px] font-black text-gray-300 uppercase tracking-[0.3em]">Page {{ currentPage }} / {{ totalPages }}</span>
          <button @click="fetchUrl(next)" :disabled="!next" class="w-10 h-10 flex items-center justify-center rounded-xl bg-gray-50 text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-20">
            <i class="fas fa-chevron-right text-xs"></i>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";

export default {
  name: "salePaymentsView",
  components: { Toast },
  data() {
    return {
      payments: [],
      saleId: this.$route.params.saleId,
      searchTerm: "",
      perPage: 10,
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      statusFilter: "",
      payment_method: "",
    };
  },
  computed: {
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
  methods: {
    goToUserDetail(id) {
      this.$router.push(`/user_detail/${id}`);
    },
    goToSaleDetail(id) {
      this.$router.push(`/property-sale/${id}`);
    },

    // ✅ unified fetch function that respects all filters
    async fetchPayments(page = 1) {
      try {
        const params = {
          page,
          page_size: this.perPage,
          search: this.searchTerm,
        };

        // ✅ add optional filters dynamically
        if (this.statusFilter && this.statusFilter !== "all") {
          params.status = this.statusFilter;
        }

        if (this.payment_method) {
          params.payment_method = this.payment_method;
        }

        // ✅ include sale ID filter if available
        const id = this.$route.query.id;
        if (id) {
          params.property_zone_sale_id__id = id;
        }

        console.log("Params for /get_sales_payments:", params);

        const res = await this.$apiGet("/get_sales_payments", params);
        this.payments = res.data || [];
        this.currentPage = res.current_page || 1;
        this.totalPages = res.total_pages || 1;
        this.next = res.next;
        this.previous = res.previous;
      } catch (err) {
        console.error("Error fetching payments:", err);
        this.payments = [];
      }
    },

    async fetchUrl(url) {
      if (!url) return;
      const res = await this.$apiGet(url);
      this.payments = res.data || [];
      this.currentPage = res.current_page;
      this.totalPages = res.total_pages;
      this.next = res.next;
      this.previous = res.previous;
    },

    // ✅ simplified — reuses fetchPayments() with filters
    async filterByStatus() {
      await this.fetchPayments(1);
    },

    async approve(payment) {
      const payload = { id: payment.id, status: "complete" };
      const res = await this.$apiPatch("/update_sales_payments", payment.id, payload);
      if (res) {
       // this.$root.$refs.toast.showToast("Payment Approved Successfully", "success");
        const salePayload={
         id:payment.property_zone_sale_id,
         status:"active"
        }

        console.log("payment",payment);
          const res = await this.$apiPatch("/update_property_zone_sale", payment.property_zone_sale_id, salePayload);
          console.log("res update the sale property",res);
        //this.fetchPayments(this.currentPage);
      }
    },

    async reject(payment) {
      const payload = { id: payment.id, status: "cancelled" };
      const res = await this.$apiPatch("/update_sales_payments", payment.id, payload);
      if (res) {
        this.$root.$refs.toast.showToast("Payment Disapproved Successfully", "success");
        this.fetchPayments(this.currentPage);
      }
    },
  },
  filters: {
    currency(v) {
      return v ? new Intl.NumberFormat().format(v) + " USD" : "";
    },
  },
};
</script>
