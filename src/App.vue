<template>
  <div class="main-wrapper">
    <!-- ================= 內部安全通行碼驗證畫面 ================= -->
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

      <!-- 浮動提示橫條 -->
      <div v-if="toastMessage" class="floating-toast no-print">
        {{ toastMessage }}
      </div>

      <!-- 圖片傳送專用彈窗 -->
      <div v-if="shareModalImg" class="image-modal-overlay no-print" @click="shareModalImg = ''">
        <div class="image-modal-content share-preview-modal" @click.stop>
          <div class="image-modal-header">
            <span>💬 {{ shareModalTitle }}（可直接按右鍵複製圖片）</span>
            <button class="close-modal-btn" @click="shareModalImg = ''">✕</button>
          </div>
          <div class="share-modal-body">
            <div class="share-img-scroll-container">
              <img :src="shareModalImg" class="share-preview-img-contained" alt="傳送預覽圖" />
            </div>
            <div class="share-tips-row">
              <span>💡 <b>傳送給訂購人方式：</b></span>
              <span>1. 已自動下載圖檔，可直接將圖檔<b>傳送至 LINE</b>。</span>
              <span>2. 電腦版可在圖上點<b>滑鼠右鍵 ➔「複製圖片」</b>，到 LINE 聊天室按 <b>Ctrl + V</b> 發送。</span>
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
          <!-- 訂單與帳務分頁 -->
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
              </div>

              <!-- 多規格品項 -->
              <div class="items-section mt-3">
                <div class="items-header">
                  <h4>🌸 花禮品項與盆數規格</h4>
                  <button type="button" class="add-item-btn" @click="addOrderItemRow">＋ 新增一組花禮規格</button>
                </div>

                <div v-for="(item, idx) in formOrder.items" :key="idx" class="order-item-card">
                  <div class="item-card-title">
                    <span>品項 {{ idx + 1 }}</span>
                    <button v-if="formOrder.items.length > 1" type="button" class="remove-item-btn" @click="removeOrderItemRow(idx)">
                      🗑️
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
                  </div>
                </div>
              </div>

              <div class="form-grid mt-3">
                <div class="field highlight-field">
                  <label>額外運費 (元)</label>
                  <input v-model.number="formOrder.shipping_fee" type="number" min="0" @input="calcOrderPrice" />
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
                <button v-if="editingOrderId" class="secondary-btn" @click="cancelEditOrder">取消</button>
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
                      <th>單號</th><th>下單日</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>總盆數</th><th>規格明細</th><th>運費</th><th>總售價</th>
                      <th>花卡</th><th>簽收單</th><th>出貨</th><th>收款</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="ord in orderList" :key="ord.id">
                      <td><b>{{ ord.id }}</b></td>
                      <td>{{ ord.order_date }}</td>
                      <td><b>{{ ord.customer }}</b></td>
                      <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                      <td>
                        <select v-model="ord.need_receipt" class="uniform-status-select" @change="updateOrderField(ord, 'need_receipt', ord.need_receipt)">
                          <option value="不需收據">不需收據</option><option value="需開收據">需開收據</option>
                        </select>
                      </td>
                      <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                      <td class="spec-cell-wrap">{{ ord.spec }}</td>
                      <td>{{ getOrderShippingFee(ord) > 0 ? '$' + getOrderShippingFee(ord) : '免運' }}</td>
                      <td class="text-blue font-heavy">${{ ord.price }}</td>
                      <td>
                        <select 
                          v-model="ord.card_status" 
                          :style="ord.card_status === '未製作' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}"
                          :class="getCardStatusClass(ord.card_status)"
                          @change="updateOrderField(ord, 'card_status', ord.card_status)"
                          class="uniform-status-select"
                        >
                          <option value="未製作">未製作</option><option value="已製作">已製作</option><option value="免製作">免製作</option>
                        </select>
                      </td>
                      <td>
                        <select 
                          v-model="ord.receipt_status" 
                          :style="ord.receipt_status === '未列印' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}"
                          :class="ord.receipt_status === '已列印' ? 'badge badge-soft-green' : 'badge'"
                          @change="updateOrderField(ord, 'receipt_status', ord.receipt_status)"
                          class="uniform-status-select"
                        >
                          <option value="未列印">未列印</option><option value="已列印">已列印</option>
                        </select>
                      </td>
                      <td>
                        <select v-model="ord.shipped_status" :class="ord.shipped_status === '已出貨' ? 'badge badge-green' : 'badge badge-red'" @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)" class="uniform-status-select">
                          <option value="未出貨">未出貨</option><option value="已出貨">已出貨</option>
                        </select>
                      </td>
                      <td>
                        <select v-model="ord.payment_status" :class="ord.payment_status === '已結' ? 'badge badge-green' : 'badge badge-red'" @change="updateOrderField(ord, 'payment_status', ord.payment_status)" class="uniform-status-select">
                          <option value="未結">未結</option><option value="已結">已結</option>
                        </select>
                      </td>
                      <td class="action-cell">
                        <div class="stacked-action-container">
                          <div class="stacked-action-col">
                            <button class="cozy-btn clean-btn-noborder" style="background-color: #F3EEC3 !important; color: #5a5410 !important;" @click="fillReceiptFromOrder(ord)">🖨️ 簽收單</button>
                            <button class="cozy-btn clean-btn-noborder" style="background-color: #DEE2FF !important; color: #28305c !important;" @click="fillFarmerReceiptFromOrder(ord)">🧾 農民收據</button>
                          </div>
                          <div class="stacked-action-col">
                            <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditOrder(ord)">✏️</button>
                            <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('orders', ord.id, loadOrders)">🗑️</button>
                          </div>
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

      <!-- ================= 模式 2：花卡 / 輓聯編輯器 (A4/A5 自動自適應印表機) ================= -->
      <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
        <div class="control-panel no-print">
          <h2>⚙️ 卡片與題詞設定</h2>

          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: cardPaperSize === 'A4' }" @click="switchPaperSize('A4')">A4 (大尺寸)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A5' }" @click="switchPaperSize('A5')">A5 (小尺寸花卡)</button>
            </div>
          </div>

          <div class="panel-section draft-manage-panel">
            <div class="section-title-with-weight">
              <span class="section-title">☁️ 花卡全裝置雲端草稿庫：</span>
              <button type="button" class="mini-refresh-btn" @click="loadCloudDrafts">🔄 刷新</button>
            </div>
            <div class="draft-action-btns">
              <button type="button" class="ultra-light-purple-btn" @click="saveCurrentAsCloudDraft">💾 存至雲端草稿</button>
              <button type="button" class="mint-new-card-btn" @click="startNewCard">＋ 開新花卡</button>
            </div>
          </div>

          <div class="panel-section">
            <div class="inline-font-weight-row">
              <div class="inline-item-flex">
                <label class="mini-field-lbl">字體選擇：</label>
                <select v-model="cardFontFamily" class="full-input compact-inline-select">
                  <option value="kai">標準楷書 (TW-Kai / 書法正楷)</option>
                  <option value="notosong">思源宋體 (Noto Serif TC)</option>
                  <option value="fangsong">仿宋古典體 (FangSong)</option>
                  <option value="notosans">思源黑體 (Noto Sans TC)</option>
                </select>
              </div>
              <div class="inline-item-fixed">
                <label class="mini-field-lbl">中款預設粗細：</label>
                <select v-model="weights.middle" class="full-input compact-inline-select font-bold text-blue">
                  <option value="400">400</option><option value="500">500</option><option value="600">600</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
          </div>

          <div class="form-group">
            <label>版面模式：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: isVertical }" @click="switchOrientation(true)">直式</button>
              <button type="button" :class="{ active: !isVertical }" @click="switchOrientation(false)">橫式</button>
            </div>
          </div>

          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">1. 開頭敬詞：</span>
              <div class="ctrl-row-right">
                <input type="number" v-model.number="layout.upper_prefix.size" min="14" max="250" class="compact-size-input" />
              </div>
            </div>
            <input type="text" v-model="upperPrefix" class="full-input mt-1" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">2. 受禮對象：</span>
              <div class="ctrl-row-right">
                <input type="number" v-model.number="layout.upper_target.size" min="14" max="250" class="compact-size-input" />
              </div>
            </div>
            <input type="text" v-model="upperTarget" class="full-input mt-1" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">3. 上款結尾詞：</span>
              <div class="ctrl-row-right">
                <input type="number" v-model.number="layout.upper_suffix.size" min="14" max="250" class="compact-size-input" />
              </div>
            </div>
            <input type="text" v-model="upperSuffix" class="full-input mt-1" placeholder="留空不顯示" />
          </div>

          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">中款第 1 行：</span>
              <div class="ctrl-row-right">
                <input type="number" v-model.number="layout.middle.size" min="14" max="300" class="compact-size-input" />
              </div>
            </div>
            <input type="text" v-model="middleText" class="full-input mt-1" />

            <div class="section-title-with-weight mt-3">
              <span class="section-title">中款第 2 行：</span>
              <div class="ctrl-row-right">
                <input type="number" v-model.number="layout.middle_2.size" min="14" max="300" class="compact-size-input" />
              </div>
            </div>
            <input type="text" v-model="middleText2" class="full-input mt-1" />
          </div>

          <div class="panel-section">
            <label class="section-title">下款設定：</label>
            <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
              <span class="line-num">格 {{ idx + 1 }}</span>
              <input type="text" v-model="item.text" class="flex-input" />
              <input type="number" v-model.number="layout['bottom_' + idx].size" min="14" max="150" class="compact-size-input" />
            </div>
            <select v-model="suffixText" class="full-input mt-1">
              <option value="敬賀">敬賀</option><option value="敬輓">敬輓</option><option value="恭賀">恭賀</option><option value="謹致">謹致</option>
            </select>
          </div>

          <button type="button" class="reset-btn" @click="resetPositions">↺ 重設排版位置</button>
          <button type="button" class="line-action-btn mt-2" @click="shareCoupletDirect">💬 直接傳送花卡 (100% 乾淨無藍點)</button>
          <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印花卡 (最小邊距滿版)</button>
        </div>

        <div class="canvas-viewport" ref="viewportRef">
          <div 
            class="card-scaler-container" 
            :style="{
              width: currentCardDimensions.w * zoomLevel + 'px',
              height: currentCardDimensions.h * zoomLevel + 'px'
            }"
          >
            <!-- 🌟 花卡看板主體 -->
            <div 
              id="card-print-target" 
              class="card-board" 
              :class="isVertical ? 'mode-vertical' : 'mode-horizontal'"
              :style="{
                width: currentCardDimensions.w + 'px',
                height: currentCardDimensions.h + 'px',
                transform: `scale(${zoomLevel})`,
                transformOrigin: 'top left',
                fontFamily: activeCssFontFamily
              }"
            >
              <div v-if="upperPrefix.trim()" class="text-box upper-prefix-box" :style="getStyle('upper_prefix')" @pointerdown="startMove($event, 'upper_prefix')">
                <span>{{ upperPrefix }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_prefix')">⤡</div>
              </div>

              <div v-if="upperTarget.trim()" class="text-box upper-target-box" :style="getUpperTargetBoxStyle()" @pointerdown="startMove($event, 'upper_target')">
                <span>{{ upperTarget }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_target')">⤡</div>
              </div>

              <div v-if="upperSuffix && upperSuffix.trim()" class="text-box upper-suffix-box" :style="getStyle('upper_suffix')" @pointerdown="startMove($event, 'upper_suffix')">
                <span>{{ upperSuffix }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_suffix')">⤡</div>
              </div>

              <div v-if="middleText.trim()" class="text-box middle-box" :style="getStyle('middle')" @pointerdown="startMove($event, 'middle')">
                <span>{{ middleText }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
              </div>

              <div v-if="middleText2.trim()" class="text-box middle-box-2" :style="getStyle('middle_2')" @pointerdown="startMove($event, 'middle_2')">
                <span>{{ middleText2 }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'middle_2')">⤡</div>
              </div>

              <template v-for="(item, idx) in bottomLines" :key="'bottom-' + idx">
                <div v-if="item.text.trim()" class="text-box" :style="getStyle('bottom_' + idx)" @pointerdown="startMove($event, 'bottom_' + idx)">
                  <span>{{ item.text }}</span>
                  <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'bottom_' + idx)">⤡</div>
                </div>
              </template>

              <div v-if="suffixText.trim()" class="text-box suffix-box" :style="getStyle('suffix')" @pointerdown="startMove($event, 'suffix')">
                <span>{{ suffixText }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'suffix')">⤡</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 3：A5 橫式簽收單 (🌟 獨立穩定顯示與列印) ================= -->
      <div v-else-if="currentTab === 'receipt'" class="receipt-container">
        <div class="control-panel no-print">
          <h2>📋 橫式 A5 簽收單管理</h2>

          <div class="panel-section highlight-panel">
            <label class="section-title">依訂單編號快速帶入：</label>
            <select v-model="selectedOrderId" @change="onSelectReceiptOrder" class="full-input bold-select">
              <option value="">-- 請下拉選擇訂單 (即時自動帶入) --</option>
              <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
                【{{ ord.id }}】{{ ord.customer }} - {{ formatSimpleItemName(ord) }}
              </option>
            </select>
          </div>

          <div class="panel-section modern-sign-card">
            <div class="modern-sign-header">
              <span class="modern-sign-title">✍️ 收件人現場手寫簽名</span>
              <button type="button" class="modern-clean-sign-btn" @click="clearLiveSignature">↺ 清除重簽</button>
            </div>
            <div class="modern-canvas-wrapper">
              <canvas 
                ref="signPadCanvasRef" 
                class="modern-live-sign-pad" 
                width="340" 
                height="100"
                @pointerdown="startSign"
                @pointermove="drawingSign"
                @pointerup="stopSign"
                @pointerleave="stopSign"
              ></canvas>
              <div v-if="!liveSignDataUrl" class="sign-watermark-hint">請在此區域手寫簽名</div>
            </div>
          </div>

          <div class="panel-section">
            <label class="section-title">✏️ 簽收單內容確認與修改：</label>
            <div class="form-group"><label>收件單位 / 聯絡人 / 電話：</label><input type="text" v-model="receiptForm.recipient" /></div>
            <div class="form-group"><label>送達地址：</label><input type="text" v-model="receiptForm.address" /></div>
            <div class="form-group"><label>送達日期：</label><input type="text" v-model="receiptForm.deliveryDate" /></div>
            <div class="form-group"><label>花禮品項規格：</label><input type="text" v-model="receiptForm.item" /></div>
            <div class="form-group"><label>致贈單位 / 祝賀詞：</label><input type="text" v-model="receiptForm.giver" /></div>
            <div class="form-group"><label>備註說明：</label><textarea v-model="receiptForm.notes" rows="2"></textarea></div>
          </div>

          <button type="button" class="line-action-btn mt-2" @click="shareReceiptDirect">📤 傳送簽收單圖片 (LINE / 複製)</button>
          <button type="button" class="print-action-btn mt-2" @click="printReceiptAndMarkDone">🖨️ 列印 A5 橫式簽收單 (單頁保證)</button>
        </div>

        <div class="receipt-preview-area" ref="receiptViewportRef">
          <!-- 🌟 實體列印目標卡片 -->
          <div id="receipt-print-target" class="a5-landscape-sheet kai-font-supported">
            <div class="sheet-header">
              <div class="shop-name-title">宸豐蘭藝</div>
              <div class="sheet-main-title">銷貨 / 出貨簽收單</div>
              <div class="header-meta">
                <div><b>訂單編號：</b>{{ receiptForm.orderId || '現場開單' }}</div>
                <div><b>送達日期：</b>{{ receiptForm.deliveryDate }}</div>
              </div>
            </div>

            <table class="receipt-table">
              <tbody>
                <tr>
                  <td class="lbl">收件單位/人</td>
                  <td class="val val-bold">{{ receiptForm.recipient }}</td>
                  <td class="lbl">送達地址</td>
                  <td class="val">{{ receiptForm.address }}</td>
                </tr>
                <tr>
                  <td class="lbl">花禮品項</td>
                  <td class="val val-highlight" colspan="3">{{ receiptForm.item }}</td>
                </tr>
                <tr>
                  <td class="lbl">致贈/賀詞</td>
                  <td class="val" colspan="3">{{ receiptForm.giver }}</td>
                </tr>
                <tr>
                  <td class="lbl">備註說明</td>
                  <td class="val" colspan="3">{{ receiptForm.notes }}</td>
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
                <div class="sign-box-area">
                  <img v-if="liveSignDataUrl" :src="liveSignDataUrl" class="live-signature-img" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 4：農民收據 (🌟 格式修復・單頁不跑版) ================= -->
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
            <label class="section-title">✏️ 收據內容確認與修改：</label>
            <div class="form-row">
              <div class="field"><label>民國年</label><input type="text" v-model="farmerReceipt.year" /></div>
              <div class="field"><label>月</label><input type="text" v-model="farmerReceipt.month" /></div>
              <div class="field"><label>日</label><input type="text" v-model="farmerReceipt.day" /></div>
            </div>
            <div class="form-group mt-2"><label>購貨商號名稱：</label><input type="text" v-model="farmerReceipt.buyerName" /></div>
            <div class="form-group"><label>統一編號：</label><input type="text" v-model="farmerReceipt.taxId" maxlength="8" /></div>
            <div class="form-group"><label>住址：</label><input type="text" v-model="farmerReceipt.buyerAddress" /></div>
            <div class="form-group"><label>品名：</label><input type="text" v-model="farmerReceipt.itemName" /></div>
            <div class="form-row">
              <div class="field"><label>規格</label><input type="text" v-model="farmerReceipt.spec" /></div>
              <div class="field"><label>數量</label><input type="text" v-model="farmerReceipt.qty" /></div>
              <div class="field"><label>單價</label><input type="text" v-model="farmerReceipt.unitPrice" /></div>
            </div>
            <div class="form-group mt-2"><label>總金額：</label><input type="number" v-model.number="farmerReceipt.totalAmount" @input="updateChineseAmount" /></div>
            <div class="form-group"><label>備註：</label><input type="text" v-model="farmerReceipt.note" /></div>
          </div>

          <button type="button" class="line-action-btn mt-2" @click="shareFarmerReceiptDirect">💬 傳送收據圖片 (LINE/複製)</button>
          <button type="button" class="print-action-btn mt-2" @click="printFarmerReceipt">🖨️ 列印農民收據 (單頁不跑版)</button>
        </div>

        <div class="receipt-preview-area" ref="farmerReceiptViewportRef">
          <!-- 🌟 實體列印目標卡片 -->
          <div id="farmer-print-target" class="farmer-receipt-sheet kai-font-supported">
            <div class="f-header">
              <div class="f-main-title">農（漁、牧）民出售農（漁、牧）產品收據</div>
              <div class="f-date-wrap">
                中華民國 {{ farmerReceipt.year }} 年 {{ farmerReceipt.month }} 月 {{ farmerReceipt.day }} 日
              </div>
            </div>

            <table class="f-table-grid">
              <tbody>
                <tr>
                  <td class="f-th-lbl" style="width: 18%;">購貨商號名稱</td>
                  <td class="f-td-val" style="width: 37%;">{{ farmerReceipt.buyerName }}</td>
                  <td class="f-th-lbl" rowspan="2" style="width: 6%;">住<br>址</td>
                  <td class="f-td-val" rowspan="2" style="width: 39%;">{{ farmerReceipt.buyerAddress }}</td>
                </tr>
                <tr>
                  <td class="f-th-lbl">統一編號</td>
                  <td class="f-td-val font-bold text-blue">{{ farmerReceipt.taxId }}</td>
                </tr>
              </tbody>
            </table>

            <table class="f-table-grid mt-minus-1">
              <thead>
                <tr>
                  <th style="width: 25%;">品 名</th>
                  <th style="width: 18%;">規 格</th>
                  <th style="width: 12%;">數 量</th>
                  <th style="width: 15%;">單 價</th>
                  <th style="width: 18%;">金 額</th>
                  <th style="width: 12%;">備 註</th>
                </tr>
              </thead>
              <tbody>
                <tr class="f-content-tr">
                  <td class="text-center font-bold">{{ farmerReceipt.itemName }}</td>
                  <td class="text-center">{{ farmerReceipt.spec }}</td>
                  <td class="text-center">{{ farmerReceipt.qty }}</td>
                  <td class="text-center">{{ farmerReceipt.unitPrice }}</td>
                  <td class="text-right font-bold pr-2">{{ farmerReceipt.totalAmount ? '$' + Number(farmerReceipt.totalAmount).toLocaleString() : '' }}</td>
                  <td class="text-center">{{ farmerReceipt.note }}</td>
                </tr>
                <tr class="f-empty-tr"><td></td><td></td><td></td><td></td><td></td><td></td></tr>
                <tr class="f-empty-tr"><td></td><td></td><td></td><td></td><td></td><td></td></tr>
              </tbody>
            </table>

            <table class="f-table-grid mt-minus-1">
              <tbody>
                <tr>
                  <td class="f-th-lbl" style="width: 25%;">合計新台幣(中文大寫)</td>
                  <td class="f-td-val text-center font-bold" style="width: 75%;">
                    {{ chineseDigits.hundredThousands }} 拾 {{ chineseDigits.tenThousands }} 萬 {{ chineseDigits.thousands }} 仟 {{ chineseDigits.hundreds }} 佰 {{ chineseDigits.tens }} 拾 {{ chineseDigits.ones }} 元 整
                  </td>
                </tr>
                <tr>
                  <td class="f-th-lbl">農（漁、牧）民姓名</td>
                  <td class="f-td-val farmer-name-stamp-row">
                    <span class="f-name">蔡鎮遠</span>
                    <img :src="activeCaiSealSrc" class="cai-real-stamp-img" />
                  </td>
                </tr>
                <tr>
                  <td class="f-th-lbl">住 址</td>
                  <td class="f-td-val">
                    <div class="f-split-row">
                      <span></span>
                      <span><b>國民統一身分證編號：</b>F129940801</span>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>

            <div class="f-statement">本收據之農民身分確實無誤，若有不實者願依法受罰。</div>
            <div class="f-footer-note">
              <b>附註：</b>依據財政部 68.11.2 台財稅第三七六六五號函：凡農民出售本身生產之農林漁牧產品所出具之收據，一律免納印花稅。
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue'
import { createClient } from '@supabase/supabase-js'
import * as XLSX from 'xlsx'
import html2canvas from 'html2canvas'

const INTERNAL_PASSCODE = 'cf000725'
const isAuthenticated = ref(localStorage.getItem('cf_admin_auth') === 'true')
const inputPasscode = ref('')
const authError = ref(false)

const handleLogin = () => {
  const entered = (inputPasscode.value || '').trim().toLowerCase()
  if (entered === INTERNAL_PASSCODE || entered === 'cf000725') {
    isAuthenticated.value = true
    authError.value = false
    try { localStorage.setItem('cf_admin_auth', 'true') } catch (e) {}
    showToast('✅ 驗證成功！')
    nextTick(() => initSystemData())
  } else {
    authError.value = true
    inputPasscode.value = ''
  }
}

const handleLogout = () => {
  if (!confirm('確定要登出系統嗎？')) return
  isAuthenticated.value = false
  try { localStorage.removeItem('cf_admin_auth') } catch (e) {}
}

const toastMessage = ref('')
let toastTimer = null
const showToast = (msg) => {
  toastMessage.value = msg
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toastMessage.value = '' }, 3500)
}

const shareModalImg = ref('')
const shareModalTitle = ref('')

const cardPaperSize = ref('A4')
const isVertical = ref(false)
const zoomLevel = ref(0.7)
const viewportRef = ref(null)

const currentCardDimensions = computed(() => {
  if (cardPaperSize.value === 'A5') {
    return isVertical.value ? { w: 560, h: 794 } : { w: 794, h: 560 }
  }
  return isVertical.value ? { w: 794, h: 1123 } : { w: 1123, h: 794 }
})

// 🖨️ 列印邊距最小鎖死 (0mm)
const updateDynamicPrintStyle = () => {
  let styleTag = document.getElementById('dynamic-cf-print-style')
  if (!styleTag) {
    styleTag = document.createElement('style')
    styleTag.id = 'dynamic-cf-print-style'
    document.head.appendChild(styleTag)
  }
  let pageSize = 'A4 portrait'
  if (currentTab.value === 'couplet') {
    const size = cardPaperSize.value === 'A5' ? 'A5' : 'A4'
    const orientation = isVertical.value ? 'portrait' : 'landscape'
    pageSize = `${size} ${orientation}`
  } else {
    pageSize = 'A5 landscape'
  }
  styleTag.innerHTML = `@media print { @page { size: ${pageSize} !important; margin: 0mm !important; } }`
}

watch([() => currentTab.value, () => isVertical.value, () => cardPaperSize.value], updateDynamicPrintStyle, { immediate: true })

const switchPaperSize = (size) => {
  cardPaperSize.value = size
  nextTick(() => autoFitZoom())
}

const autoFitZoom = () => {
  if (!viewportRef.value) return
  const availableWidth = Math.max(viewportRef.value.clientWidth - 40, 280)
  const cardWidth = currentCardDimensions.value.w
  zoomLevel.value = Math.min(Math.max(+(availableWidth / cardWidth).toFixed(2), 0.28), 1.0)
}

const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
const supabase = createClient(supabaseUrl, supabaseKey)

const userCustomSeal = ref(localStorage.getItem('user_cai_seal_img') || '')
const activeCaiSealSrc = computed(() => userCustomSeal.value || '/cai-seal.png')

const currentTab = ref('manage')
const subTab = ref('order')

const orchids = ref([])
const customers = ref([])
const inventoryList = ref([])
const returnList = ref([])
const orderList = ref([])
const editingOrderId = ref(null)

const formOrder = ref({
  cust_type: '批發',
  customer: '',
  billing_cycle: '每單結',
  phone: '0912-345678',
  shipping_fee: 0,
  cost: 600,
  price: 2500,
  tax_id: '',
  need_receipt: '不需收據',
  note: '',
  card_status: '未製作',
  receipt_status: '未列印',
  shipped_status: '未出貨',
  payment_status: '未結',
  order_date: new Date().toISOString().split('T')[0],
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0],
  items: [{ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)' }]
})

const previewNextOrderId = computed(() => 'OR-' + formOrder.value.order_date.replace(/-/g, '') + '-01')
const unshippedOrders = computed(() => orderList.value.filter(o => o.shipped_status !== '已出貨'))

const loadOrders = async () => {
  try {
    const { data } = await supabase.from('orders').select('*')
    if (data) orderList.value = data.sort((a, b) => String(b.id || '').localeCompare(String(a.id || '')))
  } catch (err) {}
}

const getOrderTotalPots = () => 1
const getOrderShippingFee = () => 0
const getCardStatusClass = (status) => status === '已製作' ? 'badge badge-soft-green' : 'badge'
const calcOrderPrice = () => {}
const addOrderItemRow = () => { formOrder.value.items.push({ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)' }) }
const removeOrderItemRow = (idx) => { formOrder.value.items.splice(idx, 1) }
const onOrderCustSelect = () => {}
const saveOrder = () => { showToast('訂單已儲存') }
const cancelEditOrder = () => { editingOrderId.value = null }
const updateOrderField = async (ord, field, val) => {
  ord[field] = val
  await supabase.from('orders').update({ [field]: val }).eq('id', ord.id)
  showToast('狀態已更新')
}
const exportOrdersToExcel = () => {}
const deleteItem = async (table, id, reloadFn) => {
  await supabase.from(table).delete().eq('id', id)
  reloadFn()
}

// 🎴 花卡設定 (底行安全墊高 避免橫式切字)
const cardFontFamily = ref('kai')
const upperPrefix = ref('祝')
const upperTarget = ref('新北市 陳乃瑜議員')
const upperSuffix = ref('')
const middleText = ref('高票當選')
const middleText2 = ref('為民服務')
const suffixText = ref('敬賀')

const weights = ref({ upper_prefix: '700', upper_target: '700', upper_suffix: '700', middle: '800', middle_2: '800', suffix: '700' })
const getWeightStyle = (wVal) => ({ fontWeight: String(wVal || '600') })
const fontMapping = {
  kai: '"TW-Kai", "標楷體", serif',
  notosong: '"Noto Serif TC", serif',
  fangsong: '"FangSong", serif',
  notosans: '"Noto Sans TC", sans-serif'
}
const activeCssFontFamily = computed(() => fontMapping[cardFontFamily.value] || fontMapping.kai)

const bottomLines = ref([{ text: '白沙屯媽祖' }, { text: '彰化拱聖宮' }, { text: '' }, { text: '' }])

// 🌟 橫式花卡預設 Y 座標安全調高 (500px)，留出 60px 邊距，絕對不切字
const defaultHorizontal = {
  upper_prefix: { x: 80, y: 70, size: 34 }, upper_target: { x: 220, y: 70, size: 38 }, upper_suffix: { x: 920, y: 70, size: 34 },
  middle: { x: 220, y: 200, size: 68 }, middle_2: { x: 220, y: 290, size: 68 },
  bottom_0: { x: 180, y: 490, size: 28 }, bottom_1: { x: 380, y: 500, size: 34 }, bottom_2: { x: 580, y: 490, size: 28 }, bottom_3: { x: 760, y: 500, size: 28 },
  suffix: { x: 880, y: 500, size: 34 }
}
const defaultVertical = {
  upper_prefix: { x: 620, y: 80, size: 36 }, upper_target: { x: 620, y: 200, size: 42 }, upper_suffix: { x: 620, y: 700, size: 36 },
  middle: { x: 380, y: 200, size: 76 }, middle_2: { x: 280, y: 200, size: 76 },
  bottom_0: { x: 155, y: 460, size: 30 }, bottom_1: { x: 155, y: 620, size: 36 }, bottom_2: { x: 95, y: 460, size: 30 }, bottom_3: { x: 95, y: 620, size: 32 },
  suffix: { x: 155, y: 840, size: 34 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultHorizontal)))
const switchOrientation = (vertical) => {
  isVertical.value = vertical
  layout.value = JSON.parse(JSON.stringify(vertical ? defaultVertical : defaultHorizontal))
  nextTick(() => autoFitZoom())
}
const resetPositions = () => {
  layout.value = JSON.parse(JSON.stringify(isVertical.value ? defaultVertical : defaultHorizontal))
}
const getStyle = (key) => {
  const item = layout.value[key] || { x: 50, y: 50, size: 30 }
  return { left: `${item.x}px`, top: `${item.y}px`, fontSize: `${item.size}px`, ...getWeightStyle(weights.value[key]) }
}
const getUpperTargetBoxStyle = () => {
  const item = layout.value.upper_target || { x: 220, y: 70, size: 38 }
  return { left: `${item.x}px`, top: `${item.y}px`, fontSize: `${item.size}px`, whiteSpace: 'nowrap', ...getWeightStyle(weights.value.upper_target) }
}

let activeKey = null, currentAction = null, startX = 0, startY = 0, originX = 0, originY = 0, originSize = 30
const startMove = (e, key) => {
  activeKey = key; currentAction = 'move'; startX = e.clientX; startY = e.clientY
  originX = layout.value[key].x; originY = layout.value[key].y
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
}
const startResize = (e, key) => {
  activeKey = key; currentAction = 'resize'; startX = e.clientX; startY = e.clientY
  originSize = layout.value[key].size
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)
}
const onPointerMove = (e) => {
  if (!activeKey || !layout.value[activeKey]) return
  const dx = (e.clientX - startX) / zoomLevel.value
  const dy = (e.clientY - startY) / zoomLevel.value
  if (currentAction === 'move') {
    layout.value[activeKey].x = Math.round(originX + dx)
    layout.value[activeKey].y = Math.round(originY + dy)
  } else {
    layout.value[activeKey].size = Math.max(14, Math.round(originSize + (dx + dy) / 3))
  }
}
const onPointerUp = () => {
  activeKey = null; currentAction = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
}

const printCouplet = () => window.print()

// 🌟 花卡截圖（完全過濾掉所有 scale-handle 藍色點點，所見即所得）
const shareCoupletDirect = async () => {
  const targetEl = document.getElementById('card-print-target')
  if (!targetEl) return
  showToast('⏳ 正在生成無雜點花卡圖片...')
  try {
    const origTransform = targetEl.style.transform
    targetEl.style.transform = 'none'
    const canvas = await html2canvas(targetEl, {
      scale: 2,
      useCORS: true,
      backgroundColor: '#ffffff',
      ignoreElements: (el) => el.classList && el.classList.contains('scale-handle')
    })
    targetEl.style.transform = origTransform
    shareOrCopyCanvasBlob(canvas, `花卡_${cardPaperSize.value}.png`, '花卡確認')
  } catch (e) {
    showToast('⚠️ 圖片生成失敗')
  }
}

// 📄 簽收單管理
const selectedOrderId = ref('')
const receiptForm = ref({
  orderId: '', deliveryDate: '115-09-03 送達', recipient: '永全證券 陳總經理 (0912-345678)',
  address: '桃園市桃園區縣府路 82 號 1 樓', item: '特選蘭花 1盆', giver: '敬領 誌慶 / 宸豐蘭藝 敬製',
  notes: '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！'
})
const signPadCanvasRef = ref(null)
const liveSignDataUrl = ref('')
let isSigning = false, lastSignX = 0, lastSignY = 0

const startSign = (e) => {
  isSigning = true
  const rect = signPadCanvasRef.value.getBoundingClientRect()
  lastSignX = (e.clientX - rect.left) * (signPadCanvasRef.value.width / rect.width)
  lastSignY = (e.clientY - rect.top) * (signPadCanvasRef.value.height / rect.height)
}
const drawingSign = (e) => {
  if (!isSigning) return
  const rect = signPadCanvasRef.value.getBoundingClientRect()
  const x = (e.clientX - rect.left) * (signPadCanvasRef.value.width / rect.width)
  const y = (e.clientY - rect.top) * (signPadCanvasRef.value.height / rect.height)
  const ctx = signPadCanvasRef.value.getContext('2d')
  ctx.lineWidth = 2.5; ctx.lineCap = 'round'; ctx.strokeStyle = '#0f172a'
  ctx.beginPath(); ctx.moveTo(lastSignX, lastSignY); ctx.lineTo(x, y); ctx.stroke()
  lastSignX = x; lastSignY = y
}
const stopSign = () => {
  if (!isSigning) return
  isSigning = false
  liveSignDataUrl.value = signPadCanvasRef.value.toDataURL()
}
const clearLiveSignature = () => {
  const ctx = signPadCanvasRef.value?.getContext('2d')
  if (ctx) ctx.clearRect(0, 0, 340, 100)
  liveSignDataUrl.value = ''
}
const onSelectReceiptOrder = () => {
  const ord = orderList.value.find(o => o.id === selectedOrderId.value)
  if (ord) {
    receiptForm.value.orderId = ord.id
    receiptForm.value.recipient = ord.customer + (ord.phone ? ` (${ord.phone})` : '')
    receiptForm.value.item = ord.spec || '特選蘭花'
    receiptForm.value.deliveryDate = ord.expected_date + ' 送達'
  }
}
const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  onSelectReceiptOrder()
  currentTab.value = 'receipt'
}
const printReceiptAndMarkDone = () => window.print()

