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
              <i class="fas fa-plus-circle text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Co-Working Space</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Register New Work Environment</p>
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
          <form id="addSpaceForm" @submit.prevent="submitForm" class="space-y-8">
            
            <div class="space-y-4">
              <div class="flex items-center gap-2 mb-2">
                <i class="fas fa-info-circle text-primary text-xs"></i>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Core Identification</p>
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div class="space-y-1">
                  <label class="custom-label">Space Name <span class="custom-star">*</span></label>
                  <input v-model="form.name" type="text" placeholder="Executive Suite A" class="custom-input !mt-0 h-11" required />
                </div>
                <div class="space-y-1">
                  <label class="custom-label">Specific Location <span class="custom-star">*</span></label>
                  <input v-model="form.location" type="text" placeholder="Floor 4, East Wing" class="custom-input !mt-0 h-11" required />
                </div>
                <div class="space-y-1">
                  <label class="custom-label">Service Category</label>
                  <select v-model="form.service_type" class="custom-input !mt-0 h-11 appearance-none">
                    <option value="" disabled>Select Space Type</option>
                    <option value="single_space">Single Hot Desk</option>
                    <option value="multiple_space">Dedicated Desk Group</option>
                    <option value="private_space">Private Office</option>
                    <option value="meeting_room">Meeting/Board Room</option>
                    <option value="office">Corporate Office</option>
                    <option value="evet_space">Event Venue</option>
                    <option value="other">Other specialized</option>
                  </select>
                </div>
                <div class="space-y-1 relative">
                  <label class="custom-label">Property Zone Association <span class="custom-star">*</span></label>
                  <div class="relative group">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400">
                      <i class="fas fa-search text-[10px]"></i>
                    </span>
                    <input
                      v-model="zoneSearch"
                      type="text"
                      class="custom-input !mt-0 !pl-9 h-11"
                      placeholder="Type to search zone..."
                      @input="searchZones"
                      @focus="zoneDropdown = true"
                      @blur="hideDropdown('zone')"
                      required
                    />
                  </div>
                  <ul v-if="zones.length > 0 && zoneDropdown" class="absolute z-50 w-full max-h-48 overflow-y-auto bg-white border border-gray-200 rounded-xl shadow-xl mt-1 animate-in fade-in slide-in-from-top-2">
                    <li v-for="zone in zones" :key="zone.id" class="px-4 py-2.5 hover:bg-primary/5 text-sm font-semibold text-gray-700 cursor-pointer border-b border-gray-50 last:border-0" @mousedown.prevent="selectZone(zone)">
                      {{ zone.name }}
                    </li>
                  </ul>
                </div>
              </div>
            </div>

            <hr class="border-gray-100" />

            <div class="space-y-4">
              <div class="flex items-center gap-2 mb-2">
                <i class="fas fa-dollar-sign text-primary text-xs"></i>
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Pricing & Capacity</p>
              </div>
              <div class="bg-gray-50/50 rounded-2xl border border-gray-100 p-6 grid grid-cols-1 md:grid-cols-4 gap-6">
                <div class="space-y-1">
                  <label class="text-[10px] font-bold text-gray-500 uppercase tracking-tighter">Capacity (Seats)</label>
                  <input v-model.number="form.capacity" type="number" min="1" class="custom-input !mt-0 h-10 font-bold" required />
                </div>
                <div class="space-y-1">
                  <label class="text-[10px] font-bold text-gray-500 uppercase tracking-tighter">Daily Rate</label>
                  <input v-model="form.price_daily" type="number" step="0.01" class="custom-input !mt-0 h-10" required />
                </div>
                <div class="space-y-1">
                  <label class="text-[10px] font-bold text-gray-500 uppercase tracking-tighter">Monthly Rate</label>
                  <input v-model="form.price_monthly" type="number" step="0.01" class="custom-input !mt-0 h-10" required />
                </div>
                <div class="space-y-1">
                  <label class="text-[10px] font-bold text-gray-500 uppercase tracking-tighter">Yearly Rate</label>
                  <input v-model="form.price_yearly" type="number" step="0.01" class="custom-input !mt-0 h-10" required />
                </div>
                <div class="md:col-span-4">
                   <div class="space-y-1">
                    <label class="text-[10px] font-bold text-gray-500 uppercase tracking-tighter">Quarterly Rate</label>
                    <input v-model="form.price_quarterly" type="number" step="0.01" class="custom-input !mt-0 h-10" required />
                  </div>
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <div class="flex justify-between items-center">
                <div class="flex items-center gap-2">
                  <i class="fas fa-concierge-bell text-primary text-xs"></i>
                  <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest">Amenities & Features</p>
                </div>
                <button type="button" @click="addAmenity" class="text-[10px] font-black text-primary uppercase tracking-widest hover:text-dprimary flex items-center gap-1.5 transition-colors">
                  <i class="fas fa-plus-circle"></i> Add Item
                </button>
              </div>

              <div class="space-y-3">
                <div v-for="(item, index) in amenities" :key="index" class="flex gap-3 items-center animate-in slide-in-from-left-2 duration-200">
                  <div class="flex-1 grid grid-cols-2 gap-3">
                    <input v-model="item.amenity" placeholder="e.g. Wi-Fi Speed" class="custom-input !mt-0 h-10 text-sm" />
                    <input v-model="item.value" placeholder="e.g. 100Mbps" class="custom-input !mt-0 h-10 text-sm" />
                  </div>
                  <button v-if="amenities.length > 1" type="button" @click="removeAmenity(index)" class="w-8 h-8 rounded-lg flex items-center justify-center text-red-400 hover:bg-red-50 transition-colors">
                    <i class="fas fa-trash-alt text-xs"></i>
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
            form="addSpaceForm"
            type="submit"
            :disabled="loading"
            class="custom-button !w-auto !px-10 flex items-center gap-2 disabled:opacity-50"
          >
            <i class="fas fa-save" v-if="!loading"></i>
            <i class="fas fa-spinner fa-spin" v-else></i>
            {{ loading ? "Synchronizing..." : "Create Space" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "../../../components/Toast.vue";

export default {
  name: "AddSpaceModal",
  components: { Toast },
  props: {
    visible: Boolean,
  },
  data() {
    return {
      form: {
        name: "",
        location: "",
        description: "",
        capacity: 1,
        price_daily: "",
        price_monthly: "",
        price_quarterly: "",
        price_yearly: "",
        zone: "",
        service_type:""
      },
       amenities: [
          { amenity: "", value: "" }
        ],
      zoneSearch: "",
      zones: [],
      zoneDropdown: false,
      loading: false, // Added loading state
    };
  },
  mounted() {
    this.fetchZones();
  },
  methods: {
     addAmenity() {
    this.amenities.push({ amenity: "", value: "" });
  },
  removeAmenity(index) {
    this.amenities.splice(index, 1);
  },
    async fetchZones() {
      try {
        const url = `/get_property_zones?search=${this.zoneSearch}`;
        const result = await this.$getZones(url);
        this.zones = result.zones;
      } catch (err) {
        console.error("Failed to fetch zones:", err);
      }
    },
    searchZones() {
      this.fetchZones();
    },
    selectZone(zone) {
      this.form.zone = zone.id;
      this.zoneSearch = zone.name;
      this.zoneDropdown = false;
    },
    hideDropdown(type) {
      setTimeout(() => {
        if (type === "zone") this.zoneDropdown = false;
      }, 200);
    },
    async submitForm() {
      this.loading = true;

    this.form.description = this.amenities
    .filter(a => a.amenity && a.value)
    .map(a => `${a.amenity.trim()}:"${a.value.trim()}"`)
    .join(",");


      try {
        const payload = { ...this.form };
        const res = await this.$apiPost("/post_coworking_space", payload);
        console.log("Space added:", res);

        if (res && res.error) {
          this.$root.$refs.toast.showToast(
            res.error || "Failed to add space",
            "error"
          );
        } else {
          this.$root.$refs.toast.showToast(
            "Co-working space added successfully!",
            "success"
          );
          this.resetForm();
          this.$emit("success");
          this.$reloadPage();
          setTimeout(() => this.$emit("close"), 1500);
        }
      } catch (err) {
        console.error("Failed to add space:", err);
        this.$root.$refs.toast.showToast(
          "Failed to add co-working space",
          "error"
        );
      } finally {
        this.loading = false;
      }
    },
    resetForm() {
      this.form = {
        name: "",
        location: "",
        description: "",
        capacity: 1,
        price_daily: "",
        price_monthly: "",
        price_quarterly: "",
        price_yearly: "",
        zone: "",
      };
      this.zoneSearch = "";
    },
  },
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500;
}
</style>
