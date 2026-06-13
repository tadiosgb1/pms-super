<template>
  <div class="relative">
    <Loading :visible="loading" message="Loading SaaS Ledger..." />

    <div class="hidden md:block bg-white rounded-[2rem] shadow-sm border border-slate-100 overflow-hidden">
      <table class="w-full text-left border-collapse">
        <thead>
          <tr class="bg-slate-50/50">
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Txn Reference</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Revenue</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Type</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400">Timestamp</th>
            <th class="px-6 py-5 text-[10px] font-black uppercase tracking-[0.2em] text-slate-400 text-right">Last Update</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-slate-50">
          <tr 
            v-for="txn in transactions" 
            :key="txn.id" 
            class="hover:bg-blue-50/30 transition-colors group"
          >
            <td class="px-6 py-4">
              <div class="flex items-center gap-2">
                <i class="fas fa-hashtag text-[10px] text-slate-300"></i>
                <span class="text-xs font-bold text-slate-500 group-hover:text-blue-600 transition-colors">{{ txn.id }}</span>
              </div>
            </td>
            
            <td class="px-6 py-4">
              <div class="flex flex-col">
                <span class="text-sm font-black text-slate-800">{{ txn.amount }}</span>
                <span class="text-[9px] font-bold text-blue-500/60 uppercase tracking-tighter">SaaS Plan Revenue</span>
              </div>
            </td>

            <td class="px-6 py-4">
              <span 
                class="px-3 py-1 rounded-full text-[9px] font-black uppercase tracking-widest inline-flex items-center gap-1.5"
                :class="txn.transaction_type === 'subscription' || txn.transaction_type === 'credit' 
                  ? 'bg-blue-50 text-blue-600' 
                  : 'bg-slate-100 text-slate-600'"
              >
                <span class="w-1 h-1 rounded-full bg-current"></span>
                {{ txn.transaction_type }}
              </span>
            </td>

            <td class="px-6 py-4">
              <span class="text-xs font-bold text-slate-600 italic">{{ formatDate(txn.created_at) }}</span>
            </td>

            <td class="px-6 py-4 text-right">
              <span class="text-[10px] font-black text-slate-400 uppercase tracking-tight">{{ formatDate(txn.updated_at) }}</span>
            </td>
          </tr>

          <tr v-if="transactions.length === 0">
            <td colspan="5" class="py-24 text-center">
              <div class="flex flex-col items-center justify-center">
                <div class="w-16 h-16 bg-slate-50 rounded-full flex items-center justify-center mb-4">
                  <i class="fas fa-receipt text-2xl text-slate-200"></i>
                </div>
                <p class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em]">No SaaS Transactions to display</p>
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
        class="bg-white rounded-[2rem] p-6 shadow-sm border border-slate-100"
      >
        <div class="flex justify-between items-center mb-4">
          <span class="px-3 py-1 bg-blue-50 text-blue-600 rounded-full text-[9px] font-black uppercase tracking-widest">
            {{ txn.transaction_type }}
          </span>
          <span class="text-[10px] font-bold text-slate-300">#{{ txn.id }}</span>
        </div>

        <div class="mb-6">
          <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest mb-1">Total Amount</p>
          <h3 class="text-2xl font-black text-slate-800">{{ txn.amount }} <span class="text-xs text-slate-400">ETB</span></h3>
        </div>
        
        <div class="space-y-3 pt-4 border-t border-slate-50">
          <div class="flex justify-between items-center">
            <span class="text-[9px] font-black text-slate-300 uppercase">Created</span>
            <span class="text-xs font-bold text-slate-600">{{ formatDate(txn.created_at) }}</span>
          </div>
          <div class="flex justify-between items-center">
            <span class="text-[9px] font-black text-slate-300 uppercase">Modified</span>
            <span class="text-xs font-bold text-slate-600">{{ formatDate(txn.updated_at) }}</span>
          </div>
        </div>
      </div>
      
      <div v-if="transactions.length === 0" class="py-12 text-center bg-white rounded-[2rem] border border-dashed border-slate-200">
        <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest">No Records Found</p>
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
    this.fetchSaasTransactions();
  },
  methods: {

    async fetchSaasTransactions() {
      this.loading=true
      try {
        const res = await this.$apiGet("/get_saas_transactions");
        this.transactions = res?.data || [];
      } catch (err) {
        console.error("Failed to load SaaS transactions:", err);
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
