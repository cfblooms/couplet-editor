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
              </div>

              <!-- 多組花禮規格設定區塊 -->
              <div class="items-section mt-3">
                <div class="items-header">
                  <h4>🌸 花禮品項與盆數規格（支援庫存帶入或自由輸入）</h4>
                  <button type="button" class="add-item-btn" @click="addOrderItemRow">＋ 新增一組花禮規格</button>
                </div>

                <datalist id="inv-flower-select-options">
                  <option 
                    v-for="f in flowerInventory" 
                    :key="f.id" 
                    :value="f.item_name"
                  >
                    【庫存 {{ f.qty }} 棵】規格: {{ f.spec }} / 成本單價 ${{ f.unit_cost || 0 }}
                  </option>
                </datalist>

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
                      🗑️
                    </button>
                  </div>
                  <div class="item-grid">
                    <div class="field">
                      <label>品種名稱</label>
                      <input 
                        v-model="item.orchid_name" 
                        list="inv-flower-select-options" 
                        @change="onFlowerSelectChange(item)" 
                        placeholder="手動輸入或選取庫存" 
                      />
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
                        <option value="無盆">無盆 (裸株/自備盆)</option>
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
                      <th class="col-id">單號</th>
                      <th class="col-date">下單日</th>
                      <th class="col-cust">客戶名稱</th>
                      <th class="col-tax">統編</th>
                      <th class="col-receipt">開收據</th>
                      <th class="col-pots">總盆數</th>
                      <th class="col-spec">規格明細</th>
                      <th class="col-ship">運費</th>
                      <th class="col-price">總售價</th>
                      <th class="col-status">花卡</th>
                      <th class="col-status">簽收單</th>
                      <th class="col-status">出貨</th>
                      <th class="col-status">收款</th>
                      <th class="col-actions">單據／操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="ord in orderList" :key="ord.id">
                      <td><b>{{ ord.id }}</b></td>
                      <td>{{ ord.order_date }}</td>
                      <td><b>{{ ord.customer }}</b></td>
                      <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                      <td>
                        <select 
                          v-model="ord.need_receipt" 
                          :class="ord.need_receipt === '需開收據' ? 'badge badge-green' : 'badge badge-gray'"
                          @change="updateOrderField(ord, 'need_receipt', ord.need_receipt)"
                          class="uniform-status-select"
                        >
                          <option value="不需收據">不需收據</option>
                          <option value="需開收據">需開收據</option>
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
                          <option value="未製作">未製作</option>
                          <option value="已製作">已製作</option>
                          <option value="免製作">免製作</option>
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
                          <option value="未列印">未列印</option>
                          <option value="已列印">已列印</option>
                        </select>
                      </td>

                      <td>
                        <select 
                          v-model="ord.shipped_status" 
                          :class="ord.shipped_status === '已出貨' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)"
                          class="uniform-status-select"
                        >
                          <option value="未出貨">未出貨</option>
                          <option value="已出貨">已出貨</option>
                        </select>
                      </td>

                      <td>
                        <select 
                          v-model="ord.payment_status" 
                          :class="ord.payment_status === '已結' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'payment_status', ord.payment_status)"
                          class="uniform-status-select"
                        >
                          <option value="未結">未結</option>
                          <option value="已結">已結</option>
                        </select>
                      </td>

                      <td class="action-cell">
                        <div class="stacked-action-container">
                          <div class="stacked-action-col">
                            <button 
                              class="cozy-btn clean-btn-noborder" 
                              style="background-color: #F3EEC3 !important; color: #5a5410 !important;" 
                              @click="fillReceiptFromOrder(ord)" 
                              title="帶入簽收單"
                            >
                              🖨️️ 簽收單
                            </button>
                            <button 
                              class="cozy-btn clean-btn-noborder" 
                              style="background-color: #DEE2FF !important; color: #28305c !important;" 
                              @click="fillFarmerReceiptFromOrder(ord)" 
                              title="帶入農民收據"
                            >
                              🧾 農民收據
                            </button>
                          </div>
                          <div class="stacked-action-col">
                            <button 
                              class="cozy-btn icon-only-btn clean-btn-noborder" 
                              style="background-color: #E0FBFC !important; color: #155e75 !important;" 
                              @click="startEditOrder(ord)" 
                              title="修改此訂單"
                            >
                              ✏️
                            </button>
                            <button 
                              class="cozy-btn icon-only-btn clean-btn-noborder" 
                              style="background-color: #ebd8da !important; color: #6e2e34 !important;" 
                              @click="deleteItem('orders', ord.id, loadOrders)" 
                              title="刪除此訂單"
                            >
                              🗑️
                            </button>
                          </div>
                        </div>
                      </td>
                    </tr>
                    <tr v-if="orderList.length === 0"><td colspan="14" class="text-center">尚無訂單資料</td></tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 2：客戶未結對帳專區 -->
          <section v-if="subTab === 'statement'" class="tab-pane">
            <div class="card-box">
              <h3>📊 客戶未結帳款彙整與對帳</h3>
              <div class="statement-filter-grid">
                <div class="field">
                  <label>選擇對帳客戶：</label>
                  <select v-model="statementCustomer">
                    <option value="">-- 請選擇客戶 (全部客戶) --</option>
                    <option v-for="c in customers" :key="c.id" :value="c.name">
                      {{ c.name }} ({{ c.type }} / {{ c.billing_cycle || '每單結' }})
                    </option>
                  </select>
                </div>
                <div class="field">
                  <label>統計期間：</label>
                  <select v-model="statementPeriod">
                    <option value="all">全部歷史紀錄</option>
                    <option value="thisWeek">本週 (週一至週日)</option>
                    <option value="thisMonth">本月 (1日至今)</option>
                    <option value="lastMonth">上月全月</option>
                    <option value="custom">🗓️ 自訂日期區間</option>
                  </select>
                </div>
                <div v-if="statementPeriod === 'custom'" class="field custom-date-range-field">
                  <label>開始日期：</label>
                  <input type="date" v-model="statementStartDate" />
                </div>
                <div v-if="statementPeriod === 'custom'" class="field custom-date-range-field">
                  <label>結束日期：</label>
                  <input type="date" v-model="statementEndDate" />
                </div>
                <div class="field">
                  <label>收款狀態篩選：</label>
                  <select v-model="statementPaymentFilter">
                    <option value="未結">僅顯示未結帳款 (對帳用)</option>
                    <option value="all">顯示全部 (含已結)</option>
                    <option value="已結">僅顯示已結帳款</option>
                  </select>
                </div>
              </div>

              <div class="statement-summary-cards mt-3">
                <div class="sum-card red-card">
                  <div class="sum-label">💰 對帳總金額</div>
                  <div class="sum-value font-heavy">${{ statementTotalAmount.toLocaleString() }} 元</div>
                </div>
                <div class="sum-card purple-card">
                  <div class="sum-label">🌸 總盆數</div>
                  <div class="sum-value font-heavy">{{ statementTotalPots }} 盆</div>
                </div>
                <div class="sum-card blue-card">
                  <div class="sum-label">📑 總訂單數</div>
                  <div class="sum-value font-heavy">{{ statementOrders.length }} 筆</div>
                </div>
                <div class="sum-card green-card">
                  <div class="sum-label">👤 客戶週期 / 類別</div>
                  <div class="sum-value font-medium">{{ currentCustomerInfoText }}</div>
                </div>
              </div>

              <div class="statement-actions mt-3">
                <button class="line-btn" @click="copyLineStatement">
                  📋 一鍵複製 LINE 對帳明細
                </button>
                <button class="excel-btn" @click="exportStatementExcel">
                  📊 下載客戶對帳單 Excel
                </button>
                <button 
                  v-if="statementOrders.length > 0 && statementPaymentFilter === '未結'"
                  class="batch-pay-btn" 
                  @click="batchMarkPaid"
                >
                  ✅ 一鍵將此清單標記為「已結清」
                </button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📑 對帳單訂單明細 ({{ statementOrders.length }} 筆)</h3>
              <div class="table-responsive">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>單號</th><th>下單日</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>總盆數</th><th>規格明細</th><th>金額</th><th>花卡狀態</th><th>簽收單狀態</th><th>收款狀態</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="ord in statementOrders" :key="ord.id">
                      <td><b>{{ ord.id }}</b></td>
                      <td>{{ ord.order_date }}</td>
                      <td><b>{{ ord.customer }}</b></td>
                      <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                      <td>{{ ord.need_receipt || '不需收據' }}</td>
                      <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                      <td class="spec-cell-wrap">{{ ord.spec }}</td>
                      <td class="text-blue font-heavy">${{ ord.price }}</td>
                      <td class="nowrap-cell">
                        <span 
                          :style="ord.card_status === '未製作' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}"
                          :class="getCardStatusClass(ord.card_status)" 
                          class="uniform-status-badge"
                        >
                          {{ ord.card_status || '未製作' }}
                        </span>
                      </td>
                      <td class="nowrap-cell">
                        <span 
                          :style="ord.receipt_status === '未列印' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}"
                          :class="ord.receipt_status === '已列印' ? 'badge badge-soft-green' : 'badge'"
                          class="uniform-status-badge"
                        >
                          {{ ord.receipt_status || '未列印' }}
                        </span>
                      </td>
                      <td class="nowrap-cell">
                        <select 
                          v-model="ord.payment_status" 
                          :class="ord.payment_status === '已結' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'payment_status', ord.payment_status)"
                          class="uniform-status-select"
                        >
                          <option value="未結">未結</option>
                          <option value="已結">已結</option>
                        </select>
                      </td>
                      <td>
                        <div class="stacked-action-col">
                          <button 
                            class="cozy-btn clean-btn-noborder" 
                            style="background-color: #F3EEC3 !important; color: #5a5410 !important;" 
                            @click="fillReceiptFromOrder(ord)"
                          >
                            🖨️ 簽收單
                          </button>
                          <button 
                            class="cozy-btn clean-btn-noborder" 
                            style="background-color: #DEE2FF !important; color: #28305c !important;" 
                            @click="fillFarmerReceiptFromOrder(ord)"
                          >
                            🧾 農民收據
                          </button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 3：進貨與庫存 -->
          <section v-if="subTab === 'inventory'" class="tab-pane">
            <div class="card-box" id="inv-form-box">
              <h3>📦 進貨與庫存清單 ({{ inventoryList.length }} 筆)</h3>
              <div class="table-responsive mt-2">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>編號</th><th>類別</th><th>品項名稱</th><th>規格</th><th>數量</th><th>單價</th><th>總成本</th><th>供應商</th><th>日期</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="inv in inventoryList" :key="inv.id">
                      <td><b>{{ inv.id }}</b></td>
                      <td><span class="badge">{{ inv.category }}</span></td>
                      <td><b>{{ inv.item_name }}</b></td>
                      <td>{{ inv.spec }}</td>
                      <td><b>{{ inv.qty }} {{ inv.category === '陶瓷盆' ? '個' : '棵' }}</b></td>
                      <td class="text-blue">${{ inv.unit_cost || (inv.qty ? Math.round(inv.cost / inv.qty) : 0) }}</td>
                      <td class="text-red"><b>${{ inv.cost }}</b></td>
                      <td>{{ inv.supplier }}</td>
                      <td>{{ inv.date }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditInv(inv)">✏️</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('inventory', inv.id, loadInventory)">🗑️</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 4：客戶資料庫 -->
          <section v-if="subTab === 'customer'" class="tab-pane">
            <div class="card-box" id="cust-form-box">
              <h3>👥 客戶名冊資料庫 ({{ customers.length }} 位)</h3>
              <div class="table-responsive mt-2">
                <table class="data-table">
                  <thead>
                    <tr><th>編號</th><th>名稱</th><th>類別</th><th>週期</th><th>電話</th><th>地址/備註</th><th>操作</th></tr>
                  </thead>
                  <tbody>
                    <tr v-for="c in customers" :key="c.id">
                      <td><b>{{ c.id }}</b></td>
                      <td><b>{{ c.name }}</b></td>
                      <td><span class="badge">{{ c.type }}</span></td>
                      <td><span class="badge badge-purple">{{ c.billing_cycle || '每單結' }}</span></td>
                      <td>{{ c.phone }}</td>
                      <td>{{ c.line_note }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditCust(c)">✏️</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('customers', c.id, loadCustomers)">🗑️</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 5：蘭花品種庫 -->
          <section v-if="subTab === 'orchid'" class="tab-pane">
            <div class="card-box" id="orchid-form-box">
              <h3>🌸 蘭花品種清單 ({{ orchids.length }} 筆)</h3>
              <div class="table-responsive mt-2">
                <table class="data-table">
                  <thead><tr><th>編號</th><th>花照</th><th>名稱</th><th>特色說明</th><th>操作</th></tr></thead>
                  <tbody>
                    <tr v-for="item in orchids" :key="item.id">
                      <td><b>{{ item.id }}</b></td>
                      <td class="photo-col">
                        <img v-if="item.photo_url" :src="item.photo_url" class="table-orchid-img" @click="openLargePhoto(item.photo_url, item.name)" />
                        <span v-else class="no-photo-badge">無照片</span>
                      </td>
                      <td><b>{{ item.name }}</b></td>
                      <td>{{ item.note }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditOrchid(item)">✏️</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('orchids', item.id, loadOrchids)">🗑️️</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 6：退貨管理區 -->
          <section v-if="subTab === 'return'" class="tab-pane">
            <div class="card-box" id="return-form-box">
              <h3>🔄 退貨紀錄清單 ({{ returnList.length }} 筆)</h3>
              <div class="table-responsive mt-2">
                <table class="data-table">
                  <thead><tr><th>退貨單號</th><th>類型</th><th>對象</th><th>品項</th><th>株數</th><th>總額</th><th>原因</th><th>操作</th></tr></thead>
                  <tbody>
                    <tr v-for="ret in returnList" :key="ret.id">
                      <td><b>{{ ret.id }}</b></td><td>{{ ret.return_type }}</td><td><b>{{ ret.party_name }}</b></td><td>{{ ret.target_item }}</td><td>{{ ret.qty }}</td><td class="text-red"><b>${{ ret.total_amount }}</b></td><td>{{ ret.reason }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditRet(ret)">✏️</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('returns', ret.id, loadReturns)">🗑️</button>
                        </div>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </section>

          <!-- 模組 7：出貨派送進度分頁 -->
          <section v-if="subTab === 'shipping'" class="tab-pane">
            <div class="card-box">
              <div class="shipping-tab-header">
                <h3>🚚 訂單出貨與派送進度總覽</h3>
                <div class="modern-pill-tabs">
                  <button :class="{ active: shippingViewFilter === 'unshipped' }" @click="shippingViewFilter = 'unshipped'" class="pill-tab-item">📦 待出貨 / 配送中 ({{ unshippedOrders.length }})</button>
                  <button :class="{ active: shippingViewFilter === 'shipped' }" @click="shippingViewFilter = 'shipped'" class="pill-tab-item">✅ 已出貨歷史 ({{ shippedOrders.length }})</button>
                  <button :class="{ active: shippingViewFilter === 'all' }" @click="shippingViewFilter = 'all'" class="pill-tab-item">全部 ({{ orderList.length }})</button>
                </div>
              </div>

              <div class="table-responsive mt-3">
                <table class="data-table">
                  <thead>
                    <tr><th>單號</th><th>預計送達日</th><th>客戶名稱</th><th>電話</th><th>送達地址/備註</th><th>總盆數</th><th>規格</th><th>花卡</th><th>簽收單</th><th>派送狀態</th><th>操作</th></tr>
                  </thead>
                  <tbody>
                    <tr v-for="ord in displayedShippingOrders" :key="ord.id">
                      <td><b>{{ ord.id }}</b></td>
                      <td class="text-blue font-bold">{{ ord.expected_date }}</td>
                      <td><b>{{ ord.customer }}</b></td>
                      <td>{{ ord.phone }}</td>
                      <td class="addr-cell">{{ getCustomerAddress(ord) }}</td>
                      <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                      <td>{{ ord.spec }}</td>
                      <td class="nowrap-cell">
                        <span :style="ord.card_status === '未製作' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}" :class="getCardStatusClass(ord.card_status)" class="uniform-status-badge">{{ ord.card_status || '未製作' }}</span>
                      </td>
                      <td class="nowrap-cell">
                        <span :style="ord.receipt_status === '未列印' ? { backgroundColor: '#F9F0F0 !important', color: '#D96B66 !important', border: '1px solid #F0D5D5 !important' } : {}" :class="ord.receipt_status === '已列印' ? 'badge badge-soft-green' : 'badge'" class="uniform-status-badge">{{ ord.receipt_status || '未列印' }}</span>
                      </td>
                      <td class="nowrap-cell">
                        <select v-model="ord.shipped_status" :class="ord.shipped_status === '已出貨' ? 'badge badge-green' : 'badge badge-red'" @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)" class="uniform-status-select">
                          <option value="未出貨">未出貨</option><option value="已出貨">已出貨</option>
                        </select>
                      </td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn clean-btn-noborder" style="background-color: #F3EEC3 !important; color: #5a5410 !important;" @click="fillReceiptFromOrder(ord)">🖨️ 簽收單</button>
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

      <!-- ================= 模式 2：花卡 / 輓聯編輯器 (A4/A5 自動自適應印表機) ================= -->
      <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
        <div class="control-panel no-print">
          <h2>⚙️ 卡片與題詞設定</h2>

          <!-- A4 / A5 尺寸切換 -->
          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: cardPaperSize === 'A4' }" @click="switchPaperSize('A4')">A4 (大尺寸)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A5' }" @click="switchPaperSize('A5')">A5 (小尺寸花卡)</button>
            </div>
          </div>

          <!-- 雲端草稿管理 -->
          <div class="panel-section draft-manage-panel">
            <div class="section-title-with-weight">
              <span class="section-title">☁️ 花卡全裝置雲端草稿庫：</span>
              <button type="button" class="mini-refresh-btn" @click="loadCloudDrafts">🔄 刷新</button>
            </div>
            <div class="draft-action-btns">
              <button type="button" class="ultra-light-purple-btn" @click="saveCurrentAsCloudDraft">💾 存至雲端草稿 (全裝置同步)</button>
              <button type="button" class="mint-new-card-btn" @click="startNewCard">＋ 開新花卡</button>
            </div>
            <div class="mt-2">
              <label class="sub-lbl">跨電腦/手機讀取暫存草稿 ({{ cloudDrafts.length }} 張)：</label>
              <div class="draft-selector-row">
                <select v-model="selectedDraftId" @change="loadCloudDraft(selectedDraftId)" class="full-input bold-select">
                  <option value="">-- 請選擇要調出的雲端草稿 --</option>
                  <option v-for="d in cloudDrafts" :key="d.id" :value="d.id">{{ d.title }}</option>
                </select>
                <button v-if="selectedDraftId" type="button" class="mini-del-draft-btn" @click="deleteCloudDraft(selectedDraftId)">🗑️</button>
              </div>
            </div>
          </div>

          <!-- 字體與粗細同一行 -->
          <div class="panel-section">
            <div class="inline-font-weight-row">
              <div class="inline-item-flex">
                <label class="mini-field-lbl">字體選擇：</label>
                <select v-model="cardFontFamily" class="full-input compact-inline-select">
                  <option value="kai">標準楷書 (TW-Kai / 書法正楷)</option>
                  <option value="notosong">思源宋體 (Noto Serif TC / 古典明體)</option>
                  <option value="fangsong">仿宋古典體 (FangSong / 秀麗骨風)</option>
                  <option value="notosans">思源黑體 (Noto Sans TC / 現代簡約)</option>
                  <option value="systemkai">系統原生楷體 (BiauKai / KaiTi)</option>
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

          <!-- 上款設定 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">1. 開頭敬詞（獨立一格）：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_prefix.size" min="14" max="250" class="compact-size-input" />
                </div>
                <select v-model="weights.upper_prefix" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
            <div class="form-row mt-1">
              <input type="text" v-model="upperPrefix" class="full-input" placeholder="例: 敬悼 或 恭祝" />
              <select v-if="cardCategory === 'funeral'" v-model="upperPrefix" style="width: 110px;">
                <option value="敬悼">敬悼</option><option value="痛悼">痛悼</option><option value="敬唁">敬唁</option><option value="追悼">追悼</option>
              </select>
              <select v-else v-model="upperPrefix" style="width: 110px;">
                <option value="祝">祝</option><option value="恭祝">恭祝</option><option value="恭賀">恭賀</option><option value="敬賀">敬賀</option>
              </select>
            </div>

            <div class="section-title-with-weight mt-2">
              <span class="section-title">2. 受禮對象 / 稱謂（獨立一格）：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_target.size" min="14" max="250" class="compact-size-input" />
                </div>
                <select v-model="weights.upper_target" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
            <template v-if="cardCategory === 'funeral'">
              <div class="form-row mt-1">
                <select v-model="funeralUpperFormat" @change="onFuneralFormatChange">
                  <option value="X媽X老夫人">X媽X老夫人 (填兩字/姓)</option><option value="X媽X夫人">X媽X夫人 (填兩字/姓)</option>
                  <option value="X公X老先生">X公X老先生 (填兩字/名)</option><option value="X公X先生">X公X先生 (填兩字/名)</option>
                  <option value="X女士">X女士</option><option value="X先生">X先生</option><option value="custom">自訂直接輸入</option>
                </select>
              </div>
              <div v-if="isDoubleXFormat" class="double-x-row mt-1">
                <div class="x-input-group"><span class="x-badge">前X</span><input type="text" v-model="targetX1" placeholder="本姓/夫姓" @input="combineTargetX" /></div>
                <span class="x-connector">＋</span>
                <div class="x-input-group"><span class="x-badge">後X</span><input type="text" v-model="targetX2" placeholder="本姓/名字" @input="combineTargetX" /></div>
              </div>
            </template>
            <input type="text" v-model="upperTarget" class="full-input mt-1" placeholder="受禮人或逝者姓名稱謂 (可直接修改結果)" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">3. 上款結尾詞（選填）：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_suffix.size" min="14" max="250" class="compact-size-input" />
                </div>
                <select v-model="weights.upper_suffix" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
            <div class="form-row mt-1">
              <input type="text" v-model="upperSuffix" class="full-input" placeholder="留空則不顯示" />
              <select v-if="cardCategory === 'funeral'" v-model="upperSuffix" style="width: 110px;">
                <option value="千古">千古</option><option value="仙逝">仙逝</option><option value="靈前">靈前</option><option value="冥前">冥前</option><option value="">(留空)</option>
              </select>
              <select v-else v-model="upperSuffix" style="width: 110px;">
                <option value="誌慶">誌慶</option><option value="大吉">大吉</option><option value="惠存">惠存</option><option value="">(留空)</option>
              </select>
            </div>
          </div>

          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">中款第 1 行（主要題詞）：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.middle.size" min="14" max="300" class="compact-size-input" />
                </div>
                <select v-model="weights.middle" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>

            <div class="form-group mt-1 mb-2">
              <label class="sub-label-tip">▼ 點選往下拉快速挑選：</label>
              <select v-model="middleText" class="full-input bold-select-dropdown">
                <option value="">-- 請下拉選擇主要題詞 --</option>
                <option v-for="phrase in availableMiddlePhrases" :key="phrase" :value="phrase">{{ phrase }}</option>
              </select>
            </div>

            <template v-if="cardCategory === 'funeral'">
              <div class="radio-row mb-1">
                <label><input type="radio" value="female" v-model="gender" /> 女性</label>
                <label><input type="radio" value="male" v-model="gender" /> 男性</label>
              </div>
              <div class="tags-container mb-2">
                <button type="button" v-for="phrase in currentFuneralPhrases" :key="phrase" class="tag-btn" @click="middleText = phrase">{{ phrase }}</button>
              </div>
            </template>
            <template v-else>
              <div class="tags-container mb-2">
                <button type="button" v-for="phrase in currentCelebPhrases" :key="phrase" class="tag-btn" @click="middleText = phrase">{{ phrase }}</button>
              </div>
            </template>

            <input type="text" v-model="middleText" class="full-input mt-1" placeholder="中款第 1 行詞語" />

            <div class="section-title-with-weight mt-3">
              <span class="section-title">中款第 2 行（選填）：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.middle_2.size" min="14" max="300" class="compact-size-input" />
                </div>
                <select v-model="weights.middle_2" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
            <input type="text" v-model="middleText2" class="full-input mt-1" placeholder="留空則不顯示第 2 行" />
          </div>

          <div class="panel-section">
            <label class="section-title">下款設定：</label>
            <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
              <span class="line-num">格 {{ idx + 1 }}</span>
              <input type="text" v-model="item.text" :placeholder="getPlaceholder(idx)" class="flex-input" />
              <div class="compact-size-wrap">
                <input type="number" v-model.number="layout['bottom_' + idx].size" min="14" max="150" class="compact-size-input" />
              </div>
              <select v-model="weights['bottom_' + idx]" class="mini-weight-select">
                <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
              </select>
            </div>

            <div class="section-title-with-weight mt-2">
              <span class="section-title">結尾敬詞：</span>
              <div class="ctrl-row-right">
                <div class="compact-size-wrap">
                  <span class="compact-size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.suffix.size" min="14" max="200" class="compact-size-input" />
                </div>
                <select v-model="weights.suffix" class="mini-weight-select">
                  <option value="400">400</option><option value="500">500</option><option value="550">550</option><option value="600">600</option><option value="650">650</option><option value="700">700</option><option value="800">800</option>
                </select>
              </div>
            </div>
            <select v-model="suffixText" class="full-input mt-1">
              <option value="敬輓">敬輓</option><option value="泣輓">泣輓</option><option value="拜輓">拜輓</option>
              <option value="敬賀">敬賀</option><option value="恭賀">恭賀</option><option value="拜賀">拜賀</option><option value="謹致">謹致</option>
            </select>
          </div>

          <button type="button" class="reset-btn" @click="resetPositions">↺ 重設排版預設位置</button>
          <button type="button" class="line-action-btn mt-2" @click="shareCoupletDirect">💬 直接傳送 / 複製花卡給客人 (免下載)</button>
          <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印花卡 / 輓聯 ({{ cardPaperSize }})</button>
        </div>

        <div class="canvas-viewport" ref="viewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.25, +(zoomLevel - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
            <button type="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
          </div>

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
                <span v-for="(token, tIdx) in parsedUpperTargetTokens" :key="tIdx" :style="token.isSmall ? { fontSize: maFontSize + 'px' } : {}">{{ token.char }}</span>
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

      <!-- ================= 模式 3：A5 橫式簽收單 (🌟 100% 原始漂亮比例) ================= -->
      <div v-else-if="currentTab === 'receipt'" class="receipt-container">
        <div class="control-panel no-print">
          <h2>📋 橫式 A5 簽收單管理</h2>

          <div class="panel-section highlight-panel">
            <label class="section-title">依訂單編號快速帶入：</label>
            <select v-model="selectedOrderId" @change="onSelectReceiptOrder" class="full-input bold-select">
              <option value="">-- 請下拉選擇訂單 (即時自動帶入) --</option>
              <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
                【{{ ord.id }}】{{ ord.customer }} - {{ formatSimpleItemName(ord) }} [{{ ord.receipt_status || '未列印' }}]
              </option>
            </select>
          </div>

          <div class="panel-section modern-sign-card">
            <div class="modern-sign-header">
              <span class="modern-sign-title">✍️ 收件人現場手寫簽名</span>
              <button type="button" class="modern-clean-sign-btn" @click="clearLiveSignature" title="清除目前的簽名並重簽">
                ↺ 清除重簽
              </button>
            </div>
            <div class="modern-canvas-wrapper">
              <canvas 
                ref="signPadCanvasRef" 
                class="modern-live-sign-pad" 
                width="340" 
                height="110"
                @pointerdown="startSign"
                @pointermove="drawingSign"
                @pointerup="stopSign"
                @pointerleave="stopSign"
              ></canvas>
              <div v-if="!liveSignDataUrl" class="sign-watermark-hint">請在此區域手寫簽名</div>
            </div>
          </div>

          <div class="panel-section">
            <label class="section-title">抬頭花店名稱：</label>
            <div class="btn-group">
              <button 
                type="button" 
                :class="{ active: shopNameMode === 'default' }" 
                @click="shopNameMode = 'default'"
              >
                宸豐蘭藝
              </button>
              <button 
                type="button" 
                :class="{ active: shopNameMode === 'custom' }" 
                @click="shopNameMode = 'custom'"
              >
                自行輸入
              </button>
            </div>
            <input 
              v-if="shopNameMode === 'custom'" 
              type="text" 
              v-model="customShopName" 
              class="full-input mt-2" 
              placeholder="請輸入自訂花店名稱" 
            />
          </div>

          <div class="panel-section">
            <label class="section-title">✏️ 簽收單內容確認與修改：</label>
            
            <div class="form-group">
              <label>收件單位 / 聯絡人 / 電話：</label>
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
              <label>花禮品項規格 (幾盆)：</label>
              <input type="text" v-model="receiptForm.item" placeholder="例：特選蘭花 1盆、特選蘭花 2盆 (共3盆)" />
            </div>

            <div class="form-group">
              <label>致贈單位 / 祝賀詞：</label>
              <input type="text" v-model="receiptForm.giver" />
            </div>

            <div class="form-group">
              <label>備註說明：</label>
              <textarea v-model="receiptForm.notes" rows="2"></textarea>
            </div>
          </div>

          <button 
            type="button" 
            class="line-action-btn mt-2" 
            @click="shareReceiptDirect"
          >
            📤 簽好直接傳送給「下單訂購人」(LINE/下載/複製)
          </button>

          <button 
            type="button" 
            class="print-action-btn mt-2" 
            :disabled="!receiptForm.recipient && !selectedOrderId"
            @click="printReceiptAndMarkDone"
          >
            🖨️ 列印 A5 橫式簽收單 (自動標記已列印)
          </button>
        </div>

        <div class="receipt-preview-area" ref="receiptViewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="receiptZoom = Math.max(0.3, +(receiptZoom - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(receiptZoom * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="receiptZoom = Math.min(1.1, +(receiptZoom + 0.05).toFixed(2))">＋</button>
            <button type="fit-btn" @click="autoFitReceipt">📱 適配螢幕</button>
          </div>

          <!-- 🌟 保留最完美的縮放容器，螢幕外觀絕不變形 -->
          <div 
            class="receipt-scaler-container" 
            :style="{
              width: (794 * receiptZoom) + 'px',
              height: (560 * receiptZoom) + 'px'
            }"
          >
            <div 
              id="receipt-print-target"
              class="a5-landscape-sheet kai-font-supported"
              :style="{
                transform: `scale(${receiptZoom})`,
                transformOrigin: 'top left'
              }"
            >
              <div class="sheet-header">
                <div class="shop-name-title">{{ displayShopName }}</div>
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
                    <td class="val">{{ receiptForm.address || '同訂購人地址 / 門市取貨' }}</td>
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
                  <div class="sign-box-area">
                    <img v-if="liveSignDataUrl" :src="liveSignDataUrl" class="live-signature-img" alt="收件人真實簽名" />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 4：農民收據 (🌟 100% 原始漂亮手刻格線版面) ================= -->
      <div v-else-if="currentTab === 'farmer_receipt'" class="receipt-container">
        <div class="control-panel no-print">
          <h2>🧾 農民出售農產品收據管理</h2>

          <div class="panel-section stamp-select-panel">
            <label class="section-title">🔴 蔡鎮遠印章（全裝置雲端同步）：</label>
            <input type="file" id="local-seal-picker" accept="image/*" style="display:none" @change="onSelectLocalSeal" />
            <button type="button" class="seal-choose-btn" @click="triggerLocalSealPicker">
              📁 上傳蔡鎮遠印章圖檔 (電腦/手機/平板全部同步)
            </button>
          </div>

          <div class="panel-section highlight-panel">
            <label class="section-title">依訂單編號自動帶入收據：</label>
            <select v-model="selectedFarmerOrderId" @change="onSelectFarmerReceiptOrder" class="full-input bold-select">
              <option value="">-- 請下拉選擇訂單 (即時自動解析) --</option>
              <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
                【{{ ord.id }}】{{ ord.customer }} - ${{ ord.price }} (統編: {{ ord.tax_id || '無' }})
              </option>
            </select>
          </div>

          <div class="panel-section">
            <label class="section-title">✏️ 收據內容確認與自由修改：</label>
            <div class="form-row">
              <div class="field">
                <label>民國年</label>
                <input type="text" v-model="farmerReceipt.year" />
              </div>
              <div class="field">
                <label>月</label>
                <input type="text" v-model="farmerReceipt.month" />
              </div>
              <div class="field">
                <label>日</label>
                <input type="text" v-model="farmerReceipt.day" />
              </div>
            </div>

            <div class="form-group mt-2">
              <label>購貨商號名稱：</label>
              <input type="text" v-model="farmerReceipt.buyerName" />
            </div>

            <div class="form-group">
              <label>統一編號：</label>
              <input type="text" v-model="farmerReceipt.taxId" maxlength="8" />
            </div>

            <div class="form-group">
              <label>住址 (整大格顯示)：</label>
              <input type="text" v-model="farmerReceipt.buyerAddress" />
            </div>

            <div class="form-group">
              <label>品名：</label>
              <input type="text" v-model="farmerReceipt.itemName" />
            </div>

            <div class="form-row">
              <div class="field">
                <label>規格</label>
                <input type="text" v-model="farmerReceipt.spec" />
              </div>
              <div class="field">
                <label>數量</label>
                <input type="text" v-model="farmerReceipt.qty" />
              </div>
              <div class="field">
                <label>單價</label>
                <input type="text" v-model="farmerReceipt.unitPrice" />
              </div>
            </div>

            <div class="form-group mt-2">
              <label>總金額 (元，沒填時留空)：</label>
              <input type="number" v-model.number="farmerReceipt.totalAmount" @input="updateChineseAmount" placeholder="沒填時不印金額" />
            </div>

            <div class="form-group">
              <label>備註：</label>
              <input type="text" v-model="farmerReceipt.note" />
            </div>
          </div>

          <button 
            type="button" 
            class="line-action-btn mt-2" 
            @click="shareFarmerReceiptDirect"
          >
            💬 直接傳送 / 複製收據給客人 (電腦LINE可直接貼上)
          </button>

          <button 
            type="button" 
            class="print-action-btn mt-2" 
            @click="printFarmerReceipt"
          >
            🖨️ 列印農民收據
          </button>
        </div>

        <!-- 右側預覽區 -->
        <div class="receipt-preview-area" ref="farmerReceiptViewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="farmerZoom = Math.max(0.3, +(farmerZoom - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(farmerZoom * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="farmerZoom = Math.min(1.1, +(farmerZoom + 0.05).toFixed(2))">＋</button>
            <button type="fit-btn" @click="autoFitFarmerReceipt">📱 適配螢幕</button>
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
              class="farmer-receipt-sheet kai-font-supported"
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

              <!-- 農民收據原始手刻格線 -->
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

                <!-- 蔡鎮遠姓名與印章 -->
                <div class="f-grid-row f-farmer-info-row">
                  <div class="f-grid-lbl f-w-head">農（漁、牧）民姓名</div>
                  <div class="f-grid-val f-farmer-stamp-cell f-flex-1">
                    <span class="f-farmer-name-clean">蔡鎮遠</span>
                    <img :src="activeCaiSealSrc" class="cai-real-stamp-img" alt="蔡鎮遠印章" />
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

              <div class="f-footer-note">
                <b>附註：</b>依據財政部 68.11.2 台財稅第三七六六五號函：自 68 年 11 月 16 日起，凡農民出售其本身所生產、捕獲或畜養之農林漁牧產品所出具之收據，一律免納印花稅，農民資格之鑑定標準，依農業發展條例第三條第三款及該條例施行細則第二條第一款規定係指直接操作或經營農業生產之自然人。
              </div>
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

// ==========================================
// 1. 基礎狀態變數
// ==========================================
const currentTab = ref('manage')
const cardPaperSize = ref('A4')
const isVertical = ref(false)
const zoomLevel = ref(0.7)
const viewportRef = ref(null)

const toastMessage = ref('')
let toastTimer = null
const showToast = (msg) => {
  toastMessage.value = msg
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toastMessage.value = '' }, 3500)
}

const shareModalImg = ref('')
const shareModalTitle = ref('')

const currentCardDimensions = computed(() => {
  if (cardPaperSize.value === 'A5') {
    return isVertical.value ? { w: 560, h: 794 } : { w: 794, h: 560 }
  }
  return isVertical.value ? { w: 794, h: 1123 } : { w: 1123, h: 794 }
})

// 🖨️ 列印最小邊界樣式注入 (0mm)
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

const receiptViewportRef = ref(null)
const receiptZoom = ref(1)
const autoFitReceipt = () => {
  if (!receiptViewportRef.value) return
  const availWidth = Math.max(receiptViewportRef.value.clientWidth - 28, 280)
  receiptZoom.value = Math.min(Math.max(+(availWidth / 794).toFixed(2), 0.35), 1.0)
}

const farmerReceiptViewportRef = ref(null)
const farmerZoom = ref(1)
const autoFitFarmerReceipt = () => {
  if (!farmerReceiptViewportRef.value) return
  const availWidth = Math.max(farmerReceiptViewportRef.value.clientWidth - 28, 280)
  farmerZoom.value = Math.min(Math.max(+(availWidth / 794).toFixed(2), 0.35), 1.0)
}

// ==========================================
// 2. 內部密碼驗證
// ==========================================
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
    showToast('✅ 驗證成功，歡迎使用內部管理系統！')
    nextTick(() => initSystemData())
  } else {
    authError.value = true
    inputPasscode.value = ''
  }
}

const handleLogout = () => {
  if (!confirm('確定要登出並鎖定系統嗎？')) return
  isAuthenticated.value = false
  try { localStorage.removeItem('cf_admin_auth') } catch (e) {}
}

// ==========================================
// 3. Supabase 連線與全模組資料函式
// ==========================================
const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
const supabase = createClient(supabaseUrl, supabaseKey)

const userCustomSeal = ref(localStorage.getItem('user_cai_seal_img') || '')
const activeCaiSealSrc = computed(() => userCustomSeal.value || '/cai-seal.png')

const fetchSealFromCloud = async () => {
  try {
    const { data } = await supabase.from('system_settings').select('value').eq('key', 'cai_seal_img').single()
    if (data && data.value) {
      userCustomSeal.value = data.value
      localStorage.setItem('user_cai_seal_img', data.value)
    }
  } catch (err) {}
}

const triggerLocalSealPicker = () => {
  document.getElementById('local-seal-picker')?.click()
}

const onSelectLocalSeal = async (e) => {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = async (event) => {
    const base64Data = event.target.result
    userCustomSeal.value = base64Data
    localStorage.setItem('user_cai_seal_img', base64Data)

    try {
      await supabase.from('system_settings').upsert({
        key: 'cai_seal_img',
        value: base64Data,
        updated_at: new Date()
      })
      showToast('✅ 印章已成功上傳至雲端！所有手機與平板打開均會自動同步！')
    } catch (err) {
      showToast('✅ 印章已在本機生效！')
    }
  }
  reader.readAsDataURL(file)
}

const subTab = ref('order')
const orchids = ref([])
const customers = ref([])
const inventoryList = ref([])
const returnList = ref([])
const orderList = ref([])

const editingOrderId = ref(null)
const editingInvId = ref(null)
const editingCustId = ref(null)
const editingOrchidId = ref(null)
const editingRetId = ref(null)

const activeModalPhoto = ref(null)
const activeModalTitle = ref('')
const openLargePhoto = (url, name) => {
  activeModalPhoto.value = url
  activeModalTitle.value = name
}

const formatSimpleItemName = (ord) => {
  if (!ord) return '特選蘭花 1盆'
  const specText = String(ord.spec || '')
  if (specText.includes(';')) {
    const parts = specText.split(';').map(s => s.trim()).filter(Boolean)
    if (parts.length > 0) {
      const parsedParts = parts.map(p => {
        const segs = p.split('|')
        const namePart = segs[0].trim()
        const potsMatch = namePart.match(/(\d+)\s*盆/)
        const pots = potsMatch ? `${potsMatch[1]}盆` : '1盆'
        const cleanName = namePart.replace(/\(\d+盆\)/g, '').replace(/\(.*?\)/g, '').trim() || '特選蘭花'
        return `${cleanName} ${pots}`
      })
      const totalPots = getOrderTotalPots(ord)
      return `${parsedParts.join('、')}（共${totalPots}盆）`
    }
  }
  const rawFirst = specText.split('|')[0].trim()
  const cleanName = rawFirst.replace(/\(.*?\)/g, '').trim() || '特選蘭花'
  const totalPots = getOrderTotalPots(ord)
  return `${cleanName} ${totalPots}盆`
}

const getOrderTotalPots = (ord) => {
  if (!ord) return 1
  const specText = String(ord.spec || '')
  const matches = [...specText.matchAll(/\((\d+)\s*盆\)/g)]
  if (matches.length > 0) {
    const sum = matches.reduce((total, m) => total + (parseInt(m[1]) || 0), 0)
    if (sum > 0) return sum
  }
  const noteText = String(ord.note || '')
  const metaMatch = noteText.match(/\[共(\d+)\s*盆/)
  if (metaMatch) return parseInt(metaMatch[1]) || 1
  return 1
}

const getOrderShippingFee = (ord) => {
  if (!ord) return 0
  const searchStr = `${ord.note || ''}`
  const shipMatch = searchStr.match(/運費\s*[:：]?\s*(\d+)/)
  return shipMatch ? parseInt(shipMatch[1]) : 0
}

const generateDateSeqIdByDate = (prefix, dateStrVal, existingList) => {
  let datePart = ''
  if (dateStrVal) {
    datePart = String(dateStrVal).replace(/-/g, '')
  } else {
    const now = new Date()
    const y = now.getFullYear()
    const m = String(now.getMonth() + 1).padStart(2, '0')
    const d = String(now.getDate()).padStart(2, '0')
    datePart = `${y}${m}${d}`
  }
  const targetPrefix = `${prefix}-${datePart}-`
  const sameDateItems = (existingList || []).filter(item => String(item.id || '').startsWith(targetPrefix))

  let maxSeq = 0
  sameDateItems.forEach(item => {
    const seqStr = String(item.id).replace(targetPrefix, '')
    const seqNum = parseInt(seqStr, 10)
    if (!isNaN(seqNum) && seqNum > maxSeq) maxSeq = seqNum
  })
  return `${targetPrefix}${String(maxSeq + 1).padStart(2, '0')}`
}

const formOrchid = ref({ name: '', note: '標準優良品種', photo_url: '' })
const formCust = ref({ name: '', type: '批發商', billing_cycle: '每單結', phone: '0912-345678', line_note: '' })
const formInv = ref({
  category: '蘭花', item_name: '', spec_spike: '單梗', spec_color: '紅', spec_size: '大', spec_height: '中',
  pot_type: '桌上盆 (100)', qty: 10, unit_cost: 100, cost: 1000, supplier: '某某花農', date: new Date().toISOString().split('T')[0]
})
const formRet = ref({
  return_type: '退給花農', party_name: '', target_item: '', qty: 2, unit_price: 150, total_amount: 300,
  date: new Date().toISOString().split('T')[0], reason: '運送碰撞 / 開花不良'
})
const formOrder = ref({
  cust_type: '批發', customer: '', billing_cycle: '每單結', phone: '0912-345678', shipping_fee: 0,
  cost: 600, price: 2500, tax_id: '', need_receipt: '不需收據', note: '', card_status: '未製作',
  receipt_status: '未列印', shipped_status: '未出貨', payment_status: '未結',
  order_date: new Date().toISOString().split('T')[0],
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0],
  items: [{ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' }]
})

const previewNextOrderId = computed(() => generateDateSeqIdByDate('OR', formOrder.value.order_date, orderList.value))
const flowerInventory = computed(() => inventoryList.value.filter(i => i.category === '蘭花'))

const loadOrders = async () => {
  try {
    const { data } = await supabase.from('orders').select('*')
    if (data) orderList.value = data.sort((a, b) => String(b.id || '').localeCompare(String(a.id || '')))
  } catch (err) {}
}
const loadInventory = async () => {
  const { data } = await supabase.from('inventory').select('*').order('created_at', { ascending: false })
  if (data) inventoryList.value = data
}
const loadCustomers = async () => {
  const { data } = await supabase.from('customers').select('*').order('created_at', { ascending: false })
  if (data) customers.value = data
}
const loadOrchids = async () => {
  const { data } = await supabase.from('orchids').select('*').order('created_at', { ascending: false })
  if (data) orchids.value = data
}
const loadReturns = async () => {
  const { data } = await supabase.from('returns').select('*').order('created_at', { ascending: false })
  if (data) returnList.value = data
}

const onFlowerSelectChange = (item) => {
  if (!item.orchid_name) return
  const matchedInv = inventoryList.value.find(i => i.category === '蘭花' && i.item_name === item.orchid_name)
  if (matchedInv) {
    const refCost = matchedInv.unit_cost || (matchedInv.qty ? Math.round(matchedInv.cost / matchedInv.qty) : 0)
    if (refCost > 0 && item.unit_price <= 0) item.unit_price = Math.round(refCost * 2)
  }
  calcOrderPrice()
}

const addOrderItemRow = () => {
  formOrder.value.items.push({ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' })
  calcOrderPrice()
}
const removeOrderItemRow = (idx) => {
  if (formOrder.value.items.length > 1) {
    formOrder.value.items.splice(idx, 1)
    calcOrderPrice()
  }
}

const calcOrderPrice = () => {
  let totalPrice = 0
  let totalCost = 0
  const potCostMap = { '桌上盆 (100)': 100, '落地盆陶瓷-喪 (100)': 100, '落地陶瓷盆-喜 (200)': 200, '羅馬盆 (280)': 280, '無盆': 0 }
  formOrder.value.items.forEach(item => {
    const p = Number(item.pots_qty) || 1
    const s = Number(item.stalks) || 0
    const u = Number(item.unit_price) || 0
    totalPrice += (s * u) * p
    const potCost = potCostMap[item.pot] || 0
    const quickCost = item.quick_pot === '使用快捷盆 (70)' ? 70 : 0
    totalCost += (s * 60 + potCost + quickCost) * p
  })
  formOrder.value.price = totalPrice + (Number(formOrder.value.shipping_fee) || 0)
  formOrder.value.cost = totalCost
}

const onOrderCustSelect = () => {
  const matched = customers.value.find(c => c.name === formOrder.value.customer)
  if (matched) {
    formOrder.value.phone = matched.phone || ''
    formOrder.value.cust_type = matched.type === '批發商' ? '批發' : matched.type
    formOrder.value.billing_cycle = matched.billing_cycle || '每單結'
  }
}

const startEditOrder = (ord) => {
  editingOrderId.value = ord.id
  const parsedItems = []
  const specText = String(ord.spec || '')
  const parts = specText.includes(';') ? specText.split(';').map(s => s.trim()).filter(Boolean) : [specText]
  parts.forEach(p => {
    const stalksMatch = p.match(/(\d+)\s*棵/)
    const unitMatch = p.match(/單價(\d+)元/)
    const potsMatch = p.match(/\((\d+)\s*盆\)/)
    const nameMatch = p.split('(')[0].trim() || '特選蘭花'
    parsedItems.push({
      orchid_name: nameMatch,
      pots_qty: potsMatch ? parseInt(potsMatch[1]) : 1,
      stalks: stalksMatch ? parseInt(stalksMatch[1]) : 10,
      unit_price: unitMatch ? parseInt(unitMatch[1]) : 250,
      pot: (ord.pot || '桌上盆 (100)').replace(/\s*\+\s*快捷盆/, '').trim(),
      quick_pot: (ord.pot || '').includes('快捷盆') ? '使用快捷盆 (70)' : '未使用'
    })
  })
  if (parsedItems.length === 0) parsedItems.push({ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' })
  const cleanNote = String(ord.note || '').replace(/\[共\d+盆,\s*運費:\d+元\]/g, '').trim()
  formOrder.value = {
    cust_type: ord.cust_type || '批發', customer: ord.customer || '', billing_cycle: ord.billing_cycle || '每單結',
    phone: ord.phone || '', shipping_fee: getOrderShippingFee(ord), cost: Number(ord.cost) || 0, price: Number(ord.price) || 0,
    tax_id: ord.tax_id || '', need_receipt: ord.need_receipt || '不需收據', note: cleanNote, card_status: ord.card_status || '未製作',
    receipt_status: ord.receipt_status || '未列印', shipped_status: ord.shipped_status || '未出貨', payment_status: ord.payment_status || '未結',
    order_date: ord.order_date, expected_date: ord.expected_date, items: parsedItems
  }
  subTab.value = 'order'
  nextTick(() => document.getElementById('order-form-box')?.scrollIntoView({ behavior: 'smooth' }))
}

const cancelEditOrder = () => {
  editingOrderId.value = null
  formOrder.value = {
    cust_type: '批發', customer: '', billing_cycle: '每單結', phone: '0912-345678', shipping_fee: 0, cost: 600, price: 2500,
    tax_id: '', need_receipt: '不需收據', note: '', card_status: '未製作', receipt_status: '未列印', shipped_status: '未出貨', payment_status: '未結',
    order_date: new Date().toISOString().split('T')[0], expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0],
    items: [{ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' }]
  }
}

const saveOrder = async () => {
  if (!formOrder.value.customer) return alert('請輸入客戶名稱！')
  const specParts = formOrder.value.items.map(item => `${item.orchid_name || '特選蘭花'} (${item.pots_qty || 1}盆) | ${item.stalks || 1}棵 (單價${item.unit_price || 0}元)`)
  const fullSpecStr = specParts.join('; ')
  const mainItem = formOrder.value.items[0] || {}
  const finalPotStr = (mainItem.pot || '桌上盆 (100)') + (mainItem.quick_pot === '使用快捷盆 (70)' ? ' + 快捷盆' : '')
  const baseNote = String(formOrder.value.note || '').replace(/\[共\d+盆,\s*運費:\d+元\]/g, '').trim()
  const totalPots = formOrder.value.items.reduce((sum, it) => sum + (Number(it.pots_qty) || 1), 0)
  const metaTag = `[共${totalPots}盆, 運費:${formOrder.value.shipping_fee || 0}元]`
  const finalNote = baseNote ? `${baseNote} ${metaTag}` : metaTag

  const payload = {
    cust_type: formOrder.value.cust_type, customer: formOrder.value.customer, billing_cycle: formOrder.value.billing_cycle,
    phone: formOrder.value.phone, spec: fullSpecStr, pot: finalPotStr, cost: formOrder.value.cost, price: formOrder.value.price,
    tax_id: formOrder.value.tax_id, need_receipt: formOrder.value.need_receipt, note: finalNote, card_status: formOrder.value.card_status,
    receipt_status: formOrder.value.receipt_status, shipped_status: formOrder.value.shipped_status, payment_status: formOrder.value.payment_status,
    order_date: formOrder.value.order_date, expected_date: formOrder.value.expected_date
  }
  if (editingOrderId.value) {
    const { error } = await supabase.from('orders').update(payload).eq('id', editingOrderId.value)
    if (!error) {
      alert(`訂單 ${editingOrderId.value} 修改成功！`)
      cancelEditOrder()
      loadOrders()
    }
  } else {
    const newId = generateDateSeqIdByDate('OR', formOrder.value.order_date, orderList.value)
    const { error } = await supabase.from('orders').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`訂單建立成功！單號：${newId}`)
      cancelEditOrder()
      loadOrders()
    }
  }
}

const updateOrderField = async (ord, field, value) => {
  ord[field] = value
  await supabase.from('orders').update({ [field]: value }).eq('id', ord.id)
  showToast(`✅ 單號 ${ord.id} 的狀態已即時更新！`)
}

const getCardStatusClass = (status) => status === '已製作' ? 'badge badge-soft-green' : (status === '免製作' ? 'badge badge-gray' : 'badge')

const shippingViewFilter = ref('unshipped')
const unshippedOrders = computed(() => orderList.value.filter(o => o.shipped_status !== '已出貨'))
const shippedOrders = computed(() => orderList.value.filter(o => o.shipped_status === '已出貨'))
const displayedShippingOrders = computed(() => shippingViewFilter.value === 'unshipped' ? unshippedOrders.value : (shippingViewFilter.value === 'shipped' ? shippedOrders.value : orderList.value))
const getCustomerAddress = (ord) => {
  const c = customers.value.find(item => item.name === ord.customer)
  return c?.line_note || ord.note || '—'
}

// 模組 2：對帳專區函式
const statementCustomer = ref('')
const statementPeriod = ref('all')
const statementStartDate = ref(new Date().toISOString().split('T')[0])
const statementEndDate = ref(new Date().toISOString().split('T')[0])
const statementPaymentFilter = ref('未結')

const statementOrders = computed(() => {
  return orderList.value.filter(o => {
    if (statementCustomer.value && o.customer !== statementCustomer.value) return false
    if (statementPaymentFilter.value !== 'all' && o.payment_status !== statementPaymentFilter.value) return false
    if (statementPeriod.value === 'all') return true
    const orderDate = new Date(o.order_date + 'T00:00:00')
    const now = new Date()
    if (statementPeriod.value === 'thisWeek') {
      const day = now.getDay() || 7
      const monday = new Date(now)
      monday.setDate(now.getDate() - day + 1)
      monday.setHours(0,0,0,0)
      const sunday = new Date(monday)
      sunday.setDate(monday.getDate() + 6)
      sunday.setHours(23,59,59,999)
      return orderDate >= monday && orderDate <= sunday
    }
    if (statementPeriod.value === 'thisMonth') {
      const firstDay = new Date(now.getFullYear(), now.getMonth(), 1)
      const lastDay = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59)
      return orderDate >= firstDay && orderDate <= lastDay
    }
    if (statementPeriod.value === 'lastMonth') {
      const firstDay = new Date(now.getFullYear(), now.getMonth() - 1, 1)
      const lastMonthEnd = new Date(now.getFullYear(), now.getMonth(), 0, 23, 59, 59)
      return orderDate >= firstDay && orderDate <= lastMonthEnd
    }
    if (statementPeriod.value === 'custom') {
      if (!statementStartDate.value || !statementEndDate.value) return true
      const sDate = new Date(statementStartDate.value + 'T00:00:00')
      const eDate = new Date(statementEndDate.value + 'T23:59:59')
      return orderDate >= sDate && orderDate <= eDate
    }
    return true
  })
})
const statementTotalAmount = computed(() => statementOrders.value.reduce((sum, o) => sum + (Number(o.price) || 0), 0))
const statementTotalPots = computed(() => statementOrders.value.reduce((sum, o) => sum + getOrderTotalPots(o), 0))
const currentCustomerInfoText = computed(() => {
  if (!statementCustomer.value) return '全部客戶統整'
  const c = customers.value.find(item => item.name === statementCustomer.value)
  return c ? `${c.type || '一般'} | ${c.billing_cycle || '每單結'}` : '一般客戶'
})

const copyLineStatement = () => {
  if (statementOrders.value.length === 0) return alert('目前無訂單可複製對帳資料！')
  let text = `🌸【宸豐蘭藝 - 對帳明細】\n客戶名稱：${statementCustomer.value || '全部客戶'}\n訂單筆數：${statementOrders.value.length} 筆\n花禮總數：${statementTotalPots.value} 盆\n對帳總額：NT$ ${statementTotalAmount.value.toLocaleString()} 元\n--------------------------------\n`
  statementOrders.value.forEach((o, index) => {
    text += `${index + 1}. [${o.order_date}] ${o.spec} ➔ $${o.price}元 (${o.payment_status})\n`
  })
  navigator.clipboard.writeText(text).then(() => showToast('✅ LINE 對帳明細已複製到剪貼簿！'))
}
const exportStatementExcel = () => {
  if (statementOrders.value.length === 0) return alert('目前無資料可匯出！')
  const worksheet = XLSX.utils.json_to_sheet(statementOrders.value)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '對帳明細')
  XLSX.writeFile(workbook, `對帳單_${new Date().toISOString().split('T')[0]}.xlsx`)
}
const batchMarkPaid = async () => {
  if (!confirm('確定標記為已結？')) return
  const ids = statementOrders.value.map(o => o.id)
  await supabase.from('orders').update({ payment_status: '已結' }).in('id', ids)
  showToast('✅ 已批次結清成功！')
  loadOrders()
}

// 模組 3：進貨函式
const startEditInv = (inv) => {
  editingInvId.value = inv.id
  formInv.value = { ...inv }
}
const deleteItem = async (table, id, reloadFn) => {
  if (!confirm(`確定要刪除編號 ${id} 嗎？`)) return
  await supabase.from(table).delete().eq('id', id)
  reloadFn()
}
const exportOrdersToExcel = () => {
  const worksheet = XLSX.utils.json_to_sheet(orderList.value)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '全部訂單')
  XLSX.writeFile(workbook, `訂單總表_${new Date().toISOString().split('T')[0]}.xlsx`)
}

// ==========================================
// 4. 花卡編輯器
// ==========================================
const cardCategory = ref('celebration')
const cardFontFamily = ref('kai')
const upperPrefix = ref('祝')
const upperTarget = ref('新北市 陳乃瑜議員')
const upperSuffix = ref('')
const middleText = ref('高票當選')
const middleText2 = ref('為民服務')
const suffixText = ref('敬賀')

const funeralUpperFormat = ref('custom')
const targetX1 = ref('')
const targetX2 = ref('')
const isDoubleXFormat = computed(() => ['X媽X老夫人', 'X媽X夫人', 'X公X老先生', 'X公X先生'].includes(funeralUpperFormat.value))

const onFuneralFormatChange = () => {
  if (funeralUpperFormat.value === 'custom') return
  if (funeralUpperFormat.value === 'X女士') upperTarget.value = '陳女士'
  else if (funeralUpperFormat.value === 'X先生') upperTarget.value = '陳先生'
  else combineTargetX()
}
const combineTargetX = () => {
  const x1 = targetX1.value.trim() || '張'
  const x2 = targetX2.value.trim() || '李'
  if (funeralUpperFormat.value === 'X媽X老夫人') upperTarget.value = `${x1}媽${x2}老夫人`
  else if (funeralUpperFormat.value === 'X媽X夫人') upperTarget.value = `${x1}媽${x2}夫人`
  else if (funeralUpperFormat.value === 'X公X老先生') upperTarget.value = `${x1}公${x2}老先生`
  else if (funeralUpperFormat.value === 'X公X先生') upperTarget.value = `${x1}公${x2}先生`
}

const celebrationType = ref('opening')
const gender = ref('female')
const ageStage = ref('f_over80')

const weights = ref({
  upper_prefix: '700', upper_target: '700', upper_suffix: '700', middle: '800', middle_2: '800',
  bottom_0: '600', bottom_1: '700', bottom_2: '600', bottom_3: '600', bottom_4: '600', bottom_5: '600', suffix: '700'
})
const getWeightStyle = (wVal) => {
  const w = String(wVal || '600')
  const styles = { fontWeight: w }
  if (w === '500') styles.textShadow = '0 0 0.4px #000'
  else if (w === '550') styles.textShadow = '0 0 0.6px #000'
  else if (w === '600') styles.textShadow = '0 0 0.8px #000'
  else if (w === '650') styles.textShadow = '0 0 1.0px #000, 0.2px 0.2px 0 #000'
  else if (w === '700') styles.textShadow = '0 0 1.2px #000, 0.3px 0.3px 0 #000'
  else if (w === '800') styles.textShadow = '0 0 1.8px #000, 0.5px 0.5px 0 #000'
  return styles
}

const fontMapping = {
  kai: '"TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "KaiTi", serif',
  notosong: '"Noto Serif TC", "Songti TC", "SimSun", "PMingLiU", serif',
  fangsong: '"FangSong", "STFangsong", "華康仿宋體", serif',
  notosans: '"Noto Sans TC", "PingFang TC", "Microsoft JhengHei", sans-serif',
  systemkai: '"標楷體", "DFKai-SB", "BiauKai", "TW-Kai", serif'
}
const activeCssFontFamily = computed(() => fontMapping[cardFontFamily.value] || fontMapping.kai)

const bottomLines = ref([{ text: '白沙屯媽祖' }, { text: '彰化拱聖宮' }, { text: '' }, { text: '' }, { text: '' }, { text: '' }])
const getPlaceholder = (idx) => ['第 1 格（例：單位 / 公司）', '第 2 格（例：職稱姓名 1）', '第 3 格（自訂聯名人 2）', '第 4 格（自訂）', '第 5 格（自訂）', '第 6 格（自訂）'][idx]

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

const celebPhrases = {
  opening: ['開幕誌慶', '開張大吉', '鴻圖大展', '駿業宏開', '生意興隆', '財源廣進', '客似雲來'],
  moving: ['喬遷之喜', '里仁為美', '金玉滿堂'],
  temple: ['聖誕千秋', '神威顯赫']
}
const currentCelebPhrases = computed(() => celebPhrases[celebrationType.value] || [])
const availableMiddlePhrases = computed(() => cardCategory.value === 'funeral' ? currentFuneralPhrases.value : currentCelebPhrases.value)

const parsedUpperTargetTokens = computed(() => upperTarget.value.split('').map(char => ({ char, isSmall: char === '媽' })))
const maFontSize = computed(() => Math.max(12, (layout.value.upper_target?.size || 40) - 20))

const defaultVertical = {
  upper_prefix: { x: 620, y: 100, size: 36 }, upper_target: { x: 620, y: 220, size: 42 }, upper_suffix: { x: 620, y: 720, size: 36 },
  middle: { x: 380, y: 220, size: 76 }, middle_2: { x: 280, y: 220, size: 76 },
  bottom_0: { x: 155, y: 480, size: 30 }, bottom_1: { x: 155, y: 640, size: 36 }, bottom_2: { x: 95, y: 480, size: 30 },
  bottom_3: { x: 95, y: 640, size: 32 }, bottom_4: { x: 40, y: 480, size: 30 }, bottom_5: { x: 40, y: 640, size: 30 },
  suffix: { x: 155, y: 860, size: 34 }
}
const defaultHorizontal = {
  upper_prefix: { x: 80, y: 80, size: 34 }, upper_target: { x: 220, y: 80, size: 38 }, upper_suffix: { x: 920, y: 80, size: 34 },
  middle: { x: 220, y: 220, size: 68 }, middle_2: { x: 220, y: 310, size: 68 },
  bottom_0: { x: 180, y: 520, size: 28 }, bottom_1: { x: 380, y: 530, size: 34 }, bottom_2: { x: 380, y: 520, size: 28 },
  bottom_3: { x: 580, y: 520, size: 28 }, bottom_4: { x: 580, y: 530, size: 28 }, bottom_5: { x: 760, y: 530, size: 28 },
  suffix: { x: 880, y: 530, size: 34 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultHorizontal)))
const switchOrientation = (vertical) => {
  isVertical.value = vertical
  layout.value = JSON.parse(JSON.stringify(vertical ? defaultVertical : defaultHorizontal))
  nextTick(() => autoFitZoom())
}
const resetPositions = () => { layout.value = JSON.parse(JSON.stringify(isVertical.value ? defaultVertical : defaultHorizontal)) }
const getStyle = (key) => {
  const item = (layout.value && layout.value[key]) ? layout.value[key] : { x: 50, y: 50, size: 30 }
  return { left: `${item.x}px`, top: `${item.y}px`, fontSize: `${item.size}px`, ...getWeightStyle(weights.value[key]) }
}
const getUpperTargetBoxStyle = () => {
  const item = (layout.value && layout.value.upper_target) ? layout.value.upper_target : { x: 220, y: 80, size: 38 }
  return { left: `${item.x}px`, top: `${item.y}px`, fontSize: `${item.size}px`, whiteSpace: 'nowrap', overflow: 'visible', zIndex: 5, ...getWeightStyle(weights.value.upper_target) }
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
    layout.value[activeKey].size = Math.max(14, Math.min(300, Math.round(originSize + (dx + dy) / 3)))
  }
}
const onPointerUp = () => {
  activeKey = null; currentAction = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
}

watch(gender, (val) => { ageStage.value = val === 'female' ? 'f_50_79' : 'm_50_69' })
const onCardCategoryChange = () => {
  if (cardCategory.value === 'funeral') {
    upperPrefix.value = '敬悼'; upperSuffix.value = '千古'; suffixText.value = '敬輓'; middleText.value = currentFuneralPhrases.value[0] || '母儀千古'; middleText2.value = ''
  } else {
    upperPrefix.value = '祝'; upperSuffix.value = ''; suffixText.value = '敬賀'; middleText.value = currentCelebPhrases.value[0] || '高票當選'; middleText2.value = '為民服務'
  }
}

const printCouplet = () => window.print()

const shareCoupletDirect = async () => {
  const targetEl = document.getElementById('card-print-target')
  if (!targetEl) return alert('找不到花卡畫面！')
  showToast('⏳ 正在生成無切字花卡高畫質圖片...')
  try {
    const originalTransform = targetEl.style.transform
    targetEl.style.transform = 'none'
    const canvas = await html2canvas(targetEl, {
      scale: 2, useCORS: true, backgroundColor: '#ffffff', logging: false,
      ignoreElements: (element) => element.classList && (element.classList.contains('scale-handle') || element.classList.contains('no-print'))
    })
    targetEl.style.transform = originalTransform
    const filename = `花卡_${cardPaperSize.value}_${new Date().toISOString().split('T')[0]}.png`
    shareOrCopyCanvasBlob(canvas, filename, `花卡確認 (${cardPaperSize.value})`, '花卡圖片準備完成')
  } catch (err) {
    showToast('⚠️ 圖片生成失敗，請重試！')
  }
}

const cloudDrafts = ref([])
const selectedDraftId = ref('')
const loadCloudDrafts = async () => {
  try {
    const { data } = await supabase.from('card_drafts').select('*').order('created_at', { ascending: false })
    if (data) cloudDrafts.value = data
  } catch (err) {}
}
const saveCurrentAsCloudDraft = async () => {
  const now = new Date()
  const timeStr = `${String(now.getMonth() + 1).padStart(2, '0')}/${String(now.getDate()).padStart(2, '0')} ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`
  const targetName = upperTarget.value.trim() || '未填對象'
  const phrase = middleText.value.trim() || '無中款'
  const draftTitle = `【${targetName}】${phrase} (${timeStr})`
  const draftId = 'draft_' + Date.now()
  await supabase.from('card_drafts').insert([{ id: draftId, title: draftTitle, data: { cardPaperSize: cardPaperSize.value, isVertical: isVertical.value, cardCategory: cardCategory.value, cardFontFamily: cardFontFamily.value, upperPrefix: upperPrefix.value, upperTarget: upperTarget.value, upperSuffix: upperSuffix.value, middleText: middleText.value, middleText2: middleText2.value, suffixText: suffixText.value, bottomLines: bottomLines.value, weights: weights.value, layout: layout.value } }])
  showToast(`☁️ 花卡已成功存至雲端！`)
  await loadCloudDrafts()
  selectedDraftId.value = draftId
}
const loadCloudDraft = (id) => {
  const item = cloudDrafts.value.find(d => d.id === id)
  if (item?.data) {
    const draft = item.data
    cardPaperSize.value = draft.cardPaperSize || 'A4'; isVertical.value = draft.isVertical; cardCategory.value = draft.cardCategory; cardFontFamily.value = draft.cardFontFamily || 'kai'
    upperPrefix.value = draft.upperPrefix || ''; upperTarget.value = draft.upperTarget || ''; upperSuffix.value = draft.upperSuffix || ''
    middleText.value = draft.middleText || ''; middleText2.value = draft.middleText2 || ''; suffixText.value = draft.suffixText || '敬輓'
    bottomLines.value = JSON.parse(JSON.stringify(draft.bottomLines)); weights.value = JSON.parse(JSON.stringify(draft.weights)); layout.value = JSON.parse(JSON.stringify(draft.layout))
    showToast(`📂 已載入「${item.title}」！`)
    nextTick(() => autoFitZoom())
  }
}
const deleteCloudDraft = async (id) => {
  if (!confirm('確定刪除草稿？')) return
  await supabase.from('card_drafts').delete().eq('id', id)
  cloudDrafts.value = cloudDrafts.value.filter(d => d.id !== id)
  selectedDraftId.value = ''
}
const startNewCard = () => {
  upperTarget.value = ''; middleText.value = cardCategory.value === 'funeral' ? '母儀千古' : '高票當選'; middleText2.value = ''; selectedDraftId.value = ''; targetX1.value = ''; targetX2.value = ''; resetPositions(); showToast('✨ 空白花卡建立完成')
}

// ==========================================
// 5. 簽收單管理 (🌟 html2canvas 截圖發送修復)
// ==========================================
const shopNameMode = ref('default')
const customShopName = ref('')
const displayShopName = computed(() => shopNameMode.value === 'default' ? '宸豐蘭藝' : (customShopName.value || '宸豐蘭藝'))

const selectedOrderId = ref('')
const receiptForm = ref({
  orderId: '', deliveryDate: '115-09-03 送達', recipient: '永全證券 陳柏榮總經理 (0912-345678)',
  address: '桃園市桃園區縣府路 82 號 1 樓', item: '特選蘭花 1盆', giver: '敬領 誌慶 / 宸豐蘭藝 敬製',
  notes: '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！'
})
const signPadCanvasRef = ref(null)
const liveSignDataUrl = ref('')
let isSigning = false, lastSignX = 0, lastSignY = 0

const getSignCoords = (e) => {
  if (!signPadCanvasRef.value) return { x: 0, y: 0 }
  const rect = signPadCanvasRef.value.getBoundingClientRect()
  return { x: (e.clientX - rect.left) * (signPadCanvasRef.value.width / rect.width), y: (e.clientY - rect.top) * (signPadCanvasRef.value.height / rect.height) }
}
const startSign = (e) => {
  isSigning = true
  const coords = getSignCoords(e); lastSignX = coords.x; lastSignY = coords.y
}
const drawingSign = (e) => {
  if (!isSigning || !signPadCanvasRef.value) return
  const ctx = signPadCanvasRef.value.getContext('2d')
  const coords = getSignCoords(e)
  ctx.lineWidth = 2.5; ctx.lineCap = 'round'; ctx.strokeStyle = '#0f172a'
  ctx.beginPath(); ctx.moveTo(lastSignX, lastSignY); ctx.lineTo(coords.x, coords.y); ctx.stroke()
  lastSignX = coords.x; lastSignY = coords.y
}
const stopSign = () => {
  if (!isSigning) return
  isSigning = false
  if (signPadCanvasRef.value) liveSignDataUrl.value = signPadCanvasRef.value.toDataURL('image/png')
}
const clearLiveSignature = () => {
  const ctx = signPadCanvasRef.value?.getContext('2d')
  if (ctx) ctx.clearRect(0, 0, 340, 110)
  liveSignDataUrl.value = ''
  showToast('已清除簽名')
}

const onSelectReceiptOrder = () => {
  const ord = orderList.value.find(o => o.id === selectedOrderId.value)
  if (ord) {
    const cust = customers.value.find(c => c.name === ord.customer)
    receiptForm.value.orderId = ord.id
    receiptForm.value.recipient = `${ord.customer} ${ord.phone ? '(' + ord.phone + ')' : ''}`
    receiptForm.value.address = cust?.line_note || '同訂購人地址 / 門市取貨'
    receiptForm.value.item = formatSimpleItemName(ord)
    receiptForm.value.deliveryDate = `${ord.expected_date} 送達`
    clearLiveSignature()
  }
}
const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  onSelectReceiptOrder()
  currentTab.value = 'receipt'
  nextTick(() => autoFitReceipt())
}
const printReceiptAndMarkDone = async () => {
  if (selectedOrderId.value) {
    const ord = orderList.value.find(o => o.id === selectedOrderId.value)
    if (ord) await updateOrderField(ord, 'receipt_status', '已列印')
  }
  window.print()
}

// 🌟 簽收單使用 html2canvas 完整截圖
const shareReceiptDirect = async () => {
  const targetEl = document.getElementById('receipt-print-target')
  if (!targetEl) return alert('找不到簽收單！')
  showToast('⏳ 正在生成簽收單圖片...')
  try {
    const origTransform = targetEl.style.transform
    targetEl.style.transform = 'none'
    const canvas = await html2canvas(targetEl, { scale: 2, backgroundColor: '#ffffff', useCORS: true, logging: false })
    targetEl.style.transform = origTransform
    const filename = `簽收單_${receiptForm.value.orderId || '現場'}_${new Date().toISOString().split('T')[0]}.png`
    shareOrCopyCanvasBlob(canvas, filename, '簽收單確認', '簽收單圖片準備完成')
  } catch (err) {
    showToast('⚠️ 圖片生成失敗，請重試！')
  }
}

// ==========================================
// 6. 農民收據
// ==========================================
const selectedFarmerOrderId = ref('')
const farmerReceipt = ref({
  year: '115', month: '09', day: '03', buyerName: '永全證券股份有限公司', taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號', itemName: '蝴蝶蘭花禮', spec: '特級蝴蝶蘭', qty: '1 盆',
  unitPrice: '2,500', totalAmount: 2500, note: ''
})
const chineseDigits = ref({ hundredThousands: '', tenThousands: '', thousands: '貳', hundreds: '伍', tens: '', ones: '' })
const digitMap = ['', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖']

const updateChineseAmount = () => {
  const amt = Math.floor(Number(farmerReceipt.value.totalAmount) || 0)
  if (amt <= 0) {
    chineseDigits.value = { hundredThousands: '', tenThousands: '', thousands: '', hundreds: '', tens: '', ones: '' }
    return
  }
  const padded = amt.toString().padStart(6, '0')
  const digits = padded.split('').map(d => Number(d) === 0 ? '—' : digitMap[Number(d)])
  chineseDigits.value = { hundredThousands: digits[0], tenThousands: digits[1], thousands: digits[2], hundreds: digits[3], tens: digits[4], ones: digits[5] }
}
const onSelectFarmerReceiptOrder = () => {
  const ord = orderList.value.find(o => o.id === selectedFarmerOrderId.value)
  if (ord) {
    const d = new Date(ord.expected_date || ord.order_date)
    farmerReceipt.value.year = (d.getFullYear() - 1911).toString()
    farmerReceipt.value.month = (d.getMonth() + 1).toString().padStart(2, '0')
    farmerReceipt.value.day = d.getDate().toString().padStart(2, '0')
    farmerReceipt.value.buyerName = ord.customer
    farmerReceipt.value.taxId = ord.tax_id || ''
    const cust = customers.value.find(c => c.name === ord.customer)
    farmerReceipt.value.buyerAddress = cust?.line_note || '桃園市'
    farmerReceipt.value.spec = formatSimpleItemName(ord)
    farmerReceipt.value.qty = `${getOrderTotalPots(ord)} 盆`
    farmerReceipt.value.totalAmount = Number(ord.price) || 0
    farmerReceipt.value.note = ord.id
    updateChineseAmount()
  }
}
const fillFarmerReceiptFromOrder = (ord) => {
  selectedFarmerOrderId.value = ord.id
  onSelectFarmerReceiptOrder()
  currentTab.value = 'farmer_receipt'
  nextTick(() => autoFitFarmerReceipt())
}
const printFarmerReceipt = () => window.print()

const shareFarmerReceiptDirect = async () => {
  const targetEl = document.getElementById('farmer-print-target')
  if (!targetEl) return alert('找不到農民收據！')
  showToast('⏳ 正在生成收據圖片...')
  try {
    const origTransform = targetEl.style.transform
    targetEl.style.transform = 'none'
    const canvas = await html2canvas(targetEl, { scale: 2, backgroundColor: '#ffffff', useCORS: true, logging: false })
    targetEl.style.transform = origTransform
    const filename = `農民收據_${farmerReceipt.value.buyerName}_${farmerReceipt.value.year}${farmerReceipt.value.month}${farmerReceipt.value.day}.png`
    shareOrCopyCanvasBlob(canvas, filename, '農民收據確認', '農民收據圖片準備完成')
  } catch (err) {
    showToast('⚠️ 圖片生成失敗，請重試！')
  }
}

const shareOrCopyCanvasBlob = async (canvas, filename, shareTitle, successMsg) => {
  const dataUrl = canvas.toDataURL('image/png')
  const downloadLink = document.createElement('a')
  downloadLink.download = filename
  downloadLink.href = dataUrl
  document.body.appendChild(downloadLink)
  downloadLink.click()
  document.body.removeChild(downloadLink)

  shareModalImg.value = dataUrl
  shareModalTitle.value = shareTitle

  canvas.toBlob(async (blob) => {
    if (!blob) return
    const file = new File([blob], filename, { type: 'image/png' })
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
      try {
        await navigator.share({ title: shareTitle, files: [file] })
        return
      } catch (err) {}
    }
    if (navigator.clipboard && navigator.clipboard.write) {
      try {
        await navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })])
        showToast(`📋 ${successMsg}！\n已複製到剪貼簿並完成下載！請直接至電腦版 LINE 按 Ctrl + V 貼上發送！`)
        return
      } catch (err) {}
    }
    showToast(`📁 ${successMsg}！\n圖檔已下載！請直接拖曳圖片或在此視窗按右鍵「複製圖片」至 LINE 發送！`)
  }, 'image/png')
}

