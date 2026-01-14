<template>
  <BaseModal
    ref="ref_group_upsert_modal"
    class_modal="w-[1296px] h-[716px]"
    class_header="text-lg font-semibold px-6 py-3"
    class_close="right-3"
    class_body="border-y border-slate-300 bg-slate-100 py-4 px-5 gap-4 flex flex-col relative"
    class_footer="px-6 py-3 flex justify-between"
  >
    <template v-slot:header>
      <template v-if="is_update">
        {{ $t('Chỉnh sửa') }}
      </template>
      <template v-else>
        {{ $t('Tạo nhóm trang') }}
      </template>
    </template>
    <template v-slot:body>
      <div
        v-if="is_loading"
        class="absolute left-1/2 -translate-y-1/2"
      >
        <Loading />
      </div>
      <div class="flex flex-col gap-1">
        <label class="text-sm font-medium">
          {{ $t('Tên nhóm') }}
          <span class="text-red-500">*</span>
        </label>
        <input
          v-model="group_name"
          :placeholder="$t('Nhập tên nhóm')"
          class="py-2 px-3 rounded-md border border-gray-200 shadow-sm tex-sm"
        />
      </div>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 min-h-0">
        <GroupSection
          :title="$t('Trang')"
          :total_item_selected="$main.countSelectedItems(map_group_pages)"
        >
          <template #action>
            <div class="flex gap-2 items-center">
              <p
                v-if="$main.countSelectedItems(map_group_pages)"
                class="text-sm font-medium"
              >
                {{ $t('Đã chọn') }}:
                {{ $main.countSelectedItems(map_group_pages) }}
                {{ $t('Trang') }}
              </p>
              <div
                class="py-1.5 px-2 rounded-lg flex gap-2 bg-white items-center"
              >
                <MagnifyingGlassIcon class="size-3.5 text-slate-500" />
                <input
                  v-model="search_page"
                  type="text"
                  class="text-xs outline-none"
                  :placeholder="$t('Tìm kiếm')"
                />
              </div>
            </div>
          </template>
          <template #group>
            <div
              v-for="(pages, key) of grouped_pages"
              class="flex flex-col text-sm"
            >
              <div class="flex gap-2 items-center font-medium sticky top-0 bg-slate-100 z-10 pb-2">
                <PageTypeIcon
                  :page_type="key"
                  class="size-5"
                />
                <p>{{ $t('v1.common.' + key?.toLocaleLowerCase()) }}</p>
              </div>
              <GroupItem
                class="mb-2 last-of-type:m-0"
                v-for="os of pages"
                :name="os?.page_info?.name"
                v-model="map_group_pages[os?.page_info?.fb_page_id || '']"
              >
                <template #avatar>
                  <PageAvatar
                    :page_info="os?.page_info"
                    class="item-avatar"
                  />
                </template>
                <template #type>
                  <PageTypeIcon
                    :page_type="os?.page_info?.type"
                    class="size-3.5"
                  />
                </template>
                <template #info>
                  {{ os?.page_info?.fb_page_id }}
                </template>
              </GroupItem>
            </div>
          </template>
        </GroupSection>
        <GroupSection
          :title="$t('Thành viên')"
          :total_item_selected="$main.countSelectedItems(map_group_staffs)"
        >
          <template #action>
            <div class="flex gap-2 items-center">
              <p
                v-if="$main.countSelectedItems(map_group_staffs)"
                class="text-sm font-medium"
              >
                {{ $t('Đã chọn') }}:
                {{ $main.countSelectedItems(map_group_staffs) }}
                {{ $t('Thành viên') }}
              </p>
              <div
                class="py-1.5 px-2 rounded-lg flex gap-2 bg-white items-center"
              >
                <MagnifyingGlassIcon class="size-3.5 text-slate-500" />
                <input
                  v-model="search_staff"
                  type="text"
                  class="text-xs outline-none"
                  :placeholder="$t('Tìm kiếm')"
                />
              </div>
            </div>
          </template>
          <template #group>
            <GroupItem
              v-for="ms of filtered_staffs"
              :name="ms?.user_info?.full_name"
              v-model="map_group_staffs[ms?.staff_id || '']"
            >
              <template #avatar>
                <StaffAvatar
                  :id="ms?.staff_id"
                  class="item-avatar"
                />
              </template>
              <template #info>
                {{ ms?.staff_id }}
                -
                <template v-if="$member_ship_helper.isAdmin(ms)">
                  {{ $t('v1.view.main.dashboard.org_staff.admin') }}
                </template>
                <template v-else>
                  {{ $t('v1.view.main.dashboard.org_staff.member') }}
                </template>
              </template>
            </GroupItem>
          </template>
        </GroupSection>
      </div>
    </template>
    <template v-slot:footer>
      <button
        @click="$main.toggleModal()"
        class="custom-btn bg-slate-200"
      >
        {{ $t('Đóng') }}
      </button>
      <button
        @click="$main.upsertGroup()"
        :disabled="!group_name || is_loading"
        class="custom-btn bg-blue-700 text-white"
      >
        <template v-if="is_update">
          {{ $t('Cập nhật') }}
        </template>
        <template v-else>
          {{ $t('Tạo nhóm') }}
        </template>
      </button>
    </template>
  </BaseModal>
