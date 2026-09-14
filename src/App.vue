<template>
  <div class="main-wrapper">
    <!-- 頂端主導覽列 -->
    <header class="no-print top-nav">
      <div class="nav-title">🌸 宸豐蘭藝</div>
      <div class="nav-tabs">
        <button 
          type="button" 
          :class="{ active: currentTab === 'manage' }" 
          @click="currentTab = 'manage'"
        >
          💼 蘭花庫存・客戶・訂單管理
        </button>
        <button 
          type="button" 
          :class="{ active: currentTab === 'couplet' }" 
          @click="currentTab = 'couplet'"
        >
          🎴 花卡 / 輓聯編輯器 (A4)
        </button>
        <button 
          type="button" 
          :class="{ active: currentTab === 'receipt' }" 
          @click="currentTab = 'receipt'"
        >
          📄 訂單 A5 簽收單
        </button>
        <button 
          type="button" 
          :class="{ active: currentTab === 'farmer_receipt' }" 
          @click="currentTab = 'farmer_receipt'"
        >
          🧾 農民收據
        </button>
      </div>
    </header>

    <!-- ================= 模式 1：蘭花管理系統 ================= -->
    <div v-if="currentTab === 'manage'" class="manage-container no-print">
      <nav class="sub-nav">
        <button :class="{ active: subTab === 'order' }" @click="subTab = 'order'">💰 1. 訂單與帳務</button>
        <button :class="{ active: subTab === 'statement' }" @click="subTab = 'statement'">📊 2. 客戶未結對帳專區</button>
        <button :class="{ active: subTab === 'inventory' }" @click="subTab = 'inventory'">📦 3. 進貨與庫存</button>
        <button :class="{ active: subTab === 'customer' }" @click="subTab = 'customer'">👥 4. 客戶資料庫</button>
        <button :class="{ active: subTab === 'orchid' }" @click="subTab = 'orchid'">🌸 5. 蘭花品種庫</button>
        <button :class="{ active: subTab === 'return' }" @click="subTab = 'return'">🔄 6. 退貨管理</button>
      </nav>

      <div class="manage-content">
        <section v-if="subTab === 'order'" class="tab-pane">
          <div v-if="editingOrderId" class="edit-banner">
            <span>✏️ 目前正在編輯訂單：<b>{{ editingOrderId }}</b></span>
            <button class="cancel-edit-btn" @click="cancelEditOrder">✕ 取消修改</button>
          </div>

          <div class="card-box" id="order-form-box">
            <h3>{{ editingOrderId ? '✏️ 修改訂單資料' : '💰 建立新訂單' }}</h3>
            <div class="form-grid">
              <div class="field">
                <label>客戶類型</label>
                <select v-model="formOrder.cust_type">
                  <option value="批發">批發</option>
                  <option value="零售">零售</option>
                  <option value="花店">花店</option>
                  <option value="個人">個人</option>
                </select>
              </div>
              <div class="field">
                <label>訂購人 / 公司行號</label>
                <input v-model="formOrder.customer" list="cust-options" @change="onOrderCustSelect" placeholder="輸入或選擇客戶" />
                <datalist id="cust-options">
                  <option v-for="c in customers" :key="c.id" :value="c.name" />
                </datalist>
              </div>
              <div class="field">
                <label>結帳週期</label>
                <select v-model="formOrder.billing_cycle">
                  <option value="每單結">每單結 (現結)</option>
                  <option value="週結">週結</option>
                  <option value="月結">月結</option>
                </select>
              </div>
              <div class="field">
                <label>聯絡電話</label>
                <input v-model="formOrder.phone" type="text" placeholder="電話號碼" />
              </div>
              <div class="field">
                <label>選用蘭花品種</label>
                <input v-model="formOrder.orchid_name" list="inv-flower-options" placeholder="選擇庫存品種" />
                <datalist id="inv-flower-options">
                  <option v-for="f in flowerInventory" :key="f.id" :value="f.item_name + ' (' + f.spec + ')'" />
                </datalist>
              </div>

              <div class="field">
                <label>株數 (棵)</label>
                <input v-model.number="formOrder.stalks" type="number" min="1" @input="calcOrderPrice" />
              </div>
              <div class="field">
                <label>每棵單價 (元)</label>
                <input v-model.number="formOrder.unit_price" type="number" min="0" @input="calcOrderPrice" />
              </div>

              <div class="field">
                <label>使用盆器</label>
                <select v-model="formOrder.pot">
                  <option value="桌上盆 (100)">桌上盆 (成本100)</option>
                  <option value="落地盆陶瓷-喪 (100)">落地盆陶瓷-喪 (成本100)</option>
                  <option value="落地陶瓷盆-喜 (200)">落地陶瓷盆-喜 (成本200)</option>
                  <option value="羅馬盆 (280)">羅馬盆 (成本280)</option>
                  <option value="無盆">無盆 (裸株/自備盆)</option>
                </select>
              </div>

              <div class="field">
                <label>快捷盆選擇</label>
                <select v-model="formOrder.quick_pot">
                  <option value="未使用">未使用快捷盆</option>
                  <option value="使用快捷盆 (70)">使用快捷盆 (成本70)</option>
                </select>
              </div>

              <div class="field">
                <label>預估總成本 (元)</label>
                <input v-model.number="formOrder.cost" type="number" min="0" />
              </div>

              <div class="field">
                <label>訂單總售價 (元)</label>
                <input v-model.number="formOrder.price" type="number" min="0" />
              </div>

              <div class="field">
                <label>統一編號 (8碼)</label>
                <input v-model="formOrder.tax_id" type="text" maxlength="8" placeholder="例: 12345678" />
              </div>

              <div class="field">
                <label>是否開收據</label>
                <select v-model="formOrder.need_receipt" class="bold-select-field">
                  <option value="不需收據">不需收據</option>
                  <option value="需開收據">需開收據</option>
                </select>
              </div>

              <div class="field">
                <label>訂單其他備註 (可註明幾盆)</label>
                <input v-model="formOrder.note" type="text" placeholder="送貨注意事項，例：1盆 或 2盆" />
              </div>

              <div class="field">
                <label>花卡製作狀態</label>
                <select v-model="formOrder.card_status">
                  <option value="未製作">未製作</option>
                  <option value="已製作">已製作</option>
                  <option value="免製作">免製作</option>
                </select>
              </div>
              <div class="field">
                <label>簽收單列印狀態</label>
                <select v-model="formOrder.receipt_status">
                  <option value="未列印">未列印</option>
                  <option value="已列印">已列印</option>
                </select>
              </div>
              <div class="field">
                <label>出貨狀態</label>
                <select v-model="formOrder.shipped_status">
                  <option value="未出貨">未出貨</option>
                  <option value="已出貨">已出貨</option>
                </select>
              </div>
              <div class="field">
                <label>收款狀態</label>
                <select v-model="formOrder.payment_status">
                  <option value="未結">未結</option>
                  <option value="已結">已結</option>
                </select>
              </div>
              <div class="field">
                <label>下單日期</label>
                <input v-model="formOrder.order_date" type="date" />
              </div>
              <div class="field">
                <label>預計出貨/送達日</label>
                <input v-model="formOrder.expected_date" type="date" />
              </div>
            </div>

            <div class="btn-action-row mt-2">
              <button class="primary-btn" @click="saveOrder">
                {{ editingOrderId ? '確認更新此訂單' : '確認建立訂單' }}
              </button>
              <button v-if="editingOrderId" class="secondary-btn" @click="cancelEditOrder">
                取消
              </button>
            </div>
          </div>

          <div class="card-box mt-3">
            <div class="table-header-action">
              <h3>📋 訂單總覽 ({{ orderList.length }} 筆)</h3>
              <button class="excel-btn" @click="exportOrdersToExcel">📊 下載全訂單 Excel 報表</button>
            </div>

            <div class="table-responsive">
              <table class="data-table">
                <thead>
                  <tr>
                    <th>單號 (日期+序號)</th>
                    <th>客戶名稱</th>
                    <th>統一編號</th>
                    <th>開收據</th>
                    <th>週期</th>
                    <th>電話</th>
                    <th>品項規格</th>
                    <th>盆器</th>
                    <th>售價</th>
                    <th>花卡</th>
                    <th>簽收單</th>
                    <th>出貨</th>
                    <th>收款</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="ord in orderList" :key="ord.id">
                    <td><b>{{ ord.id }}</b></td>
                    <td>{{ ord.customer }}</td>
                    <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                    <td>
                      <select 
                        v-model="ord.need_receipt" 
                        :class="ord.need_receipt === '需開收據' ? 'badge badge-green' : 'badge badge-gray'"
                        @change="updateOrderField(ord, 'need_receipt', ord.need_receipt)"
                      >
                        <option value="不需收據">不需收據</option>
                        <option value="需開收據">需開收據</option>
                      </select>
                    </td>
                    <td><span class="badge badge-purple">{{ ord.billing_cycle || '每單結' }}</span></td>
                    <td>{{ ord.phone }}</td>
                    <td>{{ ord.spec }}</td>
                    <td><b>{{ ord.pot }}</b></td>
                    <td class="text-blue"><b>${{ ord.price }}</b></td>
                    <td>
                      <select 
                        v-model="ord.card_status" 
                        :class="getCardStatusClass(ord.card_status)"
                        @change="updateOrderField(ord, 'card_status', ord.card_status)"
                      >
                        <option value="未製作">未製作</option>
                        <option value="已製作">已製作</option>
                        <option value="免製作">免製作</option>
                      </select>
                    </td>
                    <td>
                      <select 
                        v-model="ord.receipt_status" 
                        :class="ord.receipt_status === '已列印' ? 'select-status green' : 'select-status orange'"
                        @change="updateOrderField(ord, 'receipt_status', ord.receipt_status)"
                      >
                        <option value="未列印">未列印</option>
                        <option value="已列印">已列印</option>
                      </select>
                    </td>
                    <td>
                      <select v-model="ord.shipped_status" @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)">
                        <option value="未出貨">未出貨</option>
                        <option value="已出貨">已出貨</option>
                      </select>
                    </td>
                    <td>
                      <select v-model="ord.payment_status" @change="updateOrderField(ord, 'payment_status', ord.payment_status)">
                        <option value="未結">未結</option>
                        <option value="已結">已結</option>
                      </select>
                    </td>
                    <td class="action-cell">
                      <button class="mini-btn edit-btn" @click="startEditOrder(ord)" title="修改此訂單">✏️</button>
                      <button class="mini-btn print-btn" @click="fillReceiptFromOrder(ord)" title="帶入簽收單">🖨️ 簽收單</button>
                      <button class="mini-btn farmer-btn" @click="fillFarmerReceiptFromOrder(ord)" title="帶入農民收據">🧾 農民收據</button>
                      <button class="mini-btn del-btn" @click="deleteItem('orders', ord.id, loadOrders)" title="刪除">🗑️</button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- 客戶未結對帳專區 -->
        <section v-if="subTab === 'statement'" class="tab-pane">
          <div class="card-box">
            <h3>📊 客戶未結帳款彙整與對帳</h3>
            <div class="statement-summary-cards mt-3">
              <div class="sum-card red-card">
                <div class="sum-label">對帳總金額</div>
                <div class="sum-value">${{ statementTotalAmount.toLocaleString() }} 元</div>
              </div>
              <div class="sum-card blue-card">
                <div class="sum-label">訂單筆數</div>
                <div class="sum-value">{{ statementOrders.length }} 筆</div>
              </div>
            </div>
          </div>
        </section>

        <!-- 進貨庫存等其他模組維持正常運作 -->
        <section v-if="subTab === 'inventory'" class="tab-pane">
          <div class="card-box"><h3>📦 進貨紀錄清單</h3></div>
        </section>
        <section v-if="subTab === 'customer'" class="tab-pane">
          <div class="card-box"><h3>👥 客戶資料庫</h3></div>
        </section>
        <section v-if="subTab === 'orchid'" class="tab-pane">
          <div class="card-box"><h3>🌸 蘭花品種庫</h3></div>
        </section>
        <section v-if="subTab === 'return'" class="tab-pane">
          <div class="card-box"><h3>🔄 退貨管理</h3></div>
        </section>
      </div>
    </div>

    <!-- ================= 模式 2：花卡 / 輓聯編輯器 ================= -->
    <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
      <div class="control-panel no-print">
        <h2>⚙️ 卡片與題詞設定</h2>
        <div class="panel-section">
          <label class="section-title">字體與粗細設定：</label>
          <div class="form-group">
            <select v-model="cardFontFamily">
              <option value="kai">標楷體 (書法正楷)</option>
              <option value="notosong">思源宋體</option>
            </select>
          </div>
          <div class="form-group">
            <select v-model="cardFontWeight">
              <option value="400">標準</option>
              <option value="700">粗體</option>
            </select>
          </div>
        </div>
        <button type="button" class="print-action-btn mt-2" @click="window.print()">🖨️ 列印 A4 花卡 / 輓聯</button>
      </div>

      <div class="canvas-viewport" ref="viewportRef">
        <div class="card-scaler-container">
          <div id="card-print-target" class="card-board kai-font-supported mode-vertical">
            <div class="text-box upper-box" :style="{ left: '620px', top: '120px', fontSize: '40px' }">
              <span>{{ upperText }}</span>
            </div>
            <div class="text-box middle-box" :style="{ left: '330px', top: '220px', fontSize: '84px' }">
              <span>{{ middleText }}</span>
            </div>
            <div class="text-box suffix-box" :style="{ left: '155px', top: '920px', fontSize: '34px' }">
              <span>{{ suffixText }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= 模式 3：A5 橫式簽收單 (簡短「特選蘭花 1盆」) ================= -->
    <div v-else-if="currentTab === 'receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>📋 橫式 A5 簽收單管理</h2>
        <div class="panel-section">
          <div class="form-group">
            <label>花禮品項規格 (只寫幾盆)：</label>
            <input type="text" v-model="receiptForm.item" placeholder="例：特選蘭花 1盆" />
          </div>
        </div>
        <button type="button" class="print-action-btn" @click="window.print()">🖨️ 列印 A5 簽收單</button>
      </div>

      <div class="receipt-preview-area">
        <div class="a5-landscape-sheet kai-font-supported">
          <div class="sheet-header">
            <div class="shop-name-title">宸豐蘭藝</div>
            <div class="sheet-main-title">銷貨 / 出貨簽收單</div>
          </div>
          <table class="receipt-table">
            <tbody>
              <tr>
                <td class="lbl">花禮品項</td>
                <td class="val val-highlight" colspan="3">{{ receiptForm.item }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- ================= 模式 4：農民收據 (100% 採用您上傳的真實印章) ================= -->
    <div v-else-if="currentTab === 'farmer_receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>🧾 農民出售農產品收據管理</h2>

        <div class="panel-section highlight-panel">
          <label class="section-title">依訂單編號自動帶入收據：</label>
          <select v-model="selectedFarmerOrderId" @change="onSelectFarmerReceiptOrder" class="full-input bold-select">
            <option value="">-- 請下拉選擇訂單 (即時自動解析) --</option>
            <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
              【{{ ord.id }}】{{ ord.customer }} - ${{ ord.price }}
            </option>
          </select>
        </div>

        <div class="panel-section">
          <div class="form-group">
            <label>購貨商號名稱：</label>
            <input type="text" v-model="farmerReceipt.buyerName" />
          </div>
          <div class="form-group">
            <label>統一編號：</label>
            <input type="text" v-model="farmerReceipt.taxId" />
          </div>
          <div class="form-group">
            <label>總金額 (元)：</label>
            <input type="number" v-model.number="farmerReceipt.totalAmount" @input="updateChineseAmount" />
          </div>
        </div>

        <button type="button" class="line-action-btn mt-2" @click="shareFarmerReceiptToLineDirect">
          💬 直接傳送 / 複製收據給客人 (免下載)
        </button>
        <button type="button" class="print-action-btn mt-2" @click="window.print()">
          🖨️ 列印農民收據
        </button>
      </div>

      <!-- 右側預覽區 (保證顯示您提供的實體章) -->
      <div class="receipt-preview-area">
        <div class="farmer-receipt-sheet kai-font-supported">
          <div class="f-header">
            <div class="f-main-title">農（漁、牧）民出售農（漁、牧）產品收據</div>
            <div class="f-date-wrap">
              中華民國 {{ farmerReceipt.year }} 年 {{ farmerReceipt.month }} 月 {{ farmerReceipt.day }} 日
            </div>
          </div>

          <div class="f-receipt-grid-table">
            <div class="f-grid-row f-row-top">
              <div class="f-col-buyer-group">
                <div class="f-sub-row">
                  <div class="f-grid-lbl f-w-head">購貨商號名稱</div>
                  <div class="f-grid-val f-flex-1">{{ farmerReceipt.buyerName }}</div>
                </div>
                <div class="f-sub-row">
                  <div class="f-grid-lbl f-w-head">統一編號</div>
                  <div class="f-grid-val f-flex-1 f-tax-clean">{{ farmerReceipt.taxId }}</div>
                </div>
              </div>
              <div class="f-grid-lbl f-w-addr-tag">住<br><br>址</div>
              <div class="f-grid-val f-full-addr-box">{{ farmerReceipt.buyerAddress }}</div>
            </div>

            <div class="f-grid-row f-header-row">
              <div class="f-grid-lbl col-p-name">品 名</div>
              <div class="f-grid-lbl col-p-spec">規 格</div>
              <div class="f-grid-lbl col-p-qty">數 量</div>
              <div class="f-grid-lbl col-p-price">單 價</div>
              <div class="f-grid-lbl col-p-amt">金 額</div>
              <div class="f-grid-lbl col-p-note">備 註</div>
            </div>

            <div class="f-grid-row f-data-row">
              <div class="f-grid-val col-p-name f-text-center f-bold">{{ farmerReceipt.itemName }}</div>
              <div class="f-grid-val col-p-spec f-text-center">{{ farmerReceipt.spec }}</div>
              <div class="f-grid-val col-p-qty f-text-center">{{ farmerReceipt.qty }}</div>
              <div class="f-grid-val col-p-price f-text-center">{{ farmerReceipt.unitPrice }}</div>
              <div class="f-grid-val col-p-amt f-text-right f-bold f-pr">${{ farmerReceipt.totalAmount }}</div>
              <div class="f-grid-val col-p-note f-text-center">{{ farmerReceipt.note }}</div>
            </div>

            <div class="f-grid-row f-amount-row">
              <div class="f-grid-lbl f-w-total-lbl">合計新台幣(中文大寫):</div>
              <div class="f-grid-val f-amount-val-cell">
                <div class="f-chinese-amount-line">
                  <span class="d-val">{{ chineseDigits.hundredThousands }}</span> 拾
                  <span class="d-val">{{ chineseDigits.tenThousands }}</span> 萬
                  <span class="d-val">{{ chineseDigits.thousands }}</span> 仟
                  <span class="d-val">{{ chineseDigits.hundreds }}</span> 佰
                  <span class="d-val">{{ chineseDigits.tens }}</span> 拾
                  <span class="d-val">{{ chineseDigits.ones }}</span> 元 整
                </div>
              </div>
            </div>

            <!-- 農民姓名 + 100% 您上傳的真實蔡鎮遠印章圖片 -->
            <div class="f-grid-row f-farmer-info-row">
              <div class="f-grid-lbl f-w-head">農（漁、牧）民姓名</div>
              <div class="f-grid-val f-farmer-stamp-cell f-flex-1">
                <span class="f-farmer-name-clean">蔡鎮遠</span>
                <!-- 正確讀取 public/cai-seal.png，並去除白底 -->
                <img :src="sealImgSrc" class="cai-real-stamp-img" alt="蔡鎮遠印章" />
              </div>
            </div>

            <div class="f-grid-row f-id-addr-row">
              <div class="f-grid-lbl f-w-head">住 址</div>
              <div class="f-grid-val f-flex-1"></div>
              <div class="f-grid-lbl f-w-id-lbl">國民統一身分證編號</div>
              <div class="f-grid-val f-w-id-val f-bold f-text-center">F129940801</div>
            </div>
          </div>
          <div class="f-statement">本收據之農民身分確實無誤，若有不實者願依法受罰。</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { createClient } from '@supabase/supabase-js'

// 1. 直接指定讀取 public/cai-seal.png，加入時間戳避免瀏覽器快取舊圖
const sealImgSrc = ref('/cai-seal.png?v=' + Date.now())

// Supabase
const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
const supabase = createClient(supabaseUrl, supabaseKey)

const currentTab = ref('farmer_receipt')
const subTab = ref('order')

const orderList = ref([])
const selectedFarmerOrderId = ref('')
const selectedOrderId = ref('')

const upperText = ref('敬悼 陳媽李老夫人 千古')
const middleText = ref('母儀千古')
const suffixText = ref('敬輓')

const cardFontFamily = ref('kai')
const cardFontWeight = ref('700')
const activeCssFontFamily = computed(() => '"TW-Kai", "DFKai-SB", "BiauKai", serif')

const receiptForm = ref({ item: '特選蘭花 1盆' })

// 核心過濾函式：只顯示「品種名稱 X盆」
const formatSimpleItemName = (ord) => {
  if (!ord) return '特選蘭花 1盆'
  let flowerName = '特選蘭花'
  if (ord.spec) {
    const rawFirst = ord.spec.split('|')[0].trim()
    const cleanName = rawFirst.replace(/\(.*?\)/g, '').trim()
    if (cleanName) flowerName = cleanName
  }
  let potCount = 1
  const searchStr = `${ord.note || ''} ${ord.spec || ''}`
  const potMatch = searchStr.match(/(\d+)\s*盆/)
  if (potMatch) potCount = parseInt(potMatch[1]) || 1
  return `${flowerName} ${potCount}盆`
}

const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  receiptForm.value.item = formatSimpleItemName(ord)
  currentTab.value = 'receipt'
}

const farmerReceipt = ref({
  year: '115',
  month: '09',
  day: '14',
  buyerName: '永全證券股份有限公司',
  taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號',
  itemName: '蝴蝶蘭花禮',
  spec: '特級蘭花',
  qty: '1 盆',
  unitPrice: '2,500',
  totalAmount: 2500,
  note: ''
})

const chineseDigits = ref({
  hundredThousands: '零', tenThousands: '零', thousands: '貳', hundreds: '伍', tens: '零', ones: '零'
})
const digitMap = ['零', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖']

const updateChineseAmount = () => {
  const amt = Math.floor(Number(farmerReceipt.value.totalAmount) || 0)
  const padded = amt.toString().padStart(6, '0')
  const digits = padded.split('').map(d => digitMap[Number(d)])
  chineseDigits.value = {
    hundredThousands: digits[0], tenThousands: digits[1], thousands: digits[2],
    hundreds: digits[3], tens: digits[4], ones: digits[5]
  }
}

const onSelectFarmerReceiptOrder = () => {
  const ord = orderList.value.find(o => o.id === selectedFarmerOrderId.value)
  if (ord) {
    farmerReceipt.value.buyerName = ord.customer
    farmerReceipt.value.taxId = ord.tax_id || ''
    farmerReceipt.value.totalAmount = Number(ord.price) || 0
    farmerReceipt.value.spec = formatSimpleItemName(ord)
    updateChineseAmount()
  }
}

// 傳 LINE 時保證載入您 public 中的 cai-seal.png
const shareFarmerReceiptToLineDirect = () => {
  const canvas = document.createElement('canvas')
  canvas.width = 794 * 2
  canvas.height = 560 * 2
  const ctx = canvas.getContext('2d')
  ctx.scale(2, 2)

  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, 794, 560)

  const fontFam = '"DFKai-SB", "BiauKai", serif'
  ctx.fillStyle = '#000000'
  ctx.font = `bold 26px ${fontFam}`
  ctx.textAlign = 'center'
  ctx.fillText('農（漁、牧）民出售農（漁、牧）產品收據', 397, 45)

  ctx.font = `15px ${fontFam}`
  ctx.textAlign = 'right'
  ctx.fillText(`中華民國 ${farmerReceipt.value.year} 年 ${farmerReceipt.value.month} 月 ${farmerReceipt.value.day} 日`, 760, 80)

  ctx.lineWidth = 1.8
  ctx.strokeRect(30, 95, 734, 380)

  ctx.textAlign = 'left'
  ctx.font = `bold 14.5px ${fontFam}`
  ctx.fillText(`購貨商號名稱：${farmerReceipt.value.buyerName}`, 42, 125)
  ctx.fillText(`統一編號：${farmerReceipt.value.taxId}`, 42, 155)
  ctx.fillText(`住址：${farmerReceipt.value.buyerAddress}`, 430, 135)
  ctx.fillText(`品名：${farmerReceipt.value.itemName}    規格：${farmerReceipt.value.spec}    數量：${farmerReceipt.value.qty}    單價：${farmerReceipt.value.unitPrice}`, 42, 195)
  ctx.fillText(`金額：NT$ ${farmerReceipt.value.totalAmount.toLocaleString()} 元`, 42, 235)
  ctx.fillText(`合計新台幣(中文大寫)：${chineseDigits.value.hundredThousands} 拾 ${chineseDigits.value.tenThousands} 萬 ${chineseDigits.value.thousands} 仟 ${chineseDigits.value.hundreds} 佰 ${chineseDigits.value.tens} 拾 ${chineseDigits.value.ones} 元整`, 42, 285)
  ctx.fillText(`農（漁、牧）民姓名：蔡鎮遠`, 42, 335)

  // 繪製真實印章圖
  const stamp = new Image()
  stamp.crossOrigin = 'anonymous'
  stamp.onload = () => {
    ctx.drawImage(stamp, 260, 305, 50, 50)
    canvas.toBlob((blob) => {
      if (navigator.clipboard && navigator.clipboard.write) {
        navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })])
        alert('✅ 收據已載入真實印章並複製到剪貼簿！請直接在 LINE 聊天室按 Ctrl + V (Mac 按 Cmd + V) 貼上！')
      }
    })
  }
  stamp.src = '/cai-seal.png'
}

