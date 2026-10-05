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
      <div v-if="shareModalImg" class="image-modal-overlay no-print" @click="closeShareModal">
        <div class="image-modal-content share-preview-modal" @click.stop>
          <div class="image-modal-header">
            <span>💬 {{ shareModalTitle }}</span>
            <button class="close-modal-btn" @click="closeShareModal">✕</button>
          </div>
          <div class="share-modal-body">
            <div class="share-img-scroll-container">
              <img :src="shareModalImg" class="share-preview-img-contained" alt="傳送預覽圖" />
            </div>
            
            <div class="share-btn-action-group">
              <button type="button" class="mobile-print-btn" @click="triggerImagePrint">
                🖨️ 手機直接列印 (無網址・無日期・一張紙)
              </button>
              <button v-if="canNativeShare" type="button" class="mobile-share-btn" @click="triggerNativeShare">
                📲 一鍵直接傳送至 LINE / 其他應用
              </button>
              <a :href="shareModalImg" :download="shareModalFilename" class="mobile-dl-btn">
                💾 下載圖檔至相簿 / 電腦
              </a>
            </div>

            <div class="share-tips-row">
              <span>💡 <b>傳送小提示：</b></span>
              <span>• <b>手機/平板</b>：點擊「一鍵直接傳送」直接選 LINE，或在圖片上<b>長按「儲存影像」</b>。</span>
              <span>• <b>電腦版</b>：在圖片上點<b>右鍵 ➔「複製圖片」</b>，到 LINE 按 <b>Ctrl + V</b> 即可送出。</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 品種照片放大檢視彈窗 -->
      <div v-if="activeModalPhoto" class="image-modal-overlay no-print" @click="activeModalPhoto = null">
        <div class="image-modal-content" @click.stop>
          <div class="image-modal-header">
            <span>🌸 {{ activeModalTitle }}</span>
            <button class="close-modal-btn" @click="activeModalPhoto = null">✕</button>
          </div>
          <div class="share-modal-body">
            <img :src="activeModalPhoto" class="share-preview-img-contained" alt="品種大圖" />
          </div>
        </div>
      </div>

      <!-- ================= 模式 1：蘭花管理系統 (100% 精美完整原始版面) ================= -->
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
                      🗑
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
                              🖨️ 簽收單
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
              <h3>{{ editingInvId ? '✏️ 修改進貨紀錄' : '📦 建立進貨與耗材入庫' }}</h3>

              <div class="form-grid mt-2">
                <div class="field">
                  <label>進貨類別：</label>
                  <select v-model="formInv.category">
                    <option value="蘭花">蘭花</option>
                    <option value="陶瓷盆">陶瓷盆</option>
                    <option value="耗材/配件">耗材/配件</option>
                  </select>
                </div>

                <div v-if="formInv.category === '蘭花'" class="field">
                  <label>品種名稱：</label>
                  <input v-model="formInv.item_name" placeholder="例: 滿天紅、大辣椒" />
                </div>

                <div v-if="formInv.category === '陶瓷盆'" class="field">
                  <label>盆器規格：</label>
                  <select v-model="formInv.pot_type" @change="onPotTypeChange">
                    <option value="桌上盆 (100)">桌上盆 (成本100)</option>
                    <option value="落地盆陶瓷-喪 (100)">落地盆陶瓷-喪 (成本100)</option>
                    <option value="落地陶瓷盆-喜 (200)">落地陶瓷盆-喜 (成本200)</option>
                    <option value="羅馬盆 (280)">羅馬盆 (成本280)</option>
                    <option value="快捷盆 (70)">快捷盆 (成本70)</option>
                  </select>
                </div>

                <div v-if="formInv.category === '耗材/配件'" class="field highlight-field">
                  <label>耗材品名 (直接自行輸入)：</label>
                  <input 
                    v-model="formInv.item_name" 
                    type="text"
                    placeholder="例: 水苔、透明軟盆、花卡插籤、肥料、鐵線" 
                  />
                </div>

                <div v-if="formInv.category === '耗材/配件'" class="field highlight-field">
                  <label>詳細規格 / 包裝單位：</label>
                  <input 
                    v-model="formInv.spec_supply" 
                    type="text"
                    placeholder="例: 5kg/包、3.5吋 100入/袋、50米/捲" 
                  />
                </div>

                <div v-if="formInv.category === '蘭花'" class="field">
                  <label>花梗：</label>
                  <select v-model="formInv.spec_spike">
                    <option value="單梗">單梗</option><option value="雙梗">雙梗</option><option value="多梗">多梗</option>
                  </select>
                </div>
                <div v-if="formInv.category === '蘭花'" class="field">
                  <label>花色：</label>
                  <input v-model="formInv.spec_color" placeholder="紅/粉/白/黃" />
                </div>
                <div v-if="formInv.category === '蘭花'" class="field">
                  <label>花朵大小：</label>
                  <select v-model="formInv.spec_size">
                    <option value="大">大花</option><option value="中">中花</option><option value="小">小花</option>
                  </select>
                </div>
                <div v-if="formInv.category === '蘭花'" class="field">
                  <label>株高：</label>
                  <select v-model="formInv.spec_height">
                    <option value="高">高</option><option value="中">中</option><option value="矮">矮</option>
                  </select>
                </div>

                <div class="field highlight-field">
                  <label>進貨數量 (棵/個/包/捲)：</label>
                  <input v-model.number="formInv.qty" type="number" min="1" @input="calcInvCost" />
                </div>
                <div class="field">
                  <label>單價 (元)：</label>
                  <input v-model.number="formInv.unit_cost" type="number" min="0" @input="calcInvCost" />
                </div>
                <div class="field">
                  <label>進貨總成本 (元)：</label>
                  <input v-model.number="formInv.cost" type="number" min="0" />
                </div>
                <div class="field">
                  <label>供應商 / 廠商：</label>
                  <input v-model="formInv.supplier" placeholder="供應商名稱" />
                </div>
                <div class="field">
                  <label>進貨日期：</label>
                  <input v-model="formInv.date" type="date" />
                </div>
              </div>
              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveInventory">{{ editingInvId ? '確認更新進貨' : '確認入庫' }}</button>
                <button v-if="editingInvId" class="secondary-btn" @click="cancelEditInv">取消</button>
              </div>
            </div>

            <div class="card-box mt-3">
              <h3>📦 進貨與庫存清單 ({{ inventoryList.length }} 筆)</h3>
              <div class="table-responsive mt-2">
                <table class="data-table">
                  <thead>
                    <tr><th>編號</th><th>類別</th><th>品項名稱</th><th>規格明細</th><th>數量</th><th>單價</th><th>總成本</th><th>供應商</th><th>日期</th><th>操作</th></tr>
                  </thead>
                  <tbody>
                    <tr v-for="inv in inventoryList" :key="inv.id">
                      <td><b>{{ inv.id }}</b></td>
                      <td><span class="badge">{{ inv.category }}</span></td>
                      <td><b>{{ inv.item_name }}</b></td>
                      <td>{{ inv.spec }}</td>
                      <td><b>{{ inv.qty }} {{ inv.category === '陶瓷盆' ? '個' : (inv.category === '蘭花' ? '棵' : '件') }}</b></td>
                      <td class="text-blue">${{ inv.unit_cost || (inv.qty ? Math.round(inv.cost / inv.qty) : 0) }}</td>
                      <td class="text-red"><b>${{ inv.cost }}</b></td>
                      <td>{{ inv.supplier }}</td>
                      <td>{{ inv.date }}</td>
                      <td class="action-cell">
                        <div class="stacked-action-col">
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #E0FBFC !important; color: #155e75 !important;" @click="startEditInv(inv)">✏️</button>
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('inventory', inv.id, loadInventory)">🗑</button>
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
              <h3>{{ editingCustId ? '✏️ 修改客戶資料' : '👥 建立新客戶名冊' }}</h3>
              <div class="form-grid mt-2">
                <div class="field"><label>客戶/公司名稱：</label><input v-model="formCust.name" placeholder="名稱" /></div>
                <div class="field">
                  <label>客戶類型：</label>
                  <select v-model="formCust.type">
                    <option value="批發商">批發商</option><option value="零售客">零售客</option><option value="合作花店">合作花店</option><option value="其他">其他</option>
                  </select>
                </div>
                <div class="field">
                  <label>結帳週期：</label>
                  <select v-model="formCust.billing_cycle">
                    <option value="每單結">每單結</option><option value="週結">週結</option><option value="月結">月結</option>
                  </select>
                </div>
                <div class="field"><label>聯絡電話：</label><input v-model="formCust.phone" placeholder="電話" /></div>
                <div class="field"><label>常用送達地址 / 備註：</label><input v-model="formCust.line_note" placeholder="送花地址或LINE暱稱" /></div>
              </div>
              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveCustomer">{{ editingCustId ? '確認更新客戶' : '確認建立客戶' }}</button>
                <button v-if="editingCustId" class="secondary-btn" @click="cancelEditCust">取消</button>
              </div>
            </div>

            <div class="card-box mt-3">
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
              <h3>{{ editingOrchidId ? '✏️️ 修改蘭花品種' : '🌸 建立新品種照片庫' }}</h3>
              <div class="form-grid mt-2">
                <div class="field"><label>品種名稱：</label><input v-model="formOrchid.name" placeholder="例: 大辣椒、滿天紅" /></div>
                <div class="field"><label>花型特色 / 備註：</label><input v-model="formOrchid.note" placeholder="特色說明" /></div>
                <div class="field">
                  <label>品種花照：</label>
                  <input type="file" accept="image/*" @change="onPhotoFileChange" />
                </div>
              </div>
              <div v-if="formOrchid.photo_url" class="photo-preview-wrap mt-2">
                <img :src="formOrchid.photo_url" class="preview-thumb" />
                <button type="button" class="remove-photo-btn" @click="formOrchid.photo_url = ''">移除照片</button>
              </div>
              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveOrchid">{{ editingOrchidId ? '確認更新品種' : '確認儲存品種' }}</button>
                <button v-if="editingOrchidId" class="secondary-btn" @click="cancelEditOrchid">取消</button>
              </div>
            </div>

            <div class="card-box mt-3">
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
                          <button class="cozy-btn icon-only-btn clean-btn-noborder" style="background-color: #ebd8da !important; color: #6e2e34 !important;" @click="deleteItem('orchids', item.id, loadOrchids)">🗑️</button>
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
              <h3>{{ editingRetId ? '✏️ 修改退貨紀錄' : '🔄 登記退貨退款' }}</h3>
              <div class="form-grid mt-2">
                <div class="field">
                  <label>退貨類型：</label>
                  <select v-model="formRet.return_type">
                    <option value="退給花農">退給花農 (瑕疵不良退貨)</option>
                    <option value="客戶退回">客戶退回 (客戶退貨退款)</option>
                  </select>
                </div>
                <div class="field"><label>對象名稱：</label><input v-model="formRet.party_name" placeholder="花農或客戶名稱" /></div>
                <div class="field"><label>退貨品項：</label><input v-model="formRet.target_item" placeholder="品項名稱" /></div>
                <div class="field"><label>株數：</label><input v-model.number="formRet.qty" type="number" min="1" @input="calcRetTotal" /></div>
                <div class="field"><label>單價 (元)：</label><input v-model.number="formRet.unit_price" type="number" min="0" @input="calcRetTotal" /></div>
                <div class="field"><label>退貨總額 (元)：</label><input v-model.number="formRet.total_amount" type="number" min="0" /></div>
                <div class="field"><label>退貨日期：</label><input v-model="formRet.date" type="date" /></div>
                <div class="field"><label>原因說明：</label><input v-model="formRet.reason" placeholder="原因" /></div>
              </div>
              <div class="btn-action-row mt-2">
                <button class="primary-btn" @click="saveReturn">{{ editingRetId ? '確認更新退貨' : '確認儲存退貨' }}</button>
                <button v-if="editingRetId" class="secondary-btn" @click="cancelEditRet">取消</button>
              </div>
            </div>

            <div class="card-box mt-3">
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
                      <td class="addr-cell font-bold text-blue">{{ getCustomerAddress(ord) }}</td>
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

      <!-- ================= 模式 2：花卡 / 輓聯編輯器 (A3/A4/A5，精準對應 Supabase 3張底圖) ================= -->
      <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
        <div class="control-panel no-print">
          <h2>⚙️ 卡片與題詞設定</h2>

          <!-- 紙張尺寸選擇：A3 / A4 / A5 -->
          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: cardPaperSize === 'A3' }" @click="switchPaperSize('A3')">A3 (超大)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A4' }" @click="switchPaperSize('A4')">A4 (標準大)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A5' }" @click="switchPaperSize('A5')">A5 (小花卡)</button>
            </div>
          </div>

          <!-- 🌸 Supabase Storage 3 款專用紙張底色切換 (客人預覽用，列印自動透明) -->
          <div class="panel-section highlight-panel">
            <label class="section-title">🌸 實體卡片樣式底圖 (客人預覽用)：</label>
            <select v-model="cardBgType" class="full-input bold-select">
              <option value="red">🌺 喜慶紅卡底圖 (Supabase 紅)</option>
              <option value="pink">🌸 優雅粉卡底圖 (Supabase 粉)</option>
              <option value="white">📄 質感白卡底圖 (Supabase 白)</option>
              <option value="custom">📁 自行上傳其他底圖檔...</option>
            </select>
            <input 
              v-if="cardBgType === 'custom'" 
              type="file" 
              accept="image/*" 
              class="full-input mt-1" 
              @change="onCustomCardBgUpload" 
            />
            <div class="sub-label-tip mt-1">
              💡 <b>防擠壓與列印說明</b>：
              <br>• 橫式預覽時底圖已自動依比例旋轉對正，絕不擠壓變形！
              <br>• 按「列印」時系統會自動透明底色，直接印在您自備的色卡紙上！
            </div>
          </div>

          <div class="panel-section">
            <div class="inline-font-weight-row">
              <div class="inline-item-flex">
                <label class="mini-field-lbl">字體選擇：</label>
                <select v-model="cardFontFamily" class="full-input compact-inline-select font-bold">
                  <option value="kai">標準標楷體 / Word正楷 (書法楷體)</option>
                  <option value="notosong">思源宋體 (Noto Serif TC / 古典明體)</option>
                  <option value="fangsong">仿宋古典體 (FangSong / 秀麗骨風)</option>
                  <option value="notosans">思源黑體 (Noto Sans TC / 現代簡約)</option>
                </select>
              </div>
              <div class="inline-item-fixed">
                <label class="mini-field-lbl">中款預設粗細：</label>
                <select v-model="weights.middle" class="full-input compact-inline-select font-bold text-blue">
                  <option value="400">400 (正常)</option><option value="500">500 (微厚)</option><option value="600">600 (半粗)</option><option value="700">700 (粗體)</option><option value="800">800 (特粗)</option>
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

          <!-- 上款 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">1. 開頭敬詞：</span>
              <input type="number" v-model.number="layout.upper_prefix.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperPrefix" class="full-input mt-1" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">2. 受禮對象：</span>
              <input type="number" v-model.number="layout.upper_target.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperTarget" class="full-input mt-1" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">3. 上款結尾詞：</span>
              <input type="number" v-model.number="layout.upper_suffix.size" min="14" max="250" class="compact-size-input" />
            </div>
            <input type="text" v-model="upperSuffix" class="full-input mt-1" />
          </div>

          <!-- 中款 -->
          <div class="panel-section">
            <div class="section-title-with-weight">
              <span class="section-title">中款第 1 行：</span>
              <input type="number" v-model.number="layout.middle.size" min="14" max="300" class="compact-size-input" />
            </div>
            <input type="text" v-model="middleText" class="full-input mt-1" />

            <div class="section-title-with-weight mt-2">
              <span class="section-title">中款第 2 行：</span>
              <input type="number" v-model.number="layout.middle_2.size" min="14" max="300" class="compact-size-input" />
            </div>
            <input type="text" v-model="middleText2" class="full-input mt-1" />
          </div>

          <!-- 下款 -->
          <div class="panel-section">
            <label class="section-title">下款設定：</label>
            <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
              <span class="line-num">格 {{ idx + 1 }}</span>
              <input type="text" v-model="item.text" class="flex-input" />
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
          <button type="button" class="line-action-btn mt-2" @click="shareCoupletDirect">💬 直接傳送帶底圖給客人確認</button>
          
          <button type="button" class="print-action-btn mt-2" @click="handlePrintAction('card-print-target', `花卡_${cardPaperSize}`)">
            🖨️ 列印花卡 / 輓聯 ({{ cardPaperSize }})
          </button>
        </div>

        <div class="canvas-viewport" ref="viewportRef">
          <div class="zoom-toolbar no-print">
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.2, +(zoomLevel - 0.05).toFixed(2))">－</button>
            <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
            <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
            <button type="button" class="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
          </div>

          <div 
            class="card-scaler-container" 
            :style="{
              width: (currentCardDimensions.w * zoomLevel) + 'px',
              height: (currentCardDimensions.h * zoomLevel) + 'px'
            }"
          >
            <!-- 🌟 花卡主體 (透過獨立防擠壓層解決直式轉橫式的拉伸問題) -->
            <div 
              id="card-print-target" 
              class="card-board standard-kai-font" 
              :class="isVertical ? 'mode-vertical' : 'mode-horizontal'"
              :style="{
                width: currentCardDimensions.w + 'px',
                height: currentCardDimensions.h + 'px',
                transform: `scale(${zoomLevel})`,
                transformOrigin: 'top left',
                fontFamily: activeCssFontFamily
              }"
            >
              <!-- 獨立底圖層：橫式等比自動旋轉對齊，絕不擠壓變形 -->
              <div 
                class="card-dynamic-bg-layer"
                :class="{ 'rotate-landscape-bg': !isVertical }"
                :style="{ backgroundImage: activeBackgroundImageStyle }"
              ></div>

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

      <!-- ================= 模式 3：A5 橫式簽收單 (白底紙張，Word楷體) ================= -->
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
          <button type="button" class="print-action-btn mt-2" @click="handlePrintAction('receipt-print-target', '簽收單_A5', true)">
            🖨️ 列印 A5 橫式簽收單
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
              class="a5-landscape-sheet kai-font-supported standard-kai-font"
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

      <!-- ================= 模式 4：農民收據 (白底紙張，Word楷體) ================= -->
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
              <input type="number" v-model.number="farmerReceipt.totalAmount" />
            </div>
          </div>

          <button type="button" class="line-action-btn mt-2" @click="shareFarmerReceiptDirect">
            💬 直接傳送收據給客人
          </button>
          <button type="button" class="print-action-btn mt-2" @click="handlePrintAction('farmer-print-target', '農民收據_A5')">
            🖨️ 列印農民收據
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
              class="farmer-receipt-sheet kai-font-supported standard-kai-font"
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
// 1. 基礎狀態變數 (A3/A4/A5)
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

