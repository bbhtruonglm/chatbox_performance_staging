<template>
  <div
    class="flex items-center gap-3"
    role="separator"
    aria-orientation="horizontal"
  >
    <hr
      class="border-slate-200 border w-full"
      aria-hidden="true"
    />
    <span class="font-medium text-sm flex-shrink-0 text-slate-500">
      {{ $t('Hoặc') }}
    </span>
    <hr
      class="border-slate-200 border w-full"
      aria-hidden="true"
    />
  </div>
</template>
<style scoped lang="scss">
@import './index.scss';
</style>
