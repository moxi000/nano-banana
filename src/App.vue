<template>
    <div class="min-h-screen bg-gradient-to-br from-yellow-200 via-yellow-300 to-orange-200 text-gray-900 relative overflow-hidden">
        <!-- 香蕉装饰元素 -->
        <div class="absolute top-10 left-10 text-6xl opacity-20 animate-bounce">🍌</div>
        <div class="absolute top-32 right-20 text-4xl opacity-30 animate-pulse">🍌</div>
        <div class="absolute bottom-20 left-32 text-5xl opacity-25 animate-bounce delay-1000">🍌</div>
        <div class="absolute bottom-40 right-10 text-3xl opacity-20 animate-pulse delay-500">🍌</div>

        <div class="container mx-auto px-3 py-4 relative z-10">
            <!-- Header -->
            <div class="relative mb-6">
                <div class="bg-gradient-to-r from-orange-400 to-yellow-500 rounded-lg p-6 border-4 border-black shadow-lg">
                    <div class="text-center">
                        <h1 class="text-4xl font-black text-white mb-1 flex items-center justify-center gap-2">
                            🍌 Nano<br />
                            <span class="text-yellow-100 text-5xl">Banana</span>
                        </h1>
                        <p class="text-white text-base font-medium">上传你的图片，我来创造艺术！</p>
                    </div>
                </div>
            </div>

            <!-- API设置区域 -->
            <div class="mb-6">
                <div class="flex justify-center">
                    <button
                        @click="showApiSettings = !showApiSettings"
                        :class="[
                            'px-6 py-3 rounded-lg border-4 border-black font-bold text-sm transition-all flex items-center gap-2 shadow-lg',
                            apiKey ? 'bg-green-400 text-white hover:bg-green-500' : 'bg-red-400 text-white hover:bg-red-500 animate-pulse'
                        ]"
                    >
                        <span>🔑</span>
                        <span v-if="!apiKey">需要配置API密钥</span>
                        <span v-else>API密钥已配置</span>
                        <svg :class="['w-4 h-4 transition-transform', showApiSettings ? 'rotate-180' : '']" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                        </svg>
                    </button>
                </div>

                <!-- API设置折叠面板 -->
                <div v-if="showApiSettings" class="mt-4 max-w-2xl mx-auto">
                    <ApiKeyInput
                        v-model="apiKey"
                        v-model:endpoint="apiEndpoint"
                        v-model:model="selectedModel"
                        v-model:apiFormat="apiFormat"
                        :models="modelOptions"
                        :model-loading="isFetchingModels"
                        :model-error="modelsError"
                        @fetch-models="handleFetchModels"
                        @model-picked="handleModelPicked"
                    />
                </div>
            </div>

            <!-- 统一创作面板 -->
            <div class="mb-6">
                <div class="bg-gradient-to-r from-blue-400 via-purple-500 to-pink-500 text-white font-black px-4 py-3 rounded-t-lg border-4 border-black border-b-0 flex items-center justify-between gap-3">
                    <div class="flex items-center gap-2">
                        <span>🍌 多模态创作工坊</span>
                        <span class="px-3 py-1 rounded-full bg-white text-black text-sm border-2 border-black">{{ modeLabel }}</span>
                    </div>
                    <span class="text-sm font-semibold text-white/90">不上传图片=文生图；上传图片=图文生图</span>
                </div>
                <div class="bg-white border-4 border-black border-t-0 rounded-b-lg p-5 shadow-lg flex flex-col gap-4">
                    <div class="grid lg:grid-cols-2 gap-4 items-stretch">
                        <div class="flex flex-col gap-3 h-full bg-white border-4 border-black rounded-lg p-4">
                            <label class="font-bold flex items-center gap-2 text-base">
                                ✏️ 描述你的创意：
                                <span class="px-2 py-1 rounded-full border-2 border-black bg-yellow-100 text-xs text-gray-800">必填</span>
                            </label>
                            <div class="flex-1">
                                <textarea
                                    v-model="promptInput"
                                    placeholder="例如：阳光洒在香蕉形热气球上，漂浮在糖果色的天空中"
                                    class="w-full px-4 py-3 border-2 border-black rounded-lg resize-none focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent h-full min-h-[220px]"
                                />
                            </div>
                            <p class="text-sm text-gray-600 font-medium flex items-center gap-2">
                                <span>💡</span>
                                <span>填写描述后即可生成；若上传图片，则配合提示词进行图文生图。</span>
                            </p>
                        </div>

                        <div class="flex flex-col h-full bg-white border-4 border-black rounded-lg p-4">
                            <div class="font-bold text-base mb-2 flex items-center gap-2">📷 上传参考图片（可选）</div>
                            <div class="flex-1 flex">
                                <ImageUpload v-model="selectedImages" />
                            </div>
                        </div>
                    </div>

                    <div v-if="showAspectRatioSelector || showGemini3ProConfig" class="grid lg:grid-cols-2 gap-4">
                        <AspectRatioSelector
                            v-if="showAspectRatioSelector"
                            v-model="selectedAspectRatio"
                            :model-type="showGemini3ProConfig ? 'gemini-3-pro-image' : 'default'"
                            :image-size="gemini3ImageSize"
                            compact
                        />
                        <Gemini3ProConfig
                            v-if="showGemini3ProConfig"
                            v-model:imageSize="gemini3ImageSize"
                            v-model:enableGoogleSearch="gemini3EnableGoogleSearch"
                            compact
                        />
                    </div>
                </div>
            </div>

            <!-- 批量处理配置 -->
            <div class="mb-6">
                <div class="bg-gradient-to-r from-amber-400 to-lime-400 text-black font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">
                    ⚡ 批量处理 · 配置
                </div>
                <div class="bg-white border-4 border-black border-t-0 rounded-b-lg p-5 shadow-lg space-y-4">
                    <div class="flex items-center justify-between gap-3">
                        <div class="flex flex-col gap-1">
                            <p class="font-bold text-base flex items-center gap-2">🎛️ 手动开启批处理</p>
                            <p class="text-gray-600 text-sm">一次请求也可带多张图，需主动开启批处理模式</p>
                        </div>
                        <button
                            @click="batchEnabled = !batchEnabled"
                            :class="[
                                'w-16 h-9 rounded-full border-4 border-black flex items-center px-1 transition-all shadow-lg',
                                batchEnabled ? 'bg-green-400' : 'bg-gray-300'
                            ]"
                            type="button"
                        >
                            <span
                                :class="[
                                    'w-6 h-6 bg-white border-2 border-black rounded-full transition-all',
                                    batchEnabled ? 'translate-x-7' : ''
                                ]"
                            />
                        </button>
                    </div>

                    <div v-if="batchEnabled" class="space-y-4">
                        <div class="flex flex-wrap gap-3 items-center">
                            <span class="font-bold text-sm text-gray-800">处理方式：</span>
                            <label class="flex items-center gap-2 px-3 py-2 rounded-lg border-2 border-black cursor-pointer bg-orange-100 hover:bg-orange-200 transition">
                                <input v-model="batchMode" type="radio" value="concurrent" class="accent-black" />
                                <span class="font-bold">并发</span>
                            </label>
                            <label class="flex items-center gap-2 px-3 py-2 rounded-lg border-2 border-black cursor-pointer bg-blue-100 hover:bg-blue-200 transition">
                                <input v-model="batchMode" type="radio" value="queue" class="accent-black" />
                                <span class="font-bold">排队</span>
                            </label>
                        </div>

                        <div v-if="batchMode === 'concurrent'" class="space-y-2">
                            <label class="font-bold text-sm text-gray-800 flex items-center gap-2">
                                🔀 并发数量
                                <span class="text-xs text-gray-600">(1-8)</span>
                            </label>
                            <input
                                v-model.number="concurrencyLimit"
                                type="number"
                                min="1"
                                max="8"
                                class="w-full border-2 border-black rounded-lg px-3 py-2 focus:ring-2 focus:ring-orange-400 focus:outline-none"
                            />
                            <p class="text-xs text-gray-600">并发越高越快，但请注意 API 限流。</p>
                            <p class="text-xs text-gray-600">同批并发请求启动间隔固定 0.5 秒。</p>
                        </div>

                        <div v-if="batchMode === 'queue'" class="space-y-2">
                            <label class="font-bold text-sm text-gray-800 flex items-center gap-2">
                                ⏳ 队列冷却时间 (ms)
                            </label>
                            <input
                                v-model.number="queueCooldownMs"
                                type="number"
                                min="0"
                                step="100"
                                class="w-full border-2 border-black rounded-lg px-3 py-2 focus:ring-2 focus:ring-blue-400 focus:outline-none"
                            />
                            <p class="text-xs text-gray-600">每张图片之间的等待时间，避免触发限流。</p>
                        </div>

                        <div v-if="batchMode === 'concurrent'" class="space-y-2">
                            <label class="font-bold text-sm text-gray-800 flex items-center gap-2">
                                🧊 批次冷却时间 (ms)
                            </label>
                            <input
                                v-model.number="batchCooldownMs"
                                type="number"
                                min="0"
                                step="100"
                                class="w-full border-2 border-black rounded-lg px-3 py-2 focus:ring-2 focus:ring-lime-400 focus:outline-none"
                            />
                            <p class="text-xs text-gray-600">一批完成后等待多久再开始下一批（并发模式）。</p>
                        </div>

                        <div class="p-3 rounded-lg border-2 border-dashed border-black bg-yellow-50 text-sm text-gray-800 flex items-start gap-2">
                            <span>💡</span>
                            <p>批处理会按上传顺序逐张生成，并支持批量下载；不开启时则按单次请求处理所有上传图片。</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- 生成按钮 -->
            <div class="mb-6">
                <button
                    @click="handleGenerate"
                    :disabled="!canGenerate"
                    :class="[
                        'w-full px-6 py-4 rounded-lg font-bold text-white text-lg transition-all duration-200 flex items-center justify-center gap-3 border-4 border-black shadow-lg',
                        canGenerate
                            ? 'bg-gradient-to-r from-orange-400 to-yellow-500 hover:from-orange-500 hover:to-yellow-600 hover:-translate-y-1 hover:shadow-xl'
                            : 'bg-gray-400 cursor-not-allowed'
                    ]"
                >
                    <span v-if="!isLoading" class="flex items-center gap-2 text-xl">🍌 施展魔法（{{ modeLabel }}）</span>
                    <span v-else class="flex items-center gap-2 text-xl">🍌 正在施法...</span>
                    <div v-if="isLoading" class="w-8 h-8 border-3 border-white/30 border-t-white rounded-full animate-spin" />
                </button>
            </div>

            <!-- 生成结果区域：全宽 -->
            <div class="w-full">
                <div class="bg-black text-white font-bold px-4 py-2 rounded-t-lg border-4 border-black border-b-0 flex items-center gap-2">✨ 生成结果</div>
                <ResultDisplay
                    :result="displayResult"
                    :result-list="displayResultList"
                    :loading="displayLoading"
                    :batch-processing="isBatchProcessing"
                    :error="displayError"
                    :can-push="canPushDisplayResult"
                    @download="handleDownloadResult"
                    @download-item="handleDownloadItem"
                    @download-all="handleDownloadAll"
                    @push="handlePushDisplayResult"
                    @push-item="handlePushItem"
                    @retry-item="handleRetryItem"
                    @update-item-prompt="handleUpdateItemPrompt"
                />
            </div>

            <!-- Footer -->
            <Footer />
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import ApiKeyInput from './components/ApiKeyInput.vue'
import ImageUpload from './components/ImageUpload.vue'
import ResultDisplay from './components/ResultDisplay.vue'
import Footer from './components/Footer.vue'
import AspectRatioSelector from './components/AspectRatioSelector.vue'
import Gemini3ProConfig from './components/Gemini3ProConfig.vue'
import { fetchModels, generateImage } from './services/api'
import { LocalStorage } from './utils/storage'
import type { ApiFormat, ApiModel, BatchResultItem, GenerateRequest, ModelOption } from './types'
import { DEFAULT_API_ENDPOINT, DEFAULT_GEMINI_ENDPOINT, DEFAULT_GEMINI_MODEL, DEFAULT_MODEL_ID } from './config/api'

