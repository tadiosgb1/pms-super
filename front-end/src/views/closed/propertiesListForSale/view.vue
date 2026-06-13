<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-50 via-white to-tertiary/10">
    <Toast ref="toast" />

    <!-- Header Section -->
    <div class="px-6 pt-6 pb-4 flex justify-between items-center">
      <button
        @click="$router.back()"
        class="group flex items-center gap-2 px-4 py-2 bg-white border-2 border-slate-200 rounded-xl text-xs font-black uppercase tracking-widest text-slate-600 hover:text-primary hover:border-primary transition-all"
      >
        <i class="fas fa-arrow-left group-hover:-translate-x-1 transition-transform"></i>
        Back
      </button>
      
      <div class="flex items-center gap-2 bg-white px-3 py-1.5 rounded-lg border-2 border-slate-200">
        <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
        <span class="text-xs font-black text-slate-500 uppercase tracking-widest">Live Market Data</span>
      </div>
    </div>

    <!-- Main Content -->
    <div class="px-6 pb-6">
      <div class="bg-white rounded-[2rem] border-2 border-slate-200 overflow-hidden">
        
        <!-- Header Banner -->
        <div class="bg-gradient-to-r from-primary to-dprimary px-8 py-6">
          <div class="flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
              <h1 class="text-2xl font-black text-white tracking-tight">Property Sale Listings</h1>
              <p class="text-white/70 text-xs font-bold uppercase tracking-widest mt-1">Global Asset Inventory</p>
            </div>
            
            <div class="relative w-full md:max-w-xs">
              <i class="fas fa-search absolute left-4 top-1/2 -translate-y-1/2 text-white/60 text-sm"></i>
              <input
                v-model="searchTerm"
                @input="onSearch"
                type="search"
                placeholder="Filter by zone or price..."
                class="w-full bg-white/10 border-2 border-white/20 rounded-xl py-2.5 pl-11 pr-4 text-sm text-white placeholder:text-white/50 focus:outline-none focus:bg-white focus:text-slate-900 focus:border-white transition-all"
              />
            </div>
          </div>
        </div>

        <!-- Controls Section -->
        <div class="px-8 py-4 border-b-2 border-slate-100 flex flex-col sm:flex-row justify-between items-center gap-4">
          <div class="flex items-center gap-3 bg-slate-50 px-4 py-2.5 rounded-xl border-2 border-slate-200">
            <label class="text-xs font-black text-slate-600 uppercase tracking-wider">Show</label>
            <select
              v-model="pageSize"
              @change="fetchListings()"
              class="bg-transparent font-black text-primary text-xs outline-none cursor-pointer"
            >
              <option v-for="size in pageSizes" :key="size" :value="size">{{ size }}</option>
            </select>
            <span class="text-xs font-black text-slate-600 uppercase tracking-wider">entries</span>
          </div>

          <button @click="fetchListings()" class="p-2 text-slate-400 hover:text-primary transition-colors">
            <i class="fas fa-sync-alt text-sm"></i>
          </button>
        </div>

        <!-- Desktop Table -->
        <div class="hidden md:block overflow-x-auto">
          <table class="w-full">
            <thead>
              <tr class="bg-gradient-to-r from-primary/10 via-tertiary/20 to-secondary/10 border-b-2 border-slate-200">
                <th class="px-6 py-3 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-map-marker-alt text-primary mr-2"></i>Property / Zone
                </th>
                <th class="px-6 py-3 text-left text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-tag text-primary mr-2"></i>Market Valuation
                </th>
                <th class="px-6 py-3 text-center text-xs font-black text-slate-700 uppercase tracking-widest">
                  <i class="fas fa-handshake text-primary mr-2"></i>Actions
                </th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-100">
              <tr
                v-for="listing in listings"
                :key="listing.id"
                class="hover:bg-slate-50 transition-colors duration-150"
              >
                <td class="px-6 py-3">
                  <div class="flex items-center gap-3">
                    <div class="w-8 h-8 rounded-lg bg-primary/10 flex items-center justify-center text-primary text-xs">
                      <i class="fas fa-map-marker-alt"></i>
                    </div>
                    <button
                      @click="goToZoneDetail(listing.property_zone_id?.id || listing.property_id?.id)"
                      class="text-sm font-black text-slate-800 hover:text-primary transition-colors text-left"
                    >
                      {{ listing.property_zone_id?.name || listing.property_id?.name || 'N/A' }}
                    </button>
                  </div>
                </td>
                <td class="px-6 py-3">
                  <div class="flex flex-col gap-1">
                    <span class="text-sm font-black text-emerald-600">${{ listing.listing_price?.toLocaleString() }}</span>
                    <span class="text-xs font-bold text-slate-500 uppercase">Asking Price</span>
                  </div>
                </td>
                <td class="px-6 py-3 text-center">
                  <button
                    @click="sale(listing.id)"
                    class="px-4 py-2 bg-primary text-white text-xs font-black uppercase tracking-widest rounded-lg hover:bg-dprimary transition-all"
                  >
                    Process Sale
                  </button>
                </td>
              </tr>
              <tr v-if="listings.length === 0">
                <td colspan="3" class="text-center py-12">
                  <div class="flex flex-col items-center gap-2">
                    <i class="fas fa-inbox text-4xl text-slate-300"></i>
                    <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No listings available</p>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Mobile Cards -->
        <div class="md:hidden px-6 py-4 space-y-3">
          <div
            v-for="listing in listings"
            :key="listing.id"
            class="bg-slate-50 border-2 border-slate-200 rounded-xl p-4 space-y-3"
          >
            <div class="flex items-start justify-between gap-2">
              <div class="flex items-center gap-3 flex-1">
                <div class="w-8 h-8 rounded-lg bg-primary/10 flex items-center justify-center text-primary text-xs flex-shrink-0">
                  <i class="fas fa-home"></i>
                </div>
                <h3
                  class="font-black text-slate-800 text-sm hover:text-primary transition-colors cursor-pointer"
                  @click="goToZoneDetail(listing.property_zone_id?.id || listing.property_id?.id)"
                >
                  {{ listing.property_zone_id?.name || listing.property_id?.name || 'N/A' }}
                </h3>
              </div>
            </div>
            
            <div class="bg-white rounded-lg p-3 border border-slate-200 flex justify-between items-center">
              <span class="text-xs font-black text-slate-600 uppercase">Market Value</span>
              <span class="text-sm font-black text-emerald-600">${{ listing.listing_price?.toLocaleString() }}</span>
            </div>

            <div class="flex gap-2">
              <button
                @click="sale(listing.id)"
                class="flex-1 px-4 py-2.5 bg-primary text-white text-xs font-black uppercase rounded-lg transition-all"
              >
                Process Sale
              </button>
              <button
                @click="viewDetail(listing.id)"
                class="flex-1 px-4 py-2.5 bg-slate-200 text-slate-700 text-xs font-black uppercase rounded-lg transition-all"
              >
                Details
              </button>
            </div>
          </div>

          <div v-if="listings.length === 0" class="text-center py-12">
            <i class="fas fa-inbox text-4xl text-slate-300 mb-2"></i>
            <p class="text-slate-400 font-bold uppercase tracking-wider text-sm">No listings available</p>
          </div>
        </div>

        <!-- Pagination -->
        <div class="px-8 py-4 border-t-2 border-slate-100 flex justify-between items-center">
          <div class="flex items-center gap-2">
            <button
              :disabled="!previous"
              @click="fetchListings(previous)"
              class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all"
            >
              <i class="fas fa-chevron-left text-xs"></i>
            </button>
            
            <div class="px-4 py-2 bg-gradient-to-r from-primary to-dprimary text-white rounded-lg">
              <span class="text-xs font-black uppercase">{{ currentPage }} / {{ totalPages }}</span>
            </div>

            <button
              :disabled="!next"
              @click="fetchListings(next)"
              class="w-9 h-9 flex items-center justify-center rounded-lg border-2 border-slate-200 bg-white text-slate-600 hover:border-primary hover:text-primary hover:bg-primary/5 disabled:opacity-30 disabled:cursor-not-allowed transition-all"
            >
              <i class="fas fa-chevron-right text-xs"></i>
            </button>
          </div>
          
          <span class="text-xs font-bold text-slate-500 uppercase">Total: <span class="text-primary font-black">{{ listings.length }}</span> listings</span>
        </div>
      </div>

      <!-- Modals -->
      <AddPropertySaleListing
        v-if="visible"
        :visible="visible"
        @close="visible = false"
        @refresh="fetchListings"
      />

      <MakePropertySale
        v-if="saleVisible"
        :visible="saleVisible"
        :listing="selectedListing"
        @close="saleVisible = false"
        @success="fetchListings"
      />
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import AddPropertySaleListing from "@/views/closed/propertiesListForSale/add.vue";
import MakePropertySale from "@/views/closed/propertiesListForSale/MakePropertySale.vue";

