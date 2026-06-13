<template>
  <section id="plans" class="relative bg-slate-50 py-24 px-6 overflow-hidden">
    <div class="absolute top-0 left-0 w-full h-full opacity-40 pointer-events-none">
      <div class="absolute top-[-10%] left-[-10%] w-[40%] h-[40%] bg-primary/10 rounded-full blur-[120px]"></div>
      <div class="absolute bottom-[-10%] right-[-10%] w-[40%] h-[40%] bg-secondary/10 rounded-full blur-[120px]"></div>
    </div>

    <div class="max-w-7xl mx-auto text-center relative z-10">
      <div class="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-white shadow-sm border border-slate-100 mb-6 animate-bounce-slow">
        <span class="relative flex h-3 w-3">
          <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-secondary opacity-75"></span>
          <span class="relative inline-flex rounded-full h-3 w-3 bg-secondary"></span>
        </span>
        <span class="text-[10px] font-black uppercase tracking-[0.2em] text-slate-500">Flexible Licensing</span>
      </div>

      <h2 class="text-5xl md:text-6xl font-black text-slate-900 mb-6 tracking-tight">
        Simple Scale, <span class="text-transparent bg-clip-text bg-gradient-to-r from-primary to-dprimary">Unmatched Power.</span>
      </h2>
      <p class="text-slate-500 text-lg mb-16 max-w-2xl mx-auto font-medium leading-relaxed">
        No hidden fees. Just transparent pricing designed to grow alongside your property portfolio.
      </p>

      <transition-group 
        name="pricing-list" 
        tag="div" 
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8"
      >
        <div
          v-for="(plan, index) in plans"
          :key="plan.id"
          @click="selectPlan(plan.id)"
          class="pricing-card group relative bg-white rounded-[2.5rem] p-1 transition-all duration-500 cursor-pointer"
          :class="{ 'selected-plan': selectedPlan === plan.id }"
        >
          <div v-if="selectedPlan === plan.id" class="absolute inset-0 bg-gradient-to-br from-primary via-secondary to-dprimary rounded-[2.5rem] -m-[2px] z-0 animate-pulse-slow"></div>
          
          <div class="relative z-10 bg-white rounded-[2.4rem] p-8 h-full flex flex-col overflow-hidden">
            
            <div class="text-left mb-8">
              <div class="flex justify-between items-start">
                <h4 class="text-2xl font-black text-slate-800 group-hover:text-primary transition-colors">{{ plan.name }}</h4>
                <div v-if="index === 1" class="bg-secondary/10 text-secondary text-[9px] font-black px-3 py-1 rounded-full uppercase tracking-tighter">Recommended</div>
              </div>
              <p class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] mt-2">{{ plan.billing_cycle }}</p>
            </div>

            <div class="text-left mb-10">
              <div class="flex items-baseline gap-1">
                <span class="text-5xl font-black text-slate-900 tracking-tighter">{{ plan.price.toLocaleString() }}</span>
                <span class="text-slate-400 font-bold">ETB</span>
              </div>
              <div v-if="plan.original_price && plan.original_price > plan.price" class="flex items-center gap-2 mt-2">
                <span class="line-through text-slate-300 text-sm">{{ plan.original_price }} ETB</span>
                <span class="text-secondary text-xs font-black">SAVE {{ Math.round(((plan.original_price - plan.price)/plan.original_price)*100) }}%</span>
              </div>
            </div>

            <div class="space-y-6 mb-10 flex-grow">
              <div class="space-y-2">
                <div class="flex justify-between text-[10px] font-black uppercase tracking-widest text-slate-500">
                  <span>Locations</span>
                  <span>{{ plan.max_locations }}</span>
                </div>
                <div class="h-1.5 w-full bg-slate-100 rounded-full overflow-hidden">
                  <div class="h-full bg-primary rounded-full transition-all duration-1000" :style="{ width: (plan.max_locations / 50) * 100 + '%' }"></div>
                </div>
              </div>

              <div class="space-y-2">
                <div class="flex justify-between text-[10px] font-black uppercase tracking-widest text-slate-500">
                  <span>Staff Access</span>
                  <span>{{ plan.max_staff }} Units</span>
                </div>
                <div class="h-1.5 w-full bg-slate-100 rounded-full overflow-hidden">
                  <div class="h-full bg-secondary rounded-full transition-all duration-1000" :style="{ width: (plan.max_staff / 20) * 100 + '%' }"></div>
                </div>
              </div>

              <div class="space-y-2">
                <div class="flex justify-between text-[10px] font-black uppercase tracking-widest text-slate-500">
                  <span>Total Users</span>
                  <span>{{ plan.max_users }}</span>
                </div>
                <div class="h-1.5 w-full bg-slate-100 rounded-full overflow-hidden">
                  <div class="h-full bg-slate-800 rounded-full transition-all duration-1000" :style="{ width: (plan.max_users / 100) * 100 + '%' }"></div>
                </div>
              </div>
            </div>

            <button
              class="w-full py-4 rounded-2xl font-black text-[11px] uppercase tracking-[0.2em] transition-all duration-500 relative group/btn"
              :class="selectedPlan === plan.id ? 'bg-primary text-white shadow-xl shadow-primary/30' : 'bg-slate-50 text-slate-400 group-hover:bg-slate-900 group-hover:text-white'"
            >
              <span class="relative z-10 flex items-center justify-center gap-2">
                {{ selectedPlan === plan.id ? '✓ Plan Activated' : 'Choose This Plan' }}
              </span>
            </button>
          </div>
        </div>
      </transition-group>
    </div>
  </section>

  <RegisterUser
    :visible="showRegisterUserModal"
    @close="showRegisterUserModal = false"
    :selectedPlan="selectedPlan"
  />
</template>

<style scoped>
/* 3D Perspective Effect */
.pricing-card {
  perspective: 1000px;
  transform: translateY(0);
  transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.pricing-card:hover {
  transform: translateY(-12px) scale(1.02);
}

/* Selected State Pulse */
@keyframes pulse-slow {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(1.005); }
}
.animate-pulse-slow {
  animation: pulse-slow 3s infinite ease-in-out;
}

/* Entrance Animations */
.pricing-list-enter-active {
  transition: all 0.8s ease-out;
}
.pricing-list-enter-from {
  opacity: 0;
  transform: scale(0.9) translateY(40px);
}

/* Bouncing Badge */
@keyframes bounce-slow {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}
.animate-bounce-slow {
  animation: bounce-slow 4s infinite ease-in-out;
}
</style>

<script>
import RegisterUser from './RegisterUser.vue';
import axios from 'axios';
export default {
  name: 'PlansSection',
  components: { RegisterUser },
  data() {
    return {
      showRegisterUserModal: false,
      selectedPlan: null,
      plans: [],
    };
  },
  mounted() {
    this.fetchPlans();
  },
  methods: {
    selectPlan(id) {
      this.selectedPlan = id;
      this.showRegisterUserModal = true;
    },
    async fetchPlans() {
      try {
        //const response = await this.$apiGet("/get_plans");
        const response = await axios.get("https://alphapms.sunriseworld.org/api/get_plans");
        if (Array.isArray(response.data.data)) {
          this.plans = response.data.data;
        } else {
          console.warn("Unexpected response format:", response);
          this.plans = [];
        }
      } catch (error) {
        console.error("Failed to fetch plans", error);
        this.plans = [];
      }
    },
  },
};
</script>



