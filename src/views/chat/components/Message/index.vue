<script setup lang='ts'>
import { computed, ref } from 'vue'
import { NDropdown } from 'naive-ui'
import AvatarComponent from './Avatar.vue'
import TextComponent from './Text.vue'
import { SvgIcon } from '@/components/common'
import { copyText } from '@/utils/format'
import { useIconRender } from '@/hooks/useIconRender'
import { t } from '@/locales'

interface Props {
  dateTime?: string
  text?: string
  inversion?: boolean
  error?: boolean
  loading?: boolean
}

interface Emit {
  (ev: 'regenerate'): void
  (ev: 'delete'): void
}

const props = defineProps<Props>()

const emit = defineEmits<Emit>()

const { iconRender } = useIconRender()

const textRef = ref<HTMLElement>()

const options = [
  {
    label: '朗读',
    key: 'read',
    icon: iconRender({ icon: 'ri:volume-up-line' }),
  },
  {
    label: t('chat.copy'),
    key: 'copyText',
    icon: iconRender({ icon: 'ri:file-copy-2-line' }),
  },
  {
    label: t('common.delete'),
    key: 'delete',
    icon: iconRender({ icon: 'ri:delete-bin-line' }),
  },
]

function handleSelect(key: 'copyRaw' | 'copyText' | 'delete') {
  switch (key) {
    case 'copyText':
      copyText({ text: props.text ?? '' })
      return
    case 'delete':
      emit('delete')
  }
}

const isReading = ref(false)

const readButtonText = computed(() => {
  return isReading.value ? '取消朗读' : '朗读文字'
})

function handleRead() {
  if (isReading.value) {
    speechSynthesis.cancel()
    isReading.value = false
    return
  }

  const speech = new SpeechSynthesisUtterance(props.text ?? '')
  speech.lang = 'zh-CN'
  speech.volume = 1.0
  speech.rate = 1.0
  speech.pitch = 1.0
  speech.onstart = () => {
    isReading.value = true
  }
  speech.onend = () => {
    isReading.value = false
  }
  speechSynthesis.speak(speech)
}

function handleRegenerate() {
  emit('regenerate')
}
</script>

<template>
  <div class="flex w-full mb-6 overflow-hidden" :class="[{ 'flex-row-reverse': inversion }]">
    <div
      class="flex items-center justify-center flex-shrink-0 h-8 overflow-hidden rounded-full basis-8"
      :class="[inversion ? 'ml-2' : 'mr-2']"
    >
      <AvatarComponent :image="inversion" />
    </div>
    <div class="overflow-hidden text-sm " :class="[inversion ? 'items-end' : 'items-start']">
      <p class="text-xs text-[#b4bbc4]" :class="[inversion ? 'text-right' : 'text-left']">
        {{ dateTime }}
        <button class="transition text-neutral-300 hover:text-neutral-800 dark:hover:text-neutral-200"
          @click="handleRead">
          {{ readButtonText }}
        </button>
      </p>
      <div
        class="flex items-end gap-1 mt-2"
        :class="[inversion ? 'flex-row-reverse' : 'flex-row']"
      >
        <TextComponent
          ref="textRef"
          :inversion="inversion"
          :error="error"
          :text="text"
          :loading="loading"
        />
        <div class="flex flex-col">
          <button
            v-if="!inversion"
            class="mb-2 transition text-neutral-300 hover:text-neutral-800 dark:hover:text-neutral-300"
            @click="handleRegenerate"
          >
            <SvgIcon icon="ri:restart-line" />
          </button>
          <NDropdown :placement="!inversion ? 'right' : 'left'" :options="options" @select="handleSelect">
            <button class="transition text-neutral-300 hover:text-neutral-800 dark:hover:text-neutral-200">
              <SvgIcon icon="ri:more-2-fill" />
            </button>
          </NDropdown>
        </div>
      </div>
    </div>
  </div>
</template>
