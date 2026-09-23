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
          📄 訂單 A5 簽收單 (支援線上簽名)
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
      </nav>

      <div class="manage-content">
        <!-- 模組 1：訂單與帳務 -->
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
                    <th>單號</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>總盆數</th><th>規格明細</th><th>運費</th><th>總售價</th><th>花卡</th><th>簽收單</th><th>出貨</th><th>收款</th><th>操作</th>
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
                    <td><span class="badge badge-purple"><b>{{ getOrderTotalPots(ord) }} 盆</b></span></td>
                    <td>{{ ord.spec }}</td>
                    <td>{{ getOrderShippingFee(ord) > 0 ? '$' + getOrderShippingFee(ord) : '免運' }}</td>
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
                  <tr v-if="orderList.length === 0"><td colspan="13" class="text-center">尚無訂單資料</td></tr>
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
                  <option value="">-- 請選擇客戶 (全部未結) --</option>
                  <option v-for="c in customers" :key="c.id" :value="c.name">
                    {{ c.name }} ({{ c.type }} / {{ c.billing_cycle || '每單結' }})
                  </option>
                </select>
              </div>
              <div class="field">
                <label>統計期間：</label>
                <select v-model="statementPeriod">
                  <option value="all">全部歷史未結</option>
                  <option value="thisWeek">本週 (週一至週日)</option>
                  <option value="thisMonth">本月 (1日至今)</option>
                  <option value="lastMonth">上月全月</option>
                </select>
              </div>
              <div class="field">
                <label>收款狀態篩選：</label>
                <select v-model="statementPaymentFilter">
                  <option value="未結">僅顯示未結帳款 (對帳用)</option>
                  <option value="all">顯示全部 (含已結)</option>
                </select>
              </div>
            </div>

            <div class="statement-summary-cards mt-3">
              <div class="sum-card red-card">
                <div class="sum-label">對帳總金額</div>
                <div class="sum-value">${{ statementTotalAmount.toLocaleString() }} 元</div>
              </div>
              <div class="sum-card blue-card">
                <div class="sum-label">訂單筆數</div>
                <div class="sum-value">{{ statementOrders.length }} 筆</div>
              </div>
              <div class="sum-card green-card">
                <div class="sum-label">客戶週期 / 類別</div>
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
                    <th>單號</th><th>下單日</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>規格明細</th><th>金額</th><th>花卡</th><th>簽收單</th><th>收款狀態</th><th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="ord in statementOrders" :key="ord.id">
                    <td><b>{{ ord.id }}</b></td>
                    <td>{{ ord.order_date }}</td>
                    <td>{{ ord.customer }}</td>
                    <td class="text-purple"><b>{{ ord.tax_id || '—' }}</b></td>
                    <td>{{ ord.need_receipt || '不需收據' }}</td>
                    <td>{{ ord.spec }}</td>
                    <td class="text-blue"><b>${{ ord.price }}</b></td>
                    <td><span class="status-tag">{{ ord.card_status || '未製作' }}</span></td>
                    <td>
                      <span :class="ord.receipt_status === '已列印' ? 'badge badge-green' : 'badge badge-orange'">
                        {{ ord.receipt_status || '未列印' }}
                      </span>
                    </td>
                    <td>
                      <span :class="ord.payment_status === '未結' ? 'badge badge-red' : 'badge badge-green'">
                        {{ ord.payment_status }}
                      </span>
                    </td>
                    <td>
                      <button class="mini-btn print-btn" @click="fillReceiptFromOrder(ord)">🖨️ 簽收單</button>
                      <button class="mini-btn farmer-btn" @click="fillFarmerReceiptFromOrder(ord)">🧾 農民收據</button>
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
                      <button class="mini-btn edit-btn" @click="startEditInv(inv)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('inventory', inv.id, loadInventory)" title="刪除">🗑️</button>
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
                      <button class="mini-btn edit-btn" @click="startEditCust(c)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('customers', c.id, loadCustomers)" title="刪除">🗑️</button>
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
                      <button class="mini-btn edit-btn" @click="startEditOrchid(item)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('orchids', item.id, loadOrchids)" title="刪除">🗑️</button>
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
                      <button class="mini-btn edit-btn" @click="startEditRet(ret)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('returns', ret.id, loadReturns)" title="刪除">🗑️</button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>
      </div>
    </div>

    <!-- 照片放大燈箱 -->
    <div v-if="activeModalPhoto" class="image-modal-overlay" @click="activeModalPhoto = null">
      <div class="image-modal-content" @click.stop>
        <div class="image-modal-header">
          <span>🌸 {{ activeModalTitle }}</span>
          <button class="close-modal-btn" @click="activeModalPhoto = null">✕</button>
        </div>
        <img :src="activeModalPhoto" class="image-modal-img" alt="放大照片" />
      </div>
    </div>

    <!-- ================= 模式 2：花卡 / 輓聯編輯器 ================= -->
    <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
      <div class="control-panel no-print">
        <h2>⚙️ 卡片與題詞設定</h2>

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

        <!-- 上款獨立設定 -->
        <div class="panel-section">
          <div class="section-title-with-weight">
            <label class="section-title">1. 開頭敬詞（獨立一格）：</label>
            <select v-model="weights.upper_prefix" class="mini-weight-select" title="設定開頭敬詞粗細">
              <option value="400">400 (標準)</option>
              <option value="500">500 (中等)</option>
              <option value="600">600 (半粗)</option>
              <option value="700">700 (粗體)</option>
              <option value="800">800 (特粗)</option>
            </select>
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
              <option value="恭祝">恭祝</option>
              <option value="恭賀">恭賀</option>
              <option value="敬賀">敬賀</option>
            </select>
          </div>

          <div class="section-title-with-weight mt-2">
            <label class="section-title">2. 受禮對象 / 稱謂（獨立一格）：</label>
            <select v-model="weights.upper_target" class="mini-weight-select" title="設定稱謂粗細">
              <option value="400">400 (標準)</option>
              <option value="500">500 (中等)</option>
              <option value="600">600 (半粗)</option>
              <option value="700">700 (粗體)</option>
              <option value="800">800 (特粗)</option>
            </select>
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
            <label class="section-title">3. 上款結尾詞（獨立一格）：</label>
            <select v-model="weights.upper_suffix" class="mini-weight-select" title="設定結尾詞粗細">
              <option value="400">400 (標準)</option>
              <option value="500">500 (中等)</option>
              <option value="600">600 (半粗)</option>
              <option value="700">700 (粗體)</option>
              <option value="800">800 (特粗)</option>
            </select>
          </div>
          <div class="form-row">
            <input type="text" v-model="upperSuffix" class="full-input" placeholder="例: 千古、仙逝、靈前" />
            <select v-if="cardCategory === 'funeral'" v-model="upperSuffix" style="width: 110px;">
              <option value="千古">千古</option>
              <option value="仙逝">仙逝</option>
              <option value="靈前">靈前</option>
              <option value="冥前">冥前</option>
              <option value="便覽">便覽</option>
              <option value="淑靈">淑靈</option>
              <option value="蓮前">蓮前</option>
            </select>
            <select v-else v-model="upperSuffix" style="width: 110px;">
              <option value="誌慶">誌慶</option>
              <option value="大吉">大吉</option>
              <option value="惠存">惠存</option>
              <option value="雅存">雅存</option>
            </select>
          </div>
        </div>

        <!-- 中款獨立設定 -->
        <div class="panel-section">
          <div class="section-title-with-weight">
            <label class="section-title">中款詞語：</label>
            <select v-model="weights.middle" class="mini-weight-select" title="設定中款粗細">
              <option value="400">400 (標準)</option>
              <option value="500">500 (中等)</option>
              <option value="600">600 (半粗)</option>
              <option value="700">700 (粗體)</option>
              <option value="800">800 (特粗)</option>
            </select>
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

          <input type="text" v-model="middleText" class="full-input mt-2" placeholder="中款詞語" />
        </div>

        <!-- 下款 6 格獨立粗細設定 -->
        <div class="panel-section">
          <label class="section-title">下款設定（每個格子可個別選擇粗細）：</label>
          <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
            <span class="line-num">格 {{ idx + 1 }}</span>
            <input type="text" v-model="item.text" :placeholder="getPlaceholder(idx)" class="flex-input" />
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
              <select v-model="weights.suffix" class="mini-weight-select" title="設定敬詞粗細">
                <option value="400">400 (標準)</option>
                <option value="500">500 (中等)</option>
                <option value="600">600 (半粗)</option>
                <option value="700">700 (粗體)</option>
                <option value="800">800 (特粗)</option>
              </select>
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
        <button type="button" class="line-action-btn mt-2" @click="shareCoupletToLineDirect">💬 直接傳送 / 複製花卡給客人 (免下載)</button>
        <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印 A4 花卡 / 輓聯</button>
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
            width: (isVertical ? 794 : 1123) * zoomLevel + 'px',
            height: (isVertical ? 1123 : 794) * zoomLevel + 'px'
          }"
        >
          <div 
            id="card-print-target" 
            class="card-board" 
            :class="[
              isVertical ? 'mode-vertical' : 'mode-horizontal',
              !isVertical && cardCategory === 'celebration' ? 'style-floral' : ''
            ]"
            :style="{
              transform: `scale(${zoomLevel})`,
              transformOrigin: 'top left',
              fontFamily: activeCssFontFamily
            }"
          >
            <!-- 1. 上款開頭敬語 (獨立一格) -->
            <div 
              v-if="upperPrefix.trim()"
              class="text-box upper-prefix-box"
              :style="getStyle('upper_prefix')"
              @pointerdown="startMove($event, 'upper_prefix')"
            >
              <span>{{ upperPrefix }}</span>
              <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_prefix')">⤡</div>
            </div>

            <!-- 2. 受禮對象/稱謂 (獨立一格) -->
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

            <!-- 3. 上款結尾敬詞 (獨立一格) -->
            <div 
              v-if="upperSuffix.trim()"
              class="text-box upper-suffix-box"
              :style="getStyle('upper_suffix')"
              @pointerdown="startMove($event, 'upper_suffix')"
            >
              <span>{{ upperSuffix }}</span>
              <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper_suffix')">⤡</div>
            </div>

            <!-- 中款 -->
            <div 
              v-if="middleText.trim()"
              class="text-box middle-box"
              :style="getStyle('middle')"
              @pointerdown="startMove($event, 'middle')"
            >
              <span>{{ middleText }}</span>
              <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
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

    <!-- ================= 模式 3：A5 橫式簽收單 (線上簽名 ＋ 簽好直接回傳下單人) ================= -->
    <div v-else-if="currentTab === 'receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>📋 橫式 A5 簽收單管理</h2>

        <!-- 快速帶入 -->
        <div class="panel-section highlight-panel">
          <label class="section-title">依訂單編號快速帶入：</label>
          <select v-model="selectedOrderId" @change="onSelectReceiptOrder" class="full-input bold-select">
            <option value="">-- 請下拉選擇訂單 (即時自動帶入) --</option>
            <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
              【{{ ord.id }}】{{ ord.customer }} - {{ formatSimpleItemName(ord) }} [{{ ord.receipt_status || '未列印' }}]
            </option>
          </select>
        </div>

        <!-- 現場客戶手機/手寫線上簽名區 -->
        <div class="panel-section live-sign-panel">
          <div class="section-title-with-weight">
            <label class="section-title">✍️ 收件人線上簽名板 (送達現場簽名)：</label>
            <button type="button" class="mini-clean-btn" @click="clearLiveSignature">✕ 清除重簽</button>
          </div>
          <div class="canvas-sign-wrapper">
            <canvas 
              ref="signPadCanvasRef" 
              class="live-sign-pad" 
              width="340" 
              height="110"
              @pointerdown="startSign"
              @pointermove="drawingSign"
              @pointerup="stopSign"
              @pointerleave="stopSign"
            ></canvas>
          </div>
          <div class="sign-hint-text">※ 收件人直接在上方白色框中手寫簽名，簽完自動帶入簽收單！</div>
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

        <!-- 一鍵傳送給下單客戶（含收件人簽名） -->
        <button 
          type="button" 
          class="line-action-btn mt-2" 
          @click="shareReceiptToBuyerDirect"
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

      <!-- 右側預覽區 (現場簽名會即時顯示於簽名欄) -->
      <div class="receipt-preview-area" ref="receiptViewportRef">
        <div class="zoom-toolbar no-print">
          <button type="button" class="zoom-btn" @click="receiptZoom = Math.max(0.3, +(receiptZoom - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(receiptZoom * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="receiptZoom = Math.min(1.1, +(receiptZoom + 0.05).toFixed(2))">＋</button>
          <button type="fit-btn" @click="autoFitReceipt">📱 適配螢幕</button>
        </div>

        <div 
          class="receipt-scaler-container" 
          :style="{
            width: (794 * receiptZoom) + 'px',
            height: (560 * receiptZoom) + 'px'
          }"
        >
          <div 
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

    <!-- ================= 模式 4：農民收據 ================= -->
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
            <label>總金額 (元)：</label>
            <input type="number" v-model.number="farmerReceipt.totalAmount" @input="updateChineseAmount" />
          </div>

          <div class="form-group">
            <label>備註：</label>
            <input type="text" v-model="farmerReceipt.note" />
          </div>
        </div>

        <button 
          type="button" 
          class="line-action-btn mt-2" 
          @click="shareFarmerReceiptToLineDirect"
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

              <!-- 蔡鎮遠姓名與真實蓋章圖片排版 -->
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
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from 'vue'
import { createClient } from '@supabase/supabase-js'
import * as XLSX from 'xlsx'

