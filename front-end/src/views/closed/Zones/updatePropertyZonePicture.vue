<template>
  <div>
    <Toast ref="toast" />

    <div
      v-if="visible"
      class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm transition-all duration-300"
    >
      <div 
        class="bg-white w-full max-w-2xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100 animate-in fade-in zoom-in duration-200"
      >
        <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-5 flex justify-between items-center shadow-md shrink-0">
          <div class="flex items-center gap-3">
            <div class="p-2 bg-white/20 rounded-lg backdrop-blur-md">
              <i class="fas fa-sync-alt text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Update Zone Picture</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Modify Existing Visual Asset</p>
            </div>
          </div>
          <button 
            @click="$emit('close')" 
            class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-colors"
          >
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="flex-1 overflow-y-auto p-6 md:p-8">
          <form id="updatePictureForm" @submit.prevent="updateModalVisible = true" class="space-y-6">
            
            <div class="space-y-2">
              <div class="flex justify-between items-end">
                <label class="custom-label !mb-0">Asset Description</label>
                <span :class="['text-[10px] font-bold tracking-widest uppercase', (form.description || '').length >= 200 ? 'text-red-500' : 'text-gray-400']">
                  {{ (form.description || "").length }}/200
                </span>
              </div>
              <textarea 
                v-model="form.description" 
                maxlength="200" 
                rows="3"
                class="custom-input !mt-0 py-3 min-h-[100px] resize-none shadow-sm"
                placeholder="Describe the updated view..."
              ></textarea>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              
              <div v-if="form.property_image" class="space-y-2">
                <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest flex items-center gap-2">
                  <i class="fas fa-history"></i> Current Version
                </p>
                <div class="relative aspect-video rounded-xl overflow-hidden border border-gray-200 bg-gray-50">
                  <img :src="form.property_image" class="w-full h-full object-cover opacity-60" />
                  <div class="absolute inset-0 flex items-center justify-center">
                    <span class="bg-gray-800/60 backdrop-blur-sm text-white text-[9px] font-bold px-2 py-1 rounded uppercase">In Use</span>
                  </div>
                </div>
              </div>

              <div class="space-y-2">
                <p class="text-[10px] font-black text-primary uppercase tracking-widest flex items-center gap-2">
                  <i class="fas fa-plus-circle"></i> New Replacement
                </p>
                <label 
                  for="update-upload" 
                  class="relative aspect-video flex flex-col items-center justify-center rounded-xl border-2 border-dashed border-gray-200 bg-gray-50 hover:bg-primary/5 hover:border-primary transition-all cursor-pointer overflow-hidden group"
                >
                  <div v-if="!form.preview" class="flex flex-col items-center">
                    <i class="fas fa-upload text-gray-300 group-hover:text-primary transition-colors"></i>
                    <span class="text-[10px] font-bold text-gray-400 mt-2">Replace File</span>
                  </div>
                  
                  <img v-else :src="form.preview" class="w-full h-full object-cover animate-in fade-in" />
                  
                  <input
                    id="update-upload"
                    type="file"
                    class="hidden"
                    @change="onFileChange"
                    accept="image/*"
                  />
                </label>
              </div>

            </div>

            <div class="p-4 bg-amber-50 rounded-xl border border-amber-100 flex gap-3">
               <i class="fas fa-exclamation-triangle text-amber-500 mt-1"></i>
               <p class="text-[11px] text-amber-800 leading-relaxed font-medium">
                 <span class="font-bold uppercase tracking-tighter">Warning:</span> Updating this asset will overwrite the current image across all property listings. This action cannot be undone.
               </p>
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
            form="updatePictureForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-check-circle"></i>
            Update Asset
          </button>
        </div>
      </div>

      <ConfirmModal
        :visible="updateModalVisible"
        title="Confirm Asset Replacement"
        message="Are you sure you want to update this Property Zone Picture? The existing image will be permanently replaced."
        @cancel="updateModalVisible = false"
        @confirm="submitForm"
      />
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";

export default {
  name: "UpdatePropertyZonePicture",
  components: { Toast,ConfirmModal },
  props: {
    visible: Boolean,
    picture: Object,
    zoneId: Number,
  },
  data() {
    return {
      form: {
        id: null,
        description: "",
        property_image: "",
        property_zone_id: this.zoneId,
        file: null,
        preview: null,
      },
      updateModalVisible:false
    };
  },
  watch: {
    picture: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.form = {
            ...newVal,
            description: newVal.description || "",
            property_zone_id: this.zoneId,
            file: null,
            preview: null,
          };
        }
      },
    },
  },
  methods: {
    onFileChange(e) {
      const file = e.target.files[0];
      if (!file) return;
      this.form.file = file;
      this.form.preview = URL.createObjectURL(file);
    },
    async submitForm() {
      this.updateModalVisible=false
      try {
        const fd = new FormData();
        fd.append("description", this.form.description);
        fd.append("property_zone_id", this.zoneId);
        if (this.form.file) fd.append("property_image", this.form.file);

        const res = await this.$apiPut(
          `/update_property_zone_picture`,
          this.form.id,
          fd,
          {
            "Content-Type": "multipart/form-data",
          }
        );
        
        const msg = res?.message || "Zone picture updated";
        this.$root.$refs.toast.showToast(msg, "success");
        this.$emit("refresh");
        setTimeout(() => this.$emit("close"), 1000);
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast(
          "Failed to update zone picture",
          "error"
        );
      }
    },
  },
};
</script>
