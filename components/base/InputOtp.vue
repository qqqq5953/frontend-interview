<script setup lang="ts">
import { ref, watch, computed } from 'vue'

const props = defineProps({
  length: {
    type: Number,
    default: 6
  },
  columns: {
    type: Number,
    default: 6,
    validator: (value: number) => {
      // 注意：validator 中无法直接访问其他 props，所以需要从外部计算
      // 这里先做基本验证，详细的验证在 computed 中处理
      if (value < 1) {
        return false
      }
      return true
    }
  },
  isDisabled: {
    type: Boolean,
    default: false
  },
  error: {
    type: Boolean,
    default: false
  },
  errorMessage: {
    type: String,
    default: ''
  },
  modelValue: {
    type: String,
    default: ''
  }
})

const emit = defineEmits<{(e: 'update:modelValue', value: string): void}>()

// otp input
const otpLength = computed(() => clampLength(props.length))
const numbers = ref<string[]>(Array(otpLength.value).fill(''))

// 计算列数，确保不超过 otpLength
const columns = computed(() => {
  const maxColumns = otpLength.value
  const requestedColumns = Math.max(1, Math.floor(props.columns || maxColumns))

  if (process.dev && requestedColumns > maxColumns) {
    console.warn(
      `[InputOtp] columns (${props.columns}) 不能大于 length (${maxColumns})。columns 将被限制为 ${maxColumns}。`
    )
  }

  return Math.min(requestedColumns, maxColumns)
})

// 计算 grid 样式
const gridStyle = computed(() => {
  return {
    gridTemplateColumns: `repeat(${columns.value}, 1fr)`
  }
})

// 同步外部传入的 modelValue 到内部 numbers
watch(() => props.modelValue, (newValue) => {
  if (newValue) {
    const nums = newValue.replace(/\D/g, '').slice(0, otpLength.value)
    nums.split('').forEach((num, index) => {
      numbers.value[index] = num
    })
    // 填充剩余位置为空字符串
    for (let i = nums.length; i < otpLength.value; i++) {
      numbers.value[i] = ''
    }
  } else {
    numbers.value = Array(otpLength.value).fill('')
  }
}, { immediate: true })

watchEffect(() => {
  const value = numbers.value.join('')
  emit('update:modelValue', value)
})

function clampLength (length: number) {
  return Math.min(8, Math.max(4, Math.floor(length)))
}

function removeNonDigits (value: string) {
  return value.replace(/\D/g, '')
}

function onInput (e: Event, index: number) {
  const target = e.target as HTMLInputElement
  const validValue = removeNonDigits(target.value)
  const newValue = validValue.slice(-1)
  target.value = newValue
  numbers.value[index] = newValue

  if (newValue && index < numbers.value.length - 1) {
    onFocus(index + 1)
  }
}

function onKeydown (e: KeyboardEvent, index: number) {
  if (e.key === 'Backspace') {
    const target = e.target as HTMLInputElement
    if (!target.value && index > 0) {
      e.preventDefault()
      onFocus(index - 1)
    }
  }
}

function onPaste (e: ClipboardEvent) {
  e.preventDefault()
  const text = e.clipboardData?.getData('text') ?? ''
  const nums = removeNonDigits(text).slice(0, otpLength.value)

  nums.split('').forEach((num, index) => {
    numbers.value[index] = num
  })

  onFocus(nums.length - 1)
}

// focus
const inputs = ref<HTMLInputElement[]>([])
const activeIndex = ref(0)

function onFocus (index: number) {
  activeIndex.value = index
}

watch(activeIndex, (i) => {
  inputs.value[i]?.focus()
}, { flush: 'post' })

</script>

<template>
  <div class="flex flex-col items-center gap-2 py-2">
    <div
      class="grid gap-2"
      :style="gridStyle"
    >
      <input
        v-for="(_, i) in otpLength"
        :key="i"
        ref="inputs"
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
