<template>
  <div class="p-6 bg-gradient-to-br from-slate-50 via-white to-tertiary/10 min-h-screen text-sm text-slate-800 relative">
    <!-- Loading -->
    <Loading :visible="loading" message="Loading Broker Sales..." />

    <!-- Page Header with Gradient -->
    <div class="flex flex-col md:flex-row md:items-center justify-between mb-8 gap-4 pb-6 border-b-2 border-primary/20">
      <div class="flex items-center gap-4">
        <div class="w-14 h-14 rounded-2xl bg-gradient-to-br from-primary to-dprimary flex items-center justify-center text-white">
          <i class="fas fa-handshake text-2xl"></i>
        </div>
        <div>
          <h1 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Broker Sales</h1>
          <p class="text-xs text-slate-500 font-bold uppercase tracking-wider mt-1">Manage Property Sales</p>
        </div>
      </div>
      <button 
        @click="openAddModal" 
        class="bg-gradient-to-r from-primary to-dprimary hover:from-dprimary hover:to-primary text-white px-6 py-3 rounded-xl font-black flex items-center justify-center gap-2 text-xs uppercase tracking-wider transition-all hover:scale-105 active:scale-95"
      >
        <i class="fas fa-plus-circle text-lg"></i>
        <span>Add Sale</span>
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
          placeholder="Search sales..."
          class="border-2 border-slate-200 rounded-xl pl-12 pr-4 py-3 text-sm w-full focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary transition-all" 
        />
      </div>
      <div class="flex items-center gap-3 bg-white px-4 py-2.5 rounded-xl border-2 border-slate-200">
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
        <table class="w-full">
          <thead>
            <tr class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 border-b-2 border-slate-200">
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-hashtag text-primary mr-2"></i>#
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-tag text-primary mr-2"></i>Listing
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-dollar-sign text-primary mr-2"></i>Selling
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-receipt text-primary mr-2"></i>Fee
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-check-circle text-primary mr-2"></i>Status
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-home text-primary mr-2"></i>Property
              </th>
              <th class="px-4 py-2 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-user-tie text-primary mr-2"></i>Broker
              </th>
              <th class="px-4 py-2 text-center text-xs font-black text-slate-700 uppercase tracking-widest">
                <i class="fas fa-cog text-primary mr-2"></i>Actions
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100">
            <tr v-for="(item, index) in items" :key="item.id" class="hover:bg-slate-50 transition-colors duration-150">
              <td class="px-4 py-2">
                <span class="inline-flex items-center justify-center w-6 h-6 rounded-lg bg-primary/10 text-primary font-black text-xs">
                  {{ index + 1 }}
                </span>
              </td>
              <td class="px-4 py-2">
                <span class="font-bold text-slate-800 text-xs">${{ item.listing_price }}</span>
              </td>
              <td class="px-4 py-2">
                <span class="font-bold text-emerald-600 text-xs">${{ item.selling_price }}</span>
              </td>
              <td class="px-4 py-2">
                <span class="font-bold text-slate-700 text-xs">${{ item.posting_fee }}</span>
              </td>
              <td class="px-4 py-2">
                <span class="px-2 py-1 rounded-lg text-xs font-black uppercase" :class="item.posting_payment_status === 'approved' ? 'bg-emerald-100 text-emerald-700' : 'bg-amber-100 text-amber-700'">
                  {{ item.posting_payment_status }}
                </span>
              </td>
              <td class="px-4 py-2">
                <span class="font-bold text-slate-700 text-xs">{{ item.property?.name || 'N/A' }}</span>
              </td>
              <td class="px-4 py-2">
                <span class="font-bold text-slate-700 text-xs">{{ item.broker?.first_name || 'N/A' }}</span>
              </td>
              <td class="px-4 py-2 text-center">
                <div class="flex items-center justify-center gap-1">
                  <button @click="viewDetails(item.id)" class="w-7 h-7 bg-primary/10 text-primary rounded-lg hover:bg-primary hover:text-white transition-all flex items-center justify-center text-xs" title="View">
                    <i class="fas fa-eye"></i>
                  </button>
                  <button @click="editItem(item)" class="w-7 h-7 bg-secondary/10 text-secondary rounded-lg hover:bg-secondary hover:text-white transition-all flex items-center justify-center text-xs" title="Edit">
                    <i class="fas fa-edit"></i>
                  </button>
                  <button @click="openDeleteModal(item.id)" class="w-7 h-7 bg-red-100 text-red-600 rounded-lg hover:bg-red-600 hover:text-white transition-all flex items-center justify-center text-xs" title="Delete">
                    <i class="fas fa-trash"></i>
                  </button>
                </div>
              </td>
            </tr>
            <tr v-if="items.length === 0">
              <td colspan="8" class="text-center py-8">
                <div class="flex flex-col items-center gap-2">
                  <i class="fas fa-inbox text-4xl text-slate-300"></i>
                  <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No sales found</p>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Mobile Cards -->
    <div class="md:hidden space-y-3">
      <div v-for="(item, index) in items" :key="item.id" class="bg-white border-2 border-slate-200 rounded-xl p-4 space-y-3">
        <div class="flex justify-between items-start gap-2">
          <div class="flex-1">
            <div class="flex items-center gap-2 mb-2">
              <span class="w-6 h-6 rounded-lg bg-primary/10 text-primary flex items-center justify-center font-black text-xs">{{ index + 1 }}</span>
              <h3 class="font-black text-slate-800 text-sm">Sale #{{ index + 1 }}</h3>
            </div>
            <p class="text-xs text-slate-500">{{ item.property?.name || 'N/A' }}</p>
          </div>
          <span class="px-2 py-1 rounded-lg text-xs font-black uppercase" :class="item.posting_payment_status === 'approved' ? 'bg-emerald-100 text-emerald-700' : 'bg-amber-100 text-amber-700'">
            {{ item.posting_payment_status }}
          </span>
        </div>
        
        <div class="grid grid-cols-3 gap-2 bg-slate-50 p-3 rounded-lg border border-slate-200">
          <div>
            <p class="text-xs font-bold text-slate-500 uppercase mb-1">Listing</p>
            <p class="text-sm font-black text-slate-800">${{ item.listing_price }}</p>
          </div>
          <div>
            <p class="text-xs font-bold text-slate-500 uppercase mb-1">Selling</p>
            <p class="text-sm font-black text-emerald-600">${{ item.selling_price }}</p>
          </div>
          <div>
            <p class="text-xs font-bold text-slate-500 uppercase mb-1">Fee</p>
            <p class="text-sm font-black text-slate-800">${{ item.posting_fee }}</p>
          </div>
        </div>

        <div class="flex gap-2">
          <button @click="viewDetails(item.id)" class="flex-1 bg-primary text-white py-2 rounded-lg text-xs font-black uppercase">View</button>
          <button @click="editItem(item)" class="flex-1 bg-secondary text-white py-2 rounded-lg text-xs font-black uppercase">Edit</button>
          <button @click="openDeleteModal(item.id)" class="flex-1 bg-red-600 text-white py-2 rounded-lg text-xs font-black uppercase">Delete</button>
        </div>
      </div>

      <div v-if="items.length === 0" class="text-center py-8">
        <i class="fas fa-inbox text-4xl text-slate-300 mb-2"></i>
        <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No sales found</p>
      </div>
    </div>

    <!-- Pagination -->
    <div class="px-6 py-4 border-t-2 border-slate-100 flex justify-between items-center mt-6 bg-white rounded-b-[2rem]">
      <div class="flex items-center gap-2">
        <button @click="fetchItems(currentPage - 1)" :disabled="!previousPage" class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
          <i class="fas fa-chevron-left text-xs"></i>
        </button>
        <div class="px-4 py-2 bg-gradient-to-r from-primary to-dprimary text-white rounded-lg">
          <span class="text-xs font-black uppercase">{{ currentPage }} / {{ totalPages }}</span>
        </div>
        <button @click="fetchItems(currentPage + 1)" :disabled="!nextPage" class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all">
          <i class="fas fa-chevron-right text-xs"></i>
        </button>
      </div>
      <span class="text-xs font-bold text-slate-500 uppercase">Total: <span class="text-primary font-black">{{ count }}</span> sales</span>
    </div>

    <!-- Add/Edit Modal -->
    <AddBrokerListSales v-if="showModal && !editMode" :data="selectedItem" @close="showModal=false" @saved="fetchItems"/>
    <EditBrokerListSales v-if="showModal && editMode" :data="selectedItem" @close="showModal=false" @saved="fetchItems"/>

    <!-- Delete Confirmation Modal -->
    <delete-confirm-modal 
      :visible="deleteModalVisible"
      title="Delete Broker Sale"
      message="Are you sure you want to delete this sale?"
      @confirm="confirmDelete"
      @cancel="deleteModalVisible=false"
    />
  </div>
