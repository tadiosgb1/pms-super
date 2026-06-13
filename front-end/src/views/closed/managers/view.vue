<template>
  <div>
    <Loading :visible="loading" message="Loading managers..." />

    <div class="flex flex-col sm:flex-row gap-3 items-start sm:items-center justify-between mb-4">
      <div class="relative w-full sm:max-w-sm">
        <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
        <input v-model="searchTerm" type="search" placeholder="Search managers..."
          class="w-full pl-9 pr-4 py-2 text-sm border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary/30" />
      </div>
      <div class="flex items-center gap-2 text-sm text-gray-500">
        Show
        <select v-model="pageSize" @change="fetchManagers()" class="border border-gray-200 rounded-lg px-2 py-1 text-sm focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option v-for="s in pageSizes" :key="s" :value="s">{{ s }}</option>
        </select>
        entries
      </div>
      <button @click="showAddManager = true" class="flex items-center gap-2 bg-primary text-white text-sm font-semibold px-4 py-2 rounded-lg hover:bg-primary/90 transition whitespace-nowrap">
        <i class="fas fa-plus-circle"></i> Add Manager
      </button>
    </div>

    <div class="overflow-x-auto rounded-lg border border-gray-100">
      <table class="w-full text-sm">
        <thead class="bg-gray-50 text-xs font-semibold text-gray-500 uppercase tracking-wide">
          <tr>
            <th class="px-3 py-3 text-left cursor-pointer hover:text-primary" @click="sortBy('fullName')">
              Name <SortIcon field="fullName" :sort-key="sortKey" :sort-asc="sortAsc" />
            </th>
            <th class="px-3 py-3 text-left">Email</th>
            <th class="px-3 py-3 text-left">Groups</th>
            <th class="px-3 py-3 text-left">Owner</th>
            <th class="px-3 py-3 text-center">Status</th>
            <th class="px-3 py-3 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="manager in managers" :key="manager.id" class="hover:bg-gray-50 transition">
            <td class="px-3 py-2">
              <div class="flex items-center gap-2">
                <div class="w-7 h-7 rounded-full bg-primary/10 text-primary font-bold text-xs flex items-center justify-center uppercase shrink-0">
                  {{ ((manager.manager.first_name||'')[0]||'') }}{{ ((manager.manager.last_name||'')[0]||'') }}
                </div>
                <span class="font-medium text-gray-800 whitespace-nowrap">
                  {{ manager.manager.first_name }} {{ manager.manager.middle_name }} {{ manager.manager.last_name }}
                </span>
              </div>
            </td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ manager.manager.email || '—' }}</td>
            <td class="px-3 py-2">
              <div class="flex flex-wrap gap-1">
                <span v-for="g in manager.manager.groups" :key="g" class="px-2 py-0.5 bg-gray-100 text-gray-600 rounded text-[10px] font-semibold uppercase border border-gray-200">{{ g }}</span>
                <span v-if="!manager.manager.groups?.length" class="text-xs text-gray-300 italic">—</span>
              </div>
            </td>
            <td class="px-3 py-2 text-xs text-gray-700">
              <div class="flex items-center gap-1">
                <i class="fas fa-crown text-amber-400 text-[10px]"></i>
                {{ manager.owner.first_name }} {{ manager.owner.last_name }}
              </div>
            </td>
            <td class="px-3 py-2 text-center">
              <span :class="manager.is_active ? 'bg-green-100 text-green-700' : 'bg-orange-100 text-orange-700'"
                class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase">
                {{ manager.is_active ? 'Active' : 'Inactive' }}
              </span>
            </td>
            <td class="px-3 py-2 text-right">
              <div class="flex justify-end gap-1">
                <router-link :to="`/user_detail/${manager.id}`" class="btn-action btn-blue"><i class="fas fa-eye"></i></router-link>
                <button v-if="!manager.is_active" @click="askConfirmation('activate', manager.id)" class="btn-action btn-green"><i class="fas fa-power-off"></i></button>
                <button v-else @click="askConfirmation('deactivate', manager.id)" class="btn-action btn-orange"><i class="fas fa-ban"></i></button>
              </div>
            </td>
          </tr>
          <tr v-if="!managers.length && !loading">
            <td colspan="6" class="px-3 py-10 text-center text-sm text-gray-400 italic">No managers found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="flex items-center justify-between mt-4 text-sm text-gray-500">
      <span>Page {{ currentPage }} / {{ totalPages }}</span>
      <div class="flex gap-2">
        <button :disabled="!previous" @click="fetchManagers(previous)" class="btn-page"><i class="fas fa-chevron-left mr-1"></i> Prev</button>
        <button :disabled="!next" @click="fetchManagers(next)" class="btn-page">Next <i class="fas fa-chevron-right ml-1"></i></button>
      </div>
    </div>

    <AddManager v-if="showAddManager" :visible="showAddManager" @close="showAddManager = false" @success="fetchManagers" />
    <ConfirmModal :visible="showConfirm" :title="confirmTitle" :message="confirmMessage" @cancel="showConfirm = false" @confirm="confirmAction" />
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddManager from "./add.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Loading from "@/components/Loading.vue";

