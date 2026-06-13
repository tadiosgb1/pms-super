<template>
  <div>
    <Loading :visible="loading" message="Loading owners..." />

    <div class="flex flex-col sm:flex-row gap-3 items-start sm:items-center justify-between mb-4">
      <div class="relative w-full sm:max-w-sm">
        <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
        <input v-model="searchTerm" type="search" placeholder="Search by name, email or phone..."
          class="w-full pl-9 pr-4 py-2 text-sm border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary/30" />
      </div>
      <button v-if="showAdd" @click="visible = true" class="flex items-center gap-2 bg-primary text-white text-sm font-semibold px-4 py-2 rounded-lg hover:bg-primary/90 transition whitespace-nowrap">
        <i class="fas fa-plus-circle"></i> Add Owner
      </button>
    </div>

    <div class="overflow-x-auto rounded-lg border border-gray-100">
      <table class="w-full text-sm">
        <thead class="bg-gray-50 text-xs font-semibold text-gray-500 uppercase tracking-wide">
          <tr>
            <th class="px-3 py-3 text-left cursor-pointer hover:text-primary" @click="sortBy('full_name')">
              Name <SortIcon field="full_name" :sort-key="sortKey" :sort-asc="sortAsc" />
            </th>
            <th class="px-3 py-3 text-left">Email</th>
            <th class="px-3 py-3 text-left">Phone</th>
            <th class="px-3 py-3 text-center">Status</th>
            <th class="px-3 py-3 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="owner in owners" :key="owner.id" class="hover:bg-gray-50 transition">
            <td class="px-3 py-2">
              <div class="flex items-center gap-2">
                <div class="w-7 h-7 rounded-full bg-primary/10 text-primary font-bold text-xs flex items-center justify-center uppercase shrink-0">
                  {{ owner.full_name?.split(' ').map(n=>n[0]).slice(0,2).join('') || '?' }}
                </div>
                <span class="font-medium text-gray-800 whitespace-nowrap">{{ owner.full_name || '—' }}</span>
              </div>
            </td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ owner.email || '—' }}</td>
            <td class="px-3 py-2 text-gray-500 text-xs">{{ owner.phoneNumber || '—' }}</td>
            <td class="px-3 py-2 text-center">
              <span :class="owner.is_active ? 'bg-green-100 text-green-700' : 'bg-orange-100 text-orange-700'"
                class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase">
                {{ owner.is_active ? 'Active' : 'Inactive' }}
              </span>
            </td>
            <td class="px-3 py-2 text-right">
              <div class="flex justify-end gap-1">
                <button @click="goToDetail(owner.id)" class="btn-action btn-blue"><i class="fas fa-eye"></i></button>
                <button v-if="!owner.is_active" @click="askConfirmation('activate', owner.id)" class="btn-action btn-green"><i class="fas fa-power-off"></i></button>
                <button v-else @click="askConfirmation('deactivate', owner.id)" class="btn-action btn-orange"><i class="fas fa-ban"></i></button>
              </div>
            </td>
          </tr>
          <tr v-if="!owners.length && !loading">
            <td colspan="5" class="px-3 py-10 text-center text-sm text-gray-400 italic">No owners found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <AddOwner v-if="visible" :visible="visible" @close="visible = false" @saved="fetchOwners" />
    <UpdateOwner v-if="updateVisible" :visible="updateVisible" :ownerData="selectedOwner" @close="updateVisible = false" @updated="fetchOwners" />
    <ConfirmModal :visible="showConfirm" :title="confirmTitle" :message="confirmMessage" @cancel="showConfirm = false" @confirm="confirmAction" />
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddOwner from "./add.vue";
import UpdateOwner from "./edit.vue";
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
  name: "OwnersView",
  components: { SortIcon, AddOwner, UpdateOwner, Toast, ConfirmModal, Loading },
  data() {
    return { searchTerm: "", visible: false, updateVisible: false, selectedOwner: null, sortKey: "full_name", sortAsc: true, owners: [], ordering: "-id", showConfirm: false, selectedUser: null, selectedAction: null, loading: false, showAdd: true };
  },
  computed: {
    confirmTitle() { return this.selectedAction === "activate" ? "Activate User" : "Deactivate User"; },
    confirmMessage() { return this.selectedAction === "activate" ? "Are you sure you want to activate this user?" : "Are you sure you want to deactivate this user?"; },
  },
  mounted() { this.fetchOwners(); },
  methods: {
    async fetchOwners() {
      this.loading = true;
      try { const res = await this.$apiGet(`/get_owners?search=${this.searchTerm}&ordering=${this.ordering}`); this.owners = res.owners || []; }
      catch (e) { console.error(e); this.owners = []; } finally { this.loading = false; }
    },
    askConfirmation(action, id) { this.selectedUser = id; this.selectedAction = action; this.showConfirm = true; },
    confirmAction() {
      if (!this.selectedUser || !this.selectedAction) return;
      const p = this.selectedAction === "activate"
        ? this.$apiPost(`/activate_user/${this.selectedUser}`, { id: this.selectedUser })
        : this.$apiDelete(`/deactivate_user`, this.selectedUser);
      p.then(() => { this.$root.$refs.toast.showToast(this.selectedAction === "activate" ? "User activated" : "User deactivated", "success"); this.fetchOwners(); })
       .finally(() => { this.showConfirm = false; });
    },
    sortBy(key) { this.sortKey === key ? (this.sortAsc = !this.sortAsc) : ((this.sortKey = key), (this.sortAsc = true)); },
    goToDetail(id) { this.$router.push(`/user_detail/${id}`); },
  },
};
</script>

<style scoped>
.btn-action { @apply h-7 w-7 flex items-center justify-center rounded-lg text-xs transition; }
.btn-blue   { @apply bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white; }
.btn-green  { @apply bg-green-50 text-green-600 hover:bg-green-600 hover:text-white; }
.btn-orange { @apply bg-orange-50 text-orange-600 hover:bg-orange-600 hover:text-white; }
</style>
