<template>
  <main class="flex-1 py-8 px-4 sm:px-6 lg:px-8 bg-gray-50 dark:bg-gray-950 transition-colors duration-300">
    <!-- 主页：搜索入口 -->
    <div v-if="currentView === 'home'" class="w-full max-w-[1000px] mx-auto">
      <div class="pt-10">
        <div class="text-center w-full mb-12">
          <h1 class="font-bold mb-4 text-6xl text-gray-900 dark:text-white transition-colors duration-300">Github <span class="text-blue-600">Proxy</span></h1>
          <p class="text-gray-600 dark:text-gray-400 text-base">支持 API、Git Clone、Releases、Archive、Gist、Raw 等资源加速下载，提升 GitHub 文件下载体验。</p>
        </div>

        <div class="w-full relative z-[10001] mb-8">
          <div class="flex flex-col sm:flex-row gap-3 sm:items-start">
            <div class="flex-1 relative">
              <input
                type="text"
                v-model="githubUrl"
                @keyup.enter="handleAction"
                :placeholder="inputPlaceholder"
                class="w-full px-4 py-3 bg-white dark:bg-gray-800 text-gray-900 dark:text-gray-100 border rounded-lg focus:outline-none focus:ring-2 transition-colors duration-300 placeholder:text-gray-500 dark:placeholder:text-gray-400 h-[48px] border-gray-300 dark:border-gray-700 focus:ring-blue-500" />
            </div>
            <button
              @click="handleAction"
              :disabled="!isValidUrl"
              class="px-6 py-3 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-medium transition-colors whitespace-nowrap disabled:opacity-50 disabled:cursor-not-allowed shrink-0 h-[48px]">
              {{ actionButtonText }}
            </button>
          </div>
        </div>
      </div>

      <div>
        <div class="relative z-[9999]">
          <div v-if="isSharedMode" class="mb-3 flex items-center gap-2">
            <span class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-medium bg-green-100 dark:bg-green-900/30 text-green-700 dark:text-green-400">
              <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M8 12h8"/><path d="M12 8v8"/></svg>
              共享模式 - 已连接到节点调度中心
            </span>
          </div>
          <div class="flex flex-col md:flex-row md:items-center gap-4 mb-6">
            <span class="hidden md:block text-gray-700 dark:text-gray-300 font-medium whitespace-nowrap">节点选择：</span>
            <div class="relative w-full md:flex-1 md:max-w-xl">
              <button
                type="button"
                @click="toggleNodeList"
                :disabled="isLoadingNodes"
                class="w-full px-4 py-3 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-700 rounded-lg focus:outline-none transition-all text-left disabled:opacity-50 disabled:cursor-not-allowed h-[48px] flex items-center justify-between">
                <div class="flex items-center gap-2 text-gray-500 dark:text-gray-400 w-full">
                  <svg v-if="isLoadingNodes" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-5 h-5 animate-spin" aria-hidden="true">
                    <path d="M21 12a9 9 0 1 1-6.219-8.56"></path>
                  </svg>
                  <span>{{ selectedNode ? selectedNode.name : (isLoadingNodes ? '加载节点列表中...' : '选择节点') }}</span>
                </div>
                <svg v-if="!isLoadingNodes" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-gray-500 dark:text-gray-400 flex-shrink-0" :class="{ 'rotate-180': showNodeList }">
                  <path d="m6 9 6 6 6-6"></path>
                </svg>
              </button>
              <div v-if="showNodeList && !isLoadingNodes" class="absolute z-10 mt-1 w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-700 rounded-lg shadow-lg max-h-80 overflow-auto">
                <div v-for="node in displayedNodes" :key="node.id"
                  @click="selectNode(node)"
                  class="px-4 py-3 hover:bg-gray-100 dark:hover:bg-gray-700 cursor-pointer flex items-center justify-between transition-colors"
                  :class="{ 'bg-blue-50 dark:bg-blue-900/20': selectedNode?.id === node.id }">
                  <div class="flex flex-col">
                    <span class="text-gray-900 dark:text-gray-100 font-medium">{{ node.name }}</span>
                    <div class="flex items-center gap-2 mt-0.5">
                      <span v-if="node.isLocal" class="text-xs text-blue-500">推荐 · 本地节点</span>
                      <span v-if="node.isNew" class="text-xs text-green-500">新节点</span>
                      <span v-if="node.isBusy" class="text-xs text-orange-500">高负载中</span>
                    </div>
                  </div>
                  <div class="text-right text-xs text-gray-500 dark:text-gray-400 whitespace-nowrap ml-2">
                    <span v-if="node.score !== null && !node.isLocal">评分: {{ node.score }}</span>
                    <span v-if="node.latency !== null">{{ typeof node.latency === 'number' ? node.latency + 'ms' : node.latency }}</span>
                    <span v-if="node.speed"> | {{ formatSpeed(node.speed) }}</span>
                  </div>
                </div>
                <button
                  v-if="nodes.length > 5"
                  @click="toggleShowAllNodes"
                  class="w-full px-4 py-2 text-sm text-blue-600 dark:text-blue-400 hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors border-t border-gray-200 dark:border-gray-700">
                  {{ showAllNodes ? '收起' : `显示全部 ${nodes.length} 个节点` }}
                </button>
              </div>
            </div>
            <button
              type="button"
              @click="speedTest"
              :disabled="isLoadingNodes || nodes.length === 0"
              class="w-full md:w-auto flex items-center justify-center gap-2 px-4 py-3 rounded-lg border transition-all whitespace-nowrap bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-100 border-gray-300 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-700 disabled:opacity-50 disabled:cursor-not-allowed h-[48px]"
              title="开始节点测速">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-4 h-4" aria-hidden="true">
                <path d="m12 14 4-4"></path>
                <path d="M3.34 19a10 10 0 1 1 17.32 0"></path>
              </svg>
              <span class="text-sm font-medium">节点测速</span>
            </button>
            <button
              type="button"
              @click="toggleReleasesMode"
              class="w-full md:w-auto flex items-center justify-center md:justify-start gap-2 px-4 py-3 rounded-lg border transition-all whitespace-nowrap h-[48px] bg-white dark:bg-gray-800 text-gray-700 dark:text-gray-100 border-gray-300 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-700">
              <div class="w-10 h-6 rounded-full transition-all relative" :class="isReleasesMode ? 'bg-blue-600' : 'bg-gray-300 dark:bg-gray-600'">
                <div class="absolute top-0.5 w-5 h-5 bg-white rounded-full transition-transform duration-300" :class="isReleasesMode ? 'translate-x-4' : 'translate-x-0.5'"></div>
              </div>
              <span class="text-sm font-medium">获取Releases列表</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Releases 列表页面 -->
    <AppReleases
      v-else-if="currentView === 'releases'"
      :repoUrl="currentRepoUrl"
      :selectedNode="selectedNode"
      :getNodeUrl="getNodeUrl"
      :fromView="previousView"
      @back="goBack"
    />

    <!-- 搜索结果页面 -->
    <AppSearchResults
      v-else-if="currentView === 'search'"
      :searchQuery="searchQuery"
      :selectedNode="selectedNode"
      :getNodeUrl="getNodeUrl"
      :cachedData="searchCache"
      @back="goHome"
      @view-releases="handleViewReleasesFromSearch"
      @cache-update="handleSearchCacheUpdate"
    />
  </main>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import AppReleases from './AppReleases.vue'
