<template>
  <div class="relative flex h-full flex-col" v-if="$users.fetched">
    <div class="h-full flex-1">
      <div class="flex h-full">
        <div
          class="relative block min-h-0 flex-shrink-0 overflow-hidden hover:overflow-auto"
        >
          <slot name="sidebar" />
          <AppSidebar />
        </div>
        <div class="w-full overflow-auto" id="scrollContainer">
          <div
            v-if="$readOnlyMode"
            class="right-0 top-0 mb-3 bg-gray-100 py-3 text-sm text-gray-600"
          >
            <div class="mx-auto px-10">
              This site is running in read-only mode. Full functionality will be
              restored soon.
            </div>
          </div>
          <div>
            <slot />
          </div>
        </div>
      </div>
    </div>
    <CommandPalette />
  </div>
</template>
<script setup>
import AppSidebar from './AppSidebar.vue'
import CommandPalette from './CommandPalette/CommandPalette.vue'
</script>
