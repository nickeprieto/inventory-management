<template>
  <svg
    :width="size"
    :height="size"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-width="1.75"
    stroke-linecap="round"
    stroke-linejoin="round"
    aria-hidden="true"
    focusable="false"
    class="nav-icon"
  >
    <!-- Overview / dashboard -->
    <template v-if="name === 'overview'">
      <rect x="3" y="3" width="8" height="9" rx="1.5" />
      <rect x="13" y="3" width="8" height="5" rx="1.5" />
      <rect x="13" y="11" width="8" height="10" rx="1.5" />
      <rect x="3" y="15" width="8" height="6" rx="1.5" />
    </template>

    <!-- Inventory / boxes -->
    <template v-else-if="name === 'inventory'">
      <path d="M21 8V16.5a1.5 1.5 0 0 1-.8 1.32l-7.2 3.9a2 2 0 0 1-1.9 0l-7.2-3.9A1.5 1.5 0 0 1 3 16.5V8" />
      <path d="m3.3 7 8 4.3a1.5 1.5 0 0 0 1.4 0l8-4.3" />
      <path d="M12 21.5V11.5" />
      <path d="M3.3 7 11.1 2.8a2 2 0 0 1 1.8 0L20.7 7" />
    </template>

    <!-- Orders / clipboard list -->
    <template v-else-if="name === 'orders'">
      <path d="M9 4h6a1 1 0 0 1 1 1v1a1 1 0 0 1-1 1H9a1 1 0 0 1-1-1V5a1 1 0 0 1 1-1Z" />
      <path d="M8 5H6a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2" />
      <path d="M8 12h8" />
      <path d="M8 16h5" />
    </template>

    <!-- Finance / trending currency -->
    <template v-else-if="name === 'finance'">
      <path d="M3 17 9 11l4 4 8-8" />
      <path d="M15 4h6v6" />
    </template>

    <!-- Demand Forecast / chart line + dots -->
    <template v-else-if="name === 'demand'">
      <path d="M4 4v15a1 1 0 0 0 1 1h15" />
      <path d="m7 14 3-4 3 3 5-7" />
      <circle cx="7" cy="14" r="0.6" />
      <circle cx="10" cy="10" r="0.6" />
      <circle cx="13" cy="13" r="0.6" />
      <circle cx="18" cy="6" r="0.6" />
    </template>

    <!-- Reports / document -->
    <template v-else-if="name === 'reports'">
      <path d="M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8Z" />
      <path d="M14 3v5h5" />
      <path d="M9 13h6" />
      <path d="M9 17h6" />
    </template>

    <!-- Collapse chevron (points left; rotate via CSS for expand) -->
    <template v-else-if="name === 'collapse'">
      <path d="m15 6-6 6 6 6" />
    </template>
  </svg>
</template>

<script setup>
defineProps({
  name: { type: String, required: true },
  size: { type: [Number, String], default: 18 }
})
</script>

<style scoped>
.nav-icon {
  flex-shrink: 0;
  display: block;
}
</style>