</template>
<script setup lang="ts">
import { computed, ref } from 'vue'
import { useOrgStore } from '@/stores'
import { SingletonMemberShipHelper } from '@/utils/helper/Billing/MemberShip'
import { loadingV2 } from '@/utils/decorator/Loading'
import { error } from '@/utils/decorator/Error'
import { container } from 'tsyringe'
import { BillingAppGroup } from '@/utils/api/Billing'
import { useI18n } from 'vue-i18n'
import { filter, keys } from 'lodash'
import { groupService } from './service'

import PageAvatar from '@/components/Avatar/PageAvatar.vue'
import StaffAvatar from '@/components/Avatar/StaffAvatar.vue'
import BaseModal from '@/components/Base/BaseModal.vue'
import GroupItem from '@/views/Dashboard/Org/Setting/Group/GroupUpsertModal/GroupItem.vue'
import GroupSection from '@/views/Dashboard/Org/Setting/Group/GroupUpsertModal/GroupSection.vue'
import Loading from '@/components/Loading.vue'
import PageTypeIcon from '@/components/Avatar/PageTypeIcon.vue'
import { Toast } from '@/utils/helper/Alert/Toast'
import { MagnifyingGlassIcon } from '@heroicons/vue/24/outline'
import type { MemberShipInfo, OwnerShipInfo } from '@/service/interface/app/billing'
import { nonAccentVn } from '@/service/helper/format'
import type { IPage, PageType } from '@/service/interface/app/page'

const { GroupService } = groupService()
const $emit = defineEmits(['done'])

const orgStore = useOrgStore()
const $member_ship_helper = SingletonMemberShipHelper.getInst()
const { t: $t } = useI18n()

/**ref của modal hướng dẫn cài đặt */
const ref_group_upsert_modal = ref<InstanceType<typeof BaseModal>>()
/**cờ xác định đang thêm hay sửa */
const is_update = ref<boolean>()
/**cờ xác định đang tải dữ liệu */
const is_loading = ref<boolean>()
/**tên nhóm */
const group_name = ref<string>()
/**ánh xạ danh sách trang được chọn */
const map_group_pages = ref<Record<string, boolean>>({})
/**ánh xạ danh sách nhân viên được chọn */
const map_group_staffs = ref<Record<string, boolean>>({})
/**id nhóm được chọn khi cập nhật */
const selected_group_id = ref<string>()

/** từ khóa tìm kiếm trang */
const search_page = ref<string>('')

/** từ kháo tìm kiếm nhân viên */
const search_staff = ref<string>('')

/** danh sách các trang được group theo nền tảng */
const grouped_pages = computed<Record<PageType, OwnerShipInfo[]>>(() => {
  /** kết quả */
  let result: Record<string, OwnerShipInfo[]> = {}

  orgStore.list_os?.forEach(os => {
    /** nền tảng của trang */
    const PLATFORM = os?.page_info?.type

    // nếu không có nền tảng thì thôi
    if (!PLATFORM) return

    // nếu không thỏa mãn search thì thôi
    if (!$main.checkSearchPage(search_page.value, os.page_info || {})) return

    /** danh sách trang theo nền tảng */
    result[PLATFORM] = [...(result[PLATFORM] || []), os]
  })

  return result
})

