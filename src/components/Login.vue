<script setup>
import { ref } from 'vue'
import axios from 'axios'

const emit = defineEmits(['login-success'])

const username = ref('')
const password = ref('')
const error = ref('')
const loading = ref(false)
const rememberMe = ref(false)

const handleLogin = async () => {
  loading.value = true
  error.value = ''

  try {
    const response = await axios.post('http://localhost:8000/api/login', {
      username: username.value,
      password: password.value
    })

    if (response.data.message === 'Login successful') {
      emit('login-success', response.data.user, rememberMe.value)
    }
  } catch (e) {
    if (e.response && e.response.status === 401) {
      error.value = 'Invalid credentials'
    } else {
      error.value = 'An error occurred. Please try again.'
      console.error(e)
    }
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <section class="bg-gray-50 dark:bg-gray-900">
    <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto md:h-screen lg:py-0">
        <div class="w-full bg-white rounded-lg shadow dark:border md:mt-0 sm:max-w-md xl:p-0 dark:bg-gray-800 dark:border-gray-700">
            <div class="p-6 space-y-4 md:space-y-6 sm:p-8">
                <h1 class="text-xl font-bold leading-tight tracking-tight text-gray-900 md:text-2xl dark:text-white">
                    Sign in to your account
                </h1>
                <form class="space-y-4 md:space-y-6" @submit.prevent="handleLogin">
                    <div>
                        <label for="username" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Username</label>
                        <input 
                          type="text" 
                          name="username" 
                          id="username" 
                          v-model="username"
                          class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" 
                          placeholder="vogar" 
                          required
                        >
                    </div>
                    <div>
                        <label for="password" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
                        <input 
                          type="password" 
                          name="password" 
                          id="password" 
                          v-model="password"
                          placeholder="••••••••" 
                          class="bg-gray-50 border border-gray-300 text-gray-900 rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" 
                          required
                        >
                    </div>
                    <div class="flex items-center justify-between">
                        <div class="flex items-start">
                            <div class="flex items-center h-5">
                              <input 
                                id="remember" 
                                aria-describedby="remember" 
                                type="checkbox" 
                                v-model="rememberMe"
                                class="w-4 h-4 border border-gray-300 rounded bg-gray-50 focus:ring-3 focus:ring-primary-300 dark:bg-gray-700 dark:border-gray-600 dark:focus:ring-primary-600 dark:ring-offset-gray-800"
                              >
                            </div>
                            <div class="ml-3 text-sm">
                              <label for="remember" class="text-gray-500 dark:text-gray-300">Remember me</label>
                            </div>
                        </div>
                    </div>
                    <div v-if="error" class="text-red-500 text-sm text-center">
                      {{ error }}
                    </div>
                    <button 
                      type="submit" 
                      :disabled="loading"
                      class="w-full text-white bg-primary-600 hover:bg-primary-700 focus:ring-4 focus:outline-none focus:ring-primary-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-primary-600 dark:hover:bg-primary-700 dark:focus:ring-primary-800 disabled:opacity-50"
                    >
                      <span v-if="loading">Signing in...</span>
                      <span v-else>Sign in</span>
                    </button>
                </form>
            </div>
        </div>
    </div>
  </section>
</template>