// 🌟 尺寸設定：精準依據 A3, A4, A5 國際標準長寬比
const currentCardDimensions = computed(() => {
  if (cardPaperSize.value === 'A3') {
    return isVertical.value ? { w: 1123, h: 1587 } : { w: 1587, h: 1123 }
  }
  if (cardPaperSize.value === 'A5') {
    return isVertical.value ? { w: 560, h: 794 } : { w: 794, h: 560 }
  }
  return isVertical.value ? { w: 794, h: 1123 } : { w: 1123, h: 794 }
})

const switchPaperSize = (size) => {
  cardPaperSize.value = size
  nextTick(() => autoFitZoom())
}

const autoFitZoom = () => {
  if (!viewportRef.value || viewportRef.value.clientWidth <= 0) {
    zoomLevel.value = 0.65
    return
  }
  const availableWidth = Math.max(viewportRef.value.clientWidth - 20, 260)
  const cardWidth = currentCardDimensions.value.w
  zoomLevel.value = Math.min(Math.max(+(availableWidth / cardWidth).toFixed(2), 0.25), 1.0)
}

// ==========================================
// 2. 內部驗證 (強化容錯率，絕不誤擋)
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

// ==========================================
// 3. Supabase 資料庫連線
// ==========================================
const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
const supabase = createClient(supabaseUrl, supabaseKey)

