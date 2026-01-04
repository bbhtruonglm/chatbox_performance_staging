<template>
  <div class="custom-description">
    <span>
      {{ $t('Bạn mới sử dụng _', { name: commonStore.partner?.name }) }}
    </span>
    <button
      type="button"
      @click="$service_oauth.redirect('/oauth/register')"
      class="custom-link"
      :aria-label="$t('Bắt đầu đăng ký tài khoản mới')"
    >
      {{ $t('Bắt đầu ngay') }}
      <ArrowRightIcon
        class="w-4 h-4 lucide lucide-arrow-right"
        aria-hidden="true"
      />
    </button>
  </div>
</template>
<script setup lang="ts">
import { useCommonStore } from '@/stores'
import { composableService } from './service'
import { container } from 'tsyringe'

import { ArrowRightIcon } from '@heroicons/vue/24/solid'

const commonStore = useCommonStore()
const { ServiceOAuth } = composableService()

const $service_oauth = container.resolve(ServiceOAuth)
</script>
<style scoped lang="scss">
@import './index.scss';
</style>
