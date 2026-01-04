<template>
  <div class="rounded-lg py-3 px-2 md:px-4 bg-white flex gap-3">
    <div
      :class="class_icon"
      class="flex-shrink-0"
    >
      <slot name="icon" />
    </div>
    <div
      :class="class_content"
      class="flex flex-col gap-3 w-full min-w-0"
    >
      <div
        :class="class_header_content"
        class="flex flex-col md:flex-row justify-between gap-2"
      >
        <div
          :class="class_title"
          class="text-sm font-medium"
        >
          <slot name="title" />
        </div>
        <slot name="action" />
      </div>
      <slot name="item" />
    </div>
  </div>
</template>
<script setup lang="ts">
const $props = withDefaults(
  defineProps<{
    /**class của tiêu đề */
    class_title?: string
    /** class của content */
    class_content?: string
    /** class của header nội dung */
    class_header_content?: string
    /** class của icon */
    class_icon?: string
  }>(),
  {}
)
</script>
