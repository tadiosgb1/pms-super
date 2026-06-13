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
              <i class="fas fa-camera-retro text-white text-lg"></i>
            </div>
            <div>
              <h2 class="text-xl font-bold text-white tracking-tight">Add Zone Picture</h2>
              <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Visual Asset Documentation</p>
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
          <form id="pictureForm" @submit.prevent="submitForm" class="space-y-6">
            
            <div class="space-y-2">
              <div class="flex justify-between items-end">
                <label class="custom-label !mb-0">Description <span class="custom-star">*</span></label>
                <span :class="['text-[10px] font-bold tracking-widest uppercase', form.description.length >= 200 ? 'text-red-500' : 'text-gray-400']">
                  {{ form.description.length }}/200
                </span>
              </div>
              <textarea 
                v-model="form.description" 
                maxlength="200" 
                rows="3"
                placeholder="Briefly describe what this image represents (e.g., North entrance, utility meters)..."
                class="custom-input !mt-0 py-3 min-h-[100px] resize-none"
                required
              ></textarea>
            </div>

            <div class="space-y-3">
              <label class="custom-label">Upload Image File</label>
              
              <div class="relative group">
                <label
                  for="file-upload"
                  class="flex flex-col items-center justify-center w-full h-56 border-2 border-dashed border-gray-200 rounded-2xl cursor-pointer bg-gray-50/50 hover:bg-primary/5 hover:border-primary transition-all duration-300 overflow-hidden"
                >
                  <div v-if="!form.preview" class="flex flex-col items-center justify-center pt-5 pb-6">
                    <div class="w-16 h-16 mb-4 rounded-full bg-white shadow-sm border border-gray-100 flex items-center justify-center group-hover:scale-110 transition-transform duration-300">
                       <i class="fas fa-cloud-upload-alt text-2xl text-primary"></i>
                    </div>
                    <p class="text-sm font-bold text-gray-700">Click to upload or drag & drop</p>
                    <p class="text-[10px] text-gray-400 uppercase tracking-widest mt-1 font-semibold">PNG, JPG or WEBP (Max 5MB)</p>
                  </div>

                  <div v-else class="relative w-full h-full animate-in fade-in duration-300">
                    <img :src="form.preview" alt="preview" class="w-full h-full object-cover" />
                    <div class="absolute inset-0 bg-black/40 flex items-center justify-center opacity-0 hover:opacity-100 transition-opacity">
                      <div class="bg-white px-4 py-2 rounded-lg shadow-xl flex items-center gap-2 text-xs font-bold text-gray-700">
                        <i class="fas fa-sync-alt text-primary"></i> Replace Image
                      </div>
                    </div>
                  </div>

                  <input
                    required
                    id="file-upload"
                    type="file"
                    class="hidden"
                    @change="onFileChange"
                    accept="image/*"
                  />
                </label>
              </div>
            </div>

            <div class="p-4 bg-gray-50 rounded-xl border border-gray-100 flex gap-3">
               <i class="fas fa-lightbulb text-amber-400 mt-1"></i>
               <p class="text-[11px] text-gray-500 leading-relaxed">
                 <span class="font-bold text-gray-700 uppercase tracking-tighter">Pro Tip:</span> High-quality images of entrances and utility access points help managers verify zone boundaries more effectively.
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
            form="pictureForm"
            type="submit"
            class="custom-button !w-auto !px-10 flex items-center gap-2"
          >
            <i class="fas fa-save"></i>
            Commit Asset
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from '@/components/Toast.vue';

export default {
  name: 'AddPropertyZonePicture',
  components: { Toast },
  props: {
    visible: Boolean,
    zoneId: Number,
  },
  data() {
    return {
      form: {
        description: '',
        property_zone_id: this.zoneId,
        file: null,
        preview: null,
      },
    };
  },
  methods: {
    onFileChange(e) {
      const file = e.target.files[0];
      if (!file) return;
      this.form.file = file;
      this.form.preview = URL.createObjectURL(file);
    },
    async submitForm() {
      try {
        const fd = new FormData();
        fd.append('description', this.form.description);
        fd.append('property_zone_id', this.zoneId);
        if (this.form.file) fd.append('property_image', this.form.file);

        const res = await this.$apiPost(`/post_property_zone_picture`, fd, {
          'Content-Type': 'multipart/form-data',
        });
        const msg = res?.message || 'Zone picture added';
        this.$root.$refs.toast.showToast(msg, 'success');
        this.$emit('refresh');
        this.$emit('close');

        this.form = { description: '', zone_id: this.zoneId, file: null, preview: null };
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast('Failed to add picture', 'error');
      }
    },
  },
};
</script>
