<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-4xl rounded-2xl shadow-2xl flex flex-col max-h-[95vh] overflow-hidden border border-gray-100 animate-in fade-in zoom-in duration-200"
      >
        <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg backdrop-blur-md text-white">
              <i class="fas fa-edit text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Edit Co-Working Space</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Modify Space Configurations</p>
            </div>
          </div>
          <button 
            @click="$emit('close')" 
            class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-colors"
          >
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-6 md:p-8 space-y-8">
          <form id="updateSpaceForm" @submit.prevent="submitForm" class="space-y-8">
            
            <div class="space-y-4">
              <div class="flex items-center gap-2 mb-2">
                <i class="fas fa-fingerprint text-primary text-xs"></i>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">General Identity</p>
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="space-y-1">
                  <label class="custom-label">Display Name</label>
                  <input v-model="form.name" type="text" class="custom-input !mt-0 h-11 shadow-sm" />
                </div>
                <div class="space-y-1">
                  <label class="custom-label">Physical Location</label>
                  <input v-model="form.location" type="text" class="custom-input !mt-0 h-11 shadow-sm" />
                </div>
                <div class="md:col-span-2 space-y-1">
                  <label class="custom-label">Assigned Property Zone</label>
                  <div class="relative">
                    <select v-model="form.zone" class="custom-input !mt-0 h-11 appearance-none shadow-sm">
                      <option disabled value="">Select a zone</option>
                      <option v-for="zone in zones" :key="zone.id" :value="zone.id">
                        {{ zone.name }} — {{ zone.city }}
                      </option>
                    </select>
                    <div class="absolute inset-y-0 right-3 flex items-center pointer-events-none text-gray-400">
                      <i class="fas fa-chevron-down text-xs"></i>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <hr class="border-gray-100" />

            <div class="space-y-4">
              <div class="flex items-center gap-2 mb-2">
                <i class="fas fa-tags text-primary text-xs"></i>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Revenue & Occupancy</p>
              </div>
              <div class="bg-gray-50/50 rounded-2xl border border-gray-100 p-6 grid grid-cols-2 md:grid-cols-5 gap-4">
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-gray-400 uppercase">Seats</label>
                  <input v-model="form.capacity" type="number" class="custom-input !mt-0 h-10 font-bold" />
                </div>
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-gray-400 uppercase">Daily</label>
                  <input v-model="form.price_daily" type="number" class="custom-input !mt-0 h-10" />
                </div>
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-gray-400 uppercase">Monthly</label>
                  <input v-model="form.price_monthly" type="number" class="custom-input !mt-0 h-10" />
                </div>
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-gray-400 uppercase">Quarterly</label>
                  <input v-model="form.price_quarterly" type="number" class="custom-input !mt-0 h-10" />
                </div>
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-gray-400 uppercase">Yearly</label>
                  <input v-model="form.price_yearly" type="number" class="custom-input !mt-0 h-10" />
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <div class="flex justify-between items-center">
                <div class="flex items-center gap-2">
                  <i class="fas fa-shield-alt text-primary text-xs"></i>
                  <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Included Amenities</p>
                </div>
                <button type="button" @click="addAmenity" class="text-[10px] font-black text-primary uppercase tracking-widest hover:text-dprimary flex items-center gap-1.5 transition-colors">
                  <i class="fas fa-plus-circle"></i> New Row
                </button>
              </div>

              <div class="grid grid-cols-1 gap-3">
                <div v-for="(item, index) in amenities" :key="index" class="flex gap-3 items-center group animate-in slide-in-from-right-2 duration-200">
                  <div class="flex-1 grid grid-cols-2 gap-3">
                    <input v-model="item.amenity" placeholder="Amenity Label" class="custom-input !mt-0 h-10 text-sm shadow-sm" />
                    <input v-model="item.value" placeholder="Description/Value" class="custom-input !mt-0 h-10 text-sm shadow-sm" />
                  </div>
                  <button v-if="amenities.length > 1" type="button" @click="removeAmenity(index)" class="w-8 h-8 rounded-lg flex items-center justify-center text-red-400 hover:bg-red-50 hover:text-red-600 transition-all">
                    <i class="fas fa-times-circle"></i>
                  </button>
                </div>
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
            form="updateSpaceForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-sync-alt"></i>
            Commit Changes
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";

export default {
  name: "UpdateCoworkspace",
  components: { Toast },

  props: {
    visible: Boolean,
    space: Object,
  },

  data() {
    return {
      form: {
        id: null,
        name: "",
        location: "",
        description: "",
        capacity: null,
        price_daily: "",
        price_monthly: "",
        price_quarterly: "",
        price_yearly: "",
        zone: "",
      },

      amenities: [{ amenity: "", value: "" }],

      zones: [],
      zoneSearch: "",
    };
  },

  watch: {
    visible(val) {
      if (val) {
        this.fetchZones();
      }
    },

    space: {
      immediate: true,
      handler(val) {
        if (!val) return;

        this.form = { ...val };
        this.amenities = this.parseAmenities(val.description);
      },
    },
  },

  methods: {
    // 🔥 PROVIDED METHOD (unchanged)
    async fetchZones() {
      try {
        const url = `/get_property_zones?search=${this.zoneSearch}`;
        const result = await this.$getZones(url);
        this.zones = result.zones;
      } catch (err) {
        console.error("Failed to fetch zones:", err);
      }
    },

    parseAmenities(description) {
      if (!description) return [{ amenity: "", value: "" }];

      return description.split(",").map(pair => {
        const [key, value] = pair.split(":");
        return {
          amenity: key?.trim() || "",
          value: value?.replace(/"/g, "").trim() || "",
        };
      });
    },

    addAmenity() {
      this.amenities.push({ amenity: "", value: "" });
    },

    removeAmenity(index) {
      this.amenities.splice(index, 1);
    },

    async submitForm() {
      try {
        this.form.description = this.amenities
          .filter(a => a.amenity && a.value)
          .map(a => `${a.amenity.trim()}:"${a.value.trim()}"`)
          .join(",");

        await this.$apiPut(
          `/update_coworking_space`,
          this.form.id,
          this.form
        );

        this.$root.$refs.toast.showToast(
          "Co-Working Space updated successfully",
          "success"
        );

        this.$emit("refresh");
        this.$emit("close");
      } catch (err) {
        console.error("Update failed:", err);
        alert("Failed to update co-working space.");
      }
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-4 py-2 border border-gray-300 rounded
    focus:outline-none focus:ring-2 focus:ring-blue-500;
}
</style>