import AppSearchResults from './AppSearchResults.vue'

const githubUrl = ref('')
const isLoadingNodes = ref(true)
const showNodeList = ref(false)
const showAllNodes = ref(false)
const selectedNode = ref(null)
const isReleasesMode = ref(false)
const nodes = ref([])
const isSharedMode = ref(false)
const wsConnected = ref(false)

// 页面路由状态
const currentView = ref('home')
const previousView = ref('home')
const currentRepoUrl = ref('')
const searchQuery = ref('')
const navigationHistory = ref([])
const searchCache = ref(null)

let nodePollingTimer = null

const inputType = computed(() => {
  const url = githubUrl.value.trim()

  if (!url) return 'empty'

  if (isReleasesMode.value) return 'releases'

  const filePatterns = [
    '/blob/',
    '/raw/',
    '/releases/download/',
    '/archive/',
    '.zip',
    '.tar.gz',
    '.rar',
    '.7z',
    '.exe',
    '.msi',
    '.dmg',
    '.deb',
    '.rpm',
    '.sh',
    '.ps1'
  ]

  const isFileLink = filePatterns.some(pattern => url.toLowerCase().includes(pattern))
  if (isFileLink) return 'file'

  const repoPattern = /^https?:\/\/github\.com\/[^/]+\/[^/]+(?:\.git)?$/
  if (repoPattern.test(url)) return 'repo'

  if (url.includes('/') && !url.startsWith('http')) {
    const parts = url.split('/').filter(p => p.trim())
    if (parts.length >= 1 && parts.length <= 2) return 'search'
  }

  if (!url.startsWith('http') && url.includes('/')) return 'search'

  if (url.match(/^[\w.-]+\/[\w.-]+$/)) return 'search'

  if (url.startsWith('https://github.com') || url.startsWith('https://raw.githubusercontent.com') || url.startsWith('https://api.github.com')) {
    return 'file'
  }

  return 'search'
})

