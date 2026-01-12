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
const http = useHttp()

async function handleComplete (code: string) {
  try {
    const data = await http.post('/examples/verify-otp-simple', {
      body: { otp: code }
    }) as VerifyOtpResponse

    error.value = !data.success
    errorMessage.value = data.success ? '' : '驗證碼錯誤'
  } catch (err) {
    error.value = true
    errorMessage.value = '驗證碼錯誤'
  }
}
</script>

<template>
  <BaseInputOtp
    :length="6"
    :error="error"
    :error-message="errorMessage"
    @complete="handleComplete"
  />
</template>
