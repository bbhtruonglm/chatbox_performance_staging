<template>
  <div class="relative">
    <!-- Icon tìm kiếm -->
    <img
      class="absolute top-1/2 -translate-y-1/2 left-3 w-5 h-5 pointer-events-none"
      src="@/assets/icons/search.svg"
      alt="Search icon"
      aria-hidden="true"
    />
    <label
      for="dashboard-search"
      class="sr-only"
      >{{ placeholder }}
    </label>
    <!-- Input tìm kiếm -->
    <input
      id="dashboard-search"
      v-model="temp_input"
      type="search"
      :placeholder="placeholder"
      :aria-label="placeholder"
      class="w-full h-full rounded-full border border-gray-300 py-2 pl-10 pr-4 text-sm transition-all duration-200 placeholder:text-gray-400 hover:border-gray-400 focus:outline-none focus:border-orange-500"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

/** Khai báo v-model */
const model = defineModel<string>()

/** Giá trị input tạm thời (chưa debounce) */
const temp_input = ref<string>('')

// Phần này để xử lý logic riêng cho search page đang dùng debounce tự viết, cần update lại

/** Timer cho debounce */
let debounce_timer: ReturnType<typeof setTimeout> | null = null

/** Delay cho debounce (ms) */
const DEBOUNCE_DELAY = 300

/** Watch giá trị input tạm thời và cập nhật model sau khi debounce */
watch(temp_input, new_value => {
  /** Xóa timer cũ nếu có */
  if (debounce_timer) clearTimeout(debounce_timer)

  /** Tạo timer mới */
  debounce_timer = setTimeout(() => (model.value = new_value), DEBOUNCE_DELAY)
})

/** Watch model để đồng bộ giá trị ban đầu */
watch(
  () => model.value,
  new_value => {
    /** Đồng bộ giá trị từ model về temp_input nếu khác nhau */
    if (new_value !== temp_input.value) temp_input.value = new_value || ''
  },
  { immediate: true }
)

withDefaults(
  defineProps<{
    /**placeholder của input */
    placeholder?: string
  }>(),
  {}
)
</script>
