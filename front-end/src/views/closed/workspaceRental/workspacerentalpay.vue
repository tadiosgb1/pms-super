<template>
  <div>
    <transition name="fade">
      <div v-if="visible" class="fixed inset-0 z-[100] bg-black/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300">
        <div class="bg-white w-full max-w-2xl max-h-[95vh] rounded-2xl shadow-2xl flex flex-col overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
          
          <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
            <div class="flex items-center gap-3">
              <div class="p-2 bg-white/20 rounded-lg">
                <i class="fas fa-wallet text-white text-lg"></i>
              </div>
              <div>
                <h2 class="text-xl font-bold text-white tracking-tight">Rental Payment</h2>
                <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Settle Workspace Balance</p>
              </div>
            </div>
            <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-colors">
              <i class="fas fa-times"></i>
            </button>
          </div>

          <div class="flex bg-gray-50 p-1.5 gap-1.5 shrink-0 border-b border-gray-100">
            <button
              class="flex-1 py-2.5 text-[10px] font-black uppercase tracking-widest transition-all rounded-xl flex items-center justify-center gap-2"
              :class="tab === 'manual' ? 'bg-white text-primary shadow-sm' : 'text-gray-400 hover:text-gray-600'"
              @click="tab = 'manual'"
            >
              <i class="fas fa-hand-holding-usd"></i> Manual Entry
            </button>
            <button
              class="flex-1 py-2.5 text-[10px] font-black uppercase tracking-widest transition-all rounded-xl flex items-center justify-center gap-2"
              :class="tab === 'telebirr' ? 'bg-white text-blue-600 shadow-sm' : 'text-gray-400 hover:text-gray-600'"
              @click="tab = 'telebirr'"
            >
              <i class="fas fa-mobile-alt"></i> Telebirr Checkout
            </button>
          </div>

          <div class="flex-1 overflow-y-auto p-6 md:p-8">
            <form v-if="tab === 'manual'" id="manualPaymentForm" @submit.prevent="submitPayment" class="space-y-6">
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="space-y-1">
                  <label class="custom-label">Amount (ETB) <span class="custom-star">*</span></label>
                  <input v-model="form.amount" type="number" step="0.01" class="custom-input !mt-0 h-11" required />
                </div>

                <div class="space-y-1">
                  <label class="custom-label">Method <span class="custom-star">*</span></label>
                  <select v-model="form.payment_method" class="custom-input !mt-0 h-11 cursor-pointer" required>
                    <option value="">Select Method</option>
                    <option value="cash">Cash</option>
                    <option value="bank">Bank Transfer</option>
                    <option value="telebirr">Telebirr (Manual)</option>
                  </select>
                </div>

                <div class="md:col-span-2 space-y-1">
                  <label class="custom-label">Transaction ID / Reference <span class="custom-star">*</span></label>
                  <input v-model="form.transaction_id" type="text" placeholder="e.g. TXN12345678" class="custom-input !mt-0 h-11" required />
                </div>

                <div class="md:col-span-2 space-y-1">
                  <label class="custom-label">Cycle End Date <span class="custom-star">*</span></label>
                  <input v-model="form.cycle_end" type="date" class="custom-input !mt-0 h-11" required />
                </div>

                <div class="md:col-span-2 space-y-2">
                  <label class="custom-label">Payment Slip / Receipt</label>
                  <div 
                    class="relative border-2 border-dashed border-gray-200 rounded-2xl p-6 transition-all hover:border-primary/50 group bg-gray-50/50 flex flex-col items-center justify-center cursor-pointer"
                    @click="$refs.fileInput.click()"
                  >
                    <input 
                      type="file" 
                      ref="fileInput" 
                      class="hidden" 
                      accept="image/*" 
                      @change="handleFileUpload"
                    />
                    
                    <div v-if="!fileName" class="text-center">
                      <div class="w-10 h-10 bg-white rounded-full flex items-center justify-center shadow-sm mx-auto mb-2 text-gray-400 group-hover:text-primary transition-colors">
                        <i class="fas fa-cloud-upload-alt"></i>
                      </div>
                      <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Click to upload slip</p>
                    </div>

                    <div v-else class="flex items-center gap-3 p-2 bg-white rounded-xl border border-primary/20 animate-in fade-in zoom-in-95">
                      <i class="fas fa-image text-primary"></i>
                      <span class="text-xs font-bold text-gray-700 truncate max-w-[250px]">{{ fileName }}</span>
                      <i class="fas fa-check-circle text-green-500 text-[10px]"></i>
                    </div>
                  </div>
                </div>
              </div>
            </form>

            <div v-if="tab === 'telebirr'" class="flex flex-col items-center justify-center py-8 space-y-8 animate-in fade-in slide-in-from-bottom-4">
              <div class="w-20 h-20 bg-blue-50 rounded-3xl flex items-center justify-center border border-blue-100 shadow-inner">
                <img src="https://teller.telebirr.com.et/teller/img/telebirr_logo.svg" alt="Telebirr" class="w-12 h-12" />
              </div>
              <div class="text-center max-w-xs">
                <h2 class="text-lg font-black text-gray-800 uppercase tracking-tight">Telebirr Direct</h2>
                <p class="text-[11px] font-medium text-gray-400 mt-1 leading-relaxed">Fast, secure checkout. Your payment will be verified automatically.</p>
              </div>
              <div class="w-full max-w-sm space-y-2">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest text-center">Amount to Pay (ETB)</label>
                <input v-model="form.amount" type="number" class="w-full px-6 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-center text-2xl font-black text-primary focus:bg-white focus:border-blue-500 outline-none shadow-sm transition-all" required />
              </div>
              <button @click="startTelebirrCheckout" :disabled="isSaving" class="w-full max-w-sm py-4 bg-[#0066b3] hover:bg-[#005596] text-white rounded-2xl shadow-xl flex items-center justify-center gap-3 active:scale-95 transition-all">
                <span class="text-[11px] font-black uppercase tracking-[0.2em]">Launch Gateway</span>
                <i class="fas fa-external-link-alt text-[10px]"></i>
              </button>
            </div>
          </div>

          <div class="px-8 py-5 bg-gray-50 border-t border-gray-100 flex items-center justify-end gap-3 shrink-0">
            <button
              @click="$emit('close')"
              class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase tracking-widest transition-colors"
            >
              Cancel
            </button>
            <button 
              v-if="tab === 'manual'" 
              form="manualPaymentForm" 
              type="submit" 
              :disabled="isSaving"
              class="custom-button !w-auto !px-10 flex items-center gap-2"
            >
              <i v-if="isSaving" class="fas fa-spinner fa-spin"></i>
              <i v-else class="fas fa-check-double"></i>
              {{ isSaving ? 'Processing...' : 'Submit Payment' }}
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "WorkspaceRentalPay",
  props: {
    rentalId: { type: Number, required: true },
    visible: { type: Boolean, required: true },
  },
  data() {
    return {
      tab: "manual",
      isSaving: false,
      fileName: "",
      slipFile: null,
      form: {
        amount: "",
        payment_method: "",
        transaction_id: "",
        cycle_end: "",
        rental: this.rentalId, // Changed key to match backend expectations
        status:"pending",
      },
    };
  },
  watch: {
    rentalId: {
      immediate: true,
      handler(newVal) { if (newVal) this.form.rental_id = newVal; },
    },
    visible(val) {
      if (!val) this.resetForm();
    }
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        this.fileName = file.name;
        this.slipFile = file;
      }
    },
    async submitPayment() {
      if (this.isSaving) return;
      this.isSaving = true;

      try {
        const formData = new FormData();
        
        // Append all text fields from the form object
        Object.keys(this.form).forEach(key => {
          formData.append(key, this.form[key]);
        });
        
        // Key Fix: Append the actual file object under the correct key
        if (this.slipFile) {
          formData.append("slip_picture", this.slipFile);
        }


        const headers={
             "Content-Type": "multipart/form-data" 
        }

        await this.$apiPost("/post_rental_payment", formData, headers);

        this.$root.$refs.toast.showToast("Payment recorded successfully", "success");
        this.$emit("success"); // Inform parent to refresh data
        this.$emit("close");
      } catch (error) {
        console.error(error);
        this.$root.$refs.toast.showToast("Failed to submit payment", "error");
      } finally {
        this.isSaving = false;
      }
    },
    async startTelebirrCheckout() {
      if (!this.form.amount) {
        this.$root.$refs.toast.showToast("Please enter an amount", "warning");
        return;
      }
      this.isSaving = true;
      try {
        const response = await this.$apiPost("/telebirr_checkout", {
          amount: this.form.amount,
          rental: this.form.rental_id,
        });
        if (response.checkout_url) {
          window.open(response.checkout_url, "_blank");
          this.$emit("close");
        }
      } catch (error) {
        this.$root.$refs.toast.showToast("Checkout failed", "error");
      } finally {
        this.isSaving = false;
      }
    },
    resetForm() {
      this.form = {
        amount: "",
        payment_method: "",
        transaction_id: "",
        cycle_end: "",
        rental_id: this.rentalId,
      };
      this.fileName = "";
      this.slipFile = null;
      if (this.$refs.fileInput) this.$refs.fileInput.value = "";
    }
  },
};
</script>