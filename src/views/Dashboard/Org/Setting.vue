<template>
  <!-- <General /> -->
  <Page />
  <Staff />
  <Group />
  <Conversation />
  <CustomerInfo />
</template>
<script setup lang="ts">
import General from '@/views/Dashboard/Org/Setting/General.vue'
import Page from '@/views/Dashboard/Org/Setting/Page.vue'
import Staff from '@/views/Dashboard/Org/Setting/Staff.vue'
import Conversation from '@/views/Dashboard/Org/Setting/Conversation.vue'
import Group from '@/views/Dashboard/Org/Setting/Group.vue'
import CustomerInfo from '@/views/Dashboard/Org/Setting/CustomerInfo.vue'

import { useOrgStore } from '@/stores'
import { watch, onMounted } from 'vue'
import { read_os, read_ms } from '@/service/api/chatbox/billing'
import { toastError } from '@/service/helper/alert'

const orgStore = useOrgStore()

/** Load dữ liệu song song cho trang Setting */
async function loadData() {
  if (!orgStore.selected_org_id) return
  orgStore.is_loading = true
  try {
    await Promise.all([
      // Load Page
      read_os(orgStore.selected_org_id).then(res => {
        orgStore.list_os = res
        // Update quota page
        if (orgStore.selected_org_info?.org_package) {
          orgStore.selected_org_info.org_package.org_current_page = res.length
        }
      }),
      // Load Staff
      read_ms(orgStore.selected_org_id).then(res => {
        orgStore.list_ms = res
        // Update quota staff
        if (orgStore.selected_org_info?.org_package) {
          orgStore.selected_org_info.org_package.org_current_staff = res.filter(
            ms => ms?.ms_is_active
          ).length
        }
      }),
    ])
  } catch (e) {
    toastError(e)
  } finally {
    orgStore.is_loading = false
  }
}

// Load khi mount
onMounted(loadData)

// Load khi đổi org
watch(() => orgStore.selected_org_id, loadData)
</script>