const apiKey = ref('')
const apiEndpoint = ref('')  // 改为空字符串，避免初始化时触发 watch
const selectedImages = ref<string[]>([])
const promptInput = ref('')
const apiFormat = ref<ApiFormat>('openai')
const isLoading = ref(false)
const result = ref<string | null>(null)
const error = ref<string | null>(null)
const showApiSettings = ref(false)
const modelOptions = ref<ModelOption[]>([])
const selectedModel = ref('')  // 改为空字符串，避免初始化时使用默认值
const isFetchingModels = ref(false)
const modelsError = ref<string | null>(null)
const selectedAspectRatio = ref('1:1')  // 默认宽高比为 1:1
const batchEnabled = ref(false)
const batchMode = ref<'concurrent' | 'queue'>('concurrent')
const concurrencyLimit = ref(2)
const queueCooldownMs = ref(1200)
const batchCooldownMs = ref(1000)
const batchResults = ref<BatchResultItem[]>([])
const currentBatchId = ref('')
const lastBatchCompletedAt = ref<number | null>(null)
let hasSyncedInitialEndpoint = false

// Gemini 3 Pro Image 配置状态
const gemini3ImageSize = ref('2K')  // 默认图像尺寸
const gemini3EnableGoogleSearch = ref(false)  // 默认不启用谷歌搜索