// 浮動提示
const toastMessage = ref('')
let toastTimer = null
const showToast = (msg) => {
  toastMessage.value = msg
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => {
    toastMessage.value = ''
  }, 4500)
}

// 圖片分享彈窗
const shareModalImg = ref('')
const shareModalTitle = ref('')

// ----------------- Supabase 連線 -----------------
const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Iml2b2ZyamliZGV6Ynl4eG11dG9rIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODkxMzQ2ODMsImV4cCI6MjEwNDcxMDY4M30.di9AXuYBtuSDL1upSKQ48-U0y9PN-TtFEYivmgJgbEQ'
const supabase = createClient(supabaseUrl, supabaseKey)

// ----------------- 頁面分頁控制 -----------------
const currentTab = ref('manage')
const subTab = ref('order')

// ==========================================
// 模組資料狀態
// ==========================================
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

// 核心過濾函式：只顯示「品種名 幾盆」
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
  const searchStr = `${ord.note || ''} ${ord.spec || ''}`
  const matches = [...searchStr.matchAll(/(\d+)\s*盆/g)]
  if (matches.length > 0) {
    const sum = matches.reduce((total, m) => total + (parseInt(m[1]) || 0), 0)
    return sum > 0 ? sum : 1
  }
  return 1
}

const getOrderShippingFee = (ord) => {
  if (!ord) return 0
  const searchStr = `${ord.note || ''}`
  const shipMatch = searchStr.match(/運費\s*[:：]?\s*(\d+)/)
  return shipMatch ? parseInt(shipMatch[1]) : 0
}

const generateDateSeqId = (prefix, existingList) => {
  const now = new Date()
  const y = now.getFullYear()
  const m = String(now.getMonth() + 1).padStart(2, '0')
  const d = String(now.getDate()).padStart(2, '0')
  const dateStr = `${y}${m}${d}`
  const targetPrefix = `${prefix}-${dateStr}-`

  const todayItems = (existingList || []).filter(item => String(item.id || '').startsWith(targetPrefix))
  const nextSeq = todayItems.length + 1
  return `${targetPrefix}${String(nextSeq).padStart(2, '0')}`
}

const formOrchid = ref({ name: '', note: '標準優良品種', photo_url: '' })
const formCust = ref({ name: '', type: '批發商', billing_cycle: '每單結', phone: '0912-345678', line_note: '' })
const formInv = ref({
  category: '蘭花',
  item_name: '',
  spec_spike: '單梗',
  spec_color: '紅',
  spec_size: '大',
  spec_height: '中',
  pot_type: '桌上盆 (100)',
  qty: 10,
  unit_cost: 100,
  cost: 1000,
  supplier: '某某花農',
  date: new Date().toISOString().split('T')[0]
})
const formRet = ref({
  return_type: '退給花農',
  party_name: '',
  target_item: '',
  qty: 2,
  unit_price: 150,
  total_amount: 300,
  date: new Date().toISOString().split('T')[0],
  reason: '運送碰撞 / 開花不良'
})

// 訂單表單：支援多組花禮規格
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
  items: [
    {
      orchid_name: '特選蘭花',
      pots_qty: 1,
      stalks: 10,
      unit_price: 250,
      pot: '桌上盆 (100)',
      quick_pot: '未使用'
    }
  ]
})

const onFlowerSelectChange = (item) => {
  if (!item.orchid_name) return
  const matchedInv = inventoryList.value.find(i => 
    i.category === '蘭花' && i.item_name === item.orchid_name
  )
  if (matchedInv) {
    const refCost = matchedInv.unit_cost || (matchedInv.qty ? Math.round(matchedInv.cost / matchedInv.qty) : 0)
    if (refCost > 0 && item.unit_price <= 0) {
      item.unit_price = Math.round(refCost * 2)
    }
  }
  calcOrderPrice()
}

const addOrderItemRow = () => {
  formOrder.value.items.push({
    orchid_name: '特選蘭花',
    pots_qty: 1,
    stalks: 10,
    unit_price: 250,
    pot: '桌上盆 (100)',
    quick_pot: '未使用'
  })
  calcOrderPrice()
}

const removeOrderItemRow = (idx) => {
  if (formOrder.value.items.length > 1) {
    formOrder.value.items.splice(idx, 1)
    calcOrderPrice()
  }
}

const flowerInventory = computed(() => inventoryList.value.filter(i => i.category === '蘭花'))

const loadOrders = async () => {
  const { data } = await supabase.from('orders').select('*').order('created_at', { ascending: false })
  if (data) orderList.value = data
}