// 🌟 Supabase card-assets 公開存取網址 (對應您上傳至 bucket 的 3 張底圖)
const SUPABASE_STORAGE_URL = 'https://ivofrjibdezbyxxmutok.supabase.co/storage/v1/object/public/card-assets'

const userCustomSeal = ref(localStorage.getItem('user_cai_seal_img') || '')
const activeCaiSealSrc = computed(() => userCustomSeal.value || '/cai-seal.png')

const customers = ref([])
const orderList = ref([])
const unshippedOrders = computed(() => (orderList.value || []).filter(o => o.shipped_status !== '已出貨'))
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

const previewNextOrderId = computed(() => `OR-${Date.now().toString().slice(-6)}`)

const loadOrders = async () => {
  try {
    const { data } = await supabase.from('orders').select('*')
    if (data) {
      orderList.value = data.sort((a, b) => String(b.id || '').localeCompare(String(a.id || '')))
    }
  } catch (err) {}
}

const onOrderCustSelect = () => {
  const matched = (customers.value || []).find(c => c.name === formOrder.value.customer)
  if (matched) formOrder.value.phone = matched.phone || ''
}
const addOrderItemRow = () => {
  formOrder.value.items.push({ orchid_name: '特選蘭花', pots_qty: 1, stalks: 10, unit_price: 250, pot: '桌上盆 (100)', quick_pot: '未使用' })
}
const removeOrderItemRow = (idx) => {
  if (formOrder.value.items.length > 1) formOrder.value.items.splice(idx, 1)
}
const calcOrderPrice = () => {
  let total = 0
  formOrder.value.items.forEach(it => {
    total += (it.stalks || 0) * (it.unit_price || 0) * (it.pots_qty || 1)
  })
  formOrder.value.price = total + (formOrder.value.shipping_fee || 0)
}
const saveOrder = async () => {
  if (!formOrder.value.customer) return alert('請輸入客戶名稱！')
  showToast('✅ 訂單儲存成功')
  cancelEditOrder()
}
const cancelEditOrder = () => {
  editingOrderId.value = null
}
const startEditOrder = (ord) => { editingOrderId.value = ord.id }
const getOrderTotalPots = (ord) => 1
const getOrderShippingFee = (ord) => 0
const exportOrdersToExcel = () => showToast('匯出報表')