// 组件挂载时从本地存储读取API密钥
onMounted(() => {
    const savedApiFormat = LocalStorage.getApiFormat()
    if (savedApiFormat === 'gemini') {
        apiFormat.value = 'gemini'
    }
    const savedApiKey = apiFormat.value === 'gemini' ? LocalStorage.getGeminiApiKey() : LocalStorage.getApiKey()
    const savedEndpoint = LocalStorage.getApiEndpoint()
    const savedModelId = LocalStorage.getModelId()

    if (savedApiKey) {
        apiKey.value = savedApiKey
        showApiSettings.value = false
    } else {
        // 如果没有API密钥，自动展开设置面板
        showApiSettings.value = true
    }

    // 先设置端点，再恢复模型缓存，最后设置模型ID
    const endpointToUse =
        (apiFormat.value === 'gemini' ? savedEndpoint.trim() || DEFAULT_GEMINI_ENDPOINT : savedEndpoint.trim() || DEFAULT_API_ENDPOINT)
    const modelIdToUse = savedModelId.trim() || (apiFormat.value === 'gemini' ? DEFAULT_GEMINI_MODEL : DEFAULT_MODEL_ID)

    // 恢复模型缓存
    restoreModelOptionsFromCache(endpointToUse)

    // 设置值（这些赋值会触发 watch，但此时 hasSyncedInitialEndpoint 还是 false）
    selectedModel.value = modelIdToUse
    apiEndpoint.value = endpointToUse

    ensureSelectedOptionPresent()

    // 最后才标记初始化完成，这样后续的 watch 触发才会被当作用户操作
    hasSyncedInitialEndpoint = true
})

