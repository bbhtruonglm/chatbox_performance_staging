<template>
  <CardItem>
    <template #icon>
      <ChatBubbleLeftRightIcon class="w-5 h-5" />
    </template>
    <template #title>
      {{ $t('Hội thoại') }}
      <div class="text-xs text-slate-500">
        {{ $t('Thiết lập hội thoại khi ở chế độ Gộp trang.') }}
      </div>
    </template>
    <template #item>
      <div class="flex gap-3">
        <InboxStackIcon class="text-slate-700 size-4 flex-shrink-0" />
        <div class="flex flex-col md:flex-row w-full justify-between gap-3 min-w-0">
          <div class="flex-shrink-0">
            <h4 class="text-sm font-medium">
              {{ $t('Chế độ hiển thị hội thoại') }}
            </h4>
            <h4 class="text-sm text-slate-500">
              {{ $t('Thiết lập hiển thị hội thoại') }}
            </h4>
          </div>
          <select
            v-model="org_sort_conversation"
            class="w-full md:w-72 p-2 px-3 py-2 border border-gray-300 rounded-md shadow-sm text-sm"
          >
            <option value="NEWEST">
              {{ $t('Hội thoại mới nhất') }}
            </option>
            <option value="UNREAD">
              {{ $t('Hội thoại chưa đọc') }}
            </option>
          </select>
        </div>
      </div>
      <div class="h-px w-full bg-slate-200" />
      <div class="flex gap-3">
        <SwatchIcon class="text-slate-700 size-4 flex-shrink-0" />
        <div class="flex flex-col md:flex-row w-full justify-between gap-3 min-w-0">
          <div class="flex-shrink-0">
            <h4 class="text-sm font-medium">
              {{ $t('Chế độ hiển thị gắn nhãn') }}
            </h4>
            <h4 class="text-sm text-slate-500">
              {{ $t('Thiết lập chế độ hiển thị gắn nhãn trên hội thoại') }}
            </h4>
          </div>
          <select
            v-model="org_display_label_type"
            class="w-full md:w-72 p-2 px-3 py-2 border border-gray-300 rounded-md shadow-sm text-sm"
          >
            <option value="ICON_TOOLTIP">
              {{ $t('Hiện chấm màu có chú giải') }}
            </option>
            <option value="ICON">
              {{ $t('Chỉ hiện chấm màu') }}
            </option>
            <option value="FULL">
              {{ $t('Hiện văn bản') }}
            </option>
          </select>
        </div>
      </div>
      <div class="h-px w-full bg-slate-200" />
      <div class="flex gap-3">
        <UsersIcon class="text-slate-700 size-4 flex-shrink-0" />
        <div class="flex flex-row w-full justify-between gap-3 min-w-0">
          <div class="">
            <h4 class="text-sm font-medium">
              {{ $t('Hiện hội thoại theo nhân viên chỉ định') }}
            </h4>
            <h4 class="text-sm text-slate-500">
              {{
                $t(
                  'Khi bật chế độ này, hệ thống chỉ hiện ra hội thoại cho nhân viên được chỉ định.'
                )
              }}
            </h4>
            <h4 class="text-sm text-slate-500">
              {{ $t('Đối với Quản trị viên vẫn xem được toàn bộ hội thoại.') }}
            </h4>
          </div>
          <div>
            <Toggle
              v-model="org_is_only_visible_client_of_staff"
              class_toggle="peer-checked:bg-black"
            />
          </div>
        </div>
      </div>
      <div class="h-px w-full bg-slate-200" />
      <div class="flex gap-3">
        <UserCircleIcon class="text-slate-700 size-4 flex-shrink-0" />
        <div class="flex flex-row w-full justify-between gap-3 min-w-0">
          <div class="">
            <h4 class="text-sm font-medium">
              {{ $t('Ẩn ảnh đại diện của trang') }}
            </h4>
            <h4 class="text-sm text-slate-500">
              {{ $t('Ẩn ảnh đại diện của trang trên hội thoại') }}
            </h4>
          </div>
          <div>
            <Toggle
              v-model="org_is_hide_page_avatar"
              class_toggle="peer-checked:bg-black"
            />
          </div>
        </div>
      </div>
    </template>
  </CardItem>
</template>
<script setup lang="ts">
import { computed } from 'vue'
import { useOrgStore } from '@/stores'
import { initRequireData } from '@/views/Dashboard/Org/composable'
import { set } from 'lodash'

import CardItem from '@/components/Main/Dashboard/CardItem.vue'
import Toggle from '@/components/Toggle.vue'

import {
  SwatchIcon,
  UserCircleIcon,
  ChatBubbleLeftRightIcon,
  InboxStackIcon,
  UsersIcon,
} from '@heroicons/vue/24/solid'

const orgStore = useOrgStore()
const { updateOrg } = initRequireData()

/**chỉ hiện hội thoại của nv */
const org_is_only_visible_client_of_staff = computed({
  get() {
    // lấy dữ liệu từ store
    return orgStore.selected_org_info?.org_config
      ?.org_is_only_visible_client_of_staff
  },
  set(val) {
    // set dữ liệu vào store
    set(
      orgStore,
      'selected_org_info.org_config.org_is_only_visible_client_of_staff',
      val
    )

    // gọi api update
    updateOrg()
  },
})
/**chỉ hiện hội thoại của nv */
const org_sort_conversation = computed({
  get() {
    // lấy dữ liệu từ store
    return (
      orgStore.selected_org_info?.org_config?.org_sort_conversation || 'NEWEST'
    )
  },
  set(val) {
    // set dữ liệu vào store
    set(orgStore, 'selected_org_info.org_config.org_sort_conversation', val)

    // gọi api update
    updateOrg()
  },
})
/**cài đặt ẩn ảnh của trang trong danh sách conversation, hover vào mới hiện */
const org_is_hide_page_avatar = computed({
  get() {
    // lấy dữ liệu từ store
    return orgStore.selected_org_info?.org_config?.org_is_hide_page_avatar
  },
  set(val) {
    // set dữ liệu vào store
    set(orgStore, 'selected_org_info.org_config.org_is_hide_page_avatar', val)

    // gọi api update
    updateOrg()
  },
})
/**
 * chế độ hiển thị gắn nhãn
 * - ICON_TOOLTIP: Hiện chấm màu có chú giải
 * - ICON: Chỉ hiện chấm màu
 * - FULL: Hiện văn bản
 */
const org_display_label_type = computed({
  get() {
    // lấy dữ liệu từ store
    return (
      orgStore.selected_org_info?.org_config?.org_display_label_type ||
      'ICON_TOOLTIP'
    )
  },
  set(val) {
    // set dữ liệu vào store
    set(orgStore, 'selected_org_info.org_config.org_display_label_type', val)

    // gọi api update
    updateOrg()
  },
})
</script>
