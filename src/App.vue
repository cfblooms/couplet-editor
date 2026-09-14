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
                    <th>單號</th>
                    <th>客戶名稱</th>
                    <th>統編</th>
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
                    <th>單號</th><th>下單日</th><th>客戶名稱</th><th>統編</th><th>開收據</th><th>品項規格</th><th>金額</th><th>收款狀態</th><th>操作</th>
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
                  <tr v-if="statementOrders.length === 0">
                    <td colspan="9" class="text-center py-4 text-gray">符合條件的訂單為 0 筆</td>
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

    <!-- ================= 模式 2：花卡 / 輓聯編輯器 ================= -->
    <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
      <div class="control-panel no-print">
        <h2>⚙️ 卡片與題詞設定</h2>

        <div class="panel-section">
          <label class="section-title">字體與粗細設定：</label>
          <div class="form-group">
            <label>選擇字體：</label>
            <select v-model="cardFontFamily">
              <option value="kai">標楷體 (雲端書法正楷・全平台通用)</option>
              <option value="fangsong">華康仿宋 (古典仿宋體)</option>
              <option value="dfkai_w7">華康楷書 (DFBiaoKaiShu / DFKaiShuW7)</option>
              <option value="df_yankai">華康正顏楷體 (顏真卿厚重書法楷)</option>
              <option value="wending">文鼎楷書 (典雅硬筆楷體)</option>
              <option value="notosong">思源宋體 (現代精細宋體)</option>
            </select>
          </div>
          <div class="form-group">
            <label>文字粗細 (字重)：</label>
            <select v-model="cardFontWeight">
              <option value="400">標準 (Regular 400)</option>
              <option value="600">半粗體 (Semi-Bold 600)</option>
              <option value="700">粗體 (Bold 700)</option>
              <option value="800">特粗體 (Extra-Bold 800)</option>
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

        <template v-if="cardCategory === 'funeral'">
          <div class="panel-section">
            <label class="section-title">上款稱謂組合：</label>
            <div class="form-row">
              <select v-model="funeralUpperFormat" @change="buildFuneralUpper">
                <option value="敬悼 X媽X老夫人">敬悼 X媽X老夫人</option>
                <option value="敬悼 X媽X夫人">敬悼 X媽X夫人</option>
                <option value="敬悼 X公X老先生">敬悼 X公X老先生</option>
                <option value="敬悼 X公X先生">敬悼 X公X先生</option>
                <option value="敬悼 X女士">敬悼 X女士</option>
                <option value="敬悼 X先生">敬悼 X先生</option>
                <option value="custom">自行輸入</option>
              </select>
              <select v-model="funeralUpperSuffix" @change="buildFuneralUpper">
                <option value="千古">千古</option>
                <option value="仙逝">仙逝</option>
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
            <label class="section-title">中款常用語 (身分與年齡)：</label>
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

        <div class="panel-section">
          <label class="section-title">下款設定（共 6 格自由填寫，空白不印出）：</label>
          <div v-for="(item, idx) in bottomLines" :key="idx" class="bottom-input-group">
            <span class="line-num">格 {{ idx + 1 }}</span>
            <input type="text" v-model="item.text" :placeholder="getPlaceholder(idx)" />
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
        <button type="button" class="line-action-btn mt-2" @click="shareCoupletToLineDirect">💬 直接傳送 / 複製花卡給客人 (免下載)</button>
        <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印 A4 花卡 / 輓聯</button>
      </div>

      <div class="canvas-viewport" ref="viewportRef">
        <div class="zoom-toolbar no-print">
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.25, +(zoomLevel - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
          <button type="fit-btn" @click="zoomLevel = 1.0">🔍 100% 檢視</button>
          <button type="button" class="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
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
            class="card-board kai-font-supported" 
            :class="[
              isVertical ? 'mode-vertical' : 'mode-horizontal',
              !isVertical && cardCategory === 'celebration' ? 'style-floral' : ''
            ]"
            :style="{
              transform: `scale(${zoomLevel})`,
              transformOrigin: 'top left',
              fontFamily: activeCssFontFamily,
              fontWeight: cardFontWeight
            }"
          >
            <!-- 上款 -->
            <div 
              v-if="upperText.trim()"
              class="text-box upper-box"
              :style="getUpperBoxStyle()"
              @pointerdown="startMove($event, 'upper')"
            >
              <span 
                v-for="(token, tIdx) in parsedUpperTokens" 
                :key="tIdx" 
                :style="token.isSmall ? { fontSize: maFontSize + 'px' } : {}"
              >
                {{ token.char }}
              </span>
              <div class="scale-handle no-print" @pointerdown.stop="startResize($event, 'upper')">⤡</div>
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

    <!-- ================= 模式 3：A5 橫式簽收單 (品項規格：特選蘭花 1盆、2盆、3盆) ================= -->
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
            <input type="text" v-model="receiptForm.item" placeholder="例：特選蘭花 1盆" />
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
          class="print-action-btn" 
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
                <div class="footer-tip">※ 專車親送・現場點交・祝頌商祺</div>
              </div>
              <div class="footer-sign-box">
                <div class="sign-box-title">客戶簽收章 / 簽名欄</div>
                <div class="sign-box-area"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= 模式 4：農民收據 (100% 讀取您 public/cai-seal.png 的真實印章) ================= -->
    <div v-else-if="currentTab === 'farmer_receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>🧾 農民出售農產品收據管理</h2>

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
          💬 直接傳送 / 複製收據給客人 (免下載)
        </button>

        <button 
          type="button" 
          class="print-action-btn mt-2" 
          @click="printFarmerReceipt"
        >
          🖨️ 列印農民收據
        </button>
      </div>

      <!-- 右側預覽區 (使用真實蔡鎮遠蓋章圖片) -->
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
                  <!-- 直接讀取 public 資料夾內的 cai-seal.png -->
                  <img src="/cai-seal.png" class="cai-real-stamp-img" alt="蔡鎮遠印章" />
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

