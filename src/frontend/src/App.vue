<template>
  <div class="flex flex-col min-h-screen">
    <AppHeader :isDark="isDark" :isContact="showContact" @toggleDark="toggleDark" @goContact="showContact = true" @goHome="showContact = false" />
    <AppMain v-if="!showContact" />
    <AppContact v-else />
    <AppFooter />
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import AppHeader from './components/AppHeader.vue'
import AppMain from './components/AppMain.vue'
import AppContact from './components/AppContact.vue'
import AppFooter from './components/AppFooter.vue'

const isDark = ref(false)
const showContact = ref(false)

onMounted(() => {
  const savedTheme = localStorage.getItem('theme')
  if (savedTheme === 'dark' || (!savedTheme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    isDark.value = true
    document.documentElement.classList.add('dark')
  }
})

watch(isDark, (newVal) => {
  if (newVal) {
    document.documentElement.classList.add('dark')
    localStorage.setItem('theme', 'dark')
  } else {
    document.documentElement.classList.remove('dark')
    localStorage.setItem('theme', 'light')
  }
})

const toggleDark = () => {
  isDark.value = !isDark.value
}
</script>
