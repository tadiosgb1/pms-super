<template>
  <div class="min-h-screen bg-gray-50/50 pb-12 mt-1 z-10">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading zone detail..." />

    <div class="bg-white border-b border-gray-200 sticky top-0 z-10 px-6 py-4">
      <div class="max-w-7xl mx-auto flex flex-col md:flex-row justify-between items-center gap-4">
        <div class="flex items-center gap-4">
          <button @click="$router.back()" class="p-2 hover:bg-gray-100 rounded-full transition-colors text-gray-500">
            <i class="fas fa-arrow-left"></i>
          </button>
          <div>
            <h1 v-if="zone" class="text-2xl font-black text-gray-800 tracking-tight">
              {{ zone.name }}
            </h1>
            <div class="flex items-center gap-2 text-[10px] uppercase tracking-widest font-bold text-primary">
              <i class="fas fa-map-marker-alt"></i>
              <span>Zone Management / Details</span>
            </div>
          </div>
        </div>

        <button
          @click="addPictureVisible = true"
          class="bg-gradient-to-r from-dprimary to-primary text-white text-xs font-black uppercase tracking-widest px-6 py-3 rounded-xl shadow-lg shadow-primary/20 hover:shadow-primary/40 transform hover:scale-[1.02] active:scale-95 transition-all flex items-center gap-2"
        >
          <i class="fas fa-camera"></i>
          Add Zone Picture
        </button>
      </div>
    </div>

    <div class="max-w-7xl mx-auto p-6 space-y-8">
      
      <div v-if="zone" class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        
        <div class="lg:col-span-2 bg-white rounded-3xl border border-gray-100 shadow-sm overflow-hidden">
          <div class="bg-gray-50/50 px-8 py-4 border-b border-gray-100 flex items-center justify-between">
            <span class="text-[11px] font-black text-gray-400 uppercase tracking-widest">Geographic Information</span>
            <i class="fas fa-globe-africa text-gray-300"></i>
          </div>
          <div class="p-8 grid grid-cols-1 md:grid-cols-2 gap-8">
            <div class="space-y-1">
              <p class="text-[10px] font-bold text-primary uppercase">Street Address</p>
              <p class="text-gray-700 font-semibold">{{ zone.address || 'N/A' }}</p>
            </div>
            <div class="space-y-1">
              <p class="text-[10px] font-bold text-primary uppercase">City & State</p>
              <p class="text-gray-700 font-semibold">{{ zone.city }}, {{ zone.state }}</p>
            </div>
            <div class="space-y-1">
              <p class="text-[10px] font-bold text-primary uppercase">Registration Date</p>
              <p class="text-gray-700 font-semibold flex items-center gap-2">
                <i class="far fa-calendar-alt text-gray-400"></i>
                {{ new Date(zone.created_at).toLocaleDateString() }}
              </p>
            </div>
          </div>
        </div>
        <div class="bg-white rounded-3xl border border-gray-100 shadow-sm overflow-hidden">
          <div class="bg-gray-50/50 px-8 py-4 border-b border-gray-100">
            <span class="text-[11px] font-black text-gray-400 uppercase tracking-widest">Ownership & Stakeholders</span>
          </div>
          <div class="p-8 space-y-6">
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 rounded-2xl bg-blue-50 flex items-center justify-center text-blue-600">
                <i class="fas fa-user-tie"></i>
              </div>
              <div>
                <p class="text-[10px] font-bold text-gray-400 uppercase">Manager ID</p>
                <p class="text-gray-800 font-black">#{{ zone.manager_id }}</p>
              </div>
            </div>
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 rounded-2xl bg-purple-50 flex items-center justify-center text-purple-600">
                <i class="fas fa-crown"></i>
              </div>
              <div>
                <p class="text-[10px] font-bold text-gray-400 uppercase">Owner ID</p>
                <p class="text-gray-800 font-black">#{{ zone.owner_id }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div>{{zone?.description}}</div>

      <div class="space-y-4">
        <div class="flex items-center justify-between">
          <h3 class="text-lg font-black text-gray-800 flex items-center gap-2">
            <i class="fas fa-images text-primary"></i>
            Zone Media Gallery
          </h3>
          <span class="bg-gray-200 text-gray-600 text-[10px] font-black px-3 py-1 rounded-full uppercase">
            {{ pictures.length }} Assets
          </span>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6">
          <div v-if="zone?.images"
            v-for="pic in zone.images"
            :key="pic.id"
            class="group relative bg-white p-2 rounded-2xl border border-gray-100 shadow-sm hover:shadow-xl transition-all duration-300"
          >
            <div class="relative aspect-video rounded-xl overflow-hidden bg-gray-100">
             
              <img
                :src="BASE_URL + pic.property_image"
                class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500 cursor-pointer"
                @click="previewImage(pic.property_image)"
              />
              
              <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center gap-2">
                <button
                  @click.stop="openUpdatePicture(pic)"
                  class="bg-white/20 backdrop-blur-md text-white p-2 rounded-lg hover:bg-white/40 transition-colors"
                  title="Update"
                >
                  <i class="fas fa-sync-alt text-sm"></i>
                </button>
                <button
                  @click.stop="askDeletePicture(pic)"
                  class="bg-red-500/80 backdrop-blur-md text-white p-2 rounded-lg hover:bg-red-600 transition-colors"
                  title="Delete"
                >
                  <i class="fas fa-trash-alt text-sm"></i>
                </button>
              </div>
            </div>
            
            <div class="mt-3 px-1 flex justify-between items-center">
              <span class="text-[10px] font-bold text-gray-400 uppercase">Captured Asset</span>
              <i class="fas fa-circle-check text-green-400 text-[10px]"></i>
            </div>
          </div>
        </div>
      </div>
    </div>

    <AddPropertyZonePicture v-if="addPictureVisible" :visible="addPictureVisible" :zoneId="$route.params.id" @close="addPictureVisible = false" @refresh="fetchPictures" />
    <UpdatePropertyZonePicture v-if="updatePictureVisible" :visible="updatePictureVisible" :picture="pictureToUpdate" :zoneId="$route.params.id" @close="updatePictureVisible = false" @refresh="fetchPictures" />
    <ConfirmModal v-if="confirmDeleteVisible" :visible="confirmDeleteVisible" title="Delete Media" message="This action will permanently remove this image from the zone records. Proceed?" @confirm="confirmDeletePicture" @cancel="confirmDeleteVisible = false" />

    <div
      v-if="imagePreviewVisible"
      class="fixed inset-0 z-[100] bg-gray-950/95 backdrop-blur-md flex items-center justify-center p-4"
      @click="imagePreviewVisible = false"
    >
      <div class="relative max-w-5xl w-full animate-in zoom-in duration-300">
        <img :src="BASE_URL + imageToPreview" class="w-full h-auto max-h-[85vh] object-contain rounded-2xl shadow-2xl shadow-black/50 border border-white/10" @click.stop />
        <button @click="imagePreviewVisible = false" class="absolute -top-12 right-0 text-white/70 hover:text-white text-xl flex items-center gap-2 font-bold transition-all">
          <span>Close</span> <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
  </div>
