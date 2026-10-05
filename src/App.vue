<template>
  <div class="main-wrapper">
    <!-- ================= 內部通行碼驗證畫面 ================= -->
    <div v-if="!isAuthenticated" class="auth-lock-overlay">
      <div class="auth-lock-card">
        <div class="lock-icon">🔒</div>
        <h2>宸豐蘭藝・內部管理系統</h2>
        <p class="lock-subtitle">本系統僅限內部工作人員使用，請輸入管理密碼</p>
        
        <div class="lock-form">
          <input 
            type="password" 
            v-model="inputPasscode" 
            placeholder="請輸入內部通行密碼" 
            class="lock-input"
            autofocus
            @keydown.enter.prevent="handleLogin"
          />
          <button type="button" class="lock-btn" @click.prevent="handleLogin">
            驗證並進入系統 ➔
          </button>
        </div>
        <div v-if="authError" class="lock-error-text">
          ⚠️ 密碼錯誤，請重新輸入！
        </div>
        <div class="lock-tip">
          💡 自己人的手機與電腦登入後會自動保持登入，下次開啟無需重複輸入。
        </div>
      </div>
    </div>

    <!-- ================= 系統主畫面 ================= -->
    <div v-else class="system-root">
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
            🎴 花卡 / 輓聯編輯器 (A4/A5)
          </button>
          <button 
            type="button" 
            :class="{ active: currentTab === 'receipt' }" 
            @click="currentTab = 'receipt'"
          >
            📄 訂單 A5 簽收單 (支援線上簽名)
          </button>
          <button 
            type="button" 
            :class="{ active: currentTab === 'farmer_receipt' }" 
            @click="currentTab = 'farmer_receipt'"
          >
            🧾 農民收據
          </button>
          <button 
            type="button" 
            class="logout-nav-btn" 
            @click="handleLogout"
            title="登出並鎖定系統"
          >
            🔒 登出
          </button>
        </div>
      </header>

      <!-- 提示橫條 -->
      <div v-if="toastMessage" class="floating-toast no-print">
        {{ toastMessage }}
      </div>

      <!-- 圖片傳送 / 無色透明背景列印彈窗 -->
      <div v-if="shareModalImg" class="image-modal-overlay no-print" @click="closeShareModal">
        <div class="image-modal-content share-preview-modal" @click.stop>
          <div class="image-modal-header">
            <span>💬 {{ shareModalTitle }}</span>
            <button class="close-modal-btn" @click="closeShareModal">✕</button>
          </div>
          <div class="share-modal-body">
            <div class="share-img-scroll-container checkerboard-bg">
              <img :src="shareModalImg" class="share-preview-img-contained" alt="預覽圖" />
            </div>
            
            <div class="share-btn-action-group">
              <button type="button" class="mobile-print-btn" @click="triggerTransparentPrint">
                🖨️ 手機/電腦直接列印 (無白底・適用粉紅/紅底紙・無網址)
              </button>
              <button v-if="canNativeShare" type="button" class="mobile-share-btn" @click="triggerNativeShare">
                📲 一鍵傳送至 LINE 給客人
              </button>
              <a :href="shareModalImg" :download="shareModalFilename" class="mobile-dl-btn">
                💾 儲存透明圖檔 (PNG)
              </a>
            </div>

            <div class="share-tips-row">
              <span>💡 <b>列印與傳送提示：</b></span>
              <span>• <b>已設定無色透明</b>：圖檔不帶白色底，印在紅色或粉紅紙上時只印出墨水文字與印章。</span>
              <span>• <b>消除底部網址</b>：點擊上方「直接列印」，手機將以單張照片輸出，底端不出現網址與時間。</span>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 1：蘭花管理系統 ================= -->
      <div v-if="currentTab === 'manage'" class="manage-container no-print">
        <nav class="sub-nav">
          <button :class="{ active: subTab === 'order' }" @click="subTab = 'order'">💰 1. 訂單與帳務</button>
          <button :class="{ active: subTab === 'statement' }" @click="subTab = 'statement'">📊 2. 客戶未結對帳專區</button>
          <button :class="{ active: subTab === 'inventory' }" @click="subTab = 'inventory'">📦 3. 進貨與庫存</button>
          <button :class="{ active: subTab === 'customer' }" @click="subTab = 'customer'">👥 4. 客戶資料庫</button>
          <button :class="{ active: subTab === 'orchid' }" @click="subTab = 'orchid'">🌸 5. 蘭花品種庫</button>
          <button :class="{ active: subTab === 'return' }" @click="subTab = 'return'">🔄 6. 退貨管理</button>
          <button :class="{ active: subTab === 'shipping' }" @click="subTab = 'shipping'" class="nav-shipping-highlight">🚚 7. 出貨派送進度 ({{ unshippedOrders.length }})</button>
        </nav>

        <div class="manage-content">
          <!-- 模組 1：訂單與帳務 -->
          <section v-if="subTab === 'order'" class="tab-pane">
            <div v-if="editingOrderId" class="edit-banner">
              <span>✏️ 目前正在編輯訂單：<b>{{ editingOrderId }}</b></span>
              <button class="cancel-edit-btn" @click="cancelEditOrder">✕ 取消修改</button>
            </div>

            <div class="card-box" id="order-form-box">
              <div class="order-form-title-row">
                <h3>{{ editingOrderId ? '✏️ 修改訂單資料' : '💰 建立新訂單' }}</h3>
                <span class="preview-seq-badge right-aligned-badge">
                  預計產生單號：<b>{{ editingOrderId || previewNextOrderId }}</b>
                </span>
              </div>
              
              <div class="form-grid">
                <div class="field highlight-date-field">
                  <label>📅 下單日期：</label>
                  <input v-model="formOrder.order_date" type="date" class="bold-date-input" />
                </div>
                <div class="field">
                  <label>預計出貨 / 送達日：</label>
                  <input v-model="formOrder.expected_date" type="date" />
                </div>
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

                <!-- 🚚 送貨地址 -->
                <div class="field highlight-field triple-width-field">
                  <label>🚚 送貨地址：</label>
                  <input 
                    v-model="formOrder.shipping_address" 
                    type="text" 
                    placeholder="例: 桃園市中壢區培英路...號 或 門市自取" 
                  />
                </div>
              </div>

              <!-- 多組花禮規格 -->
              <div class="items-section mt-3">
                <div class="items-header">
                  <h4>🌸 花禮品項與盆數規格</h4>
                  <button type="button" class="add-item-btn" @click="addOrderItemRow">＋ 新增一組花禮規格</button>
                </div>

                <div 
                  v-for="(item, idx) in formOrder.items" 
                  :key="idx" 
                  class="order-item-card"
                >
                  <div class="item-card-title">
                    <span>品項 {{ idx + 1 }}</span>
                    <button 
                      v-if="formOrder.items.length > 1" 
                      type="button" 
                      class="remove-item-btn" 
                      @click="removeOrderItemRow(idx)"
                    >
                      🗑
                    </button>
                  </div>
                  <div class="item-grid">
                    <div class="field">
                      <label>品種名稱</label>
                      <input v-model="item.orchid_name" placeholder="手動輸入或選取庫存" />
                    </div>
                    <div class="field highlight-field">
                      <label>盆數 (幾盆)</label>
                      <input v-model.number="item.pots_qty" type="number" min="1" @input="calcOrderPrice" />
                    </div>
                    <div class="field">
                      <label>單盆株數 (棵/盆)</label>
                      <input v-model.number="item.stalks" type="number" min="1" @input="calcOrderPrice" />
                    </div>
                    <div class="field">
                      <label>每棵單價 (元)</label>
                      <input v-model.number="item.unit_price" type="number" min="0" @input="calcOrderPrice" />
                    </div>
                    <div class="field">
                      <label>使用盆器</label>
                      <select v-model="item.pot" @change="calcOrderPrice">
                        <option value="桌上盆 (100)">桌上盆 (成本100)</option>
                        <option value="落地盆陶瓷-喪 (100)">落地盆陶瓷-喪 (成本100)</option>
                        <option value="落地陶瓷盆-喜 (200)">落地陶瓷盆-喜 (成本200)</option>
                        <option value="羅馬盆 (280)">羅馬盆 (成本280)</option>
                        <option value="無盆">無盆</option>
                      </select>
                    </div>
                    <div class="field">
                      <label>快捷盆選擇</label>
                      <select v-model="item.quick_pot" @change="calcOrderPrice">
                        <option value="未使用">未使用快捷盆</option>
                        <option value="使用快捷盆 (70)">使用快捷盆 (成本70)</option>
                      </select>
                    </div>
                  </div>
                </div>
              </div>

              <!-- 費用與總計資訊 -->
              <div class="form-grid mt-3">
                <div class="field highlight-field">
                  <label>額外運費 (元)</label>
                  <input v-model.number="formOrder.shipping_fee" type="number" min="0" @input="calcOrderPrice" placeholder="無運費填 0" />
                </div>
                <div class="field">
                  <label>預估總成本 (元)</label>
                  <input v-model.number="formOrder.cost" type="number" min="0" />
                </div>
                <div class="field">
                  <label>訂單總售價 (含運費)</label>
                  <input v-model.number="formOrder.price" type="number" min="0" class="bold-price-input" />
                </div>
                <div class="field">
                  <label>訂單其他備註</label>
                  <input v-model="formOrder.note" type="text" placeholder="送貨注意事項" />
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

            <!-- 訂單總覽清單 -->
            <div class="card-box mt-3">
              <div class="table-header-action">
                <h3>📋 訂單總覽 ({{ orderList.length }} 筆)</h3>
                <button class="excel-btn" @click="exportOrdersToExcel">📊 下載全訂單 Excel 報表</button>
              </div>

              <div class="table-responsive">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>單號</th><th>下單日</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>總盆數</th><th>規格明細</th><th>運費</th><th>總售價</th><th>花卡</th><th>簽收單</th><th>出貨</th><th>收款</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="ord in orderList" :key="ord.id">
                      <td><b>{{ ord.id }}</b></td>
                      <td>{{ ord.order_date }}</td>
                      <td><b>{{ ord.customer }}</b></td>
                      <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                      <td>{{ ord.need_receipt || '不需收據' }}</td>
                      <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                      <td class="spec-cell-wrap">{{ ord.spec }}</td>
                      <td>{{ getOrderShippingFee(ord) > 0 ? '$' + getOrderShippingFee(ord) : '免運' }}</td>
                      <td class="text-blue font-heavy">${{ ord.price }}</td>
                      <td><span class="badge">{{ ord.card_status || '未製作' }}</span></td>
                      <td><span class="badge">{{ ord.receipt_status || '未列印' }}</span></td>
                      <td><span class="badge">{{ ord.shipped_status || '未出貨' }}</span></td>
                      <td><span class="badge">{{ ord.payment_status || '未結' }}</span></td>
                      <td class="action-cell">
                        <div class="stacked-action-container">
                          <button class="cozy-btn clean-btn-noborder" style="background-color: #F3EEC3 !important; color: #5a5410 !important;" @click="fillReceiptFromOrder(ord)">🖨️ 簽收單</button>
                          <button class="cozy-btn clean-btn-noborder" style="background-color: #DEE2FF !important; color: #28305c !important;" @click="fillFarmerReceiptFromOrder(ord)">🧾 農民收據</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditOrder(ord)">✏️</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>
        </div>
      </div>

      <!-- ================= 模式 2：花卡 / 輓聯編輯器 ================= -->
      <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
        <div class="control-panel no-print">
          <h2>⚙️ 卡片與題詞設定</h2>

          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: cardPaperSize === 'A4' }" @click="cardPaperSize = 'A4'">A4 (大尺寸)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A5' }" @click="cardPaperSize = 'A5'">A5 (小尺寸花卡)</button>
            </div>
          </div>

          <div class="panel-section">
            <div class="inline-font-weight-row">
              <div class="inline-item-flex">
                <label class="mini-field-lbl">字體選擇：</label>
                <select v-model="cardFontFamily" class="full-input compact-inline-select font-bold">
                  <option value="kai">標準標楷體 / 書法楷書體 (手機/電腦通用)</option>
                  <option value="song">古典宋體 / 明體風格</option>
                  <option value="fangsong">仿宋古典體 (FangSong)</option>
                  <option value="sans">現代簡約黑體</option>
                </select>
              </div>
              <div class="inline-item-fixed">
                <label class="mini-field-lbl">中款預設粗細：</label>
                <select v-model="weights.middle" class="full-input compact-inline-select font-bold text-blue">
                  <option value="400">400 (正常)</option><option value="500">500 (微厚)</option><option value="550">550 (中厚)</option>
                  <option value="600">600 (半粗)</option><option value="650">650 (厚粗)</option><option value="700">700 (粗體)</option><option value="800">800 (特粗)</option>
                </select>
              </div>
            </div>
          </div>

          <div class="form-group">
            <label>版面模式：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: isVertical }" @click="switchOrientation(true)">直式 (傳統輓聯)</button>
              <button type="button" :class="{ active: !isVertical }" @click="switchOrientation(false)">橫式 (現代花卡)</button>
            </div>
          </div>

          <div class="form-group">
            <label>卡片類型：</label>
            <select v-model="cardCategory" @change="onCardCategoryChange">
              <option value="funeral">喪禮弔唁</option>
              <option value="celebration">慶賀祝典</option>
            </select>
          </div>

          <!-- 上款 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">1. 開頭敬詞：</span>
              <input type="number" v-model.number="layout.upper_prefix.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperPrefix" class="full-input mt-1" placeholder="例: 敬悼 或 恭祝" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">2. 受禮對象：</span>
              <input type="number" v-model.number="layout.upper_target.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperTarget" class="full-input mt-1" placeholder="受禮人姓名稱謂" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">3. 上款結尾詞：</span>
              <input type="number" v-model.number="layout.upper_suffix.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperSuffix" class="full-input mt-1" placeholder="留空則不顯示" />
          </div>

          <!-- 中款 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">中款第 1 行：</span>
              <input type="number" v-model.number="layout.middle.size" min="14" max="300" class="compact-size-input" />
            </div>
            <input type="text" v-model="middleText" class="full-input mt-1" placeholder="中款題詞" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">中款第 2 行：</span>
              <input type="number" v-model.number="layout.middle_2.size" min="14" max="300" class="compact-size-input" />
            </div>
            <input type="text" v-model="middleText2" class="full-input mt-1" placeholder="留空則不顯示第 2 行" />
          </div>

          <!-- 下款 -->
          <div class="panel-section">
            <label class="section-title">下款設定：</label>
            <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
              <span class="line-num">格 {{ idx + 1 }}</span>
              <input type="text" v-model="item.text" class="flex-input" />
              <input type="number" v-model.number="layout['bottom_' + idx].size" min="14" max="150" class="compact-size-input" />
            </div>

            <div class="section-title-with-weight mt-2">
              <span class="section-title">結尾敬詞：</span>
              <input type="number" v-model.number="layout.suffix.size" min="14" max="200" class="compact-size-input" />
            </div>
            <select v-model="suffixText" class="full-input mt-1">
              <option value="敬輓">敬輓</option><option value="泣輓">泣輓</option><option value="拜輓">拜輓</option>
              <option value="敬賀">敬賀</option><option value="恭賀">恭賀</option><option value="拜賀">拜賀</option><option value="謹致">謹致</option>
            </select>
          </div>

          <button type="button" class="reset-btn" @click="resetPositions">↺ 重設排版預設位置</button>
          <button type="button" class="line-action-btn mt-2" @click="shareCoupletDirect">💬 直接傳送 / 複製花卡給客人</button>
          <button type="button" class="print-action-btn mt-2" @click="openTransparentPrintModal('card-print-target', `花卡_${cardPaperSize}`)">
            🖨️ 列印花卡 (無色透明底・無網址時間)
          </button>
        </div>

        <div class="canvas-viewport" ref="viewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.3, +(zoomLevel - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
          </div>

          <div 
            class="card-scaler-container" 
            :style="{
              width: (currentCardDimensions.w * zoomLevel) + 'px',
              height: (currentCardDimensions.h * zoomLevel) + 'px'
            }"
          >
            <!-- 🌟 花卡主體 (透明底色設定) -->
            <div 
              id="card-print-target" 
              class="card-board target-kai-font transparent-target" 
              :class="isVertical ? 'mode-vertical' : 'mode-horizontal'"
              :style="{
                width: currentCardDimensions.w + 'px',
                height: currentCardDimensions.h + 'px',
                transform: `scale(${zoomLevel})`,
                transformOrigin: 'top left',
                fontFamily: activeCssFontFamily
              }"
            >
              <div v-if="upperPrefix.trim()" class="text-box upper-prefix-box" :style="getStyle('upper_prefix')">
                <span>{{ upperPrefix }}</span>
              </div>
              <div v-if="upperTarget.trim()" class="text-box upper-target-box" :style="getUpperTargetBoxStyle()">
                <span>{{ upperTarget }}</span>
              </div>
              <div v-if="upperSuffix.trim()" class="text-box upper-suffix-box" :style="getStyle('upper_suffix')">
                <span>{{ upperSuffix }}</span>
              </div>
              <div v-if="middleText.trim()" class="text-box middle-box" :style="getStyle('middle')">
                <span>{{ middleText }}</span>
              </div>
              <div v-if="middleText2.trim()" class="text-box middle-box-2" :style="getStyle('middle_2')">
                <span>{{ middleText2 }}</span>
              </div>
              <template v-for="(item, idx) in bottomLines" :key="'bottom-' + idx">
                <div v-if="item.text.trim()" class="text-box" :style="getStyle('bottom_' + idx)">
                  <span>{{ item.text }}</span>
                </div>
              </template>
              <div v-if="suffixText.trim()" class="text-box suffix-box" :style="getStyle('suffix')">
                <span>{{ suffixText }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 3：A5 橫式簽收單 ================= -->
      <div v-else-if="currentTab === 'receipt'" class="receipt-container">
        <div class="control-panel no-print">
          <h2>📋 橫式 A5 簽收單管理</h2>

          <div class="panel-section highlight-panel">
            <label class="section-title">依訂單快速帶入：</label>
            <select v-model="selectedOrderId" @change="onSelectReceiptOrder" class="full-input bold-select">
              <option value="">-- 請下拉選擇訂單 --</option>
              <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
                【{{ ord.id }}】{{ ord.customer }} - {{ formatSimpleItemName(ord) }}
              </option>
            </select>
          </div>

          <div class="panel-section">
            <label class="section-title">✏ 簽收單內容確認：</label>
            <div class="form-group">
              <label>收件單位/人：</label>
              <input type="text" v-model="receiptForm.recipient" />
            </div>
            <div class="form-group">
              <label>送達地址：</label>
              <input type="text" v-model="receiptForm.address" />
            </div>
            <div class="form-group">
              <label>送達日期：</label>
              <input type="text" v-model="receiptForm.deliveryDate" />
            </div>
            <div class="form-group">
              <label>花禮品項規格：</label>
              <input type="text" v-model="receiptForm.item" />
            </div>
            <div class="form-group">
              <label>致贈單位/賀詞：</label>
              <input type="text" v-model="receiptForm.giver" />
            </div>
            <div class="form-group">
              <label>備註說明：</label>
              <textarea v-model="receiptForm.notes" rows="2"></textarea>
            </div>
          </div>

          <button type="button" class="line-action-btn mt-2" @click="shareReceiptDirect">
            📤 簽好直接傳送 (LINE/下載)
          </button>
          <button type="button" class="print-action-btn mt-2" @click="openTransparentPrintModal('receipt-print-target', '簽收單_A5')">
            🖨️ 列印 A5 簽收單 (無色透明底・無網址時間)
          </button>
        </div>

        <div class="receipt-preview-area">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="receiptZoom = Math.max(0.3, +(receiptZoom - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(receiptZoom * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="receiptZoom = Math.min(1.1, +(receiptZoom + 0.05).toFixed(2))">＋</button>
          </div>

          <div 
            class="receipt-scaler-container" 
            :style="{
              width: (794 * receiptZoom) + 'px',
              height: (560 * receiptZoom) + 'px'
            }"
          >
            <div 
              id="receipt-print-target"
              class="a5-landscape-sheet target-kai-font transparent-target"
              :style="{
                transform: `scale(${receiptZoom})`,
                transformOrigin: 'top left'
              }"
            >
              <div class="sheet-header">
                <div class="shop-name-title">宸豐蘭藝</div>
                <div class="sheet-main-title">銷貨 / 出貨簽收單</div>
                <div class="header-meta">
                  <div><b>訂單編號：</b>{{ receiptForm.orderId || '現場直接開單' }}</div>
                  <div><b>送達日期：</b>{{ receiptForm.deliveryDate || '依約定送達' }}</div>
                </div>
              </div>

              <table class="receipt-table">
                <tbody>
                  <tr>
                    <td class="lbl">收件單位/人</td>
                    <td class="val val-bold">{{ receiptForm.recipient || '—' }}</td>
                    <td class="lbl">送達地址</td>
                    <td class="val val-bold text-blue">{{ receiptForm.address || '同訂購人地址 / 門市取貨' }}</td>
                  </tr>
                  <tr>
                    <td class="lbl">花禮品項</td>
                    <td class="val val-highlight" colspan="3">
                      {{ receiptForm.item || '特選蘭花 1盆' }}
                    </td>
                  </tr>
                  <tr>
                    <td class="lbl">致贈/賀詞</td>
                    <td class="val" colspan="3">{{ receiptForm.giver || '敬領 誌慶 / 宸豐蘭藝 敬製' }}</td>
                  </tr>
                  <tr>
                    <td class="lbl">備註說明</td>
                    <td class="val" colspan="3">{{ receiptForm.notes || '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！' }}</td>
                  </tr>
                </tbody>
              </table>

              <div class="sheet-footer">
                <div class="footer-left">
                  <div>送貨司機 / 經手人：______________</div>
                  <div class="footer-tip">※ 專車親送・現場點交確認・花禮已送達</div>
                </div>
                <div class="footer-sign-box">
                  <div class="sign-box-title">客戶簽收章 / 線上簽名欄</div>
                  <div class="sign-box-area"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 4：農民收據 ================= -->
      <div v-else-if="currentTab === 'farmer_receipt'" class="receipt-container">
        <div class="control-panel no-print">
          <h2>🧾 農民出售農產品收據管理</h2>

          <div class="panel-section highlight-panel">
            <label class="section-title">依訂單自動帶入：</label>
            <select v-model="selectedFarmerOrderId" @change="onSelectFarmerReceiptOrder" class="full-input bold-select">
              <option value="">-- 請下拉選擇訂單 --</option>
              <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
                【{{ ord.id }}】{{ ord.customer }} - ${{ ord.price }}
              </option>
            </select>
          </div>

          <div class="panel-section">
            <label class="section-title">✏ 收據內容確認：</label>
            <div class="form-row">
              <input type="text" v-model="farmerReceipt.year" placeholder="年" />
              <input type="text" v-model="farmerReceipt.month" placeholder="月" />
              <input type="text" v-model="farmerReceipt.day" placeholder="日" />
            </div>
            <div class="form-group mt-2">
              <label>購貨商號名稱：</label>
              <input type="text" v-model="farmerReceipt.buyerName" />
            </div>
            <div class="form-group">
              <label>統一編號：</label>
              <input type="text" v-model="farmerReceipt.taxId" />
            </div>
            <div class="form-group">
              <label>住址：</label>
              <input type="text" v-model="farmerReceipt.buyerAddress" />
            </div>
            <div class="form-group">
              <label>總金額：</label>
              <input type="number" v-model.number="farmerReceipt.totalAmount" @input="updateChineseAmount" />
            </div>
          </div>

          <button type="button" class="line-action-btn mt-2" @click="shareFarmerReceiptDirect">
            💬 直接傳送收據給客人
          </button>
          <button type="button" class="print-action-btn mt-2" @click="openTransparentPrintModal('farmer-print-target', '農民收據_A5')">
            🖨️ 列印農民收據 (無色透明底・無網址時間)
          </button>
        </div>

        <div class="receipt-preview-area">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="farmerZoom = Math.max(0.3, +(farmerZoom - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(farmerZoom * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="farmerZoom = Math.min(1.1, +(farmerZoom + 0.05).toFixed(2))">＋</button>
          </div>

          <div 
            class="farmer-scaler-container" 
            :style="{
              width: (794 * farmerZoom) + 'px',
              height: (560 * farmerZoom) + 'px'
            }"
          >
            <div 
              id="farmer-print-target"
              class="farmer-receipt-sheet target-kai-font transparent-target"
              :style="{
                transform: `scale(${farmerZoom})`,
                transformOrigin: 'top left'
              }"
            >
              <div class="f-header">
                <div class="f-title-wrap">
                  <div class="f-main-title">農（漁、牧）民出售農（漁、牧）產品收據</div>
                </div>
                <div class="f-date-wrap">
                  中華民國 {{ farmerReceipt.year }} 年 {{ farmerReceipt.month }} 月 {{ farmerReceipt.day }} 日
                </div>
              </div>

              <!-- 手刻格線 -->
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
                  <div class="f-grid-val col-p-amt f-text-right f-bold f-pr">
                    {{ (farmerReceipt.totalAmount && Number(farmerReceipt.totalAmount) > 0) ? ('$' + Number(farmerReceipt.totalAmount).toLocaleString()) : '' }}
                  </div>
                  <div class="f-grid-val col-p-note f-text-center">{{ farmerReceipt.note }}</div>
                </div>

                <div class="f-grid-row f-data-row f-empty-row">
                  <div class="f-grid-val col-p-name"></div>
                  <div class="f-grid-val col-p-spec"></div>
                  <div class="f-grid-val col-p-qty"></div>
                  <div class="f-grid-val col-p-price"></div>
                  <div class="f-grid-val col-p-amt"></div>
                  <div class="f-grid-val col-p-note"></div>
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

                <div class="f-grid-row f-farmer-info-row">
                  <div class="f-grid-lbl f-w-head">農（漁、牧）民姓名</div>
                  <div class="f-grid-val f-farmer-stamp-cell f-flex-1">
                    <span class="f-farmer-name-clean">蔡鎮遠</span>
                    <img :src="activeCaiSealSrc" class="cai-real-stamp-img" alt="印章" />
                  </div>
                </div>

                <div class="f-grid-row f-id-addr-row">
                  <div class="f-grid-lbl f-w-head">住 址</div>
                  <div class="f-grid-val f-flex-1"></div>
                  <div class="f-grid-lbl f-w-id-lbl">國民統一身分證編號</div>
                  <div class="f-grid-val f-w-id-val f-bold f-text-center">F129940801</div>
                </div>
              </div>

              <div class="f-statement">
                本收據之農民身分確實無誤，若有不實者願依法受罰。
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { createClient } from '@supabase/supabase-js'
import * as XLSX from 'xlsx'
import html2canvas from 'html2canvas'

