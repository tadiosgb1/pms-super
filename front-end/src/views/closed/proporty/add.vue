<template>
  <Teleport to="body">
    <div v-if="visible" class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4">
      <div class="bg-white w-full max-w-3xl rounded-xl shadow-xl flex flex-col max-h-[92vh]">

        <!-- Header -->
        <div class="flex items-center justify-between px-5 py-4 border-b border-gray-100 shrink-0">
          <h2 class="text-base font-semibold text-gray-800">Add Property</h2>
          <button @click="$emit('close')" class="text-gray-400 hover:text-gray-600 text-xl leading-none">&times;</button>
        </div>

        <!-- Body -->
        <div class="overflow-y-auto p-5 space-y-6">
          <form id="addPropertyForm" @submit.prevent="submitForm">

            <!-- Basic -->
            <section class="space-y-3 mb-6">
              <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider border-b border-gray-100 pb-1">Basic Info</p>
              <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                <div class="sm:col-span-3">
                  <label class="form-label">Property Name <span class="text-red-400">*</span></label>
                  <input v-model="form.name" type="text" required placeholder="e.g. Sunset Villa" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Type <span class="text-red-400">*</span></label>
                  <select v-model="form.property_type" required class="form-input">
                    <option value="" disabled>Select</option>
                    <option value="land">Land</option>
                    <option value="house">House</option>
                    <option value="apartment">Apartment</option>
                    <option value="room">Room</option>
                  </select>
                </div>
                <div>
                  <label class="form-label">Zone <span class="text-red-400">*</span></label>
                  <select v-model="form.property_zone_id" required class="form-input">
                    <option value="" disabled>Select zone</option>
                    <option v-for="z in zones" :key="z.id" :value="z.id">{{ z.name }}</option>
                  </select>
                </div>
                <div>
                  <label class="form-label">Manager</label>
                  <select v-model="form.manager_id" class="form-input">
                    <option value="">None</option>
                    <option v-for="m in managers" :key="m.manager?.id || m.id" :value="m.manager?.id || m.id">
                      {{ m.manager?.first_name || m.first_name }} {{ m.manager?.last_name || m.last_name }}
                    </option>
                  </select>
                </div>
              </div>
            </section>

            <!-- Location -->
            <section class="space-y-3 mb-6">
              <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider border-b border-gray-100 pb-1">Location</p>
              <div class="grid grid-cols-2 sm:grid-cols-3 gap-3">
                <div class="col-span-2 sm:col-span-3">
                  <label class="form-label">Address</label>
                  <input v-model="form.address" type="text" placeholder="Street, house no." class="form-input" />
                </div>
                <div>
                  <label class="form-label">City</label>
                  <input v-model="form.city" type="text" placeholder="City" class="form-input" />
                </div>
                <div>
                  <label class="form-label">State</label>
                  <input v-model="form.state" type="text" placeholder="State" class="form-input" />
                </div>
                <div>
                  <label class="form-label">ZIP Code</label>
                  <input v-model="form.zip_code" type="text" placeholder="ZIP" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Block #</label>
                  <input v-model="form.block_number" type="text" placeholder="Block" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Floor #</label>
                  <input v-model="form.floor_number" type="text" placeholder="Floor" class="form-input" />
                </div>
                <div>
                  <label class="form-label">House #</label>
                  <input v-model="form.house_number" type="text" placeholder="House" class="form-input" />
                </div>
              </div>
            </section>

            <!-- Specs & Pricing -->
            <section class="space-y-3 mb-6">
              <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider border-b border-gray-100 pb-1">Specs & Pricing</p>
              <div class="grid grid-cols-2 sm:grid-cols-3 gap-3">
                <div>
                  <label class="form-label">Area (sqft)</label>
                  <input v-model="form.area" type="text" placeholder="0" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Bedrooms</label>
                  <input v-model.number="form.bed_rooms" type="number" placeholder="0" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Bathrooms</label>
                  <input v-model.number="form.bath_rooms" type="number" placeholder="0" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Monthly Rent</label>
                  <input v-model.number="form.rent" type="number" placeholder="0" class="form-input" />
                </div>
                <div>
                  <label class="form-label">Selling Price</label>
                  <input v-model.number="form.price" type="number" placeholder="0" class="form-input" />
                </div>
              </div>
            </section>

            <!-- Amenities -->
            <section class="space-y-3 mb-6">
              <div class="flex items-center justify-between border-b border-gray-100 pb-1">
                <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider">Amenities</p>
                <button type="button" @click="addAmenity"
                  class="text-xs text-primary font-semibold hover:underline flex items-center gap-1">
                  <i class="fas fa-plus text-[10px]"></i> Add
                </button>
              </div>
              <div v-for="(item, i) in amenities" :key="i" class="flex gap-2">
                <input v-model="item.amenity" placeholder="Feature (e.g. Wifi)" class="form-input flex-1" />
                <input v-model="item.value" placeholder="Value (e.g. Free)" class="form-input flex-1" />
                <button v-if="amenities.length > 1" type="button" @click="removeAmenity(i)"
                  class="text-red-400 hover:text-red-600 transition px-1">
                  <i class="fas fa-times text-xs"></i>
                </button>
              </div>
            </section>

            <!-- Description -->
            <section class="space-y-2">
              <p class="text-xs font-semibold text-gray-400 uppercase tracking-wider border-b border-gray-100 pb-1">Description</p>
              <textarea v-model="form.content" rows="3" placeholder="Describe the property..."
                class="form-input resize-none"></textarea>
            </section>

          </form>
        </div>

        <!-- Footer -->
        <div class="flex justify-end gap-3 px-5 py-4 border-t border-gray-100 bg-gray-50 rounded-b-xl shrink-0">
          <button type="button" @click="$emit('close')" class="btn-cancel">Cancel</button>
          <button form="addPropertyForm" type="submit" class="btn-primary">Save Property</button>
        </div>

      </div>
    </div>
  </Teleport>
