
<template>
  <div class="p-6 bg-gradient-to-br from-slate-50 via-white to-tertiary/10 min-h-screen text-sm text-slate-800 relative">
    <!-- Loading -->
    <Loading :visible="loading" message="Loading Property Types..." />

    <!-- Page Header with Gradient -->
    <div class="flex flex-col md:flex-row md:items-center justify-between mb-8 gap-4 pb-6 border-b-2 border-primary/20">
      <div class="flex items-center gap-4">
        <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-primary to-dprimary flex items-center justify-center text-white">
          <i class="fas fa-building text-2xl"></i>
        </div>
        <div>
          <h1 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Property Types</h1>
          <p class="text-xs text-slate-500 font-bold uppercase tracking-wider mt-1">Manage All Property Categories</p>
        </div>
      </div>
      <button 
        @click="openAddModal" 
        class="bg-gradient-to-r from-primary to-dprimary hover:from-dprimary hover:to-primary text-white px-6 py-3 rounded-xl font-black flex items-center justify-center gap-2 text-xs uppercase tracking-wider transition-all duration-300 hover:scale-105 active:scale-95"
      >
        <i class="fas fa-plus-circle text-lg"></i>
        <span>Add Property Type</span>
      </button>
    </div>

    <!-- Search + Page Size -->
    <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between mb-8 gap-4">
      <div class="relative flex-1 max-w-md">
        <i class="fas fa-search absolute left-4 top-1/2 transform -translate-y-1/2 text-primary"></i>
        <input 
          v-model="searchQuery" 
          @input="fetchItems(1)" 
          type="text" 
          placeholder="Search property types..."
          class="border-2 border-slate-200 rounded-xl pl-12 pr-4 py-3 text-sm w-full focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all duration-200 hover:border-primary/30" 
        />
      </div>
      <div class="flex items-center gap-3 bg-white px-4 py-3 rounded-xl border-2 border-slate-200">
        <i class="fas fa-list text-primary"></i>
        <label class="text-xs font-black text-slate-600 uppercase tracking-wider">Show</label>
        <select 
          v-model="pageSize" 
          @change="fetchItems(1)" 
          class="border-2 border-slate-200 rounded-lg px-3 py-1.5 text-sm bg-white focus:ring-2 focus:ring-primary/50 focus:border-primary font-bold text-slate-700 cursor-pointer"
        >
          <option v-for="size in [5,10,20,50,100]" :key="size" :value="size">{{ size }}</option>
        </select>
        <span class="text-xs font-black text-slate-600 uppercase tracking-wider">entries</span>
      </div>
    </div>

    <!-- Desktop Table -->
    <div class="bg-white overflow-hidden rounded-[2rem] border-2 border-slate-200 hidden md:block">
      <div class="overflow-x-auto">
        <table class="min-w-full text-sm">
          <thead class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 text-slate-800 border-b-2 border-slate-200">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-black uppercase tracking-widest">
                <i class="fas fa-hashtag text-primary mr-2"></i>#
              </th>
              <th class="px-6 py-3 text-left text-xs font-black uppercase tracking-widest">
                <i class="fas fa-tag text-primary mr-2"></i>Name
              </th>
              <th class="px-6 py-3 text-left text-xs font-black uppercase tracking-widest">
                <i class="fas fa-align-left text-primary mr-2"></i>Description
              </th>
              <th class="px-6 py-3 text-center text-xs font-black uppercase tracking-widest">
                <i class="fas fa-cog text-primary mr-2"></i>Actions
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100">
            <tr 
              v-for="(item, index) in items" 
              :key="item.id" 
              class="hover:bg-slate-50 transition-colors duration-150 border-b border-slate-100"
            >
              <td class="px-6 py-3">
                <span class="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-primary/10 text-primary font-black text-xs">
                  {{ index + 1 }}
                </span>
              </td>
              <td class="px-6 py-3">
                <span class="font-bold text-slate-800">{{ item.name }}</span>
              </td>
              <td class="px-6 py-3">
                <span class="text-slate-600 truncate max-w-xs">{{ item.description }}</span>
              </td>
              <td class="px-6 py-3">
                <div class="flex items-center justify-center gap-2">
                  <button 
                    @click="viewDetails(item.id)" 
                    class="w-8 h-8 rounded-lg bg-primary/10 text-primary hover:bg-primary hover:text-white transition-all duration-200 flex items-center justify-center text-xs"
                    title="View Details"
                  >
                    <i class="fas fa-eye"></i>
                  </button>
                  <button 
                    @click="editItem(item)" 
                    class="w-8 h-8 rounded-lg bg-secondary/10 text-secondary hover:bg-secondary hover:text-white transition-all duration-200 flex items-center justify-center text-xs"
                    title="Edit"
                  >
                    <i class="fas fa-edit"></i>
                  </button>
                  <button 
                    @click="openDeleteModal(item.id)" 
                    class="w-8 h-8 rounded-lg bg-red-100 text-red-600 hover:bg-red-600 hover:text-white transition-all duration-200 flex items-center justify-center text-xs"
                    title="Delete"
                  >
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </td>
            </tr>
            <tr v-if="items.length === 0">
              <td colspan="4" class="text-center py-8">
                <div class="flex flex-col items-center gap-3">
                  <i class="fas fa-inbox text-4xl text-slate-300"></i>
                  <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No property types found</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Mobile Cards -->
    <div class="md:hidden space-y-4">
      <div 
        v-for="(item, index) in items" 
        :key="item.id" 
        class="bg-white border-2 border-slate-200 rounded-[1.5rem] shadow-lg overflow-hidden hover:shadow-xl hover:border-primary/30 transition-all duration-300"
      >
        <div class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 px-5 py-4 border-b-2 border-slate-200">
          <div class="flex justify-between items-center">
            <div class="flex items-center gap-3">
              <span class="w-8 h-8 rounded-lg bg-primary text-white flex items-center justify-center font-black text-xs shadow-md">
                {{ index + 1 }}
              </span>
              <h2 class="font-black text-slate-800 uppercase tracking-tight">Property Type</h2>
            </div>
            <div class="flex gap-2">
              <button 
                @click="viewDetails(item.id)" 
                class="w-8 h-8 rounded-lg bg-primary/20 text-primary hover:bg-primary hover:text-white transition-all duration-200 flex items-center justify-center"
              >
                <i class="fas fa-eye text-xs"></i>
              </button>
              <button 
                @click="editItem(item)" 
                class="w-8 h-8 rounded-lg bg-secondary/20 text-secondary hover:bg-secondary hover:text-white transition-all duration-200 flex items-center justify-center"
              >
                <i class="fas fa-edit text-xs"></i>
              </button>
              <button 
                @click="openDeleteModal(item.id)" 
                class="w-8 h-8 rounded-lg bg-red-100 text-red-600 hover:bg-red-600 hover:text-white transition-all duration-200 flex items-center justify-center"
              >
                <i class="fas fa-trash text-xs"></i>
              </button>
            </div>
          </div>
        </div>
        
        <div class="p-5 space-y-3">
          <div class="bg-gradient-to-br from-primary/5 to-primary/10 rounded-xl p-3 border-2 border-primary/20">
            <div class="flex items-center gap-2 mb-1">
              <i class="fas fa-tag text-primary text-xs"></i>
              <span class="text-xs font-black text-slate-500 uppercase">Name</span>
            </div>
            <p class="font-bold text-slate-800">{{ item.name }}</p>
          </div>
          
          <div class="bg-gradient-to-br from-tertiary/30 to-primary/5 rounded-xl p-3 border-2 border-primary/20">
            <div class="flex items-center gap-2 mb-1">
              <i class="fas fa-align-left text-primary text-xs"></i>
              <span class="text-xs font-black text-slate-500 uppercase">Description</span>
            </div>
            <p class="text-slate-600 text-sm">{{ item.description }}</p>
          </div>
        </div>
      </div>
      
      <div v-if="items.length === 0" class="bg-white rounded-[1.5rem] border-2 border-slate-200 p-12 text-center">
        <i class="fas fa-inbox text-5xl text-slate-300 mb-4"></i>
        <p class="text-slate-400 font-bold uppercase tracking-wider">No property types found</p>
      </div>
    </div>

    <!-- Pagination -->
    <div class="flex flex-col sm:flex-row items-center justify-between mt-8 gap-4 bg-white px-6 py-4 rounded-xl border-2 border-slate-200">
      <span class="text-xs font-bold text-slate-600 uppercase tracking-wider">
        <i class="fas fa-info-circle text-primary mr-2"></i>
        Showing <span class="text-primary font-black">{{ (currentPage - 1) * pageSize + 1 }}</span> 
        to <span class="text-primary font-black">{{ Math.min(currentPage * pageSize, count) }}</span> 
        of <span class="text-primary font-black">{{ count }}</span> entries
      </span>
      <div class="flex items-center gap-2">
        <button 
          @click="fetchItems(currentPage - 1)" 
          :disabled="!previousPage"
          class="px-4 py-2 border-2 border-slate-200 rounded-lg hover:bg-primary hover:text-white hover:border-primary disabled:opacity-30 disabled:cursor-not-allowed disabled:hover:bg-transparent disabled:hover:text-slate-600 transition-all duration-200 font-bold text-xs uppercase tracking-wider flex items-center gap-2"
        >
          <i class="fas fa-chevron-left"></i>Previous
        </button>
        <span class="px-4 py-2 bg-gradient-to-r from-primary to-dprimary text-white rounded-lg font-black min-w-[3rem] text-center text-xs">
          {{ currentPage }}
        </span>
        <button 
          @click="fetchItems(currentPage + 1)" 
          :disabled="!nextPage"
          class="px-4 py-2 border-2 border-slate-200 rounded-lg hover:bg-primary hover:text-white hover:border-primary disabled:opacity-30 disabled:cursor-not-allowed disabled:hover:bg-transparent disabled:hover:text-slate-600 transition-all duration-200 font-bold text-xs uppercase tracking-wider flex items-center gap-2"
        >
          Next<i class="fas fa-chevron-right"></i>
        </button>
      </div>
    </div>

    <!-- Add/Edit Modal -->
    <AddPropertyTypes v-if="showModal && !editMode" :data="selectedItem" @close="showModal=false" @saved="fetchItems"/>
    <EditPropertyTypes v-if="showModal && editMode" :data="selectedItem" @close="showModal=false" @saved="fetchItems"/>

    <!-- Delete Confirmation Modal -->
    <delete-confirm-modal 
      :visible="deleteModalVisible"
      title="Delete Property Type"
      message="Are you sure you want to delete this property type? This action cannot be undone."
      @confirm="confirmDelete"
      @cancel="deleteModalVisible=false"
    />
  </div>