const calcOrderPrice = () => {
  let totalPrice = 0
  let totalCost = 0

  const potCostMap = {
    '桌上盆 (100)': 100,
    '落地盆陶瓷-喪 (100)': 100,
    '落地陶瓷盆-喜 (200)': 200,
    '羅馬盆 (280)': 280,
    '無盆': 0
  }

  formOrder.value.items.forEach(item => {
    const p = Number(item.pots_qty) || 1
    const s = Number(item.stalks) || 0
    const u = Number(item.unit_price) || 0
    totalPrice += (s * u) * p

    const potCost = potCostMap[item.pot] || 0
    const quickCost = item.quick_pot === '使用快捷盆 (70)' ? 70 : 0
    const flowerBaseCost = s * 60
    totalCost += (flowerBaseCost + potCost + quickCost) * p
  })

  const ship = Number(formOrder.value.shipping_fee) || 0
  formOrder.value.price = totalPrice + ship
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
  if (specText.includes(';')) {
    const parts = specText.split(';').map(s => s.trim()).filter(Boolean)
    parts.forEach(p => {
      const stalksMatch = p.match(/(\d+)\s*棵/)
      const unitMatch = p.match(/單價(\d+)元/)
      const potsMatch = p.match(/(\d+)\s*盆/)
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
  }

  if (parsedItems.length === 0) {
    parsedItems.push({
      orchid_name: ord.spec ? ord.spec.split('|')[0].replace(/\(.*?\)/g, '').trim() : '特選蘭花',
      pots_qty: getOrderTotalPots(ord),
      stalks: 10,
      unit_price: 250,
      pot: (ord.pot || '桌上盆 (100)').replace(/\s*\+\s*快捷盆/, '').trim(),
      quick_pot: (ord.pot || '').includes('快捷盆') ? '使用快捷盆 (70)' : '未使用'
    })
  }

  formOrder.value = {
    cust_type: ord.cust_type || '批發',
    customer: ord.customer || '',
    billing_cycle: ord.billing_cycle || '每單結',
    phone: ord.phone || '',
    shipping_fee: getOrderShippingFee(ord),
    cost: Number(ord.cost) || 0,
    price: Number(ord.price) || 0,
    tax_id: ord.tax_id || '',
    need_receipt: ord.need_receipt || '不需收據',
    note: ord.note || '',
    card_status: ord.card_status || '未製作',
    receipt_status: ord.receipt_status || '未列印',
    shipped_status: ord.shipped_status || '未出貨',
    payment_status: ord.payment_status || '未結',
    order_date: ord.order_date,
    expected_date: ord.expected_date,
    items: parsedItems
  }

  subTab.value = 'order'
  nextTick(() => {
    document.getElementById('order-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}

const cancelEditOrder = () => {
  editingOrderId.value = null
  formOrder.value = {
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
    items: [
      {
        orchid_name: '特選蘭花',
        pots_qty: 1,
        stalks: 10,
        unit_price: 250,
        pot: '桌上盆 (100)',
        quick_pot: '未使用'
      }
    ]
  }
}

const saveOrder = async () => {
  if (!formOrder.value.customer) return alert('請輸入客戶名稱！')
  
  const specParts = formOrder.value.items.map(item => {
    const p = Number(item.pots_qty) || 1
    const s = Number(item.stalks) || 1
    const u = Number(item.unit_price) || 0
    return `${item.orchid_name || '特選蘭花'} (${p}盆) | ${s}棵 (單價${u}元)`
  })
  const fullSpecStr = specParts.join('; ')

  const mainItem = formOrder.value.items[0] || {}
  const finalPotStr = (mainItem.pot || '桌上盆 (100)') + (mainItem.quick_pot === '使用快捷盆 (70)' ? ' + 快捷盆' : '')

  const payload = {
    cust_type: formOrder.value.cust_type,
    customer: formOrder.value.customer,
    billing_cycle: formOrder.value.billing_cycle,
    phone: formOrder.value.phone,
    spec: fullSpecStr,
    pot: finalPotStr,
    cost: formOrder.value.cost,
    price: formOrder.value.price,
    tax_id: formOrder.value.tax_id,
    need_receipt: formOrder.value.need_receipt,
    note: formOrder.value.note,
    card_status: formOrder.value.card_status,
    receipt_status: formOrder.value.receipt_status,
    shipped_status: formOrder.value.shipped_status,
    payment_status: formOrder.value.payment_status,
    order_date: formOrder.value.order_date,
    expected_date: formOrder.value.expected_date
  }

  const totalPots = formOrder.value.items.reduce((sum, it) => sum + (Number(it.pots_qty) || 1), 0)
  const metaTag = `[共${totalPots}盆, 運費:${formOrder.value.shipping_fee}元]`
  if (!payload.note.includes(metaTag)) {
    payload.note = payload.note ? `${payload.note} ${metaTag}` : metaTag
  }

  if (editingOrderId.value) {
    const { error } = await supabase.from('orders').update(payload).eq('id', editingOrderId.value)
    if (!error) {
      alert(`訂單 ${editingOrderId.value} 修改成功！`)
      cancelEditOrder()
      loadOrders()
    } else {
      alert('修改失敗：' + error.message)
    }
  } else {
    const newId = generateDateSeqId('OR', orderList.value)
    const { error } = await supabase.from('orders').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`訂單建立成功！單號：${newId}`)
      cancelEditOrder()
      loadOrders()
    } else {
      alert('建立失敗：' + error.message)
    }
  }
}

const updateOrderField = async (ord, field, value) => {
  const updateObj = {}
  updateObj[field] = value
  await supabase.from('orders').update(updateObj).eq('id', ord.id)
}

const getCardStatusClass = (status) => {
  if (status === '已製作') return 'select-status green'
  if (status === '免製作') return 'select-status gray'
  return 'select-status orange'
}

// ==========================================
// 2. 客戶未結對帳專區
// ==========================================
const statementCustomer = ref('')
const statementPeriod = ref('all')
const statementPaymentFilter = ref('未結')

const statementOrders = computed(() => {
  return orderList.value.filter(o => {
    if (statementCustomer.value && o.customer !== statementCustomer.value) return false
    if (statementPaymentFilter.value === '未結' && o.payment_status !== '未結') return false
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
    return true
  })
})

const statementTotalAmount = computed(() => {
  return statementOrders.value.reduce((sum, o) => sum + (Number(o.price) || 0), 0)
})

const currentCustomerInfoText = computed(() => {
  if (!statementCustomer.value) return '全部客戶統整'
  const c = customers.value.find(item => item.name === statementCustomer.value)
  return c ? `${c.type || '一般'} | ${c.billing_cycle || '每單結'}` : '一般客戶'
})

const copyLineStatement = () => {
  if (statementOrders.value.length === 0) return alert('目前無訂單可複製對帳資料！')
  const custTitle = statementCustomer.value || '全部客戶'
  const periodTextMap = { all: '歷史累計未結', thisWeek: '本週未結', thisMonth: '本月未結', lastMonth: '上月未結' }
  const periodText = periodTextMap[statementPeriod.value] || '對帳區間'

  let text = `🌸【宸豐蘭藝 - 對帳明細】\n`
  text += `--------------------------------\n`
  text += `客戶名稱：${custTitle}\n`
  text += `帳務週期：${currentCustomerInfoText.value}\n`
  text += `統計區間：${periodText}\n`
  text += `待結筆數：${statementOrders.value.length} 筆\n`
  text += `未結總額：NT$ ${statementTotalAmount.value.toLocaleString()} 元\n`
  text += `--------------------------------\n`
  text += `出貨明細清單：\n`
  statementOrders.value.forEach((o, index) => {
    text += `${index + 1}. [${o.order_date}] ${o.spec} ➔ $${o.price}元 (${o.payment_status})\n`
  })
  text += `--------------------------------\n`
  text += `※ 敬請核對帳款，感謝您的支持與惠顧！`

  navigator.clipboard.writeText(text).then(() => {
    showToast('✅ LINE 對帳明細已複製到剪貼簿！可直接貼給客戶！')
  }).catch(() => {
    alert('複製失敗，請手動複製')
  })
}

const exportStatementExcel = () => {
  if (statementOrders.value.length === 0) return alert('目前尚無資料可匯出！')
  const custName = statementCustomer.value || '全客戶'
  const exportData = statementOrders.value.map(o => ({
    '單號': o.id,
    '下單日期': o.order_date,
    '送達日期': o.expected_date,
    '客戶名稱': o.customer,
    '總盆數': getOrderTotalPots(o),
    '品項與規格': o.spec,
    '盆器': o.pot,
    '額外運費': getOrderShippingFee(o),
    '售價(元)': o.price,
    '出貨狀態': o.shipped_status,
    '收款狀態': o.payment_status
  }))
  const worksheet = XLSX.utils.json_to_sheet(exportData)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '對帳明細')
  XLSX.writeFile(workbook, `宸豐蘭藝對帳單_${custName}_${new Date().toISOString().split('T')[0]}.xlsx`)
}

const batchMarkPaid = async () => {
  if (!confirm(`確定要將這 ${statementOrders.value.length} 筆訂單一次全部標記為「已結」嗎？`)) return
  const ids = statementOrders.value.map(o => o.id)
  const { error } = await supabase.from('orders').update({ payment_status: '已結' }).in('id', ids)
  if (!error) {
    showToast('✅ 已批次結清成功！')
    loadOrders()
  } else {
    alert('更新失敗：' + error.message)
  }
}

// ==========================================
// 3. A5 橫式簽收單 (支援現場收件人線上簽名)
// ==========================================
const shopNameMode = ref('default')
const customShopName = ref('')
const displayShopName = computed(() => shopNameMode.value === 'default' ? '宸豐蘭藝' : (customShopName.value || '宸豐蘭藝'))

const receiptViewportRef = ref(null)
const receiptZoom = ref(1)
const selectedOrderId = ref('')

const receiptForm = ref({
  orderId: '',
  deliveryDate: '115-09-03 送達',
  recipient: '永全證券 陳柏榮總經理 (0912-345678)',
  address: '桃園市桃園區縣府路 82 號 1 樓',
  item: '特選蘭花 1盆',
  giver: '敬領 誌慶 / 宸豐蘭藝 敬製',
  notes: '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！'
})

// 線上手寫簽名板狀態
const signPadCanvasRef = ref(null)
const liveSignDataUrl = ref('')
let isSigning = false
let lastSignX = 0
let lastSignY = 0

const getSignCoords = (e) => {
  if (!signPadCanvasRef.value) return { x: 0, y: 0 }
  const rect = signPadCanvasRef.value.getBoundingClientRect()
  const scaleX = signPadCanvasRef.value.width / rect.width
  const scaleY = signPadCanvasRef.value.height / rect.height
  return {
    x: (e.clientX - rect.left) * scaleX,
    y: (e.clientY - rect.top) * scaleY
  }
}

const startSign = (e) => {
  isSigning = true
  const coords = getSignCoords(e)
  lastSignX = coords.x
  lastSignY = coords.y
}

const drawingSign = (e) => {
  if (!isSigning || !signPadCanvasRef.value) return
  const ctx = signPadCanvasRef.value.getContext('2d')
  const coords = getSignCoords(e)

  ctx.lineWidth = 2.5
  ctx.lineCap = 'round'
  ctx.lineJoin = 'round'
  ctx.strokeStyle = '#0f172a'

  ctx.beginPath()
  ctx.moveTo(lastSignX, lastSignY)
  ctx.lineTo(coords.x, coords.y)
  ctx.stroke()

  lastSignX = coords.x
  lastSignY = coords.y
}

const stopSign = () => {
  if (!isSigning) return
  isSigning = false
  if (signPadCanvasRef.value) {
    liveSignDataUrl.value = signPadCanvasRef.value.toDataURL('image/png')
  }
}

const clearLiveSignature = () => {
  if (signPadCanvasRef.value) {
    const ctx = signPadCanvasRef.value.getContext('2d')
    ctx.clearRect(0, 0, signPadCanvasRef.value.width, signPadCanvasRef.value.height)
    liveSignDataUrl.value = ''
    showToast('已清除簽名，請收件人重新簽署')
  }
}

const onSelectReceiptOrder = () => {
  if (!selectedOrderId.value) return
  const ord = orderList.value.find(o => o.id === selectedOrderId.value)
  if (ord) {
    const cust = customers.value.find(c => c.name === ord.customer)
    receiptForm.value.orderId = ord.id
    receiptForm.value.deliveryDate = `${ord.expected_date} 送達`
    receiptForm.value.recipient = `${ord.customer} ${ord.phone ? '(' + ord.phone + ')' : ''}`
    receiptForm.value.address = cust?.line_note || '同訂購人地址 / 門市取貨'
    receiptForm.value.item = formatSimpleItemName(ord)
    receiptForm.value.giver = `敬領 誌慶 / ${displayShopName.value} 敬製`
    receiptForm.value.notes = ord.note ? `備註：${ord.note}` : '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！'
    clearLiveSignature()
  }
}

const autoFitReceipt = () => {
  if (!receiptViewportRef.value) return
  const availWidth = Math.max(receiptViewportRef.value.clientWidth - 28, 280)
  receiptZoom.value = Math.min(Math.max(+(availWidth / 794).toFixed(2), 0.35), 1.0)
}

const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  const cust = customers.value.find(c => c.name === ord.customer)
  receiptForm.value.orderId = ord.id
  receiptForm.value.deliveryDate = `${ord.expected_date} 送達`
  receiptForm.value.recipient = `${ord.customer} ${ord.phone ? '(' + ord.phone + ')' : ''}`
  receiptForm.value.address = cust?.line_note || '同訂購人地址 / 門市取貨'
  receiptForm.value.item = formatSimpleItemName(ord)
  receiptForm.value.giver = `敬領 誌慶 / ${displayShopName.value} 敬製`
  receiptForm.value.notes = ord.note ? `備註：${ord.note}` : '花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！'

  clearLiveSignature()
  currentTab.value = 'receipt'
  nextTick(() => autoFitReceipt())
}

const printReceiptAndMarkDone = async () => {
  if (selectedOrderId.value) {
    const ord = orderList.value.find(o => o.id === selectedOrderId.value)
    if (ord) {
      await updateOrderField(ord, 'receipt_status', '已列印')
      ord.receipt_status = '已列印'
    }
  }
  window.print()
}

// 核心多功能圖片傳送函式 (同步下載 ＋ 剪貼簿 ＋ 彈窗預覽)
const shareOrCopyCanvasBlob = async (canvas, filename, shareTitle, successMsg) => {
  const dataUrl = canvas.toDataURL('image/png')

  // 1. 同步觸發瀏覽器下載
  const downloadLink = document.createElement('a')
  downloadLink.download = filename
  downloadLink.href = dataUrl
  document.body.appendChild(downloadLink)
  downloadLink.click()
  document.body.removeChild(downloadLink)

  // 2. 顯示預覽彈窗 (滑鼠右鍵直接複製)
  shareModalImg.value = dataUrl
  shareModalTitle.value = shareTitle

  canvas.toBlob(async (blob) => {
    if (!blob) return
    const file = new File([blob], filename, { type: 'image/png' })

    // 手機平板原生分享
    if (navigator.canShare && navigator.canShare({ files: [file] })) {
      try {
        await navigator.share({
          title: shareTitle,
          files: [file]
        })
        return
      } catch (err) {
        if (err.name !== 'AbortError') console.error('Share failed', err)
      }
    }

    // 寫入剪貼簿 (供電腦版 Ctrl + V 貼上)
    if (navigator.clipboard && navigator.clipboard.write) {
      try {
        await navigator.clipboard.write([
          new ClipboardItem({ 'image/png': blob })
        ])
        showToast(`📋 ${successMsg}！\n已複製到剪貼簿並完成下載！請直接至電腦版 LINE 按 Ctrl + V 貼上發送！`)
        return
      } catch (err) {
        console.warn('Clipboard write fallback', err)
      }
    }

    showToast(`📁 ${successMsg}！\n圖檔已下載！請直接拖曳圖片或在此視窗按右鍵「複製圖片」至 LINE 發送！`)
  }, 'image/png')
}

// Canvas 繪圖多行文字排版輔助函式 (防止壓格子的核心工具)
const drawWrappedText = (ctx, text, x, y, maxWidth, lineHeight, maxLines = 2) => {
  if (!text) return
  const chars = String(text).split('')
  let line = ''
  let currentY = y
  let linesCount = 1

  for (let n = 0; n < chars.length; n++) {
    const testLine = line + chars[n]
    const metrics = ctx.measureText(testLine)
    const testWidth = metrics.width
    if (testWidth > maxWidth && n > 0) {
      ctx.fillText(line, x, currentY)
      line = chars[n]
      currentY += lineHeight
      linesCount++
      if (linesCount > maxLines) {
        // 超過行數加省略號
        ctx.fillText('...', x + maxWidth - 20, currentY - lineHeight)
        return
      }
    } else {
      line = testLine
    }
  }
  ctx.fillText(line, x, currentY)
}

// 產生簽收單高畫質圖片供回傳下單客戶（解決文字壓格問題）
const shareReceiptToBuyerDirect = () => {
  const canvas = document.createElement('canvas')
  canvas.width = 794 * 2
  canvas.height = 560 * 2
  const ctx = canvas.getContext('2d')
  ctx.scale(2, 2)

  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, 794, 560)

  const fontFam = '"TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "Kaiti", serif'
  ctx.fillStyle = '#0f172a'

  // 抬頭花店名
  ctx.font = `900 26px ${fontFam}`
  ctx.textAlign = 'left'
  ctx.fillText(displayShopName.value, 45, 60)

  // 主標題
  ctx.fillStyle = '#dc2626'
  ctx.font = `bold 22px ${fontFam}`
  ctx.textAlign = 'center'
  ctx.fillText('銷貨 / 出貨簽收單', 397, 60)

  // 右上角單號與日期
  ctx.fillStyle = '#334155'
  ctx.font = `13px ${fontFam}`
  ctx.textAlign = 'right'
  ctx.fillText(`訂單編號：${receiptForm.value.orderId || '現場開單'}`, 749, 48)
  ctx.fillText(`送達日期：${receiptForm.value.deliveryDate || '依約定送達'}`, 749, 68)

  // 頂部分隔線
  ctx.lineWidth = 2.5
  ctx.strokeStyle = '#1e293b'
  ctx.beginPath()
  ctx.moveTo(45, 80)
  ctx.lineTo(749, 80)
  ctx.stroke()

  // 表格尺寸設定
  const tLeft = 45, tTop = 95, tWidth = 704
  const rowHeight = 75
  ctx.lineWidth = 1.5
  ctx.strokeStyle = '#334155'

  // 外框
  ctx.strokeRect(tLeft, tTop, tWidth, rowHeight * 4)

  // 內部橫線
  for (let i = 1; i <= 3; i++) {
    ctx.beginPath()
    ctx.moveTo(tLeft, tTop + rowHeight * i)
    ctx.lineTo(tLeft + tWidth, tTop + rowHeight * i)
    ctx.stroke()
  }

  // 垂直分割線 (精確配比，留足空間給收件人與電話)
  // [標籤 110px] [收件人 260px] [標籤 90px] [地址 244px]
  const col1W = 110
  const col2W = 260
  const col3W = 90

  ctx.beginPath()
  // 第 1 欄右側線 (收件單位/人標籤右側)
  ctx.moveTo(tLeft + col1W, tTop)
  ctx.lineTo(tLeft + col1W, tTop + rowHeight * 4)
  // 第 2 欄右側線 (收件人內容右側)
  ctx.moveTo(tLeft + col1W + col2W, tTop)
  ctx.lineTo(tLeft + col1W + col2W, tTop + rowHeight)
  // 第 3 欄右側線 (送達地址標籤右側)
  ctx.moveTo(tLeft + col1W + col2W + col3W, tTop)
  ctx.lineTo(tLeft + col1W + col2W + col3W, tTop + rowHeight)
  ctx.stroke()

  // 標籤欄底色 (灰色)
  ctx.fillStyle = '#f1f5f9'
  ctx.fillRect(tLeft + 1, tTop + 1, col1W - 2, rowHeight - 2)
  ctx.fillRect(tLeft + col1W + col2W + 1, tTop + 1, col3W - 2, rowHeight - 2)
  ctx.fillRect(tLeft + 1, tTop + rowHeight + 1, col1W - 2, rowHeight - 2)
  ctx.fillRect(tLeft + 1, tTop + rowHeight * 2 + 1, col1W - 2, rowHeight - 2)
  ctx.fillRect(tLeft + 1, tTop + rowHeight * 3 + 1, col1W - 2, rowHeight - 2)

  // 填寫欄位標籤
  ctx.fillStyle = '#1e293b'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.font = `bold 15px ${fontFam}`
  ctx.fillText('收件單位/人', tLeft + col1W / 2, tTop + rowHeight / 2)
  ctx.fillText('送達地址', tLeft + col1W + col2W + col3W / 2, tTop + rowHeight / 2)
  ctx.fillText('花禮品項', tLeft + col1W / 2, tTop + rowHeight * 1.5)
  ctx.fillText('致贈/賀詞', tLeft + col1W / 2, tTop + rowHeight * 2.5)
  ctx.fillText('備註說明', tLeft + col1W / 2, tTop + rowHeight * 3.5)

  // 填寫欄位數值 (已套用自動換行，絕不壓到旁邊格子)
  ctx.textAlign = 'left'
  ctx.textBaseline = 'top'

  // 1. 收件人與電話 (限制在 col2W - 24px 寬度內，太長自動換行，絕不越界)
  ctx.fillStyle = '#0f172a'
  ctx.font = `bold 15px ${fontFam}`
  drawWrappedText(ctx, receiptForm.value.recipient || '—', tLeft + col1W + 12, tTop + 20, col2W - 24, 22, 2)

  // 2. 送達地址 (限制在右邊格子寬度內自動換行)
  ctx.fillStyle = '#334155'
  ctx.font = `14px ${fontFam}`
  const addrWidth = tWidth - (col1W + col2W + col3W) - 24
  drawWrappedText(ctx, receiptForm.value.address || '同訂購人地址 / 門市取貨', tLeft + col1W + col2W + col3W + 12, tTop + 20, addrWidth, 20, 2)

  // 3. 花禮品項
  ctx.fillStyle = '#1e3a8a'
  ctx.font = `bold 16px ${fontFam}`
  drawWrappedText(ctx, receiptForm.value.item || '特選蘭花 1盆', tLeft + col1W + 12, tTop + rowHeight + 26, tWidth - col1W - 24, 24, 2)

  // 4. 致贈/賀詞
  ctx.fillStyle = '#111827'
  ctx.font = `15px ${fontFam}`
  drawWrappedText(ctx, receiptForm.value.giver || '敬領 誌慶 / 宸豐蘭藝 敬製', tLeft + col1W + 12, tTop + rowHeight * 2 + 26, tWidth - col1W - 24, 22, 2)

  // 5. 備註說明
  ctx.fillStyle = '#475569'
  ctx.font = `13.5px ${fontFam}`
  drawWrappedText(ctx, receiptForm.value.notes || '花禮已專車安全送達指定地點，敬請點交簽名確認。', tLeft + col1W + 12, tTop + rowHeight * 3 + 26, tWidth - col1W - 24, 20, 2)

  // 底部說明區
  ctx.fillStyle = '#334155'
  ctx.font = `14px ${fontFam}`
  ctx.fillText('送貨司機 / 經手人：______________', 45, 435)
  ctx.font = `12px ${fontFam}`
  ctx.fillStyle = '#64748b'
  ctx.fillText('※ 專車親送・現場點交確認・花禮已送達 (收件人已線上簽收)', 45, 465)

  // 客戶簽名方框
  const sBoxLeft = 529, sBoxTop = 410, sBoxW = 220, sBoxH = 80
  ctx.lineWidth = 1.5
  ctx.strokeStyle = '#475569'
  ctx.strokeRect(sBoxLeft, sBoxTop, sBoxW, sBoxH)
  ctx.fillStyle = '#e2e8f0'
  ctx.fillRect(sBoxLeft + 1, sBoxTop + 1, sBoxW - 2, 22)

  ctx.fillStyle = '#334155'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.font = `bold 12px ${fontFam}`
  ctx.fillText('客戶簽收章 / 線上簽名處', sBoxLeft + sBoxW / 2, sBoxTop + 12)

  // 繪製手寫簽名
  const finishShare = () => {
    const filename = `已簽收單據_${receiptForm.value.orderId || '現場'}_${new Date().toISOString().split('T')[0]}.png`
    shareOrCopyCanvasBlob(canvas, filename, '簽收單據回傳', '已簽名單據準備完成')
  }

  if (liveSignDataUrl.value) {
    const signImg = new Image()
    signImg.onload = () => {
      ctx.drawImage(signImg, sBoxLeft + 10, sBoxTop + 24, sBoxW - 20, sBoxH - 28)
      finishShare()
    }
    signImg.src = liveSignDataUrl.value
  } else {
    finishShare()
  }
}

// ==========================================
// 4. 農民收據
// ==========================================
const farmerReceiptViewportRef = ref(null)
const farmerZoom = ref(1)
const selectedFarmerOrderId = ref('')

const farmerReceipt = ref({
  year: '115',
  month: '09',
  day: '03',
  buyerName: '永全證券股份有限公司',
  taxId: '12345678',
  buyerAddress: '桃園市桃園區縣府路 82 號',
  itemName: '蘭花禮盆',
  spec: '特級蝴蝶蘭',
  qty: '1 盆',
  unitPrice: '2,500',
  totalAmount: 2500,
  note: ''
})

const chineseDigits = ref({
  hundredThousands: '零',
  tenThousands: '零',
  thousands: '貳',
  hundreds: '伍',
  tens: '零',
  ones: '零'
})

const digitMap = ['零', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖']

const updateChineseAmount = () => {
  const amt = Math.floor(Number(farmerReceipt.value.totalAmount) || 0)
  const padded = amt.toString().padStart(6, '0')
  const digits = padded.split('').map(d => digitMap[Number(d)])
  chineseDigits.value = {
    hundredThousands: digits[0],
    tenThousands: digits[1],
    thousands: digits[2],
    hundreds: digits[3],
    tens: digits[4],
    ones: digits[5]
  }
}

const onSelectFarmerReceiptOrder = () => {
  if (!selectedFarmerOrderId.value) return
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

    farmerReceipt.value.itemName = '蝴蝶蘭花禮'
    farmerReceipt.value.spec = formatSimpleItemName(ord)
    farmerReceipt.value.qty = `${getOrderTotalPots(ord)} 盆`
    farmerReceipt.value.unitPrice = Number(ord.price).toLocaleString()
    farmerReceipt.value.totalAmount = Number(ord.price) || 0
    farmerReceipt.value.note = ord.id

    updateChineseAmount()
  }
}

const autoFitFarmerReceipt = () => {
  if (!farmerReceiptViewportRef.value) return
  const availWidth = Math.max(farmerReceiptViewportRef.value.clientWidth - 28, 280)
  farmerZoom.value = Math.min(Math.max(+(availWidth / 794).toFixed(2), 0.35), 1.0)
}

const fillFarmerReceiptFromOrder = (ord) => {
  selectedFarmerOrderId.value = ord.id
  onSelectFarmerReceiptOrder()
  currentTab.value = 'farmer_receipt'
  nextTick(() => autoFitFarmerReceipt())
}

const printFarmerReceipt = () => window.print()

// 產生農民收據高畫質圖片
const shareFarmerReceiptToLineDirect = () => {
  const canvas = document.createElement('canvas')
  canvas.width = 794 * 2
  canvas.height = 560 * 2
  const ctx = canvas.getContext('2d')
  ctx.scale(2, 2)

  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, 794, 560)

  const fontFam = '"TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "Kaiti", serif'
  ctx.fillStyle = '#000000'

  ctx.font = `bold 26px ${fontFam}`
  ctx.textAlign = 'center'
  ctx.fillText('農（漁、牧）民出售農（漁、牧）產品收據', 397, 45)

  ctx.font = `15px ${fontFam}`
  ctx.textAlign = 'right'
  ctx.fillText(`中華民國 ${farmerReceipt.value.year} 年 ${farmerReceipt.value.month} 月 ${farmerReceipt.value.day} 日`, 760, 80)

  ctx.lineWidth = 1.8
  ctx.strokeStyle = '#000000'
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
  
  const finishCanvas = () => {
    ctx.font = `14.5px ${fontFam}`
    ctx.fillText(`住址：                     國民統一身分證編號：F129940801`, 42, 368)
    ctx.font = `11px ${fontFam}`
    ctx.fillText(`本收據之農民身分確實無誤，若有不實者願依法受罰。`, 42, 410)
    ctx.font = `9.5px ${fontFam}`
    ctx.fillText(`附註：依據財政部 68.11.2 台財稅第三七六六五號函：自 68 年 11 月 16 日起，凡農民出售其本身所生產、捕獲或畜養之農林漁牧產品所出具之收據，一律免納印花稅...`, 42, 435)

    const filename = `農民收據_${farmerReceipt.value.buyerName}_${farmerReceipt.value.year}${farmerReceipt.value.month}${farmerReceipt.value.day}.png`
    shareOrCopyCanvasBlob(canvas, filename, '農民收據確認', '農民收據圖片準備完成')
  }

  const stampImg = new Image()
  stampImg.crossOrigin = 'anonymous'
  stampImg.onload = () => {
    ctx.drawImage(stampImg, 260, 305, 50, 50)
    finishCanvas()
  }
  stampImg.onerror = () => {
    finishCanvas()
  }
  stampImg.src = activeCaiSealSrc.value
}