</template>

<script>
import Toast from "@/components/Toast.vue";

export default {
  name: "AddProperty",
  components: { Toast },
  props: { visible: Boolean },
  data() {
    return {
      managers: [], zones: [],
      amenities: [{ amenity: "", value: "" }],
      form: {
        property_zone_id: "", owner_id: localStorage.getItem("userId"),
        manager_id: "", name: "", property_type: "", address: "",
        city: "", state: "", zip_code: "", price: "", bed_rooms: "",
        bath_rooms: "", rent: "", content: "", status: "available",
        area: "", description: "", block_number: "", floor_number: "", house_number: "",
      },
    };
  },
  async mounted() {
    const isSuperuser = localStorage.getItem("is_superuser") === "true";
    const mRes = isSuperuser
      ? await this.$apiGet("/get_managers")
      : await this.$apiGet("/get_owner_managers", { owner__id: localStorage.getItem("userId") });
    this.managers = mRes.data || [];
    const zRes = await this.$getZones();
    this.zones = zRes.zones || [];
  },
  methods: {
    addAmenity() { this.amenities.push({ amenity: "", value: "" }); },
    removeAmenity(i) { this.amenities.splice(i, 1); },
    async submitForm() {
      this.form.description = this.amenities
        .filter(a => a.amenity && a.value)
        .map(a => `${a.amenity.trim()}:"${a.value.trim()}"`)
        .join(",");
      try {
        await this.$apiPost("/post_property", { ...this.form });
        this.$root.$refs.toast.showToast("Property saved successfully", "success");
        this.$emit("refresh");
        this.$emit("close");
      } catch (e) {
        this.$root.$refs.toast.showToast(e.message || "Failed to save", "error");
      }
    },
  },
};
</script>

<style scoped>
.form-label { @apply block text-xs font-medium text-gray-600 mb-1; }
.form-input  { @apply w-full border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary/30 bg-white; }
.btn-cancel  { @apply px-4 py-2 text-sm text-gray-500 border border-gray-200 rounded-lg hover:bg-gray-100 transition; }
.btn-primary { @apply px-5 py-2 text-sm font-semibold text-white bg-primary rounded-lg hover:bg-primary/90 transition; }
</style>