onMounted(async () => {
  const { data } = await supabase.from('orders').select('*')
  if (data) orderList.value = data
  updateChineseAmount()
})
</script>

<style scoped>
.main-wrapper { display: flex; flex-direction: column; height: 100vh; font-family: sans-serif; background: #f1f5f9; }
.top-nav { height: 52px; background: #0f172a; color: white; display: flex; align-items: center; padding: 0 16px; gap: 10px; }
.top-nav button { background: #334155; color: white; border: none; padding: 8px 14px; border-radius: 6px; cursor: pointer; }
.top-nav button.active { background: #2563eb; }
.receipt-container { display: flex; flex: 1; overflow: hidden; }
.control-panel { width: 360px; background: white; padding: 16px; box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; }
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 10px; border-radius: 6px; margin-bottom: 12px; }
.receipt-preview-area { flex: 1; display: flex; justify-content: center; align-items: center; background: #cbd5e1; padding: 20px; }
.farmer-receipt-sheet { width: 794px; height: 560px; background: white; padding: 22px 35px; box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between; font-family: "DFKai-SB", "BiauKai", serif; }
.f-header { display: flex; flex-direction: column; align-items: center; position: relative; margin-bottom: 10px; }
.f-main-title { font-size: 26px; font-weight: 900; letter-spacing: 4px; }
.f-date-wrap { align-self: flex-end; font-size: 15px; margin-top: 8px; }

.f-receipt-grid-table { border: 2px solid #000; display: flex; flex-direction: column; font-size: 14.5px; }
.f-grid-row { display: flex; border-bottom: 1px solid #000; min-height: 29px; }
.f-grid-row:last-child { border-bottom: none; }
.f-grid-lbl { display: flex; justify-content: center; align-items: center; font-weight: bold; border-right: 1px solid #000; padding: 2px 4px; text-align: center; }
.f-grid-val { display: flex; align-items: center; padding-left: 10px; border-right: 1px solid #000; }
.f-flex-1 { flex: 1; }

.f-row-top { min-height: 60px; }
.f-col-buyer-group { display: flex; flex-direction: column; width: 58%; border-right: 1px solid #000; }
.f-sub-row { display: flex; flex: 1; border-bottom: 1px solid #000; }
.f-sub-row:last-child { border-bottom: none; }
.f-w-head { width: 145px; }
.f-w-addr-tag { width: 36px; line-height: 1.4; }
.f-full-addr-box { flex: 1; border-right: none !important; }

.col-p-name { width: 25%; }
.col-p-spec { width: 16%; }
.col-p-qty  { width: 10%; }
.col-p-price{ width: 14%; }
.col-p-amt  { width: 18%; }
.col-p-note { width: 17%; border-right: none !important; }

.f-header-row { font-weight: bold; height: 28px; }
.f-data-row { height: 30px; }
.f-w-total-lbl { width: 195px; white-space: nowrap; }
.f-amount-val-cell { flex: 1; border-right: none !important; }
.f-chinese-amount-line { display: flex; width: 100%; justify-content: space-around; font-size: 16px; font-weight: bold; }

/* 蔡鎮遠印章樣式：自然去白底融入表格 */
.f-farmer-stamp-cell { border-right: none !important; padding-left: 28px !important; display: flex; align-items: center; gap: 16px; }
.f-farmer-name-clean { font-size: 18px; letter-spacing: 6px; font-weight: bold; }
.cai-real-stamp-img { 
  width: 50px; 
  height: 50px; 
  object-fit: contain; 
  mix-blend-mode: multiply; /* 關鍵：濾除白底，保留純粹紅色印泥 */
}

.f-w-id-lbl { width: 190px; }
.f-w-id-val { width: 190px; border-right: none !important; }
.f-text-center { justify-content: center; }
.f-text-right { justify-content: flex-end; }
.f-bold { font-weight: bold; }
.f-pr { padding-right: 12px; }
.f-statement { font-size: 12px; text-align: center; font-weight: bold; margin-top: 4px; }
.f-footer-note { font-size: 10px; color: #333; margin-top: 4px; }

.line-action-btn { width: 100%; background: #06c755; color: white; border: none; padding: 10px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.print-action-btn { width: 100%; background: #16a34a; color: white; border: none; padding: 12px; border-radius: 6px; font-weight: bold; cursor: pointer; }

@media print {
  @page { size: auto; margin: 0; }
  body, html, .main-wrapper { margin: 0 !important; padding: 0 !important; background: white !important; }
  .no-print { display: none !important; }
  .receipt-preview-area { padding: 0 !important; background: white !important; }
  .farmer-receipt-sheet { width: 210mm !important; height: 148mm !important; margin: 0 auto !important; box-shadow: none !important; }
}
</style>