const actionButtonText = computed(() => {
  switch (inputType.value) {
    case 'releases':
      return '查看'
    case 'file':
      return '下载'
    case 'repo':
      return '下载 ZIP'
    case 'search':
      return '搜索仓库'
    default:
      return isReleasesMode.value ? '查看' : 'Go'
  }
})

const inputPlaceholder = computed(() => {
  if (isReleasesMode.value) {
    return '输入 Github 仓库链接 (例如: https://github.com/owner/repo)'
  }
  return '输入 Github 文件/仓库链接，或直接输入 owner/repo 搜索'
})

const isValidUrl = computed(() => {
  if (!githubUrl.value.trim()) return false

  if (isReleasesMode.value) {
    return githubUrl.value.startsWith('https://github.com/') &&
           githubUrl.value.split('/').length >= 5
  }

  const url = githubUrl.value.trim()
  if (url.startsWith('http')) {
    return url.startsWith('https://github.com') ||
           url.startsWith('https://raw.githubusercontent.com') ||
           url.startsWith('https://api.github.com')
  }

  return url.length > 0
})

const displayedNodes = computed(() => {
  if (showAllNodes.value || nodes.value.length <= 5) {
    return nodes.value
  }
  const localNode = nodes.value.find(n => n.isLocal)
  const externalNodes = nodes.value.filter(n => !n.isLocal)
  const result = localNode ? [localNode] : []
  return [...result, ...externalNodes.slice(0, 4)]
})

