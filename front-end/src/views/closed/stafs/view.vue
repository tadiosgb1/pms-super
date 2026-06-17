<template>
  <div>
    <Loading :visible="loading" message="Loading staff..." />

    <!-- Page Header (shown when accessed as a standalone route) -->
    <div v-if="$route && $route.name === 'staffs'" class="flex items-center gap-4 mb-6 pb-5 border-b border-gray-200">
      <div class="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center text-primary">
        <i class="fas fa-users text-xl"></i>
      </div>
      <div>
        <h1 class="text-xl font-black text-gray-800 tracking-tight">Staffs</h1>
        <p class="text-xs text-gray-400 font-semibold uppercase tracking-wider mt-0.5">Manage Owner Staff Members</p>
      </div>
    </div>

    <div class="flex flex-col sm:flex-row gap-3 items-start sm:items-center justify-between mb-4">
      <div class="relative w-full sm:max-w-sm">
        <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
        <input v-model="searchTerm" type="search" placeholder="Search staff by name, email or group..."
          class="w-full pl-9 pr-4 py-2 text-sm border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary/30" />
      </div>
      <div class="flex items-center gap-2 text-sm text-gray-500">
        Show
        <select v-model="pageSize" @change="fetchStaffs()" class="border border-gray-200 rounded-lg px-2 py-1 text-sm focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option v-for="s in pageSizes" :key="s" :value="s">{{ s }}</option>
        </select>
        entries
      </div>
      <button @click="showAddStaff = true" class="flex items-center gap-2 bg-primary text-white text-sm font-semibold px-4 py-2 rounded-lg hover:bg-primary/90 transition whitespace-nowrap">
        <i class="fas fa-plus-circle"></i> Add Staff
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
            <th class="px-3 py-3 text-left">Phone</th>
            <th class="px-3 py-3 text-left">Groups</th>
            <th class="px-3 py-3 text-center">Status</th>
            <th class="px-3 py-3 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="staff in filteredAndSortedStaffs" :key="staff.id" class="hover:bg-gray-50 transition">
            <td class="px-3 py-2">
              <div class="flex items-center gap-2">
                <div class="w-7 h-7 rounded-full bg-primary/10 text-primary font-bold text-xs flex items-center justify-center uppercase shrink-0">
                  {{ ((staff.staff.first_name||'')[0]||'') }}{{ ((staff.staff.last_name||'')[0]||'') }}
                </div>
                <span class="font-medium text-gray-800 whitespace-nowrap">
                  {{ staff.staff.first_name }} {{ staff.staff.middle_name }} {{ staff.staff.last_name }}
                </span>
              </div>
            </td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ staff.staff.email || '—' }}</td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ staff.staff.phone_number || '—' }}</td>
            <td class="px-3 py-2">
              <div class="flex flex-wrap gap-1">
                <span v-for="g in staff.staff.groups" :key="g" class="px-2 py-0.5 bg-gray-100 text-gray-600 rounded text-[10px] font-semibold uppercase border border-gray-200">{{ g }}</span>
                <span v-if="!staff.staff.groups?.length" class="text-xs text-gray-300 italic">—</span>
              </div>
            </td>
            <td class="px-3 py-2 text-center">
              <span :class="staff.staff.is_active ? 'bg-green-100 text-green-700' : 'bg-orange-100 text-orange-700'"
                class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase">
                {{ staff.staff.is_active ? 'Active' : 'Inactive' }}
              </span>
            </td>
            <td class="px-3 py-2 text-right">
              <div class="flex justify-end gap-1">
                <router-link :to="`/user_detail/${staff.staff.id}`" class="btn-action btn-blue"><i class="fas fa-eye"></i></router-link>
                <button v-if="!staff.staff.is_active" @click="activateUser(staff.id)" class="btn-action btn-green"><i class="fas fa-power-off"></i></button>
                <button v-else @click="deactivateUser(staff.id)" class="btn-action btn-orange"><i class="fas fa-ban"></i></button>
              </div>
            </td>
          </tr>
          <tr v-if="!filteredAndSortedStaffs.length && !loading">
            <td colspan="6" class="px-3 py-10 text-center text-sm text-gray-400 italic">No staff found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="flex items-center justify-between mt-4 text-sm text-gray-500">
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <div class="flex gap-2">
        <button :disabled="!previous" @click="fetchStaffs(previous)" class="btn-page"><i class="fas fa-chevron-left mr-1"></i> Prev</button>
        <button :disabled="!next" @click="fetchStaffs(next)" class="btn-page">Next <i class="fas fa-chevron-right ml-1"></i></button>
      </div>
    </div>

    <AddStaff v-if="showAddStaff" :visible="showAddStaff" @close="showAddStaff = false" @success="fetchStaffs()" />
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddStaff from "./add.vue";
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
  name: "StaffsView",
  components: { SortIcon, Toast, AddStaff, Loading },
  data() {
    return { searchTerm: "", sortKey: "fullName", sortAsc: true, staffs: [], currentPage: 1, totalPages: 1, next: null, previous: null, pageSize: 20, pageSizes: [5,10,20,50,100], showAddStaff: false, loading: false };
  },
  computed: {
    filteredAndSortedStaffs() {
      const term = this.searchTerm.toLowerCase();
      let filtered = this.staffs.filter(s => {
        const u = s.staff || {};
        return `${u.first_name||""} ${u.middle_name||""} ${u.last_name||""}`.toLowerCase().includes(term)
          || (u.email||"").toLowerCase().includes(term)
          || (u.phone_number||"").toLowerCase().includes(term)
          || (u.groups||[]).join(", ").toLowerCase().includes(term);
      });
      filtered.sort((a, b) => {
        const an = `${a.staff.first_name||""} ${a.staff.last_name||""}`.toLowerCase();
        const bn = `${b.staff.first_name||""} ${b.staff.last_name||""}`.toLowerCase();
        return an < bn ? (this.sortAsc ? -1 : 1) : an > bn ? (this.sortAsc ? 1 : -1) : 0;
      });
      return filtered;
    },
  },
  mounted() { this.fetchStaffs(); },
  methods: {
    async fetchStaffs(customUrl = null) {
      this.loading = true;
      try {
        const isSuperuser = localStorage.getItem("is_superuser") === "true";
        const params = isSuperuser ? {} : { owner__id: localStorage.getItem("userId") };
        const res = await this.$apiGet(customUrl || "get_owner_staffs", params);
        this.staffs = Array.isArray(res.data) ? res.data : [];
      } catch (e) { console.error(e); this.staffs = []; } finally { this.loading = false; }
    },
    sortBy(key) { this.sortKey === key ? (this.sortAsc = !this.sortAsc) : ((this.sortKey = key), (this.sortAsc = true)); },
    activateUser(id) {
      this.$apiPost(`/activate_user/${id}`, { id }).then(() => { this.$root.$refs.toast.showToast("Staff activated", "success"); this.fetchStaffs(); });
    },
    deactivateUser(id) {
      this.$apiDelete(`/deactivate_user`, id).then(() => { this.$root.$refs.toast.showToast("Staff deactivated", "success"); this.fetchStaffs(); });
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
