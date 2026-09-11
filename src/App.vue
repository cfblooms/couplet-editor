<template>
  <div class="main-wrapper">
    <!-- 頂端模式切換列 (列印時自動隱藏) -->
    <header class="no-print top-nav">
      <div class="nav-title">🌸 花藝營運管理系統</div>
      <div class="nav-tabs">
        <button 
          type="button" 
          :class="{ active: currentTab === 'couplet' }" 
          @click="currentTab = 'couplet'"
        >
          🎴 花卡 / 輓聯編輯器
        </button>
        <button 
          type="button" 
          :class="{ active: currentTab === 'receipt' }" 
          @click="currentTab = 'receipt'"
        >
          📄 訂單 A5 簽收單
        </button>
      </div>
    </header>

    <!-- ================= 模式 1：花卡/輓聯編輯器 ================= -->
    <div v-if="currentTab === 'couplet'" class="app-container no-print">
      <!-- 左側設定控制台 -->
      <div class="control-panel">
        <h2>⚙️ 卡片與題詞設定</h2>

        <!-- 版面切換 -->
        <div class="form-group">
          <label>版面模式：</label>
          <div class="btn-group">
            <button type="button" :class="{ active: isVertical }" @click="switchOrientation(true)">
              直式 (傳統輓聯)
            </button>
            <button type="button" :class="{ active: !isVertical }" @click="switchOrientation(false)">
              橫式 (現代花卡)
            </button>
          </div>
        </div>

        <!-- 類型切換 -->
        <div class="form-group">
          <label>卡片類型：</label>
          <select v-model="cardCategory" @change="onCardCategoryChange">
            <option value="funeral">喪禮弔唁</option>
            <option value="celebration">慶賀祝典</option>
          </select>
        </div>

        <!-- ================= 喪禮設定 ================= -->
        <template v-if="cardCategory === 'funeral'">
          <div class="panel-section">
            <label class="section-title">上款稱謂組合：</label>
            <div class="form-row">
              <select v-model="funeralUpperFormat" @change="buildFuneralUpper">
                <option value="X媽X老夫人">X媽X老夫人</option>
                <option value="X媽X夫人">X媽X夫人</option>
                <option value="X公X老先生">X公X老先生</option>
                <option value="X公X先生">X公X先生</option>
                <option value="X女士">X女士</option>
                <option value="X先生">X先生</option>
                <option value="custom">自行輸入</option>
              </select>

              <select v-model="funeralUpperSuffix" @change="buildFuneralUpper">
                <option value="仙逝">仙逝</option>
                <option value="千古">千古</option>
                <option value="靈前">靈前</option>
                <option value="冥前">冥前</option>
                <option value="便覽">便覽</option>
                <option value="淑靈">淑靈</option>
                <option value="蓮前">蓮前</option>
              </select>
            </div>
            <input type="text" v-model="upperText" class="full-input mt-2" placeholder="上款文字" />
          </div>

          <div class="panel-section">
            <label class="section-title">中款常用語搭配 (身分與年齡)：</label>
            <div class="radio-row">
              <label><input type="radio" value="female" v-model="gender" /> 女性</label>
              <label><input type="radio" value="male" v-model="gender" /> 男性</label>
            </div>

            <select v-model="ageStage" class="full-input mt-2">
              <template v-if="gender === 'female'">
                <option value="f_under49">49歲以下（稱女士）</option>
                <option value="f_50_79">50至79歲（稱夫人／女士）</option>
                <option value="f_over80">80歲以上（稱老夫人）</option>
              </template>
              <template v-else>
                <option value="m_under49">49歲以下（稱先生）</option>
                <option value="m_50_69">50至69歲（稱先生）</option>
                <option value="m_70_79">70至79歲（稱老先生）</option>
                <option value="m_over80">80歲以上（稱老先生）</option>
              </template>
            </select>

            <div class="tags-container mt-2">
              <button 
                type="button" 
                v-for="phrase in currentFuneralPhrases" 
                :key="phrase" 
                class="tag-btn"
                @click="middleText = phrase"
              >
                {{ phrase }}
              </button>
            </div>
            <input type="text" v-model="middleText" class="full-input mt-2" placeholder="中款詞語" />
          </div>
        </template>

        <!-- ================= 慶賀設定 ================= -->
        <template v-else>
          <div class="panel-section">
            <label class="section-title">慶賀細項：</label>
            <select v-model="celebrationType" class="full-input" @change="onCelebrationTypeChange">
              <option value="opening">開幕</option>
              <option value="moving">搬家</option>
              <option value="temple">宮廟</option>
            </select>

            <div class="form-row mt-2">
              <select v-model="celebPrefix" @change="buildCelebrationUpper">
                <option value="恭祝">恭祝</option>
                <option value="恭賀">恭賀</option>
              </select>
              <input type="text" v-model="celebTarget" @input="buildCelebrationUpper" placeholder="受禮對象" />
            </div>
            <input type="text" v-model="upperText" class="full-input mt-2" placeholder="上款文字" />
          </div>

          <div class="panel-section">
            <label class="section-title">中款詞語：</label>
            <div class="tags-container">
              <button 
                type="button" 
                v-for="phrase in currentCelebPhrases" 
                :key="phrase" 
                class="tag-btn"
                @click="middleText = phrase"
              >
                {{ phrase }}
              </button>
            </div>
            <input type="text" v-model="middleText" class="full-input mt-2" placeholder="中款詞語" />
          </div>
        </template>

        <!-- ================= 下款設定 (增加五格自訂空白) ================= -->
        <div class="panel-section">
          <label class="section-title">下款設定（共 5 格可自由填寫，空白不顯示）：</label>
          <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
            <span class="line-num">格 {{ idx + 1 }}</span>
            <input 
              type="text" 
              v-model="item.text" 
              :placeholder="getPlaceholder(idx)"
            />
          </div>

          <div class="form-group mt-2">
            <label>結尾敬詞：</label>
            <select v-model="suffixText">
              <option value="敬輓">敬輓</option>
              <option value="泣輓">泣輓</option>
              <option value="拜輓">拜輓</option>
              <option value="敬賀">敬賀</option>
              <option value="恭賀">恭賀</option>
              <option value="拜賀">拜賀</option>
              <option value="謹致">謹致</option>
            </select>
          </div>
        </div>

        <button type="button" class="reset-btn" @click="resetPositions">↺ 重設排版預設位置</button>
      </div>

      <!-- 右側卡片畫布檢視區 -->
      <div class="canvas-viewport" ref="viewportRef">
        <!-- 縮放工具列 -->
        <div class="zoom-toolbar">
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.3, +(zoomLevel - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
          <button type="button" class="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
        </div>

        <!-- 彈性縮放容器 (解決手機版 A4 跑掉核心) -->
        <div 
          class="card-scaler-container" 
          :style="{
            width: (isVertical ? 560 : 720) * zoomLevel + 'px',
            height: (isVertical ? 840 : 490) * zoomLevel + 'px'
          }"
        >
          <div 
            class="card-board" 
            :class="[
              isVertical ? 'mode-vertical' : 'mode-horizontal',
              !isVertical && cardCategory === 'celebration' ? 'style-floral' : ''
            ]"
            :style="{
              transform: `scale(${zoomLevel})`,
              transformOrigin: 'top left'
            }"
          >
            <!-- 直式四角裁切線 -->
            <template v-if="isVertical">
              <div class="crop-mark top-left"></div>
              <div class="crop-mark top-right"></div>
              <div class="crop-mark bottom-left"></div>
              <div class="crop-mark bottom-right"></div>
            </template>

            <!-- 1. 上款 -->
            <div 
              v-if="upperText.trim()"
              class="text-box upper-box"
              :style="getStyle('upper')"
              @pointerdown="startMove($event, 'upper')"
            >
              <span>{{ upperText }}</span>
              <div class="scale-handle" @pointerdown.stop="startResize($event, 'upper')">⤡</div>
            </div>

            <!-- 2. 中款大字 -->
            <div 
              v-if="middleText.trim()"
              class="text-box middle-box"
              :style="getStyle('middle')"
              @pointerdown="startMove($event, 'middle')"
            >
              <span>{{ middleText }}</span>
              <div class="scale-handle" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
            </div>

            <!-- 3. 下款 5 格自訂欄位 -->
            <template v-for="(item, idx) in bottomLines" :key="'bottom-' + idx">
              <div 
                v-if="item.text.trim()"
                class="text-box"
                :style="getStyle('bottom_' + idx)"
                @pointerdown="startMove($event, 'bottom_' + idx)"
              >
                <span>{{ item.text }}</span>
                <div class="scale-handle" @pointerdown.stop="startResize($event, 'bottom_' + idx)">⤡</div>
              </div>
            </template>

            <!-- 4. 結尾敬詞 -->
            <div 
              v-if="suffixText.trim()"
              class="text-box suffix-box"
              :style="getStyle('suffix')"
              @pointerdown="startMove($event, 'suffix')"
            >
              <span>{{ suffixText }}</span>
              <div class="scale-handle" @pointerdown.stop="startResize($event, 'suffix')">⤡</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= 模式 2：A5 橫式簽收單 ================= -->
    <div v-else class="receipt-container">
      <div class="control-panel no-print">
        <h2>📋 簽收單內容設定</h2>
        <div class="form-group">
          <label>花店店名：</label>
          <input type="text" v-model="receipt.shopName" />
        </div>
        <div class="form-group">
          <label>送達日期：</label>
          <input type="text" v-model="receipt.deliveryDate" />
        </div>
        <div class="form-group">
          <label>送達地址：</label>
          <input type="text" v-model="receipt.address" />
        </div>
        <div class="form-group">
          <label>收件單位 / 聯絡人 / 電話：</label>
          <textarea v-model="receipt.recipient" rows="2"></textarea>
        </div>
        <div class="form-group">
          <label>致贈 / 敬領單位：</label>
          <textarea v-model="receipt.giver" rows="2"></textarea>
        </div>
        <div class="form-group">
          <label>花禮品項內容：</label>
          <input type="text" v-model="receipt.item" />
        </div>
        <hr />
        <button type="button" class="print-action-btn" @click="printReceipt">🖨️ 一鍵列印 A5 簽收單</button>
      </div>

      <div class="receipt-preview-area">
        <div class="a5-receipt-sheet">
          <div class="sheet-column header-col">
            <div class="shop-title">{{ receipt.shopName }}</div>
            <div class="sheet-badge">簽收單</div>
          </div>
          <div class="sheet-column date-col">
            <span class="field-label">送達日期：</span>
            <span>{{ receipt.deliveryDate }}</span>
          </div>
          <div class="sheet-column address-col">
            <span class="field-label">送達地址：</span>
            <span>{{ receipt.address }}</span>
          </div>
          <div class="sheet-column recipient-col">
            <span>{{ receipt.recipient }}</span>
          </div>
          <div class="sheet-column giver-col">
            <span>{{ receipt.giver }}</span>
          </div>
          <div class="sheet-column item-col">
            <span>{{ receipt.item }}</span>
          </div>
          <div class="sheet-column sign-col">
            <span class="sign-title">簽收人：</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue'

