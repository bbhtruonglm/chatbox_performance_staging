<template>
  <div class="custom-description">
    <span>
      {{ $t('Bạn mới sử dụng _', { name: commonStore.partner?.name }) }}
    </span>
    <div
      @click="$service_oauth.redirect('/oauth/register')"
      class="custom-link"
    >
      {{ $t('Bắt đầu ngay') }}
      <ArrowRightIcon class="w-4 h-4 lucide lucide-arrow-right" />
    </div>
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
