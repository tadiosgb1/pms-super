<template>
  <div>
    <Loading :visible="loading" message="Loading tenants..." />

    <div class="flex flex-col sm:flex-row gap-3 items-start sm:items-center justify-between mb-4">
      <div class="relative w-full sm:max-w-sm">
        <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
        <input v-model="searchTerm" type="search" placeholder="Search by name, email, phone..."
          class="w-full pl-9 pr-4 py-2 text-sm border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary/30" />
      </div>
      <div class="flex items-center gap-2 text-sm text-gray-500">
        Show
        <select v-model="pageSize" @change="fetchTenants()" class="border border-gray-200 rounded-lg px-2 py-1 text-sm focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option v-for="s in pageSizes" :key="s" :value="s">{{ s }}</option>
        </select>
        entries
      </div>
      <button @click="addTenantVisible = true" class="flex items-center gap-2 bg-primary text-white text-sm font-semibold px-4 py-2 rounded-lg hover:bg-primary/90 transition whitespace-nowrap">
        <i class="fas fa-plus-circle"></i> Add Tenant
      </button>
    </div>

    <div class="overflow-x-auto rounded-lg border border-gray-100">
      <table class="w-full text-sm">
        <thead class="bg-gray-50 text-xs font-semibold text-gray-500 uppercase tracking-wide">
          <tr>
            <th class="px-3 py-3 text-left cursor-pointer hover:text-primary" @click="sortBy('first_name')">
              Name <SortIcon field="first_name" :sort-key="sortKey" :sort-asc="sortAsc" />
            </th>
            <th class="px-3 py-3 text-left">Email</th>
            <th class="px-3 py-3 text-left">Phone</th>
            <th class="px-3 py-3 text-left">Property</th>
            <th class="px-3 py-3 text-center">Status</th>
            <th class="px-3 py-3 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="tenant in tenants" :key="tenant.id" class="hover:bg-gray-50 transition">
            <!-- Name + role badges on the same row below name -->
            <td class="px-3 py-2">
              <div class="flex items-center gap-2">
                <div class="w-7 h-7 rounded-full bg-primary/10 text-primary font-bold text-xs flex items-center justify-center uppercase shrink-0">
                  {{ ((tenant.first_name||'')[0]||'') }}{{ ((tenant.last_name||'')[0]||'') }}
                </div>
                <div>
                  <div class="font-medium text-gray-800 whitespace-nowrap">{{ formatFullName(tenant) }}</div>
    
                </div>
              </div>
            </td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ tenant.email || '—' }}</td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ tenant.phone_number || '—' }}</td>
            <td class="px-3 py-2 text-xs text-gray-700">{{ tenant.property_id?.name || '—' }}</td>
            <td class="px-3 py-2 text-center">
              <span :class="tenant.is_active ? 'bg-green-100 text-green-700' : 'bg-red-100 text-red-700'"
                class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase">
                {{ tenant.is_active ? 'Active' : 'Inactive' }}
              </span>
            </td>
            <td class="px-3 py-2 text-right">
              <div class="flex justify-end gap-1">
                <button @click="goToDetail(tenant.id)" class="btn-action btn-blue"><i class="fas fa-eye"></i></button>
                <button v-if="!tenant.is_active" @click="askConfirmation('activate', tenant.id)" class="btn-action btn-green"><i class="fas fa-check"></i></button>
                <button v-else @click="askConfirmation('deactivate', tenant.id)" class="btn-action btn-orange"><i class="fas fa-power-off"></i></button>
              </div>
            </td>
          </tr>
          <tr v-if="!tenants.length && !loading">
            <td colspan="6" class="px-3 py-10 text-center text-sm text-gray-400 italic">No tenants found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="flex items-center justify-between mt-4 text-sm text-gray-500">
      <span>Page {{ currentPage }} / {{ totalPages }}</span>
      <div class="flex gap-2">
        <button :disabled="!previous" @click="fetchTenants(previous)" class="btn-page"><i class="fas fa-chevron-left mr-1"></i> Prev</button>
        <button :disabled="!next" @click="fetchTenants(next)" class="btn-page">Next <i class="fas fa-chevron-right ml-1"></i></button>
      </div>
    </div>

    <ConfirmModal :visible="showConfirm" :title="confirmTitle" :message="confirmMessage" @cancel="showConfirm = false" @confirm="confirmAction" />
    <addTenant v-if="addTenantVisible" :visible="addTenantVisible" @close="addTenantVisible = false" @success="fetchTenants" />
  </div>
</template>

<script>
import ConfirmModal from "@/components/ConfirmModal.vue";
import Toast from "@/components/Toast.vue";
import addTenant from "./add.vue";
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
  name: "TenantView",
  components: { SortIcon, ConfirmModal, Toast, addTenant, Loading },
  data() {
    return { addTenantVisible: false, searchTerm: "", tenants: [], sortKey: "id", sortAsc: true, currentPage: 1, totalPages: 1, next: null, previous: null, pageSize: 10, pageSizes: [5,10,20,50,100], showConfirm: false, selectedUser: null, selectedAction: null, loading: false };
  },
  computed: {
    confirmTitle() { return this.selectedAction === "activate" ? "Activate User" : "Deactivate User"; },
    confirmMessage() { return this.selectedAction === "activate" ? "Are you sure you want to activate this user?" : "Are you sure you want to deactivate this user?"; },
  },
  watch: { searchTerm() { this.fetchTenants(); } },
  mounted() { this.fetchTenants(); },
  methods: {
    askConfirmation(action, id) { this.selectedUser = id; this.selectedAction = action; this.showConfirm = true; },
    confirmAction() {
      if (!this.selectedUser || !this.selectedAction) return;
      const p = this.selectedAction === "activate"
        ? this.$apiPost(`/activate_user/${this.selectedUser}`, { id: this.selectedUser })
        : this.$apiDelete(`/deactivate_user`, this.selectedUser);
      p.then(() => { this.$root.$refs.toast.showToast(this.selectedAction === "activate" ? "User activated" : "User deactivated", "success"); this.fetchTenants(); })
       .finally(() => { this.showConfirm = false; });
    },
    goToDetail(id) { this.$router.push(`/user_detail/${id}`); },
    async fetchTenants(url) {
      this.loading = true;
      try {
        const res = await this.$getTenants(url, this.pageSize, this.searchTerm);
        this.tenants = res.tenants || res.data || [];
        this.currentPage = res.currentPage || 1; this.totalPages = res.totalPages || 1;
        this.next = res.next || null; this.previous = res.previous || null;
      } catch (e) { console.error(e); this.tenants = []; } finally { this.loading = false; }
    },
    sortBy(key) { this.sortKey === key ? (this.sortAsc = !this.sortAsc) : ((this.sortKey = key), (this.sortAsc = true)); },
    formatDate(d) { return d ? new Date(d).toLocaleDateString() : ""; },
    formatFullName(t) { return [t.first_name, t.middle_name, t.last_name].filter(Boolean).join(" ") || `Tenant #${t.id}`; },
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
