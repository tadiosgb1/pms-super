<template>
  <div class="p-4 bg-gray-50 min-h-screen">
    <Loading :visible="loading" message="Loading properties..." />

    <!-- Page Header -->
    <div class="flex items-center gap-4 mb-6 pb-5 border-b border-gray-200">
      <div class="w-12 h-12 rounded-xl bg-primary/10 flex items-center justify-center text-primary">
        <i class="fas fa-building text-xl"></i>
      </div>
      <div>
        <h1 class="text-xl font-black text-gray-800 tracking-tight">All Properties</h1>
        <p class="text-xs text-gray-400 font-semibold uppercase tracking-wider mt-0.5">Manage Real Estate Portfolio</p>
      </div>
    </div>

    <!-- Toolbar -->
    <div class="flex flex-col sm:flex-row gap-3 items-start sm:items-center justify-between mb-4">
      <div class="flex flex-wrap gap-3 items-center flex-1">
        <div class="relative w-full sm:max-w-xs">
          <i class="fas fa-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs"></i>
          <input v-model="searchTerm" @input="fetchProperties()" type="search"
            placeholder="Name, city, owner..."
            class="w-full pl-9 pr-4 py-2 text-sm border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-primary/30 bg-white" />
        </div>

        <select v-if="!zone_id_query_set" v-model="zone_id" @change="fetchProperties()"
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm bg-white focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option value="">All Zones</option>
          <option v-for="z in zones" :key="z.id" :value="z.id">{{ z.name }}</option>
        </select>

        <select v-model="status" @change="fetchProperties()"
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm bg-white focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option value="">All Status</option>
          <option value="rent">Rented</option>
          <option value="sale">For Sale</option>
          <option value="available">Available</option>
          <option value="under_maintenance">Maintenance</option>
        </select>

        <select v-model="pageSize" @change="fetchProperties()"
          class="border border-gray-200 rounded-lg px-3 py-2 text-sm bg-white focus:outline-none focus:ring-2 focus:ring-primary/30">
          <option v-for="s in pageSizes" :key="s" :value="s">{{ s }} / page</option>
        </select>
      </div>

      <button v-if="$hasPermission('pms.add_property')" @click="visible = true"
        class="flex items-center gap-2 bg-primary text-white text-sm font-semibold px-4 py-2 rounded-lg hover:bg-primary/90 transition whitespace-nowrap">
        <i class="fas fa-plus"></i> Add Property
      </button>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto rounded-lg border border-gray-100 bg-white">
      <table class="w-full text-sm">
        <thead class="bg-gray-50 text-xs font-semibold text-gray-500 uppercase tracking-wide">
          <tr>
            <th class="px-3 py-3 text-left cursor-pointer hover:text-primary" @click="changeOrdering('name')">
              Property
            </th>
            <th class="px-3 py-3 text-left">Owner / Manager</th>
            <th class="px-3 py-3 text-left">Location</th>
            <th class="px-3 py-3 text-left">Type</th>
            <th class="px-3 py-3 text-center">Status</th>
            <th class="px-3 py-3 text-right">Actions</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="p in properties" :key="p.id" class="hover:bg-gray-50 transition">
            <td class="px-3 py-2">
              <div class="flex items-center gap-2">
                <div class="w-7 h-7 rounded-lg bg-primary/10 text-primary font-bold text-xs flex items-center justify-center uppercase shrink-0">
                  {{ (p.name||'').substring(0,2) }}
                </div>
                <span class="font-medium text-gray-800 whitespace-nowrap">{{ p.name }}</span>
              </div>
            </td>
            <td class="px-3 py-2 text-xs text-gray-600">
              <div>
                <span class="cursor-pointer hover:text-primary transition" @click="goToUserDetail(p.owner_id)">
                  {{ p.ownerName || '—' }}
                </span>
              </div>
              <div class="text-gray-400">{{ p.managerName || '—' }}</div>
            </td>
            <td class="px-3 py-2 text-xs text-gray-600">
              <div>{{ p.city || '—' }}</div>
              <div class="text-primary cursor-pointer hover:underline" @click="goToZoneDetail(p.property_zone_id)">
                {{ p.zoneName || '—' }}
              </div>
            </td>
            <td class="px-3 py-2 text-xs text-gray-500 capitalize">{{ p.property_type || '—' }}</td>
            <td class="px-3 py-2 text-center">
              <span class="px-2 py-0.5 rounded-full text-[10px] font-semibold uppercase"
                :class="{
                  'bg-green-100 text-green-700': p.status === 'available',
                  'bg-blue-100 text-blue-700': p.status === 'rent',
                  'bg-orange-100 text-orange-700': p.status === 'sale',
                  'bg-pink-100 text-pink-700': p.status === 'under_maintenance'
                }">
                {{ p.status }}
              </span>
            </td>
            <td class="px-3 py-2 text-right">
              <div class="flex justify-end gap-1">
                <button @click="goToDetail(p.id)" class="btn-action btn-green" title="View"><i class="fas fa-eye"></i></button>
                <button v-if="$hasPermission('pms.change_property')" @click="editProperty(p)" class="btn-action btn-blue" title="Edit"><i class="fas fa-edit"></i></button>
                <button @click="rentPay(p.id)" class="btn-action btn-gray" title="Payments"><i class="fas fa-receipt"></i></button>
                <button @click="openSaleModal(p.id)" class="btn-action btn-orange" title="Sale"><i class="fas fa-tag"></i></button>
                <button v-if="$hasPermission('pms.delete_property')" @click="askDeleteConfirmation(p)" class="btn-action btn-red" title="Delete"><i class="fas fa-trash-alt"></i></button>
              </div>
            </td>
          </tr>
          <tr v-if="!properties.length && !loading">
            <td colspan="6" class="px-3 py-10 text-center text-sm text-gray-400 italic">No properties found.</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="flex items-center justify-between mt-4 text-sm text-gray-500">
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <div class="flex gap-2">
        <button :disabled="!previous" @click="fetchProperties(previous)" class="btn-page"><i class="fas fa-chevron-left mr-1"></i> Prev</button>
        <button :disabled="!next" @click="fetchProperties(next)" class="btn-page">Next <i class="fas fa-chevron-right ml-1"></i></button>
      </div>
    </div>

    <AddProperty v-if="visible" :visible="visible" @close="visible = false" @refresh="fetchProperties" />
    <UpdateProperty v-if="updateVisible" :visible="updateVisible" :property="propertyToEdit" @close="updateVisible = false" @refresh="fetchProperties" />
    <ConfirmModal v-if="confirmVisible" :visible="confirmVisible" title="Delete Property"
      message="This is permanent. Remove this property?" @confirm="confirmDelete" @cancel="confirmVisible = false" />
    <SaleModal v-if="saleVisible" :visible="saleVisible" :propertyId="salePropertyId" sourceType="property"
      @close="saleVisible = false" @refresh="fetchProperties" />
  </div>
