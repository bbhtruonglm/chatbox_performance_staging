<template>
  <div class="flex flex-col items-center gap-3 py-5">
    <div class="flex flex-col items-center">
      <img
        src="@/assets/imgs/empty-page.png"
        class="w-56 h-auto"
      />
      <div class="text-2xl font-semibold">
        {{ $t('v1.view.main.dashboard.select_page.empty_page.title') }}
      </div>
      <div class="text-slate-700 text-base">
        <template v-if="$device.isDesktop()">
          {{ $t('v1.view.main.dashboard.select_page.empty_page.description') }}
        </template>
        <template v-else>
          {{ $t('Vui lòng tải ứng dụng Mobile để kết nối Trang') }}
        </template>
      </div>
    </div>
    <button
      @click="$main.openConnectPage"
      class="text-white bg-blue-600 rounded-md py-2 px-4 gap-2 flex items-center hover:brightness-90"
    >
      <template v-if="$device.isDesktop()">
        <PlusCircleIcon class="w-4 h-4" />
        <div class="text-sm font-medium">
          {{ $t('v1.view.main.dashboard.nav.select_platform') }}
        </div>
      </template>
      <template v-else>
        <ArrowDownCircleIcon class="w-4 h-4" />
        <div class="text-sm font-medium">
          {{ $t('Tải ứng dụng') }}
        </div>
      </template>
    </button>
  </div>
</template>
<script setup lang="ts">
import { KEY_TOGGLE_DROPDOWN_PICK_CONNECT_PLATFORM } from '@/views/Dashboard/symbol'
import { inject } from 'vue'
import { container } from 'tsyringe'
import { useRouter } from 'vue-router'

import PlusCircleIcon from '@/components/Icons/PlusCircle.vue'
import { ArrowDownCircleIcon } from '@heroicons/vue/24/solid'

import { Device, type IDevice } from '@/utils/helper/Device'
import { usePageManager } from '../composables/usePageManager'

const $props = withDefaults(
  defineProps<{
    /**tab được chọn khi mở modal kết nối page */
    tab?: string
  }>(),
  {}
)
const $router = useRouter()
const $device = container.resolve(Device)

// /**
//  * hàm mở dropdown tiền chọn nền tảng kết nối 
//  * @deprecated sử dụng toggleDropdown trong composable usePageManager
// */
// const toggleDropdownPickConnectPlatform = inject(
//   KEY_TOGGLE_DROPDOWN_PICK_CONNECT_PLATFORM
// )

/** composable */
const { toggleDropdown } = usePageManager()

class Main {
  /**
   * @param SERVICE_DEVICE thiết bị
   */
  constructor(
    private readonly SERVICE_DEVICE: IDevice = container.resolve(Device)
  ) {}
  /**
   * mở trang kết nối page
   * @param tab tab được chọn khi mở modal kết nối page
   */
  openConnectPage($event?: MouseEvent) {
    // nếu là thiết bị di động thì chuyển hướng sang trang download app
    if (this.SERVICE_DEVICE.isMobile()) return $router.push('/download-app')

    // mở dropdown tiền chọn nền tảng kết nối
    toggleDropdown?.($event, 'RIGHT', )
  }
}
const $main = new Main()
</script>
