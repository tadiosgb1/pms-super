<template>
  <div class="min-h-screen bg-background p-4 md:p-8">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Fetching space details..." />

    <div v-if="space" class="max-w-6xl mx-auto space-y-6 animate-in fade-in slide-in-from-bottom-4 duration-500">
      
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <button @click="$router.back()" class="text-[10px] font-black text-gray-400 uppercase tracking-widest hover:text-primary transition-colors flex items-center gap-2 mb-1">
            <i class="fas fa-arrow-left"></i> Back to Inventory
          </button>
          <h1 class="text-3xl font-black text-gray-800 tracking-tight flex items-center gap-3">
            {{ space.name }}
            <span class="text-xs font-bold bg-primary/10 text-primary px-3 py-1 rounded-full uppercase tracking-tighter">
              ID: #{{ space.id }}
            </span>
          </h1>
        </div>
        
        <div class="flex items-center gap-3">
          <button
            @click="openEdit"
            class="px-5 py-2.5 bg-white border border-gray-200 rounded-xl text-[11px] font-black text-gray-600 uppercase tracking-widest hover:bg-gray-50 hover:shadow-md transition-all flex items-center gap-2"
          >
            <i class="fas fa-edit text-blue-500"></i> Edit Details
          </button>
          <button
            @click="addPictureVisible = true"
            class="px-5 py-2.5 bg-primary text-white rounded-xl text-[11px] font-black uppercase tracking-widest shadow-lg shadow-primary/20 hover:bg-dprimary hover:-translate-y-0.5 transition-all flex items-center gap-2"
          >
            <i class="fas fa-camera"></i> Add Media
          </button>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <div class="lg:col-span-2 space-y-6">
          
          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-8">
            <div class="flex items-center gap-2 mb-6 border-b border-gray-50 pb-4">
              <i class="fas fa-info-circle text-primary"></i>
              <h2 class="text-sm font-black text-gray-700 uppercase tracking-widest">General Overview</h2>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
              <div class="space-y-4">
                <div>
                  <p class="text-[10px] font-black text-gray-400 uppercase tracking-tighter">Exact Location</p>
                  <p class="text-gray-700 font-bold flex items-center gap-2 mt-1">
                    <i class="fas fa-map-marker-alt text-red-400"></i> {{ space.location }}
                  </p>
                </div>
                <div>
                  <p class="text-[10px] font-black text-gray-400 uppercase tracking-tighter">Max Occupancy</p>
                  <p class="text-gray-700 font-bold flex items-center gap-2 mt-1">
                    <i class="fas fa-users text-blue-400"></i> {{ space.capacity }} Seats Available
                  </p>
                </div>
              </div>

              <div class="space-y-4 bg-gray-50 p-4 rounded-xl border border-gray-100">
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-tighter mb-2">Assigned Zone</p>
                <div class="flex items-center justify-between">
                  <span class="text-sm font-black text-gray-800">{{ space.zone?.name }}</span>
                  <button @click="goToZoneDetail(space.zone.id)" class="text-[9px] font-black bg-white px-3 py-1.5 rounded-lg border border-gray-200 text-primary hover:bg-primary hover:text-white transition-all uppercase">
                    Open Zone <i class="fas fa-external-link-alt ml-1"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-8">
            <div class="flex items-center gap-2 mb-6 border-b border-gray-50 pb-4">
              <i class="fas fa-concierge-bell text-primary"></i>
              <h2 class="text-sm font-black text-gray-700 uppercase tracking-widest">Amenities & Services</h2>
            </div>

            <div v-if="amenities.length" class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div
                v-for="(item, index) in amenities"
                :key="index"
                class="flex items-center justify-between p-4 bg-gray-50 rounded-xl border border-transparent hover:border-primary/20 transition-all group"
              >
                <div class="flex items-center gap-3">
                  <div class="w-2 h-2 rounded-full bg-primary group-hover:scale-150 transition-transform"></div>
                  <span class="text-xs font-bold text-gray-500 uppercase tracking-tight">{{ item.amenity }}</span>
                </div>
                <span class="text-sm font-black text-gray-800">{{ item.value }}</span>
              </div>
            </div>
            <div v-else class="text-center py-10 bg-gray-50 rounded-2xl border-2 border-dashed border-gray-200">
              <i class="fas fa-clipboard-list text-gray-300 text-3xl mb-3"></i>
              <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">No amenities configured</p>
            </div>
          </div>

          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-8">
            <div class="flex items-center gap-2 mb-6 border-b border-gray-50 pb-4">
              <i class="fas fa-images text-primary"></i>
              <h2 class="text-sm font-black text-gray-700 uppercase tracking-widest">Visual Assets</h2>
            </div>

            <div v-if="space.pictures.length" class="grid grid-cols-2 md:grid-cols-3 gap-4">
              <div
                v-for="pic in space.pictures"
                :key="pic.id"
                class="group relative aspect-video rounded-xl overflow-hidden cursor-pointer shadow-sm"
                @click="previewImage(getImageUrl(pic.image))"
              >
                <img :src="getImageUrl(pic.image)" class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110" />
                <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center">
                   <i class="fas fa-search-plus text-white text-xl"></i>
                </div>
              </div>
            </div>
            <div v-else class="text-center py-10 bg-gray-50 rounded-2xl border-2 border-dashed border-gray-200">
              <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">Gallery Empty</p>
            </div>
          </div>
        </div>

        <div class="space-y-6">
          
          <div class="bg-gradient-to-br from-gray-900 to-gray-800 rounded-2xl shadow-xl p-8 text-white relative overflow-hidden">
             <div class="relative z-10">
                <div class="flex items-center gap-2 mb-6 pb-4 border-b border-white/10">
                  <i class="fas fa-coins text-primary"></i>
                  <h2 class="text-sm font-black uppercase tracking-widest">Pricing Matrix</h2>
                </div>
                
                <div class="space-y-4">
                  <div class="flex justify-between items-center bg-white/5 p-3 rounded-xl border border-white/5">
                    <span class="text-[10px] font-black text-gray-400 uppercase">Daily Rate</span>
                    <span class="text-lg font-black text-primary">{{ space.price_daily }}</span>
                  </div>
                  <div class="flex justify-between items-center bg-white/5 p-3 rounded-xl border border-white/5">
                    <span class="text-[10px] font-black text-gray-400 uppercase">Monthly Rate</span>
                    <span class="text-lg font-black text-primary">{{ space.price_monthly }}</span>
                  </div>
                  <div class="flex justify-between items-center bg-white/5 p-3 rounded-xl border border-white/5">
                    <span class="text-[10px] font-black text-gray-400 uppercase">Quarterly Rate</span>
                    <span class="text-lg font-black text-primary">{{ space.price_quarterly }}</span>
                  </div>
                  <div class="flex justify-between items-center bg-white/5 p-3 rounded-xl border border-white/5">
                    <span class="text-[10px] font-black text-gray-400 uppercase">Yearly Rate</span>
                    <span class="text-lg font-black text-primary">{{ space.price_yearly }}</span>
                  </div>
                </div>
             </div>
             <i class="fas fa-wallet absolute -bottom-4 -right-4 text-8xl text-white/5 -rotate-12"></i>
          </div>

          <div class="bg-white rounded-2xl shadow-sm border border-gray-100 p-6">
            <h2 class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-4">System Metadata</h2>
            <div class="space-y-3">
              <div class="flex justify-between text-xs">
                <span class="text-gray-400 font-bold uppercase">Created</span>
                <span class="text-gray-700 font-black">{{ formatDate(space.created_at) }}</span>
              </div>
              <div class="flex justify-between text-xs">
                <span class="text-gray-400 font-bold uppercase">Last Sync</span>
                <span class="text-gray-700 font-black">{{ formatDate(space.updated_at) }}</span>
              </div>
            </div>
          </div>

        </div>
      </div>
    </div>

    <UpdateCoworkspace :visible="updateVisible" :space="spaceToEdit" @close="updateVisible = false" @refresh="fetchSpace" />
    <AddPropertySpacePicture v-if="addPictureVisible" :visible="addPictureVisible" :spaceId="$route.params.id" @close="addPictureVisible = false" @refresh="fetchSpace" />

    <div v-if="imagePreviewVisible" class="fixed inset-0 z-[100] flex items-center justify-center bg-black/95 backdrop-blur-md p-4 animate-in fade-in duration-300" @click="imagePreviewVisible = false">
      <div class="relative max-w-5xl w-full h-full flex items-center justify-center">
        <img :src="imageToPreview" class="max-h-full max-w-full rounded-xl shadow-2xl object-contain border border-white/10" @click.stop />
        <button @click="imagePreviewVisible = false" class="absolute top-0 right-0 p-4 text-white text-2xl hover:text-primary transition-colors">
          <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import AddPropertySpacePicture from "./spcaePicture.vue";