// ==========================================
// 5. 進貨與庫存
// ==========================================
const loadInventory = async () => {
  const { data } = await supabase.from('inventory').select('*').order('created_at', { ascending: false })
  if (data) inventoryList.value = data
}

const calcInvCost = () => {
  const qty = Number(formInv.value.qty) || 0
  const unit = Number(formInv.value.unit_cost) || 0
  formInv.value.cost = qty * unit
}

const onPotTypeChange = () => {
  const potCostMap = {
    '桌上盆 (100)': 100,
    '落地盆陶瓷-喪 (100)': 100,
    '落地陶瓷盆-喜 (200)': 200,
    '羅馬盆 (280)': 280,
    '快捷盆 (70)': 70
  }
  const unit = potCostMap[formInv.value.pot_type] || 100
  formInv.value.unit_cost = unit
  calcInvCost()
}

const startEditInv = (inv) => {
  editingInvId.value = inv.id
  const calculatedUnit = inv.unit_cost || (inv.qty ? Math.round(inv.cost / inv.qty) : 0)
  formInv.value = {
    category: inv.category,
    item_name: inv.item_name,
    spec_spike: '單梗',
    spec_color: '紅',
    spec_size: '大',
    spec_height: '中',
    pot_type: '桌上盆 (100)',
    qty: inv.qty,
    unit_cost: calculatedUnit,
    cost: inv.cost,
    supplier: inv.supplier,
    date: inv.date
  }
  nextTick(() => {
    document.getElementById('inv-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}

const cancelEditInv = () => {
  editingInvId.value = null
  formInv.value = {
    category: '蘭花',
    item_name: '',
    spec_spike: '單梗',
    spec_color: '紅',
    spec_size: '大',
    spec_height: '中',
    pot_type: '桌上盆 (100)',
    qty: 10,
    unit_cost: 100,
    cost: 1000,
    supplier: '某某花農',
    date: new Date().toISOString().split('T')[0]
  }
}

const saveInventory = async () => {
  const isFlower = formInv.value.category === '蘭花'
  const itemName = isFlower ? formInv.value.item_name : formInv.value.pot_type.split(' ')[0]
  if (!itemName) return alert('請輸入品項名稱！')
  const specDesc = isFlower 
    ? `規格:${formInv.value.spec_spike} | 顏色:${formInv.value.spec_color} | 大小:${formInv.value.spec_size} | 高矮:${formInv.value.spec_height}`
    : '固定規格'

  const payload = {
    category: formInv.value.category,
    item_name: itemName,
    spec: specDesc,
    qty: formInv.value.qty,
    unit_cost: formInv.value.unit_cost,
    cost: formInv.value.cost,
    supplier: formInv.value.supplier,
    date: formInv.value.date
  }

  if (editingInvId.value) {
    const { error } = await supabase.from('inventory').update(payload).eq('id', editingInvId.value)
    if (!error) {
      alert(`進貨紀錄 ${editingInvId.value} 修改成功！`)
      cancelEditInv()
      loadInventory()
    } else {
      alert('修改失敗：' + error.message)
    }
  } else {
    const newId = generateDateSeqId('IN', inventoryList.value)
    const { error } = await supabase.from('inventory').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`進貨紀錄新增成功！編號：${newId}`)
      cancelEditInv()
      loadInventory()
    } else {
      alert('新增失敗：' + error.message)
    }
  }
}

// ==========================================
// 6. 客戶資料庫
// ==========================================
const loadCustomers = async () => {
  const { data } = await supabase.from('customers').select('*').order('created_at', { ascending: false })
  if (data) customers.value = data
}

const startEditCust = (c) => {
  editingCustId.value = c.id
  formCust.value = {
    name: c.name,
    type: c.type || '批發商',
    billing_cycle: c.billing_cycle || '每單結',
    phone: c.phone || '',
    line_note: c.line_note || ''
  }
  nextTick(() => {
    document.getElementById('cust-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}

const cancelEditCust = () => {
  editingCustId.value = null
  formCust.value = { name: '', type: '批發商', billing_cycle: '每單結', phone: '0912-345678', line_note: '' }
}

const saveCustomer = async () => {
  if (!formCust.value.name) return alert('請輸入客戶名稱！')
  const payload = { ...formCust.value }
  if (editingCustId.value) {
    const { error } = await supabase.from('customers').update(payload).eq('id', editingCustId.value)
    if (!error) {
      alert(`客戶 ${editingCustId.value} 修改成功！`)
      cancelEditCust()
      loadCustomers()
    } else {
      alert('修改失敗：' + error.message)
    }
  } else {
    const newId = generateDateSeqId('CU', customers.value)
    const { error } = await supabase.from('customers').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`客戶建立成功！編號：${newId}`)
      cancelEditCust()
      loadCustomers()
    } else {
      alert('建立失敗：' + error.message)
    }
  }
}

// ==========================================
// 7. 蘭花品種庫
// ==========================================
const loadOrchids = async () => {
  const { data } = await supabase.from('orchids').select('*').order('created_at', { ascending: false })
  if (data) orchids.value = data
}

const onPhotoFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = (event) => {
    const img = new Image()
    img.onload = () => {
      const canvas = document.createElement('canvas')
      const maxDim = 800
      let width = img.width
      let height = img.height
      if (width > height) {
        if (width > maxDim) {
          height = Math.round((height * maxDim) / width)
          width = maxDim
        }
      } else {
        if (height > maxDim) {
          width = Math.round((width * maxDim) / height)
          height = maxDim
        }
      }
      canvas.width = width
      canvas.height = height
      const ctx = canvas.getContext('2d')
      ctx.drawImage(img, 0, 0, width, height)
      const compressed = canvas.toDataURL('image/jpeg', 0.75)
      formOrchid.value.photo_url = compressed
    }
    img.src = event.target.result
  }
  reader.readAsDataURL(file)
}

const startEditOrchid = (item) => {
  editingOrchidId.value = item.id
  formOrchid.value = { 
    name: item.name, 
    note: item.note || '',
    photo_url: item.photo_url || ''
  }
  nextTick(() => {
    document.getElementById('orchid-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}

const cancelEditOrchid = () => {
  editingOrchidId.value = null
  formOrchid.value = { name: '', note: '標準優良品種', photo_url: '' }
}

const saveOrchid = async () => {
  if (!formOrchid.value.name) return alert('請輸入品種名稱！')
  const payload = { ...formOrchid.value }
  if (editingOrchidId.value) {
    const { error } = await supabase.from('orchids').update(payload).eq('id', editingOrchidId.value)
    if (!error) {
      alert('品種修改成功！')
      cancelEditOrchid()
      loadOrchids()
    } else {
      alert('修改失敗：' + error.message)
    }
  } else {
    const newId = generateDateSeqId('FL', orchids.value)
    const { error } = await supabase.from('orchids').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`品種新增成功！編號：${newId}`)
      cancelEditOrchid()
      loadOrchids()
    } else {
      alert('新增失敗：' + error.message)
    }
  }
}

// ==========================================
// 8. 退貨管理
// ==========================================
const loadReturns = async () => {
  const { data } = await supabase.from('returns').select('*').order('created_at', { ascending: false })
  if (data) returnList.value = data
}

const calcRetTotal = () => {
  const q = Number(formRet.value.qty) || 0
  const u = Number(formRet.value.unit_price) || 0
  formRet.value.total_amount = q * u
}

const startEditRet = (ret) => {
  editingRetId.value = ret.id
  formRet.value = {
    return_type: ret.return_type,
    party_name: ret.party_name,
    target_item: ret.target_item,
    qty: ret.qty,
    unit_price: ret.unit_price,
    total_amount: ret.total_amount,
    date: ret.date,
    reason: ret.reason
  }
  nextTick(() => {
    document.getElementById('return-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}

const cancelEditRet = () => {
  editingRetId.value = null
  formRet.value = {
    return_type: '退給花農',
    party_name: '',
    target_item: '',
    qty: 2,
    unit_price: 150,
    total_amount: 300,
    date: new Date().toISOString().split('T')[0],
    reason: '運送碰撞 / 開花不良'
  }
}

const saveReturn = async () => {
  if (!formRet.value.party_name) return alert('請輸入對象名稱！')
  calcRetTotal()
  const payload = {
    return_type: formRet.value.return_type,
    party_name: formRet.value.party_name,
    target_item: formRet.value.target_item,
    qty: formRet.value.qty,
    unit_price: formRet.value.unit_price,
    total_amount: formRet.value.total_amount,
    date: formRet.value.date,
    reason: formRet.value.reason
  }
  if (editingRetId.value) {
    const { error } = await supabase.from('returns').update(payload).eq('id', editingRetId.value)
    if (!error) {
      alert('退貨紀錄修改成功！')
      cancelEditRet()
      loadReturns()
    } else {
      alert('修改失敗：' + error.message)
    }
  } else {
    const newId = generateDateSeqId('RT', returnList.value)
    const { error } = await supabase.from('returns').insert([{ id: newId, ...payload }])
    if (!error) {
      alert(`退貨紀錄儲存成功！單號：${newId}`)
      cancelEditRet()
      loadReturns()
    } else {
      alert('新增失敗：' + error.message)
    }
  }
}

const deleteItem = async (table, id, reloadFn) => {
  if (!confirm(`確定要刪除編號 ${id} 嗎？此操作無法還原！`)) return
  const { error } = await supabase.from(table).delete().eq('id', id)
  if (!error) reloadFn()
}

const exportOrdersToExcel = () => {
  if (orderList.value.length === 0) return alert('目前尚無訂單可供匯出！')
  const exportData = orderList.value.map(o => ({
    '訂單編號': o.id,
    '客戶名稱': o.customer,
    '客戶類型': o.cust_type,
    '結帳週期': o.billing_cycle || '每單結',
    '聯絡電話': o.phone,
    '總盆數': getOrderTotalPots(o),
    '品項與規格': o.spec,
    '盆器': o.pot,
    '額外運費': getOrderShippingFee(o),
    '預估成本': o.cost,
    '訂單總售價': o.price,
    '利潤': o.price - o.cost,
    '統一編號': o.tax_id || '',
    '開收據與否': o.need_receipt || '不需收據',
    '訂單備註': o.note || '',
    '花卡狀態': o.card_status || '未製作',
    '簽收單狀態': o.receipt_status || '未列印',
    '下單日期': o.order_date,
    '預計送達': o.expected_date,
    '出貨狀態': o.shipped_status,
    '收款狀態': o.payment_status
  }))
  const worksheet = XLSX.utils.json_to_sheet(exportData)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '蘭花訂單總表')
  XLSX.writeFile(workbook, `宸豐蘭藝_全部訂單清單_${new Date().toISOString().split('T')[0]}.xlsx`)
}

// ==========================================
// 9. 花卡 / 輓聯編輯器
// ==========================================
const isVertical = ref(true)
const cardCategory = ref('funeral')
const zoomLevel = ref(0.7)
const viewportRef = ref(null)

const cardFontFamily = ref('kai')

const upperPrefix = ref('敬悼')
const upperTarget = ref('陳媽李老夫人')
const upperSuffix = ref('千古')

const middleText = ref('母儀千古')
const suffixText = ref('敬輓')

const funeralUpperFormat = ref('X媽X老夫人')
const celebrationType = ref('opening')
const celebPrefix = ref('恭祝')
const celebTarget = ref('鴻運實業有限公司')

const gender = ref('female')
const ageStage = ref('f_over80')

const weights = ref({
  upper_prefix: '600',
  upper_target: '700',
  upper_suffix: '600',
  middle: '800',
  bottom_0: '600',
  bottom_1: '700',
  bottom_2: '600',
  bottom_3: '600',
  bottom_4: '600',
  bottom_5: '600',
  suffix: '700'
})

const getWeightStyle = (wVal) => {
  const w = String(wVal || '600')
  const styles = { fontWeight: w }
  if (w === '500') {
    styles.textShadow = '0 0 0.4px #000'
  } else if (w === '600') {
    styles.textShadow = '0 0 0.8px #000'
  } else if (w === '700') {
    styles.textShadow = '0 0 1.2px #000, 0.3px 0.3px 0 #000'
  } else if (w === '800') {
    styles.textShadow = '0 0 1.8px #000, 0.5px 0.5px 0 #000, -0.5px 0 0 #000'
  }
  return styles
}

const fontMapping = {
  kai: '"TW-Kai", "DFKai-SB", "BiauKai", "Kaiti", serif',
  notosong: '"Noto Serif TC", "Songti TC", "SimSun", "PMingLiU", serif',
  fangsong: '"FangSong", "STFangsong", "華康仿宋體", serif',
  notosans: '"Noto Sans TC", "PingFang TC", "Microsoft JhengHei", sans-serif',
  systemkai: '"BiauKai", "DFKai-SB", "TW-Kai", serif'
}

const activeCssFontFamily = computed(() => fontMapping[cardFontFamily.value] || fontMapping.kai)

const bottomLines = ref([
  { text: '桃園市議會' },
  { text: '議員 李宗豪' },
  { text: '' },
  { text: '' },
  { text: '' },
  { text: '' }
])
const getPlaceholder = (idx) => [
  '第 1 格（例：單位 / 公司）',
  '第 2 格（例：職稱姓名 1）',
  '第 3 格（自訂聯名人 2）',
  '第 4 格（自訂）',
  '第 5 格（自訂）',
  '第 6 格（自訂）'
][idx]

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

const parsedUpperTargetTokens = computed(() => {
  const chars = upperTarget.value.split('')
  return chars.map(char => ({
    char,
    isSmall: char === '媽'
  }))
})

const maFontSize = computed(() => {
  const baseSize = layout.value.upper_target?.size || 40
  return Math.max(12, baseSize - 20)
})

const defaultVertical = {
  upper_prefix: { x: 620, y: 100, size: 36 },
  upper_target: { x: 620, y: 220, size: 42 },
  upper_suffix: { x: 620, y: 720, size: 36 },
  middle:       { x: 330, y: 220, size: 84 },
  bottom_0:     { x: 155, y: 520, size: 30 },
  bottom_1:     { x: 155, y: 680, size: 36 },
  bottom_2:     { x: 95,  y: 520, size: 30 },
  bottom_3:     { x: 95,  y: 680, size: 32 },
  bottom_4:     { x: 40,  y: 520, size: 30 },
  bottom_5:     { x: 40,  y: 680, size: 30 },
  suffix:       { x: 155, y: 920, size: 34 }
}

const defaultHorizontal = {
  upper_prefix: { x: 120, y: 90,  size: 34 },
  upper_target: { x: 230, y: 90,  size: 38 },
  upper_suffix: { x: 600, y: 90,  size: 34 },
  middle:       { x: 220, y: 260, size: 76 },
  bottom_0:     { x: 340, y: 400, size: 28 },
  bottom_1:     { x: 340, y: 460, size: 32 },
  bottom_2:     { x: 340, y: 520, size: 28 },
  bottom_3:     { x: 340, y: 580, size: 28 },
  bottom_4:     { x: 340, y: 640, size: 28 },
  bottom_5:     { x: 340, y: 700, size: 28 },
  suffix:       { x: 620, y: 490, size: 34 }
}

const layout = ref(JSON.parse(JSON.stringify(defaultVertical)))

const switchOrientation = (vertical) => {
  isVertical.value = vertical
  resetPositions()
  nextTick(() => autoFitZoom())
}
const resetPositions = () => {
  layout.value = JSON.parse(JSON.stringify(isVertical.value ? defaultVertical : defaultHorizontal))
}

const getStyle = (key) => {
  const item = (layout.value && layout.value[key]) ? layout.value[key] : { x: 50, y: 50, size: 30 }
  const weight = weights.value[key] || '600'
  return { 
    left: `${item.x}px`, 
    top: `${item.y}px`, 
    fontSize: `${item.size}px`,
    ...getWeightStyle(weight)
  }
}

const getUpperTargetBoxStyle = () => {
  const item = (layout.value && layout.value.upper_target) ? layout.value.upper_target : { x: 620, y: 220, size: 42 }
  const weight = weights.value.upper_target || '700'
  return {
    left: `${item.x}px`,
    top: `${item.y}px`,
    fontSize: `${item.size}px`,
    ...getWeightStyle(weight)
  }
}

const autoFitZoom = () => {
  if (!viewportRef.value) return
  const availableWidth = Math.max(viewportRef.value.clientWidth - 40, 280)
  const cardWidth = isVertical.value ? 794 : 1123
  zoomLevel.value = Math.min(Math.max(+(availableWidth / cardWidth).toFixed(2), 0.28), 1.0)
}

let activeKey = null
let currentAction = null
let startX = 0, startY = 0, originX = 0, originY = 0, originSize = 30

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
  } else if (currentAction === 'resize') {
    layout.value[activeKey].size = Math.max(14, Math.min(300, Math.round(originSize + (dx + dy) / 3)))
  }
}
const onPointerUp = () => {
  activeKey = null; currentAction = null
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
}

watch(gender, (val) => { ageStage.value = val === 'female' ? 'f_50_79' : 'm_50_69' })

const onFuneralFormatChange = () => {
  if (funeralUpperFormat.value !== 'custom') {
    upperTarget.value = funeralUpperFormat.value
  }
}

const onCardCategoryChange = () => {
  if (cardCategory.value === 'funeral') {
    upperPrefix.value = '敬悼'
    upperSuffix.value = '千古'
    suffixText.value = '敬輓'
    middleText.value = currentFuneralPhrases.value[0] || '母儀千古'
  } else {
    upperPrefix.value = '恭祝'
    upperSuffix.value = '誌慶'
    suffixText.value = '敬賀'
    middleText.value = currentCelebPhrases.value[0] || '開幕誌慶'
  }
}
const onCelebrationTypeChange = () => {
  middleText.value = currentCelebPhrases.value[0] || ''
}

const printCouplet = () => {
  nextTick(() => {
    window.print()
  })
}

// 產生花卡高畫質圖片
const shareCoupletToLineDirect = () => {
  const canvas = document.createElement('canvas')
  const width = isVertical.value ? 794 : 1123
  const height = isVertical.value ? 1123 : 794
  canvas.width = width * 2
  canvas.height = height * 2
  const ctx = canvas.getContext('2d')
  ctx.scale(2, 2)

  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, width, height)

  if (!isVertical.value && cardCategory.value === 'celebration') {
    ctx.lineWidth = 12
    ctx.strokeStyle = '#fce7f3'
    ctx.strokeRect(6, 6, width - 12, height - 12)
  }

  ctx.fillStyle = '#000000'
  const targetFontFamily = activeCssFontFamily.value

  const drawTextItem = (text, item, isVertMode, weightVal, isUpperTarget = false) => {
    if (!text || !item) return
    ctx.textBaseline = 'top'
    const w = String(weightVal || '600')
    const strokeWidthMap = { '400': 0, '500': 0.4, '600': 0.8, '700': 1.4, '800': 2.2 }
    const sWidth = strokeWidthMap[w] || 0.8

    if (isVertMode) {
      let currentY = item.y
      const chars = text.split('')
      chars.forEach(char => {
        const isMa = isUpperTarget && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${w} ${curSize}px ${targetFontFamily}`
        const offsetX = isMa ? Math.round((item.size - curSize) / 2) : 0
        
        if (sWidth > 0) {
          ctx.strokeStyle = '#000000'
          ctx.lineWidth = sWidth
          ctx.strokeText(char, item.x + offsetX, currentY)
        }
        ctx.fillText(char, item.x + offsetX, currentY)
        currentY += curSize + 8
      })
    } else {
      let currentX = item.x
      const chars = text.split('')
      chars.forEach(char => {
        const isMa = isUpperTarget && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${w} ${curSize}px ${targetFontFamily}`
        const offsetY = isMa ? Math.round((item.size - curSize) / 2) : 0
        
        if (sWidth > 0) {
          ctx.strokeStyle = '#000000'
          ctx.lineWidth = sWidth
          ctx.strokeText(char, currentX, item.y + offsetY)
        }
        ctx.fillText(char, currentX, item.y + offsetY)
        currentX += curSize + 4
      })
    }
  }

  drawTextItem(upperPrefix.value, layout.value.upper_prefix, isVertical.value, weights.value.upper_prefix)
  drawTextItem(upperTarget.value, layout.value.upper_target, isVertical.value, weights.value.upper_target, true)
  drawTextItem(upperSuffix.value, layout.value.upper_suffix, isVertical.value, weights.value.upper_suffix)
  drawTextItem(middleText.value, layout.value.middle, isVertical.value, weights.value.middle)

  bottomLines.value.forEach((item, idx) => {
    if (item.text.trim()) {
      drawTextItem(item.text, layout.value['bottom_' + idx], isVertical.value, weights.value['bottom_' + idx])
    }
  })

  drawTextItem(suffixText.value, layout.value.suffix, isVertical.value, weights.value.suffix)

  const filename = `花卡_${new Date().toISOString().split('T')[0]}.png`
  shareOrCopyCanvasBlob(canvas, filename, '花卡確認', '花卡圖片準備完成')
}

// 雲端字型預載入
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

  autoFitZoom()
  autoFitReceipt()
  autoFitFarmerReceipt()
  updateChineseAmount()
  fetchSealFromCloud()
  window.addEventListener('resize', () => {
    autoFitZoom()
    autoFitReceipt()
    autoFitFarmerReceipt()
  })
  loadOrchids()
  loadCustomers()
  loadInventory()
  loadReturns()
  loadOrders()
})
</script>

<style scoped>
.main-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9;
}

/* 浮動提示橫條 */
.floating-toast {
  position: fixed;
  top: 60px;
  right: 20px;
  background-color: #0f172a;
  color: white;
  padding: 12px 18px;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0,0,0,0.3);
  font-size: 13.5px;
  line-height: 1.5;
  white-space: pre-line;
  z-index: 10000;
  border-left: 4px solid #10b981;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

/* 圖片傳送專用彈窗 */
.share-preview-modal {
  max-width: 680px !important;
  width: 90vw !important;
}
.share-modal-body {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}
.share-preview-img {
  max-width: 100%;
  max-height: 55vh;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}
.share-tips-row {
  display: flex;
  flex-direction: column;
  gap: 4px;
  background: #f0fdf4;
  border: 1px solid #bbf7d0;
  color: #166534;
  padding: 10px 14px;
  border-radius: 6px;
  font-size: 13px;
  width: 100%;
  box-sizing: border-box;
}

/* 現場收件人手寫簽名板樣式 */
.live-sign-panel {
  background: #f0fdf4;
  border: 1.5px solid #86efac;
}
.canvas-sign-wrapper {
  background: white;
  border: 1.5px dashed #4ade80;
  border-radius: 6px;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
}
.live-sign-pad {
  touch-action: none;
  cursor: crosshair;
  background-color: #ffffff;
  display: block;
}
.mini-clean-btn {
  background: #ef4444;
  color: white;
  border: none;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
}
.sign-hint-text {
  font-size: 11.5px;
  color: #166534;
  margin-top: 6px;
  line-height: 1.4;
}

/* 簽收單上的手寫簽名圖片 */
.live-signature-img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  mix-blend-mode: multiply;
}

/* 跨平台書法正楷字體類別 */
.kai-font-supported {
  font-family: "TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "Kaiti", serif !important;
}

/* 頂端導航 */
.top-nav {
  height: 52px;
  background-color: #0f172a;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  flex-shrink: 0;
  overflow-x: auto;
}
.nav-title { font-size: 16px; font-weight: 900; white-space: nowrap; margin-right: 12px; }
.nav-tabs { display: flex; gap: 8px; }
.nav-tabs button {
  background: #334155;
  color: #e2e8f0;
  border: none;
  padding: 8px 14px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  font-size: 13px;
  white-space: nowrap;
}
.nav-tabs button.active { background: #2563eb; color: white; }

/* 蘭花管理後台 */
.manage-container { display: flex; flex-direction: column; flex: 1; overflow: hidden; }
.sub-nav {
  display: flex; background: #ffffff; border-bottom: 1px solid #e2e8f0;
  padding: 8px 16px; gap: 8px; overflow-x: auto;
}
.sub-nav button {
  background: #f8fafc; border: 1px solid #cbd5e1; padding: 6px 14px;
  border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; white-space: nowrap;
}
.sub-nav button.active { background: #10b981; color: white; border-color: #10b981; }
.manage-content { flex: 1; overflow-y: auto; padding: 16px; }

/* 編輯提示橫條 */
.edit-banner {
  background: #fef3c7; border: 1.5px solid #f59e0b; color: #92400e;
  padding: 10px 16px; border-radius: 8px; margin-bottom: 12px;
  display: flex; justify-content: space-between; align-items: center; font-size: 14px;
}
.cancel-edit-btn { background: #dc2626; color: white; border: none; padding: 4px 10px; border-radius: 4px; font-weight: bold; font-size: 12px; cursor: pointer; }

.card-box { background: white; border-radius: 8px; padding: 16px; box-shadow: 0 2px 6px rgba(0,0,0,0.04); }
.card-box h3 { margin: 0 0 12px 0; font-size: 16px; color: #1e293b; }
.form-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 12px; }
.field label { display: block; font-size: 12px; font-weight: bold; color: #475569; margin-bottom: 4px; }
input, select, textarea {
  width: 100%; padding: 8px 10px; border: 1px solid #cbd5e1;
  border-radius: 6px; font-size: 13px; box-sizing: border-box;
}

/* 多規格花禮卡片設計 */
.items-section {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 14px;
}
.items-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}
.items-header h4 {
  margin: 0;
  font-size: 14px;
  color: #1e293b;
  font-weight: bold;
}
.add-item-btn {
  background: #2563eb;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-weight: bold;
  font-size: 12px;
  cursor: pointer;
}
.order-item-card {
  background: white;
  border: 1.5px solid #cbd5e1;
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 10px;
}
.item-card-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
  font-weight: bold;
  color: #3b82f6;
  margin-bottom: 8px;
  padding-bottom: 4px;
  border-bottom: 1px dashed #e2e8f0;
}
.remove-item-btn {
  background: #fee2e2;
  color: #dc2626;
  border: 1px solid #fecaca;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 11px;
  cursor: pointer;
}
.item-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 10px;
}

.highlight-field {
  background-color: #f0fdf4;
  padding: 6px;
  border-radius: 6px;
  border: 1px solid #bbf7d0;
}
.highlight-field label { color: #15803d; }
.bold-price-input { font-weight: bold; color: #1d4ed8; font-size: 15px; }

.bold-select-field { font-weight: bold; color: #1e3a8a; background: #eff6ff; }
.text-purple { color: #7e22ce; }

.btn-action-row { display: flex; gap: 8px; }
.primary-btn { background: #2563eb; color: white; border: none; padding: 8px 18px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.secondary-btn { background: #94a3b8; color: white; border: none; padding: 8px 16px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.excel-btn { background: #059669; color: white; border: none; padding: 7px 14px; border-radius: 6px; font-weight: bold; cursor: pointer; white-space: nowrap; }
.line-action-btn { background: #06c755; color: white; border: none; padding: 10px; border-radius: 6px; font-weight: bold; font-size: 14px; cursor: pointer; width: 100%; }

/* 對帳專區樣式 */
.statement-filter-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 12px; background: #f8fafc; padding: 12px; border-radius: 6px;
}
.statement-summary-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; }
.sum-card { padding: 14px; border-radius: 8px; border: 1px solid #e2e8f0; }
.red-card { background: #fef2f2; border-color: #fecaca; }
.blue-card { background: #eff6ff; border-color: #bfdbfe; }
.green-card { background: #f0fdf4; border-color: #bbf7d0; }
.sum-label { font-size: 12px; font-weight: bold; color: #475569; margin-bottom: 4px; }
.sum-value { font-size: 20px; font-weight: 900; color: #0f172a; }
.sum-value.font-medium { font-size: 15px; }

.statement-actions { display: flex; gap: 10px; flex-wrap: wrap; }
.line-btn { background: #06c755; color: white; border: none; padding: 8px 16px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.batch-pay-btn { background: #ea580c; color: white; border: none; padding: 8px 16px; border-radius: 6px; font-weight: bold; cursor: pointer; }

/* 狀態徽章 */
.select-status { font-weight: bold; padding: 4px 6px; border-radius: 4px; }
.select-status.orange { background: #fffbeb; color: #b45309; border: 1px solid #fde68a; }
.select-status.green { background: #f0fdf4; color: #15803d; border: 1px solid #bbf7d0; }
.select-status.gray { background: #f1f5f9; color: #475569; border: 1px solid #cbd5e1; }

.table-header-action { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.table-responsive { overflow-x: auto; -webkit-overflow-scrolling: touch; }
.data-table { width: 100%; border-collapse: collapse; font-size: 13px; text-align: left; min-width: 750px; }
.data-table th { background: #f8fafc; padding: 8px 10px; border-bottom: 2px solid #e2e8f0; color: #475569; white-space: nowrap; }
.data-table td { padding: 8px 10px; border-bottom: 1px solid #e2e8f0; vertical-align: middle; }
.action-cell { white-space: nowrap; }

.badge { padding: 3px 8px; border-radius: 4px; font-size: 11px; font-weight: bold; border: none; cursor: pointer; }
.badge-purple { background: #f3e8ff; color: #7e22ce; }
.badge-red { background: #fee2e2; color: #dc2626; }
.badge-orange { background: #ffedd5; color: #c2410c; }
.badge-green { background: #dcfce7; color: #16a34a; }
.badge-gray { background: #f1f5f9; color: #64748b; }
.status-tag { font-size: 12px; font-weight: bold; }

.mini-btn { border: none; padding: 4px 8px; border-radius: 4px; cursor: pointer; margin-right: 4px; }
.edit-btn { background: #3b82f6; color: white; }
.del-btn { background: #ef4444; color: white; }
.print-btn { background: #f59e0b; color: white; font-weight: bold; font-size: 12px; }
.farmer-btn { background: #8b5cf6; color: white; font-weight: bold; font-size: 12px; }

.text-red { color: #dc2626; }
.text-blue { color: #2563eb; }
.text-green { color: #16a34a; }
.text-center { text-align: center; }
.text-gray { color: #94a3b8; }
.py-4 { padding: 16px 0; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 16px; }

/* 蘭花品種照片上傳 */
.photo-preview-wrap {
  display: flex; align-items: center; gap: 12px; background: #f8fafc;
  padding: 8px 12px; border-radius: 6px; border: 1px dashed #cbd5e1;
}
.preview-label { font-size: 12px; font-weight: bold; color: #475569; }
.preview-thumb { width: 56px; height: 56px; object-fit: cover; border-radius: 6px; border: 1px solid #cbd5e1; }
.remove-photo-btn { background: #ef4444; color: white; border: none; padding: 4px 8px; border-radius: 4px; font-size: 11px; cursor: pointer; }

.photo-col { width: 60px; text-align: center; }
.table-orchid-img { width: 44px; height: 44px; object-fit: cover; border-radius: 6px; border: 1px solid #e2e8f0; cursor: pointer; }
.no-photo-badge { font-size: 11px; color: #94a3b8; }

/* 照片燈箱 */
.image-modal-overlay {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background: rgba(0,0,0,0.65); display: flex; justify-content: center; align-items: center;
  z-index: 9999;
}
.image-modal-content {
  background: white; border-radius: 12px; padding: 16px; max-width: 90vw; max-height: 90vh;
  display: flex; flex-direction: column; box-shadow: 0 20px 40px rgba(0,0,0,0.4);
}
.image-modal-header { display: flex; justify-content: space-between; align-items: center; font-weight: bold; margin-bottom: 10px; }
.close-modal-btn { background: transparent; border: none; font-size: 20px; cursor: pointer; color: #64748b; }
.image-modal-img { max-width: 80vw; max-height: 75vh; object-fit: contain; border-radius: 8px; }

/* 簽收單與收據控制面板 */
.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.couplet-screen-wrapper {
  display: flex;
  flex: 1;
  overflow: hidden;
  height: calc(100vh - 52px);
}
.control-panel {
  width: 390px; background: white; padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 10px; border-radius: 6px; margin-bottom: 10px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }
.bold-select { font-weight: bold; font-size: 14px; border-color: #3b82f6; }

.section-title-with-weight {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}
.mini-weight-select {
  width: auto !important;
  padding: 2px 6px !important;
  font-size: 11px !important;
  font-weight: bold !important;
  color: #1e3a8a !important;
  background: #eff6ff !important;
  border: 1px solid #bfdbfe !important;
  border-radius: 4px !important;
}
.flex-input {
  flex: 1;
}

/* 雲端印章面板 */
.stamp-select-panel {
  background: #fdf2f8;
  border: 1.5px dashed #db2777;
}
.seal-choose-btn {
  width: 100%;
  margin-top: 6px;
  padding: 8px;
  background: #db2777;
  color: white;
  border: none;
  border-radius: 6px;
  font-weight: bold;
  font-size: 13px;
  cursor: pointer;
}

.section-title { font-size: 13px; font-weight: bold; margin-bottom: 6px; display: block; }
.form-group { margin-bottom: 10px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 4px; color: #334155; }
.bottom-input-group { display: flex; align-items: center; gap: 6px; margin-bottom: 6px; }
.line-num { font-size: 12px; font-weight: bold; color: #64748b; width: 38px; }
.form-row, .btn-group { display: flex; gap: 6px; }
.btn-group button {
  flex: 1; padding: 7px; border: 1px solid #2563eb; background: white;
  color: #2563eb; border-radius: 4px; cursor: pointer; font-weight: bold;
}
.btn-group button.active { background: #2563eb; color: white; }
.radio-row { display: flex; gap: 14px; font-size: 13px; }
.tags-container { display: flex; flex-wrap: wrap; gap: 4px; }
.tag-btn {
  background: #eff6ff; color: #1e40af; border: 1px solid #bfdbfe;
  padding: 3px 6px; font-size: 12px; border-radius: 4px; cursor: pointer;
}
.reset-btn { width: 100%; padding: 8px; background: #f1f5f9; border: 1px dashed #94a3b8; border-radius: 4px; cursor: pointer; }

/* 畫布視窗與自適應 */
.canvas-viewport {
  flex: 1; 
  display: flex; 
  flex-direction: column; 
  align-items: center;
  overflow: auto; 
  padding: 20px 20px 80px 20px; 
  position: relative; 
  background-color: #cbd5e1;
  -webkit-overflow-scrolling: touch;
}
.receipt-preview-area {
  flex: 1; display: flex; flex-direction: column; align-items: center;
  overflow: auto; padding: 16px; position: relative; background-color: #cbd5e1;
  -webkit-overflow-scrolling: touch;
}
.zoom-toolbar {
  display: flex; align-items: center; gap: 6px; background: white;
  padding: 5px 12px; border-radius: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.15); margin-bottom: 12px;
  position: sticky; top: 0; z-index: 10;
}
.zoom-btn { width: 26px; height: 26px; border: 1px solid #cbd5e1; background: #f8fafc; border-radius: 50%; cursor: pointer; }
.zoom-text { font-size: 13px; font-weight: bold; min-width: 44px; text-align: center; }
.fit-btn { background: #2563eb; color: white; border: none; padding: 4px 10px; border-radius: 12px; font-size: 12px; cursor: pointer; }

/* 標準 A4 卡片 (794x1123，對應 210mm x 297mm) */
.card-scaler-container { 
  position: relative; 
  margin-bottom: 40px; 
  flex-shrink: 0;
}
.card-board { 
  background: #fff; 
  position: absolute; 
  box-shadow: 0 10px 30px rgba(0,0,0,0.18); 
  user-select: none; 
  touch-action: none; 
}
.card-board.mode-vertical { width: 794px; height: 1123px; }
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; }
.card-board.mode-vertical .middle-box { letter-spacing: 20px; }
.card-board.mode-horizontal { width: 1123px; height: 794px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.card-board.mode-horizontal .middle-box { letter-spacing: 16px; }
.card-board.style-floral { border: 12px solid #fce7f3; }

.text-box { position: absolute; cursor: move; padding: 4px 6px; white-space: nowrap; line-height: 1.25; color: #000; }
.text-box:hover { outline: 1px dashed #2563eb; background: rgba(37, 99, 235, 0.04); }
.scale-handle {
  position: absolute; right: -7px; bottom: -7px; width: 17px; height: 17px;
  background: #2563eb; color: white; border-radius: 3px; font-size: 11px;
  display: flex; justify-content: center; align-items: center; cursor: nwse-resize;
}

.small-ma {
  font-size: 0.65em !important;
  display: inline-block;
  vertical-align: middle;
}

/* A5 橫式簽收單 */
.receipt-scaler-container { position: relative; }
.a5-landscape-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 38px 45px;
  box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between;
  writing-mode: horizontal-tb; direction: ltr;
  color: #111827; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.sheet-header {
  display: flex; justify-content: space-between; align-items: flex-end;
  border-bottom: 2.5px solid #1e293b; padding-bottom: 8px;
}
.shop-name-title { font-size: 26px; font-weight: 900; letter-spacing: 2px; color: #0f172a; }
.sheet-main-title { font-size: 22px; font-weight: bold; letter-spacing: 4px; color: #dc2626; }
.header-meta { font-size: 13px; line-height: 1.5; text-align: right; color: #334155; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 10px 0; font-size: 15px; table-layout: fixed; }
.receipt-table td { border: 1.5px solid #334155; padding: 8px 10px; word-break: break-all; }
.receipt-table .lbl { width: 15%; background-color: #f1f5f9; font-weight: bold; text-align: center; color: #1e293b; }
.receipt-table .val { width: 35%; }
.receipt-table .val-bold { font-weight: bold; font-size: 16px; }
.receipt-table .val-highlight { font-weight: bold; color: #1e3a8a; }

.sheet-footer { display: flex; justify-content: space-between; align-items: stretch; gap: 16px; }
.footer-left { flex: 1; display: flex; flex-direction: column; justify-content: space-between; font-size: 14px; padding: 4px 0; }
.footer-tip { font-size: 12px; color: #64748b; }
.footer-sign-box {
  width: 220px; border: 1.5px dashed #475569; border-radius: 6px;
  display: flex; flex-direction: column; background-color: #fafafa;
}
.sign-box-title { background: #e2e8f0; font-size: 12px; font-weight: bold; text-align: center; padding: 3px 0; color: #334155; }
.sign-box-area { flex: 1; min-height: 52px; display: flex; justify-content: center; align-items: center; }

/* ====================================================
   農民出售農產品收據
   ==================================================== */
.farmer-scaler-container { position: relative; }
.farmer-receipt-sheet {
  width: 794px;
  height: 560px;
  background: #ffffff;
  padding: 18px 28px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  color: #000;
  box-shadow: 0 8px 24px rgba(0,0,0,0.15);
  position: absolute;
  top: 0;
  left: 0;
}

.f-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  margin-bottom: 12px;
}
.f-main-title {
  font-size: 26px;
  font-weight: 900;
  letter-spacing: 5px;
  text-align: center;
}
.f-date-wrap {
  align-self: flex-end;
  font-size: 15px;
  letter-spacing: 2px;
  margin-top: 10px;
}

.f-receipt-grid-table {
  border: 2px solid #000;
  display: flex;
  flex-direction: column;
  font-size: 14.5px;
}

.f-grid-row {
  display: flex;
  border-bottom: 1px solid #000;
  min-height: 29px;
}
.f-grid-row:last-child {
  border-bottom: none;
}

.f-grid-lbl {
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  letter-spacing: 2px;
  text-align: center;
  border-right: 1px solid #000;
  padding: 3px 5px;
  box-sizing: border-box;
  flex-shrink: 0;
}

.f-grid-val {
  display: flex;
  align-items: center;
  padding-left: 10px;
  border-right: 1px solid #000;
  box-sizing: border-box;
}
.f-grid-val:last-child {
  border-right: none;
}

.f-flex-1 { flex: 1; }

.f-row-top { min-height: 62px; }
.f-col-buyer-group {
  display: flex;
  flex-direction: column;
  width: 58%;
  border-right: 1px solid #000;
}
.f-sub-row {
  display: flex;
  flex: 1;
  border-bottom: 1px solid #000;
}
.f-sub-row:last-child {
  border-bottom: none;
}

.f-w-head { width: 145px; }
.f-w-addr-tag { width: 36px; line-height: 1.4; }
.f-full-addr-box {
  flex: 1;
  padding: 8px 12px;
  font-size: 14.5px;
  line-height: 1.5;
  border-right: none !important;
}

.f-tax-clean {
  font-size: 17px;
  font-weight: bold;
  letter-spacing: 3px;
  color: #1e3a8a;
}

.col-p-name { width: 25%; }
.col-p-spec { width: 16%; }
.col-p-qty  { width: 10%; }
.col-p-price{ width: 14%; }
.col-p-amt  { width: 18%; }
.col-p-note { width: 17%; border-right: none !important; }

.f-header-row { font-weight: bold; height: 28px; }
.f-data-row { height: 30px; }
.f-empty-row { height: 28px; }

.f-w-total-lbl { 
  width: 195px; 
  white-space: nowrap; 
  font-size: 14.5px;
}
.f-amount-val-cell {
  flex: 1;
  border-right: none !important;
  padding: 3px 12px;
}
.f-chinese-amount-line {
  display: flex;
  align-items: center;
  justify-content: space-around;
  width: 100%;
  font-size: 16px;
  font-weight: bold;
}
.f-chinese-amount-line .d-val {
  color: #1e3a8a;
  min-width: 26px;
  text-align: center;
  font-size: 17px;
}

/* 蔡鎮遠姓名與真實蓋章圖片排版 (採用您的真實印章) */
.f-farmer-stamp-cell {
  border-right: none !important;
  padding-left: 28px !important;
  display: flex;
  align-items: center;
  gap: 16px;
}
.f-farmer-name-clean {
  font-size: 18px;
  letter-spacing: 6px;
  font-weight: bold;
}
.cai-real-stamp-img {
  width: 50px;
  height: 50px;
  object-fit: contain;
  mix-blend-mode: multiply;
}

.f-w-id-lbl { width: 190px; }
.f-w-id-val { width: 190px; border-right: none !important; }

.f-text-center { justify-content: center; text-align: center; }
.f-text-right { justify-content: flex-end; text-align: right; }
.f-bold { font-weight: bold; }
.f-pr { padding-right: 12px !important; }

.f-statement {
  font-size: 12px;
  text-align: center;
  letter-spacing: 1px;
  font-weight: bold;
  margin-top: 4px;
}

.f-footer-note {
  font-size: 10px;
  line-height: 1.4;
  color: #222;
  margin-top: 4px;
  text-align: justify;
}

.print-action-btn {
  width: 100%; padding: 12px; background: #16a34a; color: white;
  border: none; border-radius: 6px; font-size: 15px; font-weight: bold; cursor: pointer;
}
.print-action-btn:disabled { background: #cbd5e1; cursor: not-allowed; }

@media (max-width: 768px) {
  .app-container, .receipt-container, .couplet-screen-wrapper { flex-direction: column; overflow-y: auto; height: auto; }
  .control-panel { width: 100%; max-height: 46vh; }
  .form-grid { grid-template-columns: 1fr; }
  .canvas-viewport, .receipt-preview-area { padding: 12px 6px 60px 6px; }
}

/* 全域精準列印樣式 (純淨 A4 1:1 列印) */
@media print {
  @page { 
    size: A4 portrait; 
    margin: 0 !important;
  }
  
  html, body, .main-wrapper, .couplet-screen-wrapper { 
    margin: 0 !important; 
    padding: 0 !important; 
    background: white !important; 
    width: 210mm !important;
    height: 297mm !important;
    overflow: visible !important;
    display: block !important;
  }
  
  .no-print { display: none !important; }
  
  .canvas-viewport, .receipt-preview-area { 
    padding: 0 !important; 
    margin: 0 !important;
    background: white !important; 
    overflow: visible !important; 
    display: block !important;
    width: 210mm !important;
    height: 297mm !important;
  }
  
  .card-scaler-container { 
    width: 210mm !important; 
    height: 297mm !important; 
    position: static !important;
    margin: 0 !important;
    padding: 0 !important;
  }
  
  #card-print-target.mode-vertical { 
    position: absolute !important; 
    top: 0 !important;
    left: 0 !important;
    width: 210mm !important; 
    height: 297mm !important; 
    transform: none !important; 
    box-shadow: none !important; 
    margin: 0 !important;
    display: block !important;
    visibility: visible !important;
    page-break-inside: avoid !important;
    page-break-after: avoid !important;
  }

  #card-print-target.mode-horizontal {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    width: 297mm !important;
    height: 210mm !important;
    transform: none !important;
    box-shadow: none !important;
    margin: 0 !important;
    display: block !important;
    visibility: visible !important;
  }

  #card-print-target * {
    visibility: visible !important;
  }

  .a5-landscape-sheet, .farmer-receipt-sheet { 
    position: relative !important; 
    transform: none !important; 
    box-shadow: none !important; 
    width: 210mm !important; 
    height: 148mm !important; 
    margin: 0 auto !important;
  }
}
</style>