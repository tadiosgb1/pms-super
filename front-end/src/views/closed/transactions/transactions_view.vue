<template>
  <div class="p-4 md:p-8 bg-slate-50 min-h-screen">
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
      <div>
        <div class="flex items-center gap-2 mb-1">
          <div class="w-2 h-6 bg-orange-500 rounded-full"></div>
          <h1 class="text-2xl font-black text-slate-800 uppercase tracking-tight">Financial Ledger</h1>
        </div>
        <p class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] ml-4">
          Monitoring all revenue streams
        </p>
      </div>

      <div class="bg-slate-200/50 p-1.5 rounded-[2rem] flex items-center shadow-inner border border-slate-200">
        <button
          v-for="tab in tabs"
          :key="tab.key"
          @click="activeTab = tab.key"
          :class="[
            activeTab === tab.key 
              ? 'bg-white text-orange-600 shadow-md transform scale-100' 
              : 'text-slate-500 hover:text-slate-700 hover:bg-slate-200/50',
            'px-6 py-2.5 rounded-[1.5rem] text-[10px] font-black uppercase tracking-widest transition-all duration-300 ease-out outline-none'
          ]"
        >
          <i :class="tab.key === 'saas' ? 'fas fa-cloud' : 'fas fa-handshake'" class="mr-2"></i>
          {{ tab.label }}
        </button>
      </div>
    </div>

    <div class="relative min-h-[400px]">
      <transition 
        name="fade-slide" 
        mode="out-in"
      >
        <div :key="activeTab" class="animate-in fade-in slide-in-from-bottom-4 duration-500">
          <SaasTransactions v-if="activeTab === 'saas'" />
          <BrokerTransactions v-if="activeTab === 'broker'" />
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import SaasTransactions from "./SaasTransactions.vue";
import BrokerTransactions from "./BrokerTransactions.vue";

export default {
  name: "TransactionsRoot",
  components: { SaasTransactions, BrokerTransactions },
  data() {
    return {
      tabs: [
        { key: "saas", label: "SaaS Sales" },
        { key: "broker", label: "Brokerage" },
      ],
      activeTab: "saas",
    };
  },
};
</script>

<style scoped>
/* Smooth transition between tabs */
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s ease;
}
.fade-slide-enter-from {
  opacity: 0;
  transform: translateX(10px);
}
.fade-slide-leave-to {
  opacity: 0;
  transform: translateX(-10px);
}
</style>