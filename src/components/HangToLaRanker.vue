<template>
  <div class="ranker-container">
    <!-- 引导步骤区域（放在排行榜上方，仅步骤1和2显示） -->
    <div class="guide-area" v-if="currentStep === 1 || currentStep === 2">
      <!-- 步骤条 -->
      <n-steps :current="currentStep" style="margin-bottom: 20px;">
        <n-step title="第一步" description="上传图片" />
        <n-step title="第二步" description="选择排序方式" />
        <n-step title="第三步" description="拖拽排行" />
      </n-steps>

      <!-- 第一步：上传图片 -->
      <div v-if="currentStep === 1" class="step-content">
        <div class="step-title">上传你要排行的图片</div>
        <div class="upload-area">
          <n-upload
            multiple
            directory-dnd
            :show-file-list="false"
            @change="handleUpload"
          >
            <n-upload-dragger style="padding: 40px;">
              <div style="margin-bottom: 12px">
                <n-icon size="64" :depth="3">
                  <span class="upload-icon">+</span>
                </n-icon>
              </div>
              <n-text style="font-size: 18px">
                点击或拖拽图片到此处上传
              </n-text>
            </n-upload-dragger>
          </n-upload>
          <div class="demo-hint">
            <n-button text type="primary" @click="loadColaDemo">
              🥤 试试可乐排行示例
            </n-button>
          </div>
        </div>
        
        <!-- 已上传的图片预览 -->
        <div v-if="pool.length > 0" class="uploaded-preview">
          <div class="preview-header">
            <span>已上传 {{ pool.length }} 张图片</span>
          </div>
          <div class="preview-list">
            <div
              v-for="item in pool"
              :key="item.id"
              class="rank-item"
            >
              <img :src="item.url" class="rank-image" draggable="false" />
              <div class="delete-btn" @click="deleteItem(item.id, pool)">×</div>
            </div>
          </div>
        </div>

        <div class="step-actions">
          <n-button 
            type="primary" 
            size="large"
            :disabled="pool.length === 0"
            @click="goToStep2"
          >
            下一步：选择排序方式
          </n-button>
        </div>
      </div>

      <!-- 第二步：选择排序方式 -->
      <div v-else-if="currentStep === 2" class="step-content">
        <div class="step-title">选择图片出现的顺序</div>
        <div class="order-selection">
          <div 
            class="order-option" 
            :class="{ active: rankMode === 'sequence' }"
            @click="rankMode = 'sequence'"
          >
            <div class="option-icon">📋</div>
            <div class="option-title">按上传顺序</div>
            <div class="option-desc">图片将按照你上传的顺序依次出现</div>
          </div>
          <div 
            class="order-option" 
            :class="{ active: rankMode === 'random' }"
            @click="rankMode = 'random'"
          >
            <div class="option-icon">🎲</div>
            <div class="option-title">随机顺序</div>
            <div class="option-desc">图片将被打乱，以随机顺序出现（盲盒模式）</div>
          </div>
        </div>
        <div class="step-actions">
          <n-button size="large" @click="goToStep1">上一步</n-button>
          <n-button 
            type="primary" 
            size="large"
            @click="startRanking"
          >
            开始排行！
          </n-button>
        </div>
      </div>
    </div>

    <!-- 排行中的状态栏 -->
    <div v-if="currentStep === 3 && rankingQueue.length > 0" class="ranking-status-bar">
      <span class="status-text">排行进度: {{ totalCount - rankingQueue.length }} / {{ totalCount }}</span>
      <n-button size="small" @click="stopRanking">停止排行</n-button>
    </div>

    <!-- 排行榜区域 -->
    <div class="tier-list-wrapper" ref="tierListWrapper">
      <div class="tier-list" id="tier-list-capture">
        <!-- 标题 -->
        <div class="header">
          <div v-if="isEditingTitle" class="title-edit-container">
            <input
              ref="titleInputRef"
              v-model="title"
              class="title-input"
              @blur="finishEditingTitle"
              @keydown.enter="finishEditingTitle"
            />
          </div>
          <n-h1
            v-else
            @dblclick="startEditingTitle"
            class="main-title"
            title="双击编辑标题"
          >
            {{ title }}
          </n-h1>
        </div>
        <div
          v-for="tier in tiers"
          :key="tier.id"
          class="tier-row"
        >
          <!-- 左侧标签 -->
          <div
            class="tier-label"
            :style="{ backgroundColor: tier.color }"
            @dblclick="startEditing(tier)"
          >
            <input
              v-if="editingTierId === tier.id"
              ref="editInput"
              v-model="tier.name"
              class="tier-name-input"
              @blur="finishEditing"
              @keydown.enter="finishEditing"
            />
            <span v-else class="tier-name">{{ tier.name }}</span>
          </div>

          <!-- 右侧放置区 -->
          <div
            class="tier-content"
            @dragover.prevent
            @dragenter="handleDragEnter(tier.id)"
            @dragleave="handleDragLeave"
            @drop="handleDrop(tier.items, tier.id)"
            :class="{ 'drag-over': isDragging, 'drag-hover': dragOverTierId === tier.id }"
          >
            <div
              v-for="item in tier.items"
              :key="item.id"
              class="rank-item"
              draggable="true"
              @dragstart="handleDragStart(item, tier.items)"
            >
              <img :src="item.url" class="rank-image" draggable="false" />
              <div class="delete-btn" @click="deleteItem(item.id, tier.items)">×</div>
            </div>
          </div>
        </div>
      </div>

      <!-- 悬浮的待排图片（第三步排行中） -->
      <div v-if="currentStep === 3 && rankingQueue.length > 0" class="floating-card">
        <transition name="fade" mode="out-in">
          <div
            :key="rankingQueue[0].id"
            class="rank-item large-rank-item"
            draggable="true"
            @dragstart="handleDragStart(rankingQueue[0], rankingQueue)"
          >
            <img :src="rankingQueue[0].url" class="rank-image" draggable="false" />
          </div>
        </transition>
      </div>
    </div>

    <!-- 排行完成后的操作区域 -->
    <div v-if="currentStep === 3 && rankingQueue.length === 0" class="complete-actions">
      <div class="complete-message">🎉 排行完成！共 {{ totalCount }} 张</div>
      <n-space>
        <n-button type="primary" @click="copyImage" :loading="isExporting">
          📋 复制图片
        </n-button>
        <n-button type="info" @click="exportImage" :loading="isExporting">
          💾 导出图片
        </n-button>
        <n-button @click="resetAll">重新开始</n-button>
      </n-space>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue'