// ==========================================
// 1. 基礎狀態變數
// ==========================================
const currentTab = ref('manage')
const subTab = ref('order')
const cardPaperSize = ref('A4')
const isVertical = ref(false)
const zoomLevel = ref(0.65)
const receiptZoom = ref(0.85)
const farmerZoom = ref(0.85)
const viewportRef = ref(null)

const toastMessage = ref('')
const showToast = (msg) => {
  toastMessage.value = msg
  setTimeout(() => { toastMessage.value = '' }, 3500)
}

const shareModalImg = ref('')
const shareModalTitle = ref('')
const shareModalFilename = ref('圖片.png')
const currentBlobToShare = ref(null)
const canNativeShare = ref(false)

const closeShareModal = () => {
  if (shareModalImg.value && shareModalImg.value.startsWith('blob:')) {
    URL.revokeObjectURL(shareModalImg.value)
  }
  shareModalImg.value = ''
  currentBlobToShare.value = null
}

const triggerNativeShare = async () => {
  if (!currentBlobToShare.value) return
  try {
    const file = new File([currentBlobToShare.value], shareModalFilename.value, { type: 'image/png' })
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
      await navigator.share({ title: shareModalTitle.value, files: [file] })
    }
  } catch (err) {}
}

const currentCardDimensions = computed(() => {
  if (cardPaperSize.value === 'A5') {
    return isVertical.value ? { w: 560, h: 794 } : { w: 794, h: 560 }
  }
  return isVertical.value ? { w: 794, h: 1123 } : { w: 1123, h: 794 }
})

