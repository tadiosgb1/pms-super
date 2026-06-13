<template>
  <transition name="fade">
    <div v-if="visible" class="fixed inset-0 z-[100] bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4">
      <div class="bg-white w-full max-w-2xl max-h-[85vh] rounded-[2.5rem] shadow-2xl flex flex-col relative animate-in zoom-in duration-200 overflow-hidden">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-4">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center text-white border border-white/20">
              <i class="fas fa-file-signature text-lg"></i>
            </div>
            <div>
              <h2 class="text-lg font-black text-white tracking-tight uppercase leading-none">New Rental</h2>
              <p class="text-white/60 text-[9px] uppercase tracking-[0.2em] font-bold mt-1">Contract Entry</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all">
            <i class="fas fa-times text-sm"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-8 space-y-8 scroll-smooth">
          <form id="rentalForm" @submit.prevent="submitForm" class="space-y-8">
            
            <div class="space-y-4">
              <div class="flex items-center gap-3">
                <span class="w-1.5 h-1.5 rounded-full bg-primary"></span>
                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em]">Guest Information</h3>
              </div>
              
              <div class="space-y-2">
                <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Full Name</label>
                <input
                  v-model="form.guest_name"
                  type="text"
                  placeholder="e.g. Abebe Bikila"
                  required
                  class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary focus:ring-4 focus:ring-primary/10 outline-none transition-all"
                />
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Email Address</label>
                  <input
                    v-model="form.guest_email"
                    type="email"
                    placeholder="guest@mail.com"
                    required
                    class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all"
                  />
                </div>
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Phone Number</label>
                  <input
                    v-model="form.guest_phone"
                    type="tel"
                    placeholder="0911..."
                    required
                    class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all"
                  />
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <div class="flex items-center gap-3">
                <span class="w-1.5 h-1.5 rounded-full bg-primary"></span>
                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em]">Contract Logic</h3>
              </div>
              
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Billing Cycle</label>
                  <div class="relative">
                    <select
                      v-model="form.cycle"
                      required
                      class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none appearance-none cursor-pointer"
                    >
                      <option disabled value="">Select Cycle</option>
                      <option value="daily">Daily</option>
                      <option value="monthly">Monthly</option>
                      <option value="quarterly">Quarterly</option>
                      <option value="yearly">Yearly</option>
                    </select>
                    <i class="fas fa-chevron-down absolute right-5 top-1/2 -translate-y-1/2 text-slate-400 pointer-events-none text-xs"></i>
                  </div>
                </div>

                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Start Date</label>
                  <input
                    v-model="form.start_date"
                    type="date"
                    required
                    class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none"
                  />
                </div>

                  <div class="space-y-2">
                  <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Next Due Date</label>
                  <input
                    v-model="form.next_due_date"
                    type="date"
                    required
                    class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none"
                  />
                </div>
              </div>

              <div class="space-y-2 relative">
                <label class="block text-[10px] font-black text-slate-500 uppercase tracking-widest ml-1">Assigned Workspace</label>
                <input
                  v-model="spaceSearch"
                  type="text"
                  placeholder="Search space..."
                  class="w-full px-5 py-4 bg-slate-50 border border-slate-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all"
                  @input="searchSpaces"
                  @focus="spaceDropdown = true"
                  @blur="hideDropdown"
                  required
                />
                
                <ul
                  v-if="spaces.length > 0 && spaceDropdown"
                  class="absolute z-50 w-full max-h-40 overflow-y-auto bg-white border border-slate-100 rounded-2xl shadow-xl mt-2 overflow-hidden"
                >
                  <li
                    v-for="space in spaces"
                    :key="space.id"
                    class="px-5 py-3 hover:bg-primary/5 hover:text-primary cursor-pointer text-sm font-bold text-slate-700 transition-colors"
                    @mousedown.prevent="selectSpace(space)"
                  >
                    {{ space.name }}
                  </li>
                </ul>
              </div>
            </div>
          </form>
        </div>

        <div class="p-8 border-t border-slate-50 flex items-center justify-end gap-4 shrink-0 bg-white">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-3 text-[10px] font-black uppercase tracking-widest text-slate-400 hover:text-red-500 transition-colors"
          >
            Cancel
          </button>
          <button
            form="rentalForm"
            type="submit"
            :disabled="isSaving"
            class="bg-dprimary hover:bg-primary disabled:bg-slate-200 text-white font-black text-[10px] uppercase tracking-[0.2em] px-10 py-4 rounded-2xl shadow-lg shadow-dprimary/20 active:scale-95 transition-all flex items-center gap-3"
          >
            <i v-if="isSaving" class="fas fa-circle-notch animate-spin"></i>
            {{ isSaving ? 'Processing...' : 'Save Rental' }}
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity 0.2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

/* Hide scrollbar for cleaner look but keep functionality */
.overflow-y-auto {
  scrollbar-width: thin;
  scrollbar-color: #e2e8f0 transparent;
}
</style>

<script>
export default {
  name: "AddRental",
  props: {
    visible: Boolean,
  },
  data() {
    return {
      isSaving: false,
      form: {
        guest_name: "",
        guest_email: "",
        guest_phone: "",
        cycle: "",
        start_date: "",
        next_due_date:"",
        is_active: true,
        user: localStorage.getItem("userId") || 0,
        space: "",
        status:"pending"
      },
      spaces: [],
      spaceSearch: "",
      spaceDropdown: false,
    };
  },
  async mounted() {
    this.fetchSpaces();
  },
  methods: {
    async fetchSpaces() {
      try {
        const response = await this.$getCoworkingSpaces();
        this.spaces = response.spaces || [];
        console.log("spaces", this.spaces);
      } catch (err) {
        console.error("Failed to fetch spaces:", err);
      }
    },

    async searchSpaces() {
      try {
        const response = await this.$getCoworkingSpaces();
        if (this.spaceSearch) {
          this.spaces = response.spaces.filter((s) =>
            s.name.toLowerCase().includes(this.spaceSearch.toLowerCase())
          );
        } else {
          this.spaces = response.spaces;
        }
      } catch (error) {
        console.error("Failed to search spaces:", error);
      }
    },

    selectSpace(space) {
      this.form.space = space.id;
      this.spaceSearch = space.name;
      this.spaceDropdown = false;
    },

    hideDropdown() {
      setTimeout(() => {
        this.spaceDropdown = false;
      }, 200);
    },

    async submitForm() {
      this.isSaving = true;
      try {
        const payload = { ...this.form };
        const res = await this.$apiPost("/post_workspace_rental", payload);
        console.log("Rental added:", res);
        this.$emit("success");
        this.resetForm();
        this.$emit("close");
      } catch (err) {
        console.error("Failed to add rental:", err);
        alert("Failed to add workspace rental.");
      } finally {
        this.isSaving = false;
      }
    },

    resetForm() {
      this.form = {
        guest_name: "",
        guest_email: "",
        guest_phone: "",
        cycle: "",
        start_date: "",
        is_active: true,
        user: localStorage.getItem("userId") || 0,
        space: "",
      };
      this.spaceSearch = "";
    },
  },
};
</script>