const SortIcon = {
  props: ["field", "sortKey", "sortAsc"],
  template: `<span class="inline-block ml-1 text-gray-400">
    <svg v-if="sortKey !== field" class="h-3 w-3 inline" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/></svg>
    <svg v-else-if="sortAsc" class="h-3 w-3 inline" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 13l4 4 4-4m0-6l-4-4-4 4"/></svg>
    <svg v-else class="h-3 w-3 inline" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/></svg>
  </span>`,
};

export default {
  name: "ManagersView",
  components: { SortIcon, Toast, AddManager, ConfirmModal, Loading },
  data() {
    return { searchTerm: "", sortKey: "fullName", sortAsc: true, managers: [], currentPage: 1, totalPages: 1, next: null, previous: null, pageSize: 10, pageSizes: [5,10,20,50,100], showAddManager: false, searchTimeout: null, showConfirm: false, selectedUser: null, selectedAction: null, loading: false };
  },
  computed: {
    confirmTitle() { return this.selectedAction === "activate" ? "Activate User" : "Deactivate User"; },
    confirmMessage() { return this.selectedAction === "activate" ? "Are you sure you want to activate this user?" : "Are you sure you want to deactivate this user?"; },
  },
  watch: {
    searchTerm(val) { clearTimeout(this.searchTimeout); this.searchTimeout = setTimeout(() => this.fetchManagers(null, val), 400); },
  },
  mounted() {
    if (this.$hasPermission("pms.view_ownermanager") || localStorage.getItem("is_superuser") === "true") this.fetchManagers();
    else this.$router.push({ name: "accessDenied" });
  },
  methods: {
    async fetchManagers(url = null, searchTerm = "") {
      this.loading = true;
      try {
        const result = await this.$getManagers(searchTerm);
        this.managers = result.managers; this.currentPage = result.currentPage; this.totalPages = result.totalPages; this.next = result.next; this.previous = result.previous;
      } catch (e) { console.error(e); this.managers = []; } finally { this.loading = false; }
    },
    sortBy(key) { this.sortKey === key ? (this.sortAsc = !this.sortAsc) : ((this.sortKey = key), (this.sortAsc = true)); },
    askConfirmation(action, id) { this.selectedUser = id; this.selectedAction = action; this.showConfirm = true; },
    confirmAction() {
      if (!this.selectedUser || !this.selectedAction) return;
      const p = this.selectedAction === "activate"
        ? this.$apiPost(`/activate_user/${this.selectedUser}`, { id: this.selectedUser })
        : this.$apiDelete(`/deactivate_user`, this.selectedUser);
      p.then(() => { this.$root.$refs.toast.showToast(this.selectedAction === "activate" ? "User activated" : "User deactivated", "success"); this.fetchManagers(); })
       .finally(() => { this.showConfirm = false; });
    },
  },
};
</script>

<style scoped>
.btn-action { @apply h-7 w-7 flex items-center justify-center rounded-lg text-xs transition; }
.btn-blue   { @apply bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white; }
.btn-green  { @apply bg-green-50 text-green-600 hover:bg-green-600 hover:text-white; }
.btn-orange { @apply bg-orange-50 text-orange-600 hover:bg-orange-600 hover:text-white; }
.btn-page   { @apply px-3 py-1.5 bg-white border border-gray-200 rounded-lg text-xs font-semibold text-gray-500 hover:text-primary hover:border-primary disabled:opacity-40 transition; }
</style>
