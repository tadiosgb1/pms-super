<template>
  <div class="min-h-screen bg-slate-50 font-sans selection:bg-primary/30">
    <section>
      <header class="bg-primary sticky top-0 z-50 border-b border-white/10 shadow-lg backdrop-blur-md bg-opacity-95 transition-all duration-500">
        <div class="max-w-7xl mx-auto px-6 py-3 flex items-center justify-between">
          
          <div class="flex items-center space-x-3 group cursor-pointer">
            <div class="h-11 w-11 bg-white rounded-xl flex items-center justify-center shadow-inner transform group-hover:rotate-12 transition-transform duration-300">
              <img src="@/assets/img/logo1.jpg" alt="Alpha PMS" class="w-9 h-9 rounded-lg" />
            </div>
            <div class="flex flex-col">
              <span class="text-xl font-black text-white tracking-tighter leading-none">ALPHA</span>
              <span class="text-[10px] font-bold text-white/70 uppercase tracking-[0.2em] leading-none mt-1">Property Suite</span>
            </div>
          </div>

          <nav class="hidden md:flex items-center space-x-8 text-white/90 text-sm font-bold uppercase tracking-widest">
            <a href="#home" class="hover:text-white transition relative after:content-[''] after:absolute after:-bottom-1 after:left-0 after:w-0 after:h-0.5 after:bg-secondary after:transition-all hover:after:w-full">Home</a>
            <a href="#features" class="hover:text-white transition relative after:content-[''] after:absolute after:-bottom-1 after:left-0 after:w-0 after:h-0.5 after:bg-secondary after:transition-all hover:after:w-full">Features</a>
            <a href="#plans" class="hover:text-white transition relative after:content-[''] after:absolute after:-bottom-1 after:left-0 after:w-0 after:h-0.5 after:bg-secondary after:transition-all hover:after:w-full">Pricing</a>
            <a href="#contact" class="hover:text-white transition relative after:content-[''] after:absolute after:-bottom-1 after:left-0 after:w-0 after:h-0.5 after:bg-secondary after:transition-all hover:after:w-full">Contact</a>
          </nav>

          <div class="hidden md:flex items-center space-x-4">
            <div class="relative">
              <select
                v-model="selectedLang"
                class="appearance-none cursor-pointer bg-white/10 hover:bg-white/20 text-white text-xs font-black uppercase tracking-widest px-4 py-2 pr-8 rounded-xl border border-white/20 focus:outline-none transition-all"
              >
                <option value="en" class="text-slate-800">EN</option>
                <option value="am" class="text-slate-800">AM</option>
                <option value="ti" class="text-slate-800">TI</option>
              </select>
              <i class="fas fa-chevron-down absolute right-3 top-1/2 -translate-y-1/2 text-white/50 text-[10px] pointer-events-none"></i>
            </div>

            <button
              @click="showLoginModal = true"
              class="flex items-center gap-2 px-6 py-2.5 font-black text-xs uppercase tracking-widest text-white bg-secondary rounded-xl shadow-lg shadow-secondary/30 hover:bg-opacity-90 active:scale-95 transition-all duration-300"
            >
              <i class="fas fa-lock"></i> Login
            </button>
          </div>

          <button @click="mobileMenuOpen = !mobileMenuOpen" class="md:hidden text-white text-2xl">
            <i class="fas" :class="mobileMenuOpen ? 'fa-times' : 'fa-bars-staggered'"></i>
          </button>
        </div>

        <transition name="menu-slide">
          <div v-if="mobileMenuOpen" class="md:hidden bg-dprimary border-t border-white/10 px-6 py-8 space-y-6">
            <nav class="flex flex-col space-y-4 text-white font-bold uppercase tracking-widest text-sm">
              <a href="#home" @click="mobileMenuOpen=false">Home</a>
              <a href="#features" @click="mobileMenuOpen=false">Features</a>
              <a href="#plans" @click="mobileMenuOpen=false">Pricing</a>
              <a href="#contact" @click="mobileMenuOpen=false">Contact</a>
            </nav>
            <button @click="showLoginModal = true" class="w-full bg-white text-primary font-black py-4 rounded-2xl shadow-xl">
              PORTAL LOGIN
            </button>
          </div>
        </transition>
      </header>
    </section>

    <Hero @openLogin="showLoginModal=true" />
    
    <div class="bg-white py-10">
      <Features />
    </div>

    <PropertiesSection />
    
    <Plans id="plans" />

    <section v-if="showForm" class="max-w-4xl mx-auto my-20 px-8 py-12 bg-white rounded-[2rem] shadow-2xl border border-slate-100 relative overflow-hidden">
      <div class="absolute top-0 right-0 w-32 h-32 bg-primary/5 rounded-bl-full"></div>
      
      <div class="relative z-10">
        <h3 class="text-3xl font-black mb-2 text-slate-800 tracking-tight">Onboard your Property</h3>
        <p class="text-slate-500 mb-8 font-medium">Complete the form below to initialize your PMS instance.</p>
        
        <form @submit.prevent="submitForm" class="grid grid-cols-1 md:grid-cols-2 gap-5">
          <div class="space-y-1">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 ml-1">First Name</label>
            <input v-model="form.first_name" type="text" placeholder="John" class="custom-input" />
          </div>
          <div class="space-y-1">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 ml-1">Last Name</label>
            <input v-model="form.last_name" type="text" placeholder="Doe" class="custom-input" />
          </div>
          <div class="space-y-1">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 ml-1">Business Email</label>
            <input v-model="form.email" type="email" placeholder="john@company.com" class="custom-input" />
          </div>
          <div class="space-y-1">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 ml-1">Mobile Number</label>
            <input v-model="form.phone_number" type="text" placeholder="+251..." class="custom-input" />
          </div>
          <div class="col-span-full space-y-1">
            <label class="text-[10px] font-black uppercase tracking-widest text-slate-400 ml-1">Secure Password</label>
            <input v-model="form.password" type="password" placeholder="••••••••" class="custom-input" />
          </div>

          <div class="col-span-full pt-4">
            <button class="w-full bg-primary hover:bg-dprimary text-white py-5 rounded-2xl text-sm font-black uppercase tracking-[0.2em] shadow-xl shadow-primary/20 transition-all active:scale-[0.98]" type="submit">
              Initialize Subscription
            </button>
          </div>
        </form>
      </div>
    </section>

    <Fqs />
    <ContactUs />
    <Footer />
    
    <LoginModal :visible="showLoginModal" @close="showLoginModal = false" />
  </div>
