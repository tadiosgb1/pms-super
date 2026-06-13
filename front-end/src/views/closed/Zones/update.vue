<template>

  <div>

    <Toast ref="toast" />

    <!-- MAIN MODAL -->
    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div
        class="bg-white w-full max-w-2xl rounded-[2rem] shadow-2xl flex flex-col max-h-[90vh] overflow-y-auto border border-gray-100"
      >
        <div class="bg-gradient-to-r from-secondary to-orange-600 px-6 py-5 flex justify-between items-center shadow-md shrink-0 sticky top-0 bg-white">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg backdrop-blur-md">
              <i class="fas fa-edit text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Edit Property Zone</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">
                Modify Zone Configuration
              </p>
            </div>
          </div>

          <button
            @click="$emit('close')"
            class="w-8 h-8 flex items-center justify-center rounded-lg bg-slate-100 hover:bg-red-100 text-slate-400 hover:text-red-600 transition-all duration-200 flex-shrink-0"
          >
            <i class="fas fa-times"></i>
          </button>
        </div>

        <!-- BODY -->
        <div class="flex-1 overflow-y-auto p-6 md:p-8">
          <form class="space-y-4">

            <!-- ZONE NAME + MANAGER -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div>
                <label class="custom-label">Zone Name *</label>
                <input v-model="form.name" class="custom-input h-11" required />
              </div>

              <div>
                <label class="custom-label">Manager *</label>
                <div class="relative">
                  <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400">
                    <i class="fas fa-user-shield text-xs"></i>
                  </span>
                  <input
                    v-model="form.manager_id"
                    type="number"
                    class="custom-input !pl-10 h-11"
                    required
                  />
                </div>
              </div>
            </div>

            <!-- LOCATION -->
            <div class="bg-gray-50 rounded-xl p-5 border border-gray-100 space-y-4">
              <div class="flex items-center gap-2">
                <i class="fas fa-map-marker-alt text-primary text-xs"></i>
                <p class="text-[10px] font-black text-gray-400 uppercase">Location Details</p>
              </div>

              <div>
                <label class="text-[11px] font-bold text-gray-600">Physical Address</label>
                <input v-model="form.address" class="custom-input h-10 shadow-sm" />
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="text-[11px] font-bold text-gray-600">City</label>
                  <input v-model="form.city" class="custom-input h-10 shadow-sm" />
                </div>
                <div>
                  <label class="text-[11px] font-bold text-gray-600">State / Region</label>
                  <input v-model="form.state" class="custom-input h-10 shadow-sm" />
                </div>
              </div>
            </div>

            <!-- LAT / LNG -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div>
                <label class="custom-label">Latitude</label>
                <input v-model="form.latitude" class="custom-input h-11 bg-gray-100 font-mono text-xs" />
              </div>

              <div>
                <label class="custom-label">Longitude</label>
                <input v-model="form.longitude" class="custom-input h-11 bg-gray-100 font-mono text-xs" />
              </div>
            </div>

            <!-- AMENITIES -->
            <div class="space-y-4 border-t border-gray-100 pt-6 mt-6">
              <div class="flex justify-between items-center">
                <label class="custom-label">Amenities / Features</label>
                <button
                  type="button"
                  @click="addAmenity"
                  class="text-[10px] font-black text-primary hover:text-dprimary uppercase flex items-center gap-1"
                >
                  <i class="fas fa-plus-circle"></i> Add New Field
                </button>
              </div>

              <div class="space-y-3">
                <div v-for="(item, index) in amenities" :key="index" class="flex gap-3 items-center">
                  <input
                    v-model="item.amenity"
                    placeholder="Feature (e.g. Water)"
                    class="custom-input h-10 w-1/2 shadow-sm"
                  />
                  <input
                    v-model="item.value"
                    placeholder="Value (e.g. 24/7)"
                    class="custom-input h-10 w-1/2 shadow-sm"
                  />

                  <button
                    v-if="amenities.length > 1"
                    type="button"
                    @click="removeAmenity(index)"
                    class="w-10 h-10 flex items-center justify-center text-red-300 hover:text-red-600"
                  >
                    <i class="fas fa-trash-alt text-xs"></i>
                  </button>
                </div>
              </div>
            </div>

          </form>
        </div>

        <!-- FOOTER -->
        <div class="px-8 py-5 bg-white border-t-2 border-slate-100 flex justify-end gap-3 sticky bottom-0">
          <button
            type="button"
            @click="$emit('close')"
            class="px-6 py-2 text-sm font-bold text-gray-400 hover:text-gray-600 uppercase"
          >
            Cancel
          </button>

          <button
            type="button"
            @click="updateModalVisible = true; visible = false"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-check-circle"></i>
            Update Zone
          </button>
        </div>

      </div>
    </div>

    <!-- INLINE CONFIRM MODAL (FIXED WITH z-[9999]) -->
    <div
      v-if="updateModalVisible"
      class="fixed inset-0 z-[9999] flex items-center justify-center p-4 bg-black/50 backdrop-blur-sm"
      @click.self="updateModalVisible = false"
    >
      <div class="bg-white rounded-xl p-6 w-full max-w-md shadow-lg">
        <h3 class="text-lg font-bold mb-4">Confirm Changes</h3>
        <p class="text-sm mb-6">Are you sure you want to apply updates to </p>

        <div class="flex justify-end gap-3">
          <button
            class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
            @click="updateModalVisible = false"
          >
            Cancel
          </button>

          <button
            class="px-4 py-2 bg-primary text-white rounded hover:bg-dprimary"
            @click="submitForm"
          >
            Confirm
          </button>
        </div>

      </div>
    </div>

  </div>

</template>



<script>

import Toast from "../../../components/Toast.vue";

export default {
  props: {
    visible: Boolean,
    zone: Object
  },
  components: { Toast },

  data() {
    return {
      form: {
        id: null,
        name: "",
        address: "",
        city: "",
        state: "",
        owner_id: null,
        manager_id: null,
        latitude: "",
        longitude: "",
        description: ""
      },
      amenities: [{ amenity: "", value: "" }],
      updateModalVisible: false
    };
  },

  watch: {
    zone: {
      immediate: true,
      handler(val) {
        if (val) this.loadEditData(val);
      }
    }
  },

  methods: {
    addAmenity() {
      this.amenities.push({ amenity: "", value: "" });
    },

    removeAmenity(index) {
      this.amenities.splice(index, 1);
    },

    loadEditData(zone) {
      this.form = { ...zone };

      if (zone.description) {
        this.amenities = zone.description.split(",").map(item => {
          const [key, val] = item.split(":");
          return { amenity: key?.trim(), value: val?.replace(/"/g, "").trim() };
        });
      } else {
        this.amenities = [{ amenity: "", value: "" }];
      }
    },

    buildDescription() {
      return this.amenities
        .filter(a => a.amenity && a.value)
        .map(a => `${a.amenity}:"${a.value}"`)
        .join(",");
    },

    async submitForm() {
      console.log("Submitting form...");
      this.updateModalVisible = false;
      this.form.description = this.buildDescription();

      try {
        const response = await this.$apiPut("/update_property_zone", this.form.id, this.form);

        if (response?.error) {
          this.$root.$refs.toast.showToast(response.error, "error");
        } else {
          this.$root.$refs.toast.showToast("Property zone updated successfully", "success");
          this.$emit("refresh");
          this.$emit("close");
        }
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast("Failed to update property zone", "error");
      }
    }
  }
};

</script>



<style scoped>
.custom-input {
  @apply border border-gray-300 rounded px-3 py-2 w-full focus:outline-none focus:ring-2 focus:ring-primary;
}
</style>