// 监听API密钥变化，自动保存到本地存储
watch(
    apiKey,
    (newApiKey: string, previousApiKey?: string) => {
        const trimmed = newApiKey.trim()
        if (trimmed) {
            if (apiFormat.value === 'gemini') {
                LocalStorage.saveGeminiApiKey(trimmed)
            } else {
                LocalStorage.saveApiKey(trimmed)
            }
        } else {
            if (apiFormat.value === 'gemini') {
                LocalStorage.clearGeminiApiKey()
            } else {
                LocalStorage.clearApiKey()
            }
            if ((previousApiKey || '').trim()) {
                LocalStorage.clearModelCache()
                modelOptions.value = []
                selectedModel.value = DEFAULT_MODEL_ID
                modelsError.value = null
            }
            showApiSettings.value = true
        }
    },
    { immediate: false }
)

watch(
    apiEndpoint,
    (newEndpoint: string, previousEndpoint?: string) => {
        const trimmed = newEndpoint.trim()
        const previousTrimmed = (previousEndpoint || '').trim()

        if (trimmed) {
            LocalStorage.saveApiEndpoint(trimmed)
        } else {
            LocalStorage.clearApiEndpoint()
        }

        // 如果是初始化阶段（在 onMounted 中），直接返回，不做任何处理
        if (!hasSyncedInitialEndpoint) {
            return
        }

        // 只有在初始化完成后，用户主动修改端点时才重置模型
        if (trimmed !== previousTrimmed) {
            modelOptions.value = []
            modelsError.value = null
            if (previousTrimmed) {
                selectedModel.value = DEFAULT_MODEL_ID
                LocalStorage.clearModelCache(previousTrimmed)
            }
            showApiSettings.value = true
        }
    },
    { immediate: false }
)

