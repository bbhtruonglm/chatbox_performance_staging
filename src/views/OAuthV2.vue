<template>
  <main
    class="size-full bg-gradient-secondary flex items-center justify-center p-4 relative"
    role="main"
  >
    <section
      class="bg-white w-full max-w-[478px] shadow-xl rounded-xl flex flex-col gap-6 p-6 max-h-full overflow-y-scroll"
      aria-label="Đăng nhập"
    >
      <img
        v-if="commonStore.partner?.logo?.full"
        :src="commonStore.partner?.logo?.full"
        :alt="commonStore.partner?.name || 'Logo'"
        class="h-7 w-auto object-contain object-left flex-shrink-0"
      />
      <RouterView />
      <slot />
      <nav
        class="flex justify-between"
        aria-label="Footer navigation"
      >
        <p class="flex items-center gap-3 text-xs">
          <a
            :href="PRIVACY"
            target="_blank"
            rel="noopener noreferrer"
            class="no-underline text-slate-500"
            :aria-label="$t('Chính sách bảo mật')"
          >
            {{ $t('Chính sách') }}
          </a>
          <a
            :href="TERM"
            target="_blank"
            rel="noopener noreferrer"
            class="no-underline text-slate-500"
            :aria-label="$t('Điều khoản sử dụng')"
          >
            {{ $t('Điều khoản') }}
          </a>
        </p>
        <Language />
      </nav>
    </section>
  </main>
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
