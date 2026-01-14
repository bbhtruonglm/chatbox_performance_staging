<template>
  <MenuTitle :title="$t('v1.view.main.dashboard.chat.filter.interact.title')" />
  <FilterCheckbox
    v-model="conversationStore.option_filter_page_data.display_style"
    true_value="INBOX"
    :false_value="undefined"
    :icon="ChatDotIcon"
    :title="$t('v1.view.main.dashboard.chat.filter.interact.message')"
  />
  <FilterCheckbox
    v-model="conversationStore.option_filter_page_data.display_style"
    true_value="COMMENT"
    :false_value="undefined"
    :icon="NewSpaperIcon"
    :title="$t('v1.view.main.dashboard.chat.filter.interact.comment')"
  />
  <template v-if="is_show_zalo">
    <FilterCheckbox
      v-model="conversationStore.option_filter_page_data.display_style"
      true_value="FRIEND"
      :false_value="undefined"
      :icon="UsersIcon"
      :title="$t('v1.view.main.dashboard.chat.filter.interact.friend')"
    />
    <FilterCheckbox
      v-model="conversationStore.option_filter_page_data.display_style"
      true_value="GROUP"
      :false_value="undefined"
      :icon="UserGroupIcon"
      :title="$t('v1.view.main.dashboard.chat.filter.interact.group')"
    />
  </template>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useConversationStore, usePageStore } from '@/stores'

import FilterCheckbox from '@/views/ChatWarper/Menu/FilterModal/FilterCheckbox.vue'
import MenuTitle from '@/components/Main/Dashboard/MenuTitle.vue'

import ChatDotIcon from '@/components/Icons/ChatDot.vue'
import NewSpaperIcon from '@/components/Icons/NewSpaper.vue'
import { UserGroupIcon, UsersIcon } from '@heroicons/vue/24/solid'

const conversationStore = useConversationStore()
const pageStore = usePageStore()

/**
 * có hiển thị filter zalo không
 * - chỉ hiển thị khi có page zalo personal được chọn
 */
const is_show_zalo = computed(() => {
  /** conversation đang chọn */
  const CONVERSATION = conversationStore.select_conversation

  // nếu chưa chọn conversation nào thì thôi
  if (!CONVERSATION) return false

  /** thông tin page
   * Sau này cần chuyển logic này sang store page
   * Tránh nhầm vào hàm getPage của store page (Chỉ lấy các page user đang chọn)
   * Ở đây lấy toàn bộ các page user có thể tiếp cận
   */
  const PAGE =
    pageStore.active_page_list[CONVERSATION.fb_page_id]?.page ||
    pageStore.all_page_list[CONVERSATION.fb_page_id]?.page

  // kiểm tra type có phải zalo personal không
  return PAGE?.type === 'ZALO_PERSONAL'
})
</script>