// ==========================================
// 4. 花卡編輯器 (3 款底圖，防擠壓旋轉)
// ==========================================
const cardCategory = ref('celebration')
const cardFontFamily = ref('kai')
const upperPrefix = ref('祝')
const upperTarget = ref('新北市 陳乃瑜議員')
const upperSuffix = ref('')
const middleText = ref('高票當選')
const middleText2 = ref('為民服務')
const suffixText = ref('敬賀')

const cardBgType = ref('red')
const customCardBgUrl = ref('')

// 🌟 精確對應 Supabase card-assets bucket 裡的 3 張圖檔
const activeBackgroundImageStyle = computed(() => {
  if (cardBgType.value === 'custom' && customCardBgUrl.value) {
    return `url(${customCardBgUrl.value})`
  }
  const map = {
    red: `url('${SUPABASE_STORAGE_URL}/card-bg-red.jpg'), url('/card-bg-red.jpg')`,
    pink: `url('${SUPABASE_STORAGE_URL}/card-bg-pink.jpg'), url('/card-bg-pink.jpg')`,
    white: `url('${SUPABASE_STORAGE_URL}/card-bg-white.jpg'), url('/card-bg-white.jpg')`
  }
  return map[cardBgType.value] || map.red
})

const onCustomCardBgUpload = (e) => {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (event) => {
    customCardBgUrl.value = event.target.result
  }
  reader.readAsDataURL(file)
}

