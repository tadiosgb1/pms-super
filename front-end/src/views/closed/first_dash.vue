<template>
  <div class="p-6 bg-slate-50 min-h-screen">
    <!-- Header Section with Role-Based Title -->
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center mb-8 gap-4">
      <div>
        <h1 class="text-2xl font-black text-slate-800 tracking-tight">
          {{ dashboardTitle }}
        </h1>
        <p class="text-sm text-slate-500 font-medium uppercase tracking-widest text-[10px]">
          {{ dashboardSubtitle }}
        </p>
      </div>
      <div class="flex items-center gap-2 px-4 py-2 bg-white rounded-xl shadow-sm border border-slate-200">
        <span class="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
        <span class="text-xs font-bold text-slate-600 uppercase tracking-tighter">Live System Status</span>
      </div>
    </div>

    <!-- Key Metrics & KPIs - Role-Based Stats Cards -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-10">
      <div
        v-for="(metric, index) in displayedMetrics"
        :key="index"
        class="bg-white shadow-sm rounded-[1.5rem] p-6 flex flex-col justify-between border border-slate-200 hover:border-primary/50 hover:shadow-xl hover:shadow-primary/5 transition-all duration-300 group"
      >
        <div class="flex justify-between items-start">
          <div>
            <h2 class="text-slate-400 text-[10px] font-black uppercase tracking-[0.15em] mb-1">
              {{ metric.label }}
            </h2>
            <p class="text-3xl font-black text-slate-800 group-hover:text-primary transition-colors">
              {{ metric.value }}
            </p>
            <p v-if="metric.subtitle" class="text-xs text-slate-500 mt-1">{{ metric.subtitle }}</p>
          </div>
          <div :class="`w-10 h-10 rounded-xl flex items-center justify-center group-hover:bg-primary/10 group-hover:text-primary transition-all ${metric.bgColor || 'bg-slate-50'} ${metric.iconColor || 'text-slate-400'}`">
            <i :class="`${metric.icon} text-sm`"></i>
          </div>
        </div>
        <div class="mt-4 border-t border-slate-50 pt-3 flex items-center justify-between">
          <span class="text-[9px] font-bold text-slate-400 uppercase tracking-widest">
            {{ metric.updateText || 'Updated Just Now' }}
          </span>
          <i class="fas fa-chevron-right text-[8px] text-slate-300"></i>
        </div>
      </div>
    </div>

    <!-- Super Admin: Platform Health & System Alerts (Sample Data) -->
    <div v-if="userRole === 'super_admin'" class="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-10">
      <div class="bg-gradient-to-br from-green-50 to-green-100 rounded-[1.5rem] p-6 border border-green-200">
        <div class="flex items-center gap-3 mb-4">
          <div class="w-10 h-10 rounded-xl bg-green-500 text-white flex items-center justify-center">
            <i class="fas fa-server"></i>
          </div>
          <div>
            <h3 class="text-sm font-black text-green-900 uppercase">System Health</h3>
            <p class="text-xs text-green-600">All Systems Operational</p>
          </div>
        </div>
        <div class="space-y-2 text-xs">
          <div class="flex justify-between"><span class="text-green-700">API Uptime:</span><span class="font-bold text-green-900">99.98%</span></div>
          <div class="flex justify-between"><span class="text-green-700">Database Load:</span><span class="font-bold text-green-900">42%</span></div>
          <div class="flex justify-between"><span class="text-green-700">Storage Usage:</span><span class="font-bold text-green-900">67%</span></div>
        </div>
      </div>

      <div class="bg-gradient-to-br from-blue-50 to-blue-100 rounded-[1.5rem] p-6 border border-blue-200">
        <div class="flex items-center gap-3 mb-4">
          <div class="w-10 h-10 rounded-xl bg-blue-500 text-white flex items-center justify-center">
            <i class="fas fa-dollar-sign"></i>
          </div>
          <div>
            <h3 class="text-sm font-black text-blue-900 uppercase">Revenue Metrics</h3>
            <p class="text-xs text-blue-600">Monthly Performance</p>
          </div>
        </div>
        <div class="space-y-2 text-xs">
          <div class="flex justify-between"><span class="text-blue-700">MRR:</span><span class="font-bold text-blue-900">${{ sampleData.mrr.toLocaleString() }}</span></div>
          <div class="flex justify-between"><span class="text-blue-700">ARPU:</span><span class="font-bold text-blue-900">${{ sampleData.arpu }}</span></div>
          <div class="flex justify-between"><span class="text-blue-700">Payment Success:</span><span class="font-bold text-blue-900">{{ sampleData.paymentSuccess }}%</span></div>
        </div>
      </div>

      <div class="bg-gradient-to-br from-amber-50 to-amber-100 rounded-[1.5rem] p-6 border border-amber-200">
        <div class="flex items-center gap-3 mb-4">
          <div class="w-10 h-10 rounded-xl bg-amber-500 text-white flex items-center justify-center">
            <i class="fas fa-headset"></i>
          </div>
          <div>
            <h3 class="text-sm font-black text-amber-900 uppercase">Support Queue</h3>
            <p class="text-xs text-amber-600">Active Tickets</p>
          </div>
        </div>
        <div class="space-y-2 text-xs">
          <div class="flex justify-between"><span class="text-amber-700">Open Tickets:</span><span class="font-bold text-amber-900">{{ sampleData.openTickets }}</span></div>
          <div class="flex justify-between"><span class="text-amber-700">Avg Response:</span><span class="font-bold text-amber-900">{{ sampleData.avgResponseTime }}</span></div>
          <div class="flex justify-between"><span class="text-amber-700">High Priority:</span><span class="font-bold text-amber-900">{{ sampleData.highPriorityTickets }}</span></div>
        </div>
      </div>
    </div>

    <!-- Landlord/Owner: Lease Expiry Alert (Sample Data) -->
    <div v-if="userRole === 'owner' || userRole === 'manager'" class="bg-gradient-to-r from-orange-50 to-red-50 rounded-[1.5rem] p-6 border border-orange-200 mb-10">
      <div class="flex items-center justify-between">
        <div class="flex items-center gap-4">
          <div class="w-12 h-12 rounded-xl bg-orange-500 text-white flex items-center justify-center">
            <i class="fas fa-calendar-times text-lg"></i>
          </div>
          <div>
            <h3 class="text-lg font-black text-orange-900 uppercase">Lease Expiry Alert</h3>
            <p class="text-sm text-orange-600">{{ sampleData.leaseExpiring30 }} leases expiring in next 30 days</p>
          </div>
        </div>
        <button class="bg-orange-500 hover:bg-orange-600 text-white px-6 py-2 rounded-xl font-bold text-xs uppercase transition">
          View Details
        </button>
      </div>
    </div>

    <!-- Coworking Owner: Utilization Gauge (Sample Data) -->
    <div v-if="userRole === 'coworking_owner'" class="bg-white rounded-[1.5rem] shadow-lg p-8 mb-10 border border-slate-200">
      <div class="flex items-center gap-4 mb-6">
        <div class="w-12 h-12 rounded-xl bg-purple-500 text-white flex items-center justify-center">
          <i class="fas fa-chart-pie text-lg"></i>
        </div>
        <div>
          <h3 class="text-xl font-black text-slate-800 uppercase">Current Space Utilization</h3>
          <p class="text-sm text-slate-500">Real-time desk and room occupancy</p>
        </div>
      </div>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div class="text-center p-4 bg-purple-50 rounded-xl">
          <p class="text-4xl font-black text-purple-600">{{ sampleData.hotDeskUtilization }}%</p>
          <p class="text-xs font-bold text-slate-600 uppercase mt-2">Hot Desks</p>
        </div>
        <div class="text-center p-4 bg-blue-50 rounded-xl">
          <p class="text-4xl font-black text-blue-600">{{ sampleData.dedicatedDeskUtilization }}%</p>
          <p class="text-xs font-bold text-slate-600 uppercase mt-2">Dedicated Desks</p>
        </div>
        <div class="text-center p-4 bg-green-50 rounded-xl">
          <p class="text-4xl font-black text-green-600">{{ sampleData.meetingRoomUtilization }}%</p>
          <p class="text-xs font-bold text-slate-600 uppercase mt-2">Meeting Rooms</p>
        </div>
      </div>
    </div>

    <!-- PMS COMPREHENSIVE REPORTS SECTION -->
    <!-- Property Rent, Sale & Coworking Analytics -->
    <div class="bg-white rounded-[2rem] shadow-xl shadow-slate-200/50 border border-slate-100 p-8 mb-10">
      <div class="flex items-center gap-4 mb-8">
        <div class="w-12 h-12 rounded-2xl bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center text-white">
          <i class="fas fa-chart-bar text-lg"></i>
        </div>
        <div>
          <h2 class="text-2xl font-black text-slate-800 tracking-tight uppercase">PMS Analytics Overview</h2>
          <p class="text-sm text-slate-500">Property Rent, Sale & Coworking Performance</p>
        </div>
      </div>

      <!-- Three Column Layout: Rent | Sale | Coworking -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 mb-8">
        <!-- PROPERTY RENT SECTION -->
        <div class="bg-gradient-to-br from-green-50 to-emerald-100 rounded-[1.5rem] p-6 border border-green-200">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-10 h-10 rounded-xl bg-green-600 text-white flex items-center justify-center">
              <i class="fas fa-home"></i>
            </div>
            <h3 class="text-lg font-black text-green-900 uppercase">Property Rent</h3>
          </div>
          <div class="space-y-3">
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-green-700 font-bold uppercase">Total Rented Units</p>
              <p class="text-2xl font-black text-green-900">{{ pmsData.rent.totalRentedUnits }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-green-700 font-bold uppercase">Monthly Rent Income</p>
              <p class="text-2xl font-black text-green-900">${{ pmsData.rent.monthlyRentIncome.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-red-700 font-bold uppercase">Overdue Rents</p>
              <p class="text-2xl font-black text-red-900">{{ pmsData.rent.overdueCount }}</p>
              <p class="text-sm text-red-700 font-bold">${{ pmsData.rent.overdueAmount.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-green-700 font-bold uppercase">Paid This Month</p>
              <p class="text-2xl font-black text-green-900">{{ pmsData.rent.paidThisMonth }}</p>
              <p class="text-sm text-green-700 font-bold">${{ pmsData.rent.paidAmount.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-amber-700 font-bold uppercase">Pending Payments</p>
              <p class="text-2xl font-black text-amber-900">{{ pmsData.rent.pendingCount }}</p>
            </div>
          </div>
        </div>

        <!-- PROPERTY SALE SECTION -->
        <div class="bg-gradient-to-br from-blue-50 to-cyan-100 rounded-[1.5rem] p-6 border border-blue-200">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-10 h-10 rounded-xl bg-blue-600 text-white flex items-center justify-center">
              <i class="fas fa-tag"></i>
            </div>
            <h3 class="text-lg font-black text-blue-900 uppercase">Property Sale</h3>
          </div>
          <div class="space-y-3">
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-blue-700 font-bold uppercase">Properties for Sale</p>
              <p class="text-2xl font-black text-blue-900">{{ pmsData.sale.totalForSale }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-green-700 font-bold uppercase">Sold This Month</p>
              <p class="text-2xl font-black text-green-900">{{ pmsData.sale.soldThisMonth }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-blue-700 font-bold uppercase">Total Sales Value</p>
              <p class="text-2xl font-black text-blue-900">${{ pmsData.sale.totalSalesValue.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-purple-700 font-bold uppercase">Avg Sale Price</p>
              <p class="text-2xl font-black text-purple-900">${{ pmsData.sale.avgSalePrice.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-amber-700 font-bold uppercase">Under Negotiation</p>
              <p class="text-2xl font-black text-amber-900">{{ pmsData.sale.underNegotiation }}</p>
            </div>
          </div>
        </div>

        <!-- COWORKING SPACE SECTION -->
        <div class="bg-gradient-to-br from-purple-50 to-pink-100 rounded-[1.5rem] p-6 border border-purple-200">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-10 h-10 rounded-xl bg-purple-600 text-white flex items-center justify-center">
              <i class="fas fa-briefcase"></i>
            </div>
            <h3 class="text-lg font-black text-purple-900 uppercase">Coworking Space</h3>
          </div>
          <div class="space-y-3">
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-purple-700 font-bold uppercase">Total Spaces</p>
              <p class="text-2xl font-black text-purple-900">{{ pmsData.coworking.totalSpaces }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-purple-700 font-bold uppercase">Active Bookings</p>
              <p class="text-2xl font-black text-purple-900">{{ pmsData.coworking.activeBookings }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-green-700 font-bold uppercase">Monthly Revenue</p>
              <p class="text-2xl font-black text-green-900">${{ pmsData.coworking.monthlyRevenue.toLocaleString() }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-purple-700 font-bold uppercase">Occupancy Rate</p>
              <p class="text-2xl font-black text-purple-900">{{ pmsData.coworking.occupancyRate }}%</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3">
              <p class="text-xs text-amber-700 font-bold uppercase">Overdue Payments</p>
              <p class="text-2xl font-black text-amber-900">{{ pmsData.coworking.overduePayments }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Zones & Roles Analytics -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
        <!-- ZONES BREAKDOWN -->
        <div class="bg-gradient-to-br from-indigo-50 to-blue-100 rounded-[1.5rem] p-6 border border-indigo-200">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-10 h-10 rounded-xl bg-indigo-600 text-white flex items-center justify-center">
              <i class="fas fa-map-marked-alt"></i>
            </div>
            <h3 class="text-lg font-black text-indigo-900 uppercase">Zones & Units</h3>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div class="bg-white/60 rounded-xl p-4 text-center">
              <p class="text-xs text-indigo-700 font-bold uppercase mb-2">Total Zones</p>
              <p class="text-3xl font-black text-indigo-900">{{ stats.totalZones }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-4 text-center">
              <p class="text-xs text-green-700 font-bold uppercase mb-2">Occupied Zones</p>
              <p class="text-3xl font-black text-green-900">{{ pmsData.zones.occupied }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-4 text-center">
              <p class="text-xs text-red-700 font-bold uppercase mb-2">Vacant Zones</p>
              <p class="text-3xl font-black text-red-900">{{ pmsData.zones.vacant }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-4 text-center">
              <p class="text-xs text-amber-700 font-bold uppercase mb-2">Under Maintenance</p>
              <p class="text-3xl font-black text-amber-900">{{ pmsData.zones.maintenance }}</p>
            </div>
          </div>
          <div class="mt-4 bg-white/60 rounded-xl p-4">
            <div class="flex justify-between items-center mb-2">
              <span class="text-xs font-bold text-indigo-700 uppercase">Occupancy Rate</span>
              <span class="text-lg font-black text-indigo-900">{{ pmsData.zones.occupancyRate }}%</span>
            </div>
            <div class="w-full bg-indigo-200 rounded-full h-3">
              <div class="bg-indigo-600 h-3 rounded-full transition-all duration-500" :style="`width: ${pmsData.zones.occupancyRate}%`"></div>
            </div>
          </div>
        </div>

        <!-- ROLES & USERS -->
        <div class="bg-gradient-to-br from-rose-50 to-orange-100 rounded-[1.5rem] p-6 border border-rose-200">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-10 h-10 rounded-xl bg-rose-600 text-white flex items-center justify-center">
              <i class="fas fa-users-cog"></i>
            </div>
            <h3 class="text-lg font-black text-rose-900 uppercase">Roles & Users</h3>
          </div>
          <div class="space-y-3">
            <div class="bg-white/60 rounded-xl p-3 flex justify-between items-center">
              <div>
                <p class="text-xs text-rose-700 font-bold uppercase">Owners</p>
                <p class="text-sm text-rose-600">Property Owners</p>
              </div>
              <p class="text-2xl font-black text-rose-900">{{ pmsData.roles.owners }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3 flex justify-between items-center">
              <div>
                <p class="text-xs text-blue-700 font-bold uppercase">Managers</p>
                <p class="text-sm text-blue-600">Property Managers</p>
              </div>
              <p class="text-2xl font-black text-blue-900">{{ pmsData.roles.managers }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3 flex justify-between items-center">
              <div>
                <p class="text-xs text-green-700 font-bold uppercase">Staff</p>
                <p class="text-sm text-green-600">Support Staff</p>
              </div>
              <p class="text-2xl font-black text-green-900">{{ pmsData.roles.staff }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3 flex justify-between items-center">
              <div>
                <p class="text-xs text-purple-700 font-bold uppercase">Tenants</p>
                <p class="text-sm text-purple-600">Active Tenants</p>
              </div>
              <p class="text-2xl font-black text-purple-900">{{ stats.totalTenants }}</p>
            </div>
            <div class="bg-white/60 rounded-xl p-3 flex justify-between items-center">
              <div>
                <p class="text-xs text-amber-700 font-bold uppercase">Brokers</p>
                <p class="text-sm text-amber-600">Real Estate Agents</p>
              </div>
              <p class="text-2xl font-black text-amber-900">{{ pmsData.roles.brokers }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Financial Summary Cards -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <div class="bg-gradient-to-br from-emerald-500 to-green-600 rounded-[1.5rem] p-6 text-white">
          <div class="flex items-center gap-3 mb-3">
            <i class="fas fa-dollar-sign text-2xl"></i>
            <p class="text-xs font-black uppercase tracking-wider opacity-90">Total Revenue</p>
          </div>
          <p class="text-3xl font-black mb-1">${{ pmsData.financial.totalRevenue.toLocaleString() }}</p>
          <p class="text-xs opacity-75">This Month</p>
        </div>

        <div class="bg-gradient-to-br from-red-500 to-rose-600 rounded-[1.5rem] p-6 text-white">
          <div class="flex items-center gap-3 mb-3">
            <i class="fas fa-exclamation-triangle text-2xl"></i>
            <p class="text-xs font-black uppercase tracking-wider opacity-90">Total Overdue</p>
          </div>
          <p class="text-3xl font-black mb-1">${{ pmsData.financial.totalOverdue.toLocaleString() }}</p>
          <p class="text-xs opacity-75">{{ pmsData.financial.overdueCount }} Accounts</p>
        </div>

        <div class="bg-gradient-to-br from-blue-500 to-cyan-600 rounded-[1.5rem] p-6 text-white">
          <div class="flex items-center gap-3 mb-3">
            <i class="fas fa-chart-line text-2xl"></i>
            <p class="text-xs font-black uppercase tracking-wider opacity-90">Expected Income</p>
          </div>
          <p class="text-3xl font-black mb-1">${{ pmsData.financial.expectedIncome.toLocaleString() }}</p>
          <p class="text-xs opacity-75">Next 30 Days</p>
        </div>

        <div class="bg-gradient-to-br from-purple-500 to-pink-600 rounded-[1.5rem] p-6 text-white">
          <div class="flex items-center gap-3 mb-3">
            <i class="fas fa-percentage text-2xl"></i>
            <p class="text-xs font-black uppercase tracking-wider opacity-90">Collection Rate</p>
          </div>
          <p class="text-3xl font-black mb-1">{{ pmsData.financial.collectionRate }}%</p>
          <p class="text-xs opacity-75">This Month</p>
        </div>
      </div>
    </div>

    <!-- Overdue Rents Detailed Table -->
    <div v-if="userRole !== 'tenant' && userRole !== 'broker'" class="bg-white rounded-[2rem] shadow-xl shadow-slate-200/50 border border-slate-100 p-8 mb-10">
      <div class="flex items-center justify-between mb-6">
        <div class="flex items-center gap-4">
          <div class="w-12 h-12 rounded-2xl bg-red-500 text-white flex items-center justify-center">
            <i class="fas fa-exclamation-circle text-lg"></i>
          </div>
          <div>
            <h2 class="text-2xl font-black text-slate-800 tracking-tight uppercase">Overdue Rents</h2>
            <p class="text-sm text-slate-500">Requires immediate attention</p>
          </div>
        </div>
        <button class="bg-red-500 hover:bg-red-600 text-white px-6 py-3 rounded-xl font-bold text-xs uppercase transition shadow-lg">
          <i class="fas fa-bell mr-2"></i>Send Reminders
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full">
          <thead>
            <tr class="border-b-2 border-slate-200">
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Tenant</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Property</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Unit</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Amount</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Due Date</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Days Overdue</th>
              <th class="text-left py-4 px-4 text-xs font-black text-slate-600 uppercase tracking-wider">Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(overdue, index) in overdueRentsList" :key="index" class="border-b border-slate-100 hover:bg-slate-50 transition">
              <td class="py-4 px-4">
                <div class="flex items-center gap-3">
                  <div class="w-8 h-8 rounded-full bg-slate-200 flex items-center justify-center text-xs font-bold text-slate-600">
                    {{ overdue.tenant.charAt(0) }}
                  </div>
                  <span class="font-bold text-slate-800">{{ overdue.tenant }}</span>
                </div>
              </td>
              <td class="py-4 px-4 text-sm text-slate-600">{{ overdue.property }}</td>
              <td class="py-4 px-4 text-sm text-slate-600">{{ overdue.unit }}</td>
              <td class="py-4 px-4 font-black text-red-600">${{ overdue.amount.toLocaleString() }}</td>
              <td class="py-4 px-4 text-sm text-slate-600">{{ overdue.dueDate }}</td>
              <td class="py-4 px-4">
                <span :class="`px-3 py-1 rounded-full text-xs font-bold ${overdue.daysOverdue > 30 ? 'bg-red-100 text-red-700' : 'bg-amber-100 text-amber-700'}`">
                  {{ overdue.daysOverdue }} days
                </span>
              </td>
              <td class="py-4 px-4">
                <button class="text-blue-600 hover:text-blue-800 font-bold text-xs uppercase">
                  <i class="fas fa-paper-plane mr-1"></i>Remind
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="is_superuser == 'true'"
      class="bg-white rounded-[2rem] shadow-xl shadow-slate-200/50 border border-slate-100 mb-10 overflow-hidden"
    >
      <div class="p-8 pb-0">
        <div class="flex flex-col md:flex-row md:items-center justify-between gap-6 mb-8">
          <div class="flex items-center gap-4">
            <div class="w-12 h-12 rounded-2xl bg-secondary/10 flex items-center justify-center text-secondary">
              <i class="fas fa-crown"></i>
            </div>
            <div>
              <h2 class="text-xl font-black text-slate-800 tracking-tight uppercase">Pricing Plan Analytics</h2>
              <p class="text-xs font-bold text-slate-400">Revenue and Tier Performance</p>
            </div>
          </div>

          <div class="flex flex-wrap items-center gap-3 bg-slate-50 p-2 rounded-2xl border border-slate-100">
            <div class="flex items-center gap-2">
              <input
                type="date"
                v-model="start_date"
                class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-700 focus:ring-2 focus:ring-primary/20 outline-none transition"
              />
              <span class="text-slate-300 text-xs">to</span>
              <input
                type="date"
                v-model="end_date"
                class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-700 focus:ring-2 focus:ring-primary/20 outline-none transition"
              />
            </div>
            <button
              @click="refreshAllReports"
              class="bg-dprimary hover:bg-primary text-white font-black text-[10px] tracking-widest px-6 py-3 rounded-xl shadow-lg shadow-dprimary/20 transition-all active:scale-95 uppercase"
            >
              Fetch Report
            </button>
          </div>
        </div>

        <div class="bg-slate-50/50 rounded-3xl p-4 border border-slate-50">
          <apexchart
            type="line"
            height="350"
            :options="pricingChartOptions"
            :series="pricingSeries"
          />
        </div>
      </div>
    </div>

    <div class="bg-white rounded-[2rem] shadow-xl shadow-slate-200/50 border border-slate-100 p-8 space-y-8">
      <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-6">
        <div class="flex items-center gap-3">
          <div class="w-2 h-8 bg-primary rounded-full"></div>
          <h2 class="text-xl font-black text-slate-800 tracking-tight uppercase">Income / Expense Overview</h2>
        </div>

        <div class="flex flex-row items-center gap-4 bg-slate-50 p-3 rounded-2xl border border-slate-100">
          <div class="flex flex-col">
            <label class="text-[9px] font-black text-slate-400 uppercase tracking-widest ml-1 mb-1">Start</label>
            <input
              type="date"
              v-model="revenue_start_date"
              @change="fetchIncomeData"
              class="bg-white border border-slate-200 rounded-xl px-3 py-1.5 text-xs font-bold text-slate-700 outline-none"
            />
          </div>
          <div class="flex flex-col">
            <label class="text-[9px] font-black text-slate-400 uppercase tracking-widest ml-1 mb-1">End</label>
            <input
              type="date"
              v-model="revenue_end_date"
              @change="fetchIncomeData"
              class="bg-white border border-slate-200 rounded-xl px-3 py-1.5 text-xs font-bold text-slate-700 outline-none"
            />
          </div>
        </div>
      </div>

      <div class="h-px bg-gradient-to-r from-transparent via-slate-200 to-transparent"></div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="p-6 rounded-[2rem] border border-slate-100 bg-slate-50/50 hover:bg-white hover:shadow-lg transition-all duration-500">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-8 h-8 rounded-lg bg-red-100 text-red-500 flex items-center justify-center text-xs"><i class="fas fa-arrow-up"></i></div>
            <h3 class="text-sm font-black text-slate-700 uppercase tracking-tight">Subscription (Expense)</h3>
          </div>
          <apexchart
            type="bar"
            height="280"
            :options="chartOptions('Subscription', incomeData.months)"
            :series="[{ name: 'Subscription', data: incomeData.subscription }]"
          />
        </div>

        <div class="p-6 rounded-[2rem] border border-slate-100 bg-slate-50/50 hover:bg-white hover:shadow-lg transition-all duration-500">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-8 h-8 rounded-lg bg-green-100 text-green-500 flex items-center justify-center text-xs"><i class="fas fa-home"></i></div>
            <h3 class="text-sm font-black text-slate-700 uppercase tracking-tight">Rent (Income)</h3>
          </div>
          <apexchart
            type="bar"
            height="280"
            :options="chartOptions('Rent', incomeData.months)"
            :series="[{ name: 'Rent', data: incomeData.rent }]"
          />
        </div>

        <div class="p-6 rounded-[2rem] border border-slate-100 bg-slate-50/50 hover:bg-white hover:shadow-lg transition-all duration-500">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-8 h-8 rounded-lg bg-blue-100 text-blue-500 flex items-center justify-center text-xs"><i class="fas fa-tag"></i></div>
            <h3 class="text-sm font-black text-slate-700 uppercase tracking-tight">Sales (Income)</h3>
          </div>
          <apexchart
            type="bar"
            height="280"
            :options="chartOptions('Sale', incomeData.months)"
            :series="[{ name: 'Sale', data: incomeData.sale }]"
          />
        </div>

        <div class="p-6 rounded-[2rem] border border-slate-100 bg-slate-50/50 hover:bg-white hover:shadow-lg transition-all duration-500">
          <div class="flex items-center gap-3 mb-6">
            <div class="w-8 h-8 rounded-lg bg-purple-100 text-purple-500 flex items-center justify-center text-xs"><i class="fas fa-briefcase"></i></div>
            <h3 class="text-sm font-black text-slate-700 uppercase tracking-tight">Workspace (Income)</h3>
          </div>
          <apexchart
            type="bar"
            height="280"
            :options="chartOptions('Workspace', incomeData.months)"
            :series="[{ name: 'Workspace', data: incomeData.workspace }]"
          />
        </div>
      </div>

      <div v-if="is_superuser == 'true'" class="bg-slate-900 rounded-[2.5rem] p-8 text-white mt-8 overflow-hidden relative group">
        <div class="absolute top-0 right-0 w-64 h-64 bg-primary/10 rounded-full blur-3xl -mr-32 -mt-32 transition-colors group-hover:bg-primary/20"></div>
        
        <div class="relative z-10 flex flex-col lg:flex-row items-center gap-12">
          <div class="lg:w-1/3">
            <h2 class="text-2xl font-black tracking-tight mb-2">User Distribution</h2>
            <p class="text-slate-400 text-sm font-medium mb-6">Analyze how your user base is split across different functional groups.</p>
            <div class="space-y-4">
              <div class="flex items-center gap-3 text-xs font-bold uppercase tracking-widest text-primary">
                <i class="fas fa-circle text-[6px]"></i> Real-time Sync
              </div>
              <div class="flex items-center gap-3 text-xs font-bold uppercase tracking-widest text-secondary">
                <i class="fas fa-circle text-[6px]"></i> Group Permissions
              </div>
            </div>
          </div>
          <div class="lg:w-2/3 w-full bg-white/5 backdrop-blur-md rounded-[2rem] p-6 border border-white/10">
            <apexchart
              type="donut"
              height="350"
              :options="userChartOptions"
              :series="userSeries"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
/**
 * Alpha PMS Dashboard - Comprehensive Property Management System
 * 
 * Based on: Alpha PMS – Dashboard & Analytics Specification
 * 
 * ============================================================================
 * PROPERTY MANAGEMENT SYSTEM (PMS) CORE FEATURES
 * ============================================================================
 * 
 * This dashboard provides comprehensive analytics for:
 * 
 * 1. PROPERTY RENT MANAGEMENT
 *    - Total rented units tracking
 *    - Monthly rent income calculation
 *    - Overdue rent monitoring (count & amount)
 *    - Paid rent tracking (this month)
 *    - Pending payment alerts
 *    - Detailed overdue rents table with tenant info
 * 
 * 2. PROPERTY SALE MANAGEMENT
 *    - Properties listed for sale
 *    - Properties sold (this month)
 *    - Total sales value tracking
 *    - Average sale price calculation
 *    - Properties under negotiation
 * 
 * 3. COWORKING SPACE MANAGEMENT
 *    - Total coworking spaces
 *    - Active bookings tracking
 *    - Monthly revenue from coworking
 *    - Occupancy rate monitoring
 *    - Overdue coworking payments
 *    - Desk utilization (hot desks, dedicated desks)
 *    - Meeting room utilization
 * 
 * 4. ZONES & UNITS ANALYTICS
 *    - Total zones/units count
 *    - Occupied zones tracking
 *    - Vacant zones monitoring
 *    - Units under maintenance
 *    - Overall occupancy rate with visual progress bar
 * 
 * 5. ROLES & USERS MANAGEMENT
 *    - Property owners count
 *    - Property managers count
 *    - Support staff count
 *    - Active tenants count
 *    - Real estate brokers count
 * 
 * 6. FINANCIAL SUMMARY
 *    - Total revenue (this month)
 *    - Total overdue amounts
 *    - Expected income (next 30 days)
 *    - Collection rate percentage
 * 
 * 7. OVERDUE RENTS DETAILED TRACKING
 *    - Tenant name and contact
 *    - Property and unit information
 *    - Overdue amount
 *    - Original due date
 *    - Days overdue calculation
 *    - Quick action buttons (send reminders)
 * 
 * ============================================================================
 * ROLE-BASED DASHBOARDS
 * ============================================================================
 * 
 * 1. SUPER ADMIN (Web Portal)
 *    - System-wide oversight, tenant management, billing, compliance
 *    - Metrics: Total properties, users, subscriptions, MRR, ARPU, occupancy, system health
 *    - Charts: Revenue trend, occupancy heatmap, user growth, coworking utilization
 *    - Actions: Approve registrations, manage pricing plans, resolve tickets, run reports
 *    - Full access to all PMS analytics
 * 
 * 2. SUPER STAFF (Web Portal)
 *    - Operational support, user assistance, system monitoring
 *    - Metrics: Support queue, pending KYC, system alerts, activity log
 *    - Actions: Respond to tickets, trigger invoices, reset passwords, view audit trails
 *    - Access to support-related PMS data
 * 
 * 3. LANDLORD / OWNER (Web Portal)
 *    - Property portfolio performance, financials, occupancy
 *    - Metrics: Portfolio summary, financials, tenant metrics, maintenance, lease expiry
 *    - Charts: Rent collection trend, vacancy by property, lease expiry calendar, expense breakdown
 *    - Actions: View P&L, adjust rent, approve maintenance, generate statements
 *    - Full access to owned properties PMS data
 * 
 * 4. OWNER STAFF / MANAGER (Web Portal)
 *    - Day-to-day property operations, maintenance, tenant communication
 *    - Metrics: Daily operations, tenant communication, property status, documents
 *    - Actions: Create work orders, log inspections, send notifications, update unit status
 *    - Access to managed properties PMS data
 * 
 * 5. COWORKING SPACE OWNER (Web Portal)
 *    - Coworking space utilization, memberships, desk/room bookings, events
 *    - Metrics: Utilization (desks, rooms), memberships, revenue, bookings, events
 *    - Charts: Utilization heatmap, membership growth, revenue mix, top members
 *    - Actions: Manage plans, block desks/rooms, approve bookings, schedule events
 *    - Full coworking analytics access
 * 
 * 6. BROKER (Mobile App - Future)
 *    - Lead management, property showings, client communication
 *    - Metrics: Leads, showings, deals, client activity
 *    - Actions: Log showings, send brochures, mark conversions, schedule follow-ups
 *    - Limited PMS data (properties for sale/rent)
 * 
 * 7. TENANT (Mobile App - Future)
 *    - Rent payment, maintenance requests, community updates, coworking bookings
 *    - Metrics: Rent & billing, maintenance, lease info, coworking, community
 *    - Actions: Pay rent, submit requests, book coworking, message landlord
 *    - Personal rent and payment data only
 * 
 * ============================================================================
 * SAMPLE DATA USAGE & SERVER INTEGRATION
 * ============================================================================
 * 
 * CURRENT STATE:
 * - All PMS-specific data is currently using sample data stored in `pmsData` object
 * - Sample overdue rents list provided for demonstration
 * - All metrics are placeholder values for UI/UX testing
 * 
 * SERVER INTEGRATION ENDPOINTS (To be implemented):
 * 
 * 1. Rent Analytics:
 *    GET /pms/rent_analytics
 *    Returns: { totalRentedUnits, monthlyRentIncome, overdueCount, overdueAmount, paidThisMonth, paidAmount, pendingCount }
 * 
 * 2. Sale Analytics:
 *    GET /pms/sale_analytics
 *    Returns: { totalForSale, soldThisMonth, totalSalesValue, avgSalePrice, underNegotiation }
 * 
 * 3. Coworking Analytics:
 *    GET /pms/coworking_analytics
 *    Returns: { totalSpaces, activeBookings, monthlyRevenue, occupancyRate, overduePayments }
 * 
 * 4. Zones Analytics:
 *    GET /pms/zones_analytics
 *    Returns: { occupied, vacant, maintenance, occupancyRate }
 * 
 * 5. Roles Analytics:
 *    GET /pms/roles_analytics
 *    Returns: { owners, managers, staff, brokers }
 * 
 * 6. Financial Summary:
 *    GET /pms/financial_summary
 *    Returns: { totalRevenue, totalOverdue, overdueCount, expectedIncome, collectionRate }
 * 
 * 7. Overdue Rents List:
 *    GET /pms/overdue_rents
 *    Returns: [{ tenant, property, unit, amount, dueDate, daysOverdue }]
 * 
 * IMPLEMENTATION NOTES:
 * - All API calls should include user_id for role-based filtering
 * - Super Admin sees all data across all properties
 * - Owners see only their properties
 * - Managers see only properties they manage
 * - Staff see only properties they're assigned to
 * - Tenants see only their own data
 * 
 * ============================================================================
 * CHARTS & VISUALIZATIONS
 * ============================================================================
 * 
 * EXISTING CHARTS:
 * - Revenue Trend (Line): Monthly platform revenue over 12 months
 * - User Distribution (Donut): User types breakdown
 * - Income/Expense Overview (Bar): Subscription, Rent, Sales, Workspace
 * - Pricing Plan Analytics (Line): Plan adoption and revenue
 * 
 * FUTURE CHARTS (To be added):
 * - Occupancy Heatmap: Geographic occupancy across cities/regions
 * - Rent Collection Trend (Line): Monthly collected vs. expected rent
 * - Vacancy by Property (Bar): Vacancy rate per property
 * - Lease Expiry Calendar (Heatmap): Upcoming lease end dates
 * - Expense Breakdown (Pie): Maintenance, marketing, PMS fees, utilities
 * - Utilization Heatmap (Hourly): Busy vs. slow hours for desks/rooms
 * - Membership Growth (Line): Active members over time
 * - Revenue Mix (Stacked Bar): Membership vs. room vs. event revenue
 * - Overdue Trend (Line): Overdue amounts over time
 * - Collection Rate Trend (Line): Monthly collection rate
 * 
 * ============================================================================
 * QUICK ACTIONS & FEATURES
 * ============================================================================
 * 
 * - Send rent reminders to overdue tenants
 * - Export overdue rents report (CSV/PDF)
 * - View detailed property P&L statements
 * - Generate tenant rent statements
 * - Schedule property inspections
 * - Approve/reject maintenance requests
 * - Manage coworking bookings
 * - View lease expiry alerts
 * - Track payment collection rates
 * - Monitor system health (Super Admin)
 * - Manage user roles and permissions
 */
export default {
  data() {
    return {
      is_superuser: false,
      userRole: '', // 'super_admin', 'super_staff', 'owner', 'manager', 'staff', 'coworking_owner', 'broker', 'tenant'
      userGroups: [],
      
      stats: {
        totalProperties: 0,
        totalZones: 0,
        totalSubscriptions: 0,
        totalTenants: 0,
      },

      // Sample data for features not yet implemented on server
      sampleData: {
        // Super Admin metrics
        totalUsers: 1247,
        activeSubscriptions: 342,
        totalCoworkingSpaces: 28,
        mrr: 45680,
        arpu: 133,
        paymentSuccess: 97.8,
        openTickets: 23,
        avgResponseTime: '2.4h',
        highPriorityTickets: 5,
        newSignupsWeek: 18,
        churnRate: 2.3,
        avgOccupancyRate: 87.5,
        
        // Landlord/Owner metrics
        occupiedUnits: 156,
        vacancyRate: 12.5,
        avgRentPerUnit: 1250,
        monthlyRentCollected: 187500,
        outstandingRent: 12300,
        latePayments: 8,
        ownerNetIncome: 165200,
        activeTenants: 156,
        leaseExpiring30: 12,
        leaseExpiring60: 23,
        leaseExpiring90: 31,
        tenantSatisfaction: 4.2,
        openMaintenanceRequests: 7,
        avgMaintenanceResolution: '3.2 days',
        
        // Coworking Owner metrics
        totalDesks: 120,
        totalMeetingRooms: 8,
        hotDeskUtilization: 78,
        dedicatedDeskUtilization: 92,
        meetingRoomUtilization: 65,
        activeMembers: 87,
        newMembersMonth: 12,
        churnedMembers: 3,
        membershipRevenue: 28900,
        meetingRoomRevenue: 4500,
        eventSpaceRevenue: 2100,
        upcomingEvents: 5,
        eventAttendanceRate: 82,
        
        // Broker metrics
        newLeadsToday: 5,
        hotLeads: 12,
        pendingFollowups: 8,
        scheduledShowingsToday: 3,
        scheduledShowingsWeek: 11,
        completedShowings: 47,
        noShows: 3,
        applicationsSubmitted: 15,
        leasesSigned: 8,
        commissionEarned: 12400,
        
        // Tenant metrics
        currentRentDue: 1250,
        rentDueDate: '2026-05-01',
        depositBalance: 2500,
        openMaintenanceReqs: 1,
        leaseEndDate: '2027-04-30',
        coworkingPassBalance: 10,
        upcomingBookings: 2,
      },

      // PMS-Specific Data (Sample - Replace with API calls)
      pmsData: {
        rent: {
          totalRentedUnits: 156,
          monthlyRentIncome: 187500,
          overdueCount: 23,
          overdueAmount: 34200,
          paidThisMonth: 133,
          paidAmount: 153300,
          pendingCount: 18,
        },
        sale: {
          totalForSale: 42,
          soldThisMonth: 7,
          totalSalesValue: 2450000,
          avgSalePrice: 350000,
          underNegotiation: 12,
        },
        coworking: {
          totalSpaces: 28,
          activeBookings: 87,
          monthlyRevenue: 35400,
          occupancyRate: 78,
          overduePayments: 5,
        },
        zones: {
          occupied: 156,
          vacant: 34,
          maintenance: 8,
          occupancyRate: 78.8,
        },
        roles: {
          owners: 45,
          managers: 23,
          staff: 67,
          brokers: 34,
        },
        financial: {
          totalRevenue: 258100,
          totalOverdue: 34200,
          overdueCount: 23,
          expectedIncome: 195000,
          collectionRate: 88.3,
        },
      },

      // Overdue Rents List (Sample Data)
      overdueRentsList: [
        { tenant: 'John Smith', property: 'Sunset Apartments', unit: 'A-101', amount: 1250, dueDate: '2026-03-15', daysOverdue: 35 },
        { tenant: 'Sarah Johnson', property: 'Downtown Plaza', unit: 'B-205', amount: 1800, dueDate: '2026-03-20', daysOverdue: 30 },
        { tenant: 'Michael Brown', property: 'Green Valley', unit: 'C-304', amount: 1500, dueDate: '2026-03-25', daysOverdue: 25 },
        { tenant: 'Emily Davis', property: 'Riverside Tower', unit: 'D-102', amount: 2100, dueDate: '2026-04-01', daysOverdue: 18 },
        { tenant: 'David Wilson', property: 'Sunset Apartments', unit: 'A-203', amount: 1350, dueDate: '2026-04-05', daysOverdue: 14 },
        { tenant: 'Lisa Anderson', property: 'Downtown Plaza', unit: 'B-401', amount: 1950, dueDate: '2026-04-08', daysOverdue: 11 },
        { tenant: 'James Martinez', property: 'Green Valley', unit: 'C-105', amount: 1600, dueDate: '2026-04-10', daysOverdue: 9 },
        { tenant: 'Jennifer Taylor', property: 'Riverside Tower', unit: 'D-308', amount: 1750, dueDate: '2026-04-12', daysOverdue: 7 },
      ],

      hugePageSize: 10000000000,
      start_date: "",
      end_date: "",

      revenue_start_date: new Date(new Date().getFullYear(), 0, 1).toISOString().split("T")[0],
      revenue_end_date: new Date().toISOString().split("T")[0],


      // Pricing report
      pricingSeries: [
        { name: "Users", type: "column", data: [] },
        { name: "Revenue ($)", type: "line", data: [] },
      ],
      pricingChartOptions: {
        chart: { height: 350, type: "line", toolbar: { show: false } },
        stroke: { width: [0, 4] },
        title: { text: "Plan Adoption and Revenue", align: "left" },
        xaxis: { categories: [] },
        yaxis: [
          { title: { text: "Users" } },
          { opposite: true, title: { text: "Revenue ($)" } },
        ],
        dataLabels: { enabled: true, enabledOnSeries: [1] },
        colors: ["#3b82f6", "#10b981"],
        legend: { position: "top" },
      },

      // Income report
      incomeData: {
        months: [],
        rent: [],
        sale: [],
        subscription: [],
        workspace: [],
      },


      incomeChartOptions: {
        chart: { id: "income-chart", stacked: false, toolbar: { show: false } },
        xaxis: { categories: [] },
        plotOptions: { bar: { horizontal: false, columnWidth: "55%" } },
        dataLabels: { enabled: false },
        legend: { position: "top" },
      },

      // User report
      userSeries: [],
      userChartOptions: {
        labels: [],
        legend: { position: "bottom" },
      },
    };
  },

  computed: {
    dashboardTitle() {
      const titles = {
        super_admin: 'Super Admin Dashboard',
        super_staff: 'Super Staff Dashboard',
        owner: 'Landlord / Owner Dashboard',
        manager: 'Owner Staff Dashboard',
        coworking_owner: 'Coworking Space Owner Dashboard',
        broker: 'Broker Dashboard',
        tenant: 'Tenant Dashboard',
      };
      return titles[this.userRole] || 'Dashboard';
    },

    dashboardSubtitle() {
      const subtitles = {
        super_admin: 'System-wide Oversight & Platform Health',
        super_staff: 'Operational Support & User Assistance',
        owner: 'Property Portfolio Performance & Financials',
        manager: 'Day-to-Day Property Operations',
        coworking_owner: 'Coworking Space Utilization & Memberships',
        broker: 'Lead Management & Property Showings',
        tenant: 'Rent Payment & Maintenance Requests',
      };
      return subtitles[this.userRole] || 'Overview and Analytics';
    },

    displayedMetrics() {
      // Return role-specific metrics based on user role
      if (this.userRole === 'super_admin') {
        return [
          { label: 'Total Properties', value: this.stats.totalProperties, icon: 'fas fa-building', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Total Users', value: this.sampleData.totalUsers, icon: 'fas fa-users', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
          { label: 'Active Subscriptions', value: this.sampleData.activeSubscriptions, icon: 'fas fa-crown', bgColor: 'bg-purple-50', iconColor: 'text-purple-500' },
          { label: 'Coworking Spaces', value: this.sampleData.totalCoworkingSpaces, icon: 'fas fa-briefcase', bgColor: 'bg-orange-50', iconColor: 'text-orange-500' },
          { label: 'MRR', value: `$${this.sampleData.mrr.toLocaleString()}`, subtitle: 'Monthly Recurring Revenue', icon: 'fas fa-dollar-sign', bgColor: 'bg-emerald-50', iconColor: 'text-emerald-500' },
          { label: 'Avg Occupancy', value: `${this.sampleData.avgOccupancyRate}%`, subtitle: 'All Properties', icon: 'fas fa-chart-pie', bgColor: 'bg-cyan-50', iconColor: 'text-cyan-500' },
          { label: 'New Signups', value: this.sampleData.newSignupsWeek, subtitle: 'This Week', icon: 'fas fa-user-plus', bgColor: 'bg-indigo-50', iconColor: 'text-indigo-500' },
          { label: 'Churn Rate', value: `${this.sampleData.churnRate}%`, subtitle: 'This Month', icon: 'fas fa-user-minus', bgColor: 'bg-red-50', iconColor: 'text-red-500' },
        ];
      } else if (this.userRole === 'super_staff') {
        return [
          { label: 'Open Tickets', value: this.sampleData.openTickets, icon: 'fas fa-ticket-alt', bgColor: 'bg-amber-50', iconColor: 'text-amber-500' },
          { label: 'High Priority', value: this.sampleData.highPriorityTickets, icon: 'fas fa-exclamation-triangle', bgColor: 'bg-red-50', iconColor: 'text-red-500' },
          { label: 'Avg Response Time', value: this.sampleData.avgResponseTime, icon: 'fas fa-clock', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Total Users', value: this.sampleData.totalUsers, icon: 'fas fa-users', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
        ];
      } else if (this.userRole === 'owner' || this.userRole === 'manager') {
        return [
          { label: 'Total Properties', value: this.stats.totalProperties, icon: 'fas fa-building', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Total Units', value: this.stats.totalZones, icon: 'fas fa-door-open', bgColor: 'bg-purple-50', iconColor: 'text-purple-500' },
          { label: 'Occupied Units', value: this.sampleData.occupiedUnits, icon: 'fas fa-home', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
          { label: 'Vacancy Rate', value: `${this.sampleData.vacancyRate}%`, icon: 'fas fa-door-closed', bgColor: 'bg-orange-50', iconColor: 'text-orange-500' },
          { label: 'Monthly Rent Collected', value: `$${this.sampleData.monthlyRentCollected.toLocaleString()}`, icon: 'fas fa-dollar-sign', bgColor: 'bg-emerald-50', iconColor: 'text-emerald-500' },
          { label: 'Outstanding Rent', value: `$${this.sampleData.outstandingRent.toLocaleString()}`, icon: 'fas fa-exclamation-circle', bgColor: 'bg-red-50', iconColor: 'text-red-500' },
          { label: 'Active Tenants', value: this.stats.totalTenants, icon: 'fas fa-users', bgColor: 'bg-cyan-50', iconColor: 'text-cyan-500' },
          { label: 'Open Maintenance', value: this.sampleData.openMaintenanceRequests, icon: 'fas fa-tools', bgColor: 'bg-amber-50', iconColor: 'text-amber-500' },
        ];
      } else if (this.userRole === 'coworking_owner') {
        return [
          { label: 'Total Desks', value: this.sampleData.totalDesks, icon: 'fas fa-chair', bgColor: 'bg-purple-50', iconColor: 'text-purple-500' },
          { label: 'Meeting Rooms', value: this.sampleData.totalMeetingRooms, icon: 'fas fa-door-open', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Active Members', value: this.sampleData.activeMembers, icon: 'fas fa-users', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
          { label: 'New Members', value: this.sampleData.newMembersMonth, subtitle: 'This Month', icon: 'fas fa-user-plus', bgColor: 'bg-emerald-50', iconColor: 'text-emerald-500' },
          { label: 'Membership Revenue', value: `$${this.sampleData.membershipRevenue.toLocaleString()}`, icon: 'fas fa-dollar-sign', bgColor: 'bg-cyan-50', iconColor: 'text-cyan-500' },
          { label: 'Meeting Room Revenue', value: `$${this.sampleData.meetingRoomRevenue.toLocaleString()}`, icon: 'fas fa-chart-line', bgColor: 'bg-indigo-50', iconColor: 'text-indigo-500' },
          { label: 'Upcoming Events', value: this.sampleData.upcomingEvents, icon: 'fas fa-calendar-alt', bgColor: 'bg-orange-50', iconColor: 'text-orange-500' },
          { label: 'Avg Utilization', value: `${Math.round((this.sampleData.hotDeskUtilization + this.sampleData.dedicatedDeskUtilization + this.sampleData.meetingRoomUtilization) / 3)}%`, icon: 'fas fa-chart-pie', bgColor: 'bg-pink-50', iconColor: 'text-pink-500' },
        ];
      } else if (this.userRole === 'broker') {
        return [
          { label: 'New Leads Today', value: this.sampleData.newLeadsToday, icon: 'fas fa-user-plus', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
          { label: 'Hot Leads', value: this.sampleData.hotLeads, icon: 'fas fa-fire', bgColor: 'bg-red-50', iconColor: 'text-red-500' },
          { label: 'Pending Follow-ups', value: this.sampleData.pendingFollowups, icon: 'fas fa-phone', bgColor: 'bg-amber-50', iconColor: 'text-amber-500' },
          { label: 'Showings Today', value: this.sampleData.scheduledShowingsToday, icon: 'fas fa-calendar-day', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Applications Submitted', value: this.sampleData.applicationsSubmitted, icon: 'fas fa-file-alt', bgColor: 'bg-purple-50', iconColor: 'text-purple-500' },
          { label: 'Leases Signed', value: this.sampleData.leasesSigned, icon: 'fas fa-file-signature', bgColor: 'bg-emerald-50', iconColor: 'text-emerald-500' },
          { label: 'Commission Earned', value: `$${this.sampleData.commissionEarned.toLocaleString()}`, subtitle: 'This Month', icon: 'fas fa-dollar-sign', bgColor: 'bg-cyan-50', iconColor: 'text-cyan-500' },
          { label: 'Completed Showings', value: this.sampleData.completedShowings, icon: 'fas fa-check-circle', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
        ];
      } else if (this.userRole === 'tenant') {
        return [
          { label: 'Current Rent Due', value: `$${this.sampleData.currentRentDue}`, subtitle: `Due: ${this.sampleData.rentDueDate}`, icon: 'fas fa-dollar-sign', bgColor: 'bg-blue-50', iconColor: 'text-blue-500' },
          { label: 'Deposit Balance', value: `$${this.sampleData.depositBalance}`, icon: 'fas fa-piggy-bank', bgColor: 'bg-green-50', iconColor: 'text-green-500' },
          { label: 'Open Maintenance', value: this.sampleData.openMaintenanceReqs, icon: 'fas fa-tools', bgColor: 'bg-amber-50', iconColor: 'text-amber-500' },
          { label: 'Lease Ends', value: this.sampleData.leaseEndDate, icon: 'fas fa-calendar-times', bgColor: 'bg-purple-50', iconColor: 'text-purple-500' },
        ];
      }
      
      // Default fallback
      return [
        { label: 'Properties', value: this.stats.totalProperties, icon: 'fas fa-building' },
        { label: 'Zones', value: this.stats.totalZones, icon: 'fas fa-map-marker-alt' },
        { label: 'Subscriptions', value: this.stats.totalSubscriptions, icon: 'fas fa-crown' },
        { label: 'Tenants', value: this.stats.totalTenants, icon: 'fas fa-users' },
      ];
    },
  },

  mounted() {

    const res=this.$apiGet("/property_report");

    console.log("res for property report",res);

    this.is_superuser = localStorage.getItem("is_superuser");
    
    // Determine user role from groups
    this.userGroups = JSON.parse(localStorage.getItem("groups") || "[]");
    this.determineUserRole();
    
    this.fetchAllData();
    if (this.is_superuser === "true") {
      this.refreshAllReports();
    }
  },

  methods: {
    determineUserRole() {
      // Determine primary role based on groups
      if (this.is_superuser === "true" || this.is_superuser === "1") {
        this.userRole = 'super_admin';
      } else if (this.userGroups.includes("super_staff")) {
        this.userRole = 'super_staff';
      } else if (this.userGroups.includes("owner")) {
        this.userRole = 'owner';
      } else if (this.userGroups.includes("manager")) {
        this.userRole = 'manager';
      } else if (this.userGroups.includes("staff")) {
        this.userRole = 'staff';
      } else if (this.userGroups.includes("coworking_owner")) {
        this.userRole = 'coworking_owner';
      } else if (this.userGroups.includes("broker")) {
        this.userRole = 'broker';
      } else if (this.userGroups.includes("tenant")) {
        this.userRole = 'tenant';
      } else {
        this.userRole = 'owner'; // default fallback
      }
      
      console.log('User Role Determined:', this.userRole);
    },
    async fetchAllData() {
      await Promise.all([
        this.fetchTotalProperties(),
        this.fetchTotalZones(),
        this.fetchTotalSubscriptions(),
        this.fetchTotalTenants(),
        this.fetchIncomeData(),
        this.fetchUserTypeReport(),
        this.fetchPMSData(), // New comprehensive PMS data fetch
      ]);
    },

    async fetchPMSData() {
      // This method will fetch comprehensive PMS data when server endpoints are ready
      // For now, using sample data defined in data()
      
      try {
        // TODO: Replace with actual API calls when endpoints are ready
        
        // Example structure for future API integration:
        // const rentData = await this.$apiGet("/pms/rent_analytics", { user_id: localStorage.getItem("userId") });
        // this.pmsData.rent = rentData;
        
        // const saleData = await this.$apiGet("/pms/sale_analytics", { user_id: localStorage.getItem("userId") });
        // this.pmsData.sale = saleData;
        
        // const coworkingData = await this.$apiGet("/pms/coworking_analytics", { user_id: localStorage.getItem("userId") });
        // this.pmsData.coworking = coworkingData;
        
        // const zonesData = await this.$apiGet("/pms/zones_analytics", { user_id: localStorage.getItem("userId") });
        // this.pmsData.zones = zonesData;
        
        // const rolesData = await this.$apiGet("/pms/roles_analytics", { user_id: localStorage.getItem("userId") });
        // this.pmsData.roles = rolesData;
        
        // const financialData = await this.$apiGet("/pms/financial_summary", { user_id: localStorage.getItem("userId") });
        // this.pmsData.financial = financialData;
        
        // const overdueList = await this.$apiGet("/pms/overdue_rents", { user_id: localStorage.getItem("userId") });
        // this.overdueRentsList = overdueList;
        
        console.log('PMS Data loaded (using sample data until server is ready)');
      } catch (error) {
        console.error("Failed to load PMS data:", error);
      }
    },

    async refreshAllReports() {
      await Promise.all([
        this.fetchSubscriptionReport(),
        this.fetchUserTypeReport(),
      ]);
    },
    chartOptions(title, months) {
      return {
        chart: { id: `${title}-chart` },
        xaxis: { categories: months },
        dataLabels: { enabled: false },
        grid: { strokeDashArray: 4 },
        title: {
          text: title,
          align: "center",
          style: { fontSize: "14px", color: "#374151" },
        },
        tooltip: { y: { formatter: (val) => val.toLocaleString() } },
        colors: ["#2563eb"],
      };
    },

    async fetchIncomeData() {
     // alert("hii")
     // if (!this.revenue_start_date || !this.revenue_end_date) return;


      const isSuperUser =
      localStorage.getItem("is_superuser") === "1" ||
      localStorage.getItem("is_superuser") === "true";

  const groups = JSON.parse(localStorage.getItem("groups") || "[]");
   const userId=localStorage.getItem('userId');
    let params = {};

    if (!isSuperUser) {
      if (groups.includes("manager")) {
        params = { 
        rent_id__property_id__property_zone_id__manager_id__id: userId,
        subscription_id__user_id__id:userId,
        space__zone__manager_id__id:userId,
        start_date: this.revenue_start_date,
        end_date: this.revenue_end_date,
        };
      } else if (groups.includes("owner")) {
        params = {
             rent_id__property_id__property_zone_id__owner_id__id: userId,
             property_zone_sale_id__property_zone_id__owner_id__id:userId,
             subscription_id__user_id__id:userId,
             space__zone__owner_id__id:userId,
             start_date: this.revenue_start_date,
             end_date: this.revenue_end_date,
         };
      } else if (groups.includes("staff")) {
        params={rent_id__property_id__property_zone_id__staff_id__id: userId,
        subscription_id__user_id__id:userId,
        space__zone__staff_id__id:userId,
        start_date: this.revenue_start_date,
        end_date: this.revenue_end_date,
        };
      } else if (groups.includes("super_staff")) {
        params = {
            start_date: this.revenue_start_date,
            end_date: this.revenue_end_date,
        };
      }
    }else{
        params = {
            start_date: this.revenue_start_date,
            end_date: this.revenue_end_date,
        };
      }
      console.log("Fetching with params for income:", params);

      try {
        const res = await this.$apiGet("/get_revenue_report", params);
        console.log("resin", res);
        this.incomeData = res;
      } catch (error) {
        console.error("Failed to load income data:", error);
        this.$root.$refs.toast?.showToast("Failed to load income data", "error");
      }
    },

    // 🟢 Fetch subscription report
async fetchSubscriptionReport() {
  try {
    let url = "/get_subscription_report";

    // Add date filters if selected
    if (this.start_date && this.end_date) {
      url += `?start_date=${this.start_date}&end_date=${this.end_date}`;
    }

    const res = await this.$apiGet(url);
    const data = res.data || res;

    // Extract only plans and revenue
    const { plans = [], revenue = [] } = data;

    // Update chart options
    this.pricingChartOptions = {
      chart: {
        type: "bar",
        toolbar: { show: false },
      },
      title: {
        text: "PLANS AND THEIR REVENUE",
        align: "center",
        style: {
          fontSize: "18px",
          fontWeight: "bold",
          color: "#4B0082", // deep purple
        },
      },
      xaxis: {
        categories: plans,
        title: {
          text: "PLAN TYPE",
          style: { color: "#333", fontWeight: "600" },
        },
      },
      yaxis: {
        title: {
          text: "REVENUE ($)",
          style: { color: "#333", fontWeight: "600" },
        },
      },
      colors: ["#7C3AED"], // violet accent
      dataLabels: {
        enabled: true,
        style: {
          colors: ["#fff"],
          fontWeight: "bold",
        },
        background: {
          enabled: true,
          foreColor: "#000",
          borderRadius: 6,
          opacity: 0.9,
        },
      },
      plotOptions: {
        bar: {
          borderRadius: 8,
          distributed: true,
          columnWidth: "45%",
        },
      },
      tooltip: {
        theme: "light",
        y: {
          formatter: (val) => `$${val.toLocaleString()}`,
        },
      },
    };

    // Update data series
    this.pricingSeries = [
      {
        name: "Revenue ($)",
        data: revenue,
      },
    ];

    // Success message
    this.$root.$refs.toast?.showToast(
      "Revenue report loaded successfully",
      "success"
    );
  } catch (err) {
    console.error(err);
    this.$root.$refs.toast?.showToast(
      "Failed to load revenue report",
      "error"
    );
  }
},


    // 🟢 Fetch user type report (replaces fetchAndGroupUsers)
    async fetchUserTypeReport() {
      try {
        const params = {
          user_id: localStorage.getItem("userId"),
        };
        let url = "/get_user_type_report";
        if (this.start_date && this.end_date) {
          url += `?start_date=${this.start_date}&end_date=${this.end_date}`;
        }

        const res = await this.$apiGet(url, params);
        const data = res.data || res;
        const { groups = [], counts = [] } = data;

        this.userChartOptions = {
          ...this.userChartOptions,
          labels: groups,
        };
        this.userSeries = counts;

        if (this.start_date && this.end_date)
          this.$root.$refs.toast?.showToast(
            "User type report loaded",
            "success"
          );
      } catch (err) {
        console.error(err);
        this.$root.$refs.toast?.showToast(
          "Failed to load user type report",
          "error"
        );
      }
    },

    // ---- Stats fetchers ----
    async fetchTotalProperties() {
      try {
        //  const res = await this.$apiGet("/get_properties", { page_size: this.hugePageSize });
        const pageSize = this.hugePageSize;
        const params = {};
        const pageUrl = "/get_properties?page=1&page_size=1000000000";
        const res = await this.$getProperties(pageUrl, params);
        console.log("res properties", res);
        //alert("hi");
        this.stats.totalProperties = res.properties?.length ?? 0;
      } catch {}
    },

    async fetchTotalZones() {
      try {
        const pageUrl = "/get_property_zones?page=1&page_size=10000000000";
        const params = {};
        const res = await this.$getZones(pageUrl, params);
        console.log("res zones", res);
        this.stats.totalZones = res.zones?.length ?? 0;
      } catch {}
    },

    async fetchTotalSubscriptions() {
      try {
        let params = {
          user_id__id: localStorage.getItem("userId"),
          page: 1,
          page_size: 10000000000,
        };

        if (localStorage.getItem("is_superuser") == "true") {
          params = {
            page: 1,
            page_size: 10000000000,
          };
        }
        const pageUrl = `/get_subscription`;
        const res = await this.$apiGet(pageUrl, params);
        console.log("res", res);
        console.log("res for subscriptions", res);
        this.stats.totalSubscriptions = res.count;
      } catch {}
    },

    async fetchTotalTenants() {
      try {
        const pageUrl = `/get_rents`;
        const params = {
          page_size: 10000000000,
        };
        const res = await this.$getTenants(pageUrl, params);
        console.log("res for tenants", res);

        this.stats.totalTenants = res.tenants.length ?? 0;
      } catch {}
    },

  },
};
</script>

<style scoped>
.bg-white:hover {
  transform: translateY(-2px);
  transition: all 0.2s ease;
}
</style>
