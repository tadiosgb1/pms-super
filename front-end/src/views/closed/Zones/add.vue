<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-2xl rounded-[2rem] shadow-2xl flex flex-col max-h-[90vh] overflow-y-auto border border-gray-100"
      >
        <div class="bg-gradient-to-r from-primary to-dprimary px-6 py-5 flex justify-between items-center shadow-md shrink-0 sticky top-0 bg-white">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg">
              <i class="fas fa-plus text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Property Zone</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Register Geographic Boundaries</p>
            </div>
          </div>
          <button 
            @click="$emit('close')" 
            class="w-8 h-8 flex items-center justify-center rounded-lg bg-slate-100 hover:bg-red-100 text-slate-400 hover:text-red-600 transition-all duration-200 flex-shrink-0"
          >
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-6 md:p-8">
          <form id="zoneForm" @submit.prevent="submitForm" class="space-y-4">
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="relative group">
                <label class="custom-label">Zone Manager <span class="custom-star">*</span></label>
                <div class="relative">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary">
                    <i class="fas fa-user-tie text-xs"></i>
                  </span>
                  <input
                    v-model="managerSearch"
                    type="text"
                    class="custom-input !mt-0 !pl-10 h-11"
                    placeholder="Search Manager..."
                    @input="searchManagers"
                    @focus="managerDropdown = true"
                    @blur="hideDropdown('manager')"
                  />
                </div>
                <ul
                  v-if="managers.length && managerDropdown"
                  class="absolute z-50 w-full max-h-48 overflow-y-auto bg-white border border-gray-200 rounded-xl shadow-xl mt-1 py-1"
                >
                  <li
                    v-for="manager in managers"
                    :key="manager.manager.id"
                    class="px-4 py-2 hover:bg-primary/5 hover:text-primary cursor-pointer text-sm font-medium transition-colors"
                    @mousedown.prevent="selectManager(manager)"
                  >
                    {{ manager.manager.first_name }} {{ manager.manager.last_name || "" }}
                  </li>
                </ul>
              </div>

              <div class="space-y-1 group">
                <label class="custom-label">Zone Name <span class="custom-star">*</span></label>
                <input v-model="form.name" placeholder="e.g. North Sector" class="custom-input !mt-0 h-11" required />
              </div>
            </div>

            <div class="bg-gray-50 rounded-xl p-5 border border-gray-100 space-y-4">
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Regional Information</p>
              
              <div class="space-y-1">
                <label class="text-[11px] font-bold text-gray-600">Street Address</label>
                <input v-model="form.address" placeholder="Full address details..." class="custom-input !mt-0 h-10 shadow-sm" />
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div class="space-y-1">
                  <label class="text-[11px] font-bold text-gray-600">City</label>
                  <input v-model="form.city" class="custom-input !mt-0 h-10 shadow-sm" />
                </div>
                <div class="space-y-1">
                  <label class="text-[11px] font-bold text-gray-600">State / Region</label>
                  <input v-model="form.state" class="custom-input !mt-0 h-10 shadow-sm" />
                </div>
              </div>
            </div>

            <div class="flex items-start gap-3 p-4 bg-amber-50 rounded-xl border border-amber-100">
              <i class="fas fa-info-circle text-amber-500 mt-1"></i>
              <p class="text-[11px] leading-relaxed text-amber-800 font-medium">
                <span class="font-bold">NB:</span> Please be physically present in the zone to capture accurate location. Otherwise, coordinates must be entered manually via system owner.
              </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="space-y-1">
                <label class="custom-label">Latitude</label>
                <input
                  v-model="form.latitude"
                  readonly
                  class="custom-input !mt-0 h-11 bg-gray-100 cursor-not-allowed font-mono text-xs"
                />
              </div>
              <div class="space-y-1">
                <label class="custom-label">Longitude</label>
                <input
                  v-model="form.longitude"
                  readonly
                  class="custom-input !mt-0 h-11 bg-gray-100 cursor-not-allowed font-mono text-xs"
                />
              </div>
            </div>

            <div class="space-y-4">
              <div class="flex justify-between items-center">
                <label class="custom-label">Amenities / Features</label>
                <button
                  type="button"
                  @click="addAmenity"
                  class="text-[10px] font-black text-primary hover:text-dprimary uppercase tracking-tighter flex items-center gap-1"
                >
                  <i class="fas fa-plus-circle"></i> Add New Field
                </button>
              </div>

              <div class="space-y-3">
                <div
                  v-for="(item, index) in amenities"
                  :key="index"
                  class="flex gap-3 items-center group animate-in slide-in-from-top-1"
                >
                  <input
                    v-model="item.amenity"
                    placeholder="Feature (e.g. Water)"
                    class="custom-input !mt-0 h-10 w-1/2 shadow-sm"
                  />
                  <input
                    v-model="item.value"
                    placeholder="Value (e.g. 24/7)"
                    class="custom-input !mt-0 h-10 w-1/2 shadow-sm"
                  />
                  <button
                    v-if="amenities.length > 1"
                    type="button"
                    @click="removeAmenity(index)"
                    class="w-10 h-10 shrink-0 flex items-center justify-center rounded-lg text-red-300 hover:text-red-600 hover:bg-red-50 transition-all"
                  >
                    <i class="fas fa-trash-alt text-xs"></i>
                  </button>
                </div>
              </div>
            </div>
          </form>
        </div>

        <div class="px-8 py-5 bg-white border-t-2 border-slate-100 flex items-center justify-end gap-3 shrink-0 sticky bottom-0">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase tracking-widest transition-colors"
          >
            Cancel
          </button>
          <button
            form="zoneForm"
            type="submit"
            :disabled="loading"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i v-if="loading" class="fas fa-spinner fa-spin"></i>
            <i v-else class="fas fa-map-pin"></i>
            {{ loading ? "Saving..." : "Save Property Zone" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>


<script>
export default {
  props: { visible: Boolean },

  data() {
    return {
      managerSearch: "",
      managerDropdown: false,
      managers: [],
      loading: false,

      amenities: [{ amenity: "", value: "" }],

      form: {
        owner_id: localStorage.getItem("userId"),
        name: "",
        address: "",
        city: "",
        state: "",
        manager_id: "",
        latitude: "",
        longitude: "",
        description: "" // ✅ IMPORTANT
      },
    };
  },

  watch: {
    visible(val) {
      if (val) this.getCurrentLocation();
    },
  },

  async mounted() {
    const result = await this.$getManagers();
    this.managers = result.managers;

    if (this.visible) this.getCurrentLocation();
  },

  methods: {
    /* Managers */
    searchManagers() {
      if (!this.managerSearch) return;
      this.managers = this.managers.filter(m =>
        `${m.manager.first_name} ${m.manager.last_name || ""}`
          .toLowerCase()
          .includes(this.managerSearch.toLowerCase())
      );
    },

    selectManager(manager) {
      this.form.manager_id = manager.manager.id;
      this.managerSearch = `${manager.manager.first_name} ${manager.manager.last_name || ""}`;
      this.managerDropdown = false;
    },

    hideDropdown() {
      setTimeout(() => (this.managerDropdown = false), 200);
    },

    /* Amenities */
    addAmenity() {
      this.amenities.push({ amenity: "", value: "" });
    },

    removeAmenity(index) {
      this.amenities.splice(index, 1);
    },

    buildDescription() {
      return this.amenities
        .filter(a => a.amenity && a.value)
        .map(a => `${a.amenity.trim()}:"${a.value.trim()}"`)
        .join(",");
    },

    /* Location */
    getCurrentLocation() {
      if (!navigator.geolocation) return;

      navigator.geolocation.getCurrentPosition(
        pos => {
          this.form.latitude = pos.coords.latitude.toFixed(6);
          this.form.longitude = pos.coords.longitude.toFixed(6);
        },
        err => console.error("Location error:", err)
      );
    },

    /* Submit */
    async submitForm() {
      this.loading = true;

      // ✅ DO NOT reassign form
      this.form.description = this.buildDescription();

      try {
        await this.$apiPost("post_property_zone", this.form);
        this.$reloadPage();
        this.$emit("close");
      } catch (err) {
       this.$emit("close");
         this.$root.$refs.toast.showToast("error",err[0]);
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply border border-gray-300 rounded px-3 py-2 w-full focus:outline-none focus:ring-2 focus:ring-primary;
}
</style>
