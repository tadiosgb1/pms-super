<template>
  <div>
    <Toast ref="toast" />
    <div
      v-if="visible"
      class="fixed inset-0 z-[70] bg-gray-900/60 backdrop-blur-sm flex items-center justify-center p-4 transition-all duration-300"
    >
      <div class="bg-white w-full max-w-2xl rounded-3xl shadow-2xl flex flex-col max-h-[90vh] overflow-hidden border border-gray-100 animate-in zoom-in duration-200">
        
        <div class="bg-gradient-to-r from-blue-700 to-primary px-8 py-6 flex justify-between items-center shrink-0">
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 bg-white/20 rounded-xl flex items-center justify-center text-white backdrop-blur-md">
              <i class="fas fa-sync-alt text-lg"></i>
            </div>
            <div>
              <h2 class="text-lg font-black text-white tracking-tight leading-none">Update Media</h2>
              <p class="text-white/60 text-[10px] uppercase tracking-widest font-bold mt-1">Replace existing asset</p>
            </div>
          </div>
          <button @click="$emit('close')" class="w-8 h-8 flex items-center justify-center rounded-full bg-black/10 text-white hover:bg-white/20 transition-all">
            <i class="fas fa-times text-xs"></i>
          </button>
        </div>

        <form @submit.prevent="updateModalVisible = true" class="flex-1 overflow-y-auto p-8 space-y-8">
          
          <div class="space-y-3">
            <div class="flex justify-between items-end px-1">
              <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest">Media Description</label>
              <span class="text-[10px] font-black text-primary bg-primary/5 px-2 py-0.5 rounded">
                {{ (form.description || '').length }}/200
              </span>
            </div>
            <textarea 
              v-model="form.description" 
              maxlength="200" 
              placeholder="Update the description for this image..."
              class="w-full px-5 py-4 bg-gray-50 border border-gray-100 rounded-2xl text-sm font-medium focus:ring-2 focus:ring-primary/20 focus:border-primary focus:bg-white transition-all outline-none min-h-[100px] resize-none"
            />
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div v-if="form.property_image" class="space-y-3">
              <label class="block text-[10px] font-black text-gray-400 uppercase tracking-widest ml-1">Current Image</label>
              <div class="relative rounded-2xl overflow-hidden border-2 border-gray-100 shadow-inner bg-gray-50 aspect-video flex items-center justify-center">
                <img :src="form.property_image" class="w-full h-full object-cover opacity-60" />
                <div class="absolute inset-0 flex items-center justify-center">
                  <span class="bg-gray-900/50 backdrop-blur-sm text-white text-[9px] font-black uppercase tracking-[0.2em] px-3 py-1 rounded-full">Old Version</span>
                </div>
              </div>
            </div>

            <div class="space-y-3">
              <label class="block text-[10px] font-black text-primary uppercase tracking-widest ml-1">New Version</label>
              <div v-if="!form.preview" class="relative group">
                <label
                  for="file-replace"
                  class="flex flex-col items-center justify-center w-full aspect-video border-2 border-dashed border-primary/30 rounded-2xl cursor-pointer bg-primary/5 hover:bg-primary/10 hover:border-primary transition-all"
                >
                  <i class="fas fa-cloud-upload-alt text-primary text-xl mb-2"></i>
                  <p class="text-[10px] font-black text-primary uppercase tracking-widest">Choose New File</p>
                  <input id="file-replace" required type="file" @change="onFileChange" accept="image/*" class="hidden" />
                </label>
              </div>
              
              <div v-else class="relative rounded-2xl overflow-hidden border-2 border-primary shadow-lg aspect-video group">
                <img :src="form.preview" class="w-full h-full object-cover" />
                <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center backdrop-blur-[2px]">
                   <label for="file-replace" class="cursor-pointer bg-white text-gray-900 px-4 py-2 rounded-xl font-black text-[10px] uppercase tracking-widest shadow-xl">
                    Change Selection
                  </label>
                </div>
                <input id="file-replace" type="file" @change="onFileChange" accept="image/*" class="hidden" />
              </div>
            </div>
          </div>

          <div class="sticky bottom-0 bg-white pt-6 pb-2 mt-auto border-t border-gray-100 flex items-center gap-4 shrink-0">
            <button 
              type="button" 
              @click="$emit('close')" 
              class="flex-1 px-6 py-4 rounded-2xl text-[11px] font-black uppercase tracking-widest text-gray-400 hover:bg-gray-50 transition-all"
            >
              Cancel
            </button>
            <button 
              type="submit" 
              class="flex-[2] bg-primary hover:bg-orange-600 text-white font-black text-[11px] uppercase tracking-widest px-8 py-4 rounded-2xl shadow-xl shadow-primary/20 transform hover:-translate-y-0.5 active:translate-y-0 transition-all flex items-center justify-center gap-2"
            >
              <i class="fas fa-sync-alt"></i> Update Asset
            </button>
          </div>
        </form>
      </div>

      <ConfirmModal
        :visible="updateModalVisible"
        title="Confirm Media Update"
        message="This will permanently replace the existing property picture. Do you wish to proceed?"
        @cancel="updateModalVisible = false"
        @confirm="submitForm"
      />
    </div>
  </div>
</template>

<script>
import Toast from '@/components/Toast.vue';
import ConfirmModal from "@/components/ConfirmModal.vue";

export default {
  name: 'UpdatePropertyPicture',
  components: { Toast,ConfirmModal },
  props: {
    visible: Boolean,
    picture: Object,
    propertyId:Number
  },
  data() {
    return {
      properties: [],
      form: {
        id: null,
        description: '',
        property_image: '',
        property_id: this.propertyId,
        file: null,
        preview: null
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
            description: newVal.description || '',  // ensure description is always a string
            file: null, 
            preview: null 
          };
        }
      }
    }
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
      this.updateModalVisible=false
      try {
        const fd = new FormData();
        fd.append('description', this.form.description);
        fd.append('property_id', this.form.property_id);
        if (this.form.file) fd.append('property_image', this.form.file);

        const res = await this.$apiPut('/update_property_picture', this.form.id, fd, { 'Content-Type': 'multipart/form-data' });
        const msg = res?.message || 'Property picture updated';
        this.$root.$refs.toast.showToast(msg, 'success');
        this.$emit('refresh');
          this.$emit("close");
      } catch (err) {
        console.error(err);
        alert('Failed to update property picture');
      }
    }
  }
};
</script>

<style scoped>
.custom-input {
  @apply w-full px-4 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-indigo-500;
}
</style>