</template>

<style scoped>
/* Modern Input Styling using Dynamic Colors */
.custom-input {
  @apply w-full bg-slate-50 border border-slate-200 rounded-xl px-4 py-3.5 text-sm font-semibold text-slate-700 
         transition-all outline-none focus:border-primary focus:ring-4 focus:ring-primary/10 placeholder:text-slate-300;
}

/* Smooth Menu Transition */
.menu-slide-enter-active, .menu-slide-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}
.menu-slide-enter-from, .menu-slide-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* Custom Selection Color */
::selection {
  background: var(--color-primary);
  color: white;
}
</style>
<script>


import Fqs from './fqs.vue'
import Footer from './footer.vue'
import Header from './header.vue'
import Plans from "./plans.vue"
import LoginModal from '../auth/login.vue'
import Hero from './hero.vue'
import Features from './features.vue'
import ContactUs from './contactUs.vue'
import PropertiesSection from './PropertiesSection.vue'
export default {
  components:{
    Plans,
    Footer,
    Header,
    Fqs,
    LoginModal,
    Hero,
    Features,
    ContactUs,
    PropertiesSection
  },
  data() {
    return {
      showLoginModal: false,
       selectedLang: "en",
      mobileMenuOpen: false,
      plans: [
        {
          id: 'basic001',
          name: 'Basic Plan',
          max_locations: 2,
          max_staff: 5,
          max_users: 10,
          max_kds: 1,
          kds_enabled: true,
          price: 199,
          billing_cycle: 'monthly'
        },
        {
          id: 'pro002',
          name: 'Pro Plan',
          max_locations: 10,
          max_staff: 25,
          max_users: 50,
          max_kds: 5,
          kds_enabled: true,
          price: 499,
          billing_cycle: 'monthly'
        },
        {
          id: 'enterprise003',
          name: 'Enterprise Plan',
          max_locations: 100,
          max_staff: 200,
          max_users: 500,
          max_kds: 50,
          kds_enabled: true,
          price: 1499,
          billing_cycle: 'monthly'
        }
      ],
      selectedPlan: null,
      showForm: false,
      form: {
        email: '',
        password: '',
        phone_number: '',
        first_name: '',
        middle_name: '',
        last_name: ''
      }
    };
  },
  methods: {
    selectPlan(planId) {
      this.selectedPlan = planId;
      this.showForm = true;
    },
    async submitForm() {
      if (!this.selectedPlan) return alert('Please select a plan.');
      const payload = { ...this.form, plan: this.selectedPlan };

      try {
        const res = await fetch('https://your-api.com/api/sign_up', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });
        const data = await res.json();
        if (res.ok) {
          alert('Registration successful!');
        } else {
          alert(data.error || 'Failed to register');
        }
      } catch (err) {
        console.error(err);
        alert('Something went wrong.');
      }
    },
    controlLogin(status){
      this.showLoginModal=status
    },
    goToLogin() {
      this.$router.push('/login');
    },
    goToRegister() {
      const el = document.getElementById('plans');
      if (el) el.scrollIntoView({ behavior: 'smooth' });
    }
  }
};
</script>