</template>

<script>
import AddBrokerListSales from "./AddBrokerListSales.vue";
import EditBrokerListSales from "./EditBrokerListSales.vue";
import Loading from "@/components/Loading.vue";
import DeleteConfirmModal from "@/components/DeleteConfirmModal.vue";

export default {
  components: { AddBrokerListSales, EditBrokerListSales, Loading, DeleteConfirmModal },

  data() {
    return {
      items: [],
      count: 0,
      nextPage: null,
      previousPage: null,
      currentPage: 1,
      pageSize: 10,
      totalPages: 1,
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
    async approveSale(id){
      const payload={
         posting_payment_status:"approved"
      }
     const res= await this.$apiPatch("/update_broker_property_sale",id ,payload);
      this.$root.$refs.toast.showToast('Successfully approved', 'success');
    },

    async approveStatusIn(id){
      const payload={
         status__in:"for_sale"
      }
     const res= await this.$apiPatch("/update_broker_property_sale",id ,payload);
      this.$root.$refs.toast.showToast('Successfully approved', 'success');
      this.$reloadPage();
    },

    async fetchItems(page = 1) {
      this.loading = true;
      this.currentPage = page;
      const params = { page: this.currentPage, page_size: this.pageSize, search: this.searchQuery };
      try {
        const response = await this.$apiGet('/get_broker_property_sales', params);
        this.items = response.data;
        this.count = response.count || 0;
        this.nextPage = response.next || null;
        this.previousPage = response.previous || null;
        this.totalPages = response.total_pages || 1;
      } catch(e) { console.error(e); }
      finally { this.loading = false; }
    },

    openAddModal() { this.editMode = false; this.selectedItem = null; this.showModal = true; },
    editItem(item) { this.editMode = true; this.selectedItem = item; this.showModal = true; },
    
    viewDetails(id) { 
      this.$router.push({ name: 'BrokerListSales-detail', params: { id } });
    },

    openDeleteModal(id) { this.deleteId = id; this.deleteModalVisible = true; },

    async confirmDelete() {
      const res = await this.$apiDelete('/brokerlistsales', this.deleteId);
      if(res) {
        this.$root.$refs.toast.showToast('Sale deleted successfully', 'success');
      }
      this.deleteModalVisible = false;
      this.fetchItems(this.currentPage);
    },
  },

  mounted() { this.fetchItems(); }
};
</script>
