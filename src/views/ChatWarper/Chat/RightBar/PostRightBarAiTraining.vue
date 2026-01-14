<template>
  <div class="bg-white rounded-lg h-full flex flex-col">
    <!-- Header -->
    <div>
      <div
        class="py-2 px-3 border-b flex-shrink-0 text-sm font-semibold flex items-center justify-between"
      >
        <div class="flex items-center gap-1">
          <SparklesIcon class="size-5" />
          {{ $t('Huấn luyện AI') }}
        </div>
      </div>
    </div>

    <!-- Nội dung -->
    <div class="p-3 flex flex-col w-full gap-1">
      <!-- Input / Textarea -->
      <div>
        <textarea
          ref="textarea_ref"
          v-if="is_expanded"
          v-model="description"
          rows="6"
          class="w-full border rounded-md px-3 py-2 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none resize-none"
          placeholder="Nhập nội dung để huấn luyện AI tư vấn khách hàng"
        ></textarea>

        <input
          v-else
          type="text"
          v-model="description"
          @focus="expandAndFocus"
          class="w-full border rounded-md px-3 py-2 text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
          placeholder="Nhập nội dung để huấn luyện AI tư vấn khách hàng"
        />
      </div>
      <div>
        <!-- Nút Lưu -->
        <div
          v-if="is_expanded"
          class="mt-2 flex justify-end gap-2"
        >
          <button
            class="px-3 py-1.5 text-sm bg-gray-200 rounded-md hover:bg-gray-300"
            @click="onCancel"
          >
            {{ $t('Hủy') }}
          </button>
          <button
            class="px-3 py-1.5 text-sm bg-blue-600 text-white rounded-md hover:bg-blue-700"
            @click="onSave"
          >
            {{ $t('Lưu') }}
          </button>
        </div>
        <div
          v-else
          class="text-xs text-slate-700"
        >
          {{
            $t(
              'Khi khách hàng tương tác vào bài viết này, AI sẽ dựa theo nội dung của bạn huấn luyện để AI tư vấn khách hàng hiệu quả.'
            )
          }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, nextTick, ref, watch } from 'vue'
import { SparklesIcon } from '@heroicons/vue/24/solid'
import { update_info_conversation } from '@/service/api/chatbox/n4-service'
import { reject } from 'lodash'
import { useConversationStore } from '@/stores'
/** Trạng thái mở rộng */
const is_expanded = ref(false)
/** mô tả */
const description = ref<string>('')
/** ref của textarea */
const textarea_ref = ref<HTMLTextAreaElement | null>(null)

/** mở rộng và focus vào textarea */
const expandAndFocus = async () => {
  // Mở rộng
  is_expanded.value = true
  // Chờ render
  await nextTick()
  // Focus vào textarea
  textarea_ref.value?.focus()
}
/** Khai báo conversation */
const conversationStore = useConversationStore()
/** Theo dõi data trong store */
watch(
  () => conversationStore.select_conversation?.data_key,
  (newVal, oldVal) => {
    /** Nếu id thay đổi thì mới cập nhật */
    if (newVal !== oldVal) {
      /** Gán giá trị */
      description.value =
        conversationStore.select_conversation?.ai_description || ''
      /** Reset trạng thái mở rộng */
      is_expanded.value = false
    }
  },
  { immediate: true }
)

/** Hủy bỏ chỉnh sửa */
const onCancel = () => {
  is_expanded.value = false
}
/**id của trang */
const page_id = computed(
  () => conversationStore.select_conversation?.fb_page_id
)
/**id của khách hàng */
const client_id = computed(
  () => conversationStore.select_conversation?.fb_client_id
)
/** Gọi API lưu dữ liệu */
/** Gọi API lưu dữ liệu */
/** Gọi API lưu dữ liệu */
const onSave = async () => {
  /** Capture lại hội thoại và nội dung tại thời điểm bấm lưu */
  const CURRENT_CONVERSATION = conversationStore.select_conversation
  /** Mô tả hiện tại */
  const CURRENT_DESCRIPTION = description.value
  // Kiểm tra xem có id trang và khách hàng không
  if (!CURRENT_CONVERSATION?.fb_page_id || !CURRENT_CONVERSATION?.fb_client_id)
    return

  /** Payload gửi đi */
  const PAYLOAD = {
    page_id: CURRENT_CONVERSATION.fb_page_id,
    client_id: CURRENT_CONVERSATION.fb_client_id,
    ai_description: CURRENT_DESCRIPTION,
  }

  try {
    /** gọi api cập nhật tên khách hàng */
    await new Promise((resolve, reject_promise) => {
      // Gọi API cập nhật thông tin hội thoại
      update_info_conversation(PAYLOAD, (error, response) => {
        /** Nếu lỗi thì reject */
        if (error) reject_promise(error)
        // Nếu thành công thì resolve
        else resolve(response)
      })
    })

    // 1. Cập nhật object đã capture (để chắc chắn)
    CURRENT_CONVERSATION.ai_description = PAYLOAD.ai_description

    // 2. Cập nhật trong danh sách hội thoại của store (QUAN TRỌNG: để đồng bộ khi chuyển qua lại)
    if (
      CURRENT_CONVERSATION.data_key &&
      conversationStore.conversation_list[CURRENT_CONVERSATION.data_key]
    ) {
      // Cập nhật mô tả trong danh sách hội thoại
      conversationStore.conversation_list[
        CURRENT_CONVERSATION.data_key
      ].ai_description = PAYLOAD.ai_description
    }

    /** 3. Check conversation đang được chọn hiện tại và cập nhật nếu khớp  */
    if (
      conversationStore.select_conversation?.fb_page_id === PAYLOAD.page_id &&
      conversationStore.select_conversation?.fb_client_id === PAYLOAD.client_id
    ) {
      // Cập nhật mô tả trong hội thoại đang chọn
      conversationStore.select_conversation.ai_description =
        PAYLOAD.ai_description

      // Đang ở đúng conversation thì tắt form
      is_expanded.value = false
    }

    console.log('Đã lưu:', PAYLOAD.ai_description)
  } catch (err) {
    console.error('Lỗi khi lưu:', err)
  }
}
</script>
