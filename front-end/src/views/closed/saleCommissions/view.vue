<template>
  <div class="min-h-screen bg-background p-4 md:p-8">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Commission Data..." />

    <div class="max-w-7xl mx-auto bg-white shadow-xl rounded-2xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
            </svg>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Sale Commissions</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Financial Distributions</p>
          </div>
        </div>

        <button
          @click="showModal = true"
          class="bg-white text-primary hover:bg-gray-50 font-bold px-6 py-2.5 rounded-lg shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2"
        >
          <span class="text-xl">+</span> Add Commission
        </button>
      </div>

      <div class="p-6 md:p-8">
        <div class="hidden md:block overflow-hidden rounded-xl border border-gray-100 shadow-sm">
          <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr class="text-left text-xs font-bold text-gray-500 uppercase tracking-widest">
                <th class="px-6 py-4">Property Sale</th>
                <th class="px-6 py-4">SaaS (%)</th>
                <th class="px-6 py-4">Broker (%)</th>
                <th class="px-6 py-4">Total (%)</th>
                <th class="px-6 py-4 text-right">Created Date</th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-100">
              <tr
                v-for="commission in commissions"
                :key="commission.id"
                class="hover:bg-primary/5 transition-colors group"
              >
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="text-sm font-bold text-gray-800 group-hover:text-primary transition-colors">
                    {{ commission.property_sale_name || commission.property_sale }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="px-2.5 py-1 bg-blue-50 text-blue-700 rounded-md text-xs font-bold border border-blue-100">
                    {{ commission.saas_commission }}%
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="px-2.5 py-1 bg-purple-50 text-purple-700 rounded-md text-xs font-bold border border-purple-100">
                    {{ commission.broker_commission }}%
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="px-2.5 py-1 bg-green-50 text-green-700 rounded-md text-xs font-bold border border-green-100">
                    {{ commission.total_commission }}%
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-right text-xs font-medium text-gray-400">
                  {{ commission.created_at | formatDate }}
                </td>
              </tr>
              <tr v-if="commissions.length === 0">
                <td colspan="5" class="text-center py-12 text-gray-400 italic">
                  No commission records found.
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden space-y-4">
          <div
            v-for="commission in commissions"
            :key="commission.id"
            class="bg-white rounded-2xl border-l-4 border-l-primary shadow-md p-5 border border-gray-100"
          >
            <div class="flex justify-between items-start mb-4">
              <div>
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Property Sale</p>
                <h3 class="font-bold text-gray-900 leading-tight">
                  {{ commission.property_sale_name || commission.property_sale }}
                </h3>
              </div>
              <span class="text-[10px] font-medium text-gray-400">{{ commission.created_at | formatDate }}</span>
            </div>

            <div class="grid grid-cols-3 gap-2 pt-4 border-t border-gray-50">
              <div class="text-center">
                <p class="text-[9px] text-gray-400 font-bold uppercase mb-1">SaaS</p>
                <span class="block text-xs font-bold text-blue-600 bg-blue-50 py-1 rounded-lg">
                  {{ commission.saas_commission }}%
                </span>
              </div>
              <div class="text-center">
                <p class="text-[9px] text-gray-400 font-bold uppercase mb-1">Broker</p>
                <span class="block text-xs font-bold text-purple-600 bg-purple-50 py-1 rounded-lg">
                  {{ commission.broker_commission }}%
                </span>
              </div>
              <div class="text-center">
                <p class="text-[9px] text-gray-400 font-bold uppercase mb-1">Total</p>
                <span class="block text-xs font-bold text-green-600 bg-green-50 py-1 rounded-lg">
                  {{ commission.total_commission }}%
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <AddCommission
      :visible="showModal"
      @close="showModal = false"
      @success="fetchCommissions"
    />
  </div>
</template>


<script>
import AddCommission from "./Add.vue";
import Loading from "@/components/Loading.vue";
export default {
  name: "CommissionView",
  components: { AddCommission,Loading },
  data() {
    return {
      showModal: false,
      commissions: [],
      loading:false,
    };
  },
  filters: {
    formatDate(value) {
      if (!value) return "";
      return new Date(value).toLocaleDateString();
    },
  },
  mounted() {
   const is_super_user=localStorage.getItem('is_superuser');
    //console.log("haspermission=",this.$hasPermission("pms.view_plan"));
    if(is_super_user=='true'){
       this.fetchCommissions();
    }else{
      this.$router.push({
        name:'accessDenied'
      });
    }
  },
  methods: {
    async fetchCommissions() {
      this.loading=true;
      try {
        const res = await this.$apiGet("/get_commissions");
        this.commissions = res.data || [];
      } catch (err) {
        console.error("Failed to fetch commissions:", err);
      }finally{
        this.loading=false;
      }
    },
  },
};
</script>
