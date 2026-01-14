<template>
  <div
    ref="select_lang_ref"
    class="relative z-10"
  >
    <div
      v-if="is_toggle_select_lang"
      class="absolute bottom-[30px] right-0 shadow-lg rounded-lg w-max flex flex-col gap-1 p-1 bg-white"
    >
      <button
        @click="changeLang(key)"
        v-for="(lang, key) of SELECT_LANG"
        class="flex py-1.5 px-2 items-center gap-3"
      >
        <img
          :src="lang.img"
          class="rounded-full w-5 h-5 object-cover"
        />
        <div class="text-sm font-medium">{{ lang.title }}</div>
      </button>
    </div>
    <button
      @click="is_toggle_select_lang = !is_toggle_select_lang"
      class="rounded flex items-center gap-1"
    >
      <img
        :src="SELECT_LANG?.[current_locale].img"
        class="rounded-full w-3 h-3 object-cover"
      />
      <div class="text-xs">
        {{ SELECT_LANG?.[current_locale]?.title }}
      </div>
      <ChevronDownIcon class="w-3 h-3" />
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { LocaleSingleton } from '@/utils/helper/Locale'

import vnSvg from '@/assets/icons/vn.svg'
import enSvg from '@/assets/icons/en.svg'
import thSvg from '@/assets/icons/th.svg'

import { ChevronDownIcon } from '@heroicons/vue/24/solid'

import type { ComponentRef } from '@/service/interface/vue'

const { locale } = useI18n()

/**dữ liệu ngôn ngữ */
interface ILangItem {
  /*cờ ngôn ngữ */
  img: string
  /*tiêu đề ngôn ngữ */
  title: string
}

const vn: ILangItem = { img: vnSvg, title: 'Tiếng Việt' }
const en: ILangItem = { img: enSvg, title: 'English' }
const th: ILangItem = { img: thSvg, title: 'ไทย' }

/**
 * một ngôn ngữ có nhiều kiểu viết tắt, nên phải mapping lại
 */
const MAPPING_LANG: { [index: string]: string } = {
  vn: 'vn',
  vi: 'vn',
  en: 'en',
  us: 'en',
  th: 'th',
}

const SELECT_LANG: Record<string, ILangItem> = { vn, en, th }

const select_lang_ref = ref<ComponentRef>()

const is_toggle_select_lang = ref(false)

const current_locale = computed(() => MAPPING_LANG[locale.value])

// lắng nghe sự kiện click ra ngoài
onMounted(() => document.body.addEventListener('click', clickOutSide))
// huỷ lắng nghe sự kiện khi un mount
onUnmounted(() => document.body.removeEventListener('click', clickOutSide))

/**ẩn option lựa cho khi click outside */
function clickOutSide($event: MouseEvent) {
  if (
    select_lang_ref.value == $event.target ||
    select_lang_ref.value.contains($event.target)
  )
    return

  is_toggle_select_lang.value = false
}
/**lưu lại lang mới vào local, và reload lại trang */
function changeLang(key: string) {
  LocaleSingleton.getInst().set(key)

  window.location.reload()
}
</script>
