<template>
  <div class="p-2 flex flex-col gap-3 flex-grow justify-center items-center">
    <div class="flex items-center gap-5">
      <!-- <div class="bg-orange-600 rounded-lg py-1 px-2 m-auto flex-shrink-0">
        <BotXIcon class="text-white w-6 h-8" />
      </div> -->
      <img
        :src="commonStore.partner?.logo?.icon"
        class="w-10 h-10 flex-shrink-0"
      />
      <ReloadIcon class="w-5 h-5 text-slate-500 flex-shrink-0" />
      <component
        :is="icon"
        class="w-10 h-10 flex-shrink-0"
      />
    </div>
    <div
      :class="class_guild"
      class="text-center"
    >
      <div class="font-semibold">
        {{ guild }}
      </div>
      <div class="text-sm text-slate-700 relative">
        {{ description }}
        <slot name="after-description" />
      </div>
    </div>
    <slot name="button" />
  </div>
</template>
<script setup lang="ts">
import BotXIcon from '@/components/Icons/BotX.vue'
import ReloadIcon from '@/components/Icons/Reload.vue'
import { useCommonStore } from '@/stores'

import type { Component } from 'vue'

const $props = withDefaults(
  defineProps<{
    /**icon của nền tảng */
    icon: Component
    /**hướng dẫn */
    guild: string
    /**chú thích */
    description: string
    /**class cho guild */
    class_guild?: string
  }>(),
  {}
)

const commonStore = useCommonStore()
</script>
