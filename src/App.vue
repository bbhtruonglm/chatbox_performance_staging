<template>
  <!-- 
    class "container-fluid" để intergrate với ext, không phải là bootstrap
    không được xoá
  -->
  <div class="w-dvw h-dvh container-fluid bg-gradient-primary">
    <Network />
    <AdBlocker />
    <Loading
      v-if="commonStore.is_loading_full_screen"
      type="FULL"
    />
    <div class="h-full overflow-y-auto">
      <router-view />
    </div>
  </div>
</template>

<script setup lang="ts">
import { useCommonStore } from '@/stores'

import Loading from '@/components/Loading.vue'
import Network from './components/Network.vue'
import AdBlocker from './components/AdBlocker.vue'
import { onMounted } from 'vue'
import { Toast } from './utils/helper/Alert/Toast'
import { N4SerivcePublicPartner } from './utils/api/N4Service/Partner'
import { setItem } from './service/helper/localStorage'
import { error } from './utils/decorator/Error'
import { container } from 'tsyringe'
import { QueryString, type IQueryString } from './utils/helper/QueryString'
import { useKeyboardShortcut } from '@/views/composables/useKeyboardShortcut'
import { confirm } from './service/helper/alert'

const commonStore = useCommonStore()
const $toast = container.resolve(Toast)

useKeyboardShortcut()

class Main {
  /**
   * @param SERVICE_QUERY_STRING query string service
   */
  constructor(
    private readonly SERVICE_QUERY_STRING: IQueryString = container.resolve(
      QueryString
    )
  ) {}

  /**lưu lại ref được truyền ở param */
  saveRef() {
    /**mã ref */
    const REF =
      this.SERVICE_QUERY_STRING.get('rf') ||
      this.SERVICE_QUERY_STRING.get('ref')

    // nếu không có ref thì thôi
    if (!REF) return

    // lưu ref vào local storage
    setItem('ref', REF)
  }

  saveCliendID() {
    /** id của client cần gửi tin nhắn tới */
    const CLIENT_ID = this.SERVICE_QUERY_STRING.get('client_id')

    // nếu không có id thì thôi
    if (!CLIENT_ID) return

    // lưu id vào local storage
    setItem('client_id', CLIENT_ID)
  }

  /**lưu lại id tổ chức nếu được truyền ở param */
  saveOrgId() {
    /**id tổ chức */
    const ORG_ID = this.SERVICE_QUERY_STRING.get('org_id')

    // nếu không có ref thì thôi
    if (!ORG_ID) return

    // lưu ref vào local storage
    setItem('selected_org_id', ORG_ID)
  }
  /**ghi đè lại token lấy từ param, phục vụ cho trường hợp mở từ app mobile */
  getParamToken() {
    /**mã xác thực */
    const ACCESS_TOKEN =
      this.SERVICE_QUERY_STRING.get('access_token') ||
      this.SERVICE_QUERY_STRING.get('token')

    // nếu không có token thì thôi
    if (!ACCESS_TOKEN) return

    // lưu token vào local storage
    setItem('access_token', ACCESS_TOKEN)
  }
  /**Lấy thông tin đối tác */
  @error($toast)
  async getPartnerInfo() {
    // lấy thông tin đối tác
    commonStore.partner = await new N4SerivcePublicPartner().readPartner()

    // nếu không có thông tin đối tác thì báo lỗi
    if (!commonStore.partner) throw 'NOT_FOUND.PARTNER'

    // thay đổi title
    document.title = commonStore.partner?.name || ''

    // cập nhật meta tags cho SEO
    $main.updateMetaTags()

    // thay đổi favicon
    $main.setFavicon()
  }
  /** Cập nhật động các meta tags SEO */
  updateMetaTags() {
    /** Tên partner */
    const PARTNER_NAME = commonStore.partner?.name || ''
    /** Description động */
    const DESCRIPTION = `${PARTNER_NAME} - Nền tảng quản lý tin nhắn, chăm sóc khách hàng đa kênh. Hỗ trợ Facebook, Zalo, và nhiều kênh khác.`

    // Cập nhật meta description
    const metaDesc = document.querySelector('meta[name="description"]')
    if (metaDesc) metaDesc.setAttribute('content', DESCRIPTION)

    // Cập nhật OG title
    const ogTitle = document.querySelector('meta[property="og:title"]')
    if (ogTitle) ogTitle.setAttribute('content', PARTNER_NAME)

    // Cập nhật OG description
    const ogDesc = document.querySelector('meta[property="og:description"]')
    if (ogDesc) ogDesc.setAttribute('content', DESCRIPTION)

    // Cập nhật Twitter title
    const twitterTitle = document.querySelector('meta[name="twitter:title"]')
    if (twitterTitle) twitterTitle.setAttribute('content', PARTNER_NAME)

    // Cập nhật Twitter description
    const twitterDesc = document.querySelector(
      'meta[name="twitter:description"]'
    )
    if (twitterDesc) twitterDesc.setAttribute('content', DESCRIPTION)

    // Cập nhật OG image nếu có logo
    if (commonStore.partner?.logo?.full) {
      /** OG Image tag */
      let ogImage = document.querySelector('meta[property="og:image"]')
      if (!ogImage) {
        ogImage = document.createElement('meta')
        ogImage.setAttribute('property', 'og:image')
        document.head.appendChild(ogImage)
      }
      ogImage.setAttribute('content', commonStore.partner.logo.full)
    }
  }
  /**Thay đổi favicon */
  setFavicon() {
    /**thẻ link */
    const LINK = document.createElement('link')

    // thêm thông tin cho thẻ link
    LINK.id = 'favicon'
    LINK.rel = 'icon'
    LINK.href = commonStore.partner?.logo?.icon || ''

    // thêm vào head
    document.head.appendChild(LINK)
  }
}
const $main = new Main()

// lấy token từ param nếu có
$main.getParamToken()
// lưu lại ref để dùng sau
$main.saveRef()
// lưu lại id client cần gửi tin nhắn tới
$main.saveCliendID()
// lưu lại id tổ chức nếu được truyền ở param
$main.saveOrgId()

// lấy thông tin đối tác khi component được mount
onMounted($main.getPartnerInfo)
</script>

<style lang="scss">
@import '@/assets/css/index.scss';
</style>