const currentTab = ref('couplet')
const isVertical = ref(true)
const cardCategory = ref('funeral')
const zoomLevel = ref(1)
const viewportRef = ref(null)

// ----------------- 下款 5 格自訂欄位 -----------------
const bottomLines = ref([
  { text: '桃園市議會' },
  { text: '議員 李宗豪' },
  { text: '' },
  { text: '' },
  { text: '' }
])

const getPlaceholder = (idx) => {
  const hints = [
    '第 1 格（例：單位 / 公司）',
    '第 2 格（例：職稱與姓名 1）',
    '第 3 格（自訂聯名人 / 單位 2）',
    '第 4 格（自訂）',
    '第 5 格（自訂）'
  ]
  return hints[idx]
}

// ----------------- 喪禮設定 -----------------
const funeralUpperFormat = ref('X媽X老夫人')
const funeralUpperSuffix = ref('千古')
const gender = ref('female')
const ageStage = ref('f_over80')

const funeralPhrases = {
  f_under49: ['芳華早謝', '遽促芳齡', '妝台月冷', '香消玉殞', '音容宛在'],
  f_50_79: ['懿範長存', '淑德永昭', '萱萎北堂', '慈雲縹緲'],
  f_over80: ['母儀千古', '駕返瑤池', '慈輝永昭', '寶婺星沉'],
  m_under49: ['星隕少微', '壯志未酬', '天不假年', '英年仙去', '音容宛在'],
  m_50_69: ['長才未盡', '棟折梁摧', '典則空留', '悵望音容', '英氣頓杳'],
  m_70_79: ['駕鶴西歸', '道範長存', '碩德堪欽', '儀型足式', '高風亮節'],
  m_over80: ['福壽全歸', '高山仰止', '碩德貽徽', '德望永昭', '典範長昭']
}
const currentFuneralPhrases = computed(() => funeralPhrases[ageStage.value] || [])

