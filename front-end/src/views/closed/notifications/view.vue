<template>
  <div class="min-h-screen bg-background p-4 md:p-2">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading Notifications..." />

    <div class="max-w-7xl mx-auto bg-white shadow-xl rounded-xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md text-white">
            <i class="fas fa-bell text-2xl animate-pulse"></i>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Notifications Center</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Real-time Activity Logs</p>
          </div>
        </div>
      </div>

      <div class="p-6 border-b border-gray-50 flex flex-col md:flex-row justify-between items-center gap-4 bg-gray-50/30">
        <div class="relative w-full md:max-w-md">
          <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400">
            <i class="fas fa-search text-sm"></i>
          </span>
          <input
            v-model="searchTerm"
            type="search"
            placeholder="Search message, type or user..."
            class="custom-input !mt-0 !pl-10 h-10 shadow-sm"
          />
        </div>

        <div class="flex items-center gap-3">
          <label class="text-xs font-bold text-gray-400 uppercase tracking-wider">Show</label>
          <select
            v-model="pageSize"
            @change="fetchNotifications()"
            class="border-gray-200 rounded-lg text-sm font-semibold focus:ring-primary focus:border-primary py-1.5"
          >
            <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
          </select>
          <span class="text-xs font-bold text-gray-400 uppercase tracking-wider">entries</span>
        </div>
      </div>

      <div class="hidden md:block overflow-x-auto p-6">
        <table class="min-w-full divide-y divide-gray-200">
          <thead>
            <tr class="text-left text-[10px] font-black text-gray-400 uppercase tracking-widest">
              <th class="px-3 py-4">Status</th>
              <th class="px-3 py-4">Message</th>
              <th class="px-3 py-4">Type</th>
              <th class="px-3 py-4">Recipient</th>
              <th class="px-3 py-4">References</th>
              <th class="px-3 py-4 text-center">Actions</th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-50">
            <tr v-for="notif in filteredNotifications" :key="notif.id" class="hover:bg-primary/5 transition-all group">
              <td class="px-3 py-4">
                <span 
                  :class="notif.is_read ? 'bg-gray-100 text-gray-400' : 'bg-blue-100 text-blue-600'"
                  class="px-2 py-1 rounded-full text-[9px] font-black uppercase"
                >
                  {{ notif.is_read ? "Read" : "Unread" }}
                </span>
              </td>

              <td class="px-3 py-4">
                <p class="text-xs font-semibold text-gray-700 leading-relaxed max-w-xs truncate group-hover:text-clip group-hover:whitespace-normal">
                  {{ notif.message }}
                </p>
                <p class="text-[9px] text-gray-400 mt-1 font-mono italic">{{ formatDate(notif.created_at) }}</p>
              </td>

              <td class="px-3 py-4">
                <span class="text-[10px] font-bold text-gray-500 bg-gray-100 px-2 py-0.5 rounded uppercase">
                  {{ notif.notification_type }}
                </span>
              </td>

              <td class="px-3 py-4">
                <div class="flex items-center gap-2">
                  <div class="w-7 h-7 bg-primary/10 rounded-full flex items-center justify-center text-primary text-[10px] font-bold">
                    {{ notif.user_id.first_name[0] }}{{ notif.user_id.last_name[0] }}
                  </div>
                  <span class="text-xs font-bold text-gray-600">
                    {{ notif.user_id.first_name }} {{ notif.user_id.last_name }}
                  </span>
                </div>
              </td>

              <td class="px-3 py-4">
                <div class="flex gap-1.5">
                  <span v-if="notif.maintenance_request_id" title="Maintenance" class="w-6 h-6 rounded bg-yellow-50 text-yellow-600 flex items-center justify-center text-[10px] border border-yellow-100">
                    <i class="fas fa-tools"></i>
                  </span>
                  <span v-if="notif.payment_id" title="Payment" class="w-6 h-6 rounded bg-purple-50 text-purple-600 flex items-center justify-center text-[10px] border border-purple-100">
                    <i class="fas fa-credit-card"></i>
                  </span>
                  <span v-if="notif.rent_id" title="Rent" class="w-6 h-6 rounded bg-orange-50 text-orange-600 flex items-center justify-center text-[10px] border border-orange-100">
                    <i class="fas fa-home"></i>
                  </span>
                </div>
              </td>

              <td class="px-3 py-4 text-center">
                <div class="flex justify-center items-center gap-1 opacity-60 group-hover:opacity-100 transition-opacity">
                  <button v-if="!notif.is_read" @click="markAsRead(notif.id)" class="p-1.5 text-blue-500 hover:bg-blue-50 rounded" title="Mark Read">
                    <i class="fas fa-check-double"></i>
                  </button>
                  <button @click="goToNotification(notif.id)" class="p-1.5 text-primary hover:bg-primary/5 rounded" title="Details">
                    <i class="fas fa-eye"></i>
                  </button>
                  <router-link v-if="notif.user_id" :to="`/users/${notif.user_id.id || notif.user_id}`" class="p-1.5 text-green-500 hover:bg-green-50 rounded" title="User">
                    <i class="fas fa-user-circle"></i>
                  </router-link>
                  <router-link v-if="notif.maintenance_request_id" :to="`/maintenance/${notif.maintenance_request_id}`" class="p-1.5 text-yellow-600 hover:bg-yellow-50 rounded" title="Maintenance Link">
                    <i class="fas fa-external-link-alt"></i>
                  </router-link>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="md:hidden grid gap-4 p-4 bg-gray-50/50">
        <div v-for="notif in filteredNotifications" :key="notif.id" 
          class="bg-white rounded-2xl shadow-sm border border-gray-100 p-5 relative overflow-hidden"
          :class="!notif.is_read ? 'border-l-4 border-l-primary' : ''"
        >
          <div class="flex justify-between items-start mb-2">
            <span class="text-[9px] font-black uppercase text-gray-400 tracking-tighter">{{ notif.notification_type }}</span>
            <span class="text-[10px] text-gray-400 font-medium">{{ formatDate(notif.created_at) }}</span>
          </div>
          <p class="font-bold text-gray-800 text-sm mb-3">{{ notif.message }}</p>
          
          <div class="flex flex-wrap items-center gap-3 pt-3 border-t border-gray-50">
             <div class="flex items-center gap-2 text-xs font-bold text-gray-500">
               <i class="fas fa-user-circle text-primary opacity-50"></i>
               {{ notif.user_id.first_name }} {{ notif.user_id.last_name }}
             </div>
          </div>

          <div class="flex justify-between items-center mt-4">
            <div class="flex gap-2">
              <router-link v-if="notif.maintenance_request_id" :to="`/maintenance/${notif.maintenance_request_id}`" class="w-8 h-8 rounded-lg bg-yellow-50 text-yellow-600 flex items-center justify-center text-xs border border-yellow-100"><i class="fas fa-tools"></i></router-link>
              <router-link v-if="notif.payment_id" :to="`/payments/${notif.payment_id}`" class="w-8 h-8 rounded-lg bg-purple-50 text-purple-600 flex items-center justify-center text-xs border border-purple-100"><i class="fas fa-credit-card"></i></router-link>
              <router-link v-if="notif.rent_id" :to="`/rents/${notif.rent_id}`" class="w-8 h-8 rounded-lg bg-orange-50 text-orange-600 flex items-center justify-center text-xs border border-orange-100"><i class="fas fa-home"></i></router-link>
            </div>
            <div class="flex gap-2">
              <button v-if="!notif.is_read" @click="markAsRead(notif.id)" class="text-[10px] font-bold uppercase text-blue-600 px-3 py-1 bg-blue-50 rounded-full">Mark Read</button>
              <button @click="goToNotification(notif.id)" class="text-[10px] font-bold uppercase text-primary px-3 py-1 bg-primary/5 rounded-full border border-primary/10">Details</button>
            </div>
          </div>
        </div>
      </div>

      <div class="p-6 bg-gray-50/50 border-t border-gray-100 flex flex-col sm:flex-row justify-between items-center gap-4">
        <p class="text-xs font-bold text-gray-400 uppercase tracking-widest">
          Showing Page <span class="text-primary">{{ currentPage }}</span> of {{ totalPages }}
        </p>
        <div class="flex gap-2">
          <button
            :disabled="!previous"
            @click="fetchNotifications(previous)"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 disabled:hover:bg-white disabled:hover:text-gray-600 transition-all shadow-sm"
          >
            <i class="fas fa-chevron-left mr-2"></i> Previous
          </button>
          <button
            :disabled="!next"
            @click="fetchNotifications(next)"
            class="px-5 py-2 rounded-xl bg-white border border-gray-200 text-sm font-bold text-gray-600 hover:bg-primary hover:text-white disabled:opacity-30 disabled:hover:bg-white disabled:hover:text-gray-600 transition-all shadow-sm"
          >
            Next <i class="fas fa-chevron-right ml-2"></i>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import Loading from "@/components/Loading.vue"; // <-- Added Loading