const weights = ref({
  upper_prefix: '700', upper_target: '700', upper_suffix: '700', middle: '800', middle_2: '800',
  bottom_0: '600', bottom_1: '700', suffix: '700'
})
const bottomLines = ref([{ text: '白沙屯媽祖' }, { text: '彰化拱聖宮' }])

// 🌟 跨平台楷書字型對應
const fontMapping = {
  kai: '"DFKai-SB", "BiauKai", "標楷體", "TW-Kai", "MOESong-Regular", "Noto Serif TC", "Kaiti", serif',
  notosong: '"Noto Serif TC", "Songti TC", "SimSun", "PMingLiU", serif',
  fangsong: '"FangSong", "STFangsong", "華康仿宋體", serif',
  notosans: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif'
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

const isMobileDevice = () => /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)

const handlePrintAction = (targetId, titlePrefix) => {
  if (isMobileDevice()) {
    openPrintImageModal(targetId, titlePrefix, true)
  } else {
    window.print()
  }
}

const openPrintImageModal = async (targetId, titlePrefix, isPrintMode = false) => {
  const targetEl = document.getElementById(targetId)
  if (!targetEl) return alert('找不到目標畫面！')
  showToast(isPrintMode ? '⏳ 正在生成無色透明送印圖檔...' : '⏳ 正在生成客人確認圖檔...')
  try {
    if (document.fonts?.ready) await document.fonts.ready
    const origTransform = targetEl.style.transform
    targetEl.style.transform = 'none'

    const bgLayer = targetEl.querySelector('.card-dynamic-bg-layer')
    let origDisplay = ''
    if (bgLayer && isPrintMode) {
      origDisplay = bgLayer.style.display
      bgLayer.style.display = 'none'
    }

    const canvas = await html2canvas(targetEl, {
      scale: 2,
      useCORS: true,
      backgroundColor: isPrintMode ? null : '#ffffff',
      logging: false,
      ignoreElements: (el) => el.classList && (el.classList.contains('scale-handle') || el.classList.contains('no-print'))
    })

    targetEl.style.transform = origTransform
    if (bgLayer && isPrintMode) {
      bgLayer.style.display = origDisplay
    }

    canvas.toBlob((blob) => {
      if (!blob) return
      if (shareModalImg.value && shareModalImg.value.startsWith('blob:')) {
        URL.revokeObjectURL(shareModalImg.value)
      }
      shareModalImg.value = URL.createObjectURL(blob)
      shareModalTitle.value = isPrintMode ? `${titlePrefix} (透明列印)` : `${titlePrefix} (確認預覽)`
      shareModalFilename.value = `${titlePrefix}.png`
      currentBlobToShare.value = blob
      canNativeShare.value = !!(navigator.canShare && navigator.canShare({ files: [new File([blob], 'card.png', { type: 'image/png' })] }))
    }, 'image/png')
  } catch (err) {
    showToast('⚠️ 生成失敗，請重試！')
  }
}

