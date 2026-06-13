<template>
  <section id="contact" class="py-24 bg-slate-50 relative overflow-hidden">
    <Toast ref="toast" />

    <div class="absolute top-0 right-0 w-[500px] h-[500px] bg-primary/5 rounded-full blur-[120px] -z-0 translate-x-1/2 -translate-y-1/2"></div>
    <div class="absolute bottom-0 left-0 w-[500px] h-[500px] bg-secondary/10 rounded-full blur-[120px] -z-0 -translate-x-1/2 translate-y-1/2"></div>

    <div class="max-w-7xl mx-auto px-6 relative z-10">
      
      <div class="max-w-3xl mb-16">
        <span class="inline-block px-4 py-1.5 mb-4 text-[10px] font-black tracking-[0.3em] text-primary bg-primary/10 rounded-full uppercase">
          Get In Touch
        </span>
        <h2 class="text-5xl font-black text-slate-900 mb-6 tracking-tight leading-none">
          Let’s Build Something <span class="text-primary italic">Great Together.</span>
        </h2>
        <p class="text-slate-500 text-lg font-medium leading-relaxed">
          Whether you're scaling a portfolio or just starting out, our team is ready to help you master your property management.
        </p>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-12 gap-12 items-stretch">
        
        <div class="lg:col-span-4 space-y-4">
          <div class="bg-slate-900 rounded-[2.5rem] p-10 text-white h-full relative overflow-hidden group">
            <div class="absolute bottom-0 right-0 w-32 h-32 bg-primary/20 rounded-full blur-2xl group-hover:bg-primary/40 transition-colors"></div>

            <h3 class="text-2xl font-black mb-10 tracking-tight">Contact Information</h3>

            <div class="space-y-10">
              <div v-for="info in contactInfo" :key="info.label" class="flex gap-6 group/item">
                <div class="w-12 h-12 rounded-2xl bg-white/10 flex items-center justify-center text-primary group-hover/item:bg-primary group-hover/item:text-white transition-all duration-300">
                  <i :class="info.icon" class="text-xl"></i>
                </div>
                <div>
                  <p class="text-[10px] font-black uppercase tracking-widest text-slate-400 mb-1">{{ info.label }}</p>
                  <p class="text-sm font-bold text-slate-100">{{ info.value }}</p>
                </div>
              </div>
            </div>

            <div class="mt-20 flex gap-4">
              <a href="#" class="w-10 h-10 rounded-full border border-white/10 flex items-center justify-center hover:bg-white hover:text-slate-900 transition-all"><i class="fab fa-linkedin-in text-xs"></i></a>
              <a href="#" class="w-10 h-10 rounded-full border border-white/10 flex items-center justify-center hover:bg-white hover:text-slate-900 transition-all"><i class="fab fa-twitter text-xs"></i></a>
              <a href="#" class="w-10 h-10 rounded-full border border-white/10 flex items-center justify-center hover:bg-white hover:text-slate-900 transition-all"><i class="fab fa-instagram text-xs"></i></a>
            </div>
          </div>
        </div>

        <div class="lg:col-span-8">
          <div class="bg-white rounded-[2.5rem] p-10 md:p-14 shadow-2xl shadow-slate-200/60 border border-slate-100">
            <form @submit.prevent="submitForm" class="space-y-8">
              
              <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                <div class="relative group">
                  <input type="text" v-model="form.full_name" required placeholder=" " 
                    class="peer w-full bg-transparent border-b-2 border-slate-200 py-3 outline-none focus:border-primary transition-colors font-bold text-slate-800" />
                  <label class="absolute left-0 top-3 text-slate-400 font-black text-[10px] uppercase tracking-widest transition-all peer-focus:-top-4 peer-focus:text-primary peer-[:not(:placeholder-shown)]:-top-4">
                    Full Name
                  </label>
                </div>

                <div class="relative group">
                  <input type="email" v-model="form.email" required placeholder=" " 
                    class="peer w-full bg-transparent border-b-2 border-slate-200 py-3 outline-none focus:border-primary transition-colors font-bold text-slate-800" />
                  <label class="absolute left-0 top-3 text-slate-400 font-black text-[10px] uppercase tracking-widest transition-all peer-focus:-top-4 peer-focus:text-primary peer-[:not(:placeholder-shown)]:-top-4">
                    Email Address
                  </label>
                </div>
              </div>

              <div class="relative group">
                <input type="text" v-model="form.subject" placeholder=" " 
                  class="peer w-full bg-transparent border-b-2 border-slate-200 py-3 outline-none focus:border-primary transition-colors font-bold text-slate-800" />
                <label class="absolute left-0 top-3 text-slate-400 font-black text-[10px] uppercase tracking-widest transition-all peer-focus:-top-4 peer-focus:text-primary peer-[:not(:placeholder-shown)]:-top-4">
                  What are you interested in?
                </label>
              </div>

              <div class="relative group">
                <textarea v-model="form.message" rows="4" placeholder=" " 
                  class="peer w-full bg-transparent border-b-2 border-slate-200 py-3 outline-none focus:border-primary transition-colors font-bold text-slate-800 resize-none"></textarea>
                <label class="absolute left-0 top-3 text-slate-400 font-black text-[10px] uppercase tracking-widest transition-all peer-focus:-top-4 peer-focus:text-primary peer-[:not(:placeholder-shown)]:-top-4">
                  Your Message
                </label>
              </div>

              <div class="flex flex-col md:flex-row md:items-center justify-between gap-6 pt-6">
                <p class="text-[10px] text-slate-400 font-bold max-w-xs uppercase tracking-widest leading-relaxed">
                  By clicking send, you agree to our <span class="text-primary underline cursor-pointer">privacy policy</span>.
                </p>
                
                <button
                  type="submit"
                  :disabled="loading"
                  class="group relative bg-primary hover:bg-dprimary text-white py-5 px-12 rounded-2xl font-black text-[11px] uppercase tracking-[0.3em] transition-all duration-300 shadow-xl shadow-primary/25 disabled:opacity-50 overflow-hidden"
                >
                  <span class="relative z-10 flex items-center gap-3">
                    {{ loading ? 'Transmitting...' : 'Send Message' }}
                    <i class="fas fa-paper-plane text-[10px] group-hover:translate-x-1 group-hover:-translate-y-1 transition-transform"></i>
                  </span>
                  <div class="absolute inset-0 bg-gradient-to-r from-transparent via-white/10 to-transparent -translate-x-full group-hover:translate-x-full transition-transform duration-1000"></div>
                </button>
              </div>

            </form>
          </div>
        </div>

      </div>
    </div>
  </section>