// ==========================================
// 2. 內部驗證與資料庫
// ==========================================
const INTERNAL_PASSCODE = 'cf000725'
const isAuthenticated = ref(localStorage.getItem('cf_admin_auth') === 'true')
const inputPasscode = ref('')
const authError = ref(false)

const handleLogin = () => {
  const entered = (inputPasscode.value || '').trim().toLowerCase()
  if (entered === INTERNAL_PASSCODE) {
    isAuthenticated.value = true
    authError.value = false
    localStorage.setItem('cf_admin_auth', 'true')
    showToast('✅ 驗證成功！')
    nextTick(() => initSystemData())
  } else {
    authError.value = true
  }
}

const handleLogout = () => {
  if (!confirm('確定要登出嗎？')) return
  isAuthenticated.value = false
  localStorage.removeItem('cf_admin_auth')
}

const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
const supabase = createClient(supabaseUrl, supabaseKey)

const userCustomSeal = ref(localStorage.getItem('user_cai_seal_img') || '')
const activeCaiSealSrc = computed(() => userCustomSeal.value || '/cai-seal.png')

const customers = ref([])
const orderList = ref([])
const editingOrderId = ref(null)

const formOrder = ref({
  cust_type: '批發', customer: '', billing_cycle: '每單結', phone: '',
  shipping_address: '', shipping_fee: 0, cost: 600, price: 2500, tax_id: '',
  need_receipt: '不需收據', note: '', card_status: '未製作', receipt_status: '未列印',
  shipped_status: '未出貨', payment_status: '未結',
  order_date: new Date().toISOString().split('T')[0],
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0],
  items: [{ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' }]
})

// ==========================================
// 3. 花卡、簽收單與字型映射
// ==========================================
const cardCategory = ref('celebration')
const cardFontFamily = ref('kai')
const upperPrefix = ref('祝')
const upperTarget = ref('新北市 陳乃瑜議員')
const upperSuffix = ref('')
const middleText = ref('高票當選')
const middleText2 = ref('為民服務')
const suffixText = ref('敬賀')

const weights = ref({
  upper_prefix: '700', upper_target: '700', upper_suffix: '700', middle: '800', middle_2: '800',
  bottom_0: '600', bottom_1: '700', suffix: '700'
})
const bottomLines = ref([{ text: '白沙屯媽祖' }, { text: '彰化拱聖宮' }])

// 🌟 書法正楷映射：優先採用 index.html 載入的 Noto Serif TC，全平台生效
const fontMapping = {
  kai: '"Noto Serif TC", "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", "Kaiti TC", serif',
  song: '"Noto Serif TC", "Songti TC", "SimSun", "PMingLiU", serif',
  fangsong: '"FangSong", "STFangsong", "華康仿宋體", serif',
  sans: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif'
}
const activeCssFontFamily = computed(() => fontMapping[cardFontFamily.value] || fontMapping.kai)

const defaultHorizontal = {
  upper_prefix: { x: 80, y: 80, size: 34 }, upper_target: { x: 220, y: 80, size: 38 }, upper_suffix: { x: 920, y: 80, size: 34 },
  middle: { x: 220, y: 220, size: 68 }, middle_2: { x: 220, y: 310, size: 68 },
  bottom_0: { x: 180, y: 520, size: 28 }, bottom_1: { x: 380, y: 530, size: 34 },
  suffix: { x: 880, y: 530, size: 34 }
}
const defaultVertical = {
  upper_prefix: { x: 620, y: 100, size: 36 }, upper_target: { x: 620, y: 220, size: 42 }, upper_suffix: { x: 620, y: 720, size: 36 },
  middle: { x: 380, y: 220, size: 76 }, middle_2: { x: 280, y: 220, size: 76 },
  bottom_0: { x: 155, y: 480, size: 30 }, bottom_1: { x: 155, y: 640, size: 36 },
  suffix: { x: 155, y: 860, size: 34 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultHorizontal)))