// ----------------- 慶賀設定 -----------------
const celebrationType = ref('opening')
const celebPrefix = ref('恭祝')
const celebTarget = ref('鴻運實業有限公司')
const celebPhrases = {
  opening: ['開幕誌慶', '開張大吉', '鴻圖大展', '駿業宏開', '生意興隆', '財源廣進', '客似雲來'],
  moving: ['喬遷之喜', '里仁為美', '金玉滿堂'],
  temple: ['聖誕千秋', '神威顯赫']
}
const currentCelebPhrases = computed(() => celebPhrases[celebrationType.value] || [])

// ----------------- 內容文字 -----------------
const upperText = ref('陳媽李老夫人 仙逝')
const middleText = ref('母儀千古')
const suffixText = ref('敬輓')

// ----------------- 簽收單資料 -----------------
const receipt = ref({
  shopName: '花花戶花藝設計',
  deliveryDate: '115.9.03 送達',
  address: '桃園市桃園區縣府路 82 號 1 樓',
  recipient: '永全證券 陳柏榮總經理 03-3352155*510 江明麗秘書',
  giver: '敬領 中華民國證券商業同業公會 理事長 陳俊宏 秘書長 簡宏明',
  item: '蘭花乙盆'
})
const printReceipt = () => window.print()

// ----------------- 座標預設配置 -----------------
const defaultVertical = {
  upper:    { x: 440, y: 70,  size: 30 },
  middle:   { x: 230, y: 150, size: 68 },
  bottom_0: { x: 105, y: 380, size: 22 },
  bottom_1: { x: 105, y: 490, size: 26 },
  bottom_2: { x: 65,  y: 380, size: 22 },
  bottom_3: { x: 65,  y: 490, size: 24 },
  bottom_4: { x: 25,  y: 380, size: 22 },
  suffix:   { x: 105, y: 640, size: 24 }
}