watch(
    apiFormat,
    newFormat => {
        LocalStorage.saveApiFormat(newFormat)
        if (newFormat === 'gemini') {
            apiEndpoint.value = DEFAULT_GEMINI_ENDPOINT
            selectedModel.value = DEFAULT_GEMINI_MODEL
            const gemKey = LocalStorage.getGeminiApiKey()
            apiKey.value = gemKey
        } else {
            apiEndpoint.value = DEFAULT_API_ENDPOINT
            selectedModel.value = DEFAULT_MODEL_ID
            const openaiKey = LocalStorage.getApiKey()
            apiKey.value = openaiKey
        }
    },
    { immediate: false }
)

watch(
    selectedModel,
    (newModel: string) => {
        const trimmed = newModel.trim()
        if (trimmed) {
            LocalStorage.saveModelId(trimmed)
        } else {
            LocalStorage.clearModelId()
            LocalStorage.clearModelCache(apiEndpoint.value)
            // 避免在初始化时重置
            if (hasSyncedInitialEndpoint) {
                selectedModel.value = DEFAULT_MODEL_ID
                showApiSettings.value = true
            }
        }
        // 只在初始化完成后才调用 ensureSelectedOptionPresent
        if (hasSyncedInitialEndpoint) {
            ensureSelectedOptionPresent()
        }
    },
    { immediate: false }
)

const handleFetchModels = async () => {
    if (apiFormat.value === 'gemini') {
        modelsError.value = 'Gemini 原生模式无需拉取模型列表'
        return
    }
    if (!apiKey.value.trim() || !apiEndpoint.value.trim()) return

    isFetchingModels.value = true
    modelsError.value = null

    try {
        const rawModels = await fetchModels(apiKey.value, apiEndpoint.value)
        const options = mapModelsToOptions(rawModels)

        if (!options.length) {
            throw new Error('未找到可用模型')
        }

        modelOptions.value = options
        LocalStorage.saveModelCache(apiEndpoint.value, options)

        const preferred =
            options.find(option => option.id === selectedModel.value) ||
            options.find(option => option.id === DEFAULT_MODEL_ID) ||
            options.find(option => option.supportsImages) ||
            options[0]

        selectedModel.value = preferred.id
        ensureSelectedOptionPresent()
    } catch (fetchError) {
        modelsError.value = fetchError instanceof Error ? fetchError.message : '无法获取模型列表'
        modelOptions.value = []
        selectedModel.value = DEFAULT_MODEL_ID
    } finally {
        isFetchingModels.value = false
    }
}

const mapModelsToOptions = (models: ApiModel[]): ModelOption[] => {
    const uniqueIds = new Set<string>()
    const options: ModelOption[] = []

    models.forEach(model => {
        if (!model?.id || uniqueIds.has(model.id)) return
        uniqueIds.add(model.id)

        const supportsImages = detectImageSupport(model)
        const label = buildModelLabel(model)
        const description = (typeof model.description === 'string' && model.description.trim()) ||
            (typeof (model as Record<string, unknown>).about === 'string' && String((model as Record<string, unknown>).about).trim()) ||
            ''

        options.push({
            id: model.id,
            label,
            description,
            supportsImages
        })
    })

    return options.sort((a, b) => {
        if (a.supportsImages !== b.supportsImages) {
            return a.supportsImages ? -1 : 1
        }
        return a.label.localeCompare(b.label)
    })
}