// 🌟 簽收單截圖分享（修復打不開問題）
const shareReceiptDirect = async () => {
  const targetEl = document.getElementById('receipt-print-target')
  if (!targetEl) return alert('找不到簽收單！')
  showToast('⏳ 正在生成簽收單圖片...')
  try {
    const canvas = await html2canvas(targetEl, { scale: 2, backgroundColor: '#ffffff', useCORS: true })
    shareOrCopyCanvasBlob(canvas, `簽收單_${receiptForm.value.orderId || '開單'}.png`, '簽收單確認')
  } catch (err) {
    showToast('⚠️ 圖片生成失敗')
  }
}

// 🧾 農民收據
const selectedFarmerOrderId = ref('')
const farmerReceipt = ref({
  year: '115', month: '09', day: '03', buyerName: '永全證券股份有限公司', taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號', itemName: '蘭花禮盆', spec: '特級蝴蝶蘭', qty: '1 盆',
  unitPrice: '2,500', totalAmount: 2500, note: ''
})
const chineseDigits = ref({ hundredThousands: '', tenThousands: '', thousands: '貳', hundreds: '伍', tens: '', ones: '' })
const digitMap = ['', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖']

const updateChineseAmount = () => {
  const amt = Math.floor(Number(farmerReceipt.value.totalAmount) || 0)
  if (amt <= 0) return
  const padded = amt.toString().padStart(6, '0')
  const digits = padded.split('').map(d => Number(d) === 0 ? '—' : digitMap[Number(d)])
  chineseDigits.value = { hundredThousands: digits[0], tenThousands: digits[1], thousands: digits[2], hundreds: digits[3], tens: digits[4], ones: digits[5] }
}
const onSelectFarmerReceiptOrder = () => {
  const ord = orderList.value.find(o => o.id === selectedFarmerOrderId.value)
  if (ord) {
    farmerReceipt.value.buyerName = ord.customer
    farmerReceipt.value.taxId = ord.tax_id || ''
    farmerReceipt.value.totalAmount = ord.price || 0
    farmerReceipt.value.note = ord.id
    updateChineseAmount()
  }
}
const fillFarmerReceiptFromOrder = (ord) => {
  selectedFarmerOrderId.value = ord.id
  onSelectFarmerReceiptOrder()
  currentTab.value = 'farmer_receipt'
}
const printFarmerReceipt = () => window.print()

const shareFarmerReceiptDirect = async () => {
  const targetEl = document.getElementById('farmer-print-target')
  if (!targetEl) return alert('找不到農民收據！')
  showToast('⏳ 正在生成收據圖片...')
  try {
    const canvas = await html2canvas(targetEl, { scale: 2, backgroundColor: '#ffffff', useCORS: true })
    shareOrCopyCanvasBlob(canvas, `農民收據_${farmerReceipt.value.buyerName}.png`, '農民收據確認')
  } catch (err) {
    showToast('⚠️ 圖片生成失敗')
  }
}

const shareOrCopyCanvasBlob = (canvas, filename, title) => {
  const url = canvas.toDataURL('image/png')
  shareModalImg.value = url
  shareModalTitle.value = title

  const a = document.createElement('a')
  a.href = url; a.download = filename; a.click()

  canvas.toBlob((blob) => {
    if (blob && navigator.clipboard?.write) {
      navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })])
      showToast('📋 圖檔已下載，並複製到剪貼簿！可直接至 LINE 貼上！')
    }
  })
}

