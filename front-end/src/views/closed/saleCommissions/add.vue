<template>
  <div>
    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-2xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100"
      >
        <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg">
              <i class="fas fa-percentage text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Sale Commission</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Financial Configuration</p>
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
          <form id="commissionForm" @submit.prevent="submitForm" class="space-y-6">
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="custom-label">SaaS Commission (%) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-cloud"></i>
                  </span>
                  <input
                    v-model="form.saas_commission"
                    type="number"
                    step="0.01"
                    placeholder="0.00"
                    required
                    class="custom-input !mt-0 !pl-10"
                  />
                </div>
              </div>

              <div class="space-y-1">
                <label class="custom-label">Broker Commission (%) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-user-tie"></i>
                  </span>
                  <input
                    v-model="form.broker_commission"
                    type="number"
                    step="0.01"
                    placeholder="0.00"
                    required
                    class="custom-input !mt-0 !pl-10"
                  />
                </div>
              </div>
            </div>

            <div class="space-y-1 relative">
              <label class="custom-label">Property Sale Code <span class="custom-star">*</span></label>
              <div class="relative group">
                <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                  <i class="fas fa-home"></i>
                </span>
                <input
                  v-model="form.property_sale"
                  type="text"
                  placeholder="Type to search property code..."
                  required
                  class="custom-input !mt-0 !pl-10"
                />
              </div>

              <ul
                v-if="propertySales && propertySales.length > 0"
                class="absolute z-20 w-full bg-white border border-gray-200 rounded-xl shadow-xl mt-1 max-h-48 overflow-y-auto modal-scroll"
              >
                <li
                  v-for="sale in propertySales"
                  :key="sale.id"
                  @click="selectPropertySale(sale)"
                  class="px-4 py-3 hover:bg-primary/5 cursor-pointer flex justify-between items-center group transition-colors border-b border-gray-50 last:border-0"
                >
                  <span class="text-sm font-semibold text-gray-700 group-hover:text-primary">{{ sale.id }}</span>
                  <i class="fas fa-chevron-right text-[10px] text-gray-300 group-hover:text-primary"></i>
                </li>
              </ul>
            </div>

            <div class="p-4 bg-blue-50 rounded-xl border border-blue-100 flex gap-3">
              <i class="fas fa-info-circle text-primary mt-0.5"></i>
              <p class="text-xs text-blue-800 leading-relaxed">
                Ensure commission percentages are entered as numerical values. These will be applied to the total sale value of the selected property.
              </p>
            </div>
          </form>
        </div>

        <div class="px-8 py-5 bg-gray-50 border-t border-gray-100 flex items-center justify-end gap-3 shrink-0">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase tracking-widest transition-colors"
          >
            Cancel
          </button>
          <button
            form="commissionForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-check-circle"></i>
            Save Commission
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "AddCommission",
  props: {
    visible: Boolean,
  },
  data() {
    return {
      form: {
        saas_commission: "",
        broker_commission: "",
        total_commission:0,
        property_sale: null, // ID of selected property
      },
      searchTerm: "",
      propertySales: [],
      showSuggestions: false,
      searchTimeout: null,
    };
  },

  methods: {
    async searchPropertySales() {
      this.showSuggestions = true;

      // debounce API calls
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(async () => {
        if (!this.searchTerm) {
          this.propertySales = [];
          return;
        }

        try {
          const res = await this.$apiGet(
            `/get_property_zone_sales?search=${encodeURIComponent(this.searchTerm)}`
          );
          console.log("search res",res);
          this.propertySales = res.data || [];
        } catch (err) {
          console.error("Failed to fetch property sales:", err);
        }
      }, 400); // wait 400ms after typing
    },

    selectPropertySale(sale) {
      this.form.property_sale = sale.id; // store ID
      this.searchTerm = sale.name || sale.code; // show name in input
      this.propertySales = [];
      this.showSuggestions = false;
    },

    async submitForm() {

 
      try {
        const payload = { ...this.form };
        const res = await this.$apiPost("/post_commission", payload);
        console.log("Commission added:", res);
        this.$emit("success");
        this.resetForm();
        this.$emit("close");
      } catch (err) {
        console.error("Failed to add commission:", err);
            this.$root.$refs.toast.showToast(err.message, 'error');
        //alert("Failed to add commission.");
      }
    },

    resetForm() {
      this.form = {
        saas_commission: "",
        broker_commission: "",
        total_commission: "",
        property_sale: 0,
      };
      this.searchTerm = "";
      this.propertySales = [];
      this.showSuggestions = false;
    },
  },
};
</script>
