<template>
  <div class="min-h-screen bg-background p-4 md:p-8">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Rent Commissions..." />

    <div class="max-w-7xl mx-auto bg-white shadow-xl rounded-2xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md text-white">
            <i class="fas fa-file-contract text-2xl"></i>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Rent Commissions</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Lease Revenue Management</p>
          </div>
        </div>

        <button
          @click="showModal = true"
          class="bg-white text-primary hover:bg-gray-50 font-bold px-6 py-2.5 rounded-lg shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2"
        >
          <i class="fas fa-plus-circle"></i> Add Commission
        </button>
      </div>

      <div class="p-6 md:p-8">
        <div class="hidden md:block overflow-hidden rounded-xl border border-gray-100 shadow-sm">
          <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr class="text-left text-[11px] font-black text-gray-400 uppercase tracking-widest">
                <th class="px-6 py-4">SaaS Commission</th>
                <th class="px-6 py-4">Broker Commission</th>
                <th class="px-6 py-4 text-center">Total Commission</th>
                <th class="px-6 py-4">Rent Reference</th>
                <th class="px-6 py-4 text-right">Created At</th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-100">
              <tr
                v-for="commission in commissions"
                :key="commission.id"
                class="hover:bg-primary/5 transition-colors group"
              >
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
                <td class="px-6 py-4 whitespace-nowrap text-center">
                  <span class="px-3 py-1 bg-green-50 text-green-700 rounded-md text-xs font-extrabold border border-green-100">
                    {{ commission.total_commission }}%
                  </span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                   <div class="flex items-center gap-2">
                     <i class="fas fa-key text-gray-300 text-[10px]"></i>
                     <span class="text-sm font-bold text-gray-700">{{ commission.rent }}</span>
                   </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-right text-xs font-medium text-gray-400">
                  {{ commission.created_at | formatDate }}
                </td>
              </tr>
              <tr v-if="commissions.length === 0">
                <td colspan="5" class="text-center py-12 text-gray-400 italic">
                   <i class="fas fa-folder-open block text-3xl mb-2 opacity-20"></i>
                   No commissions found.
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden space-y-4">
          <div
            v-for="commission in commissions"
            :key="commission.id"
            class="bg-white rounded-2xl border border-gray-100 shadow-md p-5 border-l-4 border-l-primary"
          >
            <div class="flex justify-between items-center mb-4">
              <span class="text-xs font-black text-gray-400 uppercase tracking-widest">Rent #{{ commission.rent }}</span>
              <span class="text-[10px] text-gray-400">{{ commission.created_at | formatDate }}</span>
            </div>
            
            <div class="flex items-center justify-between bg-gray-50 p-3 rounded-xl">
              <div class="text-center">
                <p class="text-[9px] font-bold text-gray-400 uppercase">SaaS</p>
                <p class="text-sm font-bold text-blue-600">{{ commission.saas_commission }}%</p>
              </div>
              <div class="text-center">
                <p class="text-[9px] font-bold text-gray-400 uppercase">Broker</p>
                <p class="text-sm font-bold text-purple-600">{{ commission.broker_commission }}%</p>
              </div>
              <div class="h-8 w-[1px] bg-gray-200"></div>
              <div class="text-center">
                <p class="text-[9px] font-bold text-gray-400 uppercase">Total</p>
                <p class="text-sm font-black text-green-600">{{ commission.total_commission }}%</p>
              </div>
            </div>
          </div>
          
          <div v-if="commissions.length === 0" class="text-center py-10 text-gray-400 italic bg-gray-50 rounded-xl">
             No commissions found.
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
        const res = await this.$apiGet("/get_rent_commissions");
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
