<template>
  <div>
    <Toast ref="toast" />
    <div
      v-if="visible"
      class="fixed inset-0 z-[70] bg-black/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
    >
      <div class="bg-white w-full max-w-4xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0 shadow-md">
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center text-white backdrop-blur-md">
              <i class="fas fa-file-signature text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Execute Property Sale</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Finalize Transaction Agreement</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all">
            <i class="fas fa-times text-xs"></i>
          </button>
        </div>

        <form @submit.prevent="submitSale" class="flex-1 overflow-y-auto p-8 space-y-8">
          
          <div class="space-y-4">
            <div class="flex justify-between items-center border-b border-gray-100 pb-2">
              <h3 class="text-[10px] font-black text-primary uppercase tracking-[0.2em]">1. Buyer Information</h3>
              <button type="button" @click="toggleBuyerMode" class="text-[10px] font-black text-gray-400 hover:text-primary uppercase tracking-widest transition-colors flex items-center gap-2">
                <i :class="buyerMode === 'search' ? 'fas fa-plus-circle' : 'fas fa-search'"></i>
                {{ buyerMode === 'search' ? 'Create New Buyer' : 'Search Existing' }}
              </button>
            </div>

            <div v-if="buyerMode === 'search'" class="relative group">
              <div class="relative">
                <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-gray-400 group-focus-within:text-primary">
                  <i class="fas fa-search text-xs"></i>
                </span>
                <input
                  v-model="buyerSearch"
                  @input="fetchBuyers"
                  @focus="buyerDropdown = true"
                  class="custom-input !pl-11 h-12"
                  placeholder="Type name or phone number..."
                />
              </div>
              <ul v-if="buyerDropdown && buyers.length" class="absolute z-10 w-full mt-2 bg-white border border-gray-100 rounded-xl shadow-xl overflow-hidden animate-in slide-in-from-top-2">
                <li
                  v-for="u in buyers" :key="u.id"
                  @mousedown.prevent="selectBuyer(u)"
                  class="px-5 py-3 hover:bg-primary/5 hover:text-primary cursor-pointer flex items-center justify-between border-b last:border-0 border-gray-50 transition-colors"
                >
                  <span class="text-sm font-bold">{{ u.first_name }} {{ u.middle_name }} {{ u.last_name }}</span>
                  <span class="text-[10px] font-black text-gray-400 bg-gray-100 px-2 py-1 rounded-lg">{{ u.phone_number }}</span>
                </li>
              </ul>
            </div>

            <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-4 animate-in fade-in slide-in-from-right-4">
              <input v-model="buyer.first_name" placeholder="First Name" class="custom-input h-11" />
              <input v-model="buyer.middle_name" placeholder="Middle Name" class="custom-input h-11" />
              <input v-model="buyer.last_name" placeholder="Last Name" class="custom-input h-11" />
              <input v-model="buyer.email" placeholder="Email Address" class="custom-input h-11" />
              <input v-model="buyer.phone_number" placeholder="Phone" class="custom-input h-11" />
              <input v-model="buyer.password" type="password" placeholder="Password" class="custom-input h-11" />
            </div>
          </div>

          <div class="space-y-4">
            <div class="flex justify-between items-center border-b border-gray-100 pb-2">
              <h3 class="text-[10px] font-black text-primary uppercase tracking-[0.2em]">2. Broker Information</h3>
              <button type="button" @click="toggleBrokerMode" class="text-[10px] font-black text-gray-400 hover:text-primary uppercase tracking-widest transition-colors flex items-center gap-2">
                <i :class="brokerMode === 'search' ? 'fas fa-plus-circle' : 'fas fa-search'"></i>
                {{ brokerMode === 'search' ? 'Create New Broker' : 'Search Existing' }}
              </button>
            </div>

            <div v-if="brokerMode === 'search'" class="relative group">
              <div class="relative">
                 <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-gray-400 group-focus-within:text-primary">
                  <i class="fas fa-user-tie text-xs"></i>
                </span>
                <input
                  v-model="brokerSearch"
                  @input="fetchBrokers"
                  @focus="brokerDropdown = true"
                  class="custom-input !pl-11 h-12"
                  placeholder="Search authorized brokers..."
                />
              </div>
              <ul v-if="brokerDropdown && brokers.length" class="absolute z-10 w-full mt-2 bg-white border border-gray-100 rounded-xl shadow-xl overflow-hidden animate-in slide-in-from-top-2">
                <li
                  v-for="u in brokers" :key="u.id"
                  @mousedown.prevent="selectBroker(u)"
                  class="px-5 py-3 hover:bg-primary/5 hover:text-primary cursor-pointer border-b last:border-0 border-gray-50 transition-colors"
                >
                  <span class="text-sm font-bold">{{ u.first_name }} {{ u.middle_name }} {{ u.last_name }}</span>
                </li>
              </ul>
            </div>

            <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-4 animate-in fade-in slide-in-from-right-4">
               <input v-model="broker.first_name" placeholder="First Name" class="custom-input h-11" />
               <input v-model="broker.last_name" placeholder="Last Name" class="custom-input h-11" />
               <input v-model="broker.email" placeholder="Email Address" class="custom-input h-11" />
            </div>
          </div>

          <div class="bg-gray-50 rounded-2xl p-6 border border-gray-100 space-y-6">
            <h3 class="text-[10px] font-black text-primary uppercase tracking-[0.2em] border-b border-gray-200 pb-2">3. Payment Execution</h3>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="custom-label">Method <span class="custom-star">*</span></label>
                <select v-model="payment_method" class="custom-input h-11 appearance-none" required>
                  <option value="">Select Method</option>
                  <option value="bank_transfer">Bank Transfer</option>
                  <option value="cash">Cash</option>
                  <option value="check">Check</option>
                  <option value="mobile_money">Mobile Money</option>
                </select>
              </div>
              <div class="space-y-1">
                <label class="custom-label">Transaction Ref <span class="custom-star">*</span></label>
                <input v-model="transaction_id" class="custom-input h-11" placeholder="TID-XXXXXX" required />
              </div>
            </div>

            <div class="space-y-2">
              <label class="custom-label">Payment Verification (Slip)</label>
              <div class="relative group">
                <label class="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed border-gray-200 rounded-2xl cursor-pointer bg-white hover:bg-primary/5 hover:border-primary transition-all">
                  <div class="flex flex-col items-center justify-center pt-5 pb-6">
                    <i class="fas fa-file-invoice text-gray-300 group-hover:text-primary mb-2 text-xl"></i>
                    <p class="text-[10px] font-black text-gray-500 uppercase tracking-widest">Upload Receipt Image</p>
                  </div>
                  <input type="file" class="hidden" accept="image/*" @change="e => slip_picture = e.target.files[0]" required />
                </label>
              </div>
            </div>
          </div>
        </form>

        <div class="px-8 py-5 bg-gray-50 border-t border-gray-100 flex items-center justify-end gap-3 shrink-0">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase tracking-widest transition-colors"
          >
            Abort
          </button>
          <button 
            @click="submitSale"
            type="submit" 
            :disabled="loading"
            class="custom-button !w-auto !px-10 flex items-center gap-2 shadow-lg shadow-primary/20"
          >
            <i v-if="loading" class="fas fa-circle-notch animate-spin"></i>
            <i v-else class="fas fa-check-double"></i>
            {{ loading ? 'Securing...' : 'Complete Asset Sale' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: { visible: Boolean, listing: Object },
  data() {
    return {
      loading: false,
      // File Upload
      slip_picture: null,
      // Buyer / Broker Fields
      buyerMode: "search",
      brokerMode: "search",
      buyers: [],
      buyerSearch: "",
      buyerDropdown: false,
      selectedBuyer: null,
      
      brokers: [],
      brokerSearch: "",
      brokerDropdown: false,
      selectedBroker: null,

      buyer: {
        first_name: "",
        middle_name: "",
        last_name: "",
        email: "",
        phone_number: "",
        address: "",
        password: "",
        plan: 1,
        start_date:new Date()
      },

      broker: {
        first_name: "",
        middle_name: "",
        last_name: "",
        email: "",
        phone_number: "",
        address: "",
        password: "",
        start_date:new Date()
      },

      payment_method: "",
      transaction_id: "",
    };
  },

  methods: {
    toggleBuyerMode() {
      this.buyerMode = this.buyerMode === "search" ? "add" : "search";
      this.selectedBuyer = null;
    },
    toggleBrokerMode() {
      this.brokerMode = this.brokerMode === "search" ? "add" : "search";
      this.selectedBroker = null;
    },

    async fetchBuyers() {
      const res = await this.$apiGet("/get_users", {
        search: this.buyerSearch,
        is_owner: true,
      });
      this.buyers = res.data || [];
    },

    selectBuyer(u) {
      this.selectedBuyer = u;
      this.buyerSearch = `${u.first_name} ${u.last_name}`;
      this.buyerDropdown = false;
    },

    async fetchBrokers() {
      const res = await this.$apiGet("/get_users", {
        search: this.brokerSearch,
        is_broker: true,
      });
      this.brokers = res.data || [];
    },


    selectBroker(u) {
      this.selectedBroker = u;
      this.brokerSearch = `${u.first_name} ${u.last_name}`;
      this.brokerDropdown = false;
    },


    async submitSale() {
      if (!this.slip_picture) {
        alert("Please upload slip picture");
        return;
      }

      this.loading = true;

      try {
        const buyerId =
          this.buyerMode === "search"
            ? this.selectedBuyer.id
            : (await this.$apiPost("/sign_up", {
                ...this.buyer,
                is_owner: true,
              })).id;

        const brokerId =
          this.brokerMode === "search"
            ? this.selectedBroker.id
            : (await this.$apiPost("/sign_up", {
                ...this.broker,
                is_broker: true,
              })).id;
        // Build multipart form data
        const fd = new FormData();
        fd.append("buyer_id", buyerId);
        fd.append("broker_id", brokerId);
        fd.append("selling_price", this.listing.listing_price);
        fd.append("property_zone_sale_id", this.listing.property_zone_id?.id);
        fd.append("payment_method", this.payment_method);
        fd.append("transaction_id", this.transaction_id);
        fd.append("status", "pending");
        fd.append("slip_picture", this.slip_picture);
        // Submit using form data
        await this.$apiPost("/sell_property", fd, true); 
        // (true) optional depending on your api wrapper to force multipart
        this.$emit("success");
        this.$emit("close");
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-3 py-2 border rounded;
}
.dropdown {
  @apply absolute bg-white border w-full max-h-40 overflow-y-auto z-50;
}
.dropdown li {
  @apply px-3 py-2 hover:bg-gray-100 cursor-pointer;
}
</style>
