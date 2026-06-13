
<template>
  <div class="p-6 bg-gradient-to-br from-slate-50 via-white to-tertiary/10 min-h-screen text-sm text-slate-800">
    <!-- Loading -->
    <Loading :visible="loading" message="Loading Broker Sale Details..." />

    <!-- Page Header with Gradient -->
    <div class="flex items-center justify-between mb-8 pb-6 border-b-2 border-primary/20">
      <div class="flex items-center gap-4">
        <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-primary to-dprimary flex items-center justify-center text-white shadow-xl shadow-primary/30">
          <i class="fas fa-file-contract text-2xl"></i>
        </div>
        <div>
          <h1 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Broker Sale Details</h1>
          <p class="text-xs text-slate-500 font-bold uppercase tracking-wider mt-1">Complete Information</p>
        </div>
      </div>
      <button 
        @click="$router.back()" 
        class="px-6 py-3 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl font-bold text-xs uppercase tracking-wider transition-all shadow-md hover:shadow-lg flex items-center gap-2"
      >
        <i class="fas fa-arrow-left"></i>
        Back
      </button>
    </div>

    <!-- Detail Card - Desktop -->
    <div class="bg-white overflow-hidden rounded-[2rem] border-2 border-slate-200 shadow-xl hidden md:block">
      <div class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 px-8 py-6 border-b-2 border-slate-200">
        <h2 class="text-lg font-black text-slate-800 uppercase tracking-tight">
          <i class="fas fa-handshake text-primary mr-3"></i>Sale Information
        </h2>
      </div>
      
      <div class="p-8 space-y-6">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <!-- Listing Price -->
          <div class="bg-gradient-to-br from-primary/5 to-primary/10 rounded-2xl p-6 border-2 border-primary/20 hover:border-primary/40 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-primary/20 flex items-center justify-center">
                <i class="fas fa-tag text-primary text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Listing Price</span>
            </div>
            <p class="text-2xl font-black text-primary ml-11">${{ item.listing_price }}</p>
          </div>

          <!-- Selling Price -->
          <div class="bg-gradient-to-br from-emerald-50 to-emerald-100 rounded-2xl p-6 border-2 border-emerald-200 hover:border-emerald-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-emerald-200 flex items-center justify-center">
                <i class="fas fa-dollar-sign text-emerald-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Selling Price</span>
            </div>
            <p class="text-2xl font-black text-emerald-600 ml-11">${{ item.selling_price }}</p>
          </div>

          <!-- Posting Fee -->
          <div class="bg-gradient-to-br from-secondary/5 to-secondary/10 rounded-2xl p-6 border-2 border-secondary/20 hover:border-secondary/40 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-secondary/20 flex items-center justify-center">
                <i class="fas fa-receipt text-secondary text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Posting Fee</span>
            </div>
            <p class="text-2xl font-black text-secondary ml-11">${{ item.posting_fee }}</p>
          </div>

          <!-- Payment Status -->
          <div class="bg-gradient-to-br from-blue-50 to-blue-100 rounded-2xl p-6 border-2 border-blue-200 hover:border-blue-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-blue-200 flex items-center justify-center">
                <i class="fas fa-check-circle text-blue-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Payment Status</span>
            </div>
            <p class="text-lg font-black text-blue-600 ml-11">{{ item.posting_payment_status }}</p>
          </div>

          <!-- Property -->
          <div class="bg-gradient-to-br from-purple-50 to-purple-100 rounded-2xl p-6 border-2 border-purple-200 hover:border-purple-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-purple-200 flex items-center justify-center">
                <i class="fas fa-home text-purple-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Property</span>
            </div>
            <p class="text-lg font-black text-purple-600 ml-11">{{ item.property?.name || 'N/A' }}</p>
          </div>

          <!-- Broker -->
          <div class="bg-gradient-to-br from-orange-50 to-orange-100 rounded-2xl p-6 border-2 border-orange-200 hover:border-orange-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-orange-200 flex items-center justify-center">
                <i class="fas fa-user-tie text-orange-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Broker</span>
            </div>
            <p class="text-lg font-black text-orange-600 ml-11">{{ item.broker?.first_name || 'N/A' }}</p>
          </div>

          <!-- Buyer -->
          <div class="bg-gradient-to-br from-pink-50 to-pink-100 rounded-2xl p-6 border-2 border-pink-200 hover:border-pink-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-pink-200 flex items-center justify-center">
                <i class="fas fa-user text-pink-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Buyer</span>
            </div>
            <p class="text-lg font-black text-pink-600 ml-11">{{ item.buyer?.first_name || 'N/A' }}</p>
          </div>

          <!-- Status -->
          <div class="bg-gradient-to-br from-cyan-50 to-cyan-100 rounded-2xl p-6 border-2 border-cyan-200 hover:border-cyan-400 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-cyan-200 flex items-center justify-center">
                <i class="fas fa-info-circle text-cyan-600 text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Status</span>
            </div>
            <p class="text-lg font-black text-cyan-600 ml-11">{{ item.status }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Mobile View -->
    <div class="md:hidden bg-white rounded-[2rem] border-2 border-slate-200 shadow-xl overflow-hidden">
      <div class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 px-6 py-5 border-b-2 border-slate-200">
        <h2 class="text-base font-black text-slate-800 uppercase tracking-tight">
          <i class="fas fa-handshake text-primary mr-2"></i>Details
        </h2>
      </div>
      
      <div class="p-6 space-y-4">
        <div class="bg-gradient-to-br from-primary/5 to-primary/10 rounded-xl p-4 border-2 border-primary/20">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-tag text-primary text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Listing Price</span>
          </div>
          <p class="text-xl font-black text-primary">${{ item.listing_price }}</p>
        </div>

        <div class="bg-gradient-to-br from-emerald-50 to-emerald-100 rounded-xl p-4 border-2 border-emerald-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-dollar-sign text-emerald-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Selling Price</span>
          </div>
          <p class="text-xl font-black text-emerald-600">${{ item.selling_price }}</p>
        </div>

        <div class="bg-gradient-to-br from-secondary/5 to-secondary/10 rounded-xl p-4 border-2 border-secondary/20">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-receipt text-secondary text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Posting Fee</span>
          </div>
          <p class="text-lg font-black text-secondary">${{ item.posting_fee }}</p>
        </div>

        <div class="bg-gradient-to-br from-blue-50 to-blue-100 rounded-xl p-4 border-2 border-blue-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-check-circle text-blue-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Payment Status</span>
          </div>
          <p class="text-sm font-black text-blue-600">{{ item.posting_payment_status }}</p>
        </div>

        <div class="bg-gradient-to-br from-purple-50 to-purple-100 rounded-xl p-4 border-2 border-purple-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-home text-purple-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Property</span>
          </div>
          <p class="text-sm font-black text-purple-600">{{ item.property?.name || 'N/A' }}</p>
        </div>

        <div class="bg-gradient-to-br from-orange-50 to-orange-100 rounded-xl p-4 border-2 border-orange-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-user-tie text-orange-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Broker</span>
          </div>
          <p class="text-sm font-black text-orange-600">{{ item.broker?.first_name || 'N/A' }}</p>
        </div>

        <div class="bg-gradient-to-br from-pink-50 to-pink-100 rounded-xl p-4 border-2 border-pink-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-user text-pink-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Buyer</span>
          </div>
          <p class="text-sm font-black text-pink-600">{{ item.buyer?.first_name || 'N/A' }}</p>
        </div>

        <div class="bg-gradient-to-br from-cyan-50 to-cyan-100 rounded-xl p-4 border-2 border-cyan-200">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-info-circle text-cyan-600 text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Status</span>
          </div>
          <p class="text-sm font-black text-cyan-600">{{ item.status }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Loading from "@/components/Loading.vue";

export default {
  components: { Loading },
  data() {
    return {
      item: {},
      loading: false,
    };
  },
  async mounted() {
    this.loading = true;
    const id = this.$route.params.id;
    try {
      const response = await this.$apiGetById('/brokerlistsales', id);
      this.item = response || {};
    } catch (error) {
      console.error(error);
    } finally {
      this.loading = false;
    }
  },
};
</script>
