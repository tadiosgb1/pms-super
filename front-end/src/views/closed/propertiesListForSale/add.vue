<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-[70] bg-gray-900/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
    >
      <div class="bg-white w-full max-w-lg rounded-3xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0 gap-8">
          <div class="flex items-center gap-3 text-white">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center backdrop-blur-md">
              <i class="fas fa-tag text-lg"></i>
            </div>
            <div>
              <h2 class="text-lg font-black tracking-tight leading-none">{{ modalTitle }}</h2>
              <p class="text-white/60 text-[10px] uppercase tracking-widest font-bold mt-1">Pricing & Valuation</p>
            </div>
          </div>
          <button 
            @click="$emit('close')" 
            class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all shrink-0"
          >
            <i class="fas fa-times text-xs"></i>
          </button>
        </div>

        <form @submit.prevent="submitSale" class="flex-1 overflow-y-auto p-8 flex flex-col">
          
          <div class="space-y-4 flex-1">
            <div class="space-y-1">
              <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Listing Price</label>
              <div class="relative group">
                <span class="absolute left-4 top-3.5 text-gray-400 text-xs font-bold group-focus-within:text-primary transition-colors">$</span>
                <input
                  v-model="listingPrice"
                  type="number"
                  placeholder="0.00"
                  class="w-full pl-8 pr-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-sm font-bold focus:ring-2 focus:ring-primary/10 focus:border-primary outline-none transition-all placeholder:text-gray-300"
                  required
                />
              </div>
              <p class="text-[9px] text-gray-400 font-medium mt-2 ml-1">Set the market entry price for this asset listing.</p>
            </div>
          </div>

          <div class="sticky bottom-0 bg-white pt-8 pb-2 mt-8 border-t border-gray-50 flex items-center gap-4 shrink-0">
            <button
              type="button"
              @click="$emit('close')"
              class="flex-1 px-6 py-4 rounded-2xl text-[11px] font-black uppercase tracking-widest text-gray-500 bg-gray-50 hover:bg-gray-100 transition-all"
            >
              Cancel
            </button>
            <button
              type="submit"
              :disabled="loading"
              class="flex-[2] bg-primary hover:bg-orange-600 disabled:bg-gray-300 text-white font-black text-[11px] uppercase tracking-widest px-8 py-4 rounded-2xl shadow-xl shadow-primary/20 transform hover:-translate-y-0.5 active:translate-y-0 transition-all flex items-center justify-center gap-2"
            >
              <i v-if="loading" class="fas fa-circle-notch animate-spin"></i>
              <span v-if="!loading">{{ loading ? 'Submitting...' : 'Confirm Submission' }}</span>
              <span v-else>Processing</span>
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "../../../components/Toast.vue";

export default {
  name: "SaleModal",
  components: { Toast },
  props: {
    visible: Boolean,
    sourceType: {
      type: String,
      required: true, // 'property' or 'zone'
    },
    propertyId: {
      type: Number,
      default: null,
    },
    propertyZoneId: {
      type: Number,
      default: null,
    },
  },
  data() {
    return {
      listingPrice: "",
      loading: false,
    };
  },
  computed: {
    modalTitle() {
      return this.sourceType === "zone"
        ? "Add Property Zone for Sale"
        : "Add Property for Sale";
    },
  },
  methods: {
    async submitSale() {
      try {
        this.loading = true;

        const payload = {
          listing_price: this.listingPrice,
        };

        console.log("payload",payload);
        // Dynamically include ID based on source
        if (this.sourceType === "zone") {
          payload.property_zone_id = this.propertyZoneId;
        } else {
          payload.property_id = this.propertyId;
        }

        const res = await this.$apiPost("/create_property_sale_listing", payload);


        
        if (res && res.error) {
          this.$root.$refs.toast.showToast(res.error, "error");
        } else {
          this.$root.$refs.toast.showToast(
            `${this.modalTitle} successfully!`,
            "success"
          );
          this.$emit("close");
          this.$emit("refresh");
        }
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast("Failed to create sale listing", "error");
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-indigo-500;
}
</style>