const switchOrientation = (val) => {
  isVertical.value = val
  layout.value = JSON.parse(JSON.stringify(val ? defaultVertical : defaultHorizontal))
}
const resetPositions = () => {
  layout.value = JSON.parse(JSON.stringify(isVertical.value ? defaultVertical : defaultHorizontal))
}

const getStyle = (key) => {
  const item = layout.value?.[key] || { x: 50, y: 50, size: 30 }
  return { 
    left: `${item.x ?? 50}px`, 
    top: `${item.y ?? 50}px`, 
    fontSize: `${item.size ?? 30}px`, 
    fontWeight: weights.value?.[key] || '700'
  }
}
const getUpperTargetBoxStyle = () => {
  const item = layout.value?.upper_target || { x: 220, y: 80, size: 38 }
  return { 
    left: `${item.x ?? 220}px`, 
    top: `${item.y ?? 80}px`, 
    fontSize: `${item.size ?? 38}px`, 
    whiteSpace: 'nowrap', 
    fontWeight: weights.value?.upper_target || '700'
  }
}

// =========================================================================
// 🌟 100% 無色透明背景 (Transparent PNG) 產出引擎
// =========================================================================
const openTransparentPrintModal = async (targetId, titlePrefix) => {
  const targetEl = document.getElementById(targetId)
  if (!targetEl) return alert('找不到目標畫面！')
  showToast('⏳ 正在為您產出「無色透明背景」高畫質圖檔...')
  try {
    if (document.fonts?.ready) await document.fonts.ready
    const origTransform = targetEl.style.transform
    targetEl.style.transform = 'none'

    const canvas = await html2canvas(targetEl, {
      scale: 2,
      useCORS: true,
      backgroundColor: null, 
      logging: false,
      ignoreElements: (el) => el.classList && (el.classList.contains('scale-handle') || el.classList.contains('no-print'))
    })
    targetEl.style.transform = origTransform

    canvas.toBlob((blob) => {
      if (!blob) return
      if (shareModalImg.value && shareModalImg.value.startsWith('blob:')) {
        URL.revokeObjectURL(shareModalImg.value)
      }
      shareModalImg.value = URL.createObjectURL(blob)
      shareModalTitle.value = `${titlePrefix} (無色透明底)`
      shareModalFilename.value = `${titlePrefix}_透明底.png`
      currentBlobToShare.value = blob
      canNativeShare.value = !!(navigator.canShare && navigator.canShare({ files: [new File([blob], 'print.png', { type: 'image/png' })] }))
    }, 'image/png')
  } catch (err) {
    showToast('⚠️ 生成失敗，請重試！')
  }
}

