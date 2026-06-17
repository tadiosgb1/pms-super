<template>
  <div class="min-h-screen bg-slate-50 pb-10">
    <Toast ref="toast" />
    <Loading :visible="loading" message="Loading broker profile..." />

    <!-- Back -->
    <div class="px-4 md:px-8 pt-5 pb-2">
      <button @click="$router.back()"
        class="flex items-center gap-2 text-xs font-bold text-slate-500 hover:text-primary transition uppercase tracking-wider">
        <i class="fas fa-arrow-left text-[10px]"></i> Back to Brokers
      </button>
    </div>

    <div v-if="broker" class="px-4 md:px-8 space-y-5 max-w-7xl mx-auto">

      <!-- ── Hero Profile Card ───────────────────────────────────────── -->
      <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
        <div class="bg-gradient-to-r from-primary to-dprimary px-6 py-6 flex flex-col sm:flex-row sm:items-center gap-5">
          <!-- Avatar -->
          <div class="w-16 h-16 rounded-2xl bg-white/20 border-2 border-white/30 flex items-center justify-center text-white font-black text-2xl uppercase shrink-0">
            {{ initials(broker.first_name, broker.last_name) }}
          </div>

          <div class="flex-1 min-w-0">
            <h1 class="text-xl font-black text-white tracking-tight truncate">
              {{ fullName }}
            </h1>
            <div class="flex flex-wrap gap-2 mt-2">
              <span class="px-2 py-0.5 rounded-full bg-white/15 text-white/90 text-[10px] font-bold uppercase tracking-wider border border-white/20">
                <i class="fas fa-id-card mr-1"></i>{{ broker.license_number || 'No License' }}
              </span>
              <span class="px-2 py-0.5 rounded-full bg-white/15 text-white/90 text-[10px] font-bold uppercase tracking-wider border border-white/20">
                <i class="fas fa-percent mr-1"></i>{{ broker.commission_rate || 0 }}% Commission Rate
              </span>
            </div>
          </div>

          <span :class="broker.is_active ? 'bg-emerald-400/20 text-emerald-100 border-emerald-300/40' : 'bg-orange-400/20 text-orange-100 border-orange-300/40'"
            class="px-3 py-1.5 rounded-full text-[10px] font-black uppercase border tracking-widest shrink-0">
            {{ broker.is_active ? '● Active' : '○ Inactive' }}
          </span>
        </div>

        <!-- Contact row -->
        <div class="grid grid-cols-1 sm:grid-cols-4 divide-y sm:divide-y-0 sm:divide-x divide-slate-100 px-6 py-4">
          <div class="py-3 sm:py-0 sm:pr-6">
            <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest mb-1">Email</p>
            <p class="text-xs font-semibold text-slate-700 truncate">{{ broker.email || '—' }}</p>
          </div>
          <div class="py-3 sm:py-0 sm:px-6">
            <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest mb-1">Phone</p>
            <p class="text-xs font-semibold text-slate-700">{{ broker.phone_number || '—' }}</p>
          </div>
          <div class="py-3 sm:py-0 sm:px-6">
            <p class="text-[9px] font-black text-slateite-400 uppercase tracking-widest mb-1">Wallet / Bank</p>
            <p class="text-xs font-semibold text-slate-700 truncate">{{ broker.wallet || '—' }}</p>
          </div>
          <div class="py-3 sm:py-0 sm:pl-6">
            <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest mb-1">Date Joined</p>
            <p class="text-xs font-semibold text-slate-700">{{ fmt(broker.date_joined) }}</p>
          </div>
        </div>
      </div>

      <!-- ── KPI Summary Cards ──────────────────────────────────────── -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-3">
        <div class="bg-white rounded-xl border border-slate-100 shadow-sm p-4 flex flex-col gap-1">
          <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Total Sale Listings</p>
          <p class="text-2xl font-black text-primary">{{ saleListing.length }}</p>
          <p class="text-[9px] text-slate-400">properties posted for sale</p>
        </div>
        <div class="bg-white rounded-xl border border-slate-100 shadow-sm p-4 flex flex-col gap-1">
          <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Approved Sales</p>
          <p class="text-2xl font-black text-emerald-600">{{ approvedSales }}</p>
          <p class="text-[9px] text-slate-400">payment approved</p>
        </div>
        <div class="bg-white rounded-xl border border-slate-100 shadow-sm p-4 flex flex-col gap-1">
          <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Total Posting Fees</p>
          <p class="text-2xl font-black text-amber-600">{{ currency(totalPostingFees) }}</p>
          <p class="text-[9px] text-slate-400">fees paid for listing</p>
        </div>
        <div class="bg-white rounded-xl border border-slate-100 shadow-sm p-4 flex flex-col gap-1">
          <p class="text-[9px] font-black text-slate-400 uppercase tracking-widest">Broker Commissions</p>
          <p class="text-2xl font-black text-purple-600">{{ totalCommissions }}</p>
          <p class="text-[9px] text-slate-400">commission records</p>
        </div>
      </div>

      <!-- ── Tabs ──────────────────────────────────────────────────── -->
      <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
        <div class="border-b border-slate-100 flex gap-0.5 px-4 pt-3 overflow-x-auto">
          <button
            v-for="tab in tabs" :key="tab.key"
            @click="activeTab = tab.key"
            class="flex items-center gap-1.5 px-4 py-2.5 text-[11px] font-bold rounded-t-lg border-b-2 whitespace-nowrap transition-all"
            :class="activeTab === tab.key
              ? 'border-primary text-primary bg-primary/5'
              : 'border-transparent text-slate-500 hover:text-slate-700 hover:bg-slate-50'"
          >
            <i :class="tab.icon"></i>
            {{ tab.label }}
            <span v-if="tab.count !== undefined"
              class="px-1.5 py-0.5 rounded-full text-[9px] font-black"
              :class="activeTab === tab.key ? 'bg-primary text-white' : 'bg-slate-100 text-slate-500'">
              {{ tab.count }}
            </span>
          </button>
        </div>

        <!-- ── Tab: Sale Listings ─────────────────────────────────── -->
        <div v-if="activeTab === 'sale_listings'" class="p-5">
          <Loading :visible="loadingSales" message="Loading sale listings..." />
          <div class="overflow-x-auto rounded-lg border border-slate-100">
            <table class="w-full text-xs">
              <thead class="bg-slate-50 text-[10px] font-black text-slate-500 uppercase tracking-wide">
                <tr>
                  <th class="px-3 py-2.5 text-left">Property</th>
                  <th class="px-3 py-2.5 text-right">Listing Price</th>
                  <th class="px-3 py-2.5 text-right">Selling Price</th>
                  <th class="px-3 py-2.5 text-right">Posting Fee</th>
                  <th class="px-3 py-2.5 text-center">Payment Status</th>
                  <th class="px-3 py-2.5 text-center">Status</th>
                  <th class="px-3 py-2.5 text-left">Date</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr v-for="s in saleListing" :key="s.id" class="hover:bg-slate-50 transition">
                  <td class="px-3 py-2.5 font-semibold text-slate-800">
                    {{ s.property?.name || s.property_zone?.name || '—' }}
                  </td>
                  <td class="px-3 py-2.5 text-right text-slate-600">{{ currency(s.listing_price) }}</td>
                  <td class="px-3 py-2.5 text-right font-bold text-emerald-600">{{ currency(s.selling_price) }}</td>
                  <td class="px-3 py-2.5 text-right text-amber-700 font-semibold">{{ currency(s.posting_fee) }}</td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded-full text-[9px] font-black uppercase"
                      :class="s.posting_payment_status === 'approved'
                        ? 'bg-emerald-100 text-emerald-700'
                        : 'bg-amber-100 text-amber-700'">
                      {{ s.posting_payment_status || 'pending' }}
                    </span>
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded-full text-[9px] font-black uppercase"
                      :class="s.status === 'sold'
                        ? 'bg-blue-100 text-blue-700'
                        : s.status === 'for_sale'
                        ? 'bg-orange-100 text-orange-700'
                        : 'bg-slate-100 text-slate-600'">
                      {{ s.status || '—' }}
                    </span>
                  </td>
                  <td class="px-3 py-2.5 text-slate-400">{{ fmt(s.created_at) }}</td>
                </tr>
                <tr v-if="!saleListing.length && !loadingSales">
                  <td colspan="7" class="py-10 text-center text-slate-400 italic">No sale listings found for this broker.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- ── Tab: Sale Commissions ──────────────────────────────── -->
        <div v-if="activeTab === 'sale_commissions'" class="p-5">
          <Loading :visible="loadingSaleComm" message="Loading sale commissions..." />
          <div class="overflow-x-auto rounded-lg border border-slate-100">
            <table class="w-full text-xs">
              <thead class="bg-slate-50 text-[10px] font-black text-slate-500 uppercase tracking-wide">
                <tr>
                  <th class="px-3 py-2.5 text-left">Property Sale</th>
                  <th class="px-3 py-2.5 text-center">SaaS %</th>
                  <th class="px-3 py-2.5 text-center">Broker %</th>
                  <th class="px-3 py-2.5 text-center">Total %</th>
                  <th class="px-3 py-2.5 text-left">Date</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr v-for="c in saleCommissions" :key="c.id" class="hover:bg-slate-50 transition">
                  <td class="px-3 py-2.5 font-semibold text-slate-800">
                    {{ c.property_sale_name || c.property_sale || '—' }}
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-blue-50 text-blue-700 border border-blue-100 font-bold">{{ c.saas_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-purple-50 text-purple-700 border border-purple-100 font-bold">{{ c.broker_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-emerald-50 text-emerald-700 border border-emerald-100 font-black">{{ c.total_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-slate-400">{{ fmt(c.created_at) }}</td>
                </tr>
                <tr v-if="!saleCommissions.length && !loadingSaleComm">
                  <td colspan="5" class="py-10 text-center text-slate-400 italic">No sale commissions found.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- ── Tab: Rent Commissions ──────────────────────────────── -->
        <div v-if="activeTab === 'rent_commissions'" class="p-5">
          <Loading :visible="loadingRentComm" message="Loading rent commissions..." />
          <div class="overflow-x-auto rounded-lg border border-slate-100">
            <table class="w-full text-xs">
              <thead class="bg-slate-50 text-[10px] font-black text-slate-500 uppercase tracking-wide">
                <tr>
                  <th class="px-3 py-2.5 text-left">Rent Ref</th>
                  <th class="px-3 py-2.5 text-center">SaaS %</th>
                  <th class="px-3 py-2.5 text-center">Broker %</th>
                  <th class="px-3 py-2.5 text-center">Total %</th>
                  <th class="px-3 py-2.5 text-left">Date</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr v-for="c in rentCommissions" :key="c.id" class="hover:bg-slate-50 transition">
                  <td class="px-3 py-2.5 font-semibold text-slate-700">#{{ c.rent || '—' }}</td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-blue-50 text-blue-700 border border-blue-100 font-bold">{{ c.saas_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-purple-50 text-purple-700 border border-purple-100 font-bold">{{ c.broker_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded bg-emerald-50 text-emerald-700 border border-emerald-100 font-black">{{ c.total_commission }}%</span>
                  </td>
                  <td class="px-3 py-2.5 text-slate-400">{{ fmt(c.created_at) }}</td>
                </tr>
                <tr v-if="!rentCommissions.length && !loadingRentComm">
                  <td colspan="5" class="py-10 text-center text-slate-400 italic">No rent commissions found.</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- ── Tab: Posting Fees ──────────────────────────────────── -->
        <div v-if="activeTab === 'posting_fees'" class="p-5">
          <div class="mb-4 p-4 bg-amber-50 border border-amber-200 rounded-xl flex items-start gap-3">
            <i class="fas fa-info-circle text-amber-500 mt-0.5 shrink-0"></i>
            <div>
              <p class="text-xs font-bold text-amber-800">Posting Fees Summary</p>
              <p class="text-[11px] text-amber-700 mt-0.5">
                These are fees paid by the broker to list properties for sale on the platform.
              </p>
            </div>
          </div>
          <div class="overflow-x-auto rounded-lg border border-slate-100">
            <table class="w-full text-xs">
              <thead class="bg-slate-50 text-[10px] font-black text-slate-500 uppercase tracking-wide">
                <tr>
                  <th class="px-3 py-2.5 text-left">Property</th>
                  <th class="px-3 py-2.5 text-right">Listing Price</th>
                  <th class="px-3 py-2.5 text-right">Posting Fee</th>
                  <th class="px-3 py-2.5 text-center">Fee Status</th>
                  <th class="px-3 py-2.5 text-left">Date</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100">
                <tr v-for="s in saleListing" :key="s.id" class="hover:bg-slate-50 transition">
                  <td class="px-3 py-2.5 font-semibold text-slate-800">
                    {{ s.property?.name || s.property_zone?.name || '—' }}
                  </td>
                  <td class="px-3 py-2.5 text-right text-slate-600">{{ currency(s.listing_price) }}</td>
                  <td class="px-3 py-2.5 text-right font-black text-amber-700">{{ currency(s.posting_fee) }}</td>
                  <td class="px-3 py-2.5 text-center">
                    <span class="px-2 py-0.5 rounded-full text-[9px] font-black uppercase"
                      :class="s.posting_payment_status === 'approved'
                        ? 'bg-emerald-100 text-emerald-700'
                        : 'bg-amber-100 text-amber-700'">
                      {{ s.posting_payment_status || 'pending' }}
                    </span>
                  </td>
                  <td class="px-3 py-2.5 text-slate-400">{{ fmt(s.created_at) }}</td>
                </tr>
                <tr v-if="!saleListing.length && !loadingSales">
                  <td colspan="5" class="py-10 text-center text-slate-400 italic">No posting fee records found.</td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- Fee total footer -->
          <div v-if="saleListing.length" class="mt-4 flex justify-end">
            <div class="bg-amber-50 border border-amber-200 rounded-xl px-6 py-3 flex items-center gap-4">
              <span class="text-xs font-black text-amber-700 uppercase tracking-wider">Total Posting Fees Paid</span>
              <span class="text-lg font-black text-amber-800">{{ currency(totalPostingFees) }}</span>
            </div>
          </div>
        </div>

      </div><!-- end tab card -->
    </div><!-- end v-if broker -->

    <!-- Not found state -->
    <div v-if="!broker && !loading" class="flex flex-col items-center justify-center mt-20 text-slate-400">
      <i class="fas fa-user-slash text-5xl mb-4"></i>
      <p class="text-sm font-bold uppercase tracking-wider">Broker not found</p>
    </div>
  </div>
</template>

<script>
import Toast from "@/components/Toast.vue";
import Loading from "@/components/Loading.vue";

export default {
  name: "BrokerDetail",
  components: { Toast, Loading },

  data() {
    return {
      broker: null,
      saleListing:    [],
      saleCommissions: [],
      rentCommissions: [],

      loading:         false,
      loadingSales:    false,
      loadingSaleComm: false,
      loadingRentComm: false,

      activeTab: "sale_listings",
    };
  },

  computed: {
    fullName() {
      if (!this.broker) return "";
      return [this.broker.first_name, this.broker.middle_name, this.broker.last_name]
        .filter(Boolean).join(" ");
    },
    approvedSales() {
      return this.saleListing.filter(s => s.posting_payment_status === "approved").length;
    },
    totalPostingFees() {
      return this.saleListing.reduce((sum, s) => sum + (parseFloat(s.posting_fee) || 0), 0);
    },
    totalCommissions() {
      return this.saleCommissions.length + this.rentCommissions.length;
    },
    tabs() {
      return [
        { key: "sale_listings",   label: "Sale Listings",      icon: "fas fa-tags",         count: this.saleListing.length },
        { key: "posting_fees",    label: "Posting Fees",        icon: "fas fa-receipt",       count: this.saleListing.length },
        { key: "sale_commissions",label: "Sale Commissions",    icon: "fas fa-money-bill-wave",count: this.saleCommissions.length },
        { key: "rent_commissions",label: "Rent Commissions",    icon: "fas fa-file-contract", count: this.rentCommissions.length },
      ];
    },
  },

  async mounted() {
    const profileId = this.$route.params.id;   // broker profile ID (from broker list)
    await this.loadBroker(profileId);
  },

  methods: {
    async loadBroker(profileId) {
      this.loading = true;
      try {
        // Fetch the broker profile by id via the list endpoint
        const res = await this.$apiGet("/get_broker_profiles", { page_size: 1000 });
        const list = res.data || [];
        const profile = (list.results || list).find(b => String(b.id) === String(profileId));

        if (profile) {
          // Enrich with user details
          const user = await this.$apiGetById("/get_user", profile.user);
          this.broker = {
            ...profile,
            first_name:   user.first_name   || "",
            middle_name:  user.middle_name  || "",
            last_name:    user.last_name    || "",
            email:        user.email        || "",
            phone_number: user.phone_number || "",
            is_active:    user.is_active    ?? true,
            date_joined:  user.date_joined  || "",
          };

          // Load all tab data in parallel using broker user id as the filter key
          this.loadSaleListing(profileId);
          this.loadSaleCommissions(profileId);
          this.loadRentCommissions(profileId);
        }
      } catch (e) {
        console.error(e);
        this.broker = null;
      } finally {
        this.loading = false;
      }
    },

    async loadSaleListing(brokerId) {
      this.loadingSales = true;
      try {
        const res = await this.$apiGet("/get_broker_property_sales", {
          broker__id: brokerId,
          page_size: 1000,
        });
        this.saleListing = res.data || [];
      } catch (e) {
        console.error(e);
        this.saleListing = [];
      } finally {
        this.loadingSales = false;
      }
    },

    async loadSaleCommissions(brokerId) {
      this.loadingSaleComm = true;
      try {
        const res = await this.$apiGet("/get_commissions", {
          broker__id: brokerId,
          page_size: 1000,
        });
        this.saleCommissions = res.data || [];
      } catch (e) {
        console.error(e);
        this.saleCommissions = [];
      } finally {
        this.loadingSaleComm = false;
      }
    },

    async loadRentCommissions(brokerId) {
      this.loadingRentComm = true;
      try {
        const res = await this.$apiGet("/get_rent_commissions", {
          broker__id: brokerId,
          page_size: 1000,
        });
        this.rentCommissions = res.data || [];
      } catch (e) {
        console.error(e);
        this.rentCommissions = [];
      } finally {
        this.loadingRentComm = false;
      }
    },

    initials(first, last) {
      return ((first || "")[0] || "") + ((last || "")[0] || "") || "?";
    },
    fmt(d) {
      return d ? new Date(d).toLocaleDateString() : "—";
    },
    currency(v) {
      if (v === null || v === undefined || v === "") return "—";
      const n = parseFloat(v);
      if (isNaN(n)) return v;
      return n.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    },
  },
};
</script>

<style scoped>
.btn-action { @apply h-7 w-7 flex items-center justify-center rounded-lg text-xs transition; }
.btn-blue   { @apply bg-blue-50 text-blue-600 hover:bg-blue-600 hover:text-white; }
</style>
