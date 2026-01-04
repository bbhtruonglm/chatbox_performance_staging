<template>
  <div class="h-full w-full flex relative gap-3">
    <nav
      class="h-full flex-shrink-0 w-56 bg-white rounded-md p-2 gap-1 flex-col overflow-y-auto hidden md:flex"
      aria-label="Menu điều hướng"
    >
      <slot name="menu" />
    </nav>
    <main
      :class="class_content"
      class="w-full h-full min-w-0"
      role="main"
    >
      <slot name="content" />
    </main>
  </div>
</template>
<script setup lang="ts">
const $props = withDefaults(
  defineProps<{
    /**class của nội dung */
    class_content?: string
  }>(),
  {}
)
</script>
