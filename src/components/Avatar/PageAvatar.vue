<template>
  <div class="overflow-hidden bg-slate-100 rounded-oval">
    <img
      v-if="page_info?.type === 'ZALO_PERSONAL'"
      loading="lazy"
      :src="loadImageUrl()"
      class="w-full h-full object-contain"
    />
    <img
      loading="lazy"
      v-else-if="page_info?.type === 'FB_INSTAGRAM'"
      :src="loadImageUrl()"
      @error="onImgError"
      class="w-full h-full"
    />
    <img
      loading="lazy"
      v-else-if="page_info?.type === 'TIKTOK'"
      :src="loadImageUrl()"
      @error="onImgError"
      class="w-full h-full"
    />
    <img
      v-else-if="page_info?.avatar"
      loading="lazy"
      :src="page_info?.avatar"
      @error="onImgError"
      class="w-full h-full object-contain"
    />
    <img
      loading="lazy"
      v-else-if="page_info?.type === 'FB_MESS'"
      :src="loadImageUrl()"
      @error="onImgError"
      class="w-full h-full"
    />

    <template v-else-if="page_info?.type === 'WEBSITE'">
      <img
        v-if="page_info?.avatar"
        loading="lazy"
        :src="page_info?.avatar"
        @error="onImgError"
        class="w-full h-full"
      />
      <WebIcon
        v-else
        class="w-full h-full"
      />
    </template>

    <img
      loading="lazy"
      v-else-if="page_info?.type === 'ZALO_OA'"
      :src="page_info?.avatar || zaloSvg"
      class="w-full h-full"
      @error="onImgError"
    />
  </div>
</template>

<script setup lang="ts">
import { SingletonCdn } from '@/utils/helper/Cdn'

import zaloSvg from '@/assets/icons/zalo.svg'
import WebIcon from '@/components/Icons/Web.vue'
import pageFallbackSvg from '@/assets/imgs/retion.svg'

import type { IPage } from '@/service/interface/app/page'

const $cdn = SingletonCdn.getInst()

const $props = withDefaults(
  defineProps<{
    /**dữ liệu của trang */
    page_info?: IPage
  }>(),
  {}
)
console.log($props.page_info, 'page info')

/**xử lý khi ảnh lỗi */
function onImgError(e: Event) {
  /** element img */
  const TARGET = e.target as HTMLImageElement
  /** số lần thử lại */
  const RETRY_COUNT = parseInt(TARGET.dataset.retry || '0')

  // nếu chưa retry lần nào -> thử lấy avatar gốc từ props
  if (RETRY_COUNT === 0) {
    TARGET.dataset.retry = '1'
    /** avatar gốc */
    const AVATAR = $props.page_info?.avatar

    // nếu có avatar thì thử load
    if (AVATAR) {
      TARGET.src = AVATAR
      return
    }
  }

  // nếu đã retry 1 lần hoặc chưa quá giới hạn -> fallback sang ảnh mặc định
  if (RETRY_COUNT <= 1) {
    TARGET.dataset.retry = '2'
    // Hiện tại đang fallback chung cho tất cả các loại trang nhưng về sau phải sửa lại
    TARGET.src = pageFallbackSvg
    return
  }

  // nếu vẫn lỗi thì bỏ qua
  TARGET.onerror = null
}

/**tạo url ảnh */
function loadImageUrl(page_id?: string) {
  /**id của trang */
  const PAGE_ID = page_id || $props.page_info?.fb_page_id

  console.log(PAGE_ID, 'page id')
  switch ($props.page_info?.type) {
    case 'ZALO_PERSONAL':
      return $cdn.zlpPageAvt(PAGE_ID)
    case 'FB_MESS':
      return $cdn.fbPageAvt(PAGE_ID)
    case 'FB_INSTAGRAM':
      return $cdn.igPageAvt(PAGE_ID)
    default:
      return $props.page_info?.avatar
  }
}
</script>
