<template>
  <div class="p-4 md:p-6 bg-gray-50 min-h-screen">
    <Loading :visible="loading" message="Loading property..." />

    <div v-if="property" class="max-w-6xl mx-auto space-y-5">

      <!-- Page header -->
      <div class="flex flex-wrap items-center gap-3 justify-between">
        <div class="flex items-center gap-3">
          <button @click="$router.go(-1)"
            class="flex items-center gap-1 text-sm text-gray-500 hover:text-primary transition">
            <i class="fas fa-arrow-left text-xs"></i> Back
          </button>
          <h1 class="text-lg font-semibold text-gray-800">{{ property.name }}</h1>
          <span class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase"
            :class="{
              'bg-green-100 text-green-700': property.status === 'available',
              'bg-blue-100 text-blue-700': property.status === 'rent',
              'bg-orange-100 text-orange-700': property.status === 'sale',
              'bg-pink-100 text-pink-700': property.status === 'under_maintenance'
            }">
            {{ property.status }}
          </span>
        </div>
        <button v-if="$hasPermission('pms.change_property')" @click="editProperty(property)"
          class="flex items-center gap-2 text-sm text-blue-600 border border-blue-200 bg-blue-50 px-4 py-2 rounded-lg hover:bg-blue-100 transition">
          <i class="fas fa-edit text-xs"></i> Edit
        </button>
      </div>

      <!-- Stat cards -->
      <div class="grid grid-cols-2 sm:grid-cols-4 gap-3">
        <div class="bg-white rounded-xl border border-gray-100 p-4">
          <p class="text-xs text-gray-400 font-medium mb-1">Monthly Rent</p>
          <p class="text-lg font-bold text-primary">{{ property.rent || '—' }}</p>
        </div>
        <div class="bg-white rounded-xl border border-gray-100 p-4">
          <p class="text-xs text-gray-400 font-medium mb-1">Selling Price</p>
          <p class="text-lg font-bold text-orange-600">{{ property.price?.toLocaleString() || '—' }}</p>
        </div>
        <div class="bg-white rounded-xl border border-gray-100 p-4">
          <p class="text-xs text-gray-400 font-medium mb-1">Type</p>
          <p class="text-base font-bold text-gray-700 capitalize">{{ property.property_type || '—' }}</p>
        </div>
        <div class="bg-white rounded-xl border border-gray-100 p-4">
          <p class="text-xs text-gray-400 font-medium mb-1">Rooms</p>
          <p class="text-base font-bold text-gray-700">{{ property.bed_rooms || 0 }} BD · {{ property.bath_rooms || 0 }} BA</p>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-5">

        <!-- Left: Details + Amenities + Maintenance -->
        <div class="lg:col-span-2 space-y-5">

          <!-- Specifications -->
          <div class="bg-white rounded-xl border border-gray-100 shadow-sm overflow-hidden">
            <div class="px-5 py-3 bg-gray-50 border-b border-gray-100">
              <h2 class="text-sm font-semibold text-gray-700">Specifications</h2>
            </div>
            <div class="p-5 grid grid-cols-2 gap-x-8 gap-y-3">
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">City</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.city || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">State</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.state || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">ZIP</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.zip_code || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Area</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.area || '—' }} sqft</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Block #</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.block_number || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Floor #</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.floor_number || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">House #</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.house_number || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Address</span>
                <span class="text-xs font-semibold text-gray-700 text-right max-w-[140px]">{{ property.address || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Created</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.created_at || '—' }}</span>
              </div>
              <div class="flex justify-between border-b border-gray-50 pb-2">
                <span class="text-xs text-gray-400">Updated</span>
                <span class="text-xs font-semibold text-gray-700">{{ property.updated_at || '—' }}</span>
              </div>
            </div>
          </div>

          <!-- Amenities -->
          <div v-if="amenities.length" class="bg-white rounded-xl border border-gray-100 shadow-sm overflow-hidden">
            <div class="px-5 py-3 bg-gray-50 border-b border-gray-100">
              <h2 class="text-sm font-semibold text-gray-700">Amenities</h2>
            </div>
            <div class="p-5 grid grid-cols-1 sm:grid-cols-2 gap-2">
              <div v-for="(item, i) in amenities" :key="i"
                class="flex justify-between items-center px-3 py-2 bg-gray-50 rounded-lg border border-gray-100">
                <span class="text-xs text-gray-600 font-medium">{{ item.amenity }}</span>
                <span class="text-xs text-primary font-semibold">{{ item.value }}</span>
              </div>
            </div>
          </div>

          <!-- Maintenance -->
          <div class="bg-white rounded-xl border border-gray-100 shadow-sm overflow-hidden">
            <div class="px-5 py-3 bg-gray-50 border-b border-gray-100">
              <h2 class="text-sm font-semibold text-gray-700">Maintenance History</h2>
            </div>
            <div class="p-2">
              <Maintenance :visible="true" :propertyId="property.id" @close="() => {}" />
            </div>
          </div>

        </div>

        <!-- Right: Gallery -->
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <h2 class="text-sm font-semibold text-gray-700">Photos</h2>
            <button @click="addPictureVisible = true"
              class="text-xs font-semibold text-primary border border-primary/30 bg-primary/5 px-3 py-1 rounded-lg hover:bg-primary/10 transition">
              <i class="fas fa-plus text-[10px] mr-1"></i> Add
            </button>
          </div>

          <div v-if="property.property_pictures?.length" class="space-y-3">
            <div v-for="pic in visiblePictures" :key="pic.id"
              class="group rounded-xl overflow-hidden border border-gray-100 bg-white shadow-sm">
              <div class="aspect-video overflow-hidden relative">
                <img :src="pic.property_image" :alt="pic.description"
                  class="object-cover w-full h-full cursor-pointer transition-transform duration-300 group-hover:scale-105"
                  @click="previewImage(pic.property_image)" />
                <div class="absolute top-2 right-2 flex gap-1 opacity-0 group-hover:opacity-100 transition">
                  <button @click.stop="openUpdatePicture(pic)"
                    class="w-7 h-7 bg-white rounded-full text-blue-600 text-xs shadow flex items-center justify-center hover:bg-blue-600 hover:text-white transition">
                    <i class="fas fa-pen"></i>
                  </button>
                  <button @click.stop="askDeletePicture(pic)"
                    class="w-7 h-7 bg-white rounded-full text-red-600 text-xs shadow flex items-center justify-center hover:bg-red-600 hover:text-white transition">
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </div>
              <p class="text-xs text-gray-500 px-3 py-2 truncate">{{ pic.description }}</p>
            </div>

            <button v-if="remainingPicturesCount > 0 && !showAllPictures"
              @click="showAllPictures = true"
              class="w-full py-3 border border-dashed border-gray-200 rounded-xl text-xs text-gray-400 font-semibold hover:border-primary hover:text-primary transition">
              +{{ remainingPicturesCount }} more
            </button>
          </div>

          <div v-else class="rounded-xl border border-dashed border-gray-200 p-10 text-center bg-white">
            <i class="fas fa-images text-gray-200 text-3xl mb-2 block"></i>
            <p class="text-xs text-gray-400">No photos yet</p>
          </div>
        </div>

      </div>
    </div>

    <div v-else-if="!loading" class="text-center py-20 text-sm text-gray-400 italic">Property not found.</div>

    <!-- Modals -->
    <AddPictureModal v-if="addPictureVisible && property" :visible="addPictureVisible"
      :propertyId="property.id" :propertyName="property.name"
      @close="addPictureVisible = false" @refresh="fetchProperty" />
    <UpdatePictureModal v-if="updatePictureVisible && property" :visible="updatePictureVisible"
      :picture="pictureToUpdate" :propertyId="property.id"
      @close="updatePictureVisible = false" @refresh="fetchProperty" />
    <ConfirmModal v-if="confirmDeleteVisible" :visible="confirmDeleteVisible"
      title="Delete Photo" message="Delete this photo?"
      @confirm="confirmDeletePicture" @cancel="confirmDeleteVisible = false" />
    <UpdateProperty v-if="updateVisible" :visible="updateVisible" :property="propertyToEdit"
      @close="updateVisible = false" @refresh="fetchProperty" />

    <!-- Image preview lightbox -->
    <div v-if="imagePreviewVisible"
      class="fixed inset-0 bg-black/90 flex items-center justify-center z-[100]"
      @click="imagePreviewVisible = false">
      <img :src="imageToPreview" class="max-h-[88vh] max-w-[92vw] rounded-xl shadow-2xl" @click.stop />
      <button @click="imagePreviewVisible = false"
        class="absolute top-5 right-5 w-10 h-10 bg-white/10 hover:bg-white/20 text-white rounded-full flex items-center justify-center transition">
        <i class="fas fa-times"></i>
      </button>
    </div>
  </div>