const detectImageSupport = (model: ApiModel): boolean => {
    const caps = model.capabilities
    if (caps && typeof caps === 'object') {
        if ((caps as Record<string, unknown>).image === true) return true
        if ((caps as Record<string, unknown>).images === true) return true
        if ((caps as Record<string, unknown>).vision === true) return true
        if ((caps as Record<string, unknown>).multimodal === true) return true
    }

    const tags = (model as Record<string, unknown>).tags
    if (Array.isArray(tags) && tags.some(tag => typeof tag === 'string' && /image|vision|photo|picture|art|draw/i.test(tag))) {
        return true
    }

    return /image|vision|flux|art|picture|photo|illustration/i.test(model.id)
}

const buildModelLabel = (model: ApiModel): string => {
    if (model.name && typeof model.name === 'string' && model.name.trim()) {
        return model.name.trim()
    }
    const segments = model.id.split('/')
    const lastSegment = segments[segments.length - 1]
    return lastSegment || model.id
}

const handleModelPicked = () => {
    if (!selectedModel.value.trim()) return
    modelsError.value = null
    if (!showApiSettings.value) return

    setTimeout(() => {
        if (selectedModel.value.trim()) {
            showApiSettings.value = false
        }
    }, 600)
}

const restoreModelOptionsFromCache = (endpoint: string) => {
    const trimmedEndpoint = endpoint.trim()
    if (!trimmedEndpoint) return

    const cached = LocalStorage.getModelCache(trimmedEndpoint)
    if (!cached.length) return

    modelOptions.value = cached
    ensureSelectedOptionPresent()
}

const ensureSelectedOptionPresent = () => {
    const currentId = selectedModel.value.trim()
    if (!currentId) return

    const exists = modelOptions.value.some(option => option.id === currentId)
    if (!exists) {
        modelOptions.value = [
            ...modelOptions.value,
            {
                id: currentId,
                label: buildFallbackLabel(currentId),
                description: '',
                supportsImages: true
            }
        ]
    }

    modelOptions.value = modelOptions.value.sort((a, b) => {
        if (a.supportsImages !== b.supportsImages) {
            return a.supportsImages ? -1 : 1
        }
        return a.label.localeCompare(b.label)
    })
}

const buildFallbackLabel = (modelId: string): string => {
    const segments = modelId.split('/')
    const lastSegment = segments[segments.length - 1]
    return lastSegment || modelId
}

const pushImageToUpload = (image: string | null) => {
    if (!image) return
    const filtered = selectedImages.value.filter(existing => existing !== image)
    selectedImages.value = [image, ...filtered]
}

const shouldUseBatchMode = computed(() => batchEnabled.value && selectedImages.value.length > 0)

const resolvedConcurrency = computed(() => Math.max(1, Math.min(8, Math.floor(concurrencyLimit.value) || 1)))

const resolvedCooldown = computed(() => Math.max(0, Math.floor(queueCooldownMs.value) || 0))

const isBatchProcessing = computed(() => batchResults.value.some(item => item.processing))

const displayLoading = computed(() => {
    if (isBatchProcessing.value) return false
    return isLoading.value
})

const displayResultList = computed(() => batchResults.value)

const displayResult = computed(() => {
    if (batchResults.value.length > 0) return null
    return result.value
})

const displayError = computed(() => error.value)

const canPushDisplayResult = computed(() => {
    const hasBatchOutput = batchResults.value.some(item => item.output)
    if (hasBatchOutput) return true
    return Boolean(displayResult.value)
})

const modeLabel = computed(() => (selectedImages.value.length > 0 ? '图文生图' : '文生图'))

const canGenerate = computed(
    () =>
        apiKey.value.trim() &&
        apiEndpoint.value.trim() &&
        selectedModel.value.trim() &&
        promptInput.value.trim() &&
        !isLoading.value
)

