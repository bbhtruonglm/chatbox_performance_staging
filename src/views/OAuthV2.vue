<template>
  <section
    class="size-full bg-gradient-secondary flex items-center justify-center p-4 relative"
  >
    <div
      class="bg-white w-full max-w-[478px] shadow-xl rounded-xl flex flex-col gap-6 p-6 max-h-full overflow-y-scroll"
    >
      <div
        :style="{
          backgroundImage: `url(${commonStore.partner?.logo?.full})`,
        }"
        class="h-7 w-full bg-contain bg-no-repeat bg-left flex-shrink-0"
      />
      <RouterView />
      <slot />
      <div class="flex justify-between">
        <p class="flex items-center gap-3 text-xs">
          <a
            :href="PRIVACY"
            target="_blank"
            class="no-underline text-slate-500"
          >
            {{ $t('Chính sách') }}
          </a>
          <a
            :href="TERM"
            target="_blank"
            class="no-underline text-slate-500"
          >
            {{ $t('Điều khoản') }}
          </a>
        </p>
        <Language />
      </div>
    </div>
  </section>
</template>
<script setup lang="ts">
import { useCommonStore } from '@/stores'

import Language from '@/components/Language.vue'
import { composableService } from '@/views/OAuth/service'

const { TERM, PRIVACY } = composableService()
const commonStore = useCommonStore()
</script>
<style lang="scss" scoped>
.bg-gradient-secondary {
  background: linear-gradient(180deg, hsla(0, 0%, 100%, 0) 50%, #fff, #fff),
    linear-gradient(90deg, #fdefe3, #f3f4f3 50%, #dce5ff);
}
</style>
