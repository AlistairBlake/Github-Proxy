<template>
  <main class="flex-1 flex items-start justify-center py-8 px-4 sm:px-6 lg:px-8 bg-gradient-to-br from-gray-100 to-gray-200 dark:from-gray-900 dark:to-gray-950 transition-colors duration-300">
    <div class="w-full max-w-[1000px] mx-auto">
      <div class="pt-6">
        <button @click="$emit('back')" class="inline-flex items-center gap-2 text-blue-600 hover:text-blue-700 dark:text-blue-400 dark:hover:text-blue-300 font-medium transition-colors mb-6">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="m15 18-6-6 6-6"></path>
          </svg>
          返回首页
        </button>

        <div class="flex items-center justify-between mb-6">
          <div>
            <h1 class="font-bold text-3xl text-gray-900 dark:text-white">搜索结果</h1>
            <p class="text-gray-600 dark:text-gray-400 text-sm mt-1">搜索关键词: "{{ searchQuery }}"</p>
          </div>
        </div>

        <div v-if="loadingSearch" class="text-center py-12">
          <svg class="animate-spin h-8 w-8 text-blue-600 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <p class="mt-4 text-gray-600 dark:text-gray-400">搜索仓库中...</p>
        </div>

        <div v-else-if="searchError" class="bg-red-50 dark:bg-red-900/20 border border-red-200 dark:border-red-800 rounded-xl p-6 text-center">
          <svg class="h-12 w-12 text-red-500 mx-auto mb-3" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <p class="text-red-700 dark:text-red-400">{{ searchError }}</p>
        </div>

        <div v-else-if="searchResults.length === 0" class="text-center py-8">
          <p class="text-gray-600 dark:text-gray-400">未找到匹配的仓库</p>
        </div>

        <div v-else class="space-y-4">
          <div class="flex items-center justify-between mb-4">
            <h2 class="text-xl font-bold text-gray-900 dark:text-white">仓库列表</h2>
            <span class="text-sm text-gray-500 dark:text-gray-400">共 {{ searchResults.length }} 个仓库</span>
          </div>

          <div v-for="repo in searchResults" :key="repo.id" class="bg-white/60 dark:bg-gray-800/60 backdrop-blur-sm border border-gray-200/50 dark:border-gray-700/50 rounded-xl p-6 hover:shadow-md transition-shadow">
            <div class="flex items-start justify-between">
              <div class="flex-1">
                <div class="flex items-center gap-3 mb-2">
                  <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" class="text-gray-700 dark:text-gray-300">
                    <path d="M9 19c-5 1.5-5-2.5-7-3m14 6v-3.87a3.37 3.37 0 0 0-.94-2.61c3.14-.35 6.44-1.54 6.44-7A5.44 5.44 0 0 0 20 4.77 5.07 5.07 0 0 0 19.91 1S18.73.65 16 2.48a13.38 13.38 0 0 0-7 0C6.27.65 5.09 1 5.09 1A5.07 5.07 0 0 0 5 4.77a5.44 5.44 0 0 0-1.5 3.78c0 5.42 3.3 6.61 6.44 7A3.37 3.37 0 0 0 9 18.13V22"></path>
                  </svg>
                  <a :href="repo.html_url" target="_blank" rel="noopener noreferrer" class="text-xl font-semibold text-blue-600 hover:text-blue-700 dark:text-blue-400 dark:hover:text-blue-300">
                    {{ repo.full_name }}
                  </a>
                </div>
                <p class="text-sm text-gray-600 dark:text-gray-400 mb-3">{{ repo.description || '暂无描述' }}</p>
                <div class="flex items-center gap-4 text-xs text-gray-500 dark:text-gray-400">
                  <span v-if="repo.language" class="flex items-center gap-1">
                    <span class="w-3 h-3 rounded-full bg-blue-500"></span>
                    {{ repo.language }}
                  </span>
                  <span class="flex items-center gap-1">
                    <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 12 17.77 7 14.14 2 9.27 8.91 8.26 12 2"></polygon>
                    </svg>
                    {{ formatStarCount(repo.stargazers_count) }}
                  </span>
                  <span class="flex items-center gap-1">
                    <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                      <circle cx="12" cy="18" r="3"></circle>
                      <circle cx="6" cy="6" r="3"></circle>
                      <circle cx="18" cy="6" r="3"></circle>
                      <line x1="8.7" y1="14.7" x2="15.3" y2="9.3"></line>
                      <line x1="15.3" y1="14.7" x2="8.7" y2="9.3"></line>
                    </svg>
                    {{ repo.forks_count }}
                  </span>
                  <span>更新于 {{ formatDate(repo.updated_at) }}</span>
                </div>
              </div>
              <div class="flex flex-col gap-2 ml-4">
                <button
                  @click="downloadRepoZip(repo)"
                  :disabled="!selectedNode"
                  class="px-4 py-2 bg-blue-600 hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed text-white text-sm font-medium rounded-lg transition-colors whitespace-nowrap"
                >
                  下载 ZIP
                </button>
                <button
                  @click="viewReleases(repo)"
                  class="px-4 py-2 border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 text-sm font-medium rounded-lg transition-colors whitespace-nowrap"
                >
                  Releases
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  searchQuery: String,
  selectedNode: Object,
  getNodeUrl: Function
})

