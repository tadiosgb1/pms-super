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
              <i class="fas fa-key text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Rent Commission</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Lease Allocation</p>
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
          <form id="rentCommissionForm" @submit.prevent="submitForm" class="space-y-6">
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="custom-label">SaaS Commission (%) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-percentage text-xs"></i>
                  </span>
                  <input
                    v-model="form.saas_commission"
                    type="number"
                    step="0.01"
                    placeholder="0.00"
                    required
                    class="custom-input !mt-0 !pl-10 h-11"
                  />
                </div>
              </div>

              <div class="space-y-1">
                <label class="custom-label">Broker Commission (%) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-user-tie text-xs"></i>
                  </span>
                  <input
                    v-model="form.broker_commission"
                    type="number"
                    step="0.01"
                    placeholder="0.00"
                    required
                    class="custom-input !mt-0 !pl-10 h-11"
                  />
                </div>
              </div>

              <div class="space-y-1 md:col-span-2">
                <label class="custom-label">Total Commission (%) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-calculator text-xs"></i>
                  </span>
                  <input
                    v-model="form.total_commission"
                    type="number"
                    step="0.01"
                    placeholder="Combined total"
                    required
                    class="custom-input !mt-0 !pl-10 h-11 bg-gray-50/50"
                  />
                </div>
              </div>

              <div class="space-y-1 md:col-span-2 relative">
                <label class="custom-label">Rent Reference / ID <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-search text-xs"></i>
                  </span>
                  <input
                    v-model="form.rent"
                    type="text"
                    placeholder="Search by Rent ID or Tenant..."
                    required
                    class="custom-input !mt-0 !pl-10 h-11"
                  />
                </div>

                <ul
                  v-if="propertyRents && propertyRents.length && showSuggestions"
                  class="absolute z-20 w-full bg-white border border-gray-200 rounded-xl shadow-xl mt-1 max-h-48 overflow-y-auto modal-scroll"
                >
                  <li
                    v-for="rent in propertyRents"
                    :key="rent.id"
                    @click="selectPropertyRent(rent)"
                    class="px-4 py-3 hover:bg-primary/5 cursor-pointer flex justify-between items-center group transition-colors border-b border-gray-50 last:border-0"
                  >
                    <div>
                      <span class="text-sm font-bold text-gray-700 group-hover:text-primary tracking-tight">#{{ rent.id }}</span>
                      <p v-if="rent.tenant_name" class="text-[10px] text-gray-400 italic">{{ rent.tenant_name }}</p>
                    </div>
                    <i class="fas fa-plus text-[10px] text-gray-300 group-hover:text-primary"></i>
                  </li>
                </ul>
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
            Cancel
          </button>
          <button
            form="rentCommissionForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-save"></i>
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
        total_commission: "",
        rent: 0, // ID of selected property
      },
      searchTerm: "",
      propertyRents: [],
      showSuggestions: false,
      searchTimeout: null,
    };
  },

  methods: {
    async searchPropertyRents() {
      this.showSuggestions = true;

      // debounce API calls
      clearTimeout(this.searchTimeout);
      this.searchTimeout = setTimeout(async () => {
        if (!this.searchTerm) {
          this.propertyRents = [];
          return;
        }

        try {
          const res = await this.$apiGet(
            `/get_rents?search=${encodeURIComponent(this.searchTerm)}`
          );
          console.log("search res", res);
          this.propertyRents = res.data || [];
        } catch (err) {
          console.error("Failed to fetch property rents:", err);
        }
      }, 400); // wait 400ms after typing
    },

    selectPropertyRent(rent) {
      this.form.rent = rent.id; // store ID
      this.searchTerm = rent.id; // show property ID in input (you can change to rent.address)
      this.propertyRents = [];
      this.showSuggestions = false;
    },

    async submitForm() {
      try {
        const payload = { ...this.form };
        const res = await this.$apiPost("/post_rent_commission", payload);
        console.log("Commission added:", res);
        this.$emit("success");
        this.resetForm();
        this.$emit("close");
      } catch (err) {
        console.error("Failed to add commission:", err);
        alert("Failed to add commission.");
      }
    },

    resetForm() {
      this.form = {
        saas_commission: "",
        broker_commission: "",
        total_commission: "",
        rent: 0,
      };
      this.searchTerm = "";
      this.propertyRents = [];
      this.showSuggestions = false;
    },
  },
};
</script>