import UpdateCoworkspace from "./update.vue";

export default {
  name: "CoworkingSpaceDetail",

  components: {
    AddPropertySpacePicture,
    UpdateCoworkspace,
  },

  data() {
    return {
      space: { pictures: [] },
      loading: true,
      error: null,

      addPictureVisible: false,

      // edit modal
      updateVisible: false,
      spaceToEdit: null,

      // image preview
      imagePreviewVisible: false,
      imageToPreview: null,
    };
  },

  computed: {
    amenities() {
      if (!this.space.description) return [];

      return this.space.description.split(",").map(pair => {
        const [key, value] = pair.split(":");
        return {
          amenity: key?.trim(),
          value: value?.replace(/"/g, "").trim(),
        };
      });
    },
  },

  mounted() {
    this.fetchSpace();
  },

  methods: {
    openEdit() {
      this.spaceToEdit = { ...this.space };
      this.updateVisible = true;
    },

    getImageUrl(imagePath) {
      if (!imagePath) return "";
      const apiBase = import.meta.env.VITE_APP_BASE_URL_LOCAL;
      return `${apiBase.replace("/api", "")}${imagePath}`;
    },

    previewImage(img) {
      this.imageToPreview = img;
      this.imagePreviewVisible = true;
    },

    goToZoneDetail(id) {
      this.$router.push(`/zones/${id}`);
    },

    async fetchSpace() {
      this.loading = true;
      try {
        const id = this.$route.params.id;
        const res = await this.$apiGetById(`/get_coworking_space`, id);
        this.space = res;
      } catch (err) {
        console.error(err);
        this.error = "Failed to load coworking space details.";
      } finally {
        this.loading = false;
      }
    },

    formatDate(dateStr) {
      return dateStr ? new Date(dateStr).toLocaleString() : "N/A";
    },
  },
};
</script>
