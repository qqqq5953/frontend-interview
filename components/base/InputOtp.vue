<script setup lang="ts">
import { ref, watch, computed } from 'vue'

  type Props = {
      length?: number
      isDisabled?: boolean
      error?: boolean
      errorMessage?: string
  }

const props = withDefaults(defineProps<Props>(), {
  length: 6,
  isDisabled: false,
  error: false,
  errorMessage: ''
})

const emit = defineEmits<{(e: 'complete', value: string): void }>()

// otp input
const otpLength = computed(() => Math.min(8, Math.max(4, Math.floor(Number(props.length)))))
const numbers = ref<string[]>(Array(otpLength.value).fill(''))
const isComplete = computed(() => numbers.value.every(v => v !== ''))

watch(isComplete, (done) => {
  if (done) {
    emit('complete', numbers.value.join(''))
  }
})

function onInput (e: Event, index: number) {
  const target = e.target as HTMLInputElement
  const validValue = target.value.replace(/\D/g, '')
  const newValue = validValue.slice(-1)
  target.value = newValue
  numbers.value[index] = newValue

  if (newValue && index < numbers.value.length - 1) {
    activeIndex.value = index + 1
  }
}

function onKeydown (e: KeyboardEvent, index: number) {
  if (e.key === 'Backspace') {
    const target = e.target as HTMLInputElement
    if (!target.value && index > 0) {
      e.preventDefault()
      activeIndex.value = index - 1
    }
  }
}

function onPaste (e: ClipboardEvent) {
  e.preventDefault()
  const text = e.clipboardData?.getData('text') ?? ''
  const nums = text.replace(/\D/g, '').slice(0, otpLength.value)

  nums.split('').forEach((num, index) => {
    numbers.value[index] = num
  })

  activeIndex.value = nums.length - 1
}

// focus
const inputs = ref<HTMLInputElement[]>([])
const activeIndex = ref(0)

function setInputRef (el: HTMLInputElement | null, i: number) {
  if (el) {
    inputs.value[i] = el
  }
}

function onFocus (index: number) {
  activeIndex.value = index
}

watch(activeIndex, (i) => {
  inputs.value[i]?.focus()
}, { flush: 'post' })

</script>

<template>
  <div class="flex flex-col items-center gap-2 py-2">
    <div class="flex gap-2">
      <input
        v-for="(_, i) in otpLength"
        :key="i"
        :ref="el => setInputRef(el as HTMLInputElement, i)"
        type="text"
        inputmode="numeric"
        maxlength="1"
        class="size-12 rounded border text-center transition
        focus:outline-none focus:ring-2"
        :class="[
          error
            ? 'border-red-500 focus:ring-red-500'
            : 'border-gray-300 focus:ring-blue-500'
        ]"
        :disabled="isDisabled"
        :value="numbers[i]"
        @input="e => onInput(e, i)"
        @keydown="e => onKeydown(e, i)"
        @focus="onFocus(i)"
        @paste="onPaste"
      >
    </div>
    <p
      v-if="error"
      class="mt-2 text-sm text-red-500"
    >
      {{ errorMessage }}
    </p>
  </div>
</template>