</template>

<script>
import AddPictureModal from "@/views/closed/proporty/AddPropertyPicture.vue";
import UpdatePictureModal from "@/views/closed/proporty/UpdatePropertyPicture.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import UpdateProperty from "@/views/closed/proporty/update.vue";
import Toast from "@/components/Toast.vue";
import Maintenance from "@/views/closed/maintenanceRequests/view1.vue";
import Loading from "@/components/Loading.vue";

export default {
  name: "PropertyDetail",
  components: { AddPictureModal, UpdatePictureModal, UpdateProperty, ConfirmModal, Toast, Maintenance, Loading },
  data() {
    return {
      property: null, loading: false,
      addPictureVisible: false, updatePictureVisible: false, pictureToUpdate: null,
      confirmDeleteVisible: false, pictureToDelete: null,
      showAllPictures: false, imagePreviewVisible: false, imageToPreview: null,
      updateVisible: false, propertyToEdit: null,
    };
  },
  computed: {
    amenities() {
      if (!this.property?.description) return [];
      return this.property.description.split(",").map(pair => {
        const [k, v] = pair.split(":");
        return { amenity: k?.trim() || "", value: v?.replace(/"/g, "").trim() || "" };
      }).filter(a => a.amenity);
    },
    visiblePictures() {
      const pics = this.property?.property_pictures || [];
      return this.showAllPictures ? pics : pics.slice(0, 3);
    },
    remainingPicturesCount() {
      return Math.max(0, (this.property?.property_pictures?.length || 0) - 3);
    },
  },
  mounted() { this.fetchProperty(); },
  methods: {
    editProperty(p) { this.propertyToEdit = p; this.updateVisible = true; },
    async fetchProperty() {
      this.loading = true;
      try {
        const res = await this.$apiGet(`/get_property/${this.$route.params.id}`);
        this.property = res.data || res;
      } catch (e) { console.error(e); } finally { this.loading = false; }
    },
    openUpdatePicture(pic) { this.pictureToUpdate = pic; this.updatePictureVisible = true; },
    askDeletePicture(pic) { this.pictureToDelete = pic; this.confirmDeleteVisible = true; },
    async confirmDeletePicture() {
      this.confirmDeleteVisible = false;
      if (!this.pictureToDelete) return;
      try {
        await this.$apiDelete(`/delete_property_picture/${this.pictureToDelete.id}`);
        this.$root.$refs.toast.showToast("Photo deleted", "success");
        this.fetchProperty();
      } catch (e) { this.$root.$refs.toast.showToast("Failed to delete", "error"); }
      this.pictureToDelete = null;
    },
    previewImage(url) { this.imageToPreview = url; this.imagePreviewVisible = true; },
  },
};
</script>