import html2canvas from 'html2canvas'
import {
  NH1,
  NH2,
  NP,
  NUpload,
  NUploadDragger,
  NIcon,
  NText,
  useMessage,
  UploadFileInfo,
  NButton,
  NRadio,
  NRadioGroup,
  NSpace,
  NSteps,
  NStep
} from 'naive-ui'

// --- 类型定义 ---

/**
 * 代表一个图片项
 */
interface RankItem {
  id: string
  url: string
}

/**
 * 代表一个层级
 */
interface Tier {
  id: string
  name: string
  color: string
  items: RankItem[]
}

// --- 状态定义 ---

const message = useMessage()

const title = ref('从夯到拉排行')
const isEditingTitle = ref(false)
const titleInputRef = ref<HTMLInputElement | null>(null)

/**
 * 默认的层级定义
 */
const tiers = ref<Tier[]>([
  { id: 't1', name: '夯', color: '#ff4d4f', items: [] },     // 红色 - 夯
  { id: 't2', name: '顶级', color: '#ff7a45', items: [] },   // 橙色 - 顶级
  { id: 't3', name: '人上人', color: '#fadb14', items: [] }, // 黄色 - 人上人
  { id: 't4', name: 'NPC', color: '#fffbe6', items: [] },    // 米色 - NPC (文字颜色可能需要深色)
  { id: 't5', name: '拉完了', color: '#f0f0f0', items: [] }  // 灰色 - 拉爆了
])

/**
 * 未排序的图片池
 */
const pool = ref<RankItem[]>([])

/**
 * 当前正在拖拽的图片项
 */
const draggedItem = ref<RankItem | null>(null)

