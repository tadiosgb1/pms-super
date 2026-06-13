<template>
  <div class="min-h-screen bg-background p-4 md:p-2">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Configuring Plan Tiers..." />

    <div class="max-w-7xl mx-auto bg-white  rounded-xl overflow-hidden border border-gray-100">
      
      <div class="bg-gradient-to-r from-dprimary to-primary px-6 py-6 flex flex-col sm:flex-row justify-between items-center gap-4 shadow-lg">
        <div class="flex items-center gap-4">
          <div class="bg-white/20 p-2.5 rounded-xl backdrop-blur-md text-white">
            <i class="fas fa-layer-group text-2xl"></i>
          </div>
          <div>
            <h1 class="text-2xl font-bold text-white tracking-tight">Subscription Plans</h1>
            <p class="text-white/70 text-[10px] uppercase tracking-widest font-bold">Pricing & Limitation Logic</p>
          </div>
        </div>

        <button
          @click="visible = true"
          class="bg-white text-primary hover:bg-gray-50 font-bold px-6 py-2.5 rounded-xl shadow-lg transition-all duration-300 transform hover:scale-105 active:scale-95 flex items-center gap-2"
        >
          <i class="fas fa-plus-circle"></i> Create New Plan
        </button>
      </div>

      <div class="p-6">
        <div class="relative w-full max-w-md mb-8 group">
          <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-gray-400 group-focus-within:text-primary transition-colors">
            <i class="fas fa-search text-xs"></i>
          </span>
          <input
            v-model="searchTerm"
            type="search"
            placeholder="Search plans by name..."
            class="custom-input !mt-0 !pl-10 h-11 shadow-sm border-gray-200 focus:border-primary"
          />
        </div>

        <div class="overflow-hidden rounded-xl border border-gray-100 shadow-sm hidden md:block">
          <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
              <tr class="text-left text-[11px] font-black text-gray-400 uppercase tracking-widest">
                <th class="px-6 py-4 cursor-pointer hover:text-primary transition-colors" @click="sortBy('name')">
                  Tier Name <SortIcon :field="'name'" :sort-key="sortKey" :sort-asc="sortAsc" />
                </th>
                <th class="px-6 py-4 text-center">Resource Limits</th>
                <th class="px-6 py-4 cursor-pointer hover:text-primary transition-colors" @click="sortBy('price')">
                  Price <SortIcon :field="'price'" :sort-key="sortKey" :sort-asc="sortAsc" />
                </th>
                <th class="px-6 py-4 cursor-pointer hover:text-primary transition-colors" @click="sortBy('billing_cycle')">
                  Billing Cycle <SortIcon :field="'billing_cycle'" :sort-key="sortKey" :sort-asc="sortAsc" />
                </th>
                <th class="px-6 py-4 text-right">Actions</th>
              </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-50">
              <tr v-for="plan in filteredAndSortedPlans" :key="plan.id" class="hover:bg-primary/5 transition-all">
                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="text-sm font-black text-gray-700 uppercase tracking-tight">{{ plan.name }}</span>
                </td>
                
                <td class="px-6 py-4">
                  <div class="flex items-center justify-center gap-3">
                    <div class="flex flex-col items-center px-3 py-1 bg-blue-50 rounded-lg border border-blue-100" title="Max Locations">
                      <span class="text-[10px] font-bold text-blue-400 uppercase">LOC</span>
                      <span class="text-xs font-black text-blue-700">{{ plan.max_locations }}</span>
                    </div>
                    <div class="flex flex-col items-center px-3 py-1 bg-purple-50 rounded-lg border border-purple-100" title="Max Staff">
                      <span class="text-[10px] font-bold text-purple-400 uppercase">STF</span>
                      <span class="text-xs font-black text-purple-700">{{ plan.max_staff }}</span>
                    </div>
                    <div class="flex flex-col items-center px-3 py-1 bg-amber-50 rounded-lg border border-amber-100" title="Max Users">
                      <span class="text-[10px] font-bold text-amber-400 uppercase">USR</span>
                      <span class="text-xs font-black text-amber-700">{{ plan.max_users }}</span>
                    </div>
                  </div>
                </td>

                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="text-sm font-bold text-primary">{{ plan.price }}</span>
                </td>
                
                <td class="px-6 py-4 whitespace-nowrap">
                  <span class="px-3 py-1 bg-gray-100 rounded-full text-[10px] font-black uppercase text-gray-500 tracking-wider">
                    {{ plan.billing_cycle }}
                  </span>
                </td>

                <td class="px-6 py-4 text-right">
                  <div class="flex justify-end gap-2">
                    <button @click="editPlan(plan)" class="h-8 w-8 flex items-center justify-center rounded-lg bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white transition-all shadow-sm">
                      <i class="fas fa-edit text-xs"></i>
                    </button>
                    <button @click="askDeleteConfirmation(plan)" class="h-8 w-8 flex items-center justify-center rounded-lg bg-red-50 text-red-500 hover:bg-red-500 hover:text-white transition-all shadow-sm">
                      <i class="fas fa-trash-alt text-xs"></i>
                    </button>
                  </div>
                </td>
              </tr>
              <tr v-if="filteredAndSortedPlans.length === 0">
                <td colspan="5" class="text-center py-12 text-gray-400 italic">No plan tiers found matching your search.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="md:hidden grid gap-4">
          <div
            v-for="plan in filteredAndSortedPlans"
            :key="plan.id"
            class="bg-white border border-gray-100 rounded-2xl p-5 shadow-md relative overflow-hidden"
          >
            <div class="absolute top-0 right-0 p-3 flex gap-2">
               <button @click="editPlan(plan)" class="text-blue-500 p-2"><i class="fas fa-edit"></i></button>
               <button @click="askDeleteConfirmation(plan)" class="text-red-400 p-2"><i class="fas fa-trash-alt"></i></button>
            </div>

            <h3 class="text-lg font-black text-gray-800 uppercase tracking-tight mb-4 border-b pb-2 border-gray-50">{{ plan.name }}</h3>
            
            <div class="grid grid-cols-3 gap-2 mb-4">
              <div class="text-center p-2 bg-gray-50 rounded-xl">
                <p class="text-[9px] font-bold text-gray-400 uppercase">Locs</p>
                <p class="font-bold text-gray-700">{{ plan.max_locations }}</p>
              </div>
              <div class="text-center p-2 bg-gray-50 rounded-xl">
                <p class="text-[9px] font-bold text-gray-400 uppercase">Staff</p>
                <p class="font-bold text-gray-700">{{ plan.max_staff }}</p>
              </div>
              <div class="text-center p-2 bg-gray-50 rounded-xl">
                <p class="text-[9px] font-bold text-gray-400 uppercase">Users</p>
                <p class="font-bold text-gray-700">{{ plan.max_users }}</p>
              </div>
            </div>

            <div class="flex justify-between items-center">
              <span class="text-primary font-black text-lg">{{ plan.price }}</span>
              <span class="text-[10px] font-bold text-gray-400 uppercase tracking-widest bg-gray-100 px-2 py-1 rounded-md">{{ plan.billing_cycle }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <AddPlan v-if="visible" :visible="visible" @close="visible = false" />
    
    <ConfirmModal
      v-if="confirmVisible"
      :visible="confirmVisible"
      title="Delete Plan Tier"
      message="Warning: Deleting this plan will affect all future subscriptions for this tier. Are you sure?"
      @confirm="confirmDelete"
      @cancel="confirmVisible = false"
    />

    <UpdatePlan
      v-if="updateVisible"
      :visible="updateVisible"
      :plan="planToEdit"
      @close="updateVisible = false"
      @refresh="fetchPlans"
    />
  </div>
</template>


<script>
import AddPlan from '@/views/closed/plans/add.vue';
import ConfirmModal from '@/components/ConfirmModal.vue'; // adjust path as needed
import UpdatePlan from '@/views/closed/plans/update.vue';
import Toast from "../../../components/Toast.vue";
import Loading from "@/components/Loading.vue";

const SortIcon = {
  props: ['field', 'sortKey', 'sortAsc'],
  template: `
    <span class="inline-block ml-1 text-gray-500">
      <svg v-if="sortKey !== field" xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/>
      </svg>
      <svg v-else-if="sortAsc" xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 13l4 4 4-4m0-6l-4-4-4 4"/>
      </svg>
      <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l4-4 4 4m0 6l-4 4-4-4"/>
      </svg>
    </span>
  `
};

export default {
  name: 'plansView',
  components: { SortIcon, AddPlan, ConfirmModal, UpdatePlan ,Toast ,Loading},
  data() {
    return {
      searchTerm: '',
      visible: false,

      confirmVisible: false,
      planToDelete: null,

      updateVisible: false,
      planToEdit: null,

      sortKey: 'name',
      sortAsc: true,
      plans: [],
      loading:false,
    };
  },
  computed: {
    filteredAndSortedPlans() {
      const term = this.searchTerm.toLowerCase();
      let filtered = this.plans.filter(s =>
        s.name.toLowerCase().includes(term) ||
        String(s.max_locations).toLowerCase().includes(term) ||
        String(s.max_staff).toLowerCase().includes(term) ||
        String(s.max_kds).toLowerCase().includes(term) ||
        s.kds_enabled.toLowerCase().includes(term) ||
        String(s.price).toLowerCase().includes(term) ||
        s.billing_cycle.toLowerCase().includes(term)
      );

      filtered.sort((a, b) => {
        let res = 0;
        if (a[this.sortKey] < b[this.sortKey]) res = -1;
        if (a[this.sortKey] > b[this.sortKey]) res = 1;
        return this.sortAsc ? res : -res;
      });

      return filtered;
    }
  },
  mounted() {
   const is_super_user=localStorage.getItem('is_superuser');
    //console.log("haspermission=",this.$hasPermission("pms.view_plan"));
    if(is_super_user=='true'){
       this.fetchPlans();
    }else{
      this.$router.push({
        name:'accessDenied'
      });
    }
  },
  methods: {
    async fetchPlans() {
      this.loading=true;
      try {
        const response = await this.$apiGet("/get_plans");
        if (Array.isArray(response.data)) {
          this.plans = response.data;
        } else {
          console.warn("Unexpected response format:", response);
          this.plans = [];
        }
      } catch (error) {
        console.error("Failed to fetch plans:", error);
        this.plans = [];
        alert("Failed to load plans. Please try again later.");
      }finally{
        this.loading=false;
      }
    },
    sortBy(key) {
      if (this.sortKey === key) {
        this.sortAsc = !this.sortAsc;
      } else {
        this.sortKey = key;
        this.sortAsc = true;
      }
    },
    editPlan(plan) {
  this.planToEdit = plan;
  this.updateVisible = true;
}
,
    askDeleteConfirmation(plan) {
  this.planToDelete = plan;
  this.confirmVisible = true;
},

async confirmDelete() {
  this.confirmVisible = false;
  try {
    const response = await this.$apiDelete(`/delete_plan/${this.planToDelete.id}`);
    this.$root.$refs.toast.showToast(response.message, 'success');
    this.fetchPlans();
  } catch (error) {
    alert("Failed to delete plan.");
    console.error(error);
  }
  this.planToDelete = null;
},

  }
};
</script>