const buildRequest = (prompt: string, images: string[]): GenerateRequest => {
    const endpoint =
        apiFormat.value === 'gemini'
            ? apiEndpoint.value.trim() || DEFAULT_GEMINI_ENDPOINT
            : apiEndpoint.value.trim() || DEFAULT_API_ENDPOINT
    const model =
        apiFormat.value === 'gemini'
            ? selectedModel.value.trim() || DEFAULT_GEMINI_MODEL
            : selectedModel.value.trim() || DEFAULT_MODEL_ID

    const request: GenerateRequest = {
        prompt,
        images,
        apikey: apiKey.value,
        endpoint,
        model,
        apiFormat: apiFormat.value
    }

    if (showAspectRatioSelector.value) {
        request.aspectRatio = selectedAspectRatio.value
    }

    if (showGemini3ProConfig.value) {
        request.imageSize = gemini3ImageSize.value
        request.enableGoogleSearch = gemini3EnableGoogleSearch.value
    }

    return request
}

// 判断是否显示宽高比选择器（Gemini 2.5 Flash Image 系列和 Gemini 3 Pro Image 模型时显示）
const showAspectRatioSelector = computed(() => {
    const modelId = selectedModel.value.toLowerCase().trim()
    if (!modelId) return false

    const segments = modelId.split('/')
    const normalizedId = segments[segments.length - 1]
    return normalizedId === 'gemini-2.5-flash-image' ||
           normalizedId === 'gemini-2.5-flash-image-preview' ||
           modelId.includes('gemini-3-pro-image')
})

// 判断是否显示 Gemini 3 Pro Image 配置
const showGemini3ProConfig = computed(() => {
    const modelId = selectedModel.value.toLowerCase().trim()
    if (!modelId) return false
    return modelId.includes('gemini-3-pro-image')
})

const handlePushDisplayResult = () => {
    pushImageToUpload(displayResult.value)
}

const handlePushItem = (index: number) => {
    const target = displayResultList.value[index]
    if (!target?.output) return
    pushImageToUpload(target.output)
}

const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms))

const resolveExtension = (image: string, contentType?: string | null): string => {
    const dataMatch = image.match(/^data:image\/([a-zA-Z0-9+]+);/)
    if (dataMatch?.[1]) {
        return dataMatch[1]
    }
    if (contentType?.includes('jpeg') || contentType?.includes('jpg')) return 'jpg'
    if (contentType?.includes('png')) return 'png'
    if (contentType?.includes('webp')) return 'webp'
    if (contentType?.includes('gif')) return 'gif'
    return 'png'
}

const downloadImageAsset = async (image: string | null, suffix: string) => {
    if (!image) return
    if (typeof window === 'undefined') return

    let downloadUrl = image
    let revokeUrl: string | null = null
    let extension = resolveExtension(image)

    try {
        if (!image.startsWith('data:')) {
            const response = await fetch(image)
            const blob = await response.blob()
            downloadUrl = URL.createObjectURL(blob)
            revokeUrl = downloadUrl
            extension = resolveExtension(image, response.headers.get('Content-Type'))
        }

        const link = document.createElement('a')
        link.href = downloadUrl
        link.download = `nano-banana-${suffix}.${extension}`
        link.rel = 'noopener'
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)

        if (revokeUrl) {
            URL.revokeObjectURL(revokeUrl)
        }
    } catch (downloadError) {
        window.open(image, '_blank', 'noopener')
    }
}

const handleDownloadResult = async () => {
    await downloadImageAsset(displayResult.value, `${Date.now()}`)
}

const handleDownloadItem = async (index: number) => {
    const item = displayResultList.value[index]
    if (!item?.output) return
    const batchId = currentBatchId.value || Date.now().toString()
    await downloadImageAsset(item.output, `${batchId}-${item.seq}`)
}

const handleDownloadAll = async () => {
    const batchId = currentBatchId.value || Date.now().toString()
    const outputs = displayResultList.value.filter(item => item.output)

    for (let i = 0; i < outputs.length; i++) {
        const item = outputs[i]
        await downloadImageAsset(item.output as string, `${batchId}-${item.seq}`)
    }
}

