<template>
  <div class="min-h-screen bg-slate-50/50 pb-12">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Syncing agreement data..." />

    <div class="bg-white border-b border-gray-100 px-6 py-4 flex items-center justify-between sticky top-0 z-30 shadow-sm">
      <button
        @click="$router.back()"
        class="flex items-center gap-2 text-[10px] font-black uppercase tracking-[0.2em] text-gray-400 hover:text-primary transition-colors"
      >
        <i class="fas fa-arrow-left"></i> Back to Fleet
      </button>
      
      <div class="flex items-center gap-3">
        <button
          @click="editRent(rent)"
          class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest text-secondary bg-secondary/5 hover:bg-secondary hover:text-white transition-all"
        >
          <i class="fas fa-edit mr-1"></i> Edit Agreement
        </button>
        <button
          @click="addPictureVisible = true"
          class="px-4 py-2 rounded-xl text-[10px] font-black uppercase tracking-widest text-white bg-primary shadow-lg shadow-primary/20 hover:bg-dprimary transition-all"
        >
          <i class="fas fa-camera mr-1"></i> Add Picture
        </button>
      </div>
    </div>

    <div v-if="rent" class="max-w-7xl mx-auto p-6 animate-in fade-in slide-in-from-bottom-4 duration-500">
      
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        
        <div class="lg:col-span-1 space-y-6">
          <div class="bg-white rounded-[2.5rem] shadow-sm border border-gray-100 overflow-hidden">
            <div class="bg-gradient-to-br from-dprimary to-primary p-8 text-white">
              <div class="text-[10px] font-black uppercase tracking-[0.3em] opacity-60 mb-2">Lease Status</div>
              <div class="flex items-center justify-between">
                <h1 class="text-2xl font-black tracking-tight uppercase">{{ rent.status }}</h1>
                <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center backdrop-blur-md">
                  <i class="fas fa-shield-alt"></i>
                </div>
              </div>
            </div>

            <div class="p-8">
              <ul class="space-y-6">
                <li class="group">
                  <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Asset Assigned</p>
                  <p class="text-sm font-bold text-gray-700 group-hover:text-primary transition-colors cursor-default">
                    {{ rent.property_id?.name || 'N/A' }}
                  </p>
                </li>
                <li class="group border-t border-gray-50 pt-4">
                  <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Primary Tenant</p>
                  <p class="text-sm font-bold text-gray-700">
                    {{ rent.user_id?.first_name }} {{ rent.user_id?.last_name }}
                  </p>
                </li>
                <li class="group border-t border-gray-50 pt-4">
                  <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest mb-1">Agreement Type</p>
                  <span class="inline-block px-3 py-1 bg-gray-100 rounded-full text-[10px] font-black text-gray-500 uppercase">
                    {{ rent.rent_type }}
                  </span>
                </li>
              </ul>
            </div>
          </div>

          <button 
            @click="viewPayments(rent.id)"
            class="w-full bg-white border border-gray-100 p-6 rounded-[2rem] flex items-center justify-between group hover:border-secondary transition-all shadow-sm"
          >
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 bg-secondary/10 rounded-2xl flex items-center justify-center text-secondary group-hover:scale-110 transition-transform">
                <i class="fas fa-receipt"></i>
              </div>
              <div class="text-left">
                <p class="text-xs font-black uppercase tracking-widest text-gray-700">Financial Ledger</p>
                <p class="text-[10px] font-bold text-gray-400">View payment history</p>
              </div>
            </div>
            <i class="fas fa-chevron-right text-gray-300 group-hover:text-secondary transition-colors"></i>
          </button>
        </div>

        <div class="lg:col-span-2 space-y-8">
          
          <div class="bg-white rounded-[2.5rem] p-8 border border-gray-100 shadow-sm grid grid-cols-2 md:grid-cols-4 gap-6">
            <div class="space-y-1">
              <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest">Start Date</p>
              <p class="text-sm font-bold text-gray-700">{{ rent.start_date }}</p>
            </div>
            <div class="space-y-1">
              <p class="text-[9px] font-black text-gray-400 uppercase tracking-widest">End Date</p>
              <p class="text-sm font-bold text-gray-700">{{ rent.end_date || 'Ongoing' }}</p>
            </div>
            <div class="space-y-1">
              <p class="text-[9px] font-black text-secondary uppercase tracking-widest">Rent Amount</p>
              <p class="text-lg font-black text-gray-800">${{ rent.rent_amount }}</p>
            </div>
            <div class="space-y-1">
              <p class="text-[9px] font-black text-primary uppercase tracking-widest">Cycle</p>
              <p class="text-sm font-bold text-gray-700 capitalize">{{ rent.payment_cycle }}</p>
            </div>
          </div>

          <div class="bg-white rounded-[2.5rem] p-8 border border-gray-100 shadow-sm">
            <div class="flex items-center justify-between mb-8">
              <h3 class="text-[10px] font-black text-primary uppercase tracking-[0.2em]">Agreement Documentation</h3>
              <span class="text-[10px] font-bold text-gray-400">{{ pictures?.length || 0 }} Files Attached</span>
            </div>

            <div v-if="pictures?.length" class="grid grid-cols-2 md:grid-cols-3 gap-6">
              <div
                v-for="pic in visiblePictures"
                :key="pic.id"
                class="group relative rounded-[2rem] overflow-hidden bg-gray-100 border border-gray-100 h-48 cursor-pointer shadow-sm hover:shadow-xl transition-all duration-300"
                @click="previewImage(pic.rent_image)"
              >
                <img
                  :src="pic.rent_image"
                  class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110"
                />
                
                <div class="absolute inset-0 bg-gradient-to-t from-black/80 via-black/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex flex-col justify-end p-4">
                  <p class="text-white text-[10px] font-bold truncate mb-3">{{ pic.description }}</p>
                  <div class="flex gap-2">
                    <button @click.stop="openUpdatePicture(pic)" class="flex-1 py-2 bg-white/20 backdrop-blur-md rounded-lg text-[9px] font-black uppercase text-white hover:bg-white/40 transition-colors">Update</button>
                    <button @click.stop="askDeletePicture(pic)" class="px-3 py-2 bg-rose-500/80 backdrop-blur-md rounded-lg text-[9px] font-black uppercase text-white hover:bg-rose-600 transition-colors">Delete</button>
                  </div>
                </div>
              </div>

              <div
                v-if="remainingPicturesCount > 0 && !showAllPictures"
                @click="showAllPictures = true"
                class="flex flex-col items-center justify-center border-2 border-dashed border-gray-100 rounded-[2rem] text-primary hover:border-primary hover:bg-primary/5 transition-all cursor-pointer h-48 group"
              >
                <span class="text-2xl font-black group-hover:scale-125 transition-transform">+{{ remainingPicturesCount }}</span>
                <span class="text-[9px] font-black uppercase tracking-widest">More Pictures</span>
              </div>
            </div>

            <div v-else class="py-12 text-center border-2 border-dashed border-gray-50 rounded-[2rem]">
              <i class="fas fa-images text-gray-200 text-4xl mb-3"></i>
              <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">No documentation uploaded</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="!loading" class="flex flex-col items-center justify-center py-20">
      <div class="w-20 h-20 bg-gray-100 rounded-full flex items-center justify-center text-gray-300 mb-4 text-2xl">
        <i class="fas fa-search"></i>
      </div>
      <p class="text-sm font-black text-gray-400 uppercase tracking-widest">Rent record not found</p>
    </div>

    <AddPictureModal v-if="addPictureVisible" :visible="addPictureVisible" :rentId="rentId" @close="addPictureVisible = false" @refresh="fetchPictures" />
    <UpdatePictureModal v-if="updatePictureVisible" :visible="updatePictureVisible" :picture="pictureToUpdate" :rentId="rentId" @close="updatePictureVisible = false" @refresh="fetchPictures" />
    <ConfirmModal v-if="confirmDeleteVisible" :visible="confirmDeleteVisible" title="Confirm Delete" message="Remove this image from agreement?" @confirm="confirmDeletePicture" @cancel="confirmDeleteVisible = false" />

    <div
      v-if="imagePreviewVisible"
      class="fixed inset-0 bg-slate-900/95 flex items-center justify-center z-[100] p-4 backdrop-blur-sm animate-in fade-in duration-300"
      @click="imagePreviewVisible = false"
    >
      <div class="relative max-w-5xl w-full">
        <img :src="imageToPreview" class="w-full max-h-[85vh] object-contain rounded-2xl shadow-2xl shadow-black/50" @click.stop />
        <button @click="imagePreviewVisible = false" class="absolute -top-12 right-0 text-white text-xl hover:text-primary transition-colors">
          <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import AddPictureModal from "@/views/closed/rent/addRentPicture.vue";
