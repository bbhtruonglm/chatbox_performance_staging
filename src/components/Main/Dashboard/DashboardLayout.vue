<template>
  <div class="h-full w-full flex relative gap-3">
    <div
      class="h-full flex-shrink-0 w-56 bg-white rounded-md p-2 gap-1 flex-col overflow-y-auto hidden md:flex"
    >
      <slot name="menu" />
    </div>
    <div
      :class="class_content"
      class="w-full h-full min-w-0"
    >
      <slot name="content" />
    </div>
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