/**
 * 当前拖拽项的来源列表（用于移动后删除原位置的项）
 */
const sourceList = ref<RankItem[] | null>(null)

/**
 * 是否正在拖拽中（用于UI反馈）
 */
const isDragging = ref(false)
const dragOverTierId = ref<string | null>(null)
const isRanking = ref(false)
const rankMode = ref<'sequence' | 'random'>('sequence')
const isExporting = ref(false)

/**
 * 排行队列（开始排行后使用）
 */
const rankingQueue = ref<RankItem[]>([])

/**
 * 总图片数量（用于显示进度）
 */
const totalCount = ref(0)

/**
 * 当前步骤 1=上传 2=选择顺序 3=排行中
 */
const currentStep = ref(1)

/**
 * 当前正在编辑的层级ID
 */
const editingTierId = ref<string | null>(null)

/**
 * 编辑输入框的引用
 */
const editInput = ref<HTMLInputElement[] | null>(null)

/**
 * 排行榜区域的引用（用于滚动定位）
 */
const tierListWrapper = ref<HTMLElement | null>(null)

// --- 方法 ---

/**
 * 打乱数组 (Fisher-Yates Shuffle)
 */
// @ts-ignore
const shuffle = <T>(array: T[]): T[] => {
  const result = [...array]
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    // @ts-ignore
    ;[result[i], result[j]] = [result[j], result[i]]
  }
  return result
}

/**
 * 切换排行模式
 */
const toggleRanking = () => {
  if (isRanking.value) {
    // 停止排行
    isRanking.value = false
    return
  }
  
  // 开始排行
  if (pool.value.length === 0) {
    message.warning('请先上传图片')
    return
  }

  if (rankMode.value === 'random') {
    pool.value = shuffle(pool.value)
  }
  
  isRanking.value = true
}

/**
 * 进入第一步
 */
const goToStep1 = () => {
  currentStep.value = 1
}

/**
 * 进入第二步
 */
const goToStep2 = () => {
  if (pool.value.length === 0) {
    message.warning('请先上传图片')
    return
  }
  currentStep.value = 2
}

/**
 * 可乐品牌示例数据
 */
const colaDemoData = [
  { name: '可口可乐', url: '/demo/cola/coca-cola.svg' },
  { name: '百事可乐', url: '/demo/cola/pepsi.svg' },
  { name: '崂山可乐', url: '/demo/cola/laoshan.svg' },
  { name: '天府可乐', url: '/demo/cola/tianfu.svg' },
  { name: '非常可乐', url: '/demo/cola/feichang.svg' },
]

/**
 * 加载可乐示例
 */
const loadColaDemo = () => {
  // 清空现有图片
  pool.value = []
  
  // 添加可乐品牌图片
  colaDemoData.forEach((cola, index) => {
    pool.value.push({
      id: `cola-${Date.now()}-${index}`,
      url: cola.url
    })
  })
  
  // 修改标题
  title.value = '可乐排行榜'
  
  message.success(`已加载 ${colaDemoData.length} 款可乐，开始你的排行吧！`)
}

/**
 * 开始排行（进入第三步）
 */
const startRanking = () => {
  // 复制图片到排行队列
  let queue = [...pool.value]
  
  // 如果是随机模式，打乱顺序
  if (rankMode.value === 'random') {
    queue = shuffle(queue)
  }
  
  rankingQueue.value = queue
  totalCount.value = queue.length
  
  // 清空原始池
  pool.value = []
  
  currentStep.value = 3
  isRanking.value = true

  // 滚动到排行榜区域
  nextTick(() => {
    tierListWrapper.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
  })
}

/**
 * 停止排行
 */
const stopRanking = () => {
  // 将剩余的图片放回pool
  pool.value = [...rankingQueue.value]
  rankingQueue.value = []
  isRanking.value = false
  currentStep.value = 1
}

/**
 * 重新开始
 */
const resetAll = () => {
  pool.value = []
  rankingQueue.value = []
  totalCount.value = 0
  isRanking.value = false
  currentStep.value = 1
  // 清空所有层级的图片
  tiers.value.forEach(tier => {
    tier.items = []
  })
}

