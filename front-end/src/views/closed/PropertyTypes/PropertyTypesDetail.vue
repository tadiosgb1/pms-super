
<template>
  <div class="p-6 bg-gradient-to-br from-slate-50 via-white to-tertiary/10 min-h-screen text-sm text-slate-800">
    <!-- Loading -->
    <Loading :visible="loading" message="Loading Property Type Details..." />

    <!-- Page Header with Gradient -->
    <div class="flex items-center justify-between mb-8 pb-6 border-b-2 border-primary/20">
      <div class="flex items-center gap-4">
        <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-primary to-dprimary flex items-center justify-center text-white shadow-xl shadow-primary/30">
          <i class="fas fa-info-circle text-2xl"></i>
        </div>
        <div>
          <h1 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Property Type Details</h1>
          <p class="text-xs text-slate-500 font-bold uppercase tracking-wider mt-1">Complete Information</p>
        </div>
      </div>
      <button 
        @click="$router.back()" 
        class="px-6 py-3 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl font-bold text-xs uppercase tracking-wider transition-all duration-200 shadow-md hover:shadow-lg flex items-center gap-2"
      >
        <i class="fas fa-arrow-left"></i>
        Back
      </button>
    </div>

    <!-- Detail Card - Desktop -->
    <div class="bg-white overflow-hidden rounded-[2rem] border-2 border-slate-200 shadow-xl hidden md:block">
      <div class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 px-8 py-6 border-b-2 border-slate-200">
        <h2 class="text-lg font-black text-slate-800 uppercase tracking-tight">
          <i class="fas fa-building text-primary mr-3"></i>Property Type Information
        </h2>
      </div>
      
      <div class="p-8 space-y-6">
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <!-- ID Card -->
          <div class="bg-gradient-to-br from-primary/5 to-primary/10 rounded-2xl p-6 border-2 border-primary/20 hover:border-primary/40 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-primary/20 flex items-center justify-center">
                <i class="fas fa-hashtag text-primary text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">ID</span>
            </div>
            <p class="text-2xl font-black text-primary ml-11">{{ item.id }}</p>
          </div>

          <!-- Name Card -->
          <div class="bg-gradient-to-br from-secondary/5 to-secondary/10 rounded-2xl p-6 border-2 border-secondary/20 hover:border-secondary/40 transition-all duration-300 hover:shadow-lg">
            <div class="flex items-center gap-3 mb-2">
              <div class="w-8 h-8 rounded-lg bg-secondary/20 flex items-center justify-center">
                <i class="fas fa-tag text-secondary text-sm"></i>
              </div>
              <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Name</span>
            </div>
            <p class="text-2xl font-black text-secondary ml-11">{{ item.name }}</p>
          </div>
        </div>

        <!-- Description Card - Full Width -->
        <div class="bg-gradient-to-br from-tertiary/30 to-primary/5 rounded-2xl p-6 border-2 border-primary/20 hover:border-primary/40 transition-all duration-300 hover:shadow-lg">
          <div class="flex items-center gap-3 mb-4">
            <div class="w-8 h-8 rounded-lg bg-primary/20 flex items-center justify-center">
              <i class="fas fa-align-left text-primary text-sm"></i>
            </div>
            <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Description</span>
          </div>
          <p class="text-base text-slate-700 leading-relaxed ml-11">{{ item.description }}</p>
        </div>
      </div>
    </div>

    <!-- Mobile View -->
    <div class="md:hidden bg-white rounded-[2rem] border-2 border-slate-200 shadow-xl overflow-hidden">
      <div class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 px-6 py-5 border-b-2 border-slate-200">
        <h2 class="text-base font-black text-slate-800 uppercase tracking-tight">
          <i class="fas fa-building text-primary mr-2"></i>Details
        </h2>
      </div>
      
      <div class="p-6 space-y-4">
        <!-- ID -->
        <div class="bg-gradient-to-br from-primary/5 to-primary/10 rounded-xl p-4 border-2 border-primary/20">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-hashtag text-primary text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">ID</span>
          </div>
          <p class="text-xl font-black text-primary">{{ item.id }}</p>
        </div>

        <!-- Name -->
        <div class="bg-gradient-to-br from-secondary/5 to-secondary/10 rounded-xl p-4 border-2 border-secondary/20">
          <div class="flex items-center gap-2 mb-1">
            <i class="fas fa-tag text-secondary text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Name</span>
          </div>
          <p class="text-xl font-black text-secondary">{{ item.name }}</p>
        </div>

        <!-- Description -->
        <div class="bg-gradient-to-br from-tertiary/30 to-primary/5 rounded-xl p-4 border-2 border-primary/20">
          <div class="flex items-center gap-2 mb-2">
            <i class="fas fa-align-left text-primary text-xs"></i>
            <span class="text-xs font-black text-slate-500 uppercase">Description</span>
          </div>
          <p class="text-sm text-slate-700 leading-relaxed">{{ item.description }}</p>
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
      const response = await this.$apiGetById('/propertytypes', id);
      this.item = response || {};
    } catch (error) {
      console.error(error);
    } finally {
      this.loading = false;
    }
  },
};
</script>