const defaultHorizontal = {
  upper:    { x: 140, y: 65,  size: 28 },
  middle:   { x: 160, y: 180, size: 58 },
  bottom_0: { x: 240, y: 300, size: 21 },
  bottom_1: { x: 240, y: 345, size: 24 },
  bottom_2: { x: 240, y: 390, size: 21 },
  bottom_3: { x: 240, y: 430, size: 21 },
  bottom_4: { x: 240, y: 470, size: 21 },
  suffix:   { x: 400, y: 345, size: 24 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultVertical)))

const switchOrientation = (vertical) => {
  isVertical.value = vertical
  resetPositions()
  nextTick(() => autoFitZoom())
}

const resetPositions = () => {
  const target = isVertical.value ? defaultVertical : defaultHorizontal
  layout.value = JSON.parse(JSON.stringify(target))
}

const getStyle = (key) => {
  const item = layout.value[key] || { x: 50, y: 50, size: 22 }
  return {
    left: `${item.x}px`,
    top: `${item.y}px`,
    fontSize: `${item.size}px`
  }
}

// ----------------- 自適應手機螢幕縮放 -----------------
const autoFitZoom = () => {
  if (!viewportRef.value) return
  const availableWidth = viewportRef.value.clientWidth - 24
  const cardWidth = isVertical.value ? 560 : 720
  const fitScale = +(availableWidth / cardWidth).toFixed(2)
  zoomLevel.value = Math.min(Math.max(fitScale, 0.35), 1.0)
}

