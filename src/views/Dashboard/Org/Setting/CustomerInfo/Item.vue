<template>
  <div class="flex">
    <div class="w-28 flex-shrink-0">
      {{ title }}
    </div>
    <template v-if="$slots.default">
      <slot />
    </template>
    <template v-else>
      <div
        v-if="!is_edit && !model"
        class="text-slate-400"
      >
        {{ $t('v1.view.main.dashboard.org.setting.customer_info.empty') }}
      </div>
      <div class="truncate min-w-0 flex-grow">
        <input
          v-if="is_edit"
          v-model="model"
          type="text"
          class="focus:outline-none w-full border-b"
          :placeholder="title"
        />
        <div v-else>
          {{ model }}
        </div>
      </div>
    </template>
  </div>
</template>
<script setup lang="ts">
const $props = withDefaults(
  defineProps<{
    /**tiêu đề */
    title: string
    /**có cho phép chỉnh sửa không */
    is_edit: boolean
  }>(),
  {}
)

/** Khai báo v-model */
const model = defineModel()
</script>
