<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-3xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100"
      >
        <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg">
              <i class="fas fa-tools text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Maintenance Request</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">New Service Ticket</p>
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
          <form id="maintenanceForm" @submit.prevent="submitForm" class="space-y-6">
            
            <div class="space-y-1">
              <label class="custom-label">Detailed Description <span class="custom-star">*</span></label>
              <textarea
                v-model="form.description"
                rows="4"
                required
                class="custom-input !mt-0 !py-3 leading-relaxed"
                placeholder="Describe the issue (e.g., Leaking pipe in the kitchen...)"
              ></textarea>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="custom-label">Initial Status</label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary pointer-events-none">
                    <i class="fas fa-info-circle text-xs"></i>
                  </span>
                  <select v-model="form.status" class="custom-input !mt-0 !pl-10 h-11 appearance-none">
                    <option value="pending">Pending</option>
                    <option value="active">Active</option>
                    <option value="resolved">Resolved</option>
                  </select>
                </div>
              </div>

              <div class="space-y-1 relative">
                <label class="custom-label">Property <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-building text-xs"></i>
                  </span>
                  <input
                    v-model="propertySearch"
                    type="text"
                    required
                    placeholder="Search Property..."
                    class="custom-input !mt-0 !pl-10 h-11"
                    @input="searchProperties"
                    @focus="propertyDropdown = true"
                    @blur="hideDropdown('property')"
                  />
                </div>
                <ul
                  v-if="properties.length > 0 && propertyDropdown"
                  class="absolute z-50 w-full bg-white border border-gray-200 rounded-xl shadow-xl mt-1 max-h-48 overflow-y-auto modal-scroll"
                >
                  <li
                    v-for="property in properties"
                    :key="property.id"
                    @mousedown.prevent="selectProperty(property)"
                    class="px-4 py-3 hover:bg-primary/5 cursor-pointer flex justify-between items-center group transition-colors border-b border-gray-50 last:border-0"
                  >
                    <span class="text-xs font-bold text-gray-700 group-hover:text-primary">{{ property.name }}</span>
                    <i class="fas fa-plus text-[10px] text-gray-300 group-hover:text-primary"></i>
                  </li>
                </ul>
              </div>

              <div class="space-y-1 relative md:col-span-2">
                <label class="custom-label">Requester (Tenant) <span class="custom-star">*</span></label>
                <div class="relative group">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-user text-xs"></i>
                  </span>
                  <input
                    v-model="userSearch"
                    type="text"
                    required
                    autocomplete="off"
                    placeholder="Search by name..."
                    class="custom-input !mt-0 !pl-10 h-11"
                    @input="searchUsers"
                    @focus="userDropdown = true"
                    @blur="hideDropdown('user')"
                  />
                </div>
                <ul
                  v-if="users.length > 0 && userDropdown"
                  class="absolute z-50 w-full bg-white border border-gray-200 rounded-xl shadow-xl mt-1 max-h-48 overflow-y-auto modal-scroll"
                >
                  <li
                    v-for="user in users"
                    :key="user.id"
                    @mousedown.prevent="selectUser(user)"
                    class="px-4 py-3 hover:bg-primary/5 cursor-pointer flex justify-between items-center group transition-colors border-b border-gray-50 last:border-0"
                  >
                    <div class="flex items-center gap-3">
                      <div class="w-6 h-6 rounded-full bg-gray-100 flex items-center justify-center text-[10px] font-bold text-gray-500">
                        {{ user.first_name[0] }}{{ user.last_name[0] }}
                      </div>
                      <span class="text-xs font-bold text-gray-700 group-hover:text-primary">
                        {{ user.first_name }} {{ user.middle_name }} {{ user.last_name }}
                      </span>
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
            form="maintenanceForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-save"></i>
            Save Request
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";

export default {
  name: "MaintenanceRequestAdd",
  components: { Toast },
  props: {
    visible: Boolean,
  },
  data() {
    return {
      form: {
        status: "pending",
        description: "",
        requested_at: new Date().toISOString().split("T")[0],
        user_id: "",
        property_id: "",
      },
      properties: [],
      users: [],
      propertySearch: "",
      userSearch: "",
      propertyDropdown: false,
      userDropdown: false,
      ordering:"-id"
    };
  },
  mounted() {
    this.fetchProperties();
    this.fetchUsers();
  },
  methods: {
    // ✅ Fetch all properties initially
    async fetchProperties(url=null) {
      try {
         const pageUrl =
          url ||
          `/get_properties?search=${this.propertySearch}&ordering=${this.ordering}`;

        let result = [];
      
          result = await this.$getProperties(pageUrl);
       

        this.properties = result.properties;
      
      } catch (err) {
        console.error("Error fetching properties:", err);
      }
    },

    // ✅ Fetch all users initially
   async fetchUsers(url = null) {
  try {
    const pageUrl = url || `/get_rents?search=${this.userSearch}`;

    // 👇 FIX: Bind `this` so $apiGet works properly inside getTenants
    const response = await this.$getTenants.call(this, pageUrl);

    this.users = response.tenants || response.data || [];
  } catch (err) {
    console.error("Error fetching users:", err);
  }
},

    // ✅ Search properties like AddSale
    searchProperties() {
    
        this.fetchProperties();
     
    },

    // ✅ Search users
    searchUsers() {
     
        this.fetchUsers();
     
    },

    // ✅ Selectors
    selectProperty(property) {
      this.form.property_id = property.id;
      this.propertySearch = property.name;
      this.propertyDropdown = false;
    },
    selectUser(user) {
      this.form.user_id = user.id;
      this.userSearch = `${user.first_name} ${user.middle_name} ${user.last_name}`;
      this.userDropdown = false;
    },

    // ✅ Hide dropdowns gracefully
    hideDropdown(type) {
      setTimeout(() => {
        if (type === "property") this.propertyDropdown = false;
        if (type === "user") this.userDropdown = false;
      }, 200);
    },

    // ✅ Submit
    async submitForm() {
      try {
        this.form.requested_at = new Date().toISOString().split("T")[0];

        const res = await this.$apiPost("/post_maintenance_request", this.form);
        if (res?.data?.error) {
          this.$root.$refs.toast.showToast(res.data.error || "Failed to save request", "error");
        } else {
          this.$root.$refs.toast.showToast("Maintenance request saved successfully", "success");
        }

        // Reset form
        this.form = {
          status: "pending",
          description: "",
          requested_at: new Date().toISOString().split("T")[0],
          user_id: "",
          property_id: "",
        };
        this.propertySearch = "";
        this.userSearch = "";

        setTimeout(() => this.$emit("close"), 2000);
      } catch (error) {
        console.error("Error saving request:", error);
        this.$refs.toast.showToast("Failed to save request", "error");
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
