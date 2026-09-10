<template>
  <div class="app-container">
    <!-- 左側設定控制台 -->
    <div class="control-panel">
      <h2>⚙️ 卡片與題詞設定</h2>

      <!-- 1. 版面方向 -->
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

      <!-- 2. 卡片大類別 -->
      <div class="form-group">
        <label>卡片類型：</label>
        <select v-model="cardCategory" @change="onCardCategoryChange">
          <option value="funeral">喪禮弔唁</option>
          <option value="celebration">慶賀祝典</option>
        </select>
      </div>

      <!-- ================= 喪禮專屬設定 ================= -->
      <template v-if="cardCategory === 'funeral'">
        <!-- 上款格式與後綴 -->
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
              <option value="custom">自行輸入</option>
            </select>
          </div>

          <!-- 上款內容預覽與微調 -->
          <input 
            type="text" 
            v-model="upperText" 
            class="full-input mt-2" 
            placeholder="上款文字 (例如：林公大明老先生 千古)"
          />
        </div>

        <!-- 中款身分與年齡搭配 -->
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

          <!-- 詞庫按鈕 -->
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

          <!-- 中款文字輸入框 -->
          <input 
            type="text" 
            v-model="middleText" 
            class="full-input mt-2" 
            placeholder="中款詞語 (點選上方或自行輸入)" 
          />
        </div>
      </template>

      <!-- ================= 慶賀專屬設定 ================= -->
      <template v-else>
        <div class="panel-section">
          <label class="section-title">慶賀細項：</label>
          <select v-model="celebrationType" class="full-input" @change="onCelebrationTypeChange">
            <option value="opening">開幕</option>
            <option value="moving">搬家</option>
            <option value="temple">宮廟</option>
          </select>

          <!-- 上款格式選擇 -->
          <div class="form-row mt-2">
            <select v-model="celebPrefix" @change="buildCelebrationUpper">
              <option value="恭祝">恭祝</option>
              <option value="恭賀">恭賀</option>
              <option value="custom">自行輸入</option>
            </select>
            <input 
              type="text" 
              v-model="celebTarget" 
              @input="buildCelebrationUpper" 
              placeholder="受禮對象 (如：鼎盛企業 / 天后宮)" 
            />
          </div>

          <input 
            type="text" 
            v-model="upperText" 
            class="full-input mt-2" 
            placeholder="上款文字" 
          />
        </div>

        <div class="panel-section">
          <label class="section-title">中款詞語 (免選身分，點擊直接套用)：</label>
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

          <input 
            type="text" 
            v-model="middleText" 
            class="full-input mt-2" 
            placeholder="中款詞語 (點選上方或自行輸入)" 
          />
        </div>
      </template>

      <!-- ================= 下款設定 ================= -->
      <div class="panel-section">
        <label class="section-title">下款設定：</label>
        <div class="form-group">
          <input type="text" v-model="unitText" placeholder="下款單位 (例：桃園市議會 / 衛生福利部)" />
        </div>
        <div class="form-group">
          <input type="text" v-model="nameText" placeholder="下款姓名 (支援多人 空格分開)" />
        </div>
        <div class="form-group">
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

      <div class="tip-box">
        💡 <b>操作指南</b>：
        <br />• <b>按住方塊拖動</b>：任意微調畫布上的位置。
        <br />• <b>按住右下角 ⤡ 拖拉</b>：隨意放大縮小字體尺寸。
      </div>
    </div>

    <!-- 右側畫布預覽區 -->
    <div class="canvas-viewport">
      <div 
        class="card-board" 
        :class="[
          isVertical ? 'mode-vertical' : 'mode-horizontal',
          !isVertical && cardCategory === 'celebration' ? 'style-floral' : ''
        ]"
      >
        <!-- 直式長條四角定位線 -->
        <template v-if="isVertical">
          <div class="crop-mark top-left"></div>
          <div class="crop-mark top-right"></div>
          <div class="crop-mark bottom-left"></div>
          <div class="crop-mark bottom-right"></div>
        </template>

        <!-- 1. 上款 -->
        <div 
          class="text-box upper-box"
          :style="getStyle('upper')"
          @pointerdown="startMove($event, 'upper')"
        >
          <span>{{ upperText }}</span>
          <div class="scale-handle" @pointerdown.stop="startResize($event, 'upper')">⤡</div>
        </div>

        <!-- 2. 中款大字 -->
        <div 
          class="text-box middle-box"
          :style="getStyle('middle')"
          @pointerdown="startMove($event, 'middle')"
        >
          <span>{{ middleText }}</span>
          <div class="scale-handle" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
        </div>

        <!-- 3. 下款單位 -->
        <div 
          class="text-box unit-box"
          :style="getStyle('unit')"
          @pointerdown="startMove($event, 'unit')"
        >
          <span>{{ unitText }}</span>
          <div class="scale-handle" @pointerdown.stop="startResize($event, 'unit')">⤡</div>
        </div>

        <!-- 4. 下款姓名 -->
        <div 
          class="text-box name-box"
          :style="getStyle('name')"
          @pointerdown="startMove($event, 'name')"
        >
          <span>{{ nameText }}</span>
          <div class="scale-handle" @pointerdown.stop="startResize($event, 'name')">⤡</div>
        </div>

        <!-- 5. 下款結尾敬語 -->
        <div 
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
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const isVertical = ref(true)
const cardCategory = ref('funeral') // 'funeral' | 'celebration'

