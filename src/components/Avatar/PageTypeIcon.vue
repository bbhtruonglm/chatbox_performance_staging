<template>
  <FacebookIcon v-if="page_type === 'FB_MESS'" />

  <img
    loading="lazy"
    v-else-if="page_type === 'FB_INSTAGRAM'"
    src="@/assets/icons/instagram-color.svg"
    alt="Instagram"
  />

  <WebIcon v-else-if="page_type === 'WEBSITE'" />

  <img
    loading="lazy"
    v-else-if="page_type === 'FB_WHATSAPP'"
    src="@/assets/icons/whatsapp.svg"
    alt="Whatsapp"
  />

  <ZaloIcon
    v-else-if="page_type === 'ZALO_OA'"
    alt="Zalo OA"
  />
  <ZaloIcon
    v-else-if="page_type === 'ZALO_PERSONAL'"
    alt="Zalo Personal"
  />
  <TiktokIcon
    v-else-if="page_type === 'TIKTOK'"
    alt="Tiktok"
  />

  <div v-else />
</template>

<script setup lang="ts">
import WebIcon from '@/components/Icons/Web.vue'
import FacebookIcon from '@/components/Icons/Facebook.vue'
import ZaloIcon from '@/components/Icons/Zalo.vue'
import TiktokIcon from '@/components/Icons/Tiktok.vue'

import type { PageType } from '@/service/interface/app/page'

withDefaults(
  defineProps<{
    page_type?: PageType
  }>(),
  {}
)
</script>