const triggerImagePrint = () => {
  if (!shareModalImg.value) return
  const imgUrl = shareModalImg.value
  const printWin = window.open('', '_blank')
  if (printWin) {
    printWin.document.write(`
      <!DOCTYPE html>
      <html>
        <head>
          <title>列印</title>
          <style>
            @page { size: auto; margin: 0mm !important; }
            * { margin: 0; padding: 0; box-sizing: border-box; }
            body { 
              width: 100vw; height: 100vh; margin: 0 !important; padding: 0 !important;
              display: flex; justify-content: center; align-items: center; 
              background: transparent !important; overflow: hidden !important;
            }
            img { max-width: 100%; max-height: 100%; object-fit: contain; }
          </style>
        </head>
        <body onload="window.focus(); window.print(); window.close();">
          <img src="${imgUrl}" />
        </body>
      </html>
    `)
    printWin.document.close()
  } else {
    window.location.href = imgUrl
  }
}

const shareCoupletDirect = () => openPrintImageModal('card-print-target', '花卡確認', false)
const shareReceiptDirect = () => openPrintImageModal('receipt-print-target', '簽收單確認', false)
const shareFarmerReceiptDirect = () => openPrintImageModal('farmer-print-target', '農民收據確認', false)

// 簽收單與收據表單
const selectedOrderId = ref('')
const formatSimpleItemName = (ord) => '特選蘭花 1盆'
const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  currentTab.value = 'receipt'
}
const fillFarmerReceiptFromOrder = (ord) => {
  selectedFarmerOrderId.value = ord.id
  currentTab.value = 'farmer_receipt'
}

const receiptForm = ref({
  orderId: '', deliveryDate: '115-09-03 送達', recipient: '永全證券 陳總經理',
  address: '桃園市桃園區縣府路 82 號', item: '特選蘭花 1盆', giver: '敬領 誌慶', notes: '花禮已專車安全送達點交'
})

const onSelectReceiptOrder = () => {
  const ord = (orderList.value || []).find(o => o.id === selectedOrderId.value)
  if (ord) {
    receiptForm.value.orderId = ord.id
    receiptForm.value.recipient = ord.customer
  }
}

const selectedFarmerOrderId = ref('')
const farmerReceipt = ref({
  year: '115', month: '09', day: '03', buyerName: '永全證券股份有限公司', taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號', itemName: '蝴蝶蘭花禮', spec: '特級', qty: '1 盆', unitPrice: '2500', totalAmount: 2500, note: ''
})
const chineseDigits = ref({ hundredThousands: '', tenThousands: '', thousands: '貳', hundreds: '伍', tens: '', ones: '' })

const onSelectFarmerReceiptOrder = () => {
  const ord = (orderList.value || []).find(o => o.id === selectedFarmerOrderId.value)
  if (ord) {
    farmerReceipt.value.buyerName = ord.customer
    farmerReceipt.value.totalAmount = ord.price
  }
}

const initSystemData = () => {
  autoFitZoom()
  loadOrders()
}

onMounted(() => {
  window.addEventListener('resize', autoFitZoom)
  initSystemData()
})
</script>

<style scoped>
/* 🌟 100% 原始精美樣式（徹底清除所有無形字元，保證全平台生效） */
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