onMounted(() => {
  autoFitZoom()
  window.addEventListener('resize', autoFitZoom)
})

// ----------------- 拖拉與縮放 (精準換算 zoomLevel) -----------------
let activeKey = null
let currentAction = null
let startX = 0
let startY = 0
let originX = 0
let originY = 0
let originSize = 24

const startMove = (e, key) => {
  activeKey = key
  currentAction = 'move'
  startX = e.clientX
  startY = e.clientY
  originX = layout.value[key].x
  originY = layout.value[key].y
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
}

const startResize = (e, key) => {
  activeKey = key
  currentAction = 'resize'
  startX = e.clientX
  startY = e.clientY
  originSize = layout.value[key].size
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
}

const onPointerMove = (e) => {
  if (!activeKey) return
  // 除以當前縮放倍率，保證手指與文字 1:1 移動不脫鉤
  const dx = (e.clientX - startX) / zoomLevel.value
  const dy = (e.clientY - startY) / zoomLevel.value

  if (currentAction === 'move') {
    layout.value[activeKey].x = Math.round(originX + dx)
    layout.value[activeKey].y = Math.round(originY + dy)
  } else if (currentAction === 'resize') {
    const scaleDelta = (dx + dy) / 3
    layout.value[activeKey].size = Math.max(14, Math.min(120, Math.round(originSize + scaleDelta)))
  }
}

const onPointerUp = () => {
  activeKey = null
  currentAction = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
}

// ----------------- 聯動輔助 -----------------
watch(gender, (val) => {
  ageStage.value = val === 'female' ? 'f_50_79' : 'm_50_69'
})
const buildFuneralUpper = () => {
  if (funeralUpperFormat.value === 'custom') return
  upperText.value = `${funeralUpperFormat.value} ${funeralUpperSuffix.value}`
}
const buildCelebrationUpper = () => {
  if (celebPrefix.value === 'custom') return
  upperText.value = `${celebPrefix.value} ${celebTarget.value}`
}
const onCardCategoryChange = () => {
  if (cardCategory.value === 'funeral') {
    suffixText.value = '敬輓'
    buildFuneralUpper()
    middleText.value = currentFuneralPhrases.value[0] || ''
  } else {
    suffixText.value = '敬賀'
    buildCelebrationUpper()
    middleText.value = currentCelebPhrases.value[0] || ''
  }
}
const onCelebrationTypeChange = () => {
  middleText.value = currentCelebPhrases.value[0] || ''
}
</script>

<style scoped>
.main-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: "DFKai-SB", "BiauKai", "Kaiti TC", "KaiTi", serif;
  background-color: #f1f5f9;
}