const initSystemData = () => {
  autoFitZoom()
  autoFitReceipt()
  autoFitFarmerReceipt()
  updateChineseAmount()
  fetchSealFromCloud()
  loadOrders()
  loadInventory()
  loadCustomers()
  loadOrchids()
  loadReturns()
  loadCloudDrafts()
}

onMounted(() => {
  if (!document.getElementById('google-noto-fonts-cdn')) {
    const link = document.createElement('link')
    link.id = 'google-noto-fonts-cdn'
    link.rel = 'stylesheet'
    link.href = 'https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700;800&family=Noto+Serif+TC:wght@400;500;600;700;800&display=swap'
    document.head.appendChild(link)
  }

  if (!document.getElementById('cns11643-tw-kai-font')) {
    const style = document.createElement('style')
    style.id = 'cns11643-tw-kai-font'
    style.innerHTML = `
      @font-face {
        font-family: 'TW-Kai';
        src: url('https://cdn.jsdelivr.net/gh/fontsource/tw-kai/files/tw-kai-400-normal.woff2') format('woff2');
        font-weight: 400;
        font-display: swap;
      }
    `
    document.head.appendChild(style)
  }

  window.addEventListener('resize', () => {
    autoFitZoom()
    autoFitReceipt()
    autoFitFarmerReceipt()
  })

  if (isAuthenticated.value) initSystemData()
})
</script>

