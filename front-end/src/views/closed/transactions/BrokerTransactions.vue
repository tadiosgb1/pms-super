<template>
  <div class="relative">
    <Loading :visible="loading" message="Syncing ledger..." />

    <div class="hidden md:block bg-white rounded-[2rem] shadow-sm border border-slate-100 overflow-hidden">
      <table class="w-full text-left border-collapse">
        <thead>
          <tr class="bg-slate-50/50">
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Reference ID</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Amount</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Type</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Created Date</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400 text-right">Last Sync</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-slate-50">
          <tr 
            v-for="txn in transactions" 
            :key="txn.id" 
            class="hover:bg-slate-50/80 transition-colors group"
          >
            <td class="px-6 py-4">
              <span class="text-xs font-bold text-slate-400 group-hover:text-orange-500 transition-colors">#{{ txn.id }}</span>
            </td>
            
            <td class="px-6 py-4">
              <div class="flex flex-col">
                <span class="text-sm font-black text-slate-800">{{ txn.amount }}</span>
                <span class="text-[9px] font-bold text-slate-400 uppercase tracking-tighter">ETB Currency</span>
              </div>
            </td>

            <td class="px-6 py-4">
              <span 
                class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-block"
                :class="txn.transaction_type === 'credit' ? 'bg-emerald-50 text-emerald-600' : 'bg-amber-50 text-amber-600'"
              >
                {{ txn.transaction_type }}
              </span>
            </td>

            <td class="px-6 py-4">
              <span class="text-xs font-bold text-slate-600">{{ formatDate(txn.created_at) }}</span>
            </td>

            <td class="px-6 py-4 text-right">
              <span class="text-[10px] font-black text-slate-400 uppercase tracking-tight">{{ formatDate(txn.updated_at) }}</span>
            </td>
          </tr>

          <tr v-if="transactions.length === 0">
            <td colspan="5" class="py-20 text-center">
              <div class="flex flex-col items-center justify-center space-y-3">
                <i class="fas fa-folder-open text-3xl text-slate-200"></i>
                <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest">No transaction history found</p>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="md:hidden space-y-4">
      <div
        v-for="txn in transactions"
        :key="txn.id"
        class="bg-white rounded-3xl p-6 shadow-sm border border-slate-100 relative overflow-hidden"
      >
        <div class="flex justify-between items-start mb-4">
          <div>
            <span class="text-[9px] font-black text-slate-400 uppercase tracking-[0.2em] block mb-1">Ref #{{ txn.id }}</span>
            <h3 class="text-xl font-black text-slate-800">{{ txn.amount }} <span class="text-xs font-medium text-slate-400">ETB</span></h3>
          </div>
          <span 
            class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest"
            :class="txn.transaction_type === 'credit' ? 'bg-emerald-50 text-emerald-600' : 'bg-amber-50 text-amber-600'"
          >
            {{ txn.transaction_type }}
          </span>
        </div>
        
        <div class="grid grid-cols-2 gap-4 border-t border-slate-50 pt-4">
          <div>
            <span class="text-[8px] font-black text-slate-300 uppercase tracking-widest">Date</span>
            <p class="text-[11px] font-bold text-slate-600">{{ formatDate(txn.created_at) }}</p>
          </div>
          <div class="text-right">
            <span class="text-[8px] font-black text-slate-300 uppercase tracking-widest">Sync</span>
            <p class="text-[11px] font-bold text-slate-600">{{ formatDate(txn.updated_at) }}</p>
          </div>
        </div>
      </div>
      
      <div v-if="transactions.length === 0" class="py-12 text-center bg-white rounded-3xl border border-dashed border-slate-200">
        <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest text-center">Empty Records</p>
      </div>
    </div>
  </div>
</template>


<script>
import Loading from "@/components/Loading.vue";
export default {
  components:{
    Loading
  },
  data() {
    return {
      transactions: [],
      loading:false,
    };
  },
  mounted() {
    this.fetchBrokerTransactions();
  },
  methods: {
    async fetchBrokerTransactions() {
      this.loading=true;
      try {
        const res = await this.$apiGet("/get_broker_transactions");
        this.transactions = res?.data || [];
      } catch (err) {
        console.error("Failed to load Broker transactions:", err);
      }finally{
        this.loading=false;
      }
    },
    formatDate(dateStr) {
      return new Date(dateStr).toLocaleString();
    },
  },
};
</script>
