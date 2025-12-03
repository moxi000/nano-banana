<template>
    <div class="bg-white border-4 border-black border-t-0 rounded-b-lg p-4 shadow-lg min-h-[400px] flex flex-col">
        <div class="flex-1 bg-gray-50 border-2 border-black rounded-lg p-6 flex items-center justify-center">
            <!-- Loading State -->
            <div v-if="loading" class="text-center">
                <div class="w-12 h-12 border-4 border-yellow-300 border-t-orange-500 rounded-full animate-spin mx-auto mb-4" />
                <p class="font-bold text-base flex items-center justify-center gap-2">🍌 正在创造魔法...</p>
                <p class="text-gray-600">请稍等片刻</p>
            </div>

            <!-- Batch Result List -->
            <div v-else-if="resultList.length > 0" class="w-full h-full flex flex-col gap-3">
                <div class="flex flex-wrap items-center justify-between gap-3">
                    <div class="flex items-center gap-3">
                        <p class="font-bold text-base flex items-center gap-2">🍌 批量生成结果</p>
                        <span class="px-2 py-1 rounded-full text-xs border-2 border-black bg-yellow-200 font-bold">共 {{ resultList.length }} 张</span>
                        <span
                            v-if="batchProcessing"
                            class="px-2 py-1 rounded-full text-xs border-2 border-black bg-green-200 font-bold animate-pulse"
                        >
                            处理中...
                        </span>
                    </div>
                    <div class="flex items-center gap-2">
                        <button
                            @click="$emit('download-all')"
                            class="px-4 py-2 bg-yellow-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-yellow-400 transition-all flex items-center justify-center gap-2"
                        >
                            ⬇️ 批量下载
                        </button>
                    </div>
                </div>

                <div v-if="error" class="px-3 py-2 border-2 border-dashed border-red-500 bg-red-50 text-sm text-red-700 rounded-lg">
                    {{ error }}
                </div>

                <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div
                        v-for="(item, index) in resultList"
                        :key="index"
                        class="relative bg-white border-2 border-black rounded-lg p-3 shadow-sm flex flex-col gap-3"
                    >
                        <div class="flex items-center justify-between text-sm font-bold">
                            <span class="flex items-center gap-2">
                                <span class="px-2 py-1 rounded-full bg-yellow-100 border border-black">#{{ item.seq }}</span>
                                <span>按上传顺序</span>
                            </span>
                            <span v-if="item.processing" class="text-xs text-gray-600 flex items-center gap-1">
                                <span class="w-3 h-3 border-2 border-yellow-300 border-t-orange-500 rounded-full animate-spin" />
                                处理中
                            </span>
                            <span v-else-if="!item.output && !item.error" class="text-xs text-gray-600">排队/处理中</span>
                        </div>

                        <template v-if="item.output">
                            <div class="aspect-square bg-gray-100 border-2 border-black rounded-lg overflow-hidden flex items-center justify-center">
                                <img :src="item.output" alt="批量生成图片" class="max-h-full max-w-full object-contain" />
                            </div>
                            <div class="flex flex-col gap-2">
                                <label class="text-xs font-bold text-gray-800 flex items-center gap-2">
                                    📝 微调提示词
                                    <span class="text-[11px] text-gray-600">(仅作用于本张重试)</span>
                                </label>
                                <textarea
                                    :value="item.prompt"
                                    @input="$emit('update-item-prompt', index, ($event.target as HTMLTextAreaElement).value)"
                                    class="w-full px-3 py-2 border-2 border-black rounded-lg focus:ring-2 focus:ring-purple-500 focus:outline-none text-sm"
                                    rows="3"
                                />
                            </div>

                            <div class="flex flex-wrap gap-2">
                                <button
                                    @click="$emit('retry-item', index)"
                                    class="px-3 py-2 bg-blue-200 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-blue-300 transition-all flex items-center justify-center gap-2 disabled:opacity-60"
                                    :disabled="item.processing"
                                >
                                    🔁 单张重试
                                </button>
                                <button
                                    v-if="canPush"
                                    @click="$emit('push-item', index)"
                                    class="px-3 py-2 bg-green-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-green-400 transition-all flex items-center justify-center gap-2"
                                >
                                    🎨 二次创作
                                </button>
                                <button
                                    @click="$emit('download-item', index)"
                                    class="px-3 py-2 bg-yellow-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-yellow-400 transition-all flex items-center justify-center gap-2"
                                >
                                    ⬇️ 下载
                                </button>
                            </div>
                        </template>

                        <template v-else-if="item.error">
                            <div class="flex flex-col items-start gap-2 p-3 rounded-lg bg-red-50 border-2 border-red-400 text-sm text-red-700">
                                <span class="font-bold flex items-center gap-2">❌ 生成失败</span>
                                <span>{{ item.error }}</span>
                                <div class="flex flex-col gap-2 w-full">
                                    <label class="text-xs font-bold text-gray-800 flex items-center gap-2">
                                        📝 微调提示词
                                        <span class="text-[11px] text-gray-600">(仅作用于本张重试)</span>
                                    </label>
                                    <textarea
                                        :value="item.prompt"
                                        @input="$emit('update-item-prompt', index, ($event.target as HTMLTextAreaElement).value)"
                                        class="w-full px-3 py-2 border-2 border-black rounded-lg focus:ring-2 focus:ring-purple-500 focus:outline-none text-sm"
                                        rows="3"
                                    />
                                    <button
                                        @click="$emit('retry-item', index)"
                                        class="px-3 py-2 bg-blue-200 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-blue-300 transition-all flex items-center justify-center gap-2 disabled:opacity-60"
                                        :disabled="item.processing"
                                    >
                                        🔁 单张重试
                                    </button>
                                </div>
                            </div>
                        </template>

                        <template v-else>
                            <div class="flex-1 flex flex-col items-center justify-center gap-2 text-center">
                                <div class="w-10 h-10 border-4 border-yellow-300 border-t-orange-500 rounded-full animate-spin" />
                                <p class="text-sm font-bold text-gray-700">等待生成</p>
                            </div>
                        </template>
                    </div>
                </div>
            </div>

            <!-- Single Result Image -->
            <div v-else-if="result" class="w-full h-full flex items-center justify-center relative">
                <img :src="result" alt="生成的艺术作品" class="max-w-full max-h-[600px] rounded-lg border-2 border-black shadow-lg object-contain" />
                <div class="absolute bottom-4 right-4 flex flex-col gap-2 items-stretch">
                    <button
                        v-if="canPush"
                        @click="$emit('push')"
                        class="px-4 py-2 bg-green-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-green-400 transition-all flex items-center justify-center gap-2"
                    >
                        🎨 二次创作
                    </button>
                    <button
                        @click="$emit('download')"
                        class="px-4 py-2 bg-yellow-300 text-black font-bold border-2 border-black rounded-lg shadow-lg hover:bg-yellow-400 transition-all flex items-center justify-center gap-2"
                    >
                        ⬇️ 下载图片
                    </button>
                </div>
            </div>

            <!-- Error State -->
            <div v-else-if="error" class="text-center">
                <div class="text-red-500 text-6xl mb-4">🍌💥</div>
                <p class="text-red-600 font-bold text-base mb-2">哎呀！出了点问题</p>
                <p class="text-gray-600 text-sm">{{ error }}</p>
            </div>

            <!-- Empty State -->
            <div v-else class="text-center">
                <div class="w-12 h-12 border-4 border-gray-300 rounded-lg mx-auto mb-4 flex items-center justify-center">
                    <span class="text-2xl">🍌</span>
                </div>
                <h3 class="font-bold text-base mb-2 flex items-center justify-center gap-2">🍌 等待魔法开始...</h3>
                <p class="text-gray-600">上传图片并选择风格开始创作</p>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import type { BatchResultItem } from '../types'

defineProps<{
    result: string | null
    resultList: BatchResultItem[]
    loading: boolean
    batchProcessing?: boolean
    error: string | null
    canPush: boolean
}>()

defineEmits<{
    download: []
    downloadItem: [index: number]
    downloadAll: []
    push: []
    pushItem: [index: number]
    retryItem: [index: number]
    updateItemPrompt: [index: number, value: string]
}>()
</script>