/** danh sách nhân viên đã được lọc theo tìm kiếm */
const filtered_staffs = computed<MemberShipInfo[]>(() => {
  return (
    orgStore.list_ms?.filter(ms =>
      $main.checkSearchStaff(search_staff.value, ms)
    ) || []
  )
})

const $toast = container.resolve(Toast)

class Main {
  /**
   * @param API_GROUP API nhóm
   */
  constructor(
    private readonly SERVICE_GROUP = container.resolve(GroupService),
    private readonly API_GROUP = container.resolve(BillingAppGroup)
  ) {}

  /**đếm số phần tử được chọn */
  countSelectedItems(map: Record<string, boolean>) {
    return this.SERVICE_GROUP.getSelectedIds(map).length
  }
  /**làm sạch dữ liệu */
  clearData() {
    group_name.value = undefined
    map_group_pages.value = {}
    map_group_staffs.value = {}
    selected_group_id.value = undefined
  }
  /**nhập dữ liệu nhóm muốn sửa */
  setData(group: IGroup) {
    group_name.value = group.group_name
    map_group_pages.value = this.SERVICE_GROUP.setSelectedIds(group.group_pages)
    map_group_staffs.value = this.SERVICE_GROUP.setSelectedIds(
      group.group_staffs
    )
    selected_group_id.value = group.group_id
  }
  /**ẩn hiện modal */
  toggleModal(group?: IGroup) {
    // cập nhật cờ xác định đang thêm hay sửa
    is_update.value = !!group

    // làm sạch dữ liệu
    this.clearData()

    // nhập dữ liệu nhóm muốn sửa
    if (group) this.setData(group)

    // ẩn hiện modal
    ref_group_upsert_modal.value?.toggleModal?.()
  }
  /**thêm/sửa Nhóm */
  @loadingV2(is_loading, 'value')
  @error($toast)
  async upsertGroup() {
    // check tên nhóm
    if (!group_name.value?.trim()) {
      throw $t('Tên nhóm không hợp lệ')
    }

    // cập nhật nhóm cho tổ chức
    if (is_update.value)
      await this.API_GROUP.updateGroup(
        selected_group_id.value!,
        group_name.value!,
        this.SERVICE_GROUP.getSelectedIds(map_group_pages.value),
        this.SERVICE_GROUP.getSelectedIds(map_group_staffs.value)
      )
    // tạo nhóm cho tổ chức
    else
      await this.API_GROUP.createGroup(
        group_name.value!,
        this.SERVICE_GROUP.getSelectedIds(map_group_pages.value),
        this.SERVICE_GROUP.getSelectedIds(map_group_staffs.value)
      )

    // báo component cha cập nhật lại dữ liệu
    $emit('done')

    // đóng modal
    this.toggleModal()
  }

  /** kiểm tra xem từ khóa có nằm trong tên hoặc id trang không */
  checkSearchPage(key_word: string, page: IPage) {
    /** từ khóa khi được bỏ dấu và viết hoa và xóa dấu cách*/
    const KEY_WORD = nonAccentVn(key_word)?.replace(/ /g, '')

    /** tên trang khi được bỏ dấu và viết hoa và xóa dấu cách */
    const PAGE_NAME = nonAccentVn(page?.name || '')?.replace(/ /g, '')

    return PAGE_NAME.includes(KEY_WORD) || page?.fb_page_id?.includes(KEY_WORD)
  }

  /** kiểm tra xem từ khóa có năm trong tên hoặc id của nhân sự không */
  checkSearchStaff(key_word: string, staff?: MemberShipInfo) {
    /** thông tin của nhân sự */
    const STAFF = staff?.user_info

    /**—from khóa khi được bỏ dấu và viết hoa và xóa dấu cách*/
    const KEY_WORD = nonAccentVn(key_word)?.replace(/ /g, '')

    /** tên nhân sự khi được bỏ dấu và viết hoa và xóa dấu cách */
    const STAFF_NAME = nonAccentVn(STAFF?.full_name || '')?.replace(/ /g, '')

    return STAFF_NAME.includes(KEY_WORD) || staff?.staff_id?.includes(KEY_WORD)
  }
}
const $main = new Main()

defineExpose({ toggleModal: $main.toggleModal.bind($main) })
</script>
<style lang="scss" scoped>
.custom-btn {
  @apply py-2 px-4 rounded-md font-medium text-sm;
}
.item-avatar {
  @apply size-8 rounded-full;
}
</style>
