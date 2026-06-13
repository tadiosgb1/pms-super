<template>
  <div class="bg-slate-50/50 min-h-screen pb-10">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Verifying subscription ledger..." />

    <div
      v-if="showImageModal"
      class="fixed inset-0 bg-slate-900/90 backdrop-blur-md flex justify-center items-center z-[1000] p-4 animate-in fade-in duration-300"
      @click.self="showImageModal = false"
    >
      <div class="relative max-w-4xl w-full group">
        <button 
          @click="showImageModal = false" 
          class="absolute -top-12 right-0 text-white hover:text-rose-400 transition-colors flex items-center gap-2 font-black text-[10px] uppercase tracking-widest"
        >
          Close Preview <i class="fas fa-times text-lg"></i>
        </button>
        <img :src="selectedSlipImage" class="w-full h-auto rounded-[2rem] shadow-2xl border border-white/10 object-contain max-h-[80vh]" />
      </div>
    </div>

    <div class="max-w-[1600px] mx-auto p-4 sm:p-2">
      <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-7 flex justify-between items-center">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center text-white backdrop-blur-md border border-white/20">
              <i class="fas fa-crown text-xl text-amber-300"></i>
            </div>
            <div>
              <h1 class="text-xl font-black text-white tracking-tight uppercase leading-none">Subscription Ledger</h1>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Provider Revenue Management</p>
            </div>
          </div>
        </div>

        <div class="p-6 border-b border-gray-50 flex flex-wrap items-center justify-between gap-6">
          <div class="flex flex-wrap items-center gap-4">
            <div class="relative">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
              <input
                v-model="searchTerm"
                @input="fetchPayments(1)"
                type="search"
                placeholder="Find transaction..."
                class="pl-10 pr-4 py-2.5 bg-gray-50 border border-transparent rounded-xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all w-full sm:w-64"
              />
            </div>

            <div class="flex items-center gap-2 bg-gray-50 p-1.5 rounded-xl border border-gray-100">
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Status</span>
                <select v-model="statusFilter" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All</option>
                  <option value="paid">Paid</option>
                  <option value="pending">Pending</option>
                  <option value="canceled">Cancelled</option>
                </select>
              </div>
              <div class="w-px h-4 bg-gray-200"></div>
              <div class="flex items-center px-3 gap-2">
                <span class="text-[9px] font-black uppercase text-gray-400">Method</span>
                <select v-model="payment_method" @change="fetchPayments(1)" class="bg-transparent text-xs font-bold text-gray-700 outline-none cursor-pointer">
                  <option value="">All</option>
                  <option value="telebirr">Telebirr</option>
                  <option value="cash">Cash</option>
                </select>
              </div>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <span class="text-[10px] font-black uppercase text-gray-400">Show</span>
            <select v-model.number="perPage" @change="fetchPayments(1)" class="bg-gray-50 px-3 py-1.5 rounded-lg text-xs font-bold text-gray-600 outline-none border-none cursor-pointer">
              <option v-for="n in [10,20,50]" :key="n" :value="n">{{ n }}</option>
            </select>
          </div>
        </div>

        <div class="hidden lg:block overflow-x-auto">
          <table class="w-full border-collapse">
            <thead>
              <tr class="bg-gray-50/50 text-[10px] font-black uppercase tracking-widest text-gray-400 border-b border-gray-50">
                <th class="px-6 py-4 text-left">Transaction Ref</th>
                <th class="px-6 py-4 text-left">Owner & Plan</th>
                <th class="px-6 py-4 text-left">Financials</th>
                <th class="px-6 py-4 text-left">Timeline</th>
                <th class="px-6 py-4 text-center">Receipt</th>
                <th v-if="$hasPermission('pms.change_subscriptionpayment')" class="px-6 py-4 text-right">Review</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-50">
              <tr v-for="p in filteredAndSortedPayments" :key="p.id" class="group hover:bg-slate-50/80 transition-all">
                <td class="px-6 py-4">
                  <div class="text-xs font-mono font-bold text-gray-800">{{ p.transaction_id }}</div>
                  <div class="mt-1 flex gap-2">
                    <button @click="goToSubDetail(p.subscription_id)" class="text-[9px] font-black uppercase text-primary hover:underline">Sub ID: {{ p.subscription_id }}</button>
                  </div>
                </td>
                <td class="px-6 py-4">
                  <div class="text-sm font-black text-gray-700 uppercase tracking-tight">{{ p.planName }}</div>
                  <button @click="goToUserDetail(p.user_id)" class="text-[10px] font-bold text-gray-400 hover:text-secondary transition-colors italic">View Property Owner</button>
                </td>
                <td class="px-6 py-4">
                  <div class="text-sm font-black text-gray-800">{{ p.amount }} ETB</div>
                  <div class="text-[9px] font-black uppercase text-secondary tracking-widest">{{ p.payment_method }}</div>
                </td>
                <td class="px-6 py-4">
                  <div class="text-[10px] font-black text-gray-300 uppercase mb-1 tracking-tighter">Valid Until</div>
                  <div class="text-xs font-bold text-gray-600">{{ p.end_date || "-" }}</div>
                </td>
                <td class="px-6 py-4 text-center">
                  <button 
                    v-if="p.slip_picture" 
                    @click="openSlipModal(p.slip_picture)"
                    class="w-10 h-10 rounded-xl bg-gray-100 text-gray-400 hover:bg-primary/10 hover:text-primary transition-all flex items-center justify-center mx-auto"
                  >
                    <i class="fas fa-image text-sm"></i>
                  </button>
                  <span v-else class="text-[9px] font-black text-gray-300 uppercase tracking-widest">No Slip</span>
                </td>
                <td v-if="$hasPermission('pms.change_subscriptionpayment')" class="px-6 py-4 text-right">
                  <div class="flex justify-end gap-2 opacity-0 group-hover:opacity-100 transition-opacity">
                    <button v-if="p.status !== 'paid'" @click="askConfirmation('approve', p)" class="px-4 py-2 bg-emerald-50 text-emerald-600 rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-emerald-600 hover:text-white transition-all">Verify</button>
                    <button v-if="p.status !== 'canceled'" @click="askConfirmation('reject', p)" class="px-4 py-2 bg-rose-50 text-rose-600 rounded-xl text-[10px] font-black uppercase tracking-widest hover:bg-rose-600 hover:text-white transition-all">Void</button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="lg:hidden p-4 space-y-4">
          <div v-for="p in filteredAndSortedPayments" :key="p.id" class="bg-white border border-gray-100 rounded-[2rem] p-5 shadow-sm space-y-4">
            <div class="flex justify-between items-start">
              <div>
                <p class="text-[10px] font-black text-gray-300 uppercase tracking-widest mb-1">{{ p.transaction_id }}</p>
                <p class="text-lg font-black text-gray-800 leading-tight">{{ p.planName }}</p>
              </div>
              <button v-if="p.slip_picture" @click="openSlipModal(p.slip_picture)" class="w-12 h-12 bg-gray-50 rounded-2xl flex items-center justify-center text-primary">
                <i class="fas fa-receipt"></i>
              </button>
            </div>
            
            <div class="flex justify-between items-end border-t border-gray-50 pt-4">
              <div>
                <span class="text-sm font-black text-primary">{{ p.amount }} ETB</span>
                <span class="mx-2 text-gray-200">|</span>
                <span class="text-[10px] font-black uppercase text-gray-400 tracking-widest">{{ p.payment_method }}</span>
              </div>
              <div v-if="$hasPermission('pms.change_subscriptionpayment')" class="flex gap-2">
                <button v-if="p.status !== 'paid'" @click="askConfirmation('approve', p)" class="p-2 bg-emerald-50 text-emerald-600 rounded-lg"><i class="fas fa-check"></i></button>
                <button v-if="p.status !== 'canceled'" @click="askConfirmation('reject', p)" class="p-2 bg-rose-50 text-rose-600 rounded-lg"><i class="fas fa-times"></i></button>
              </div>
            </div>
          </div>
        </div>

        <div class="p-8 border-t border-gray-50 flex items-center justify-between">
          <button @click="fetchUrl(previous)" :disabled="!previous" class="px-6 py-2 bg-gray-50 rounded-xl text-[10px] font-black uppercase tracking-widest text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-30">
            Previous
          </button>
          <span class="text-[10px] font-black text-gray-300 uppercase tracking-[0.3em]">Page {{ currentPage }} / {{ totalPages }}</span>
          <button @click="fetchUrl(next)" :disabled="!next" class="px-6 py-2 bg-gray-50 rounded-xl text-[10px] font-black uppercase tracking-widest text-gray-400 hover:bg-primary hover:text-white transition-all disabled:opacity-30">
            Next
          </button>
        </div>
      </div>
    </div>

    <ConfirmModal
      :visible="showConfirm"
      :title="confirmTitle"
      :message="confirmMessage"
      @cancel="showConfirm = false"
      @confirm="confirmAction"
    />
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Loading from "@/components/Loading.vue";

