<template>
  <div class="p-4 md:p-8 bg-slate-50 min-h-screen font-sans text-slate-900">
    
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
      <div>
        <div class="flex items-center gap-3 mb-1">
          <div class="w-2 h-8 bg-indigo-600 rounded-full"></div>
          <h1 class="text-2xl font-black uppercase tracking-tight text-slate-800">Management Analytics</h1>
        </div>
        <p class="text-[10px] font-black text-slate-400 uppercase tracking-[0.2em] ml-5">Operational Overview & Reporting</p>
      </div>
      
      <div class="flex items-center gap-2 bg-white px-4 py-2 rounded-2xl shadow-sm border border-slate-100">
        <span class="text-[10px] font-black text-slate-500 uppercase tracking-widest">System Live</span>
        <span class="relative flex h-2 w-2">
          <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-emerald-400 opacity-75"></span>
          <span class="relative inline-flex rounded-full h-2 w-2 bg-emerald-500"></span>
        </span>
      </div>
    </div>

    <div class="space-y-8">
      
      <section class="bg-white rounded-[2.5rem] shadow-xl shadow-slate-200/50 border border-slate-100 overflow-hidden transition-all">
        <div class="p-8">
          <div class="flex flex-col lg:flex-row gap-10">
            
            <div class="flex-1 space-y-6">
              <div class="flex items-center justify-between">
                <h2 class="text-[11px] font-black text-indigo-600 uppercase tracking-[0.2em]">User Demographics</h2>
                <div class="flex bg-slate-100 p-1 rounded-xl">
                  <button
                    v-for="tab in userTabs"
                    :key="tab.key"
                    @click="activeUserTab = tab.key; fetchUsers()"
                    :class="[activeUserTab === tab.key ? 'bg-white text-indigo-600 shadow-sm' : 'text-slate-400 hover:text-slate-600']"
                    class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase tracking-widest transition-all"
                  >
                    {{ tab.label }}
                  </button>
                </div>
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 bg-slate-50 p-6 rounded-3xl border border-slate-100">
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-slate-400 uppercase ml-1">Date From</label>
                  <input type="date" v-model="userFilter.from" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 focus:ring-2 focus:ring-indigo-500/20 shadow-sm transition-all" />
                </div>
                <div class="space-y-1">
                  <label class="text-[9px] font-black text-slate-400 uppercase ml-1">Date To</label>
                  <input type="date" v-model="userFilter.to" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 focus:ring-2 focus:ring-indigo-500/20 shadow-sm transition-all" />
                </div>
                <div class="flex items-end">
                  <button @click="applyUserFilter" class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-[10px] font-black uppercase tracking-widest transition-all active:scale-95">
                    Apply Filter
                  </button>
                </div>
              </div>

              <div class="flex items-center justify-between pt-4 border-t border-slate-50">
                <div>
                  <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-1">Total {{ activeUserTab }}</p>
                  <p class="text-4xl font-black text-slate-800 tracking-tighter">{{ filteredUsers.length }}</p>
                </div>
                <button @click="downloadUsers" class="flex items-center gap-2 px-6 py-3 bg-slate-800 hover:bg-black text-white rounded-2xl text-[10px] font-black uppercase tracking-widest transition-all shadow-lg">
                  <i class="fas fa-file-export text-xs"></i> Export List
                </button>
              </div>
            </div>

            <div class="flex-1 min-h-[300px] bg-slate-50/50 rounded-3xl p-6 flex items-center justify-center border border-slate-100">
              <apexchart type="donut" width="100%" height="300" :options="userChartOptions" :series="userSeries" />
            </div>
          </div>
        </div>
      </section>

      <section class="bg-white rounded-[2.5rem] shadow-xl shadow-slate-200/50 border border-slate-100 overflow-hidden transition-all">
        <div class="p-8">
          <div class="flex flex-col lg:flex-row-reverse gap-10">
            
            <div class="flex-1 space-y-6">
              <div class="flex flex-wrap items-center justify-between gap-4">
                <h2 class="text-[11px] font-black text-emerald-600 uppercase tracking-[0.2em]">Financial Streams</h2>
                <div class="flex flex-wrap bg-slate-100 p-1 rounded-xl">
                  <button
                    v-for="tab in paymentTabs"
                    :key="tab.key"
                    @click="activePaymentTab = tab.key"
                    :class="[activePaymentTab === tab.key ? 'bg-white text-emerald-600 shadow-sm' : 'text-slate-400 hover:text-slate-600']"
                    class="px-4 py-1.5 rounded-lg text-[10px] font-black uppercase tracking-widest transition-all"
                  >
                    {{ tab.label }}
                  </button>
                </div>
              </div>

              <div class="bg-slate-50 p-6 rounded-3xl border border-slate-100">
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 items-end">
                  <div class="space-y-1">
                    <label class="text-[9px] font-black text-slate-400 uppercase ml-1">From</label>
                    <input type="date" v-show="activePaymentTab === 'subscription'" v-model="subscriptionPaymentFilter.from" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'sales'" v-model="salesPaymentFilter.from" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'rent'" v-model="rentPaymentFilter.from" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'rental'" v-model="rentalPaymentFilter.from" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                  </div>
                  <div class="space-y-1">
                    <label class="text-[9px] font-black text-slate-400 uppercase ml-1">To</label>
                    <input type="date" v-show="activePaymentTab === 'subscription'" v-model="subscriptionPaymentFilter.to" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'sales'" v-model="salesPaymentFilter.to" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'rent'" v-model="rentPaymentFilter.to" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                    <input type="date" v-show="activePaymentTab === 'rental'" v-model="rentalPaymentFilter.to" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm" />
                  </div>

                  <div v-if="activePaymentTab === 'rental'" class="space-y-1">
                    <label class="text-[9px] font-black text-slate-400 uppercase ml-1">Logic Type</label>
                    <select v-model="cowrk_type" class="w-full bg-white border-none rounded-xl text-xs font-bold p-3 shadow-sm appearance-none outline-none">
                      <option value="rental__start_date">Agreement Start</option>
                      <option value="paid_at">Paid at</option>
                      <option value="cycle_start">Cycle Start</option>
                    </select>
                  </div>

                  <div :class="activePaymentTab === 'rental' ? '' : 'sm:col-span-2 lg:col-span-2'">
                    <button 
                      @click="activePaymentTab === 'subscription' ? applySubscriptionFilter() : 
                              activePaymentTab === 'sales' ? applySalesFilter() : 
                              activePaymentTab === 'rent' ? applyRentFilter() : applyRentalFilter()" 
                      class="w-full py-3 text-white rounded-xl text-[10px] font-black uppercase tracking-widest shadow-md transition-all active:scale-95"
                      :class="[activePaymentTab === 'subscription' ? 'bg-teal-600' : activePaymentTab === 'sales' ? 'bg-orange-600' : activePaymentTab === 'rent' ? 'bg-indigo-600' : 'bg-purple-600']"
                    >
                      Update View
                    </button>
                  </div>
                </div>
              </div>

              <div class="flex items-center justify-between pt-4 border-t border-slate-50">
                <div>
                  <p class="text-[10px] font-black text-slate-400 uppercase tracking-widest mb-1 capitalize">{{ activePaymentTab }} Payments</p>
                  <p class="text-4xl font-black text-slate-800 tracking-tighter">
                    {{ activePaymentTab === 'subscription' ? filteredSubscriptionPayments.length : 
                       activePaymentTab === 'sales' ? filteredSalesPayments.length : 
                       activePaymentTab === 'rent' ? filteredRentPayments.length : filteredRentalPayments.length }}
                  </p>
                </div>
                <button 
                  @click="activePaymentTab === 'subscription' ? downloadSubscriptionPayments() : 
                          activePaymentTab === 'sales' ? downloadSalesPayments() : 
                          activePaymentTab === 'rent' ? downloadRentPayments() : downloadRentalPayments()"
                  class="flex items-center gap-2 px-6 py-3 rounded-2xl text-[10px] font-black uppercase tracking-widest transition-all shadow-lg text-white"
                  :class="[activePaymentTab === 'subscription' ? 'bg-teal-600 shadow-teal-200' : activePaymentTab === 'sales' ? 'bg-orange-600 shadow-orange-200' : 'bg-indigo-600 shadow-indigo-200']"
                >
                  <i class="fas fa-download text-xs"></i> Export CSV
                </button>
              </div>
            </div>

            <div class="flex-1 min-h-[300px] bg-slate-50/50 rounded-3xl p-6 flex items-center justify-center border border-slate-100">
              <apexchart type="donut" width="100%" height="300" :options="paymentChartOptions" :series="paymentSeries" />
            </div>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";
