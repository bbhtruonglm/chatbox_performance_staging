<template>
  <div
    class="dashboard-header bg-white rounded-lg py-3 px-3 md:px-6 flex justify-between"
  >
    <button
      type="button"
      @click="$router.push('/dashboard/select-page')"
      class="flex gap-3 items-center"
      :aria-label="$t('v1.view.main.dashboard.nav.home')"
    >
      <img
        :src="commonStore.partner?.logo?.full"
        :alt="commonStore.partner?.name || 'Logo'"
        width="120"
        height="32"
        class="h-8 w-auto"
      />
    </button>
    <div class="flex gap-3 md:gap-7 items-center">
      <slot name="right" />
      <User
        position="BOTTOM"
        :back="237"
      />
      <button
        type="button"
        @click="attach_ref?.toggleDropdown"
        v-tooltip.bottom="$t('v1.view.main.dashboard.nav.menu')"
        class="rounded-lg group"
        :aria-label="$t('v1.view.main.dashboard.nav.menu')"
      >
        <Squares2X2Icon
          class="w-7 h-7 m-auto group-hover:text-red-600"
          aria-hidden="true"
        />
      </button>
    </div>
    <Dropdown
      ref="attach_ref"
      :is_fit="false"
      :back="300"
      width="349px"
      height="auto"
      position="BOTTOM"
      class_content="flex flex-col gap-1 max-h-[calc(100vh-100px)]"
    >
      <OrgSetting @close="attach_ref?.toggleDropdown" />
    </Dropdown>
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import { useCommonStore } from '@/stores'

import User from '@/components/User.vue'

import { Squares2X2Icon } from '@heroicons/vue/24/solid'
import Dropdown from '@/components/Dropdown.vue'
import OrgSetting from '@/components/Main/OrgSetting.vue'

const commonStore = useCommonStore()

/**ref của menu đính kèm */
const attach_ref = ref<InstanceType<typeof Dropdown>>()
</script>