const runConcurrentTasks = async (tasks: Array<() => Promise<void>>, limit: number) => {
    const launchSpacingMs = 500
    const workerCount = Math.min(limit, tasks.length)
    let cursor = 0

    const workers = Array.from({ length: workerCount }, async () => {
        while (cursor < tasks.length) {
            const current = cursor
            cursor += 1
            await delay(current * launchSpacingMs)
            await tasks[current]()
        }
    })

    await Promise.all(workers)
}

const runQueuedTasks = async (tasks: Array<() => Promise<void>>, cooldown: number) => {
    for (let i = 0; i < tasks.length; i++) {
        await tasks[i]()
        if (i < tasks.length - 1 && cooldown > 0) {
            await delay(cooldown)
        }
    }
}

const handleUpdateItemPrompt = (index: number, value: string) => {
    const target = batchResults.value[index]
    if (!target) return
    batchResults.value[index] = { ...target, prompt: value }
}

const handleRetryItem = async (index: number) => {
    const target = batchResults.value[index]
    if (!target) return

    batchResults.value[index] = { ...target, processing: true, error: null }

    try {
        const response = await generateImage(buildRequest(target.prompt.trim() || promptInput.value.trim(), [target.input]))
        batchResults.value[index] = { ...target, output: response.imageUrl, error: null, processing: false }
    } catch (err) {
        batchResults.value[index] = {
            ...target,
            output: null,
            error: err instanceof Error ? err.message : '生成失败',
            processing: false
        }
    }
}

const handleGenerate = async () => {
    if (!canGenerate.value) return

    if (shouldUseBatchMode.value && batchMode.value === 'concurrent' && lastBatchCompletedAt.value) {
        const elapsed = Date.now() - lastBatchCompletedAt.value
        const waitMs = Math.max(0, Math.floor(batchCooldownMs.value) - elapsed)
        if (waitMs > 0) {
            await delay(waitMs)
        }
    }

    isLoading.value = true
    error.value = null
    // 立即清除之前的结果，确保用户看到新的生成过程
    result.value = null
    batchResults.value = []
    currentBatchId.value = ''

    try {
        const prompt = promptInput.value.trim()

        if (shouldUseBatchMode.value) {
            currentBatchId.value = Date.now().toString()
            const imagesToProcess = [...selectedImages.value]
            batchResults.value = imagesToProcess.map((image, idx) => ({
                input: image,
                output: null,
                error: null,
                seq: idx + 1,
                prompt,
                processing: true
            }))

            const tasks = imagesToProcess.map((image, index) => async () => {
                try {
                    const response = await generateImage(buildRequest(batchResults.value[index].prompt, [image]))
                    batchResults.value[index] = {
                        ...batchResults.value[index],
                        output: response.imageUrl,
                        error: null,
                        processing: false
                    }
                } catch (taskError) {
                    batchResults.value[index] = {
                        ...batchResults.value[index],
                        output: null,
                        error: taskError instanceof Error ? taskError.message : '生成失败',
                        processing: false
                    }
                }
            })

            if (batchMode.value === 'queue') {
                await runQueuedTasks(tasks, resolvedCooldown.value)
            } else {
                await runConcurrentTasks(tasks, resolvedConcurrency.value)
            }

            const hasFailure = batchResults.value.some(item => item.error)
            const hasSuccess = batchResults.value.some(item => item.output)

            if (hasFailure && hasSuccess) {
                error.value = '部分图片生成失败，请查看对应卡片提示'
            } else if (hasFailure) {
                error.value = '批量生成失败，请检查配置后重试'
            } else {
                error.value = null
            }
        } else {
            const response = await generateImage(buildRequest(prompt, selectedImages.value))
            result.value = response.imageUrl
        }
    } catch (err) {
        error.value = err instanceof Error ? err.message : '生成失败'
        // 生成失败时也要清除结果
        result.value = null
    } finally {
        isLoading.value = false
        if (shouldUseBatchMode.value && batchMode.value === 'concurrent') {
            lastBatchCompletedAt.value = Date.now()
        }
    }
}

</script>