const emit = defineEmits(['back', 'view-releases'])

const searchResults = ref([])
const loadingSearch = ref(false)
const searchError = ref('')

onMounted(() => {
  if (props.searchQuery) {
    searchRepositories()
  }
})

const searchRepositories = async () => {
  if (!props.searchQuery.trim() || !props.selectedNode) return

  loadingSearch.value = true
  searchError.value = ''
  searchResults.value = []

  try {
    let query = props.searchQuery.trim()

    if (!query.includes('/') && !query.startsWith('http')) {
      query = query + ' in:name'
    } else if (query.includes('/') && !query.startsWith('http')) {
      const parts = query.split('/').filter(p => p.trim())
      if (parts.length === 2) {
        query = `user:${parts[0]} ${parts[1]} in:name`
      } else if (parts.length === 1) {
        query = `${parts[0]} in:name`
      }
    }

    const apiPath = `search/repositories?q=${encodeURIComponent(query)}&per_page=10&sort=stars&order=desc`
    const proxyUrl = '/' + `https://api.github.com/${apiPath}`

    const response = await fetch(proxyUrl)

    if (!response.ok) {
      throw new Error(`搜索失败: ${response.status} ${response.statusText}`)
    }

    const data = await response.json()

    if (data.items && Array.isArray(data.items)) {
      searchResults.value = data.items.map(item => ({
        id: item.id,
        full_name: item.full_name,
        description: item.description,
        html_url: item.html_url,
        stargazers_count: item.stargazers_count,
        forks_count: item.forks_count,
        language: item.language,
        updated_at: item.updated_at
      }))

      if (searchResults.value.length === 0) {
        searchError.value = '未找到匹配的仓库'
      }
    } else {
      throw new Error(data.message || '搜索失败')
    }
  } catch (error) {
    console.error('搜索仓库失败:', error)
    searchError.value = error.message || '搜索失败，请稍后重试'
  } finally {
    loadingSearch.value = false
  }
}

const downloadRepoZip = async (repo) => {
  if (!props.selectedNode || !repo.html_url) return

  const parts = repo.html_url.split('/').filter(p => p)
  if (parts.length < 4) return

  const owner = parts[2]
  const repoName = parts[3]

  if (!owner || !repoName) return

  let branch = 'main'
  try {
    const resp = await fetch(`/api/repo/${owner}/${repoName}/branch`)
    if (resp.ok) {
      const data = await resp.json()
      branch = data.branch || 'main'
    }
  } catch (e) {
    console.error('Failed to fetch default branch, using main:', e)
  }

  const zipUrl = `https://github.com/${owner}/${repoName}/archive/refs/heads/${branch}.zip`
  const proxyUrl = props.getNodeUrl(props.selectedNode) + '/' + zipUrl
  window.open(proxyUrl, '_blank')
}

const viewReleases = (repo) => {
  emit('view-releases', repo.html_url)
}

const formatDate = (dateString) => {
  if (!dateString) return '未知'
  const date = new Date(dateString)
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const formatStarCount = (count) => {
  if (!count && count !== 0) return '0'
  if (count >= 1000000) {
    return (count / 1000000).toFixed(1) + 'M'
  }
  if (count >= 1000) {
    return (count / 1000).toFixed(1) + 'K'
  }
  return count.toString()
}
</script>