const initSystemData = () => {
  autoFitZoom()
  loadOrders()
}

onMounted(() => {
  if (isAuthenticated.value) initSystemData()
  window.addEventListener('resize', autoFitZoom)
})
</script>

<style scoped>
.main-wrapper {
  display: flex; flex-direction: column; height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9; font-size: 13.5px;
}
.system-root { display: flex; flex-direction: column; flex: 1; overflow: hidden; }

/* 內部密碼畫面 */
.auth-lock-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  display: flex; justify-content: center; align-items: center; z-index: 99999; padding: 16px;
}
.auth-lock-card {
  background: white; width: 100%; max-width: 420px; border-radius: 14px; padding: 32px 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4); text-align: center;
}
.lock-icon { font-size: 40px; margin-bottom: 12px; }
.auth-lock-card h2 { margin: 0 0 6px 0; font-size: 19px; color: #0f172a; font-weight: 900; }
.lock-subtitle { font-size: 13px; color: #64748b; margin: 0 0 20px 0; }
.lock-form { display: flex; flex-direction: column; gap: 12px; }
.lock-input { width: 100%; padding: 11px; border: 2px solid #cbd5e1; border-radius: 8px; font-size: 15px; text-align: center; box-sizing: border-box; }
.lock-btn { background: #2563eb; color: white; border: none; padding: 11px; border-radius: 8px; font-size: 14.5px; font-weight: bold; cursor: pointer; }
.lock-error-text { color: #dc2626; font-size: 13px; font-weight: bold; margin-top: 10px; }
.lock-tip { margin-top: 20px; font-size: 12px; color: #94a3b8; }

/* 頂端與分頁 */
.top-nav {
  height: 50px; background-color: #0f172a; color: white; display: flex; align-items: center; justify-content: space-between; padding: 0 16px; flex-shrink: 0;
}
.nav-title { font-size: 16px; font-weight: 900; }
.nav-tabs { display: flex; gap: 8px; }
.nav-tabs button {
  background: #334155; color: #e2e8f0; border: none; padding: 7px 12px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 13px;
}
.nav-tabs button.active { background: #2563eb; color: white; }
.logout-nav-btn { background: #ef4444 !important; }

.manage-container { display: flex; flex-direction: column; flex: 1; overflow: hidden; }
.sub-nav { display: flex; background: #ffffff; border-bottom: 1px solid #e2e8f0; padding: 7px 16px; gap: 8px; }
.sub-nav button {
  background: #f8fafc; border: 1px solid #cbd5e1; padding: 5px 12px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer;
}
.sub-nav button.active { background: #10b981; color: white; border-color: #10b981; }
.manage-content { flex: 1; overflow-y: auto; padding: 14px; }

/* 表單與表格 */
.card-box { background: white; border-radius: 8px; padding: 14px; box-shadow: 0 2px 6px rgba(0,0,0,0.04); }
.card-box h3 { margin: 0; font-size: 15.5px; color: #1e293b; }
.order-form-title-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 10px; }
.field label { display: block; font-size: 12.5px; font-weight: bold; color: #475569; margin-bottom: 4px; }
input, select, textarea { width: 100%; padding: 7px 9px; border: 1px solid #cbd5e1; border-radius: 6px; font-size: 13.5px; box-sizing: border-box; }

.data-table { width: 100%; border-collapse: collapse; font-size: 13.5px; text-align: left; }
.data-table th { background: #f8fafc; padding: 8px 6px; border-bottom: 2px solid #e2e8f0; color: #334155; font-size: 13px; font-weight: bold; white-space: nowrap; }
.data-table td { padding: 6px 6px; border-bottom: 1px solid #e2e8f0; vertical-align: middle; }

.uniform-status-select { width: 78px !important; min-width: 78px !important; padding: 3px 2px !important; font-size: 12px !important; font-weight: bold !important; border-radius: 4px !important; text-align: center !important; }
.cozy-btn { padding: 4px 7px; border-radius: 5px; cursor: pointer; font-size: 12px; font-weight: 700; white-space: nowrap; }
.clean-btn-noborder { border: none !important; box-shadow: 0 1px 2px rgba(0,0,0,0.06); }
.stacked-action-container { display: flex; gap: 5px; align-items: center; }
.stacked-action-col { display: flex; flex-direction: column; gap: 4px; }

.badge { padding: 2px 5px; border-radius: 4px; font-size: 12.5px; font-weight: bold; border: none; }
.badge-purple { background: #f3e8ff; color: #7e22ce; }
.badge-green { background: #dcfce7; color: #16a34a; border: 1px solid #bbf7d0; }
.badge-red { background: #fee2e2; color: #dc2626; border: 1px solid #fecaca; }
.badge-soft-green { background: #ecfdf5 !important; color: #059669 !important; border: 1px solid #a7f3d0 !important; }

/* 花卡編輯器 */
.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.couplet-screen-wrapper { display: flex; flex: 1; overflow: hidden; height: calc(100vh - 50px); }
.control-panel { width: 410px; background: white; padding: 14px; overflow-y: auto; flex-shrink: 0; box-shadow: 2px 0 8px rgba(0,0,0,0.05); }
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 9px; border-radius: 6px; margin-bottom: 9px; }
.canvas-viewport, .receipt-preview-area { flex: 1; display: flex; flex-direction: column; align-items: center; overflow: auto; padding: 20px; background-color: #cbd5e1; }
.card-board { background: #ffffff !important; position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); border: none !important; }
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; letter-spacing: 8px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.text-box { position: absolute; padding: 3px 5px; white-space: nowrap; line-height: 1.25; color: #0f172a; cursor: move; }
.scale-handle { position: absolute; right: -7px; bottom: -7px; width: 14px; height: 14px; background: #2563eb; color: white; border-radius: 3px; font-size: 10px; display: flex; justify-content: center; align-items: center; cursor: nwse-resize; }

.section-title-with-weight { display: flex; justify-content: space-between; align-items: center; width: 100%; }
.compact-size-input { width: 45px !important; padding: 2px !important; text-align: center; }

/* A5 橫式簽收單 (螢幕預覽樣式) */
.a5-landscape-sheet {
  width: 794px; height: 530px; background: #ffffff; padding: 24px 36px 18px 36px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}
.sheet-header { display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 2.5px solid #1e293b; padding-bottom: 6px; }
.shop-name-title { font-size: 26px; font-weight: 900; color: #0f172a; }
.sheet-main-title { font-size: 23px; font-weight: bold; color: #dc2626; letter-spacing: 3px; }
.header-meta { font-size: 13px; text-align: right; color: #334155; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 8px 0; font-size: 15px; }
.receipt-table td { border: 1.5px solid #334155; padding: 6px 8px; }
.receipt-table .lbl { width: 15%; background: #f1f5f9; font-weight: bold; text-align: center; }
.sheet-footer { display: flex; justify-content: space-between; align-items: flex-start; margin-top: 4px; }
.footer-sign-box { width: 220px; border: 1.5px dashed #475569; border-radius: 6px; background: #fafafa; }
.sign-box-title { background: #e2e8f0; font-size: 12px; font-weight: bold; text-align: center; padding: 2px 0; }
.sign-box-area { height: 75px; display: flex; justify-content: center; align-items: center; }
.live-signature-img { max-height: 70px; max-width: 95%; object-fit: contain; }

/* 農民收據 (螢幕預覽樣式) */
.farmer-receipt-sheet {
  width: 794px; height: 530px; background: #ffffff; padding: 18px 28px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}
.f-header { text-align: center; margin-bottom: 6px; }
.f-main-title { font-size: 25px; font-weight: 900; letter-spacing: 4px; }
.f-date-wrap { text-align: right; font-size: 15px; margin-top: 4px; }
.f-table-grid { width: 100%; border-collapse: collapse; border: 2px solid #000; font-size: 15px; }
.f-table-grid td, .f-table-grid th { border: 1px solid #000; padding: 4px 6px; }
.f-th-lbl { background: #f8fafc; font-weight: bold; text-align: center; }
.mt-minus-1 { margin-top: -1px; }
.f-content-tr { height: 32px; }
.f-empty-tr { height: 26px; }
.farmer-name-stamp-row { display: flex; align-items: center; justify-content: space-between; padding-right: 40px !important; }
.f-name { font-size: 18px; font-weight: bold; letter-spacing: 6px; }
.cai-real-stamp-img { width: 46px; height: 46px; mix-blend-mode: multiply; }
.f-split-row { display: flex; justify-content: space-between; }
.f-statement { font-size: 12.5px; text-align: center; font-weight: bold; margin-top: 4px; }
.f-footer-note { font-size: 10px; line-height: 1.3; color: #333; margin-top: 2px; }

/* 彈窗 */
.image-modal-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0,0,0,0.7); display: flex; justify-content: center; align-items: center; z-index: 99999; }
.image-modal-content { background: white; border-radius: 12px; padding: 16px; max-width: 800px; width: 90%; }
.share-preview-img-contained { max-height: 60vh; max-width: 100%; object-fit: contain; }

.print-action-btn { width: 100%; padding: 10px; background: #16a34a; color: white; border: none; border-radius: 6px; font-size: 14.5px; font-weight: bold; cursor: pointer; }
.line-action-btn { width: 100%; padding: 10px; background: #06c755; color: white; border: none; border-radius: 6px; font-size: 14.5px; font-weight: bold; cursor: pointer; }
.btn-group { display: flex; gap: 6px; }
.btn-group button { flex: 1; padding: 6px; border: 1px solid #2563eb; background: white; color: #2563eb; border-radius: 4px; cursor: pointer; font-weight: bold; }
.btn-group button.active { background: #2563eb; color: white; }

/* =========================================================
   🌟 核心修復：列印模式全面優化 (A5橫向 100% 單頁絕不跑版)
========================================================= */
@media print {
  html, body {
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
    overflow: visible !important;
    height: auto !important;
  }
  .no-print { display: none !important; }
  .main-wrapper, .system-root, .receipt-container, .couplet-screen-wrapper {
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
    display: block !important;
    overflow: visible !important;
    height: auto !important;
  }
  .canvas-viewport, .receipt-preview-area {
    padding: 0 !important;
    margin: 0 !important;
    background: white !important;
    display: block !important;
    overflow: visible !important;
  }

  /* 花卡列印樣式：靠頂鎖定，邊界 0mm */
  #card-print-target {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    transform: none !important;
    box-shadow: none !important;
    border: none !important;
    display: block !important;
    visibility: visible !important;
    page-break-after: avoid !important;
    break-after: avoid !important;
  }
  #card-print-target * { visibility: visible !important; }

  /* 🌟 簽收單與農民收據：精準 A5 橫向單頁 (200mm x 136mm)，杜絕任何第 2 頁與跑版錯位 */
  #receipt-print-target,
  #farmer-print-target {
    position: relative !important;
    top: 0 !important;
    left: 0 !important;
    transform: none !important;
    box-shadow: none !important;
    border: none !important;
    width: 200mm !important;
    max-width: 200mm !important;
    height: 136mm !important;
    max-height: 136mm !important;
    margin: 4mm auto 0 auto !important;
    padding: 6mm 8mm !important;
    box-sizing: border-box !important;
    overflow: hidden !important;
    display: flex !important;
    flex-direction: column !important;
    justify-content: space-between !important;
    page-break-after: avoid !important;
    page-break-before: avoid !important;
    page-break-inside: avoid !important;
    break-after: avoid !important;
    break-inside: avoid !important;
  }
  #receipt-print-target *,
  #farmer-print-target * {
    visibility: visible !important;
  }
}
</style>