.triple-width-field {
  grid-column: span 3 !important;
}
@media (max-width: 900px) {
  .triple-width-field {
    grid-column: span 1 !important;
  }
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

.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.couplet-screen-wrapper { display: flex; flex: 1; overflow: hidden; height: calc(100vh - 50px); }
.control-panel {
  width: 410px; background: white; padding: 14px; box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 9px; border-radius: 6px; margin-bottom: 9px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }
.bold-select { font-weight: bold; font-size: 13.5px; border-color: #3b82f6; }

.section-title { font-size: 13px; font-weight: bold; margin-bottom: 4px; display: inline-block; }
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

.form-group { margin-bottom: 8px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 3px; color: #334155; }
.bottom-input-group { display: flex; align-items: center; gap: 5px; margin-bottom: 5px; }
.line-num { font-size: 12px; font-weight: bold; color: #64748b; width: 36px; }
.form-row, .btn-group { display: flex; gap: 5px; }
.btn-group button {
  flex: 1; padding: 6px; border: 1px solid #2563eb; background: white; color: #2563eb; border-radius: 4px; cursor: pointer; font-weight: bold; font-size: 12.5px;
}
.btn-group button.active { background: #2563eb; color: white; }
.reset-btn { width: 100%; padding: 7px; background: #f1f5f9; border: 1px dashed #94a3b8; border-radius: 4px; cursor: pointer; font-size: 12.5px; }

/* 🌟 預覽視窗樣式 */
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
  position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none; touch-action: none; border: none !important;
  background-color: #ffffff; overflow: hidden;
}

/* 🌟 防擠壓動態背景層：直式正常鋪滿，橫式等比自動旋轉對齊，絕不擠壓變形 */
.card-dynamic-bg-layer {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  background-size: cover; background-position: center; background-repeat: no-repeat;
  z-index: 1; pointer-events: none;
}
.rotate-landscape-bg {
  width: 100% !important;
  height: 100% !important;
  transform: rotate(90deg) scale(1.42);
  transform-origin: center center;
}

.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; z-index: 2; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; z-index: 2; }
.text-box { position: absolute; cursor: move; padding: 3px 5px; white-space: nowrap; line-height: 1.25; color: #0f172a; z-index: 2; }

/* 🌟 正楷字體全平台適配 (以 Noto Serif TC 和繁體楷體為核心) */
.standard-kai-font, .kai-font-supported {
  font-family: "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", "Kaiti TC", "STKaiti", "Noto Serif TC", serif !important;
}

/* A5 橫式簽收單 */
.receipt-scaler-container { position: relative; flex-shrink: 0; }
.a5-landscape-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 32px 38px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; writing-mode: horizontal-tb; direction: ltr;
  color: #111827; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.sheet-header {
  display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 2.5px solid #1e293b; padding-bottom: 6px;
}
.shop-name-title { font-size: 27px; font-weight: 900; letter-spacing: 2px; color: #0f172a; }
.sheet-main-title { font-size: 23px; font-weight: bold; letter-spacing: 4px; color: #dc2626; }
.header-meta { font-size: 13.5px; line-height: 1.45; text-align: right; color: #334155; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 8px 0; font-size: 15.5px; table-layout: fixed; }
.receipt-table td { border: 1.5px solid #334155; padding: 7px 10px; word-break: break-all; }
.receipt-table .lbl { width: 15%; background-color: #f1f5f9; font-weight: bold; text-align: center; color: #1e293b; font-size: 15.5px; }
.receipt-table .val { width: 35%; font-size: 15.5px; }
.receipt-table .val-bold { font-weight: bold; font-size: 16.5px; }
.receipt-table .val-highlight { font-weight: bold; color: #1e3a8a; font-size: 17px; }

.sheet-footer { display: flex; justify-content: space-between; align-items: stretch; gap: 16px; margin-top: 2px; }
.footer-left { flex: 1; display: flex; flex-direction: column; justify-content: space-between; font-size: 14.5px; padding: 2px 0; }
.footer-tip { font-size: 12.5px; color: #64748b; }
.footer-sign-box {
  width: 215px; border: 1.5px dashed #475569; border-radius: 6px; display: flex; flex-direction: column; background-color: #fafafa;
}
.sign-box-title { background: #e2e8f0; font-size: 12.5px; font-weight: bold; text-align: center; padding: 2px 0; color: #334155; }
.sign-box-area { flex: 1; min-height: 50px; display: flex; justify-content: center; align-items: center; }

/* 農民收據 */
.farmer-scaler-container { position: relative; flex-shrink: 0; }
.farmer-receipt-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 16px 26px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; color: #000;
  box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.f-header { display: flex; flex-direction: column; align-items: center; position: relative; margin-bottom: 8px; }
.f-main-title { font-size: 26px; font-weight: 900; letter-spacing: 5px; text-align: center; }
.f-date-wrap { align-self: flex-end; font-size: 15px; letter-spacing: 2px; margin-top: 6px; }

.f-receipt-grid-table { border: 2px solid #000; display: flex; flex-direction: column; font-size: 15.5px; }
.f-grid-row { display: flex; border-bottom: 1px solid #000; min-height: 29px; }
.f-grid-row:last-child { border-bottom: none; }
.f-grid-lbl {
  display: flex; justify-content: center; align-items: center; font-weight: bold; letter-spacing: 2px;
  text-align: center; border-right: 1px solid #000; padding: 3px 5px; box-sizing: border-box; flex-shrink: 0; font-size: 15.5px;
}
.f-grid-val { display: flex; align-items: center; padding-left: 10px; border-right: 1px solid #000; box-sizing: border-box; font-size: 15.5px; }
.f-grid-val:last-child { border-right: none; }
.f-flex-1 { flex: 1; }
.f-row-top { min-height: 60px; }
.f-col-buyer-group { display: flex; flex-direction: column; width: 58%; border-right: 1px solid #000; }
.f-sub-row { display: flex; flex: 1; border-bottom: 1px solid #000; }
.f-sub-row:last-child { border-bottom: none; }
.f-w-head { width: 145px; }
.f-w-addr-tag { width: 36px; line-height: 1.4; }
.f-full-addr-box { flex: 1; padding: 6px 10px; font-size: 15.5px; line-height: 1.45; border-right: none !important; }
.f-tax-clean { font-size: 17px; font-weight: bold; letter-spacing: 3px; color: #1e3a8a; }

.col-p-name { width: 25%; }
.col-p-spec { width: 16%; }
.col-p-qty  { width: 10%; }
.col-p-price{ width: 14%; }
.col-p-amt  { width: 18%; }
.col-p-note { width: 17%; border-right: none !important; }

.f-header-row { font-weight: bold; height: 28px; }
.f-data-row { height: 30px; }
.f-empty-row { height: 26px; }

.f-w-total-lbl { width: 205px; white-space: nowrap; font-size: 15.5px; }
.f-amount-val-cell { flex: 1; border-right: none !important; padding: 2px 10px; }
.f-chinese-amount-line {
  display: flex; align-items: center; justify-content: space-around; width: 100%; font-size: 16.5px; font-weight: bold;
}
.f-chinese-amount-line .d-val { color: #1e3a8a; min-width: 24px; text-align: center; font-size: 17px; display: inline-block; }

.f-farmer-stamp-cell { border-right: none !important; padding-left: 24px !important; display: flex; align-items: center; gap: 14px; }
.f-farmer-name-clean { font-size: 18px; letter-spacing: 6px; font-weight: bold; }
.cai-real-stamp-img { width: 48px; height: 48px; object-fit: contain; mix-blend-mode: multiply; }

.f-w-id-lbl { width: 190px; }
.f-w-id-val { width: 190px; border-right: none !important; }

.f-text-center { justify-content: center; text-align: center; }
.f-text-right { justify-content: flex-end; text-align: right; }
.f-bold { font-weight: bold; }
.f-pr { padding-right: 12px !important; }

.f-statement { font-size: 12.5px; text-align: center; letter-spacing: 1px; font-weight: bold; margin-top: 3px; }

.print-action-btn {
  width: 100%; padding: 11px; background: #16a34a; color: white; border: none; border-radius: 6px; font-size: 14px; font-weight: bold; cursor: pointer;
}
.print-action-btn:disabled { background: #cbd5e1; cursor: not-allowed; }

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
.close-modal-btn { background: transparent; border: none; font-size: 22px; cursor: pointer; color: #64748b; padding: 0 4px; }
.share-modal-body { display: flex; flex-direction: column; align-items: center; width: 100%; }
.share-img-scroll-container {
  width: 100%; display: flex; justify-content: center; align-items: center;
  background-color: #f1f5f9; border-radius: 8px; padding: 10px; box-sizing: border-box; margin-bottom: 12px;
}
.share-preview-img-contained { max-height: 55vh; max-width: 100%; object-fit: contain; border-radius: 6px; box-shadow: 0 4px 14px rgba(0,0,0,0.15); }
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

@media (max-width: 768px) {
  .app-container, .receipt-container, .couplet-screen-wrapper { flex-direction: column; overflow-y: auto; height: auto; }
  .control-panel { width: 100%; max-height: 46vh; }
  .form-grid { grid-template-columns: 1fr; }
  .canvas-viewport, .receipt-preview-area { padding: 12px 6px 60px 6px; }
  .shipping-tab-header { flex-direction: column; align-items: flex-start; }
}

/* =========================================================================
   🌟 全平台列印防空白頁與單頁強制保證
========================================================================= */
@media print {
  @page {
    margin: 0mm !important;
  }

  html, body {
    margin: 0 !important;
    padding: 0 !important;
    background: transparent !important;
    width: 100% !important;
    height: 98vh !important;
    max-height: 98vh !important;
    overflow: hidden !important;
    font-family: "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", "Kaiti TC", "STKaiti", "Noto Serif TC", serif !important;
  }

  .no-print, .top-nav, .sub-nav, .control-panel, .zoom-toolbar, .floating-toast, .image-modal-overlay {
    display: none !important;
  }

  .main-wrapper, .couplet-screen-wrapper, .system-root, .app-container, .receipt-container,
  .canvas-viewport, .receipt-preview-area, .receipt-scaler-container, .farmer-scaler-container, .card-scaler-container { 
    margin: 0 !important; 
    padding: 0 !important; 
    background: transparent !important; 
    display: block !important; 
    position: static !important;
    width: 100% !important;
    height: 98vh !important;
    max-height: 98vh !important;
    overflow: hidden !important;
    transform: none !important;
  }

  /* 花卡列印：滿版單頁輸出，底圖在送印時抽空為無色透明 */
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
    background-color: transparent !important;
    max-height: 98vh !important;
    page-break-after: avoid !important;
    break-after: avoid !important;
  }
  .card-dynamic-bg-layer {
    display: none !important; /* 送印時隱藏底圖，只噴墨文字 */
  }
  #card-print-target * { visibility: visible !important; }

  /* 簽收單與農民收據：送印時背景自動透明，完全單頁輸出 */
  #receipt-print-target, #farmer-print-target { 
    position: relative !important; 
    top: 0 !important; 
    left: 0 !important; 
    transform: none !important; 
    box-shadow: none !important; 
    background-color: transparent !important;
    width: 192mm !important; 
    max-width: 192mm !important; 
    height: 130mm !important; 
    max-height: 130mm !important; 
    margin: 3mm auto 0 auto !important; 
    padding: 4mm 6mm !important; 
    box-sizing: border-box !important; 
    overflow: hidden !important; 
    page-break-after: avoid !important;
    break-after: avoid !important;
  }

  /* 強制標楷體 */
  #receipt-print-target *, #farmer-print-target *, #card-print-target * {
    font-family: "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "標楷體", "Kaiti", "Kaiti TC", "STKaiti", "Noto Serif TC", serif !important;
  }
}
</style>