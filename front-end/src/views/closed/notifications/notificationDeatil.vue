<template>
  <div class="min-h-screen bg-background p-4 md:p-8 flex justify-center items-start">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Notification Detail..." />

    <div class="w-full max-w-4xl">
      <div class="flex justify-between items-center mb-6">
        <div class="flex items-center gap-3">
          <router-link
            to="/notifications"
            class="w-10 h-10 flex items-center justify-center rounded-xl bg-white shadow-sm border border-gray-100 text-gray-400 hover:text-primary hover:border-primary transition-all"
          >
            <i class="fas fa-chevron-left"></i>
          </router-link>
          <h1 class="text-2xl font-bold text-gray-800 tracking-tight">Notification Archive</h1>
        </div>
        
        <button
          v-if="!notification.is_read"
          @click="markAsRead(notification.id)"
          class="px-5 py-2 bg-primary text-white text-sm font-bold rounded-xl shadow-lg shadow-primary/20 hover:bg-dprimary transition-all transform hover:scale-105 active:scale-95"
        >
          <i class="fas fa-check-circle mr-2"></i> Mark as Read
        </button>
      </div>

      <div class="bg-white rounded-2xl shadow-xl overflow-hidden border border-gray-100 flex flex-col md:flex-row">
        <div class="w-full md:w-2 bg-gradient-to-b from-primary to-dprimary"></div>

        <div class="flex-1 p-6 md:p-10">
          <div class="flex justify-between items-start mb-8">
            <div>
              <span class="text-[10px] font-black uppercase tracking-[0.2em] text-primary bg-primary/5 px-3 py-1 rounded-full border border-primary/10">
                {{ notification.notification_type }}
              </span>
              <h2 class="mt-4 text-3xl font-bold text-gray-900 leading-tight">Activity Details</h2>
            </div>
            <div 
              :class="notification.is_read ? 'bg-green-50 text-green-600 border-green-100' : 'bg-amber-50 text-amber-600 border-amber-100'"
              class="px-4 py-1.5 rounded-lg border text-xs font-black uppercase tracking-widest flex items-center gap-2"
            >
              <span :class="notification.is_read ? 'bg-green-600' : 'bg-amber-600'" class="w-2 h-2 rounded-full animate-pulse"></span>
              {{ notification.is_read ? 'Read' : 'Unread' }}
            </div>
          </div>

          <div class="bg-gray-50 rounded-2xl p-6 mb-8 border border-gray-100 relative">
             <i class="fas fa-quote-left absolute top-4 left-4 text-gray-200 text-2xl"></i>
             <p class="text-gray-700 text-lg leading-relaxed relative z-10 pl-6">
               {{ notification.message }}
             </p>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-y-6 gap-x-12 pt-6 border-t border-gray-50">
            <div class="space-y-4">
              <h4 class="text-[11px] font-black text-gray-400 uppercase tracking-widest">Timeline</h4>
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg bg-gray-100 flex items-center justify-center text-gray-400">
                  <i class="fas fa-calendar-alt text-xs"></i>
                </div>
                <div>
                  <p class="text-[10px] text-gray-400 font-bold uppercase">Created</p>
                  <p class="text-sm font-semibold text-gray-700">{{ formatDate(notification.created_at) }}</p>
                </div>
              </div>
              <div v-if="notification.read_at" class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg bg-green-50 flex items-center justify-center text-green-500">
                  <i class="fas fa-eye text-xs"></i>
                </div>
                <div>
                  <p class="text-[10px] text-gray-400 font-bold uppercase">Viewed On</p>
                  <p class="text-sm font-semibold text-gray-700">{{ formatDate(notification.read_at) }}</p>
                </div>
              </div>
            </div>

            <div class="space-y-4">
              <h4 class="text-[11px] font-black text-gray-400 uppercase tracking-widest">Linked Resources</h4>
              
              <div v-if="notification.user_id" class="flex items-center justify-between p-3 bg-primary/5 rounded-xl border border-primary/10">
                <div class="flex items-center gap-3">
                  <div class="w-8 h-8 rounded-full bg-primary text-white flex items-center justify-center text-[10px] font-bold uppercase">
                    {{ notification.user_id.first_name[0] }}{{ notification.user_id.last_name[0] }}
                  </div>
                  <span class="text-xs font-bold text-gray-700">
                    {{ notification.user_id.first_name }} {{ notification.user_id.last_name }}
                  </span>
                </div>
                <router-link :to="`/users/${notification.user_id.id || notification.user_id}`" class="text-primary hover:text-dprimary">
                  <i class="fas fa-external-link-alt text-xs"></i>
                </router-link>
              </div>

              <router-link v-if="notification.maintenance_request_id" :to="`/maintenance/${notification.maintenance_request_id}`" 
                class="flex items-center justify-between p-3 bg-yellow-50 rounded-xl border border-yellow-100 hover:bg-yellow-100 transition-colors">
                <div class="flex items-center gap-3">
                  <i class="fas fa-tools text-yellow-600"></i>
                  <span class="text-xs font-bold text-yellow-700">Maintenance Request</span>
                </div>
                <span class="text-[10px] font-mono text-yellow-600">#{{ notification.maintenance_request_id }}</span>
              </router-link>

              <router-link v-if="notification.payment_id" :to="`/payments/${notification.payment_id}`" 
                class="flex items-center justify-between p-3 bg-purple-50 rounded-xl border border-purple-100 hover:bg-purple-100 transition-colors">
                <div class="flex items-center gap-3">
                  <i class="fas fa-credit-card text-purple-600"></i>
                  <span class="text-xs font-bold text-purple-700">Payment Details</span>
                </div>
                <span class="text-[10px] font-mono text-purple-600">#{{ notification.payment_id }}</span>
              </router-link>

              <router-link v-if="notification.rent_id" :to="`/rents/${notification.rent_id}`" 
                class="flex items-center justify-between p-3 bg-orange-50 rounded-xl border border-orange-100 hover:bg-orange-100 transition-colors">
                <div class="flex items-center gap-3">
                  <i class="fas fa-home text-orange-600"></i>
                  <span class="text-xs font-bold text-orange-700">Rent Agreement</span>
                </div>
                <span class="text-[10px] font-mono text-orange-600">#{{ notification.rent_id }}</span>
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import Loading from "@/components/Loading.vue"; // <-- Added Loading

export default {
  components:{
    Loading
  },
  name: "NotificationDetail",
  data() {
    return {
      notification: {},
      loading:false,
    };
  },
  mounted() {
    this.fetchNotification();
  },
  methods: {
    async fetchNotification() {
      this.loading=true;
      try {
        const id = this.$route.params.id;
        const res = await this.$apiGet(`/get_notification/${id}`);
        this.notification = res || {};
      } catch (error) {
        console.error("Failed to fetch notification:", error);
        this.notification = {};
      }finally{
        this.loading=false;
      }
    },
    async markAsRead(id) {
      try {
        const payload = { 
          is_read: true ,
          id:id,
          user:localStorage.getItem('userId')
        };
        const res = await this.$apiPost(`/post_notification_user`,  payload);
        if (res) {
          this.notification.is_read = true;
          this.notification.read_at = new Date().toISOString();
           this.$root.$refs.toast.showToast("Successfully marked as red", "success");
        }
      } catch (err) {
        console.error("Failed to mark as read:", err);
      }
    },
    formatDate(dateString) {
      if (!dateString) return "-";
      return new Date(dateString).toLocaleString();
    },
  },
};
</script>

<style scoped>
/* Optional: add subtle hover effect */
</style>