<style scoped>
/* 🌟 100% 原始漂亮樣式，完全不改動任何螢幕 class */
.main-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9;
  font-size: 13.5px;
}
.system-root {
  display: flex;
  flex-direction: column;
  flex: 1;
  overflow: hidden;
}

.auth-lock-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  display: flex; justify-content: center; align-items: center; z-index: 99999; padding: 16px; box-sizing: border-box;
}
.auth-lock-card {
  background: white; width: 100%; max-width: 420px; border-radius: 14px; padding: 32px 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4); text-align: center; box-sizing: border-box;
}
.lock-icon { font-size: 40px; margin-bottom: 12px; }
.auth-lock-card h2 { margin: 0 0 6px 0; font-size: 19px; color: #0f172a; font-weight: 900; }
.lock-subtitle { font-size: 13px; color: #64748b; margin: 0 0 20px 0; line-height: 1.5; }
.lock-form { display: flex; flex-direction: column; gap: 12px; }
.lock-input {
  width: 100%; padding: 11px 13px; border: 2px solid #cbd5e1; border-radius: 8px; font-size: 15px;
  text-align: center; letter-spacing: 2px; box-sizing: border-box; outline: none; transition: border-color 0.2s;
}
.lock-input:focus { border-color: #2563eb; }
.lock-btn {
  background: #2563eb; color: white; border: none; padding: 11px; border-radius: 8px; font-size: 14.5px; font-weight: bold; cursor: pointer; transition: background 0.2s;
}
.lock-btn:hover { background: #1d4ed8; }
.lock-error-text { color: #dc2626; font-size: 13px; font-weight: bold; margin-top: 10px; }
.lock-tip { margin-top: 20px; font-size: 12px; color: #94a3b8; line-height: 1.5; }
.logout-nav-btn {
  background: #ef4444 !important; color: white !important; border: none !important; padding: 5px 10px !important;
  border-radius: 6px !important; font-size: 12.5px !important; font-weight: bold !important; cursor: pointer !important; margin-left: 8px;
}

.top-nav {
  height: 50px; background-color: #0f172a; color: white; display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px; flex-shrink: 0; overflow-x: auto;
}
.nav-title { font-size: 16px; font-weight: 900; white-space: nowrap; margin-right: 12px; }
.nav-tabs { display: flex; gap: 8px; align-items: center; }
.nav-tabs button {
  background: #334155; color: #e2e8f0; border: none; padding: 7px 12px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 13px; white-space: nowrap;
}
.nav-tabs button.active { background: #2563eb; color: white; }

.manage-container { display: flex; flex-direction: column; flex: 1; overflow: hidden; }
.sub-nav {
  display: flex; background: #ffffff; border-bottom: 1px solid #e2e8f0; padding: 7px 16px; gap: 8px; overflow-x: auto;
}
.sub-nav button {
  background: #f8fafc; border: 1px solid #cbd5e1; padding: 5px 12px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; white-space: nowrap;
}
.sub-nav button.active { background: #10b981; color: white; border-color: #10b981; }
.manage-content { flex: 1; overflow-y: auto; padding: 14px; }

.edit-banner {
  background: #fef3c7; border: 1.5px solid #f59e0b; color: #92400e;
  padding: 8px 14px; border-radius: 8px; margin-bottom: 12px;
  display: flex; justify-content: space-between; align-items: center; font-size: 13.5px;
}
.cancel-edit-btn { background: #dc2626; color: white; border: none; padding: 4px 10px; border-radius: 4px; font-weight: bold; font-size: 12px; cursor: pointer; }

.card-box { background: white; border-radius: 8px; padding: 14px; box-shadow: 0 2px 6px rgba(0,0,0,0.04); }
.card-box h3 { margin: 0; font-size: 15.5px; color: #1e293b; }

.order-form-title-row {
  display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; flex-wrap: wrap; gap: 8px;
}
.right-aligned-badge { margin-left: auto; }

.form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 10px; }
.field label { display: block; font-size: 12.5px; font-weight: bold; color: #475569; margin-bottom: 4px; }
input, select, textarea {
  width: 100%; padding: 7px 9px; border: 1px solid #cbd5e1;
  border-radius: 6px; font-size: 13.5px; box-sizing: border-box;
}

.items-section {
  background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 12px;
}
.items-header {
  display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px;
}
.items-header h4 { margin: 0; font-size: 13.5px; color: #1e293b; font-weight: bold; }
.add-item-btn {
  background: #2563eb; color: white; border: none; padding: 5px 10px; border-radius: 6px; font-weight: bold; font-size: 12.5px; cursor: pointer;
}
.order-item-card {
  background: white; border: 1.5px solid #cbd5e1; border-radius: 6px; padding: 10px; margin-bottom: 8px;
}
.item-card-title {
  display: flex; justify-content: space-between; align-items: center; font-size: 13px; font-weight: bold; color: #3b82f6; margin-bottom: 6px; padding-bottom: 4px; border-bottom: 1px dashed #e2e8f0;
}
.remove-item-btn {
  background: #fee2e2; color: #dc2626; border: 1px solid #fecaca; padding: 2px 5px; border-radius: 4px; font-size: 12px; cursor: pointer;
}
.item-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 8px;
}

.highlight-field { background-color: #f0fdf4; padding: 6px; border-radius: 6px; border: 1px solid #bbf7d0; }
.highlight-field label { color: #15803d; }
.bold-price-input { font-weight: bold; color: #1d4ed8; font-size: 15px; }
.bold-select-field { font-weight: bold; color: #1e3a8a; background: #eff6ff; }
.text-purple { color: #7e22ce; }
.font-heavy { font-weight: 800 !important; font-size: 15px !important; }

.btn-action-row { display: flex; gap: 8px; }
.primary-btn { background: #2563eb; color: white; border: none; padding: 7px 16px; border-radius: 6px; font-weight: bold; font-size: 13.5px; cursor: pointer; }
.secondary-btn { background: #94a3b8; color: white; border: none; padding: 7px 14px; border-radius: 6px; font-weight: bold; font-size: 13.5px; cursor: pointer; }
.excel-btn { background: #059669; color: white; border: none; padding: 6px 12px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; white-space: nowrap; }
.line-action-btn { background: #06c755; color: white; border: none; padding: 10px; border-radius: 6px; font-weight: bold; font-size: 14.5px; cursor: pointer; width: 100%; }

.statement-filter-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 10px; background: #f8fafc; padding: 10px; border-radius: 6px;
}
.custom-date-range-field {
  background: #fefce8; border: 1.5px solid #fde047; border-radius: 6px; padding: 6px;
}
.statement-summary-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 10px; }
.sum-card { padding: 12px; border-radius: 8px; border: 1px solid #e2e8f0; }
.red-card { background: #fef2f2; border-color: #fecaca; }
.blue-card { background: #eff6ff; border-color: #bfdbfe; }
.purple-card { background: #faf5ff; border-color: #e9d5ff; }
.green-card { background: #f0fdf4; border-color: #bbf7d0; }
.sum-label { font-size: 12.5px; font-weight: bold; color: #475569; margin-bottom: 4px; }
.sum-value { font-size: 20px; font-weight: 900; color: #0f172a; }
.sum-value.font-medium { font-size: 15px; }

.statement-actions { display: flex; gap: 10px; flex-wrap: wrap; }
.line-btn { background: #06c755; color: white; border: none; padding: 7px 14px; border-radius: 6px; font-weight: bold; font-size: 13.5px; cursor: pointer; }
.batch-pay-btn { background: #ea580c; color: white; border: none; padding: 7px 14px; border-radius: 6px; font-weight: bold; font-size: 13.5px; cursor: pointer; }

.table-header-action { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.table-responsive { overflow-x: auto; -webkit-overflow-scrolling: touch; }
.data-table { width: 100%; border-collapse: collapse; font-size: 13.5px; text-align: left; min-width: 1050px; }
.data-table th { background: #f8fafc; padding: 8px 6px; border-bottom: 2px solid #e2e8f0; color: #334155; font-size: 13px; font-weight: bold; white-space: nowrap; }
.data-table td { padding: 6px 6px; border-bottom: 1px solid #e2e8f0; vertical-align: middle; }

.col-status, .col-receipt {
  width: 82px !important;
  min-width: 82px !important;
  text-align: center;
}
.uniform-status-select {
  width: 78px !important;
  min-width: 78px !important;
  padding: 3px 2px !important;
  font-size: 12px !important;
  font-weight: bold !important;
  border-radius: 4px !important;
  cursor: pointer !important;
  text-align: center !important;
  box-sizing: border-box !important;
}
.uniform-status-badge {
  display: inline-block !important;
  width: 74px !important;
  padding: 3px 2px !important;
  font-size: 12px !important;
  font-weight: bold !important;
  border-radius: 4px !important;
  text-align: center !important;
  box-sizing: border-box !important;
}

.spec-cell-wrap { max-width: 220px; line-height: 1.35; word-break: break-all; font-size: 13px; }
.action-cell { white-space: nowrap; }
.stacked-action-container { display: flex; gap: 5px; align-items: center; }
.stacked-action-col { display: flex; flex-direction: column; gap: 4px; }

.cozy-btn {
  padding: 4px 7px; border-radius: 5px; cursor: pointer; font-size: 12px; font-weight: 700;
  white-space: nowrap; text-align: center; transition: all 0.15s ease-in-out;
}
.clean-btn-noborder {
  border: none !important;
  box-shadow: 0 1px 2px rgba(0,0,0,0.06);
}
.icon-only-btn {
  padding: 3px 6px !important; font-size: 13.5px !important; min-width: 28px;
}

.badge { padding: 2px 5px; border-radius: 4px; font-size: 12.5px; font-weight: bold; border: none; cursor: pointer; }
.badge-purple { background: #f3e8ff; color: #7e22ce; }
.badge-green { background: #dcfce7; color: #16a34a; border: 1px solid #bbf7d0; }
.badge-gray { background: #f1f5f9; color: #64748b; border: 1px solid #cbd5e1; }
.badge-red { background: #fee2e2; color: #dc2626; border: 1px solid #fecaca; }
.badge-soft-green { background: #ecfdf5 !important; color: #059669 !important; border: 1px solid #a7f3d0 !important; }

.text-red { color: #dc2626; }
.text-blue { color: #2563eb; }
.text-green { color: #16a34a; }
.text-center { text-align: center; }
.text-gray { color: #94a3b8; }
.py-4 { padding: 14px 0; }
.mt-1 { margin-top: 4px; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 14px; }
.mb-1 { margin-bottom: 4px; }
.mb-2 { margin-bottom: 8px; }

.shipping-tab-header {
  display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 8px; margin-bottom: 8px;
}
.modern-pill-tabs {
  display: inline-flex; background: #e2e8f0; padding: 3px; border-radius: 30px; gap: 3px;
}
.pill-tab-item {
  border: none; background: transparent; padding: 5px 12px; border-radius: 20px; font-size: 12.5px; font-weight: bold;
  color: #475569; cursor: pointer; transition: all 0.2s ease;
}
.pill-tab-item:hover { color: #0f172a; }
.pill-tab-item.active {
  background: #1e293b; color: #ffffff; box-shadow: 0 2px 5px rgba(15, 23, 42, 0.2);
}

.modern-sign-card {
  background: #f8fafc !important; border: 1.5px solid #cbd5e1 !important; border-radius: 8px !important; padding: 10px !important;
}
.modern-sign-header {
  display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px;
}
.modern-sign-title {
  font-size: 13px; font-weight: bold; color: #1e293b;
}
.modern-clean-sign-btn {
  background: #ffffff; border: 1px solid #cbd5e1; color: #475569; padding: 2px 7px; border-radius: 4px;
  font-size: 11.5px; font-weight: bold; cursor: pointer; transition: all 0.15s ease;
}
.modern-clean-sign-btn:hover {
  background: #fee2e2; color: #dc2626; border-color: #fca5a5;
}
.modern-canvas-wrapper {
  position: relative; width: 100%; display: flex; justify-content: center;
}
.modern-live-sign-pad {
  width: 100%; height: 100px; background: #ffffff; border: 1.5px dashed #94a3b8; border-radius: 6px;
  cursor: crosshair; touch-action: none; box-shadow: inset 0 2px 4px rgba(0,0,0,0.03);
}
.sign-watermark-hint {
  position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
  font-size: 12.5px; color: #cbd5e1; font-weight: bold; pointer-events: none; user-select: none;
}

.double-x-row {
  display: flex; align-items: center; gap: 6px; background: #eff6ff; padding: 5px 7px; border-radius: 6px; border: 1px dashed #93c5fd;
}
.x-input-group { display: flex; align-items: center; gap: 4px; flex: 1; }
.x-badge {
  background: #3b82f6; color: white; font-size: 11px; font-weight: bold; padding: 2px 5px; border-radius: 4px; white-space: nowrap;
}
.x-connector { font-weight: bold; color: #60a5fa; }
.sub-label-tip { font-size: 11.5px; color: #64748b; font-weight: bold; display: block; margin-bottom: 2px; }
.bold-select-dropdown { background: #f8fafc; font-weight: bold; color: #1e3a8a; border-color: #93c5fd; }

.photo-preview-wrap {
  display: flex; align-items: center; gap: 10px; background: #f8fafc; padding: 6px 10px; border-radius: 6px; border: 1px dashed #cbd5e1;
}
.preview-label { font-size: 12px; font-weight: bold; color: #475569; }
.preview-thumb { width: 50px; height: 50px; object-fit: cover; border-radius: 6px; border: 1px solid #cbd5e1; }
.remove-photo-btn { background: #ef4444; color: white; border: none; padding: 3px 6px; border-radius: 4px; font-size: 11px; cursor: pointer; }
.photo-col { width: 56px; text-align: center; }
.table-orchid-img { width: 40px; height: 40px; object-fit: cover; border-radius: 6px; border: 1px solid #e2e8f0; cursor: pointer; }
.no-photo-badge { font-size: 11px; color: #94a3b8; }

.image-modal-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0,0,0,0.7);
  display: flex; justify-content: center; align-items: center; z-index: 9999; padding: 16px; box-sizing: border-box;
}
.image-modal-content {
  background: white; border-radius: 12px; padding: 16px; max-width: 820px; width: 100%; max-height: 92vh;
  display: flex; flex-direction: column; box-shadow: 0 20px 40px rgba(0,0,0,0.4); box-sizing: border-box;
}
.image-modal-header { display: flex; justify-content: space-between; align-items: center; font-weight: bold; margin-bottom: 10px; font-size: 14.5px; }
.close-modal-btn { background: transparent; border: none; font-size: 20px; cursor: pointer; color: #64748b; }
.share-modal-body { display: flex; flex-direction: column; align-items: center; overflow: hidden; width: 100%; }
.share-img-scroll-container {
  width: 100%; display: flex; justify-content: center; align-items: center;
  background-color: #f8fafc; border-radius: 8px; padding: 10px; box-sizing: border-box; margin-bottom: 10px;
}
.share-preview-img-contained { max-height: 60vh; max-width: 100%; object-fit: contain; border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.12); }
.share-tips-row { display: flex; flex-direction: column; gap: 4px; font-size: 12.5px; color: #334155; line-height: 1.5; width: 100%; }

.draft-manage-panel { background: #fdfefe !important; border: 1.5px solid #dbeafe !important; }
.draft-action-btns { display: flex; gap: 6px; margin-top: 6px; }
.ultra-light-purple-btn {
  flex: 1.3; background: linear-gradient(135deg, #ede9fe 0%, #e9d5ff 100%); color: #4c1d95; border: 1.5px solid #c4b5fd;
  padding: 7px 9px; border-radius: 6px; font-size: 12px; font-weight: bold; cursor: pointer;
  box-shadow: 0 1px 3px rgba(167, 139, 250, 0.15); transition: all 0.15s ease-in-out;
}
.ultra-light-purple-btn:hover { background: linear-gradient(135deg, #e0e7ff 0%, #ddd6fe 100%); color: #31104b; transform: translateY(-1px); }
.mint-new-card-btn {
  flex: 0.9; background: #ecfdf5; color: #047857; border: 1.5px solid #a7f3d0;
  padding: 7px 9px; border-radius: 6px; font-size: 12px; font-weight: bold; cursor: pointer; transition: all 0.15s ease-in-out;
}
.mint-new-card-btn:hover { background: #d1fae5; border-color: #6ee7b7; }
.draft-selector-row { display: flex; gap: 6px; align-items: center; }
.mini-refresh-btn { background: #eff6ff; color: #2563eb; border: 1px solid #bfdbfe; border-radius: 4px; font-size: 11px; padding: 2px 5px; cursor: pointer; }
.mini-del-draft-btn { background: #fee2e2; border: 1px solid #fecaca; border-radius: 4px; padding: 5px 8px; cursor: pointer; font-size: 13px; }

.inline-font-weight-row { display: flex; gap: 8px; align-items: flex-end; width: 100%; }
.inline-item-flex { flex: 1.6; }
.inline-item-fixed { flex: 1.1; }
.mini-field-lbl { display: block; font-size: 11.5px; font-weight: bold; color: #475569; margin-bottom: 3px; }
.compact-inline-select { padding: 5px 7px !important; font-size: 12.5px !important; }
.compact-size-wrap {
  display: flex; align-items: center; gap: 2px; background: #ffffff; border: 1px solid #cbd5e1; border-radius: 4px; padding: 1px 3px; flex-shrink: 0;
}
.compact-size-lbl { font-size: 11px; font-weight: bold; color: #64748b; }
.compact-size-input {
  width: 40px !important; padding: 1px 2px !important; font-size: 11.5px !important; font-weight: bold !important; text-align: center; border: none !important; outline: none;
}

.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.couplet-screen-wrapper { display: flex; flex: 1; overflow: hidden; height: calc(100vh - 50px); }
.control-panel {
  width: 410px; background: white; padding: 14px; box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 9px; border-radius: 6px; margin-bottom: 9px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }
.bold-select { font-weight: bold; font-size: 13.5px; border-color: #3b82f6; }

.section-title-with-weight { display: flex; justify-content: space-between; align-items: center; width: 100%; }
.ctrl-row-right { display: flex; align-items: center; gap: 5px; flex-shrink: 0; }
.mini-weight-select {
  width: auto !important; padding: 2px 4px !important; font-size: 11px !important; font-weight: bold !important;
  color: #1e3a8a !important; background: #eff6ff !important; border: 1px solid #bfdbfe !important; border-radius: 4px !important; flex-shrink: 0;
}
.flex-input { flex: 1; }

.stamp-select-panel { background: #fdf2f8; border: 1.5px dashed #db2777; }
.seal-choose-btn {
  width: 100%; margin-top: 6px; padding: 7px; background: #db2777; color: white; border: none; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer;
}

.section-title { font-size: 13px; font-weight: bold; margin-bottom: 4px; display: inline-block; }
.form-group { margin-bottom: 8px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 3px; color: #334155; }
.bottom-input-group { display: flex; align-items: center; gap: 5px; margin-bottom: 5px; }
.line-num { font-size: 12px; font-weight: bold; color: #64748b; width: 36px; }
.form-row, .btn-group { display: flex; gap: 5px; }
.btn-group button {
  flex: 1; padding: 6px; border: 1px solid #2563eb; background: white; color: #2563eb; border-radius: 4px; cursor: pointer; font-weight: bold; font-size: 12.5px;
}
.btn-group button.active { background: #2563eb; color: white; }
.radio-row { display: flex; gap: 12px; font-size: 12.5px; }
.tags-container { display: flex; flex-wrap: wrap; gap: 4px; }
.tag-btn { background: #eff6ff; color: #1e40af; border: 1px solid #bfdbfe; padding: 3px 5px; font-size: 12px; border-radius: 4px; cursor: pointer; }
.reset-btn { width: 100%; padding: 7px; background: #f1f5f9; border: 1px dashed #94a3b8; border-radius: 4px; cursor: pointer; font-size: 12.5px; }

.canvas-viewport {
  flex: 1; display: flex; flex-direction: column; align-items: center; overflow: auto;
  padding: 18px 18px 70px 18px; position: relative; background-color: #cbd5e1; -webkit-overflow-scrolling: touch;
}
.receipt-preview-area {
  flex: 1; display: flex; flex-direction: column; align-items: center; overflow: auto;
  padding: 14px; position: relative; background-color: #cbd5e1; -webkit-overflow-scrolling: touch;
}
.zoom-toolbar {
  display: flex; align-items: center; gap: 5px; background: white; padding: 4px 10px;
  border-radius: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.15); margin-bottom: 10px; position: sticky; top: 0; z-index: 10;
}
.zoom-btn { width: 24px; height: 24px; border: 1px solid #cbd5e1; background: #f8fafc; border-radius: 50%; cursor: pointer; }
.zoom-text { font-size: 12.5px; font-weight: bold; min-width: 40px; text-align: center; }
.fit-btn { background: #2563eb; color: white; border: none; padding: 4px 8px; border-radius: 12px; font-size: 12px; cursor: pointer; }

.card-scaler-container { position: relative; margin-bottom: 35px; flex-shrink: 0; }
.card-board {
  background: #ffffff !important; position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none; touch-action: none; border: none !important;
}
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; }
.card-board.mode-vertical .middle-box,
.card-board.mode-vertical .middle-box-2 { letter-spacing: 20px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.card-board.mode-horizontal .middle-box,
.card-board.mode-horizontal .middle-box-2 { letter-spacing: 16px; }

.text-box { position: absolute; cursor: move; padding: 3px 5px; white-space: nowrap; line-height: 1.25; color: #0f172a; }
.text-box:hover { outline: 1px dashed #2563eb; background: rgba(37, 99, 235, 0.04); }
.scale-handle {
  position: absolute; right: -7px; bottom: -7px; width: 16px; height: 16px;
  background: #2563eb; color: white; border-radius: 3px; font-size: 10.5px; display: flex; justify-content: center; align-items: center; cursor: nwse-resize;
}

.kai-font-supported {
  font-family: "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", serif !important;
}

/* 🌟 A5 橫式簽收單 (100% 原始漂亮比例 794x560) */
.receipt-scaler-container { position: relative; flex-shrink: 0; }
.a5-landscape-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 36px 42px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; writing-mode: horizontal-tb; direction: ltr;
  color: #111827; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.sheet-header {
  display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 2.5px solid #1e293b; padding-bottom: 8px;
}
.shop-name-title { font-size: 28px; font-weight: 900; letter-spacing: 2px; color: #0f172a; }
.sheet-main-title { font-size: 24px; font-weight: bold; letter-spacing: 4px; color: #dc2626; }
.header-meta { font-size: 14px; line-height: 1.5; text-align: right; color: #334155; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 10px 0; font-size: 16px; table-layout: fixed; }
.receipt-table td { border: 1.5px solid #334155; padding: 8px 10px; word-break: break-all; }
.receipt-table .lbl { width: 15%; background-color: #f1f5f9; font-weight: bold; text-align: center; color: #1e293b; font-size: 16px; }
.receipt-table .val { width: 35%; font-size: 16px; }
.receipt-table .val-bold { font-weight: bold; font-size: 17px; }
.receipt-table .val-highlight { font-weight: bold; color: #1e3a8a; font-size: 17.5px; }

.sheet-footer { display: flex; justify-content: space-between; align-items: stretch; gap: 16px; }
.footer-left { flex: 1; display: flex; flex-direction: column; justify-content: space-between; font-size: 15px; padding: 4px 0; }
.footer-tip { font-size: 13px; color: #64748b; }
.footer-sign-box {
  width: 220px; border: 1.5px dashed #475569; border-radius: 6px; display: flex; flex-direction: column; background-color: #fafafa;
}
.sign-box-title { background: #e2e8f0; font-size: 13px; font-weight: bold; text-align: center; padding: 3px 0; color: #334155; }
.sign-box-area { flex: 1; min-height: 52px; display: flex; justify-content: center; align-items: center; }
.live-signature-img { max-height: 65px; max-width: 95%; object-fit: contain; }

/* 🌟 農民收據 (100% 原始漂亮手刻格線版面) */
.farmer-scaler-container { position: relative; flex-shrink: 0; }
.farmer-receipt-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 18px 28px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; color: #000;
  box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.f-header { display: flex; flex-direction: column; align-items: center; position: relative; margin-bottom: 12px; }
.f-main-title { font-size: 27px; font-weight: 900; letter-spacing: 5px; text-align: center; }
.f-date-wrap { align-self: flex-end; font-size: 16px; letter-spacing: 2px; margin-top: 10px; }

.f-receipt-grid-table { border: 2px solid #000; display: flex; flex-direction: column; font-size: 16px; }
.f-grid-row { display: flex; border-bottom: 1px solid #000; min-height: 31px; }
.f-grid-row:last-child { border-bottom: none; }
.f-grid-lbl {
  display: flex; justify-content: center; align-items: center; font-weight: bold; letter-spacing: 2px;
  text-align: center; border-right: 1px solid #000; padding: 3px 5px; box-sizing: border-box; flex-shrink: 0; font-size: 16px;
}
.f-grid-val { display: flex; align-items: center; padding-left: 10px; border-right: 1px solid #000; box-sizing: border-box; font-size: 16px; }
.f-grid-val:last-child { border-right: none; }
.f-flex-1 { flex: 1; }
.f-row-top { min-height: 64px; }
.f-col-buyer-group { display: flex; flex-direction: column; width: 58%; border-right: 1px solid #000; }
.f-sub-row { display: flex; flex: 1; border-bottom: 1px solid #000; }
.f-sub-row:last-child { border-bottom: none; }
.f-w-head { width: 145px; }
.f-w-addr-tag { width: 36px; line-height: 1.4; }
.f-full-addr-box { flex: 1; padding: 8px 12px; font-size: 16px; line-height: 1.5; border-right: none !important; }
.f-tax-clean { font-size: 18px; font-weight: bold; letter-spacing: 3px; color: #1e3a8a; }

.col-p-name { width: 25%; }
.col-p-spec { width: 16%; }
.col-p-qty  { width: 10%; }
.col-p-price{ width: 14%; }
.col-p-amt  { width: 18%; }
.col-p-note { width: 17%; border-right: none !important; }

.f-header-row { font-weight: bold; height: 30px; }
.f-data-row { height: 32px; }
.f-empty-row { height: 28px; }

.f-w-total-lbl { width: 205px; white-space: nowrap; font-size: 16px; }
.f-amount-val-cell { flex: 1; border-right: none !important; padding: 3px 12px; }
.f-chinese-amount-line {
  display: flex; align-items: center; justify-content: space-around; width: 100%; font-size: 17px; font-weight: bold;
}
.f-chinese-amount-line .d-val { color: #1e3a8a; min-width: 26px; text-align: center; font-size: 18px; display: inline-block; }

.f-farmer-stamp-cell { border-right: none !important; padding-left: 28px !important; display: flex; align-items: center; gap: 16px; }
.f-farmer-name-clean { font-size: 19px; letter-spacing: 6px; font-weight: bold; }
.cai-real-stamp-img { width: 50px; height: 50px; object-fit: contain; mix-blend-mode: multiply; }

.f-w-id-lbl { width: 190px; }
.f-w-id-val { width: 190px; border-right: none !important; }

.f-text-center { justify-content: center; text-align: center; }
.f-text-right { justify-content: flex-end; text-align: right; }
.f-bold { font-weight: bold; }
.f-pr { padding-right: 12px !important; }

.f-statement { font-size: 13px; text-align: center; letter-spacing: 1px; font-weight: bold; margin-top: 4px; }
.f-footer-note { font-size: 10.5px; line-height: 1.4; color: #222; margin-top: 4px; text-align: justify; }

.print-action-btn {
  width: 100%; padding: 11px; background: #16a34a; color: white; border: none; border-radius: 6px; font-size: 15px; font-weight: bold; cursor: pointer;
}
.print-action-btn:disabled { background: #cbd5e1; cursor: not-allowed; }

@media (max-width: 768px) {
  .app-container, .receipt-container, .couplet-screen-wrapper { flex-direction: column; overflow-y: auto; height: auto; }
  .control-panel { width: 100%; max-height: 46vh; }
  .form-grid { grid-template-columns: 1fr; }
  .canvas-viewport, .receipt-preview-area { padding: 12px 6px 60px 6px; }
  .shipping-tab-header { flex-direction: column; align-items: flex-start; }
}

/* =========================================================
   🌟 列印模式專用修復：螢幕 100% 不變，只在印表機輸出時適配單頁
========================================================= */
@media print {
  html, body {
    margin: 0 !important;
    padding: 0 !important;
    background: white !important;
    overflow: visible !important;
    height: 100% !important;
  }
  .main-wrapper, .couplet-screen-wrapper, .system-root { 
    margin: 0 !important; 
    padding: 0 !important; 
    background: white !important; 
    overflow: visible !important; 
    display: block !important; 
    height: auto !important;
  }
  .no-print { display: none !important; }
  .canvas-viewport, .receipt-preview-area { 
    padding: 0 !important; 
    margin: 0 !important; 
    background: white !important; 
    overflow: visible !important; 
    display: block !important; 
  }
  .card-scaler-container { 
    position: static !important; 
    margin: 0 !important; 
    padding: 0 !important; 
  }
  #card-print-target { 
    position: absolute !important; 
    top: 0 !important; 
    left: 0 !important; 
    transform: none !important; 
    box-shadow: none !important; 
    margin: 0 !important; 
    display: block !important; 
    visibility: visible !important; 
    border: none !important;
    page-break-after: avoid !important;
    page-break-inside: avoid !important;
  }
  #card-print-target * { visibility: visible !important; }

  /* 🌟 簽收單與農民收據：僅在印表機下將高度收至 136mm，確保送貨司機與簽名欄百分之百印出在第 1 頁 */
  .a5-landscape-sheet, .farmer-receipt-sheet { 
    position: relative !important; 
    transform: none !important; 
    box-shadow: none !important; 
    width: 200mm !important; 
    max-width: 200mm !important;
    height: 136mm !important; 
    max-height: 136mm !important;
    margin: 3mm auto 0 auto !important; 
    padding: 5mm 8mm !important;
    box-sizing: border-box !important;
    overflow: hidden !important;
    page-break-after: avoid !important;
    page-break-inside: avoid !important;
    break-inside: avoid !important;
  }
}
</style>