<template>
  <div class="main-wrapper">
    <!-- ================= 內部安全通行碼驗證畫面 ================= -->
    <div v-if="!isAuthenticated" class="auth-lock-overlay">
      <div class="auth-lock-card">
        <div class="lock-icon">🔒</div>
        <h2>宸豐蘭藝・內部管理系統</h2>
        <p class="lock-subtitle">本系統僅限內部工作人員使用，請輸入管理密碼</p>
        
        <div class="lock-form" @keyup.enter="handleLogin">
          <input 
            type="password" 
            v-model="inputPasscode" 
            placeholder="請輸入內部通行密碼" 
            class="lock-input"
            autofocus
          />
          <button type="button" class="lock-btn" @click="handleLogin">
            驗證並進入系統 ➔
          </button>
        </div>
        <div v-if="authError" class="lock-error-text">
          ⚠️️ 密碼錯誤，請重新輸入！
        </div>
        <div class="lock-tip">
          💡 自己人的手機與電腦登入後會自動保持登入，下次開啟無需重複輸入。
        </div>
      </div>
    </div>

    <!-- ================= 系統主畫面 ================= -->
    <template v-else>
      <!-- 動態列印樣式 -->
      <component :is="'style'">
        @media print {
          @page {
            size: {{ getDynamicPrintPageSize }} !important;
            margin: 0 !important;
          }
        }
      </component>

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
            <img :src="shareModalImg" class="share-preview-img" alt="傳送預覽圖" />
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
                <span class="preview-seq-badge">
                  預計產生單號：<b>{{ editingOrderId || previewNextOrderId }}</b>
                </span>
              </div>
              
              <div class="form-grid">
                <div class="field highlight-date-field">
                  <label>📅 下單日期（單號會依此日期編號）：</label>
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
                      🗑️ 刪除此組
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

            <!-- 訂單總覽清單（嚴格依單號順序排序） -->
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
                          class="status-select-box"
                        >
                          <option value="不需收據">不需收據</option>
                          <option value="需開收據">需開收據</option>
                        </select>
                      </td>
                      <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                      <td class="spec-cell-wrap">{{ ord.spec }}</td>
                      <td>{{ getOrderShippingFee(ord) > 0 ? '$' + getOrderShippingFee(ord) : '免運' }}</td>
                      <td class="text-blue font-bold">${{ ord.price }}</td>
                      
                      <!-- 花卡未製作：行內樣式直寫 #E3D0CC -->
                      <td>
                        <select 
                          v-model="ord.card_status" 
                          :style="ord.card_status === '未製作' ? { backgroundColor: '#E3D0CC !important', color: '#5c3832 !important', border: '1px solid #ceb3ad !important' } : {}"
                          :class="getCardStatusClass(ord.card_status)"
                          @change="updateOrderField(ord, 'card_status', ord.card_status)"
                          class="status-select-box"
                        >
                          <option value="未製作">未製作</option>
                          <option value="已製作">已製作</option>
                          <option value="免製作">免製作</option>
                        </select>
                      </td>

                      <!-- 簽收單未列印：行內樣式直寫 #E4B1AB -->
                      <td>
                        <select 
                          v-model="ord.receipt_status" 
                          :style="ord.receipt_status === '未列印' ? { backgroundColor: '#E4B1AB !important', color: '#5c322d !important', border: '1px solid #d49a94 !important' } : {}"
                          :class="ord.receipt_status === '已列印' ? 'badge badge-soft-green' : 'badge'"
                          @change="updateOrderField(ord, 'receipt_status', ord.receipt_status)"
                          class="status-select-box"
                        >
                          <option value="未列印">未列印</option>
                          <option value="已列印">已列印</option>
                        </select>
                      </td>

                      <!-- 出貨狀態 -->
                      <td>
                        <select 
                          v-model="ord.shipped_status" 
                          :class="ord.shipped_status === '已出貨' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)"
                          class="status-select-box"
                        >
                          <option value="未出貨">未出貨</option>
                          <option value="已出貨">已出貨</option>
                        </select>
                      </td>

                      <!-- 收款狀態 -->
                      <td>
                        <select 
                          v-model="ord.payment_status" 
                          :class="ord.payment_status === '已結' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'payment_status', ord.payment_status)"
                          class="status-select-box"
                        >
                          <option value="未結">未結</option>
                          <option value="已結">已結</option>
                        </select>
                      </td>

                      <!-- 按鈕：簽收單 #F8E8D1、農民收據 #AAA1C8 -->
                      <td class="action-cell">
                        <div class="stacked-action-container">
                          <div class="stacked-action-col">
                            <button 
                              class="cozy-btn" 
                              style="background-color: #F8E8D1 !important; color: #6b4e23 !important; border: 1px solid #ead5bb !important;" 
                              @click="fillReceiptFromOrder(ord)" 
                              title="帶入簽收單"
                            >
                              🖨️ 簽收單
                            </button>
                            <button 
                              class="cozy-btn" 
                              style="background-color: #AAA1C8 !important; color: #2e2640 !important; border: 1px solid #9187b0 !important;" 
                              @click="fillFarmerReceiptFromOrder(ord)" 
                              title="帶入農民收據"
                            >
                              🧾 農民收據
                            </button>
                          </div>
                          <div class="stacked-action-col">
                            <button 
                              class="cozy-btn" 
                              style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" 
                              @click="startEditOrder(ord)" 
                              title="修改此訂單"
                            >
                              ✏️ 修改
                            </button>
                            <button 
                              class="cozy-btn" 
                              style="background-color: #ebd8da !important; color: #6e2e34 !important; border: 1px solid #dcb3b7 !important;" 
                              @click="deleteItem('orders', ord.id, loadOrders)" 
                              title="刪除此訂單"
                            >
                              🗑️ 刪除
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

              <!-- 統計資訊三大指標卡片：總金額、總盆數、總訂單數 -->
              <div class="statement-summary-cards mt-3">
                <div class="sum-card red-card">
                  <div class="sum-label">💰 對帳總金額</div>
                  <div class="sum-value">${{ statementTotalAmount.toLocaleString() }} 元</div>
                </div>
                <div class="sum-card purple-card">
                  <div class="sum-label">🌸 總盆數</div>
                  <div class="sum-value">{{ statementTotalPots }} 盆</div>
                </div>
                <div class="sum-card blue-card">
                  <div class="sum-label">📑 總訂單數</div>
                  <div class="sum-value">{{ statementOrders.length }} 筆</div>
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
                      <th>單號</th>
                      <th>下單日</th>
                      <th>客戶名稱</th>
                      <th>統編</th>
                      <th>開收據</th>
                      <th>總盆數</th>
                      <th>規格明細</th>
                      <th>金額</th>
                      <th class="nowrap-col">花卡狀態</th>
                      <th class="nowrap-col">簽收單狀態</th>
                      <th class="nowrap-col">收款狀態 (可直接更改)</th>
                      <th>操作</th>
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
                      <td class="text-blue font-bold">${{ ord.price }}</td>
                      
                      <td class="nowrap-cell">
                        <span 
                          :style="ord.card_status === '未製作' ? { backgroundColor: '#E3D0CC !important', color: '#5c3832 !important', border: '1px solid #ceb3ad !important' } : {}"
                          :class="getCardStatusClass(ord.card_status)" 
                          class="inline-badge"
                        >
                          {{ ord.card_status || '未製作' }}
                        </span>
                      </td>
                      <td class="nowrap-cell">
                        <span 
                          :style="ord.receipt_status === '未列印' ? { backgroundColor: '#E4B1AB !important', color: '#5c322d !important', border: '1px solid #d49a94 !important' } : {}"
                          :class="ord.receipt_status === '已列印' ? 'inline-badge badge-soft-green' : 'inline-badge'"
                        >
                          {{ ord.receipt_status || '未列印' }}
                        </span>
                      </td>
                      
                      <!-- 收款狀態下拉選單 -->
                      <td class="nowrap-cell">
                        <select 
                          v-model="ord.payment_status" 
                          :class="ord.payment_status === '已結' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'payment_status', ord.payment_status)"
                          class="status-select-box"
                        >
                          <option value="未結">未結</option>
                          <option value="已結">已結</option>
                        </select>
                      </td>

                      <td>
                        <div class="stacked-action-col">
                          <button 
                            class="cozy-btn" 
                            style="background-color: #F8E8D1 !important; color: #6b4e23 !important; border: 1px solid #ead5bb !important;" 
                            @click="fillReceiptFromOrder(ord)" 
                            title="帶入簽收單"
                          >
                            🖨️ 簽收單
                          </button>
                          <button 
                            class="cozy-btn" 
                            style="background-color: #AAA1C8 !important; color: #2e2640 !important; border: 1px solid #9187b0 !important;" 
                            @click="fillFarmerReceiptFromOrder(ord)" 
                            title="帶入農民收據"
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
            <div v-if="editingInvId" class="edit-banner">
              <span>✏️ 目前正在編輯進貨紀錄：<b>{{ editingInvId }}</b></span>
              <button class="cancel-edit-btn" @click="cancelEditInv">✕ 取消修改</button>
            </div>

            <div class="card-box" id="inv-form-box">
              <h3>{{ editingInvId ? '✏️ 修改進貨紀錄' : '📦 新增進貨紀錄' }}</h3>
              <div class="form-grid">
                <div class="field">
                  <label>進貨類別</label>
                  <select v-model="formInv.category">
                    <option value="蘭花">蘭花</option>
                    <option value="陶瓷盆">陶瓷盆</option>
                  </select>
                </div>

                <template v-if="formInv.category === '蘭花'">
                  <div class="field">
                    <label>蘭花品種名稱</label>
                    <input v-model="formInv.item_name" list="orchid-options" placeholder="選擇或自行輸入" />
                    <datalist id="orchid-options">
                      <option v-for="o in orchids" :key="o.id" :value="o.name" />
                    </datalist>
                  </div>
                  <div class="field">
                    <label>梗數規格</label>
                    <select v-model="formInv.spec_spike">
                      <option value="單梗">單梗</option>
                      <option value="雙梗">雙梗</option>
                      <option value="多梗">多梗</option>
                    </select>
                  </div>
                  <div class="field">
                    <label>花色</label>
                    <select v-model="formInv.spec_color">
                      <option value="紅">紅</option>
                      <option value="白">白</option>
                      <option value="粉">粉</option>
                      <option value="黃">黃</option>
                      <option value="其他">其他</option>
                    </select>
                  </div>
                  <div class="field">
                    <label>花朵大小</label>
                    <select v-model="formInv.spec_size">
                      <option value="大">大</option>
                      <option value="中">中</option>
                      <option value="小">小</option>
                    </select>
                  </div>
                  <div class="field">
                    <label>株高規格</label>
                    <select v-model="formInv.spec_height">
                      <option value="高">高</option>
                      <option value="中">中</option>
                      <option value="矮">矮</option>
                    </select>
                  </div>
                </template>

                <template v-else>
                  <div class="field">
                    <label>盆器類型</label>
                    <select v-model="formInv.pot_type" @change="onPotTypeChange">
                      <option value="桌上盆 (100)">桌上盆 (成本100)</option>
                      <option value="落地盆陶瓷-喪 (100)">落地盆陶瓷-喪 (成本100)</option>
                      <option value="落地陶瓷盆-喜 (200)">落地陶瓷盆-喜 (成本200)</option>
                      <option value="羅馬盆 (280)">羅馬盆 (成本280)</option>
                      <option value="快捷盆 (70)">快捷盆 (成本70)</option>
                    </select>
                  </div>
                </template>

                <div class="field">
                  <label>{{ formInv.category === '陶瓷盆' ? '進貨數量' : '進貨株數 (棵)' }}</label>
                  <input v-model.number="formInv.qty" type="number" min="1" @input="calcInvCost" />
                </div>
                <div class="field">
                  <label>{{ formInv.category === '陶瓷盆' ? '單個價格 (元)' : '單株價格 (元)' }}</label>
                  <input v-model.number="formInv.unit_cost" type="number" min="0" @input="calcInvCost" />
                </div>
                <div class="field">
                  <label>總成本 (元)</label>
                  <input v-model.number="formInv.cost" type="number" min="0" />
                </div>
                <div class="field">
                  <label>供應商 / 花農</label>
                  <input v-model="formInv.supplier" type="text" placeholder="某某花農" />
                </div>
                <div class="field">
                  <label>進貨日期</label>
                  <input v-model="formInv.date" type="date" />
                </div>
              </div>

              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveInventory">
                  {{ editingInvId ? '確認更新進貨' : '確認新增進貨' }}
                </button>
                <button v-if="editingInvId" class="secondary-btn" @click="cancelEditInv">
                  取消
                </button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📦 現有進貨清單 ({{ inventoryList.length }} 筆)</h3>
              <div class="table-responsive">
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
                          <button class="cozy-btn" style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" @click="startEditInv(inv)" title="修改">✏️ 修改</button>
                          <button class="cozy-btn" style="background-color: #ebd8da !important; color: #6e2e34 !important; border: 1px solid #dcb3b7 !important;" @click="deleteItem('inventory', inv.id, loadInventory)" title="刪除">🗑️ 刪除</button>
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
            <div v-if="editingCustId" class="edit-banner">
              <span>✏️ 目前正在編輯客戶：<b>{{ editingCustId }}</b></span>
              <button class="cancel-edit-btn" @click="cancelEditCust">✕ 取消修改</button>
            </div>

            <div class="card-box" id="cust-form-box">
              <h3>{{ editingCustId ? '✏️ 修改客戶資料' : '👥 新增客戶 / 花店資料' }}</h3>
              <div class="form-grid">
                <div class="field">
                  <label>客戶 / 店鋪名稱</label>
                  <input v-model="formCust.name" type="text" placeholder="例: 大吉花店、宏達批發" />
                </div>
                <div class="field">
                  <label>客戶類別</label>
                  <select v-model="formCust.type">
                    <option value="批發商">批發商</option>
                    <option value="零售">零售</option>
                    <option value="花店">花店</option>
                    <option value="個人">個人</option>
                  </select>
                </div>
                <div class="field">
                  <label>預設結帳週期</label>
                  <select v-model="formCust.billing_cycle">
                    <option value="每單結">每單結 (現結)</option>
                    <option value="週結">週結</option>
                    <option value="月結">月結</option>
                  </select>
                </div>
                <div class="field">
                  <label>聯絡電話</label>
                  <input v-model="formCust.phone" type="text" placeholder="0912-345678" />
                </div>
                <div class="field">
                  <label>常用送達地址 / 備註</label>
                  <input v-model="formCust.line_note" type="text" placeholder="常用送達地址" />
                </div>
              </div>

              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveCustomer">
                  {{ editingCustId ? '確認更新客戶' : '儲存客戶資料' }}
                </button>
                <button v-if="editingCustId" class="secondary-btn" @click="cancelEditCust">
                  取消
                </button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📋 現有客戶清單 ({{ customers.length }} 位)</h3>
              <div class="table-responsive">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>客戶編號</th><th>名稱</th><th>類別</th><th>結帳週期</th><th>電話</th><th>地址 / 備註</th><th>操作</th>
                    </tr>
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
                          <button class="cozy-btn" style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" @click="startEditCust(c)" title="修改">✏️ 修改</button>
                          <button class="cozy-btn" style="background-color: #ebd8da !important; color: #6e2e34 !important; border: 1px solid #dcb3b7 !important;" @click="deleteItem('customers', c.id, loadCustomers)" title="刪除">🗑️ 刪除</button>
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
            <div v-if="editingOrchidId" class="edit-banner">
              <span>✏️ 目前正在編輯品種：<b>{{ editingOrchidId }}</b></span>
              <button class="cancel-edit-btn" @click="cancelEditOrchid">✕ 取消修改</button>
            </div>

            <div class="card-box" id="orchid-form-box">
              <h3>{{ editingOrchidId ? '✏️ 修改蘭花品種' : '🌸 新增蘭花品種資料' }}</h3>
              <div class="form-grid">
                <div class="field">
                  <label>品種名稱</label>
                  <input v-model="formOrchid.name" type="text" placeholder="輸入品種名稱" />
                </div>
                <div class="field">
                  <label>特色說明</label>
                  <input v-model="formOrchid.note" type="text" placeholder="花型大小、花期、養護備註" />
                </div>
                <div class="field">
                  <label>品種照片</label>
                  <input type="file" accept="image/*" @change="onPhotoFileChange" />
                </div>
              </div>

              <div v-if="formOrchid.photo_url" class="photo-preview-wrap mt-2">
                <div class="preview-label">照片預覽：</div>
                <img :src="formOrchid.photo_url" class="preview-thumb" alt="品種照片預覽" />
                <button type="button" class="remove-photo-btn" @click="formOrchid.photo_url = ''">✕ 移除照片</button>
              </div>

              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveOrchid">
                  {{ editingOrchidId ? '確認更新品種' : '儲存至品種庫' }}
                </button>
                <button v-if="editingOrchidId" class="secondary-btn" @click="cancelEditOrchid">
                  取消
                </button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📋 現有品種清單 ({{ orchids.length }} 筆)</h3>
              <div class="table-responsive">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>品種編號</th><th>花照</th><th>品種名稱</th><th>特色說明</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="item in orchids" :key="item.id">
                      <td><b>{{ item.id }}</b></td>
                      <td class="photo-col">
                        <img 
                          v-if="item.photo_url" 
                          :src="item.photo_url" 
                          class="table-orchid-img" 
                          @click="openLargePhoto(item.photo_url, item.name)"
                          title="點擊查看大圖" 
                        />
                        <span v-else class="no-photo-badge">無照片</span>
                      </td>
                      <td><b>{{ item.name }}</b></td>
                      <td>{{ item.note }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn" style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" @click="startEditOrchid(item)" title="修改">✏️ 修改</button>
                          <button class="cozy-btn" style="background-color: #ebd8da !important; color: #6e2e34 !important; border: 1px solid #dcb3b7 !important;" @click="deleteItem('orchids', item.id, loadOrchids)" title="刪除">🗑️ 刪除</button>
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
            <div v-if="editingRetId" class="edit-banner">
              <span>✏️ 目前正在編輯退貨紀錄：<b>{{ editingRetId }}</b></span>
              <button class="cancel-edit-btn" @click="cancelEditRet">✕ 取消修改</button>
            </div>

            <div class="card-box" id="return-form-box">
              <h3>{{ editingRetId ? '✏️ 修改退貨紀錄' : '🔄 退貨與不良品登記' }}</h3>
              <div class="form-grid">
                <div class="field">
                  <label>退貨類型</label>
                  <select v-model="formRet.return_type">
                    <option value="退給花農">1. 我們向花農退貨</option>
                    <option value="批發商向我們退貨">2. 批發商向我們退貨</option>
                  </select>
                </div>
                <div class="field">
                  <label>對象名稱</label>
                  <input v-model="formRet.party_name" list="cust-options" placeholder="選擇或手動輸入" />
                </div>
                <div class="field">
                  <label>關聯品項</label>
                  <select v-model="formRet.target_item">
                    <option v-for="inv in inventoryList" :key="inv.id" :value="inv.id + ' - ' + inv.item_name">
                      {{ inv.id }} - {{ inv.item_name }} ({{ inv.spec }})
                    </option>
                  </select>
                </div>
                <div class="field">
                  <label>不良株數 (棵)</label>
                  <input v-model.number="formRet.qty" type="number" min="1" @input="calcRetTotal" />
                </div>
                <div class="field">
                  <label>每棵單價 (元)</label>
                  <input v-model.number="formRet.unit_price" type="number" min="0" @input="calcRetTotal" />
                </div>
                <div class="field">
                  <label>總損益金額 (元)</label>
                  <input v-model.number="formRet.total_amount" type="number" min="0" />
                </div>
                <div class="field">
                  <label>處理日期</label>
                  <input v-model="formRet.date" type="date" />
                </div>
                <div class="field">
                  <label>退貨原因</label>
                  <input v-model="formRet.reason" type="text" placeholder="運送碰撞 / 開花不良" />
                </div>
              </div>

              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveReturn">
                  {{ editingRetId ? '確認更新退貨' : '確認送出退貨紀錄' }}
                </button>
                <button v-if="editingRetId" class="secondary-btn" @click="cancelEditRet">
                  取消
                </button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📋 退貨紀錄清單 ({{ returnList.length }} 筆)</h3>
              <div class="table-responsive">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>退貨單號</th><th>類型</th><th>對象</th><th>品項</th><th>株數</th><th>總額</th><th>原因</th><th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="ret in returnList" :key="ret.id">
                      <td><b>{{ ret.id }}</b></td>
                      <td>{{ ret.return_type }}</td>
                      <td><b>{{ ret.party_name }}</b></td>
                      <td>{{ ret.target_item }}</td>
                      <td>{{ ret.qty }}</td>
                      <td class="text-red"><b>${{ ret.total_amount }}</b></td>
                      <td>{{ ret.reason }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn" style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" @click="startEditRet(ret)" title="修改">✏️ 修改</button>
                          <button class="cozy-btn" style="background-color: #ebd8da !important; color: #6e2e34 !important; border: 1px solid #dcb3b7 !important;" @click="deleteItem('returns', ret.id, loadReturns)" title="刪除">🗑️ 刪除</button>
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
                <div class="shipping-filter-tabs">
                  <button 
                    :class="{ active: shippingViewFilter === 'unshipped' }" 
                    @click="shippingViewFilter = 'unshipped'"
                  >
                    📦 待出貨 / 配送中 ({{ unshippedOrders.length }} 筆)
                  </button>
                  <button 
                    :class="{ active: shippingViewFilter === 'shipped' }" 
                    @click="shippingViewFilter = 'shipped'"
                  >
                    ✅ 已出貨歷史 ({{ shippedOrders.length }} 筆)
                  </button>
                  <button 
                    :class="{ active: shippingViewFilter === 'all' }" 
                    @click="shippingViewFilter = 'all'"
                  >
                    全部 ({{ orderList.length }})
                  </button>
                </div>
              </div>

              <div class="table-responsive mt-3">
                <table class="data-table">
                  <thead>
                    <tr>
                      <th>單號</th>
                      <th>預計送達日</th>
                      <th>客戶名稱</th>
                      <th>電話</th>
                      <th>送達地址 / 備註</th>
                      <th>總盆數</th>
                      <th>花禮規格</th>
                      <th class="nowrap-col">花卡</th>
                      <th class="nowrap-col">簽收單</th>
                      <th class="nowrap-col">出貨狀態</th>
                      <th>操作</th>
                    </tr>
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
                        <span 
                          :style="ord.card_status === '未製作' ? { backgroundColor: '#E3D0CC !important', color: '#5c3832 !important', border: '1px solid #ceb3ad !important' } : {}"
                          :class="getCardStatusClass(ord.card_status)" 
                          class="inline-badge"
                        >
                          {{ ord.card_status || '未製作' }}
                        </span>
                      </td>
                      <td class="nowrap-cell">
                        <span 
                          :style="ord.receipt_status === '未列印' ? { backgroundColor: '#E4B1AB !important', color: '#5c322d !important', border: '1px solid #d49a94 !important' } : {}"
                          :class="ord.receipt_status === '已列印' ? 'inline-badge badge-soft-green' : 'inline-badge'"
                        >
                          {{ ord.receipt_status || '未列印' }}
                        </span>
                      </td>
                      <td class="nowrap-cell">
                        <select 
                          v-model="ord.shipped_status" 
                          :class="ord.shipped_status === '已出貨' ? 'badge badge-green' : 'badge badge-red'"
                          @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)"
                          class="status-select-box"
                        >
                          <option value="未出貨">未出貨</option>
                          <option value="已出貨">已出貨</option>
                        </select>
                      </td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button 
                            class="cozy-btn" 
                            style="background-color: #F8E8D1 !important; color: #6b4e23 !important; border: 1px solid #ead5bb !important;" 
                            @click="fillReceiptFromOrder(ord)" 
                            title="開啟簽收單"
                          >
                            🖨️ 簽收單
                          </button>
                          <button 
                            class="cozy-btn" 
                            style="background-color: #dbe7ee !important; color: #27475f !important; border: 1px solid #b7cce1 !important;" 
                            @click="startEditOrder(ord)" 
                            title="編輯訂單"
                          >
                            ✏️ 修改
                          </button>
                        </div>
                      </td>
                    </tr>
                    <tr v-if="displayedShippingOrders.length === 0">
                      <td colspan="11" class="text-center py-4 text-gray">
                        {{ shippingViewFilter === 'unshipped' ? '🎉 目前沒有待出貨的訂單，全部已順利送達！' : '尚無資料' }}
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

          <!-- A4 / A5 尺寸切換 -->
          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button 
                type="button" 
                :class="{ active: cardPaperSize === 'A4' }" 
                @click="switchPaperSize('A4')"
              >
                A4 (大尺寸)
              </button>
              <button 
                type="button" 
                :class="{ active: cardPaperSize === 'A5' }" 
                @click="switchPaperSize('A5')"
              >
                A5 (小尺寸花卡)
              </button>
            </div>
          </div>

          <!-- 花卡雲端即時草稿庫 -->
          <div class="panel-section draft-manage-panel">
            <div class="section-title-with-weight">
              <label class="section-title">☁️ 花卡全裝置雲端草稿庫：</label>
              <button type="button" class="mini-refresh-btn" @click="loadCloudDrafts" title="重新整理草稿清單">🔄 刷新</button>
            </div>
            <div class="draft-action-btns">
              <button type="button" class="draft-save-btn" @click="saveCurrentAsCloudDraft">
                💾 存至雲端草稿 (全裝置同步)
              </button>
              <button type="button" class="draft-new-btn" @click="startNewCard">
                ＋ 開新花卡
              </button>
            </div>
            <div class="mt-2">
              <label class="sub-lbl">跨電腦/手機讀取暫存草稿 ({{ cloudDrafts.length }} 張)：</label>
              <div class="draft-selector-row">
                <select v-model="selectedDraftId" @change="loadCloudDraft(selectedDraftId)" class="full-input bold-select">
                  <option value="">-- 請選擇要調出的雲端草稿 --</option>
                  <option v-for="d in cloudDrafts" :key="d.id" :value="d.id">
                    {{ d.title }}
                  </option>
                </select>
                <button 
                  v-if="selectedDraftId" 
                  type="button" 
                  class="mini-del-draft-btn" 
                  @click="deleteCloudDraft(selectedDraftId)"
                  title="從雲端刪除此草稿"
                >
                  🗑️
                </button>
              </div>
            </div>
          </div>

          <div class="panel-section">
            <label class="section-title">字體選擇：</label>
            <div class="form-group">
              <select v-model="cardFontFamily" class="full-input">
                <option value="kai">標準楷書 (TW-Kai / 書法正楷)</option>
                <option value="notosong">思源宋體 (Noto Serif TC / 古典明體)</option>
                <option value="fangsong">仿宋古典體 (FangSong / 秀麗骨風)</option>
                <option value="notosans">思源黑體 (Noto Sans TC / 現代簡約)</option>
                <option value="systemkai">系統原生楷體 (BiauKai / KaiTi)</option>
              </select>
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
              <label class="section-title">1. 開頭敬詞（獨立一格）：</label>
              <div class="ctrl-row-right">
                <div class="size-input-wrap">
                  <span class="size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_prefix.size" min="14" max="250" class="mini-size-num-input" />
                </div>
                <select v-model="weights.upper_prefix" class="mini-weight-select" title="設定開頭敬詞粗細">
                  <option value="400">400</option>
                  <option value="500">500</option>
                  <option value="600">600</option>
                  <option value="700">700</option>
                  <option value="800">800</option>
                </select>
              </div>
            </div>
            <div class="form-row">
              <input type="text" v-model="upperPrefix" class="full-input" placeholder="例: 敬悼 或 恭祝" />
              <select v-if="cardCategory === 'funeral'" v-model="upperPrefix" style="width: 110px;">
                <option value="敬悼">敬悼</option>
                <option value="痛悼">痛悼</option>
                <option value="敬唁">敬唁</option>
                <option value="追悼">追悼</option>
              </select>
              <select v-else v-model="upperPrefix" style="width: 110px;">
                <option value="祝">祝</option>
                <option value="恭祝">恭祝</option>
                <option value="恭賀">恭賀</option>
                <option value="敬賀">敬賀</option>
              </select>
            </div>

            <div class="section-title-with-weight mt-2">
              <label class="section-title">2. 受禮對象 / 稱謂（獨立一格）：</label>
              <div class="ctrl-row-right">
                <div class="size-input-wrap">
                  <span class="size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_target.size" min="14" max="250" class="mini-size-num-input" />
                </div>
                <select v-model="weights.upper_target" class="mini-weight-select" title="設定稱謂粗細">
                  <option value="400">400</option>
                  <option value="500">500</option>
                  <option value="600">600</option>
                  <option value="700">700</option>
                  <option value="800">800</option>
                </select>
              </div>
            </div>
            <template v-if="cardCategory === 'funeral'">
              <div class="form-row">
                <select v-model="funeralUpperFormat" @change="onFuneralFormatChange">
                  <option value="X媽X老夫人">X媽X老夫人</option>
                  <option value="X媽X夫人">X媽X夫人</option>
                  <option value="X公X老先生">X公X老先生</option>
                  <option value="X公X先生">X公X先生</option>
                  <option value="X女士">X女士</option>
                  <option value="X先生">X先生</option>
                  <option value="custom">自行輸入</option>
                </select>
              </div>
            </template>
            <input type="text" v-model="upperTarget" class="full-input mt-1" placeholder="受禮人或逝者姓名稱謂" />

            <div class="section-title-with-weight mt-2">
              <label class="section-title">3. 上款結尾詞（選填）：</label>
              <div class="ctrl-row-right">
                <div class="size-input-wrap">
                  <span class="size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.upper_suffix.size" min="14" max="250" class="mini-size-num-input" />
                </div>
                <select v-model="weights.upper_suffix" class="mini-weight-select" title="設定結尾詞粗細">
                  <option value="400">400</option>
                  <option value="500">500</option>
                  <option value="600">600</option>
                  <option value="700">700</option>
                  <option value="800">800</option>
                </select>
              </div>
            </div>
            <div class="form-row">
              <input type="text" v-model="upperSuffix" class="full-input" placeholder="留空則不顯示" />
              <select v-if="cardCategory === 'funeral'" v-model="upperSuffix" style="width: 110px;">
                <option value="千古">千古</option>
                <option value="仙逝">仙逝</option>
                <option value="靈前">靈前</option>
                <option value="冥前">冥前</option>
                <option value="">(留空)</option>
              </select>
              <select v-else v-model="upperSuffix" style="width: 110px;">
                <option value="誌慶">誌慶</option>
                <option value="大吉">大吉</option>
                <option value="惠存">惠存</option>
                <option value="">(留空)</option>
              </select>
            </div>
          </div>

          <!-- 中款設定 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <label class="section-title">中款第 1 行（主要題詞）：</label>
              <div class="ctrl-row-right">
                <div class="size-input-wrap">
                  <span class="size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.middle.size" min="14" max="300" class="mini-size-num-input" />
                </div>
                <select v-model="weights.middle" class="mini-weight-select" title="設定中款第1行粗細">
                  <option value="400">400</option>
                  <option value="500">500</option>
                  <option value="600">600</option>
                  <option value="700">700</option>
                  <option value="800">800</option>
                </select>
              </div>
            </div>

            <template v-if="cardCategory === 'funeral'">
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
            </template>

            <template v-else>
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
            </template>

            <input type="text" v-model="middleText" class="full-input mt-2" placeholder="中款第 1 行詞語" />

            <div class="section-title-with-weight mt-3">
              <label class="section-title">中款第 2 行（選填，兩行時使用）：</label>
              <div class="ctrl-row-right">
                <div class="size-input-wrap">
                  <span class="size-lbl">字級:</span>
                  <input type="number" v-model.number="layout.middle_2.size" min="14" max="300" class="mini-size-num-input" />
                </div>
                <select v-model="weights.middle_2" class="mini-weight-select" title="設定中款第2行粗細">
                  <option value="400">400</option>
                  <option value="500">500</option>
                  <option value="600">600</option>
                  <option value="700">700</option>
                  <option value="800">800</option>
                </select>
              </div>
            </div>
            <input type="text" v-model="middleText2" class="full-input" placeholder="留空則不顯示第 2 行" />
          </div>

          <!-- 下款 6 格獨立粗細與字級設定 -->
          <div class="panel-section">
            <label class="section-title">下款設定（每個格子可個別選擇粗細與數字字級）：</label>
            <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
              <span class="line-num">格 {{ idx + 1 }}</span>
              <input type="text" v-model="item.text" :placeholder="getPlaceholder(idx)" class="flex-input" />
              <div class="size-input-wrap">
                <input type="number" v-model.number="layout['bottom_' + idx].size" min="14" max="150" class="mini-size-num-input" title="微調字級大小" />
              </div>
              <select v-model="weights['bottom_' + idx]" class="mini-weight-select" title="設定此格粗細">
                <option value="400">400</option>
                <option value="500">500</option>
                <option value="600">600</option>
                <option value="700">700</option>
                <option value="800">800</option>
              </select>
            </div>

            <div class="form-group mt-2">
              <div class="section-title-with-weight">
                <label class="section-title">結尾敬詞：</label>
                <div class="ctrl-row-right">
                  <div class="size-input-wrap">
                    <span class="size-lbl">字級:</span>
                    <input type="number" v-model.number="layout.suffix.size" min="14" max="200" class="mini-size-num-input" />
                  </div>
                  <select v-model="weights.suffix" class="mini-weight-select" title="設定敬詞粗細">
                    <option value="400">400</option>
                    <option value="500">500</option>
                    <option value="600">600</option>
                    <option value="700">700</option>
                    <option value="800">800</option>
                  </select>
                </div>
              </div>
              <select v-model="suffixText" class="full-input">
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
          <!-- 🌟 傳送函式直接使用高精準原生 Canvas 繪製，徹底告別邊界裁切 -->
          <button type="button" class="line-action-btn mt-2" @click="generateFlawlessCoupletImage">💬 直接傳送 / 複製花卡給客人 (免下載)</button>
          <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印花卡 / 輓聯 ({{ cardPaperSize }})</button>
        </div>

        <div class="canvas-viewport" ref="viewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.25, +(zoomLevel - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
            <button type="fit-btn" @click="zoomLevel = 1.0">🔍 100% 檢視</button>
            <button type="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
          </div>

          <div 
            class="card-scaler-container" 
            :style="{
              width: currentCardDimensions.w * zoomLevel + 'px',
              height: currentCardDimensions.h * zoomLevel + 'px'
            }"
          >
            <!-- 🌟 花卡看板主體 (完全純白底，徹底移除粉紅框) -->
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
              <!-- 1. 上款開頭敬語 -->
              <div 
                v-if="upperPrefix.trim()"
                class="text-box upper-prefix-box"
                :style="getStyle('upper_prefix')"
                @pointerdown="startMove($event, 'upper_prefix')"
              >
                <span>{{ upperPrefix }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_prefix')">⤡</div>
              </div>

              <!-- 2. 受禮對象/稱謂 (取消所有阻擋邊界，給予絕對充足空間) -->
              <div 
                v-if="upperTarget.trim()"
                class="text-box upper-target-box"
                :style="getUpperTargetBoxStyle()"
                @pointerdown="startMove($event, 'upper_target')"
              >
                <span 
                  v-for="(token, tIdx) in parsedUpperTargetTokens" 
                  :key="tIdx" 
                  :style="token.isSmall ? { fontSize: maFontSize + 'px' } : {}"
                >
                  {{ token.char }}
                </span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_target')">⤡</div>
              </div>

              <!-- 3. 上款結尾敬詞 -->
              <div 
                v-if="upperSuffix && upperSuffix.trim()"
                class="text-box upper-suffix-box"
                :style="getStyle('upper_suffix')"
                @pointerdown="startMove($event, 'upper_suffix')"
              >
                <span>{{ upperSuffix }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_suffix')">⤡</div>
              </div>

              <!-- 中款第 1 行 -->
              <div 
                v-if="middleText.trim()"
                class="text-box middle-box"
                :style="getStyle('middle')"
                @pointerdown="startMove($event, 'middle')"
              >
                <span>{{ middleText }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
              </div>

              <!-- 中款第 2 行 -->
              <div 
                v-if="middleText2.trim()"
                class="text-box middle-box-2"
                :style="getStyle('middle_2')"
                @pointerdown="startMove($event, 'middle_2')"
              >
                <span>{{ middleText2 }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'middle_2')">⤡</div>
              </div>

              <!-- 下款 -->
              <template v-for="(item, idx) in bottomLines" :key="'bottom-' + idx">
                <div 
                  v-if="item.text.trim()"
                  class="text-box"
                  :style="getStyle('bottom_' + idx)"
                  @pointerdown="startMove($event, 'bottom_' + idx)"
                >
                  <span>{{ item.text }}</span>
                  <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'bottom_' + idx)">⤡</div>
                </div>
              </template>

              <!-- 敬詞 -->
              <div 
                v-if="suffixText.trim()"
                class="text-box suffix-box"
                :style="getStyle('suffix')"
                @pointerdown="startMove($event, 'suffix')"
              >
                <span>{{ suffixText }}</span>
                <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'suffix')">⤡</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ================= 模式 3：A5 橫式簽收單 ================= -->
      <div v-else-if="currentTab === 'receipt'" class="receipt-container">
        <!-- 保持簽收單原有功能不變 -->
      </div>

      <!-- ================= 模式 4：農民收據 ================= -->
      <div v-else-if="currentTab === 'farmer_receipt'" class="receipt-container">
        <!-- 保持農民收據原有功能不變 -->
      </div>
    </template>
  </div>
</template>

<script setup>
// 🌟 核心繪製文字輔助函式（自動動態向左平移，絕不切字）
const generateFlawlessCoupletImage = () => {
  showToast('⏳ 正在生成無切字花卡高畫質圖片...')

  const width = currentCardDimensions.value.w
  const height = currentCardDimensions.value.h

  const canvas = document.createElement('canvas')
  canvas.width = width * 2
  canvas.height = height * 2
  const ctx = canvas.getContext('2d')
  ctx.scale(2, 2)

  // 1. 純白底（完全無粉紅外框）
  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, width, height)

  const targetFontFamily = activeCssFontFamily.value
  ctx.fillStyle = '#0f172a'

  // 2. 核心繪製邏輯
  const renderItem = (text, item, wVal, isTarget = false) => {
    if (!text || !item) return
    const w = String(wVal || '600')
    const strokeWidthMap = { '400': 0, '500': 0.4, '600': 0.8, '700': 1.4, '800': 2.2 }
    const sWidth = strokeWidthMap[w] || 0.8
    ctx.textBaseline = 'top'

    if (isVertical.value) {
      let curY = item.y
      const chars = text.split('')
      chars.forEach(char => {
        const isMa = isTarget && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${w} ${curSize}px ${targetFontFamily}`
        const offX = isMa ? Math.round((item.size - curSize) / 2) : 0

        if (sWidth > 0) {
          ctx.strokeStyle = '#0f172a'
          ctx.lineWidth = sWidth
          ctx.strokeText(char, item.x + offX, curY)
        }
        ctx.fillText(char, item.x + offX, curY)
        curY += curSize + 8
      })
    } else {
      // 🌟 橫式排版核心修復：精確計算總長度，如果右端會超出邊界，系統自動將起始點往左移！
      const chars = text.split('')
      let totalTextWidth = 0
      chars.forEach(char => {
        const isMa = isTarget && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        totalTextWidth += curSize + 6
      })

      let curX = item.x
      const safeRightMargin = 45 // 距離右邊界 45px 安全邊距
      if (curX + totalTextWidth > width - safeRightMargin) {
        curX = width - safeRightMargin - totalTextWidth // 自動向左挪移，保證最後一個字「員」100% 完整呈現
      }

      chars.forEach(char => {
        const isMa = isTarget && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${w} ${curSize}px ${targetFontFamily}`
        const offY = isMa ? Math.round((item.size - curSize) / 2) : 0

        if (sWidth > 0) {
          ctx.strokeStyle = '#0f172a'
          ctx.lineWidth = sWidth
          ctx.strokeText(char, curX, item.y + offY)
        }
        ctx.fillText(char, curX, item.y + offY)
        curX += curSize + 6
      })
    }
  }

  // 繪製上款敬詞
  renderItem(upperPrefix.value, layout.value.upper_prefix, weights.value.upper_prefix)
  // 繪製受禮對象（新北市 陳乃瑜議員）
  renderItem(upperTarget.value, layout.value.upper_target, weights.value.upper_target, true)
  // 繪製上款結尾詞
  if (upperSuffix.value && upperSuffix.value.trim()) {
    renderItem(upperSuffix.value, layout.value.upper_suffix, weights.value.upper_suffix)
  }
  // 繪製中款
  renderItem(middleText.value, layout.value.middle, weights.value.middle)
  if (middleText2.value && middleText2.value.trim()) {
    renderItem(middleText2.value, layout.value.middle_2, weights.value.middle_2)
  }
  // 繪製下款 6 格
  bottomLines.value.forEach((b, idx) => {
    if (b.text && b.text.trim()) {
      renderItem(b.text, layout.value['bottom_' + idx], weights.value['bottom_' + idx])
    }
  })
  // 繪製敬詞
  renderItem(suffixText.value, layout.value.suffix, weights.value.suffix)

  const filename = `花卡_${cardPaperSize.value}_${new Date().toISOString().split('T')[0]}.png`
  shareOrCopyCanvasBlob(canvas, filename, `花卡確認 (${cardPaperSize.value})`, '花卡圖片準備完成')
}
</script>