</template>

<script>
import AddProperty from "@/views/closed/proporty/add.vue";
import UpdateProperty from "@/views/closed/proporty/update.vue";
import ConfirmModal from "@/components/ConfirmModal.vue";
import Toast from "@/components/Toast.vue";
import SaleModal from "../propertiesListForSale/add.vue";
import Loading from "@/components/Loading.vue";

export default {
  name: "PropertyView",
  components: { AddProperty, UpdateProperty, ConfirmModal, Toast, SaleModal, Loading },
  data() {
    return {
      zones: [], zone_id: "", zone_id_query_set: false,
      properties: [], visible: false, updateVisible: false, confirmVisible: false,
      propertyToEdit: null, propertyToDelete: null,
      searchTerm: "", currentPage: 1, totalPages: 1, next: null, previous: null,
      pageSize: 10, pageSizes: [10, 20, 50, 100, 1000],
      ordering: "-id", status: "",
      saleVisible: false, salePropertyId: null, loading: false,
    };
  },
  async created() {
    if (this.$route.query.zone_id) this.zone_id_query_set = true;
    const r = await this.$getZones();
    this.zones = r.zones || [];
    await this.fetchProperties();
  },
  methods: {
    async fetchProperties(url = null) {
      this.loading = true;
      try {
        const pageUrl = url || `/get_properties?page=${this.currentPage}&page_size=${this.pageSize}&search=${this.searchTerm}&ordering=${this.ordering}`;
        const params = { status: this.status };
        if (this.$route.query.zone_id || this.zone_id) params.property_zone_id = this.$route.query.zone_id || this.zone_id;
        if (this.searchTerm) params.search = this.searchTerm;
        const result = await this.$getProperties(pageUrl, params);
        this.properties = result.properties || [];
        this.currentPage = result.currentPage;
        this.totalPages = result.totalPages;
        this.next = result.next;
        this.previous = result.previous;
      } catch (e) {
        console.error(e); this.properties = [];
      } finally { this.loading = false; }
    },
    changeOrdering(field) {
      this.ordering = this.ordering === field ? `-${field}` : field;
      this.fetchProperties();
    },
    editProperty(p) { this.propertyToEdit = p; this.updateVisible = true; },
    askDeleteConfirmation(p) { this.propertyToDelete = p; this.confirmVisible = true; },
    async confirmDelete() {
      this.confirmVisible = false;
      try {
        const res = await this.$apiDelete(`/delete_property/${this.propertyToDelete.id}`);
        this.$root.$refs.toast.showToast(res.message || "Deleted", "success");
        this.fetchProperties();
      } catch (e) { this.$root.$refs.toast.showToast("Failed to delete", "error"); }
      this.propertyToDelete = null;
    },
    openSaleModal(id) { this.salePropertyId = id; this.saleVisible = true; },
    goToUserDetail(id) { if (id) this.$router.push(`/user_detail/${id}`); },
    goToZoneDetail(id) { if (id) this.$router.push(`/zones/${id}`); },
    goToDetail(id) { if (id) this.$router.push({ name: "PropertyDetail", params: { id } }); },
    rentPay(id) { if (id) this.$router.push({ name: "rentPay", params: { id } }); },
  },
};
</script>

<style scoped>
.btn-action { @apply h-7 w-7 flex items-center justify-center rounded-lg text-xs transition; }
.btn-blue   { @apply bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white; }
.btn-green  { @apply bg-green-50 text-green-600 hover:bg-green-600 hover:text-white; }
.btn-orange { @apply bg-orange-50 text-orange-600 hover:bg-orange-600 hover:text-white; }
.btn-red    { @apply bg-red-50 text-red-600 hover:bg-red-600 hover:text-white; }
.btn-gray   { @apply bg-gray-100 text-gray-500 hover:bg-gray-500 hover:text-white; }
.btn-page   { @apply px-3 py-1.5 bg-white border border-gray-200 rounded-lg text-xs font-semibold text-gray-500 hover:text-primary hover:border-primary disabled:opacity-40 transition; }
</style>