/**
 * 导出排行榜为图片
 */
const exportImage = async () => {
  isExporting.value = true
  try {
    const element = document.getElementById('tier-list-capture')
    if (!element) {
      message.error('找不到排行榜元素')
      return
    }

    const canvas = await html2canvas(element, {
      backgroundColor: '#ffffff',
      scale: 2, // 提高清晰度
      useCORS: true,
      allowTaint: true
    })

    // 创建下载链接
    const link = document.createElement('a')
    link.download = `${title.value}-排行榜.png`
    link.href = canvas.toDataURL('image/png')
    link.click()
    
    message.success('图片已保存！')
  } catch (error) {
    console.error('导出失败:', error)
    message.error('导出失败，请重试')
  } finally {
    isExporting.value = false
  }
}

/**
 * 复制排行榜图片到剪贴板
 */
const copyImage = async () => {
  isExporting.value = true
  try {
    const element = document.getElementById('tier-list-capture')
    if (!element) {
      message.error('找不到排行榜元素')
      return
    }

    const canvas = await html2canvas(element, {
      backgroundColor: '#ffffff',
      scale: 2,
      useCORS: true,
      allowTaint: true
    })

    // 将 canvas 转换为 blob
    canvas.toBlob(async (blob) => {
      if (!blob) {
        message.error('生成图片失败')
        isExporting.value = false
        return
      }

      try {
        // 使用 Clipboard API 复制图片
        await navigator.clipboard.write([
          new ClipboardItem({
            'image/png': blob
          })
        ])
        message.success('图片已复制到剪贴板！')
      } catch (clipboardError) {
        console.error('复制到剪贴板失败:', clipboardError)
        message.error('复制失败，请尝试导出图片')
      } finally {
        isExporting.value = false
      }
    }, 'image/png')
  } catch (error) {
    console.error('复制失败:', error)
    message.error('复制失败，请重试')
    isExporting.value = false
  }
}

/**
 * 开始编辑层级名称
 * @param {Tier} tier - The tier to edit
 */
const startEditing = (tier: Tier) => {
  editingTierId.value = tier.id
  nextTick(() => {
    const input = editInput.value?.[0]
    if (input) {
      input.focus()
      input.select()
    }
  })
}

/**
 * 结束编辑
 */
const finishEditing = () => {
  editingTierId.value = null
}

const startEditingTitle = () => {
  isEditingTitle.value = true
  nextTick(() => {
    titleInputRef.value?.focus()
    titleInputRef.value?.select()
  })
}

const finishEditingTitle = () => {
  isEditingTitle.value = false
  if (!title.value.trim()) {
    title.value = '从夯到拉排行'
  }
}

/**
 * 处理文件上传
 * @param {Object} options - Upload options
 * @param {UploadFileInfo[]} options.fileList - List of files
 */
const handleUpload = (data: { fileList: UploadFileInfo[] }) => {
  if (!data.fileList || data.fileList.length === 0) return
  const lastItem = data.fileList[data.fileList.length - 1]
  if (!lastItem) return
  const file = lastItem.file
  if (!file) return

  if (!file.type.startsWith('image/')) {
    message.error('请上传图片文件')
    return
  }

  const reader = new FileReader()
  reader.onload = (e) => {
    if (e.target?.result) {
      pool.value.push({
        id: generateId(),
        url: e.target.result as string
      })
    }
  }
  reader.readAsDataURL(file)
  
  // 清空上传列表，避免重复显示
  data.fileList.pop()
}

/**
 * 生成唯一ID
 * @returns {string} Unique ID
 */
const generateId = (): string => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2)
}

/**
 * 开始拖拽处理
 * @param {RankItem} item - The item being dragged
 * @param {RankItem[]} list - The list containing the item
 */
const handleDragStart = (item: RankItem, list: RankItem[]) => {
  draggedItem.value = item
  sourceList.value = list
  isDragging.value = true
}

/**
 * 拖拽进入层级
 * @param {string} tierId - The tier ID being entered
 */
const handleDragEnter = (tierId: string) => {
  dragOverTierId.value = tierId
}