const loadNodes = async () => {
  isLoadingNodes.value = true
  try {
    const response = await fetch('/api/nodes')
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`)
    }
    const data = await response.json()
    updateNodes(data)
  } catch (error) {
    console.error('加载节点失败:', error)
  } finally {
    isLoadingNodes.value = false
  }
}

const updateNodes = (data) => {
  isSharedMode.value = data.shared
  const oldSelected = selectedNode.value
  nodes.value = (data.nodes || []).map(node => ({
    id: node.id,
    name: node.name,
    url: node.url,
    isLocal: node.isLocal,
    score: node.score,
    isBusy: node.isBusy,
    isNew: node.isNew,
    latency: null,
    speed: null
  }))
  if (oldSelected) {
    const updated = nodes.value.find(n => n.url === oldSelected.url)
    if (updated) {
      updated.latency = oldSelected.latency
      updated.speed = oldSelected.speed
      selectedNode.value = updated
    }
  }
  if (!selectedNode.value) {
    const localNode = nodes.value.find(n => n.isLocal)
    if (localNode) {
      selectedNode.value = localNode
    } else if (nodes.value.length > 0) {
      selectedNode.value = nodes.value[0]
    }
  }
}

const startNodePolling = () => {
  if (nodePollingTimer) clearInterval(nodePollingTimer)
  loadNodes()
  nodePollingTimer = setInterval(loadNodes, 30000)
}

onMounted(() => {
  loadNodes()
  startNodePolling()
})

onUnmounted(() => {
  if (nodePollingTimer) clearInterval(nodePollingTimer)
})

watch(isReleasesMode, (newVal) => {
  // 切换模式时不需要重置视图，保持当前位置
})

const toggleNodeList = () => {
  if (!isLoadingNodes.value) {
    showNodeList.value = !showNodeList.value
  }
}

const selectNode = (node) => {
  selectedNode.value = node
  showNodeList.value = false
}

const toggleShowAllNodes = () => {
  showAllNodes.value = !showAllNodes.value
}

const getNodeUrl = (node) => {
  let url = node.url
  if (!url.startsWith('http://') && !url.startsWith('https://')) {
    url = window.location.protocol + '//' + url
  }
  return url
}

const isValidNodeUrl = (url) => {
  if (!url) return false
  if (!url.startsWith('http://') && !url.startsWith('https://')) {
    url = window.location.protocol + '//' + url
  }
  try {
    const u = new URL(url)
    const host = u.hostname
    return host !== '127.0.0.1' && host !== 'localhost' && host !== '0.0.0.0'
  } catch {
    return false
  }
}

const speedTest = async () => {
  if (nodes.value.length === 0 || isLoadingNodes.value) return

  const validNodes = nodes.value.filter(n => n.isLocal || isValidNodeUrl(n.url))

  const testNode = (node) => {
    return new Promise((resolve) => {
      const startTime = Date.now()
      const testUrl = getNodeUrl(node) + '/favicon.ico'

      const xhr = new XMLHttpRequest()
      xhr.open('GET', testUrl, true)
      xhr.responseType = 'blob'
      xhr.timeout = 8000

      let headersReceived = false
      xhr.onreadystatechange = () => {
        if (xhr.readyState === 2 && !headersReceived) {
          headersReceived = true
          node.latency = Date.now() - startTime
        }
      }

      xhr.onload = () => {
        const totalTime = Date.now() - startTime
        if (!node.latency) node.latency = totalTime

        const blob = xhr.response
        if (blob && blob.size > 0) {
          const sizeInKB = blob.size / 1024
          const timeInSeconds = totalTime / 1000
          const speed = timeInSeconds > 0 ? sizeInKB / timeInSeconds : sizeInKB * 1000
          node.speed = speed
        } else {
          node.speed = 0
        }

        resolve()
      }

      xhr.onerror = () => {
        node.latency = '超时'
        node.speed = null
        resolve()
      }

      xhr.ontimeout = () => {
        node.latency = '超时'
        node.speed = null
        resolve()
      }

      xhr.send()
    })
  }

  const tests = validNodes.map(node => testNode(node))
  await Promise.all(tests)

  nodes.value.sort((a, b) => {
    if (a.isLocal) return -1
    if (b.isLocal) return 1

    if (a.speed === null && b.speed !== null) return 1
    if (b.speed !== null && a.speed === null) return -1
    if (a.speed !== null && b.speed !== null) return b.speed - a.speed

    if (typeof a.latency === 'number' && typeof b.latency === 'number') {
      return a.latency - b.latency
    }
    return 0
  })
}

const formatSpeed = (speed) => {
  if (speed === null || speed === undefined) return ''
  if (speed >= 1024) return (speed / 1024).toFixed(1) + ' MB/s'
  return speed.toFixed(0) + ' KB/s'
}

// 页面导航方法
const navigateTo = (view, options = {}) => {
  if (currentView.value === 'search') {
    searchCache.value = {
      query: searchQuery.value,
      results: window.__searchResultsCache || null,
      totalResults: window.__searchTotalResultsCache || 0,
      currentPage: window.__searchCurrentPageCache || 1,
      sortBy: window.__searchSortByCache || '',
      sortOrder: window.__searchSortOrderCache || 'desc',
      searchScope: window.__searchScopeCache || 'all'
    }
  }

  if (currentView.value !== 'home') {
    navigationHistory.value.push({
      view: currentView.value,
      repoUrl: currentRepoUrl.value,
      searchQuery: searchQuery.value
    })
  }

  previousView.value = currentView.value
  currentView.value = view

  if (options.repoUrl !== undefined) {
    currentRepoUrl.value = options.repoUrl
  }
  if (options.searchQuery !== undefined) {
    searchQuery.value = options.searchQuery
  }
}

const goBack = () => {
  if (navigationHistory.value.length > 0) {
    const previousState = navigationHistory.value.pop()
    previousView.value = previousState.view === 'home' ? 'home' : currentView.value
    currentView.value = previousState.view
    currentRepoUrl.value = previousState.repoUrl || ''
    searchQuery.value = previousState.searchQuery || ''
  } else {
    goHome()
  }
}

const goHome = () => {
  currentView.value = 'home'
  previousView.value = 'home'
  currentRepoUrl.value = ''
  searchQuery.value = ''
  navigationHistory.value = []
}

const handleAction = () => {
  if (!isValidUrl.value) return

  switch (inputType.value) {
    case 'releases':
      navigateTo('releases', { repoUrl: githubUrl.value.trim() })
      break
    case 'file':
      downloadFile()
      break
    case 'repo':
      downloadRepoZip()
      break
    case 'search':
      navigateTo('search', { searchQuery: githubUrl.value.trim() })
      break
    default:
      if (isReleasesMode.value) {
        navigateTo('releases', { repoUrl: githubUrl.value.trim() })
      } else {
        downloadFile()
      }
  }
}

const downloadFile = () => {
  if (!selectedNode.value) return
  const proxyUrl = getNodeUrl(selectedNode.value) + '/' + githubUrl.value.trim()
  window.open(proxyUrl, '_blank')
}

const downloadRepoZip = async () => {
  if (!selectedNode.value || !githubUrl.value) return

  const repoUrl = githubUrl.value.replace('.git', '').trim()
  const parts = repoUrl.split('/').filter(p => p)

  if (parts.length < 4) return

  const owner = parts[2]
  const repo = parts[3]

  if (!owner || !repo) return

  let branch = 'main'
  try {
    const resp = await fetch(`/api/repo/${owner}/${repo}/branch`)
    if (resp.ok) {
      const data = await resp.json()
      branch = data.branch || 'main'
    }
  } catch (e) {
    console.error('Failed to fetch default branch, using main:', e)
  }

  const zipUrl = `https://github.com/${owner}/${repo}/archive/refs/heads/${branch}.zip`
  const proxyUrl = getNodeUrl(selectedNode.value) + '/' + zipUrl
  window.open(proxyUrl, '_blank')
}

const handleViewReleasesFromSearch = (repoUrl) => {
  navigateTo('releases', { repoUrl: repoUrl })
}

const handleSearchCacheUpdate = (cacheData) => {
  searchCache.value = cacheData
}

const toggleReleasesMode = () => {
  isReleasesMode.value = !isReleasesMode.value
}

document.addEventListener('click', (e) => {
  if (!e.target.closest('.relative.z-\\[9999\\]')) {
    showNodeList.value = false
  }
})
</script>
