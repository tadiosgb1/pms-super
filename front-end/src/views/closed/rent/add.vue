<template>
  <div>
    <Toast ref="toast" />
    <div
      v-if="visible"
      class="fixed inset-0 z-[50] bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
    >
      <div class="bg-white w-full max-w-[950px] rounded-[2.5rem] shadow-2xl flex flex-col max-h-[92vh] overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 bg-white/20 rounded-2xl flex items-center justify-center text-white border border-white/20 backdrop-blur-md">
              <i class="fas fa-file-contract text-xl"></i>
            </div>
            <div>
              <h2 class="text-xl font-black text-white tracking-tight uppercase">Add New Rent</h2>
              <p class="text-white/60 text-[10px] uppercase tracking-[0.2em] font-bold mt-1">Contract Configuration</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-10 h-10 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all border border-white/10">
            <i class="fas fa-times"></i>
          </button>
        </div>

        <form @submit.prevent="handleSubmit" class="flex-1 overflow-y-auto p-8 space-y-10 scrollbar-thin scrollbar-thumb-gray-200 scrollbar-track-transparent">
          
          <div class="space-y-6">
            <div class="flex items-center gap-4">
              <span class="flex-none text-[10px] font-black text-primary uppercase tracking-[0.2em]">01. Property & Type</span>
              <div class="flex-1 h-[1px] bg-gray-100"></div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="relative group">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Property</label>
                <div class="relative">
                  <i class="fas fa-building absolute left-4 top-1/2 -translate-y-1/2 text-gray-300 group-focus-within:text-primary transition-colors"></i>
                  <input
                    v-model="propertySearch"
                    @input="fetchProperties"
                    @focus="propertyDropdown = true"
                    class="w-full pl-11 pr-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:ring-4 focus:ring-primary/5 focus:border-primary outline-none transition-all"
                    placeholder="Search Property"
                    required
                  />
                </div>
                <ul v-if="propertyDropdown && properties.length" class="absolute z-20 w-full mt-2 bg-white border border-gray-100 rounded-2xl shadow-xl max-h-48 overflow-y-auto">
                  <li
                    v-for="p in properties" :key="p.id"
                    @mousedown.prevent="selectProperty(p)"
                    class="px-5 py-3 hover:bg-primary/5 cursor-pointer text-sm font-bold text-gray-600 border-b last:border-0 border-gray-50 transition-colors"
                  >
                    {{ p.name }}
                  </li>
                </ul>
              </div>

              <div class="space-y-1">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Rent Type</label>
                <select v-model="form.rent_type" class="w-full px-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:border-primary outline-none appearance-none cursor-pointer" required>
                  <option value="">Select Rent Type</option>
                  <option value="lease">Lease</option>
                  <option value="house_rent">House Rent</option>
                  <option value="vehicle_rent">Vehicle Rent</option>
                </select>
              </div>

              <div class="grid grid-cols-2 gap-4">
                <div class="space-y-1">
                  <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Start Date</label>
                  <input v-model="form.start_date" type="date" class="w-full px-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:border-primary outline-none" required />
                </div>
                <div class="space-y-1">
                  <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">End Date</label>
                  <input v-model="form.end_date" type="date" class="w-full px-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:border-primary outline-none" />
                </div>
              </div>

              <div class="space-y-1">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Payment Cycle</label>
                <select v-model="form.payment_cycle" class="w-full px-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:border-primary outline-none appearance-none cursor-pointer" required>
                  <option value="">Payment Cycle</option>
                  <option value="weekly">Weekly</option>
                  <option value="monthly">Monthly</option>
                  <option value="yearly">Yearly</option>
                </select>
              </div>
            </div>
          </div>

          <div class="bg-gray-50/80 rounded-[2.5rem] p-8 space-y-6 border border-gray-100/50">
            <div class="flex items-center gap-4">
              <span class="flex-none text-[10px] font-black text-secondary uppercase tracking-[0.2em]">02. Pricing</span>
              <div class="flex-1 h-[1px] bg-white"></div>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Rent Amount</label>
                <input v-model="form.rent_amount" type="number" class="w-full px-5 py-3.5 bg-white border border-gray-100 rounded-2xl text-sm font-bold focus:border-primary outline-none shadow-sm" placeholder="Rent Amount" required />
              </div>
              <div class="space-y-1">
                <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1 mb-2">Deposit Amount</label>
                <input v-model="form.deposit_amount" type="number" class="w-full px-5 py-3.5 bg-white border border-gray-100 rounded-2xl text-sm font-bold focus:border-secondary outline-none shadow-sm" placeholder="Deposit Amount" required />
              </div>
            </div>
          </div>

          <div class="space-y-5">
            <div class="flex justify-between items-center border-b border-gray-100 pb-3">
              <h3 class="text-[10px] font-black text-primary uppercase tracking-[0.2em]">03. Tenant Information</h3>
              <button type="button" @click="toggleTenantMode" class="px-4 py-1.5 rounded-full bg-primary/5 text-primary text-[10px] font-black uppercase tracking-widest hover:bg-primary hover:text-white transition-all">
                {{ tenantMode === 'search' ? '➕ Add New Tenant' : '🔍 Search Existing' }}
              </button>
            </div>

            <div v-if="tenantMode === 'search'" class="relative group">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-gray-300 group-focus-within:text-primary transition-colors"></i>
              <input
                v-model="tenantSearch"
                @input="fetchTenants"
                @focus="tenantDropdown = true"
                class="w-full pl-11 pr-4 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-primary outline-none transition-all shadow-sm"
                placeholder="Search Tenant by Name or Phone"
              />
              <ul v-if="tenantDropdown && tenants.length" class="absolute z-10 w-full mt-2 bg-white border border-gray-100 rounded-2xl shadow-xl max-h-48 overflow-y-auto">
                <li
                  v-for="u in tenants" :key="u.id"
                  @mousedown.prevent="selectTenant(u)"
                  class="px-5 py-3 hover:bg-primary/5 cursor-pointer text-[13px] font-bold text-gray-600 border-b last:border-0 border-gray-50 transition-colors"
                >
                  {{ u.first_name }} {{ u.middle_name }} {{ u.last_name }} 
                  <span class="text-primary text-[11px] ml-2 opacity-60">({{ u.phone_number }})</span>
                </li>
              </ul>
            </div>

            <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-3 animate-in fade-in slide-in-from-bottom-2 duration-300">
              <input v-model="tenant.first_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="First Name" />
              <input v-model="tenant.middle_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Middle Name" />
              <input v-model="tenant.last_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Last Name" />
              <input v-model="tenant.email" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Email" />
              <input v-model="tenant.phone_number" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Phone Number" />
              <input v-model="tenant.address" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Address" />
              <input v-model="tenant.password" type="password" class="md:col-span-3 px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-primary" placeholder="Set Password" />
            </div>
          </div>

          <div class="space-y-5">
            <div class="flex justify-between items-center border-b border-gray-100 pb-3">
              <h3 class="text-[10px] font-black text-secondary uppercase tracking-[0.2em]">04. Broker Attribution</h3>
              <button type="button" @click="toggleBrokerMode" class="px-4 py-1.5 rounded-full bg-secondary/5 text-secondary text-[10px] font-black uppercase tracking-widest hover:bg-secondary hover:text-white transition-all">
                {{ brokerMode === 'search' ? '➕ Add New Broker' : '🔍 Search Existing' }}
              </button>
            </div>

            <div v-if="brokerMode === 'search'" class="relative group">
              <input
                v-model="brokerSearch"
                @input="fetchBrokers"
                @focus="brokerDropdown = true"
                class="w-full px-5 py-3.5 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-bold focus:bg-white focus:border-secondary outline-none transition-all shadow-sm"
                placeholder="Search Broker"
              />
              <ul v-if="brokerDropdown && brokers.length" class="absolute z-10 w-full mt-2 bg-white border border-gray-100 rounded-2xl shadow-xl max-h-40 overflow-y-auto">
                <li
                  v-for="u in brokers" :key="u.id"
                  @mousedown.prevent="selectBroker(u)"
                  class="px-5 py-3 hover:bg-secondary/5 cursor-pointer text-[13px] font-bold text-gray-600 border-b last:border-0 border-gray-50 transition-colors"
                >
                  {{ u.first_name }} {{ u.middle_name }} {{ u.last_name }}
                </li>
              </ul>
            </div>

            <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-3 animate-in fade-in slide-in-from-bottom-2 duration-300">
              <input v-model="brokerForm.first_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="First Name" />
              <input v-model="brokerForm.middle_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Middle Name" />
              <input v-model="brokerForm.last_name" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Last Name" />
              <input v-model="brokerForm.email" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Email" />
              <input v-model="brokerForm.phone_number" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Phone Number" />
              <input v-model="brokerForm.address" class="px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Address" />
              <input v-model="brokerForm.password" type="password" class="md:col-span-3 px-4 py-3 bg-gray-50 border border-gray-100 rounded-xl text-xs font-bold outline-none focus:border-secondary" placeholder="Set Password" />
            </div>
          </div>

          <div class="sticky bottom-0 bg-white pt-6 pb-2 border-t border-gray-50 flex items-center justify-end gap-5">
            <button type="button" @click="$emit('close')" class="px-8 py-4 rounded-2xl text-[10px] font-black uppercase tracking-[0.2em] text-gray-400 hover:text-rose-500 hover:bg-rose-50 transition-all">
              Discard
            </button>
            <button 
              type="submit" 
              :disabled="loading"
              class="bg-primary hover:bg-dprimary disabled:bg-gray-200 text-white font-black text-[10px] uppercase tracking-[0.2em] px-12 py-4 rounded-2xl shadow-2xl shadow-primary/30 active:scale-95 transition-all flex items-center gap-3"
            >
              <i v-if="loading" class="fas fa-circle-notch animate-spin text-xs"></i>
              {{ loading ? "Saving Record..." : "Confirm & Save Rent" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>


<script>
import Toast from "@/components/Toast.vue";

export default {
  components: { Toast },
  props: { visible: Boolean },

  data() {
    return {
      loading: false,

      tenantMode: "search",
      brokerMode: "search",

      properties: [],
      propertySearch: "",
      propertyDropdown: false,

      tenants: [],
      tenantSearch: "",
      tenantDropdown: false,
      selectedTenant: null,

      brokers: [],
      brokerSearch: "",
      brokerDropdown: false,
      selectedBroker: null,
      status:"pending",

      form: {
        property_id: "",
        rent_type: "",
        start_date: "",
      
        end_date: "",
        payment_cycle: "",
        rent_amount: "",
        deposit_amount: "",
        status:"pending"
      },

      tenant: {
        first_name: "",
        middle_name: "",
        last_name: "",
        email: "",
        phone_number: "",
        address: "",
        password: "",
        is_tenant:true,
      },

      brokerForm: {
        first_name: "",
        middle_name: "",
        last_name: "",
        email: "",
        phone_number: "",
        address: "",
        password: "",

      },
    };
  },

  methods: {
    toggleTenantMode() {
      this.tenantMode = this.tenantMode === "search" ? "add" : "search";
      this.selectedTenant = null;
      this.tenantSearch = "";
    },

    toggleBrokerMode() {
      this.brokerMode = this.brokerMode === "search" ? "add" : "search";
      this.selectedBroker = null;
      this.brokerSearch = "";
    },

    async fetchProperties() {
      const res = await this.$apiGet("/get_properties", { search: this.propertySearch });
      this.properties = res.data || [];
    },

    selectProperty(p) {
      this.form.property_id = p.id;
      this.propertySearch = p.name;
      this.propertyDropdown = false;
    },

    async fetchTenants() {
      const res = await this.$apiGet("/get_users", {
        search: this.tenantSearch,
        is_tenant: true,
      });
      this.tenants = res.data || [];
    },

    selectTenant(u) {
      this.selectedTenant = u;
      this.tenantSearch = `${u.first_name} ${u.last_name}`;
      this.tenantDropdown = false;
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

    async handleSubmit() {
      this.loading = true;
      try {
        
          const tenantId =
          this.tenantMode === "search" && this.selectedTenant
            ? this.selectedTenant.id
            : (await this.$apiPost("/sign_up", { ...this.tenant, is_tenant: true })).user.id;

            console.log("Tenant ID:", tenantId);

          // const brokerId =
          // this.brokerMode === "search" && this.selectedBroker
          //   ? this.selectedBroker.id
          //   : (await this.$apiPost("/sign_up", { ...this.brokerForm, is_broker: true })).id;

        await this.$apiPost("/post_rent", {
          ...this.form,
          user_id: tenantId,
          broker: null,
        });

        this.$root.$refs.toast.showToast("Rent added successfully", "success");
        this.$emit("refresh");
        this.$emit("close");
      } catch (e) {
      
         this.$root.$refs.toast.showToast(e.message.error,"error");

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
.custom-select {
  @apply w-full px-3 py-2 border rounded;
}
.dropdown {
  @apply absolute bg-white border w-full max-h-40 overflow-y-auto z-50;
}
.dropdown li {
  @apply px-3 py-2 hover:bg-gray-100 cursor-pointer;
}
</style>