// ----------------- 喪禮設定 -----------------
const funeralUpperFormat = ref('X媽X老夫人')
const funeralUpperSuffix = ref('千古')
const gender = ref('female')
const ageStage = ref('f_over80')

// 喪禮各年齡常用語
const funeralPhrases = {
  f_under49: ['芳華早謝', '遽促芳齡', '妝台月冷', '香消玉殞', '音容宛在'],
  f_50_79: ['懿範長存', '淑德永昭', '萱萎北堂', '慈雲縹緲'],
  f_over80: ['母儀千古', '駕返瑤池', '慈輝永昭', '寶婺星沉'],
  m_under49: ['星隕少微', '壯志未酬', '天不假年', '英年仙去', '音容宛在'],
  m_50_69: ['長才未盡', '棟折梁摧', '典則空留', '悵望音容', '英氣頓杳'],
  m_70_79: ['駕鶴西歸', '道範長存', '碩德堪欽', '儀型足式', '高風亮節'],
  m_over80: ['福壽全歸', '高山仰止', '碩德貽徽', '德望永昭', '典範長昭']
}

const currentFuneralPhrases = computed(() => {
  return funeralPhrases[ageStage.value] || []
})

// ----------------- 慶賀設定 -----------------
const celebrationType = ref('opening') // opening | moving | temple
const celebPrefix = ref('恭祝')
const celebTarget = ref('鴻運實業有限公司')

const celebPhrases = {
  opening: ['開幕誌慶', '開張大吉', '鴻圖大展', '駿業宏開', '生意興隆', '財源廣進', '客似雲來'],
  moving: ['喬遷之喜', '里仁為美', '金玉滿堂'],
  temple: ['聖誕千秋', '神威顯赫']
}

const currentCelebPhrases = computed(() => {
  return celebPhrases[celebrationType.value] || []
})

// ----------------- 文字內容 -----------------
const upperText = ref('陳媽李老夫人 仙逝')
const middleText = ref('母儀千古')
const unitText = ref('桃園市議會')
const nameText = ref('議員 李宗豪')
const suffixText = ref('敬輓')

// 切換喪禮性別時調整年齡預設
watch(gender, (val) => {
  ageStage.value = val === 'female' ? 'f_50_79' : 'm_50_69'
})

// 組裝喪禮上款
const buildFuneralUpper = () => {
  if (funeralUpperFormat.value === 'custom' || funeralUpperSuffix.value === 'custom') return
  upperText.value = `${funeralUpperFormat.value} ${funeralUpperSuffix.value}`
}

// 組裝慶賀上款
const buildCelebrationUpper = () => {
  if (celebPrefix.value === 'custom') return
  upperText.value = `${celebPrefix.value} ${celebTarget.value}`
}

// 切換大類別
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

// 切換慶賀子類別
const onCelebrationTypeChange = () => {
  middleText.value = currentCelebPhrases.value[0] || ''
}

// ----------------- 版面與拖曳縮放 -----------------
const defaultVertical = {
  upper:  { x: 440, y: 70,  size: 30 },
  middle: { x: 230, y: 150, size: 68 },
  unit:   { x: 100, y: 440, size: 22 },
  name:   { x: 100, y: 550, size: 26 },
  suffix: { x: 100, y: 670, size: 24 }
}

const defaultHorizontal = {
  upper:  { x: 140, y: 65,  size: 28 },
  middle: { x: 160, y: 180, size: 58 },
  unit:   { x: 270, y: 310, size: 22 },
  name:   { x: 220, y: 360, size: 24 },
  suffix: { x: 390, y: 360, size: 24 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultVertical)))

const switchOrientation = (vertical) => {
  isVertical.value = vertical
  resetPositions()
}

const resetPositions = () => {
  const target = isVertical.value ? defaultVertical : defaultHorizontal
  layout.value = JSON.parse(JSON.stringify(target))
}

const getStyle = (key) => {
  const item = layout.value[key]
  return {
    left: `${item.x}px`,
    top: `${item.y}px`,
    fontSize: `${item.size}px`
  }
}

let activeKey = null
let currentAction = null // 'move' | 'resize'
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
  const dx = e.clientX - startX
  const dy = e.clientY - startY

  if (currentAction === 'move') {
    layout.value[activeKey].x = originX + dx
    layout.value[activeKey].y = originY + dy
  } else if (currentAction === 'resize') {
    const scaleDelta = (dx + dy) / 3
    const newSize = Math.max(14, Math.min(120, Math.round(originSize + scaleDelta)))
    layout.value[activeKey].size = newSize
  }
}

