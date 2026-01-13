<script setup lang="ts">
type VerifyOtpResponse = {
  success: boolean
  data: {
    verified: boolean
    timestamp: string
  }
}

const error = ref(false)
const errorMessage = ref('')
const otpValue = ref('')
const http = useHttp()

watch(otpValue, async (newValue) => {
  // 只有当 OTP 完整时才进行验证
  if (newValue.length === 6) {
    try {
      const data = await http.post('/examples/verify-otp-simple', {
        body: { otp: newValue }
      }) as VerifyOtpResponse

      error.value = !data.success
      errorMessage.value = data.success ? '' : '驗證碼錯誤'
    } catch (err) {
      error.value = true
      errorMessage.value = '驗證碼錯誤'
    }
  } else {
    // 如果 OTP 不完整，清除错误状态
    error.value = false
    errorMessage.value = ''
  }
})
</script>

<template>
  <BaseInputOtp
    v-model="otpValue"
    :length="6"
    :error="error"
    :error-message="errorMessage"
    :columns="3"
  />
</template>