// ----------------- Supabase 連線 -----------------
const supabaseUrl = 'https://ivofrjibdezbyxxmutok.supabase.co'
const supabaseKey = 'sb_publishable_b9oJamVY0UutjpXogYH6tQ_W4iuOiyr'
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
  if (potMatch) {
    potCount = parseInt(potMatch[1]) || 1
  }

  return `${flowerName} ${potCount}盆`
}

// 產生「當日日期 + 流水號」
const generateDateSeqId = (prefix, existingList) => {
  const now = new Date()
  const y = now.getFullYear()
  const m = String(now.getMonth() + 1).padStart(2, '0')
  const d = String(now.getDate()).padStart(2, '0')
  const dateStr = `${y}${m}${d}`
  const targetPrefix = `${prefix}-${dateStr}-`

  const todayItems = existingList.filter(item => String(item.id || '').startsWith(targetPrefix))
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
const formOrder = ref({
  cust_type: '批發',
  customer: '',
  billing_cycle: '每單結',
  phone: '0912-345678',
  orchid_name: '',
  stalks: 10,
  unit_price: 250,
  pot: '桌上盆 (100)',
  quick_pot: '未使用',
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
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0]
})

const flowerInventory = computed(() => inventoryList.value.filter(i => i.category === '蘭花'))

// ==========================================
// 1. 訂單模組
// ==========================================
const loadOrders = async () => {
  const { data } = await supabase.from('orders').select('*').order('created_at', { ascending: false })
  if (data) orderList.value = data
}

const calcOrderPrice = () => {
  const s = Number(formOrder.value.stalks) || 0
  const u = Number(formOrder.value.unit_price) || 0
  if (u > 0) {
    formOrder.value.price = s * u
  }
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
  const hasQuick = (ord.pot || '').includes('快捷盆')
  const cleanPot = (ord.pot || '').replace(/\s*\+\s*快捷盆/, '').trim()

  formOrder.value = {
    cust_type: ord.cust_type || '批發',
    customer: ord.customer || '',
    billing_cycle: ord.billing_cycle || '每單結',
    phone: ord.phone || '',
    orchid_name: ord.spec ? ord.spec.split('|')[0].trim() : '',
    stalks: 10,
    unit_price: 250,
    pot: cleanPot || '桌上盆 (100)',
    quick_pot: hasQuick ? '使用快捷盆 (70)' : '未使用',
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
    expected_date: ord.expected_date
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
    orchid_name: '',
    stalks: 10,
    unit_price: 250,
    pot: '桌上盆 (100)',
    quick_pot: '未使用',
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
    expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0]
  }
}