const triggerTransparentPrint = () => {
  if (!shareModalImg.value) return
  const imgUrl = shareModalImg.value
  const printWin = window.open('', '_blank')
  if (printWin) {
    printWin.document.write(`
      <html>
        <head>
          <title>列印</title>
          <style>
            @page { size: auto; margin: 0mm !important; }
            * { margin: 0; padding: 0; box-sizing: border-box; }
            body { 
              display: flex; justify-content: center; align-items: center; 
              min-height: 100vh; background: transparent !important; 
            }
            img { max-width: 100%; max-height: 100vh; object-fit: contain; }
          </style>
        </head>
        <body onload="window.print(); window.close();">
          <img src="${imgUrl}" />
        </body>
      </html>
    `)
    printWin.document.close()
  } else {
    window.location.href = imgUrl
  }
}

const shareCoupletDirect = () => openTransparentPrintModal('card-print-target', '花卡傳送')
const shareReceiptDirect = () => openTransparentPrintModal('receipt-print-target', '簽收單傳送')
const shareFarmerReceiptDirect = () => openTransparentPrintModal('farmer-print-target', '農民收據傳送')

// 簽收單與收據表單
const selectedOrderId = ref('')
const receiptForm = ref({
  orderId: '', deliveryDate: '115-09-03 送達', recipient: '永全證券 陳總經理',
  address: '桃園市桃園區縣府路 82 號', item: '特選蘭花 1盆', giver: '敬領 誌慶', notes: '花禮已專車安全送達點交'
})