const onPointerUp = () => {
  activeKey = null
  currentAction = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
}
</script>

<style scoped>
.app-container {
  display: flex;
  height: 100vh;
  background-color: #f0f2f5;
  font-family: "DFKai-SB", "BiauKai", "Kaiti TC", "KaiTi", serif;
  overflow: hidden;
}

/* 控制面板 */
.control-panel {
  width: 400px;
  background: #ffffff;
  padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.08);
  overflow-y: auto;
  font-family: sans-serif;
  z-index: 10;
}
.control-panel h2 {
  font-size: 18px;
  margin: 0 0 14px 0;
}
.panel-section {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 12px;
}
.section-title {
  display: block;
  font-size: 13px;
  font-weight: bold;
  color: #1f2937;
  margin-bottom: 6px;
}
.form-group {
  margin-bottom: 10px;
}
.form-group label {
  display: block;
  font-size: 13px;
  font-weight: bold;
  margin-bottom: 4px;
}
.form-row {
  display: flex;
  gap: 8px;
}
.form-row select, .form-row input {
  flex: 1;
}
.full-input {
  width: 100%;
  box-sizing: border-box;
}
.mt-2 {
  margin-top: 8px;
}
input, select {
  padding: 7px 9px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 13px;
  box-sizing: border-box;
}
.radio-row {
  display: flex;
  gap: 16px;
  font-size: 13px;
  margin-bottom: 6px;
}

/* 方向切換按鈕 */
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
  font-weight: bold;
  border-radius: 4px;
  cursor: pointer;
}
.btn-group button.active {
  background: #2563eb;
  color: white;
}

/* 常用語按鈕 */
.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}
.tag-btn {
  background: #eff6ff;
  color: #1e40af;
  border: 1px solid #bfdbfe;
  border-radius: 4px;
  padding: 4px 7px;
  font-size: 13px;
  cursor: pointer;
  font-family: "DFKai-SB", "BiauKai", serif;
}
.tag-btn:hover {
  background: #dbeafe;
}

.reset-btn {
  width: 100%;
  padding: 8px;
  background: #f3f4f6;
  border: 1px dashed #9ca3af;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-size: 13px;
}
.tip-box {
  background: #f0fdf4;
  border-left: 3px solid #22c55e;
  padding: 8px 10px;
  font-size: 12px;
  color: #166534;
  margin-top: 10px;
  line-height: 1.5;
}

/* 畫布視窗 */
.canvas-viewport {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: auto;
  padding: 20px;
}

.card-board {
  background: #ffffff;
  position: relative;
  box-shadow: 0 10px 25px rgba(0,0,0,0.12);
  user-select: none;
  touch-action: none;
}

/* 直式模式 */
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

/* 裁切標記 */
.crop-mark {
  position: absolute;
  width: 12px;
  height: 12px;
  border-color: #9ca3af;
  border-style: solid;
}
.crop-mark.top-left { top: 15px; left: 15px; border-width: 1px 0 0 1px; }
.crop-mark.top-right { top: 15px; right: 15px; border-width: 1px 1px 0 0; }
.crop-mark.bottom-left { bottom: 15px; left: 15px; border-width: 0 0 1px 1px; }
.crop-mark.bottom-right { bottom: 15px; right: 15px; border-width: 0 1px 1px 0; }

/* 橫式模式 */
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
  background: #ffffff;
  border: 10px solid #fce7f3;
  box-shadow: inset 0 0 50px rgba(244, 114, 182, 0.15), 0 10px 25px rgba(0,0,0,0.1);
}

/* 拖曳方塊 */
.text-box {
  position: absolute;
  cursor: move;
  font-weight: bold;
  padding: 4px 6px;
  border: 1px dashed transparent;
  white-space: nowrap;
  display: inline-block;
  line-height: 1.2;
}
.text-box:hover {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.04);
}

/* 縮放手柄 */
.scale-handle {
  position: absolute;
  right: -8px;
  bottom: -8px;
  width: 18px;
  height: 18px;
  background: #3b82f6;
  color: white;
  border-radius: 3px;
  font-size: 12px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: nwse-resize;
  opacity: 0;
  transition: opacity 0.15s;
}
.text-box:hover .scale-handle {
  opacity: 1;
}

/* 手機適配 */
@media (max-width: 768px) {
  .app-container {
    flex-direction: column;
    height: auto;
    overflow-y: auto;
  }
  .control-panel {
    width: 100%;
  }
  .canvas-viewport {
    padding: 10px;
    overflow: hidden;
  }
  .card-board.mode-vertical {
    transform: scale(0.55);
    transform-origin: top center;
    margin-bottom: -360px;
  }
  .card-board.mode-horizontal {
    transform: scale(0.48);
    transform-origin: top center;
    margin-bottom: -240px;
  }
}
</style>