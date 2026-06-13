<template>
  <div class="min-h-screen bg-background p-4 md:p-2">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading subscriptions..." />

    <div class="max-w-7xl mx-auto bg-white shadow-xl rounded-xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md text-white">
            <i class="fas fa-file-invoice-dollar text-2xl"></i>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Subscriptions</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Billing & Plan Management</p>
          </div>
        </div>

        <button
          v-if="addSubsc"
          @click="visible = true"
          class="bg-white text-primary hover:bg-gray-50 font-bold px-6 py-2.5 rounded-xl shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2"
        >
          <i class="fas fa-plus-circle"></i> New Subscription
        </button>
      </div>

      <div class="p-6 bg-gray-50/50 border-b border-gray-100">
        <div class="flex flex-col lg:flex-row justify-between gap-4">
          <div class="relative w-full lg:w-1/3">
            <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400">
              <i class="fas fa-search text-xs"></i>
            </span>
            <input
              v-model="searchTerm"
              type="search"
              placeholder="Search by plan or owner..."
              class="custom-input !mt-0 !pl-10 h-11 shadow-sm"
            />
          </div>

          <div class="flex flex-wrap items-center gap-4">
            <div v-if="is_super_user == 'true'" class="flex items-center gap-2 bg-white px-3 py-1.5 rounded-xl border border-gray-100 shadow-sm">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-wider">Status</label>
              <select v-model="status" @change="fetchSubscriptions()" class="border-none text-sm font-bold text-primary focus:ring-0 cursor-pointer">
                <option value="">All Plans</option>
                <option value="pending">Pending</option>
                <option value="active">Active</option>
                <option value="expired">Expired</option>
              </select>
            </div>

            <div class="flex items-center gap-2 bg-white px-3 py-1.5 rounded-xl border border-gray-100 shadow-sm">
              <label class="text-[10px] font-black text-gray-400 uppercase tracking-wider">Show</label>
              <select v-model="pageSize" @change="fetchSubscriptions()" class="border-none text-sm font-bold text-primary focus:ring-0 cursor-pointer">
                <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <div class="hidden lg:block p-6">
        <div class="overflow-hidden rounded-xl border border-gray-100 shadow-sm">
          <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr class="text-left text-[11px] font-black text-gray-400 uppercase tracking-widest">
                <th class="px-6 py-4">Plan & Price</th>
                <th class="px-6 py-4">Billing Cycle</th>
                <th class="px-6 py-4">Account Owner</th>
                <th class="px-6 py-4 text-center">Status</th>
                <th class="px-6 py-4 text-right">Actions</th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-50 text-sm">
              <tr v-for="subscription in filteredSubscriptions" :key="subscription.id" class="hover:bg-primary/5 transition-all group">
                <td class="px-6 py-4">
                  <div class="font-bold text-gray-800">{{ subscription.plan_name }}</div>
                  <div class="text-primary font-medium text-xs">{{ subscription.price }}</div>
                </td>
                <td class="px-6 py-4">
                  <div class="flex flex-col gap-1 text-[11px]">
                    <span class="text-gray-500 font-medium">Starts: {{ formatDate(subscription.start_date) }}</span>
                    <span class="text-gray-400 italic">Ends: {{ formatDate(subscription.end_date) }}</span>
                  </div>
                </td>
                <td class="px-6 py-4 font-medium text-gray-600">
                  <div class="flex items-center gap-2">
                    <div class="w-7 h-7 rounded-full bg-gray-100 flex items-center justify-center text-[10px] font-black text-gray-400 border border-gray-200 uppercase">
                      {{ (subscription.ownerName || 'U')[0] }}
                    </div>
                    {{ subscription.ownerName || 'Unknown' }}
                  </div>
                </td>
                <td class="px-6 py-4 text-center">
                  <span 
                    :class="{
                      'bg-amber-100 text-amber-700 border-amber-200': subscription.status == 'pending',
                      'bg-red-100 text-red-700 border-red-200': subscription.status == 'expired',
                      'bg-green-100 text-green-700 border-green-200': subscription.status == 'active'
                    }"
                    class="px-3 py-1 rounded-full text-[10px] font-black uppercase border tracking-tighter"
                  >
                    {{ subscription.status }}
                  </span>
                </td>
                <td class="px-6 py-4">
                  <div class="flex justify-end gap-2">
                    <button v-if="is_super_user != 'true' && subscription.status === 'pending'" @click="pay(subscription)" 
                      class="h-8 w-8 flex items-center justify-center rounded-lg bg-green-600 text-white hover:bg-green-700 transition-colors shadow-sm" title="Pay Now">
                      <i class="fas fa-credit-card text-xs"></i>
                    </button>

                    <button @click="payment(subscription.id)" 
                      class="h-8 w-8 flex items-center justify-center rounded-lg bg-blue-50 text-blue-600 border border-blue-100 hover:bg-blue-100 transition-colors" title="View Payments">
                      <i class="fas fa-list-ul text-xs"></i>
                    </button>

                    <button v-if="is_super_user != 'true'" @click="openUpgradeModal(subscription)"
                      class="h-8 w-8 flex items-center justify-center rounded-lg bg-orange-50 text-orange-600 border border-orange-100 hover:bg-orange-100 transition-colors" title="Upgrade Plan">
                      <i class="fas fa-arrow-up text-xs"></i>
                    </button>

                    <button @click="askDeactivateConfirmation(subscription)"
                      class="h-8 w-8 flex items-center justify-center rounded-lg bg-gray-50 text-gray-400 border border-gray-200 hover:bg-red-50 hover:text-red-600 hover:border-red-100 transition-colors" title="Deactivate">
                      <i class="fas fa-power-off text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="!filteredSubscriptions.length">
                <td colspan="5" class="text-center py-10 text-gray-400 italic">No matching subscriptions found.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <div class="lg:hidden p-4 space-y-4">
        <div v-for="subscription in filteredSubscriptions" :key="subscription.id" class="bg-white rounded-2xl border border-gray-100 shadow-md p-5">
          <div class="flex justify-between items-start mb-4">
            <div>
              <h3 class="font-bold text-gray-800 text-lg leading-none">{{ subscription.plan_name }}</h3>
              <span class="text-primary font-bold text-sm leading-8">{{ subscription.price }}</span>
            </div>
            <span 
              :class="subscription.status == 'active' ? 'bg-green-100 text-green-700 border-green-200' : 'bg-amber-100 text-amber-700 border-amber-200'"
              class="px-3 py-1 rounded-full text-[10px] font-black uppercase border"
            >
              {{ subscription.status }}
            </span>
          </div>

          <div class="grid grid-cols-2 gap-4 text-[11px] mb-4">
            <div class="text-gray-500">
              <p class="uppercase font-black text-gray-400 tracking-widest mb-1 text-[9px]">Owner</p>
              <p class="font-bold">{{ subscription.ownerName || 'Unknown' }}</p>
            </div>
            <div class="text-gray-500">
              <p class="uppercase font-black text-gray-400 tracking-widest mb-1 text-[9px]">Billing Cycle</p>
              <p class="font-bold">{{ formatDate(subscription.start_date) }} - {{ formatDate(subscription.end_date) }}</p>
            </div>
          </div>

          <div class="flex flex-wrap gap-2 pt-4 border-t border-gray-50">
            <button v-if="is_super_user != 'true' && subscription.status === 'pending'" @click="pay(subscription)" 
              class="flex-1 px-3 py-2 bg-green-600 text-white rounded-xl text-xs font-bold transition-all shadow-md">
              <i class="fas fa-credit-card mr-1"></i> Pay
            </button>
            <button @click="payment(subscription.id)" 
              class="flex-1 px-3 py-2 bg-blue-50 text-blue-600 border border-blue-100 rounded-xl text-xs font-bold transition-all">
              <i class="fas fa-receipt mr-1"></i> Logs
            </button>
            <button v-if="is_super_user != 'true'" @click="openUpgradeModal(subscription)"
              class="flex-1 px-3 py-2 bg-orange-50 text-orange-600 border border-orange-100 rounded-xl text-xs font-bold transition-all">
              <i class="fas fa-rocket mr-1"></i> Upgrade
            </button>
          </div>
        </div>
      </div>

      <div class="p-6 bg-gray-50/50 border-t border-gray-100 flex flex-col sm:flex-row justify-between items-center gap-4">
        <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">
          Showing Page <span class="text-primary">{{ currentPage }}</span> of {{ totalPages }}
        </p>
        <div class="flex gap-2">
          <button @click="fetchSubscriptions(previous)" :disabled="!previous"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm">
            <i class="fas fa-chevron-left mr-2"></i> Previous
          </button>
          <button @click="fetchSubscriptions(next)" :disabled="!next"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 transition-all shadow-sm">
            Next <i class="fas fa-chevron-right ml-2"></i>
          </button>
        </div>
      </div>
    </div>

    <PaymentModal v-if="paymentVisible" :visible="paymentVisible" :payload="paymentPayload" @close="paymentVisible = false" @paid="handlePaymentSuccess" />
    <UpgradeSubscriptionModal v-if="showUpgradeModal" :visible="showUpgradeModal" :subscriptionId="selectedSubscriptionId" @close="showUpgradeModal = false" @plan-upgraded="fetchSubscriptions" />
    <ConfirmModal v-if="confirmVisible" :visible="confirmVisible" title="Confirm Deactivate" message="Are you sure you want to deactivate this subscription? Access to premium features will be restricted." @confirm="Deactivate" @cancel="confirmVisible = false" />
  </div>