/* 導覽列 */
.top-nav {
  height: 50px;
  background-color: #0f172a;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  font-family: sans-serif;
  flex-shrink: 0;
}
.nav-title {
  font-size: 15px;
  font-weight: bold;
}
.nav-tabs {
  display: flex;
  gap: 8px;
}
.nav-tabs button {
  background: #334155;
  color: #e2e8f0;
  border: none;
  padding: 6px 12px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 13px;
}
.nav-tabs button.active {
  background: #2563eb;
  color: white;
}

/* 核心容器 */
.app-container, .receipt-container {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* 左側面板 */
.control-panel {
  width: 380px;
  background: white;
  padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.06);
  overflow-y: auto;
  font-family: sans-serif;
  flex-shrink: 0;
  z-index: 10;
}
.control-panel h2 {
  font-size: 16px;
  margin: 0 0 12px 0;
}
.panel-section {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 10px;
}
.section-title {
  font-size: 13px;
  font-weight: bold;
  display: block;
  margin-bottom: 6px;
  color: #1e293b;
}
.form-group {
  margin-bottom: 10px;
}
.form-group label {
  display: block;
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 4px;
}
input, select, textarea {
  width: 100%;
  padding: 7px 9px;
  border: 1px solid #cbd5e1;
  border-radius: 4px;
  font-size: 13px;
  box-sizing: border-box;
}

/* 下款五格清單 */
.bottom-input-group {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 6px;
}
.line-num {
  font-size: 12px;
  font-weight: bold;
  color: #64748b;
  width: 38px;
  flex-shrink: 0;
}
.bottom-input-group input {
  flex: 1;
}