</template>

<script>
import AddPropertyTypes from "./AddPropertyTypes.vue";
import EditPropertyTypes from "./EditPropertyTypes.vue";
import Loading from "@/components/Loading.vue";
import DeleteConfirmModal from "@/components/DeleteConfirmModal.vue";

export default {
  components: { AddPropertyTypes, EditPropertyTypes, Loading, DeleteConfirmModal },

  data() {
    return {
      items: [],
      count: 0,
      nextPage: null,
      previousPage: null,
      currentPage: 1,
      pageSize: 10,
      searchQuery: "",
      showModal: false,
      editMode: false,
      selectedItem: null,
      loading: false,
      deleteModalVisible: false,
      deleteId: null,
    };
  },

  methods: {
    async fetchItems(page = 1) {
      this.loading = true;
      this.currentPage = page;
      const params = { page: this.currentPage, page_size: this.pageSize, search: this.searchQuery };
      try {
        const response = await this.$apiGet('/get_property_types', params);
        this.items = response.data;
        this.count = response.count || 0;
        this.nextPage = response.next || null;
        this.previousPage = response.previous || null;
      } catch(e) { console.error(e); }
      finally { this.loading = false; }
    },

    openAddModal() { this.editMode = false; this.selectedItem = null; this.showModal = true; },
    editItem(item) { this.editMode = true; this.selectedItem = item; this.showModal = true; },
    
    // Navigate using static route name
    viewDetails(id) { 
      this.$router.push({ name: 'PropertyTypes-detail', params: { id } });
    },

    openDeleteModal(id) { this.deleteId = id; this.deleteModalVisible = true; },

    // Delete with toast
    async confirmDelete() {
      const res = await this.$apiDelete('/delete_property_type', this.deleteId);
      if(res) {
        this.$root.$refs.toast.showToast('PropertyTypes deleted successfully', 'success');
      }
      this.deleteModalVisible = false;
      this.fetchItems(this.currentPage);
    },
  },

  mounted() { this.fetchItems(); }
};
</script>
