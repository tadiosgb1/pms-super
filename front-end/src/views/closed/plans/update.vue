<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-2xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100"
      >
        <div class="bg-gradient-to-r from-blue-700 to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg">
              <i class="fas fa-edit text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Edit Plan </h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Modifying: {{ form.name }}</p>
            </div>
          </div>
          <button 
            @click="$emit('close')" 
            class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-colors"
          >
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-6 md:p-8 modal-scroll">
          <form id="editPlanForm" @submit.prevent="submitForm" class="space-y-6">
            
            <div class="space-y-1">
              <label class="custom-label">Plan Name <span class="custom-star">*</span></label>
              <div class="relative group">
                <input 
                  v-model="form.name" 
                  type="text" 
                  required
                  class="custom-input !mt-0 h-11" 
                />
              </div>
            </div>

            <div class="bg-blue-50/30 rounded-xl p-5 border border-blue-100/50">
              <p class="text-[10px] font-black text-blue-400 uppercase tracking-widest mb-4 flex items-center gap-2">
                <i class="fas fa-sliders-h"></i> Capacity Adjustments
              </p>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div class="space-y-1">
                  <label class="text-[11px] font-bold text-gray-600">Max Locations</label>
                  <div class="relative group">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                      <i class="fas fa-map-marker-alt text-[10px]"></i>
                    </span>
                    <input v-model="form.max_locations" type="number" required class="custom-input !mt-0 !pl-9 h-10 shadow-sm" />
                  </div>
                </div>

                <div class="space-y-1">
                  <label class="text-[11px] font-bold text-gray-600">Max Staff</label>
                  <div class="relative group">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                      <i class="fas fa-users-cog text-[10px]"></i>
                    </span>
                    <input v-model="form.max_staff" type="number" required class="custom-input !mt-0 !pl-9 h-10 shadow-sm" />
                  </div>
                </div>

                <div class="space-y-1">
                  <label class="text-[11px] font-bold text-gray-600">Max Users</label>
                  <div class="relative group">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                      <i class="fas fa-user-shield text-[10px]"></i>
                    </span>
                    <input v-model="form.max_users" type="number" required class="custom-input !mt-0 !pl-9 h-10 shadow-sm" />
                  </div>
                </div>
              </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 pt-2">
              <div class="space-y-1">
                <label class="custom-label">Updated Price <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-dollar-sign text-xs"></i>
                  </span>
                  <input v-model="form.price" type="number" step="0.01" required class="custom-input !mt-0 !pl-8 h-11" />
                </div>
              </div>

              <div class="space-y-1">
                <label class="custom-label">Billing Cycle <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary pointer-events-none">
                    <i class="fas fa-sync-alt text-xs"></i>
                  </span>
                  <select v-model="form.billing_cycle" required class="custom-input !mt-0 !pl-10 h-11 appearance-none">
                    <option value="daily">Daily</option>
                    <option value="monthly">Monthly</option>
                    <option value="yearly">Yearly</option>
                  </select>
                </div>
              </div>
            </div>
          </form>
        </div>

        <div class="px-8 py-5 bg-gray-50 border-t border-gray-100 flex items-center justify-end gap-3 shrink-0">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase tracking-widest transition-colors"
          >
            Discard
          </button>
          <button
            form="editPlanForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2 shadow-lg hover:shadow-primary/30 transition-all"
          >
            <i class="fas fa-check-circle"></i>
            Update Plan
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from '@/components/Toast.vue';

export default {
  name: 'UpdatePlan',
  components: { Toast },
  props: {
    visible: Boolean,
    plan: Object,
  },
  data() {
    return {
      form: {
        id: null,
        name: '',
        max_locations: '',
        max_staff: '',
        max_users: '',
        max_kds: '27',
        kds_enabled: false,
        price: null,
        billing_cycle: '',
        updated_at: new Date().toISOString(),
      },
    };
  },
  watch: {
    plan: {
      immediate: true,
      handler(val) {
        if (val) {
          this.form = {
            ...val,
            created_at: val.created_at ? val.created_at.split('T')[0] : '',
            updated_at: val.updated_at ? val.updated_at.split('T')[0] : '',
          };
        }
      },
    },
  },
  methods: {
    async submitForm() {
      try {
        await this.$apiPut(`/update_plan`, this.form.id, this.form);
        this.$root.$refs.toast.showToast('Plan updated successfully', 'success');

        setTimeout(() => {
          this.$emit('close');
        }, 3000);

        this.$emit('refresh');
      } catch (err) {
        console.error('Update failed:', err);
         this.$root.$refs.toast.showToast(err.message,"error")
        //alert('Failed to update plan.');
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
