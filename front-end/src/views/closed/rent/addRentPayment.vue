<template>
  <div>
    <Toast ref="toast" />
    <div
      v-if="visible"
      class="fixed inset-0 z-[80] bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
     >
      <div class="bg-white w-full max-w-[850px] max-h-[90vh] rounded-[2.5rem] shadow-2xl flex flex-col relative animate-in zoom-in duration-200 border border-gray-100 overflow-hidden">
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center text-white border border-white/20 backdrop-blur-md">
              <i class="fas fa-money-bill-wave text-xl"></i>
            </div>
            <div>
              <h2 class="text-xl font-black text-white tracking-tight uppercase leading-none">Pay Rent</h2>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Settlement Processing</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-10 h-10 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all border border-white/10">
            <i class="fas fa-times"></i>
          </button>
        </div>
        <form
          @submit.prevent="submitPayment"
          class="flex-1 overflow-y-auto p-8 space-y-8 scrollbar-thin scrollbar-thumb-gray-200 scrollbar-track-transparent"
          enctype="multipart/form-data"
        >
          <input type="hidden" v-model="form.rent_id" />
          <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
            <div class="space-y-6">
              <div class="space-y-3">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Select Billing Cycles</label>
                <div class="space-y-2 max-h-56 overflow-y-auto border border-gray-100 rounded-[1.5rem] p-4 bg-gray-50/50 scrollbar-thin">
                  <div
                    v-for="cycle in cycles"
                    :key="cycle.id"
                    class="group flex items-center space-x-3 bg-white p-3 rounded-xl border border-transparent hover:border-primary/30 transition-all cursor-pointer shadow-sm"
                  >
                    <div class="relative flex items-center">
                      <input
                        type="checkbox"
                        :value="cycle.id"
                        v-model="form.cycle_ids"
                        class="peer h-5 w-5 rounded border-gray-300 text-primary focus:ring-primary/20 transition-all cursor-pointer"
                      />
                    </div>
                    <span class="text-xs font-bold text-gray-600 group-hover:text-primary transition-colors">
                      {{ cycle.cycle_start }} <i class="fas fa-arrow-right mx-1 opacity-30"></i> {{ cycle.cycle_end }}
                    </span>
                  </div>
                  <div v-if="!cycles.length" class="text-center py-4 text-xs font-bold text-gray-300 italic uppercase">
                    No pending cycles
                  </div>
                </div>
                <p class="text-[9px] font-bold text-gray-400 uppercase tracking-tighter ml-1">
                  <i class="fas fa-info-circle mr-1"></i> Multi-cycle selection supported
                </p>
              </div>

              <div class="space-y-2">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Payment Channel</label>
                <div class="relative">
                  <select
                    v-model="form.payment_method"
                    class="w-full pl-4 pr-10 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none appearance-none cursor-pointer transition-all"
                    required
                  >
                    <option value="">Select method</option>
                    <option value="cash">Cash</option>
                    <option value="bank">Bank Transfer</option>
                    <option value="cbe">CBE</option>
                    <option value="tellbirr">Tele Birr</option>
                  </select>
                  <i class="fas fa-chevron-down absolute right-4 top-1/2 -translate-y-1/2 text-gray-300 pointer-events-none text-xs"></i>
                </div>
              </div>
            </div>

            <div class="space-y-6">
              <div class="space-y-2">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Reference / Transaction ID</label>
                <input
                  v-model="form.transaction_id"
                  type="text"
                  class="w-full px-5 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all shadow-sm"
                  placeholder="TXN-000000"
                  required
                />
              </div>

              <div class="space-y-2">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Proof of Payment (Slip)</label>
                <label
                  for="slip-upload"
                  class="flex flex-col items-center justify-center w-full h-44 border-2 border-dashed border-gray-200 rounded-[2rem] cursor-pointer bg-gray-50/50 hover:bg-secondary/5 hover:border-secondary transition-all group overflow-hidden relative"
                >
                  <div v-if="!form.preview" class="flex flex-col items-center justify-center pt-5 pb-6">
                    <div class="w-12 h-12 bg-white rounded-xl shadow-sm flex items-center justify-center text-gray-300 group-hover:text-secondary transition-colors mb-3">
                      <i class="fas fa-receipt text-lg"></i>
                    </div>
                    <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest group-hover:text-secondary transition-colors">Attach Receipt</p>
                  </div>
                  
                  <img v-else :src="form.preview" class="absolute inset-0 w-full h-full object-cover transition-transform group-hover:scale-105" />
                  
                  <input
                    id="slip-upload"
                    type="file"
                    accept="image/*"
                    @change="handleFileUpload"
                    class="hidden"
                    required
                  />
                </label>
              </div>
            </div>
          </div>
        </form>

        <div class="p-8 border-t border-gray-50 flex items-center justify-end gap-5 shrink-0 bg-white rounded-b-[2.5rem]">
          <button
            type="button"
            @click="$emit('close')"
            class="px-8 py-4 rounded-2xl text-[10px] font-black uppercase tracking-[0.2em] text-gray-400 hover:text-rose-500 hover:bg-rose-50 transition-all"
          >
            Cancel
          </button>

          <button
            @click="submitPayment"
            :disabled="loading"
            class="bg-primary hover:bg-dprimary disabled:bg-gray-200 text-white font-black text-[10px] uppercase tracking-[0.2em] px-12 py-4 rounded-2xl shadow-xl shadow-primary/30 active:scale-95 transition-all flex items-center gap-3"
          >
            <i v-if="loading" class="fas fa-circle-notch animate-spin"></i>
            {{ loading ? "Processing..." : "Submit Payment" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "../../../components/Toast.vue";
export default {
  name: "MakePaymentModal",
  components: { Toast },
  props: {
    rentId: {
      type: Number,
      required: true,
    },
    visible: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      form: {
        rent_id: this.rentId,
        cycle_ids: [],
        payment_method: "",
        transaction_id: "",
        slip_picture: null,
      },
      cycles: [],
      loading: false,
    };
  },

  watch: {
    rentId: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.fetchCycles(newVal);
        }
      },
    },
  },

  methods: {
    handleFileUpload(event) {
      this.form.slip_picture = event.target.files[0];
    },

    async fetchCycles(rentId) {
      try {
        const response = await this.$apiGetById("get_rent_cycles", rentId);
        this.cycles = response.data || [];
      } catch (error) {
        console.error("Failed to fetch cycles:", error);
        this.cycles = [];
      }
    },

    async submitPayment() {

      if (!this.form.cycle_ids.length) {
         this.$root.$refs.toast.showToast('Please select at least one cycle',"error")
        return;
      }

      if (!this.form.slip_picture) {
       // this.$refs.toast.showToast("Slip picture is required", "error");
        this.$root.$refs.toast.showToast('Slip picture is required',"error")
        return;
      }

      this.loading = true;
      try {
        const fd = new FormData();
        fd.append("rent_id", this.form.rent_id);
        fd.append("payment_method", this.form.payment_method);
        fd.append("transaction_id", this.form.transaction_id);
        fd.append("slip_picture", this.form.slip_picture);
        this.form.cycle_ids.forEach((id) => {
          fd.append("cycle_ids", id);
        });
        const response = await this.$apiPost("make_payment", fd);
        this.$root.$refs.toast.showToast("Rent paid successfully", "success");
        this.$emit("success", response);
        this.$emit("close");
      } catch (error) {
        console.error("Failed to make payment:", error);
        this.$root.$refs.toast.showToast("Failed to submit payment", "error");
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
.animate-spin {
  animation: spin 1s linear infinite;
}
</style>