export default {
  name: "PropertySaleListingView",
  components: { Toast, AddPropertySaleListing,MakePropertySale },
  data() {
    return {
      listings: [],
      visible: false,
      searchTerm: "",
      saleVisible: false, // ✅ add this
      selectedListing: null, // ✅ add this
      next: null,
      previous: null,
      pageSize: 10,
      pageSizes: [5, 10, 20, 50, 100],
      currentPage: 1,
      totalPages: 1,
    };
  },
  mounted() {
   // alert("hii")
   
    this.fetchListings();
  },
  methods: {
    async fetchListings(url = `/get_property_zone_sales?search=${this.searchTerm}&page_size=${this.pageSize}`) {
      try {
        const response = await this.$apiGet(url);
        console.log("response listings", response);
        if (response && response.data) {
          this.listings = response.data || [];
          this.next = response.next;
          this.previous = response.previous;
          this.currentPage = response.current_page;
          this.totalPages = response.total_pages;
        }
      } catch (error) {
        console.error("Failed to fetch property sale listings:", error);
        this.listings = [];
      }
    },
    onSearch() {
      this.currentPage = 1;
      this.fetchListings();
    },
    goToZoneDetail(zoneId) {
      if (zoneId)
        this.$router.push({
          name: "PropertyZoneDetail",
          params: { id: zoneId },
        });
    },


    viewDetail(id) {
      this.$router.push({ name: "property-sale-detail", params: { id } });
    },
    

     sale(listingId) {
      const listing = this.listings.find((item) => item.id === listingId);
      if (listing) {
        this.selectedListing = listing;
        this.saleVisible = true;
      }
    },
    formatCurrency(value) {
      if (!value) return "N/A";
      return new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
      }).format(value);
    },
  },
};
</script>

<style scoped>
@media (max-width: 768px) {
  .card {
    transition: box-shadow 0.2s ease-in-out;
  }
  .card:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
}
</style>