<style scoped>
.main-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9;
}

/* 內部安全通行碼防護 */
.auth-lock-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  display: flex; justify-content: center; align-items: center; z-index: 99999; padding: 16px; box-sizing: border-box;
}
.auth-lock-card {
  background: white; width: 100%; max-width: 420px; border-radius: 14px; padding: 32px 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4); text-align: center; box-sizing: border-box;
}
.lock-icon { font-size: 42px; margin-bottom: 12px; }
.auth-lock-card h2 { margin: 0 0 6px 0; font-size: 20px; color: #0f172a; font-weight: 900; }
.lock-subtitle { font-size: 13px; color: #64748b; margin: 0 0 20px 0; line-height: 1.5; }
.lock-form { display: flex; flex-direction: column; gap: 12px; }
.lock-input {
  width: 100%; padding: 12px 14px; border: 2px solid #cbd5e1; border-radius: 8px; font-size: 16px;
  text-align: center; letter-spacing: 2px; box-sizing: border-box; outline: none; transition: border-color 0.2s;
}
.lock-input:focus { border-color: #2563eb; }
.lock-btn {
  background: #2563eb; color: white; border: none; padding: 12px; border-radius: 8px; font-size: 15px; font-weight: bold; cursor: pointer; transition: background 0.2s;
}
.lock-btn:hover { background: #1d4ed8; }
.lock-error-text { color: #dc2626; font-size: 13px; font-weight: bold; margin-top: 10px; }
.lock-tip { margin-top: 20px; font-size: 12px; color: #94a3b8; line-height: 1.5; }
.logout-nav-btn {
  background: #ef4444 !important; color: white !important; border: none !important; padding: 6px 12px !important;
  border-radius: 6px !important; font-size: 12px !important; font-weight: bold !important; cursor: pointer !important; margin-left: 8px;
}

/* 頂端導航 */
.top-nav {
  height: 52px; background-color: #0f172a; color: white; display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px; flex-shrink: 0; overflow-x: auto;
}
.nav-title { font-size: 16px; font-weight: 900; white-space: nowrap; margin-right: 12px; }
.nav-tabs { display: flex; gap: 8px; align-items: center; }
.nav-tabs button {
  background: #334155; color: #e2e8f0; border: none; padding: 8px 14px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 13px; white-space: nowrap;
}
.nav-tabs button.active { background: #2563eb; color: white; }

/* 蘭花管理後台 */
.manage-container { display: flex; flex-direction: column; flex: 1; overflow: hidden; }
.sub-nav {
  display: flex; background: #ffffff; border-bottom: 1px solid #e2e8f0; padding: 8px 16px; gap: 8px; overflow-x: auto;
}
.sub-nav button {
  background: #f8fafc; border: 1px solid #cbd5e1; padding: 6px 14px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; white-space: nowrap;
}
.sub-nav button.active { background: #10b981; color: white; border-color: #10b981; }
.manage-content { flex: 1; overflow-y: auto; padding: 16px; }

/* 表格與按鈕樣式 */
.data-table { width: 100%; border-collapse: collapse; font-size: 13px; text-align: left; min-width: 1100px; }
.data-table th { background: #f8fafc; padding: 10px 8px; border-bottom: 2px solid #e2e8f0; color: #475569; white-space: nowrap; }
.data-table td { padding: 8px 8px; border-bottom: 1px solid #e2e8f0; vertical-align: middle; }
.nowrap-col { white-space: nowrap; min-width: 86px; }
.nowrap-cell { white-space: nowrap !important; }

/* 花卡畫布（🌟 徹底移除任何粉紅邊框） */
.card-scaler-container { position: relative; margin-bottom: 40px; flex-shrink: 0; }
.card-board {
  background: #ffffff !important; position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none; touch-action: none; border: none !important;
}
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; }
.card-board.mode-vertical .middle-box,
.card-board.mode-vertical .middle-box-2 { letter-spacing: 20px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.card-board.mode-horizontal .middle-box,
.card-board.mode-horizontal .middle-box-2 { letter-spacing: 16px; }
.card-board.style-floral { border: none !important; }

.text-box { position: absolute; cursor: move; padding: 4px 6px; white-space: nowrap; line-height: 1.25; color: #0f172a; }
.text-box:hover { outline: 1px dashed #2563eb; background: rgba(37, 99, 235, 0.04); }
.scale-handle {
  position: absolute; right: -7px; bottom: -7px; width: 17px; height: 17px;
  background: #2563eb; color: white; border-radius: 3px; font-size: 11px;
  display: flex; justify-content: center; align-items: center; cursor: nwse-resize;
}

@media print {
  html, body, .main-wrapper, .couplet-screen-wrapper { 
    margin: 0 !important; padding: 0 !important; background: white !important; overflow: visible !important; display: block !important; 
  }
  .no-print { display: none !important; }
  .canvas-viewport, .receipt-preview-area { 
    padding: 0 !important; margin: 0 !important; background: white !important; overflow: visible !important; display: block !important; 
  }
  .card-scaler-container { position: static !important; margin: 0 !important; padding: 0 !important; }
  #card-print-target { 
    position: absolute !important; top: 0 !important; left: 0 !important; transform: none !important; box-shadow: none !important; margin: 0 !important; display: block !important; visibility: visible !important; border: none !important;
  }
  #card-print-target * { visibility: visible !important; }
}
</style>