/**
 * 拖拽离开层级
 */
const handleDragLeave = () => {
  // 使用延迟来避免子元素触发的闪烁
  setTimeout(() => {
    if (dragOverTierId.value) {
      // 会在 dragenter 时重新设置，这里不立即清除
    }
  }, 50)
}

/**
 * 放置处理
 * @param {RankItem[]} targetList - The list receiving the item
 * @param {string} tierId - The tier ID (optional)
 */
const handleDrop = (targetList: RankItem[], tierId?: string) => {
  isDragging.value = false
  dragOverTierId.value = null
  if (!draggedItem.value || !sourceList.value) return

  // 如果是在同一个列表中，这里可以处理排序逻辑（暂时简单处理为添加到末尾）
  // 也可以实现插入排序，但需要更复杂的 dragover 计算
  
  // 从原列表中移除
  const index = sourceList.value.findIndex(i => i.id === draggedItem.value?.id)
  if (index > -1) {
    sourceList.value.splice(index, 1)
  }

  // 添加到新列表
  targetList.push(draggedItem.value)

  // 重置状态
  draggedItem.value = null
  sourceList.value = null
}

/**
 * 删除图片
 * @param {string} id - Item ID to delete
 * @param {RankItem[]} list - List containing the item
 */
const deleteItem = (id: string, list: RankItem[]) => {
  const index = list.findIndex(i => i.id === id)
  if (index > -1) {
    list.splice(index, 1)
  }
}
</script>

<style scoped>
.ranker-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}

.header {
  text-align: center;
  padding: 15px;
  background: #fff;
}

/* 梯队列表样式 */
.tier-list {
  background: #f0f0f0;
  border: 2px solid #ddd;
  margin-bottom: 20px;
}

.tier-row {
  display: flex;
  min-height: 100px;
  border-bottom: 1px solid #ddd;
}

.tier-row:last-child {
  border-bottom: none;
}

.tier-label {
  width: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 24px;
  color: #000; /* 默认黑色文字 */
  text-shadow: 0 1px 0 rgba(255,255,255,0.3);
  padding: 10px;
  text-align: center;
  flex-shrink: 0;
  cursor: pointer;
  user-select: none;
}

.tier-name-input {
  width: 100%;
  font-size: 20px;
  font-weight: bold;
  text-align: center;
  background: rgba(255, 255, 255, 0.5);
  border: none;
  outline: none;
  padding: 4px;
  border-radius: 4px;
}

.title-input {
  font-size: 36px; /* 匹配 h1 大小 */
  font-weight: 900;
  text-align: center;
  border: none;
  border-bottom: 2px solid #d03050;
  outline: none;
  background: transparent;
  width: 100%;
  max-width: 800px;
  padding: 4px;
  color: #d03050;
  font-family: inherit;
}

.main-title {
  font-size: 36px;
  font-weight: 900;
  color: #d03050; /* 使用夯的红色 */
  text-shadow: 2px 2px 0px rgba(0,0,0,0.1);
  cursor: pointer;
  margin: 0;
  margin: 0;
  line-height: 1.2;
}

/* 特殊处理 NPC 文字颜色，因为背景浅 */
.tier-label[style*="#fffbe6"],
.tier-label[style*="#f0f0f0"] {
  color: #333;
}

.tier-content {
  flex: 1;
  background: #fff;
  padding: 10px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-content: flex-start;
  min-height: 100px; /* 确保有足够区域放置 */
  transition: all 0.3s ease;
}

/* 拖拽时所有层级的轻微提示 */
.tier-content.drag-over {
  background: #f5f5f5;
}

/* 拖拽悬停时的动画效果 */
.tier-content.drag-hover {
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  box-shadow: inset 0 0 20px rgba(24, 160, 88, 0.3);
  transform: scale(1.02);
  border: 2px dashed #18a058;
  margin: -2px;
}

/* 图片池样式 */
.pool-area {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
}

.pool-flex-container {
  display: flex;
  gap: 20px;
  align-items: stretch; /* 确保子元素高度一致 */
}

.upload-section {
  width: 200px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
}