import VueApexCharts from "vue3-apexcharts";

export default {
  components: { apexchart: VueApexCharts },
  data() {
    return {
       totalUsers: 0,
      cowrk_type: "rental__start_date",
      activeUserTab: "owners",
      activePaymentTab: "subscription",
      userTabs: [
        { key: "owners", label: "Owners" },
        { key: "tenants", label: "Tenants" },
        { key: "staffs", label: "Staffs" },
        { key: "managers", label: "Managers" },
      ],
      paymentTabs: [
        { key: "subscription", label: "Subscription" },
        { key: "sales", label: "Sales" },
        { key: "rent", label: "Rent" },
        { key: "rental", label: "Co-work-space" },
      ],

      users: [],
      subscriptionPayments: [],
      salesPayments: [],
      rentPayments: [],
      rentalPayments: [],

      filteredUsers: [],
      filteredSubscriptionPayments: [],
      filteredSalesPayments: [],
      filteredRentPayments: [],
      filteredRentalPayments: [],

      userFilter: { from: "", to: "" },
      subscriptionPaymentFilter: { from: "", to: "" },
      salesPaymentFilter: { from: "", to: "" },
      rentPaymentFilter: { from: "", to: "" },
      rentalPaymentFilter: { from: "", to: "" },

      userSeries: [1, 1, 1, 1],
      userChartOptions: { labels: ["Owners", "Tenants", "Staffs", "Managers"], legend: { position: "bottom" } },
      paymentSeries: [10, 20, 15, 5],
      paymentChartOptions: { labels: ["Subscription", "Sales", "Rent", "Workspace"], legend: { position: "bottom" } },
    };
  },
  mounted() {
    this.fetchUsers();
    this.fetchAllPayments();
  },
  methods: {
    /** USERS */
 async fetchUsers() {
    try {
      let endpoint = "";
      switch (this.activeUserTab) {
        case "owners":
          endpoint = "/get_owners";
          break;
        case "managers":
          endpoint = "/get_managers";
          break;
        case "staffs":
          endpoint = "/get_staffs";
          break;
        case "tenants":
          endpoint = "/get_tenants";
          break;
      }

      const res = await this.$apiGet(endpoint);

      // Handle owners separately because API sends plain array
      if (this.activeUserTab === "owners") {
        this.users = res?.owners || [];
        this.totalUsers = this.users.length; // count manually
      } else {
        this.users = res?.data || [];
        this.totalUsers = res?.count ?? this.users.length; // fallback
      }

      this.filteredUsers = [...this.users];
    } catch (err) {
      console.error(err);
      this.users = [];
      this.filteredUsers = [];
      this.totalUsers = 0;
    }
  },


    async applyUserFilter() {
      if (!this.userFilter.from || !this.userFilter.to) {
        this.filteredUsers = [...this.users];
        return;
      }
      const from = new Date(this.userFilter.from);
      const to = new Date(this.userFilter.to);
      this.filteredUsers = this.users.filter(u => {
        const date = new Date(u.created_at);
        return date >= from && date <= to;
      });
    },

    /** PAYMENTS */
    async fetchAllPayments() {
      await this.fetchSubscriptionPayments();
      await this.fetchSalesPayments();
      await this.fetchRentPayments();
      await this.fetchRentalPayments();
    },
    async fetchSubscriptionPayments() {
      const res = await this.$apiGet("/get_subscription_payment");
      this.subscriptionPayments = res?.data || [];
      this.filteredSubscriptionPayments = [...this.subscriptionPayments];
    },
    async fetchSalesPayments() {
      const res = await this.$apiGet("/get_sales_payments");
      this.salesPayments = res?.data || [];
      this.filteredSalesPayments = [...this.salesPayments];
    },
    async fetchRentPayments() {
      const res = await this.$apiGet("/get_payments");
      this.rentPayments = res?.data || [];
      this.filteredRentPayments = [...this.rentPayments];
    },
    async fetchRentalPayments() {
      const res = await this.$apiGet("/get_rental_payments");
      this.rentalPayments = res?.data || [];
      this.filteredRentalPayments = [...this.rentalPayments];
    },

    async applySubscriptionFilter() {
      try {
        if (!this.subscriptionPaymentFilter.from || !this.subscriptionPaymentFilter.to) {
          this.filteredSubscriptionPayments = [...this.subscriptionPayments];
          return;
        }
        const params = {
          paid_at__gte: this.subscriptionPaymentFilter.from,
          paid_at__lte: this.subscriptionPaymentFilter.to,
        };
        const res = await this.$apiGet("/get_subscription_payment", params);
        
        this.subscriptionPayments = res?.data || [];
        this.filteredSubscriptionPayments = [...this.subscriptionPayments];
      } catch (error) {
        console.error("Error fetching filtered subscription payments:", error);
      }
    },

    async applySalesFilter() {
      try {
        if (!this.salesPaymentFilter.from || !this.salesPaymentFilter.to) {
          this.filteredSalesPayments = [...this.salesPayments];
          return;
        }
        const params = {
          due_date__gte: this.salesPaymentFilter.from,
          due_date__lte: this.salesPaymentFilter.to,
        };
        const res = await this.$apiGet("/get_sales_payments", params);
        this.salesPayments = res?.data || [];
        this.filteredSalesPayments = [...this.salesPayments];
      } catch (error) {
        console.error("Error fetching filtered sales payments:", error);
      }
    },

    async applyRentFilter() {
      try {
        if (!this.rentPaymentFilter.from || !this.rentPaymentFilter.to) {
          this.filteredRentPayments = [...this.rentPayments];
          return;
        }
        const params = {
          due_date__gte: this.rentPaymentFilter.from,
          due_date__lte: this.rentPaymentFilter.to,
        };
        const res = await this.$apiGet("/get_payments", params);
        this.rentPayments = res?.data || [];
        this.filteredRentPayments = [...this.rentPayments];
      } catch (error) {
        console.error("Error fetching filtered rent payments:", error);
      }
    },

    async applyRentalFilter() {
      try {
        if (!this.rentalPaymentFilter.from || !this.rentalPaymentFilter.to) {
          this.filteredRentalPayments = [...this.rentalPayments];
          return;
        }
        const params = {
          [`${this.cowrk_type}__gte`]: this.rentalPaymentFilter.from,
          [`${this.cowrk_type}__lte`]: this.rentalPaymentFilter.to,
        };
        const res = await this.$apiGet("/get_rental_payments", params);
        this.rentalPayments = res?.data || [];
        this.filteredRentalPayments = [...this.rentalPayments];
      } catch (error) {
        console.error("Error fetching filtered rental payments:", error);
      }
    },

    /** EXPORT */
    exportExcel(data, fileName, sheetName) {
      const ws = XLSX.utils.json_to_sheet(data);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, sheetName);
      XLSX.writeFile(wb, fileName);
    },
    downloadUsers() {
      this.exportExcel(this.filteredUsers, `${this.activeUserTab}.xlsx`, "Users");
    },
    downloadSubscriptionPayments() {
      this.exportExcel(this.filteredSubscriptionPayments, "SubscriptionPayments.xlsx", "Subscription");
    },
    downloadSalesPayments() {
      this.exportExcel(this.filteredSalesPayments, "SalesPayments.xlsx", "Sales");
    },
    downloadRentPayments() {
      this.exportExcel(this.filteredRentPayments, "RentPayments.xlsx", "Rent");
    },
    downloadRentalPayments() {
      this.exportExcel(this.filteredRentalPayments, "RentalPayments.xlsx", "Rental");
    },
  },
};
</script>