import UpdatePictureModal from "@/views/closed/rent/updateRentPicture.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Toast from "@/components/Toast.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  name: "RentDetailView",
  components: { AddPictureModal, UpdatePictureModal, ConfirmModal, Toast,Loading },
  data() {
    return {
      rent: null,
      pictures: [],
      rentId: "",
      addPictureVisible: false,
      updatePictureVisible: false,
      confirmDeleteVisible: false,
      pictureToUpdate: null,
      pictureToDelete: null,
      imagePreviewVisible: false,
      imageToPreview: null,
      showAllPictures: false,
      loading:false
    };
  },
   computed: {
    // 🆕 ADDED Computed Properties for picture display
    visiblePictures() {
      if (!this.pictures) return [];
      return this.showAllPictures ? this.pictures : this.pictures.slice(0, 3);
    },
    remainingPicturesCount() {
      if (!this.pictures) return 0;
      return Math.max(0, this.pictures.length - 3);
    },
  },
  mounted() {
    this.rentId = this.$route.params.id;
    this.fetchRent();
    this.fetchPictures();
  },
  methods: {
    async fetchRent() {
      this.loading=true
      try {
        const response = await this.$apiGetById(`/get_rent`, this.rentId);
        this.rent = response;
      } catch (error) {
        console.error("Failed to fetch rent:", error);
        this.rent = null;
      }
      finally {
        this.loading=false
      }
    },
    async fetchPictures() {
      if (!this.rentId) return;
      try {
        const response = await this.$apiGet(`/get_rent_pictures`, {
          rent_id: this.rentId,
        });
        this.pictures = response?.data || [];
      } catch (error) {
        console.error("Failed to fetch pictures:", error);
        this.pictures = [];
      }
    },

    // Picture actions
    openUpdatePicture(picture) {
      this.pictureToUpdate = picture;
      this.updatePictureVisible = true;
    },
    askDeletePicture(picture) {
      this.pictureToDelete = picture;
      this.confirmDeleteVisible = true;
    },
    async confirmDeletePicture() {
      this.confirmDeleteVisible = false;
      if (!this.pictureToDelete) return;
      try {
        const res = await this.$apiDelete(
          `/delete_rent_picture/${this.pictureToDelete.id}`
        );
        this.$root.$refs.toast.showToast(res.message || "Picture deleted", "success");
        this.fetchPictures();
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast("Failed to delete picture.", "error");
      }
      this.pictureToDelete = null;
    },

    previewImage(imageUrl) {
      this.imageToPreview = imageUrl;
      this.imagePreviewVisible = true;
    },

    // Other actions
    viewPayments(rentId) {
      this.$router.push({
          name: "rents_payment_detail",
          params: { id: rentId },
        });
    },
    editRent(rent) {
      console.log("Edit rent:", rent);
    },
  },
};
</script>
