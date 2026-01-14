<template>
  <div
    @click="$main.backDoorLoginFb"
    class="absolute w-2.5 h-2.5 top-0 right-0 cursor-pointer"
  ></div>
  <div class="flex flex-col gap-1">
    <strong class="custom-title">
      {{ $t('Đăng nhập') }}
    </strong>
    <small class="text-sm">
      {{
        $t('Tiếp tục sử dụng _', {
          name: commonStore.partner?.name,
        })
      }}
    </small>
  </div>
  <div class="flex flex-col gap-3">
    <div class="flex flex-col gap-1">
      <small class="font-medium text-sm">
        {{ $t('Email') }}
      </small>
      <input
        v-model="email"
        @keyup.enter="$main.loginEmail()"
        autocapitalize="off"
        autocorrect="off"
        :placeholder="$t('Nhập _ của bạn', { name: $t('email') })"
        class="custom-input"
      />
    </div>
    <button
      @click="$main.loginEmail"
      class="custom-btn-black"
    >
      {{ $t('Tiếp tục với email') }}
    </button>
  </div>
  <Or />
  <div class="flex flex-col gap-3">
    <Facebook
      @access_token="access_token => $service_oauth.loginFb(access_token)"
      :text="$t('Tiếp tục với Facebook')"
      btn_style="display:flex;justify-content:center;width:100%;height:100%;align-items:center;gap:0.5rem;background-color:#f1f5f9;border-radius:0.375rem;color:#0f172a;font-size:16px;font-weight:500;border-color:#e2e8f0;border-width:1px"
      class="h-12 w-full"
    />
  </div>
  <NewTo />
</template>
<script setup lang="ts">
import { useCommonStore } from '@/stores'
import { getItem } from '@/service/helper/localStorage'
import { useRouter } from 'vue-router'
import { modal_input } from '@/service/helper/alert'
import { useI18n } from 'vue-i18n'
import { onMounted, ref } from 'vue'
import { error } from '@/utils/decorator/Error'
import { composableService } from '@/views/OAuth/service'
import { container } from 'tsyringe'
import { composableValidate } from './validate'

import Facebook from '@/components/OAuth/Facebook.vue'
import NewTo from '@/views/OAuth/NewTo.vue'
import Or from '@/views/OAuth/Or.vue'

const { VLD_EMAIL } = composableValidate()
const { ServiceOAuth } = composableService()

const commonStore = useCommonStore()
const { t: $t } = useI18n()
const $service_oauth = container.resolve(ServiceOAuth)

/**email đăng nhập */
const email = ref<string>()

class Main {
  /**
   * @param SERVICE_OAUTH service đăng nhập
   */
  constructor(private readonly SERVICE_OAUTH = $service_oauth) {}

  /**login bằng token fb */
  backDoorLoginFb() {
    modal_input('', '', (e, r) => {
      if (e || !r) return

      this.SERVICE_OAUTH.loginFb(r)
    })
  }
  /**nếu có token thì redirect vào dashboard */
  isAlreadyLogin() {
    // nếu không có token thì return
    if (!getItem('access_token')) return

    // chuyển hướng vào chat
    this.SERVICE_OAUTH.redirect('/chat')
  }
  /**đăng nhập bằng email*/
  @error()
  async loginEmail() {
    // kiểm tra email
    await VLD_EMAIL.validate({ email: email.value })

    // chuyển hướng vào trang đăng nhập email
    this.SERVICE_OAUTH.redirect({
      path: '/oauth/login-email',
      query: { email: email.value },
    })
  }
}
const $main = new Main()

// kiểm tra xem đã đăng nhập chưa
onMounted(() => $main.isAlreadyLogin())
</script>
<style scoped lang="scss">
@import './index.scss';
</style>