</template>

<script>
import Toast from "../../../components/Toast.vue";
import axios from "axios";

export default {
  name: "ContactUs",
  components: { Toast },
  data() {
    return {
      loading: false,
      contactInfo: [
        { label: 'Address', value: 'Sur Constraction,HO,B2-03', icon: 'fas fa-map-marker-alt' },
        { label: 'Email', value: 'alphaengineering41@gmail.com', icon: 'fas fa-envelope' },
        { label: 'Phone', value: '0922421141', icon: 'fas fa-phone' },
      ],
      form: {
        full_name: "",
        email: "",
        subject: "",
        message: "",
      },
    };
  },
  methods: {
    async submitForm() {
      this.loading = true;
      try {
        const payload = { ...this.form };
        const response = await axios.post('https://alphapms.sunriseworld.org/api/contact_us', payload);
        
        this.$root.$refs.toast.showToast(response.data.success || "Message sent successfully!", "success");
        this.form = { full_name: "", email: "", subject: "", message: "" };
      } catch (error) {
        this.$root.$refs.toast.showToast("Connection error. Please try again.", "error");
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
/* Focus transition for floating labels */
input:focus ~ label,
textarea:focus ~ label {
  color: var(--color-primary);
}

/* Custom easing for the shine effect */
.transition-transform {
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}
</style>