.form-row {
  display: flex;
  gap: 6px;
}
.btn-group {
  display: flex;
  gap: 8px;
}
.btn-group button {
  flex: 1;
  padding: 7px;
  border: 1px solid #2563eb;
  background: white;
  color: #2563eb;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.btn-group button.active {
  background: #2563eb;
  color: white;
}
.radio-row {
  display: flex;
  gap: 14px;
  font-size: 13px;
}
.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}
.tag-btn {
  background: #eff6ff;
  color: #1e40af;
  border: 1px solid #bfdbfe;
  padding: 3px 6px;
  font-size: 12px;
  border-radius: 4px;
  cursor: pointer;
  font-family: "DFKai-SB", serif;
}
.reset-btn {
  width: 100%;
  padding: 8px;
  background: #f1f5f9;
  border: 1px dashed #94a3b8;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.mt-2 { margin-top: 6px; }
.full-input { width: 100%; }

/* 畫布視窗 */
.canvas-viewport {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: auto;
  padding: 16px;
  position: relative;
  background-color: #cbd5e1;
}

/* 縮放工具列 */
.zoom-toolbar {
  display: flex;
  align-items: center;
  gap: 6px;
  background: white;
  padding: 5px 12px;
  border-radius: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  margin-bottom: 12px;
  font-family: sans-serif;
  z-index: 5;
}
.zoom-btn {
  width: 26px;
  height: 26px;
  border: 1px solid #cbd5e1;
  background: #f8fafc;
  border-radius: 50%;
  cursor: pointer;
  font-weight: bold;
}
.zoom-text {
  font-size: 13px;
  font-weight: bold;
  min-width: 44px;
  text-align: center;
}
.fit-btn {
  background: #2563eb;
  color: white;
  border: none;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
  cursor: pointer;
  font-weight: bold;
}

/* 實體卡片 */
.card-scaler-container {
  position: relative;
  transition: width 0.1s, height 0.1s;
}
.card-board {
  background: #ffffff;
  position: absolute;
  box-shadow: 0 10px 30px rgba(0,0,0,0.18);
  user-select: none;
  touch-action: none;
}
.card-board.mode-vertical {
  width: 560px;
  height: 840px;
  border: 1px solid #d1d5db;
}
.card-board.mode-vertical .text-box {
  writing-mode: vertical-rl;
  text-orientation: upright;
  letter-spacing: 6px;
}
.card-board.mode-vertical .middle-box {
  letter-spacing: 14px;
  font-weight: 900;
}
.card-board.mode-horizontal {
  width: 720px;
  height: 490px;
  border: 1px solid #e5e7eb;
}
.card-board.mode-horizontal .text-box {
  writing-mode: horizontal-tb;
  letter-spacing: 4px;
}
.card-board.mode-horizontal .middle-box {
  letter-spacing: 12px;
  font-weight: 900;
}
.card-board.style-floral {
  border: 10px solid #fce7f3;
  box-shadow: inset 0 0 50px rgba(244, 114, 182, 0.15), 0 10px 30px rgba(0,0,0,0.15);
}

/* 直式四角裁切線 */
.crop-mark {
  position: absolute;
  width: 12px;
  height: 12px;
  border-color: #94a3b8;
  border-style: solid;
}
.crop-mark.top-left { top: 15px; left: 15px; border-width: 1px 0 0 1px; }
.crop-mark.top-right { top: 15px; right: 15px; border-width: 1px 1px 0 0; }
.crop-mark.bottom-left { bottom: 15px; left: 15px; border-width: 0 0 1px 1px; }
.crop-mark.bottom-right { bottom: 15px; right: 15px; border-width: 0 1px 1px 0; }

/* 拖曳字塊 */
.text-box {
  position: absolute;
  cursor: move;
  font-weight: bold;
  padding: 3px 5px;
  white-space: nowrap;
  line-height: 1.2;
}
.text-box:hover {
  outline: 1px dashed #2563eb;
  background: rgba(37, 99, 235, 0.04);
}
.scale-handle {
  position: absolute;
  right: -7px;
  bottom: -7px;
  width: 17px;
  height: 17px;
  background: #2563eb;
  color: white;
  border-radius: 3px;
  font-size: 11px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: nwse-resize;
  opacity: 0.8;
}

/* 簽收單樣式 */
.receipt-preview-area {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: auto;
  padding: 20px;
  background-color: #475569;
}
.a5-receipt-sheet {
  width: 210mm;
  height: 148mm;
  background: #ffffff;
  padding: 16mm 18mm;
  box-sizing: border-box;
  display: flex;
  flex-direction: row-reverse;
  justify-content: flex-start;
  gap: 16mm;
  writing-mode: vertical-rl;
  text-orientation: upright;
  font-family: "DFKai-SB", "BiauKai", serif;
  color: #111827;
}
.sheet-column {
  line-height: 1.6;
  letter-spacing: 2px;
  font-size: 16px;
  white-space: pre-wrap;
}
.header-col {
  display: flex;
  align-items: center;
  gap: 12px;
}
.shop-title {
  font-size: 21px;
  font-weight: bold;
  border-left: 2px solid #111;
  padding-left: 4px;
}
.sheet-badge {
  color: #dc2626;
  font-size: 26px;
  font-weight: 900;
  letter-spacing: 6px;
}
.sign-col {
  font-size: 18px;
  font-weight: bold;
  margin-left: auto;
}
.print-action-btn {
  width: 100%;
  padding: 12px;
  background: #16a34a;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
  margin-top: 10px;
}

/* 手機版排版調優 */
@media (max-width: 768px) {
  .app-container {
    flex-direction: column;
    overflow-y: auto;
  }
  .control-panel {
    width: 100%;
    max-height: 48vh;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  }
  .canvas-viewport {
    padding: 12px 6px;
  }
}

/* 列印模式 */
@media print {
  @page {
    size: A5 landscape;
    margin: 0;
  }
  body, html, .main-wrapper {
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
  }
  .no-print {
    display: none !important;
  }
  .receipt-preview-area {
    padding: 0 !important;
    background: white !important;
  }
  .a5-receipt-sheet {
    box-shadow: none !important;
    width: 210mm !important;
    height: 148mm !important;
    padding: 12mm 15mm !important;
  }
}
</style>