</template>
<script>
import Toast from "@/components/Toast.vue";
import AddPropertyZonePicture from "./addPropertyZonePicture.vue";
import UpdatePropertyZonePicture from "./updatePropertyZonePicture.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  name: "ZoneDetail",
  components: { Toast, AddPropertyZonePicture, UpdatePropertyZonePicture, ConfirmModal,Loading },
  data() {
    return {
      zone: null,
      pictures: [],
      addPictureVisible: false,
      updatePictureVisible: false,
      confirmDeleteVisible: false,
      pictureToUpdate: null,
      pictureToDelete: null,
      imagePreviewVisible: false,
      imageToPreview: null,
      loading:false,
      BASE_URL: import.meta.env.VITE_BASE_SERVER_URL
    };
  },
  mounted() {
    this.fetchZone();
  },
  methods: {

    async fetchZone() {
      this.loading=true;
      try {
        const res = await this.$apiGet(`/get_property_zone/${this.$route.params.id}`);
        this.zone = res.data || res;
      } catch (err) {
        console.error("Failed to fetch zone details", err);
        this.$refs.toast.showToast("Could not load zone details.", "error");
      }finally{
        this.loading=false;
      }
    },

    openUpdatePicture(pic) {
      this.pictureToUpdate = pic;
      this.updatePictureVisible = true;
    },
    askDeletePicture(pic) {
      this.pictureToDelete = pic;
      this.confirmDeleteVisible = true;
    },
    async confirmDeletePicture() {
      this.confirmDeleteVisible = false;
      if (!this.pictureToDelete) return;
      try {
        const res = await this.$apiDelete(`/delete_property_zone_picture/${this.pictureToDelete.id}`);
        this.$root.$refs.toast.showToast(res.message || "Picture deleted", "success");
        this.fetchPictures();
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast("Failed to delete picture", "error");
      }
      this.pictureToDelete = null;
    },
previewImage(img) {
  this.imageToPreview = img; // no BASE_URL added here
  this.imagePreviewVisible = true;
}
  },
};
</script>
