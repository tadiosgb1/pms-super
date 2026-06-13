<template>
  <div>
    <Toast ref="toast" />
    <div
      v-if="visible"
      class="fixed inset-0 z-[70] bg-gray-900/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
    >
      <div class="bg-white w-full max-w-lg rounded-3xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
        
        <div class="bg-gradient-to-r from-dprimary to-primary px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center text-white backdrop-blur-md">
              <i class="fas fa-camera-retro"></i>
            </div>
            <div>
              <h2 class="text-lg font-black text-white tracking-tight leading-none">Add Media</h2>
              <p class="text-white/60 text-[10px] uppercase tracking-widest font-bold mt-1">Enhance Property Listing</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all">
            <i class="fas fa-times text-xs"></i>
          </button>
        </div>

        <form @submit.prevent="submitForm" class="flex-1 overflow-y-auto p-8 space-y-6">
          
          <div class="space-y-3">
            <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Select Image Asset</label>
            
            <div v-if="!form.preview" class="relative group">
              <label
                for="file-upload"
                class="flex flex-col items-center justify-center w-full h-56 border-2 border-dashed border-gray-200 rounded-3xl cursor-pointer bg-gray-50 group-hover:bg-primary/5 group-hover:border-primary/50 transition-all duration-300"
              >
                <div class="flex flex-col items-center justify-center pt-5 pb-6">
                  <div class="w-16 h-16 bg-white rounded-2xl shadow-sm flex items-center justify-center text-gray-300 group-hover:text-primary group-hover:scale-110 transition-all duration-300 mb-4">
                    <i class="fas fa-cloud-upload-alt text-2xl"></i>
                  </div>
                  <p class="text-sm font-black text-gray-500 group-hover:text-primary transition-colors">Click to browse files</p>
                  <p class="text-[10px] text-gray-400 font-bold uppercase tracking-tighter mt-1 text-center px-4">JPG, PNG up to 10MB</p>
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

            <div v-else class="relative group rounded-3xl overflow-hidden shadow-lg border-4 border-white ring-1 ring-gray-100">
              <img :src="form.preview" alt="preview" class="w-full h-auto max-h-72 object-cover" />
              <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center backdrop-blur-[2px]">
                <label for="file-upload" class="cursor-pointer bg-white text-gray-900 px-4 py-2 rounded-xl font-black text-[10px] uppercase tracking-widest shadow-xl transform translate-y-2 group-hover:translate-y-0 transition-all">
                  Change Image
                </label>
              </div>
              <input id="file-upload" type="file" class="hidden" @change="onFileChange" accept="image/*" />
            </div>
          </div>

          <div class="space-y-3 pb-4">
            <div class="flex justify-between items-end px-1">
              <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest">Media Description</label>
              <span class="text-[10px] font-black text-primary bg-primary/5 px-2 py-0.5 rounded">{{ form.description.length }}/200</span>
            </div>
            <textarea 
              v-model="form.description" 
              maxlength="200" 
              placeholder="e.g. Master Bedroom with Ocean View..."
              class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-medium focus:ring-2 focus:ring-primary/20 focus:border-primary focus:bg-white transition-all outline-none min-h-[120px] resize-none"
            />
          </div>

          <div class="sticky bottom-0 bg-white pt-4 pb-2 mt-auto border-t border-gray-50 flex items-center gap-4 shrink-0">
            <button 
              type="button" 
              @click="$emit('close')" 
              class="flex-1 px-6 py-4 rounded-2xl text-[11px] font-black uppercase tracking-widest text-gray-400 hover:bg-gray-50 transition-all"
            >
              Cancel
            </button>
            <button 
              type="submit" 
              class="flex-[2] bg-primary hover:bg-dprimary text-white font-black text-[11px] uppercase tracking-widest px-8 py-4 rounded-2xl shadow-xl shadow-primary/20 transform hover:-translate-y-0.5 active:translate-y-0 transition-all flex items-center justify-center gap-2"
            >
              <i class="fas fa-check-circle"></i> Upload Asset
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from '@/components/Toast.vue';

export default {
  name: 'AddPropertyPicture',
  components: { Toast },
  props: {
    visible: Boolean,
    propertyId:Number,
    propertyName:String
  },
  data() {
    return {
      properties: [],
      form: {
        description: '',
        property_id: this.propertyId,
        file: null,
        preview: null
      }
    };
  },
  mounted() {
    this.fetchProperties();
  },
  methods: {
    async fetchProperties() {
      try {
        const res = await this.$apiGet('/get_properties');
        this.properties = res.data || res;
      } catch (err) {
        console.error('Failed to fetch properties', err);
      }
    },
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
        fd.append('property_id', this.form.property_id);
        if (this.form.file) fd.append('property_image', this.form.file);

        const res = await this.$apiPost('/post_property_picture', fd, { 'Content-Type': 'multipart/form-data' });
        const msg = res?.message || 'Property picture added';
        this.$root.$refs.toast.showToast(msg, 'success');
        this.$emit('refresh');
        this.$emit('close');
         //this.$reloadPage();


        // Reset form
        this.form.description = '';
        this.form.property_id = '';
        this.form.file = null;
        this.form.preview = null;
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast.showToast('Failed to add picture', 'error');
      }
    }
  }
};
</script>

<style scoped>
/* scoped styles */
</style>