.full-height-upload {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 150px;
}

.full-height-upload {
  /* 去掉强制高度，让它自然填充剩余空间，或者设定一个最小高度 */
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 150px;
}

.upload-wrapper {
  flex: 1;
}

/* 强制覆盖 Naive UI 样式以撑满高度 */
:deep(.n-upload),
:deep(.n-upload-trigger),
:deep(.n-upload-dragger) {
  height: 100% !important;
  box-sizing: border-box;
}

:deep(.n-upload-dragger) {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.upload-icon {
  font-size: 48px;
  line-height: 1;
}

.pool-content {
  flex: 1;
  min-height: 150px;
  background: #f9f9f9;
  border: 2px dashed #ddd;
  border-radius: 4px;
  padding: 15px;
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  align-content: flex-start;
}

.empty-pool {
  width: 100%;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
}

/* 图片项样式 */
.rank-item {
  width: 80px;
  height: 80px;
  position: relative;
  cursor: grab;
  transition: transform 0.2s;
}

.rank-item:active {
  cursor: grabbing;
}

.rank-item:hover {
  transform: scale(1.05);
  z-index: 10;
}

.rank-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 4px;
  border: 2px solid transparent;
}

.rank-item:hover .rank-image {
  border-color: #18a058;
}

.delete-btn {
  position: absolute;
  top: -8px;
  right: -8px;
  width: 20px;
  height: 20px;
  background: #d03050;
  color: white;
  border-radius: 50%;
  text-align: center;
  line-height: 20px;
  font-size: 14px;
  cursor: pointer;
  display: none;
}

.rank-item:hover .delete-btn {
  display: block;
}

.ranking-stage {
  flex: 1;
  min-height: 300px;
  background: #f0f0f0;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 2px dashed #18a058;
}

.current-rank-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.stage-info {
  font-size: 18px;
  font-weight: bold;
  color: #666;
}

.stage-hint {
  color: #999;
  font-size: 14px;
}

.ranking-complete {
  text-align: center;
}

.step-label {
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
  font-size: 14px;
}

.step-content {
  padding: 20px 0;
}

.step-title {
  font-size: 24px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 30px;
  color: #333;
}

.upload-area {
  max-width: 500px;
  margin: 0 auto 30px;
}

.demo-hint {
  text-align: center;
  margin-top: 15px;
}

.uploaded-preview {
  margin-top: 20px;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
}

.preview-header {
  font-weight: bold;
  margin-bottom: 15px;
  color: #666;
}

.preview-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.step-actions {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 30px;
}

.order-selection {
  display: flex;
  gap: 20px;
  justify-content: center;
  margin: 30px 0;
}

.order-option {
  width: 250px;
  padding: 30px 20px;
  border: 2px solid #ddd;
  border-radius: 12px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
  background: #fff;
}

.order-option:hover {
  border-color: #18a058;
  box-shadow: 0 4px 12px rgba(24, 160, 88, 0.15);
}

.order-option.active {
  border-color: #18a058;
  background: linear-gradient(135deg, #f0fff4 0%, #e8f5e9 100%);
}

.option-icon {
  font-size: 48px;
  margin-bottom: 15px;
}

.option-title {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 10px;
  color: #333;
}

.option-desc {
  font-size: 14px;
  color: #666;
}

.ranking-stage-full {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px;
  background: linear-gradient(135deg, #f5f5f5 0%, #e8e8e8 100%);
  border-radius: 12px;
  border: 2px dashed #18a058;
}

.stage-progress {
  font-size: 20px;
  font-weight: bold;
  color: #18a058;
  margin-bottom: 30px;
}

.large-rank-item {
  width: 200px;
  height: 200px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
  transition: all 0.3s ease;
}

.large-rank-item:hover {
  transform: scale(1.02);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* 拖拽悬停时层级标签的动画 */
.tier-row:has(.drag-hover) .tier-label {
  animation: pulse-label 0.8s ease-in-out infinite;
}

@keyframes pulse-label {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

/* 引导区域样式 */
.guide-area {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  margin-bottom: 20px;
}

/* 排行榜外层容器（用于定位悬浮元素） */
.tier-list-wrapper {
  position: relative;
}

/* 悬浮卡片样式 */
.floating-card {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 100;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.25);
  padding: 8px;
  backdrop-filter: blur(4px);
}

.floating-card .large-rank-item {
  width: 150px;
  height: 150px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.3);
  cursor: grab;
}

.floating-card .large-rank-item:active {
  cursor: grabbing;
}

/* 排行中的状态栏 */
.ranking-status-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 20px;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  border-radius: 8px;
  margin-bottom: 15px;
}

.status-text {
  font-weight: bold;
  color: #2e7d32;
}

/* 完成后的操作区域 */
.complete-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  border-radius: 8px;
  margin-top: 15px;
}

