<template>
  <button
    type="button"
    :class="{ 'bg-slate-800 text-white hover:bg-slate-800': is_selected }"
    class="flex gap-3 items-center py-1.5 px-2 text-sm font-medium hover:bg-slate-100 rounded-md w-full cursor-pointer"
    :aria-pressed="is_selected"
    :aria-label="title"
  >
    <div
      :class="is_selected ? 'bg-gray-800' : 'bg-slate-100'"
      class="w-8 h-8 rounded-full flex items-center justify-center flex-shrink-0"
      aria-hidden="true"
    >
      <img
        v-if="img"
        :src="img"
        :class="class_icon"
        alt=""
      />
      <component
        v-else
        :is="icon"
        :class="class_icon"
      />
    </div>
    <span class="flex-grow text-left">
      {{ title }}
    </span>
    <slot />
  </button>
</template>
<script setup lang="ts">
import Badge from '@/components/Badge.vue'

import type { Component } from 'vue'

const $props = withDefaults(
  defineProps<{
    /** Icon vue */
    icon: Component
    /**hình ảnh thay thế cho icon */
    img?: string
    /** Tiêu đề */
    title: string
    /** Đã chọn */
    is_selected?: boolean
    /** Icon class */
    class_icon?: string
  }>(),
  {
    class_icon: 'size-5',
  }
)
</script>
