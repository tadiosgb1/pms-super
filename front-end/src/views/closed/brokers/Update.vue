<template>
  <Teleport to="body">
    <div v-if="visible" class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4">
      <div class="bg-white w-full max-w-md rounded-xl shadow-xl flex flex-col">

        <div class="flex items-center justify-between px-5 py-4 border-b border-gray-100">
          <h2 class="text-base font-semibold text-gray-800">Update Broker</h2>
          <button @click="$emit('close')" class="text-gray-400 hover:text-gray-600 text-xl leading-none">&times;</button>
        </div>

        <form id="updateBrokerForm" @submit.prevent="updateBroker" class="p-5 space-y-4">
          <div>
            <label class="form-label">License Number</label>
            <input v-model="form.license_number" type="text" placeholder="BR-XXXXXX" class="form-input" />
          </div>
          <div>
            <label class="form-label">Commission Rate (%)</label>
            <input v-model="form.commission_rate" type="number" placeholder="e.g. 5.0" class="form-input" />
          </div>
          <div>
            <label class="form-label">Wallet</label>
            <input v-model="form.wallet" type="text" placeholder="Wallet address" class="form-input" />
          </div>
        </form>

        <div class="flex justify-end gap-3 px-5 py-4 border-t border-gray-100 bg-gray-50 rounded-b-xl">
          <button type="button" @click="$emit('close')" class="btn-cancel">Cancel</button>
          <button form="updateBrokerForm" type="submit" :disabled="loading" class="btn-primary">
            {{ loading ? 'Updating...' : 'Update' }}
          </button>
        </div>

      </div>
    </div>
  </Teleport>
</template>

<script>
export default {
  name: "UpdateBroker",
  props: {
    visible: { type: Boolean, default: false },
    broker: { type: Object, default: null },
  },
  data() {
    return { form: { license_number: "", commission_rate: "", wallet: "" }, loading: false };
  },
  watch: {
    visible(val) { if (val && this.broker) this.fillForm(this.broker); },
    broker: { handler(v) { if (v && this.visible) this.fillForm(v); }, deep: true },
  },
  methods: {
    fillForm(b) {
      this.form = { license_number: b.license_number || "", commission_rate: b.commission_rate || "", wallet: b.wallet || "" };
    },
    async updateBroker() {
      this.loading = true;
      try {
        await this.$apiPatch("/update_broker_profile", this.broker.id, { ...this.form });
        this.$root.$refs.toast.showToast("Broker updated successfully", "success");
        this.$emit("success");
        this.$emit("close");
      } catch (e) {
        this.$root.$refs.toast.showToast(e.message || "Failed to update", "error");
        this.$emit("close");
      } finally { this.loading = false; }
    },
  },
};
</script>

<style scoped>
.form-label { @apply block text-xs font-medium text-gray-600 mb-1; }
.form-input  { @apply w-full border border-gray-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-primary/30; }
.btn-cancel  { @apply px-4 py-2 text-sm text-gray-500 border border-gray-200 rounded-lg hover:bg-gray-100 transition; }
.btn-primary { @apply px-5 py-2 text-sm font-semibold text-white bg-primary rounded-lg hover:bg-primary/90 transition disabled:opacity-60; }
</style>