const selectedFarmerOrderId = ref('')
const farmerReceipt = ref({
  year: '115', month: '09', day: '03', buyerName: '永全證券股份有限公司', taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號', itemName: '蝴蝶蘭花禮', spec: '特級', qty: '1 盆', unitPrice: '2500', totalAmount: 2500, note: ''
})
const chineseDigits = ref({ hundredThousands: '', tenThousands: '', thousands: '貳', hundreds: '伍', tens: '', ones: '' })

const initSystemData = () => {
  zoomLevel.value = 0.65
  receiptZoom.value = 0.85
  farmerZoom.value = 0.85
}

onMounted(() => {
  initSystemData()
})
</script>

<style scoped>
/* 🌟 管理介面保持清爽標準字體 */
.main-wrapper {
  display: flex; flex-direction: column; height: 100vh; font-size: 13.5px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9;
}
.system-root { display: flex; flex-direction: column; flex: 1; overflow: hidden; }

/* 頂端列 */
.top-nav {
  height: 50px; background-color: #0f172a; color: white; display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px; flex-shrink: 0;
}
.nav-title { font-size: 16px; font-weight: 900; }
.nav-tabs { display: flex; gap: 8px; }
.nav-tabs button {
  background: #334155; color: #e2e8f0; border: none; padding: 7px 12px; border-radius: 6px; cursor: pointer; font-weight: bold;
}
.nav-tabs button.active { background: #2563eb; color: white; }

.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.couplet-screen-wrapper { display: flex; flex: 1; overflow: hidden; height: calc(100vh - 50px); }
.control-panel {
  width: 390px; background: white; padding: 14px; box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 9px; border-radius: 6px; margin-bottom: 9px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }

.section-title { font-size: 13px; font-weight: bold; margin-bottom: 4px; display: inline-block; }
.section-title-with-weight { display: flex; justify-content: space-between; align-items: center; }
.form-group { margin-bottom: 8px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 3px; }
input, select, textarea { width: 100%; padding: 6px 8px; border: 1px solid #cbd5e1; border-radius: 5px; box-sizing: border-box; }
.compact-size-input { width: 50px; padding: 2px 4px; text-align: center; font-weight: bold; }
.btn-group { display: flex; gap: 5px; }
.btn-group button { flex: 1; padding: 6px; border: 1px solid #2563eb; background: white; color: #2563eb; border-radius: 4px; font-weight: bold; }
.btn-group button.active { background: #2563eb; color: white; }

.print-action-btn {
  width: 100%; padding: 11px; background: #16a34a; color: white; border: none; border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer;
}
.line-action-btn {
  width: 100%; padding: 10px; background: #06c755; color: white; border: none; border-radius: 6px; font-weight: bold; cursor: pointer;
}

/* 預覽視窗 */
.canvas-viewport, .receipt-preview-area {
  flex: 1; display: flex; flex-direction: column; align-items: center; overflow: auto; padding: 18px; background-color: #cbd5e1;
}
.zoom-toolbar {
  display: flex; align-items: center; gap: 5px; background: white; padding: 4px 10px; border-radius: 20px; margin-bottom: 10px;
}
.zoom-btn { width: 24px; height: 24px; border: 1px solid #cbd5e1; background: #f8fafc; border-radius: 50%; cursor: pointer; }
.zoom-text { font-size: 12.5px; font-weight: bold; min-width: 40px; text-align: center; }

.card-scaler-container, .receipt-scaler-container, .farmer-scaler-container { position: relative; flex-shrink: 0; }
.card-board {
  position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none;
}
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.text-box { position: absolute; cursor: move; white-space: nowrap; color: #0f172a; padding: 2px 4px; }

/* 🌟 單據與花卡專屬字體 */
.target-kai-font {
  font-family: "Noto Serif TC", "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", "Kaiti TC", serif !important;
}

/* 🌟 無色透明背景 */
.transparent-target {
  background-color: transparent !important;
}
.card-scaler-container, .receipt-scaler-container, .farmer-scaler-container {
  background-color: rgba(255, 255, 255, 0.95);
  border-radius: 4px;
}

/* 簽收單 */
.a5-landscape-sheet {
  width: 794px; height: 560px; padding: 32px 38px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute;
}
.sheet-header { display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 2.5px solid #1e293b; padding-bottom: 6px; }
.shop-name-title { font-size: 27px; font-weight: 900; }
.sheet-main-title { font-size: 23px; font-weight: bold; color: #dc2626; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 8px 0; }
.receipt-table td { border: 1.5px solid #334155; padding: 7px 10px; }
.receipt-table .lbl { width: 15%; background: rgba(241, 245, 249, 0.5); font-weight: bold; text-align: center; }
.sheet-footer { display: flex; justify-content: space-between; }
.footer-sign-box { width: 215px; border: 1.5px dashed #475569; border-radius: 6px; display: flex; flex-direction: column; }
.sign-box-title { font-size: 12.5px; font-weight: bold; text-align: center; padding: 2px 0; }
.sign-box-area { flex: 1; min-height: 50px; }

/* 農民收據 */
.farmer-receipt-sheet {
  width: 794px; height: 560px; padding: 16px 26px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute;
}
.f-main-title { font-size: 24px; font-weight: 900; text-align: center; }
.f-date-wrap { align-self: flex-end; font-size: 14px; margin-top: 4px; }
.f-receipt-grid-table { border: 2px solid #000; display: flex; flex-direction: column; }
.f-grid-row { display: flex; border-bottom: 1px solid #000; min-height: 29px; }
.f-grid-lbl { display: flex; justify-content: center; align-items: center; font-weight: bold; border-right: 1px solid #000; padding: 3px; }
.f-grid-val { display: flex; align-items: center; padding-left: 8px; border-right: 1px solid #000; }
.f-flex-1 { flex: 1; }
.f-col-buyer-group { display: flex; flex-direction: column; width: 58%; border-right: 1px solid #000; }
.f-sub-row { display: flex; flex: 1; border-bottom: 1px solid #000; }
.f-sub-row:last-child { border-bottom: none; }
.f-w-head { width: 140px; }
.f-w-addr-tag { width: 36px; }
.f-full-addr-box { flex: 1; border-right: none !important; }
.col-p-name { width: 25%; } .col-p-spec { width: 16%; } .col-p-qty { width: 10%; } .col-p-price { width: 14%; } .col-p-amt { width: 18%; } .col-p-note { width: 17%; border-right: none !important; }
.f-w-total-lbl { width: 200px; }
.f-amount-val-cell { flex: 1; border-right: none !important; }
.f-chinese-amount-line { display: flex; justify-content: space-around; width: 100%; font-weight: bold; }
.f-farmer-stamp-cell { border-right: none !important; display: flex; align-items: center; gap: 14px; padding-left: 20px; }
.cai-real-stamp-img { width: 44px; height: 44px; mix-blend-mode: multiply; }
.f-w-id-lbl { width: 180px; } .f-w-id-val { width: 180px; border-right: none !important; }
.f-statement { font-size: 12px; text-align: center; font-weight: bold; margin-top: 4px; }

/* 彈窗樣式 */
.image-modal-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0,0,0,0.78);
  display: flex; justify-content: center; align-items: center; z-index: 99999; padding: 14px; box-sizing: border-box;
}
.image-modal-content {
  background: white; border-radius: 12px; padding: 16px; max-width: 820px; width: 100%; max-height: 94vh;
  display: flex; flex-direction: column; box-shadow: 0 20px 40px rgba(0,0,0,0.4); box-sizing: border-box; overflow-y: auto;
}
.image-modal-header { display: flex; justify-content: space-between; align-items: center; font-weight: bold; margin-bottom: 10px; font-size: 15px; }
.close-modal-btn { background: transparent; border: none; font-size: 22px; cursor: pointer; color: #64748b; }
.share-modal-body { display: flex; flex-direction: column; align-items: center; width: 100%; }

/* 棋盤格透明提示背景 */
.checkerboard-bg {
  background-image: linear-gradient(45deg, #e2e8f0 25%, transparent 25%), 
                    linear-gradient(-45deg, #e2e8f0 25%, transparent 25%), 
                    linear-gradient(45deg, transparent 75%, #e2e8f0 75%), 
                    linear-gradient(-45deg, transparent 75%, #e2e8f0 75%);
  background-size: 16px 16px;
  background-position: 0 0, 0 8px, 8px -8px, -8px 0px;
}

.share-img-scroll-container {
  width: 100%; display: flex; justify-content: center; align-items: center;
  border-radius: 8px; padding: 10px; box-sizing: border-box; margin-bottom: 12px;
}
.share-preview-img-contained { max-height: 55vh; max-width: 100%; object-fit: contain; }
.share-btn-action-group { display: flex; gap: 8px; width: 100%; margin-bottom: 10px; flex-wrap: wrap; }
.mobile-print-btn {
  flex: 1.5; min-width: 180px; background: #16a34a; color: white; border: none; padding: 10px;
  border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; text-align: center;
}
.mobile-share-btn {
  flex: 1; min-width: 140px; background: #06c755; color: white; border: none; padding: 10px;
  border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; text-align: center;
}
.mobile-dl-btn {
  flex: 1; min-width: 120px; background: #2563eb; color: white; text-decoration: none; padding: 10px;
  border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer; text-align: center; display: inline-block; box-sizing: border-box;
}
.share-tips-row { display: flex; flex-direction: column; gap: 3px; font-size: 12px; color: #475569; line-height: 1.5; width: 100%; background: #f8fafc; padding: 8px 10px; border-radius: 6px; }
</style>