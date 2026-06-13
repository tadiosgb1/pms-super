<template>
  <div class="min-h-screen bg-background p-6 md:p-10">
    <div class="max-w-7xl mx-auto mb-10">
      <div class="flex items-center gap-4 mb-2">
        <div class="p-3 bg-primary rounded-2xl shadow-lg shadow-primary/20">
          <i class="fas fa-cogs text-white text-2xl"></i>
        </div>
        <h1 class="text-3xl font-black text-gray-800 tracking-tight">System Configurations</h1>
      </div>
      <p class="text-gray-500 font-medium ml-1">Global settings and financial rule-sets for the platform.</p>
    </div>

    <div class="max-w-7xl mx-auto grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      
      <div class="group bg-white rounded-3xl shadow-sm border border-gray-100 p-8 hover:shadow-2xl hover:shadow-primary/10 transition-all duration-300 relative overflow-hidden">
        <div class="absolute -top-10 -right-10 w-32 h-32 bg-primary/5 rounded-full group-hover:scale-150 transition-transform duration-500"></div>
        
        <div class="relative z-10 flex flex-col h-full">
          <div class="w-14 h-14 bg-blue-50 text-blue-600 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-blue-600 group-hover:text-white transition-colors duration-300">
            <i class="fas fa-layer-group text-2xl"></i>
          </div>
          
          <h3 class="text-xl font-bold text-gray-800 mb-3">Subscription Plans</h3>
          <p class="text-gray-500 text-sm leading-relaxed mb-8">
            Manage tier limitations, feature access, and pricing structures for all customer accounts.
          </p>
          
          <div class="mt-auto">
            <router-link
              to="/plans_view"
              class="w-full inline-flex items-center justify-center gap-2 px-6 py-3 bg-gray-50 text-gray-700 font-bold rounded-xl group-hover:bg-primary group-hover:text-white transition-all duration-300"
            >
              Configure Plans <i class="fas fa-arrow-right text-xs"></i>
            </router-link>
          </div>
        </div>
      </div>

      <!-- <div class="group bg-white rounded-3xl shadow-sm border border-gray-100 p-8 hover:shadow-2xl hover:shadow-primary/10 transition-all duration-300 relative overflow-hidden">
        <div class="absolute -top-10 -right-10 w-32 h-32 bg-green-500/5 rounded-full group-hover:scale-150 transition-transform duration-500"></div>
        
        <div class="relative z-10 flex flex-col h-full">
          <div class="w-14 h-14 bg-green-50 text-green-600 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-green-600 group-hover:text-white transition-colors duration-300">
            <i class="fas fa-hand-holding-usd text-2xl"></i>
          </div>
          
          <h3 class="text-xl font-bold text-gray-800 mb-3">Sale Commissions</h3>
          <p class="text-gray-500 text-sm leading-relaxed mb-8">
            Define global percentage splits for SaaS fees, broker earnings, and total tax handling for sales.
          </p>
          
          <div class="mt-auto">
            <router-link
              to="/sale-commissions"
              class="w-full inline-flex items-center justify-center gap-2 px-6 py-3 bg-gray-50 text-gray-700 font-bold rounded-xl group-hover:bg-green-600 group-hover:text-white transition-all duration-300"
            >
              Adjust Sales <i class="fas fa-arrow-right text-xs"></i>
            </router-link>
          </div>
        </div>
      </div> -->
<!-- 
      <div class="group bg-white rounded-3xl shadow-sm border border-gray-100 p-8 hover:shadow-2xl hover:shadow-primary/10 transition-all duration-300 relative overflow-hidden">
        <div class="absolute -top-10 -right-10 w-32 h-32 bg-orange-500/5 rounded-full group-hover:scale-150 transition-transform duration-500"></div>
        
        <div class="relative z-10 flex flex-col h-full">
          <div class="w-14 h-14 bg-orange-50 text-orange-600 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-orange-600 group-hover:text-white transition-colors duration-300">
            <i class="fas fa-key text-2xl"></i>
          </div>
          
          <h3 class="text-xl font-bold text-gray-800 mb-3">Rent Commissions</h3>
          <p class="text-gray-500 text-sm leading-relaxed mb-8">
            Set brokerage and platform rules for rental property transactions and recurring fees.
          </p>
          
          <div class="mt-auto">
            <router-link
              to="/rent-commissions"
              class="w-full inline-flex items-center justify-center gap-2 px-6 py-3 bg-gray-50 text-gray-700 font-bold rounded-xl group-hover:bg-orange-600 group-hover:text-white transition-all duration-300"
            >
              Adjust Rentals <i class="fas fa-arrow-right text-xs"></i>
            </router-link>
          </div>
        </div>
      </div> -->


      
      <div class="group bg-white rounded-3xl shadow-sm border border-gray-100 p-8 hover:shadow-2xl hover:shadow-primary/10 transition-all duration-300 relative overflow-hidden">
        <div class="absolute -top-10 -right-10 w-32 h-32 bg-orange-500/5 rounded-full group-hover:scale-150 transition-transform duration-500"></div>
        
        <div class="relative z-10 flex flex-col h-full">
          <div class="w-14 h-14 bg-orange-50 text-orange-600 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-orange-600 group-hover:text-white transition-colors duration-300">
            <i class="fas fa-key text-2xl"></i>
          </div>
          
          <h3 class="text-xl font-bold text-gray-800 mb-3">Post Fee set up</h3>
          <p class="text-gray-500 text-sm leading-relaxed mb-8">
            Set Post fee for tenants and brokers
          </p>
          
          <div class="mt-auto">
            <router-link
              to="dashboard/systemconfigurations"
              class="w-full inline-flex items-center justify-center gap-2 px-6 py-3 bg-gray-50 text-gray-700 font-bold rounded-xl group-hover:bg-orange-600 group-hover:text-white transition-all duration-300"
            >
              Post fee fee <i class="fas fa-arrow-right text-xs"></i>
            </router-link>
          </div>
        </div>
      </div>
    </div>

    <div class="max-w-7xl mx-auto mt-12 p-6 bg-blue-50 rounded-2xl border border-blue-100 flex items-start gap-4">
      <i class="fas fa-info-circle text-blue-500 mt-1"></i>
      <p class="text-sm text-blue-800 leading-relaxed">
        <strong>Note:</strong> Changes made in these sections are global. Updating commission splits or plan limits will reflect immediately for new transactions and subscription renewals.
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Configurations",
};
</script>

<style scoped>

</style>