</template>



<script>
import AddSubscription from "./add.vue";
import UpdateSubscription from "./update.vue";
import PaymentModal from "./Payment.vue";
import UpgradeSubscriptionModal from "./upgradePlan.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Toast from "@/components/Toast.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  name: "subscriptionView",
  components: {
    AddSubscription,
    UpdateSubscription,
    ConfirmModal,
    Toast,
    PaymentModal,
    UpgradeSubscriptionModal,
    Loading
    
  },
  data() {
    return {
      subscriptions: [],
      searchTerm: "",
      visible: false,
      updateVisible: false,
      confirmVisible: false,
      showUpgradeModal: false,
      paymentVisible: false,
      paymentPayload: null,
      editing: null,
      deleting: null,
      is_super_user:false,
      selectedSubscriptionId: null,
      // ✅ Pagination state
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      pageSize: 10,
      pageSizes: [5, 10, 20, 50, 100],
      status:"",
      ordering: "-id",
      subscriptionToAD:null,
      loading:false
     
    };
  },
  computed: {
    filteredSubscriptions() {
      const term = this.searchTerm.toLowerCase();
      return this.subscriptions.filter((sub) =>
        Object.values(sub).some((val) =>
          String(val).toLowerCase().includes(term)
        )
      );
    },
  },
  mounted() {

    this.is_super_user=localStorage.getItem('is_superuser');
    console.log("is super user",this.is_super_user)

    this.fetchSubscriptions();

  },
  methods: {
    openUpgradeModal(subscription) {
      this.selectedSubscriptionId = subscription.id; // store the subscription ID
      this.showUpgradeModal = true; // open the modal
    },
    async fetchSubscriptions(url = null) {
      this.loading=true
      try {
        let params = {
          user_id__id: localStorage.getItem("userId"),
          ordering: "-id",
          status:this.status
        };

        if (localStorage.getItem("is_superuser") == "true") {
          params = {
            ordering: "-id",
            status:this.status
          };
        }

        console.log("params for subscription",params)

        const pageUrl =
          url || `/get_subscription?page=1&page_size=${this.pageSize}&search=${this.searchTerm}&ordering=${this.ordering}`;
        const res = await this.$apiGet(pageUrl, params);
        console.log("res", res);

        this.subscriptions = res.data || [];
        this.currentPage = res.current_page || 1;
        this.totalPages = res.total_pages || 1;
        this.next = res.next || null;
        this.previous = res.previous || null;

        // Fetch owner for each subscription (handle missing IDs)
        await Promise.all(
          this.subscriptions.map(async (sub) => {
            if (sub.user_id) {
              try {
                const ownerRes = await this.$apiGetById(
                  "get_user",
                  sub.user_id
                );
                sub.ownerName = ownerRes.first_name || "Unknown";
              } catch (err) {
                console.warn(`Failed to fetch user ${sub.user_id}`, err);
                sub.ownerName = "Unknown";
              }
            } else {
              sub.ownerName = "Unknown"; // fallback if no user_id
            }
          })
        );

        console.log("subscriptions", this.subscriptions);
      } catch (e) {
        console.error("Error fetching subscriptions", e);
        this.subscriptions = [];
        this.currentPage = 1;
        this.totalPages = 1;
        this.next = null;
        this.previous = null;
      }
      finally {
        this.loading=false
      }
    },

    //   async fetch() {
    //   let params = {};

    //   try {
    //     const isSuperuser = localStorage.getItem("is_superuser") === "true";
    //     // convert to boolean
    //     if (!isSuperuser) {
    //       params = {
    //         user_id__id: localStorage.getItem("userId"),
    //       };
    //     }

    //     console.log("params", params);

    //     const res = await this.$apiGet("/get_subscription", params);

    //     this.subscriptions = res.data || [];
    //   } catch (e) {
    //     console.error("Error fetching subscriptions", e);
    //     this.subscriptions = [];
    //   }
    // },

    formatDate(dateStr) {
      if (!dateStr) return "";
      const date = new Date(dateStr);
      return date.toLocaleDateString();
    },
    edit(subscription) {
      this.editing = subscription;
      this.updateVisible = true;
    },
    pay(subscription) {
      this.paymentPayload = {
        user_id: subscription.user_id,
        subscription_id: subscription.id,
        amount: subscription.price,
      };
      this.paymentVisible = true;
    },
    confirmDelete(subscription) {
      this.deleting = subscription;
      this.confirmVisible = true;
    },
    async deleteSubscription() {
      try {
        await this.$apiDelete(`/delete_subscription/${this.deleting.id}`);
        this.$root.$refs.toast.showToast(
          "Subscription deleted successfully",
          "success"
        );
        this.fetchSubscriptions();
      } catch (e) {
        this.$root.$refs.toast.showToast(
          "Failed to delete subscription",
          "error"
        );
      } finally {
        this.confirmVisible = false;
        this.deleting = null;
      }
    },
    payment(subscriptionId) {
      if (subscriptionId)
        this.$router.push({
          name: "subscriptionsPayment_view",
          params: { id: subscriptionId },
        });
    },
     askDeactivateConfirmation(subscription) {
      this.subscriptionToAD=subscription
      this.confirmVisible = true;
    },
    async Deactivate(){
      this.confirmVisible=false
       try{
       console.log("subscription id",this.subscriptionToAD.id)
await this.$apiPatch(`/update_subscription`,{status:"terminated"},this.subscriptionToAD.id);
        this.$root.$refs.toast.showToast(
          "Subscription Deactivated successfully",
          "success"
        );
       }catch (e){
this.$root.$refs.toast.showToast(
          "Failed to Deactivate subscription",
          "error"
        );
       }
    },
    handlePaymentSuccess() {
      this.$root.$refs.toast.showToast("Payment successful", "success");
      this.paymentVisible = false;
      this.fetchSubscriptions();
    },
  },
};
</script>
