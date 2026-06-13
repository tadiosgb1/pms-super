<template>
  <transition name="fade">
    <div v-if="visible" class="fixed inset-0 z-[100] bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4">
      <div class="bg-white w-full max-w-2xl max-h-[90vh] rounded-[2.5rem] shadow-2xl flex flex-col relative animate-in zoom-in duration-200 overflow-hidden mx-auto">
        
        <div class="bg-gradient-to-r from-orange-600 to-orange-500 px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-4">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center text-white border border-white/20">
              <i class="fas fa-edit text-lg"></i>
            </div>
            <div>
              <h2 class="text-lg font-black text-white tracking-tight uppercase leading-none">Edit Rental</h2>
              <p class="text-white/60 text-[9px] uppercase tracking-[0.2em] font-bold mt-1">Modify Contract Details</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all">
            <i class="fas fa-times text-sm"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-8 space-y-8 scroll-smooth">
          <form id="editRentalForm" @submit.prevent="submitForm" class="space-y-8">
            
            <div class="space-y-4">
              <div class="flex items-center gap-3">
                <span class="w-1.5 h-1.5 rounded-full bg-orange-500"></span>
                <h3 class="text-[10px] font-black text-gray-400 uppercase tracking-[0.2em]">Guest Details</h3>
              </div>
              
              <div class="space-y-2">
                <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Guest Name</label>
                <input 
                  v-model="form.guest_name" 
                  type="text" 
                  class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none transition-all" 
                  required 
                />
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Email</label>
                  <input 
                    v-model="form.guest_email" 
                    type="email" 
                    class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none transition-all" 
                    required 
                  />
                </div>
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Phone</label>
                  <input 
                    v-model="form.guest_phone" 
                    type="text" 
                    class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none transition-all" 
                    required 
                  />
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <div class="flex items-center gap-3">
                <span class="w-1.5 h-1.5 rounded-full bg-blue-500"></span>
                <h3 class="text-[10px] font-black text-gray-400 uppercase tracking-[0.2em]">Rental Settings</h3>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Billing Cycle</label>
                  <select 
                    v-model="form.cycle" 
                    class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none appearance-none cursor-pointer" 
                    required
                  >
                    <option value="daily">Daily</option>
                    <option value="monthly">Monthly</option>
                    <option value="quarterly">Quarterly</option>
                    <option value="yearly">Yearly</option>
                  </select>
                </div>
                <div class="space-y-2">
                  <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Start Date</label>
                  <input 
                    v-model="form.start_date" 
                    type="date" 
                    class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none transition-all" 
                    required 
                  />
                </div>
              </div>

              <div class="space-y-2">
                <label class="block text-[10px] font-black text-gray-500 uppercase tracking-widest ml-1">Workspace Assignment</label>
                <select 
                  v-model="form.space" 
                  class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-orange-500 outline-none appearance-none cursor-pointer" 
                  required
                >
                  <option v-for="space in spaces" :key="space.id" :value="space.id">
                    {{ space.name }}
                  </option>
                </select>
              </div>

              <div class="flex items-center justify-between px-6 py-5 bg-slate-50 rounded-[1.5rem] border border-slate-100 transition-all">
                <div>
                  <p class="text-[11px] font-black text-slate-700 uppercase tracking-tight">Contract Active Status</p>
                  <p class="text-[9px] font-bold text-slate-400 uppercase tracking-widest mt-0.5">Toggle to enable/disable rental access</p>
                </div>
                <label class="relative inline-flex items-center cursor-pointer">
                  <input type="checkbox" v-model="form.is_active" class="sr-only peer">
                  <div class="w-12 h-6 bg-gray-200 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[4px] after:left-[4px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-orange-500"></div>
                </label>
              </div>
            </div>
          </form>
        </div>

        <div class="p-8 border-t border-gray-50 flex items-center justify-end gap-4 shrink-0 bg-white">
          <button 
            @click="$emit('close')" 
            type="button" 
            class="px-6 py-3 text-[10px] font-black uppercase tracking-widest text-gray-400 hover:text-rose-500 transition-colors"
          >
            Cancel
          </button>
          <button 
            form="editRentalForm" 
            type="submit" 
            class="bg-orange-500 hover:bg-orange-600 text-white font-black text-[10px] uppercase tracking-[0.2em] px-10 py-4 rounded-2xl shadow-xl shadow-orange-500/20 active:scale-95 transition-all"
          >
            Update Rental
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import Toast from "@/components/Toast.vue";

export default {
  name: "WorkspaceRentalUpdate",
  components: { Toast },
  props: {
    visible: Boolean,
    rental: Object,
  },
  data() {
    return {
      form: {
        id: null,
        guest_name: "",
        guest_email: "",
        guest_phone: "",
        cycle: "daily",
        start_date: "",
        is_active: true,
        user: 0,
        space: 0,
      },
      spaces: [], // fetch from API
    };
  },
  watch: {
    rental: {
      immediate: true,
      handler(val) {
        if (val) {
          this.form = {
            id: val.id,
            guest_name: val.guest_name || "",
            guest_email: val.guest_email || "",
            guest_phone: val.guest_phone || "",
            cycle: val.cycle || "daily",
            start_date: val.start_date || "",
            is_active: val.is_active ?? true,
            user: val.user?.id || 0,   // use ID only
            space: val.space?.id || 0, // use ID only
          };
        }
      },
    },
  },
  mounted() {
    this.fetchSpaces();
  },
  methods: {
    async fetchSpaces() {
      try {
        const res = await this.$apiGet("/get_coworking_spaces");
        this.spaces = res.data || [];
      } catch (err) {
        console.error("Failed to fetch spaces:", err);
      }
    },
    async submitForm() {
      try {
        console.log("Updating rental:", this.form);

        // send PUT with correct body
        await this.$apiPut(`/update_workspace_rental`, this.form.id,this.form);

        this.$root.$refs.toast.showToast(
          "Workspace rental updated successfully",
          "success"
        );
        this.$emit("refresh");
        setTimeout(() => this.$emit("close"), 2000);
      } catch (err) {
        console.error("Update failed:", err);
        this.$root.$refs.toast.showToast("Failed to update rental", "error");
      }
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-4 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-primary;
}
</style>