.complete-message {
  font-size: 18px;
  font-weight: bold;
  color: #2e7d32;
}

/* ========== 移动端适配 ========== */
@media screen and (max-width: 768px) {
  .ranker-container {
    padding: 10px;
  }

  /* 标题适配 */
  .main-title {
    font-size: 24px;
  }

  .title-input {
    font-size: 24px;
  }

  .header {
    padding: 10px;
  }

  /* 层级标签适配 */
  .tier-label {
    width: 60px;
    font-size: 16px;
    padding: 5px;
  }

  .tier-name-input {
    font-size: 14px;
  }

  .tier-row {
    min-height: 70px;
  }

  .tier-content {
    padding: 5px;
    gap: 5px;
    min-height: 70px;
  }

  /* 图片项适配 */
  .rank-item {
    width: 50px;
    height: 50px;
  }

  .delete-btn {
    width: 16px;
    height: 16px;
    line-height: 16px;
    font-size: 12px;
    top: -5px;
    right: -5px;
  }

  /* 引导区域适配 */
  .guide-area {
    padding: 15px;
  }

  .step-title {
    font-size: 18px;
    margin-bottom: 20px;
  }

  .upload-area {
    margin: 0 auto 20px;
  }

  :deep(.n-upload-dragger) {
    padding: 20px !important;
  }

  .upload-icon {
    font-size: 32px;
  }

  /* 排序选项适配 */
  .order-selection {
    flex-direction: column;
    gap: 15px;
  }

  .order-option {
    width: 100%;
    padding: 20px 15px;
  }

  .option-icon {
    font-size: 36px;
    margin-bottom: 10px;
  }

  .option-title {
    font-size: 16px;
  }

  .option-desc {
    font-size: 12px;
  }

  /* 步骤按钮适配 */
  .step-actions {
    flex-direction: column;
    gap: 10px;
  }

  .step-actions .n-button {
    width: 100%;
  }

  /* 预览列表适配 */
  .preview-list {
    gap: 8px;
  }

  .preview-header {
    font-size: 14px;
  }

  /* 悬浮卡片适配 */
  .floating-card {
    padding: 6px;
  }

  .floating-card .large-rank-item {
    width: 100px;
    height: 100px;
  }

  /* 状态栏适配 */
  .ranking-status-bar {
    padding: 10px 15px;
    flex-wrap: wrap;
    gap: 10px;
  }

  .status-text {
    font-size: 14px;
  }

  /* 完成操作区域适配 */
  .complete-actions {
    flex-direction: column;
    gap: 12px;
    text-align: center;
    padding: 15px;
  }

  .complete-message {
    font-size: 16px;
  }

  /* 步骤条适配 */
  :deep(.n-steps) {
    flex-wrap: wrap;
  }

  :deep(.n-step) {
    flex: 1;
    min-width: 80px;
  }

  :deep(.n-step .n-step-content__title) {
    font-size: 12px;
  }

  :deep(.n-step .n-step-content__description) {
    font-size: 10px;
  }
}

/* 超小屏幕适配 */
@media screen and (max-width: 400px) {
  .tier-label {
    width: 45px;
    font-size: 14px;
  }

  .rank-item {
    width: 40px;
    height: 40px;
  }

  .floating-card .large-rank-item {
    width: 80px;
    height: 80px;
  }

  .main-title {
    font-size: 20px;
  }

  .title-input {
    font-size: 20px;
  }
}
</style>