export default {
  name: "NotificationsView",
  components: { Toast ,Loading},
  data() {
    return {
      notifications: [],
      searchTerm: "",
      currentPage: 1,
      totalPages: 1,
      next: null,
      previous: null,
      pageSize: 1000,
      pageSizes: [5, 10, 20, 50, 100],
      loading:false,
    };
  },
  computed: {
    filteredNotifications() {
      const term = this.searchTerm.toLowerCase();
      return this.notifications.filter(
        (n) =>
          n.message.toLowerCase().includes(term) ||
          n.notification_type.toLowerCase().includes(term) ||
          (n.is_read ? "read" : "unread").includes(term)
      );
    },
  },
  mounted() {
    this.fetchNotifications();
  },
  methods: {
    goToNotification(id) {
    this.$router.push({ name: 'notificationDetail', params: { id } });
   },
    async fetchNotifications(customUrl = null) {
      this.loading=true;
      try {
        let params = {
          user_id__email:localStorage.getItem('email'),
          page_size: this.pageSize,
          page: this.currentPage };
          if(localStorage.getItem('is_superuser'=='true')){
         params = {
          page_size: this.pageSize,
          page: this.currentPage };
          }
        const url = customUrl || "get_notifications";
        console.log("params for notif",params);

        const res = await this.$apiGet(`/get_notifications`, params);

        this.notifications = res.data || [];
        this.currentPage = res.current_page || 1;
        this.totalPages = res.total_pages || 1;
        this.next = res.next;
        this.previous = res.previous;
      } catch (err) {
        console.error("Failed to fetch notifications:", err);
        this.notifications = [];
      }finally{
        this.loading=false;
      }
    },
   async markAsRead(id) {

    console.log("id",id);

      try {
        const payload = { is_read: true };
        const res = await this.$apiPatch(`/update_notification`, id, payload);
       
        this.$root.$refs.toast.showToast("Successfully marked as red", "success");
        
      } catch (err) {
        console.error("Failed to mark as read:", err);
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleString();
    },
  },
};
</script>