export default {
  components: { Toast, ConfirmModal, Loading },

  data() {
    return {
      payments: [],
      searchTerm: "",
      statusFilter: "",
      payment_method: "",
      perPage: 10,
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      loading: false,
      showConfirm: false,
      selectedPayment: null,
      selectedAction: "",
      showImageModal: false,
      selectedSlipImage: "",
    };
  },

  computed: {
    filteredAndSortedPayments() {
      let filtered = this.payments.filter(p =>
        JSON.stringify(p).toLowerCase().includes(this.searchTerm.toLowerCase())
      );

      if (this.statusFilter)
        filtered = filtered.filter(p => p.status === this.statusFilter);

      return filtered;
    },

    confirmTitle() {
      return this.selectedAction === "approve" ? "Approve Payment" : "Reject Payment";
    },

    confirmMessage() {
      return this.selectedAction === "approve"
        ? "Do you want to approve this payment?"
        : "Are you sure you want to reject this payment?";
    },
  },

  mounted() {
    this.fetchPayments(1);
  },

  methods: {
    openSlipModal(url) {
      this.selectedSlipImage = url;
      this.showImageModal = true;
    },

    goToUserDetail(id) {
      this.$router.push(`/user_detail/${id}`);
    },

    goToSubDetail(id) {
      this.$router.push(`/sub-detail/${id}`);
    },

    async fetchPayments(page = 1) {
      this.loading = true;

      try {
        let params = {
          page,
          page_size: this.perPage,
          payment_method: this.payment_method,
          subscription_id: this.$route.params.id,
        };

        if (localStorage.getItem("is_superuser") !== "true") {
          params.user_id = localStorage.getItem("userId");
        }

        const response = await this.$apiGet("/get_subscription_payment", params);

        this.payments = response.data || [];
        this.currentPage = response.current_page;
        this.totalPages = response.total_pages;
        this.next = response.next;
        this.previous = response.previous;

        // Fetch user & plan info
        await Promise.all(
          this.payments.map(async (p) => {
            if (p.user_id) {
              const u = await this.$apiGetById("get_user", p.user_id);
              p.ownerName = u.first_name || "Unknown";
            }
            if (p.subscription_id) {
              const s = await this.$apiGetById("get_subscription", p.subscription_id);
              p.planName = s.plan_name || "Unknown Plan";
            }
          })
        );
      } catch (err) {
        console.error(err);
      }

      this.loading = false;
    },

    async fetchUrl(url) {
      if (!url) return;

      try {
        const res = await this.$apiGet(url);
        this.payments = res.data || [];
        this.currentPage = res.current_page;
        this.totalPages = res.total_pages;
        this.next = res.next;
        this.previous = res.previous;
      } catch (err) {
        console.error(err);
      }
    },

    askConfirmation(action, payment) {
      this.selectedPayment = payment;
      this.selectedAction = action;
      this.showConfirm = true;
    },

    async confirmAction() {
      const p = this.selectedPayment;
      if (!p) return;

      const newStatus = this.selectedAction === "approve" ? "paid" : "canceled";

      const res = await this.$apiPatch(`/update_subscription_payment`, p.id, {
        id: p.id,
        status: newStatus,
      });

      if (res) {
        const newSubStatus = newStatus === "paid" ? "active" : "pending";

        await this.$apiPatch(`/update_subscription`, res.subscription_id, {
          id: res.subscription_id,
          status: newSubStatus,
        });

        this.$root.$refs.toast.showToast(
          newStatus === "paid" ? "Payment approved!" : "Payment rejected!",
          "success"
        );
      }

      this.showConfirm = false;
      this.fetchPayments(this.currentPage);
    },
  },
};
</script>