const saveOrder = async () => {
  if (!formOrder.value.customer) return alert('請輸入客戶名稱！')
  
  const specStr = formOrder.value.unit_price > 0
    ? `${formOrder.value.orchid_name || '特選蘭花'} | ${formOrder.value.stalks}棵 (單價${formOrder.value.unit_price}元)`
    : `${formOrder.value.orchid_name || '特選蘭花'} | ${formOrder.value.stalks}棵`

  const finalPotStr = formOrder.value.pot + (formOrder.value.quick_pot === '使用快捷盆 (70)' ? ' + 快捷盆' : '')

  const payload = {
    cust_type: formOrder.value.cust_type,
    customer: formOrder.value.customer,
    billing_cycle: formOrder.value.billing_cycle,
    phone: formOrder.value.phone,
    spec: specStr,
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
    text += `${index + 1}. [${o.order_date}] ${o.spec} (${o.pot}) ➔ $${o.price}元 (${o.payment_status})\n`
  })
  text += `--------------------------------\n`
  text += `※ 敬請核對帳款，感謝您的支持與惠顧！`

  navigator.clipboard.writeText(text).then(() => {
    alert('✅ LINE 對帳明細已複製到剪貼簿！可直接貼給客戶！')
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
    '品項與規格': o.spec,
    '盆器': o.pot,
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
    alert('✅ 已批次結清成功！')
    loadOrders()
  } else {
    alert('更新失敗：' + error.message)
  }
}

// ==========================================
// 3. A5 橫式簽收單
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
    farmerReceipt.value.qty = '1 盆'
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

