<template>
  <div class="wrapper">
    <div class="card">
      <h1>API Tester</h1>
      <div class="field">
        <label>Your Email Address</label>
        <input
          v-model="email"
          type="email"
          placeholder="email@gmail.com"
          maxlength="254"
          @input="sanitizeEmail"
        />
        <span class="error" v-if="errors.email">{{ errors.email }}</span>
      </div>

      <div class="field">
        <label>Your API Endpoint URL</label>
        <input
          v-model="apiUrl"
          type="url"
          placeholder="YOUR_API_ENDPOINT_URL"
          maxlength="500"
          @input="sanitizeUrl"
        />
        <span class="error" v-if="errors.apiUrl">{{ errors.apiUrl }}</span>
      </div>

      <button :disabled="loading" @click="testEndpoint">
        {{ loading ? 'Testing...' : 'Test Endpoint' }}
      </button>

      <div v-if="result" :class="['result', resultIsError ? 'error' : 'success']">
        <pre>{{ result }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import axios from 'axios'

const email = ref('')
const apiUrl = ref('')
const loading = ref(false)
const result = ref(null)
const resultIsError = ref(false)

const errors = reactive({
  email: '',
  apiUrl: ''
})

const sanitizeEmail = () => {
  email.value = email.value.replace(/[<>'"]/g, '')
  errors.email = ''
}

const sanitizeUrl = () => {
  apiUrl.value = apiUrl.value.replace(/[<>'"]/g, '')
  errors.apiUrl = ''
}

const validate = () => {
  let valid = true

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!email.value) {
    errors.email = 'Email is required'
    valid = false
  } else if (!emailRegex.test(email.value)) {
    errors.email = 'Please enter a valid email address'
    valid = false
  }

  if (!apiUrl.value) {
    errors.apiUrl = 'API URL is required'
    valid = false
  } else if (!apiUrl.value.startsWith('https://')) {
    errors.apiUrl = 'URL must start with https://'
    valid = false
  }

  return valid
}

const testEndpoint = async () => {
  result.value = null
  resultIsError.value = false

  if (!validate()) return

  loading.value = true

  const testUrl = `https://yhxzjyykdsfkdrmdxgho.supabase.co/functions/v1/application-task?url=${encodeURIComponent(apiUrl.value)}&email=${encodeURIComponent(email.value)}`

  try {
    const res = await axios.get(testUrl);
    result.value = res.data.message || 'No response message'

  } catch (err) {
    result.value = 'Request failed. Please check your URL and try again.'
    resultIsError.value = true
  } finally {
    loading.value = false
  }
}
</script>


<style scoped>
* { box-sizing: border-box; margin: 0; padding: 0; }

.wrapper {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.card {
  border-radius: 16px;
  padding: 40px;
  width: 100%;
  max-width: 500px;
}

h1 {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 8px;
}

.subtitle {
  font-size: 14px;
  margin-bottom: 32px;
}

.field { margin-bottom: 20px; }

label {
  display: block;
  font-size: 13px;
  margin-bottom: 6px;
}

input {
  width: 100%;
  border-radius: 8px;
  padding: 12px 16px;
  font-size: 14px;
  outline: none;
  transition: border-color 0.2s;
  font-family: inherit;
}

.error {
  display: block;
  font-size: 12px;
  margin-top: 4px;
}

button {
  width: 100%;
  border: none;
  border-radius: 8px;
  padding: 14px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.2s;
  font-family: inherit;
}

button:hover { opacity: 0.9; }
button:disabled { opacity: 0.5; cursor: not-allowed; }

.result {
  margin-top: 24px;
  border-radius: 8px;
  padding: 16px;
}

.result h3 {
  font-size: 13px;
  color: #aaa;
  margin-bottom: 8px;
}

.result pre {
  font-size: 13px;
  white-space: pre-wrap;
  word-break: break-all;
}
</style>