const shareOrCopyCanvasBlob = async (canvas, filename, shareTitle, successMsg) => {
  canvas.toBlob(async (blob) => {
    if (!blob) return alert('圖片生成失敗，請重試！')
    const file = new File([blob], filename, { type: 'image/png' })

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

    if (navigator.clipboard && navigator.clipboard.write) {
      try {
        await navigator.clipboard.write([
          new ClipboardItem({ 'image/png': blob })
        ])
        alert(`✅ ${successMsg}\n\n已直接複製到您的電腦剪貼簿！請直接開啟 LINE 聊天室按 Ctrl + V (Mac 按 Cmd + V) 貼上即可傳送！`)
        return
      } catch (err) {
        console.warn('Clipboard write failed, fallback to download', err)
      }
    }

    const link = document.createElement('a')
    link.download = filename
    link.href = canvas.toDataURL('image/png')
    link.click()
    alert(`✅ ${successMsg} 已自動下載，請開啟 LINE 傳送給客戶！`)
  }, 'image/png')
}

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
  const targetWeight = cardFontWeight.value

  const drawTextItem = (text, item, isVertMode, isUpper = false) => {
    if (!text) return
    ctx.textBaseline = 'top'
    if (isVertMode) {
      let currentY = item.y
      const chars = text.split('')
      chars.forEach(char => {
        const isMa = isUpper && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${targetWeight} ${curSize}px ${targetFontFamily}`
        const offsetX = isMa ? Math.round((item.size - curSize) / 2) : 0
        ctx.fillText(char, item.x + offsetX, currentY)
        currentY += curSize + 8
      })
    } else {
      let currentX = item.x
      const chars = text.split('')
      chars.forEach(char => {
        const isMa = isUpper && char === '媽'
        const curSize = isMa ? Math.max(12, item.size - 20) : item.size
        ctx.font = `${targetWeight} ${curSize}px ${targetFontFamily}`
        const offsetY = isMa ? Math.round((item.size - curSize) / 2) : 0
        ctx.fillText(char, currentX, item.y + offsetY)
        currentX += curSize + 4
      })
    }
  }

  drawTextItem(upperText.value, layout.value.upper, isVertical.value, true)
  drawTextItem(middleText.value, layout.value.middle, isVertical.value)

  bottomLines.value.forEach((item, idx) => {
    if (item.text.trim()) {
      drawTextItem(item.text, layout.value['bottom_' + idx], isVertical.value)
    }
  })

  drawTextItem(suffixText.value, layout.value.suffix, isVertical.value)

  const filename = `花卡_${new Date().toISOString().split('T')[0]}.png`
  shareOrCopyCanvasBlob(canvas, filename, '花卡確認', '花卡圖片準備完成')
}

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
  
  const stampImg = new Image()
  stampImg.crossOrigin = 'anonymous'
  stampImg.onload = () => {
    ctx.drawImage(stampImg, 260, 305, 50, 50)
    ctx.font = `14.5px ${fontFam}`
    ctx.fillText(`住址：                     國民統一身分證編號：F129940801`, 42, 368)
    ctx.font = `11px ${fontFam}`
    ctx.fillText(`本收據之農民身分確實無誤，若有不實者願依法受罰。`, 42, 410)
    ctx.font = `9.5px ${fontFam}`
    ctx.fillText(`附註：依據財政部 68.11.2 台財稅第三七六六五號函：自 68 年 11 月 16 日起，凡農民出售其本身所生產、捕獲或畜養之農林漁牧產品所出具之收據，一律免納印花稅...`, 42, 435)

    const filename = `農民收據_${farmerReceipt.value.buyerName}_${farmerReceipt.value.year}${farmerReceipt.value.month}${farmerReceipt.value.day}.png`
    shareOrCopyCanvasBlob(canvas, filename, '農民收據確認', '農民收據圖片準備完成')
  }
  stampImg.onerror = () => {
    ctx.font = `14.5px ${fontFam}`
    ctx.fillText(`住址：                     國民統一身分證編號：F129940801`, 42, 368)
    ctx.font = `11px ${fontFam}`
    ctx.fillText(`本收據之農民身分確實無誤，若有不實者願依法受罰。`, 42, 410)
    ctx.font = `9.5px ${fontFam}`
    ctx.fillText(`附註：依據財政部 68.11.2 台財稅第三七六六五號函：自 68 年 11 月 16 日起，凡農民出售其本身所生產、捕獲或畜養之農林漁牧產品所出具之收據，一律免納印花稅...`, 42, 435)

    const filename = `農民收據_${farmerReceipt.value.buyerName}_${farmerReceipt.value.year}${farmerReceipt.value.month}${farmerReceipt.value.day}.png`
    shareOrCopyCanvasBlob(canvas, filename, '農民收據確認', '農民收據圖片準備完成')
  }
  stampImg.src = '/cai-seal.png'
}

const exportCoupletImage = shareCoupletToLineDirect
const exportFarmerReceiptImage = shareFarmerReceiptToLineDirect

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
  const total = formRet.value.qty * formRet.value.unit_price
  const payload = {
    return_type: formRet.value.return_type,
    party_name: formRet.value.party_name,
    target_item: formRet.value.target_item,
    qty: formRet.value.qty,
    unit_price: formRet.value.unit_price,
    total_amount: total,
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

// 匯出全部訂單 Excel
const exportOrdersToExcel = () => {
  if (orderList.value.length === 0) return alert('目前尚無訂單可供匯出！')
  const exportData = orderList.value.map(o => ({
    '訂單編號': o.id,
    '客戶名稱': o.customer,
    '客戶類型': o.cust_type,
    '結帳週期': o.billing_cycle || '每單結',
    '聯絡電話': o.phone,
    '品種與規格': o.spec,
    '盆器與快捷盆': o.pot,
    '預估成本': o.cost,
    '訂單售價': o.price,
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
const cardFontWeight = ref('700')

const fontMapping = {
  kai: '"TW-Kai", "MOESong-Regular", "DFKai-SB", "BiauKai", "Kaiti", serif',
  fangsong: '"DFPFangSong-B5", "DFPKai-B5", "FangSong", "STFangsong", "華康仿宋體", "仿宋", serif',
  dfkai_w7: '"DFBiaoKaiShu", "DFKaiShu-W7", "DFPKaiShu-W7", "DFKaiShuW7", "華康楷書體", "TW-Kai", "DFKai-SB", "BiauKai", serif',
  df_yankai: '"DFYanKai-W7", "DFYanKai", "DFPYanKai-W7", "DFPYanKai", "華康正顏楷體", "DFBiaoKaiShu", "TW-Kai", "DFKai-SB", serif',
  wending: '"AR PL UKai TW", "AR PL KaitiM Big5", "文鼎楷書", "TW-Kai", "DFKai-SB", serif',
  notosong: '"Noto Serif TC", "Songti TC", "SimSun", serif'
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

const funeralUpperFormat = ref('敬悼 X媽X老夫人')
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

const celebrationType = ref('opening')
const celebPrefix = ref('恭祝')
const celebTarget = ref('鴻運實業有限公司')
const celebPhrases = {
  opening: ['開幕誌慶', '開張大吉', '鴻圖大展', '駿業宏開', '生意興隆', '財源廣進', '客似雲來'],
  moving: ['喬遷之喜', '里仁為美', '金玉滿堂'],
  temple: ['聖誕千秋', '神威顯赫']
}
const currentCelebPhrases = computed(() => celebPhrases[celebrationType.value] || [])

const upperText = ref('敬悼 陳媽李老夫人 千古')
const middleText = ref('母儀千古')
const suffixText = ref('敬輓')

const parsedUpperTokens = computed(() => {
  const chars = upperText.value.split('')
  return chars.map(char => ({
    char,
    isSmall: char === '媽'
  }))
})

const maFontSize = computed(() => {
  const baseSize = layout.value.upper?.size || 40
  return Math.max(12, baseSize - 20)
})

// 標準 A4 寬高：直式 794x1123，橫式 1123x794
const defaultVertical = {
  upper:    { x: 620, y: 120, size: 40 },
  middle:   { x: 330, y: 220, size: 84 },
  bottom_0: { x: 155, y: 520, size: 30 },
  bottom_1: { x: 155, y: 680, size: 36 },
  bottom_2: { x: 95,  y: 520, size: 30 },
  bottom_3: { x: 95,  y: 680, size: 32 },
  bottom_4: { x: 40,  y: 520, size: 30 },
  bottom_5: { x: 40,  y: 680, size: 30 },
  suffix:   { x: 155, y: 920, size: 34 }
}
const defaultHorizontal = {
  upper:    { x: 180, y: 100, size: 36 },
  middle:   { x: 220, y: 260, size: 76 },
  bottom_0: { x: 340, y: 400, size: 28 },
  bottom_1: { x: 340, y: 460, size: 32 },
  bottom_2: { x: 340, y: 520, size: 28 },
  bottom_3: { x: 340, y: 580, size: 28 },
  bottom_4: { x: 340, y: 640, size: 28 },
  bottom_5: { x: 340, y: 700, size: 28 },
  suffix:   { x: 620, y: 490, size: 34 }
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
  const item = layout.value[key] || { x: 50, y: 50, size: 30 }
  return { 
    left: `${item.x}px`, 
    top: `${item.y}px`, 
    fontSize: `${item.size}px`,
    fontWeight: cardFontWeight.value
  }
}

const getUpperBoxStyle = () => {
  const item = layout.value.upper || { x: 620, y: 120, size: 40 }
  return {
    left: `${item.x}px`,
    top: `${item.y}px`,
    fontSize: `${item.size}px`,
    fontWeight: cardFontWeight.value
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
  if (!activeKey) return
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
const buildFuneralUpper = () => {
  if (funeralUpperFormat.value !== 'custom') upperText.value = `${funeralUpperFormat.value} ${funeralUpperSuffix.value}`
}
const buildCelebrationUpper = () => {
  if (celebPrefix.value !== 'custom') upperText.value = `${celebPrefix.value} ${celebTarget.value}`
}
const onCardCategoryChange = () => {
  if (cardCategory.value === 'funeral') {
    suffixText.value = '敬輓'; buildFuneralUpper(); middleText.value = currentFuneralPhrases.value[0] || ''
  } else {
    suffixText.value = '敬賀'; buildCelebrationUpper(); middleText.value = currentCelebPhrases.value[0] || ''
  }
}
const onCelebrationTypeChange = () => { middleText.value = currentCelebPhrases.value[0] || '' }

const printCouplet = () => {
  nextTick(() => {
    window.print()
  })
}

onMounted(() => {
  if (!document.getElementById('cns11643-tw-kai-font')) {
    const style = document.createElement('style')
    style.id = 'cns11643-tw-kai-font'
    style.innerHTML = `
      @font-face {
        font-family: 'TW-Kai';
        src: url('https://cdn.jsdelivr.net/gh/fontsource/tw-kai/files/tw-kai-400-normal.woff2') format('woff2'),
             url('https://cdn.jsdelivr.net/gh/fontsource/tw-kai/files/tw-kai-400-normal.woff') format('woff');
        font-weight: 400;
        font-style: normal;
        font-display: swap;
      }
    `
    document.head.appendChild(style)
  }

  if (!document.getElementById('noto-serif-tc-font')) {
    const link = document.createElement('link')
    link.id = 'noto-serif-tc-font'
    link.rel = 'stylesheet'
    link.href = 'https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;600;700;800&display=swap'
    document.head.appendChild(link)
  }

  autoFitZoom()
  autoFitReceipt()
  autoFitFarmerReceipt()
  updateChineseAmount()
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
  width: 380px; background: white; padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 10px; border-radius: 6px; margin-bottom: 10px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }
.bold-select { font-weight: bold; font-size: 14px; border-color: #3b82f6; }

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
.receipt-table { width: 100%; border-collapse: collapse; margin: 10px 0; font-size: 15px; }
.receipt-table td { border: 1.5px solid #334155; padding: 8px 10px; }
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
.sign-box-area { flex: 1; min-height: 48px; }

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

/* 蔡鎮遠姓名與真實蓋章圖片排版 */
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
  width: 48px;
  height: 48px;
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

/* ====================================================
   全域精準列印樣式
   ==================================================== */
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