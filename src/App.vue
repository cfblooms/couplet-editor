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
      </div>
    </header>

    <!-- ================= 模式 1：蘭花管理系統 (五大模組) ================= -->
    <div v-if="currentTab === 'manage'" class="manage-container no-print">
      <nav class="sub-nav">
        <button :class="{ active: subTab === 'order' }" @click="subTab = 'order'">💰 1. 訂單與帳務</button>
        <button :class="{ active: subTab === 'statement' }" @click="subTab = 'statement'">📊 2. 客戶未結對帳專區</button>
        <button :class="{ active: subTab === 'inventory' }" @click="subTab = 'inventory'">📦 3. 進貨與庫存</button>
        <button :class="{ active: subTab === 'customer' }" @click="subTab = 'customer'">👥 4. 客戶資料庫</button>
        <button :class="{ active: subTab === 'orchid' }" @click="subTab = 'orchid'">🌸 5. 蘭花品種</button>
        <button :class="{ active: subTab === 'return' }" @click="subTab = 'return'">🔄 6. 退貨管理</button>
      </nav>

      <div class="manage-content">
        <!-- ================= 模組：訂單與帳務 ================= -->
        <section v-if="subTab === 'order'" class="tab-pane">
          <!-- 編輯提示橫條 -->
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

              <!-- 批發規格 (批發株數已改為株數) -->
              <template v-if="formOrder.cust_type === '批發'">
                <div class="field">
                  <label>株數 (棵)</label>
                  <input v-model.number="formOrder.batch_qty" type="number" min="1" />
                </div>
                <div class="field">
                  <label>每棵單價 (元)</label>
                  <input v-model.number="formOrder.batch_price" type="number" min="0" />
                </div>
              </template>
              <!-- 零售 / 花店規格 -->
              <template v-else>
                <div class="field">
                  <label>株數 (棵)</label>
                  <input v-model.number="formOrder.stalks" type="number" min="1" />
                </div>
                <div class="field">
                  <label>使用盆器</label>
                  <select v-model="formOrder.pot">
                    <option value="桌上盆 (100)">桌上盆 (100)</option>
                    <option value="落地盆-喪 (100)">落地盆-喪 (100)</option>
                    <option value="落地盆-喜 (200)">落地盆-喜 (200)</option>
                    <option value="羅馬盆 (280)">羅馬盆 (280)</option>
                    <option value="無盆">無盆</option>
                  </select>
                </div>
              </template>

              <div class="field">
                <label>預估成本 (元)</label>
                <input v-model.number="formOrder.cost" type="number" min="0" />
              </div>
              <div class="field">
                <label>訂單總售價 (元)</label>
                <input 
                  v-if="formOrder.cust_type === '批發'" 
                  :value="formOrder.batch_qty * formOrder.batch_price" 
                  type="text" 
                  disabled 
                />
                <input 
                  v-else 
                  v-model.number="formOrder.price" 
                  type="number" 
                  min="0" 
                />
              </div>

              <div class="field">
                <label>賀卡製作狀態</label>
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
                    <th>週期</th>
                    <th>電話</th>
                    <th>品項規格</th>
                    <th>盆器</th>
                    <th>售價</th>
                    <th>賀卡</th>
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
                    <td><span class="badge badge-purple">{{ ord.billing_cycle || '每單結' }}</span></td>
                    <td>{{ ord.phone }}</td>
                    <td>{{ ord.spec }}</td>
                    <td>{{ ord.pot }}</td>
                    <td class="text-blue"><b>${{ ord.price }}</b></td>
                    <!-- 賀卡狀態 -->
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
                    <!-- 簽收單狀態 -->
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
                    <!-- 出貨狀態 -->
                    <td>
                      <select v-model="ord.shipped_status" @change="updateOrderField(ord, 'shipped_status', ord.shipped_status)">
                        <option value="未出貨">未出貨</option>
                        <option value="已出貨">已出貨</option>
                      </select>
                    </td>
                    <!-- 收款狀態 -->
                    <td>
                      <select v-model="ord.payment_status" @change="updateOrderField(ord, 'payment_status', ord.payment_status)">
                        <option value="未結">未結</option>
                        <option value="已結">已結</option>
                      </select>
                    </td>
                    <td class="action-cell">
                      <button class="mini-btn edit-btn" @click="startEditOrder(ord)" title="修改此訂單">✏️</button>
                      <button class="mini-btn print-btn" @click="fillReceiptFromOrder(ord)" title="直接依單號帶入 A5 簽收單">🖨️ 轉簽收單</button>
                      <button class="mini-btn del-btn" @click="deleteItem('orders', ord.id, loadOrders)" title="刪除">🗑️</button>
                    </td>
                  </tr>
                  <tr v-if="orderList.length === 0"><td colspan="12" class="text-center">尚無訂單資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- ================= 模組：客戶未結帳款對帳專區 ================= -->
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

            <!-- 統計看版 -->
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

            <!-- 操作列 -->
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
                    <th>品項與規格細節</th>
                    <th>盆器</th>
                    <th>金額</th>
                    <th>賀卡</th>
                    <th>簽收單</th>
                    <th>收款狀態</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="ord in statementOrders" :key="ord.id">
                    <td><b>{{ ord.id }}</b></td>
                    <td>{{ ord.order_date }}</td>
                    <td>{{ ord.customer }}</td>
                    <td>{{ ord.spec }}</td>
                    <td>{{ ord.pot }}</td>
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
                    </td>
                  </tr>
                  <tr v-if="statementOrders.length === 0">
                    <td colspan="10" class="text-center py-4 text-gray">符合條件的訂單為 0 筆</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- ================= 模組：進貨與庫存 ================= -->
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
                    <option value="落地盆-喪 (100)">落地盆-喪 (100)</option>
                    <option value="落地盆-喜 (200)">落地盆-喜 (200)</option>
                    <option value="羅馬盆 (280)">羅馬盆 (280)</option>
                  </select>
                </div>
              </template>

              <div class="field">
                <label>進貨數量</label>
                <input v-model.number="formInv.qty" type="number" min="1" />
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
                    <th>編號</th>
                    <th>類別</th>
                    <th>品項名稱</th>
                    <th>規格</th>
                    <th>進貨數</th>
                    <th>總成本</th>
                    <th>供應商</th>
                    <th>日期</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="inv in inventoryList" :key="inv.id">
                    <td>{{ inv.id }}</td>
                    <td><span class="badge">{{ inv.category }}</span></td>
                    <td><b>{{ inv.item_name }}</b></td>
                    <td>{{ inv.spec }}</td>
                    <td>{{ inv.qty }}</td>
                    <td>${{ inv.cost }}</td>
                    <td>{{ inv.supplier }}</td>
                    <td>{{ inv.date }}</td>
                    <td class="action-cell">
                      <button class="mini-btn edit-btn" @click="startEditInv(inv)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('inventory', inv.id, loadInventory)" title="刪除">🗑️</button>
                    </td>
                  </tr>
                  <tr v-if="inventoryList.length === 0"><td colspan="9" class="text-center">尚無進貨資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- ================= 模組：客戶資料庫 ================= -->
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
                <label>預設送達地址 / 備註</label>
                <input v-model="formCust.line_note" type="text" placeholder="常用送達地址 (開單時自動帶入簽收單)" />
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
                    <th>編號</th>
                    <th>名稱</th>
                    <th>類別</th>
                    <th>結帳週期</th>
                    <th>電話</th>
                    <th>地址 / 備註</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="c in customers" :key="c.id">
                    <td>{{ c.id }}</td>
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
                  <tr v-if="customers.length === 0"><td colspan="7" class="text-center">尚無客戶資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- ================= 模組：蘭花品種庫 ================= -->
        <section v-if="subTab === 'orchid'" class="tab-pane">
          <div v-if="editingOrchidId" class="edit-banner">
            <span>✏️ 目前正在編輯品種：<b>{{ editingOrchidId }}</b></span>
            <button class="cancel-edit-btn" @click="cancelEditOrchid">✕ 取消修改</button>
          </div>

          <div class="card-box" id="orchid-form-box">
            <h3>{{ editingOrchidId ? '✏️ 修改蘭花品種' : '🌸 新增蘭花品種資料' }}</h3>
            <div class="form-grid">
              <div class="field">
                <label>品種名稱 (例: 大辣椒、V3)</label>
                <input v-model="formOrchid.name" type="text" placeholder="輸入品種名稱" />
              </div>
              <div class="field">
                <label>特色說明</label>
                <input v-model="formOrchid.note" type="text" placeholder="品種特色說明" />
              </div>
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
                    <th>編號</th>
                    <th>品種名稱</th>
                    <th>特色說明</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in orchids" :key="item.id">
                    <td>{{ item.id }}</td>
                    <td><b>{{ item.name }}</b></td>
                    <td>{{ item.note }}</td>
                    <td class="action-cell">
                      <button class="mini-btn edit-btn" @click="startEditOrchid(item)" title="修改">✏️</button>
                      <button class="mini-btn del-btn" @click="deleteItem('orchids', item.id, loadOrchids)" title="刪除">🗑️</button>
                    </td>
                  </tr>
                  <tr v-if="orchids.length === 0"><td colspan="4" class="text-center">尚無品種資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- ================= 模組：退貨管理區 ================= -->
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
                  <option value="退給花農">1. 我們向花農退貨 (退給供應商)</option>
                  <option value="客戶退回">2. 批發商向我們退貨 (客戶退回)</option>
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
                <input v-model.number="formRet.qty" type="number" min="1" />
              </div>
              <div class="field">
                <label>每棵單價 (元)</label>
                <input v-model.number="formRet.unit_price" type="number" min="0" />
              </div>
              <div class="field">
                <label>總損益金額</label>
                <input :value="formRet.qty * formRet.unit_price" type="text" disabled />
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
                    <th>編號</th>
                    <th>類型</th>
                    <th>對象</th>
                    <th>品項</th>
                    <th>株數</th>
                    <th>總額</th>
                    <th>原因</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="ret in returnList" :key="ret.id">
                    <td>{{ ret.id }}</td>
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
                  <tr v-if="returnList.length === 0"><td colspan="8" class="text-center">尚無退貨資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>
      </div>
    </div>

    <!-- ================= 模式 2：花卡 / 輓聯編輯器 (A4 比例) ================= -->
    <div v-else-if="currentTab === 'couplet'" class="app-container no-print">
      <div class="control-panel">
        <h2>⚙️ 卡片與題詞設定</h2>

        <div class="form-group">
          <label>版面模式 (標準 A4)：</label>
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

        <!-- 喪禮設定 -->
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

        <!-- 慶賀設定 -->
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

        <!-- 下款 6 格設定 -->
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
        <button type="button" class="print-action-btn mt-2" @click="printCouplet">🖨️ 列印 A4 花卡 / 輓聯</button>
      </div>

      <!-- 右側畫布視窗 -->
      <div class="canvas-viewport" ref="viewportRef">
        <div class="zoom-toolbar">
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.25, +(zoomLevel - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
          <button type="button" class="fit-btn" @click="autoFitZoom">📱 配合手機螢幕</button>
        </div>

        <div 
          class="card-scaler-container" 
          :style="{
            width: (isVertical ? 560 : 792) * zoomLevel + 'px',
            height: (isVertical ? 792 : 560) * zoomLevel + 'px'
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
            <template v-if="isVertical">
              <div class="crop-mark top-left"></div>
              <div class="crop-mark top-right"></div>
              <div class="crop-mark bottom-left"></div>
              <div class="crop-mark bottom-right"></div>
            </template>

            <div 
              v-if="upperText.trim()"
              class="text-box upper-box"
              :style="getStyle('upper')"
              @pointerdown="startMove($event, 'upper')"
            >
              <span>{{ upperText }}</span>
              <div class="scale-handle" @pointerdown.stop="startResize($event, 'upper')">⤡</div>
            </div>

            <div 
              v-if="middleText.trim()"
              class="text-box middle-box"
              :style="getStyle('middle')"
              @pointerdown="startMove($event, 'middle')"
            >
              <span>{{ middleText }}</span>
              <div class="scale-handle" @pointerdown.stop="startResize($event, 'middle')">⤡</div>
            </div>

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

    <!-- ================= 模式 3：A5 橫式簽收單 (完全免打字・單號全自動帶入) ================= -->
    <div v-else-if="currentTab === 'receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>📋 橫式 A5 簽收單（依單號自動生成）</h2>

        <!-- 依單號自動帶入 -->
        <div class="panel-section highlight-panel">
          <label class="section-title">選擇要列印的訂單編號：</label>
          <select v-model="selectedOrderId" class="full-input bold-select">
            <option value="">-- 請下拉選擇訂單 (即時自動帶入) --</option>
            <option v-for="ord in orderList" :key="ord.id" :value="ord.id">
              【{{ ord.id }}】{{ ord.customer }} - {{ ord.spec }} [{{ ord.receipt_status || '未列印' }}]
            </option>
          </select>
        </div>

        <!-- 花店店名切換 -->
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

        <!-- 送達地址微調 -->
        <div class="panel-section">
          <label class="section-title">送達地址確認 (自動讀取客戶庫)：</label>
          <input 
            type="text" 
            v-model="receiptAddressOverride" 
            class="full-input" 
            placeholder="地址自動帶入，亦可在此臨時微調" 
          />
        </div>

        <!-- 當前訂單狀態展示 -->
        <div v-if="activeReceiptOrder" class="panel-section order-quick-info">
          <div><b>當前單號：</b>{{ activeReceiptOrder.id }}</div>
          <div><b>訂購客戶：</b>{{ activeReceiptOrder.customer }} ({{ activeReceiptOrder.phone || '無電話' }})</div>
          <div><b>預定送達：</b>{{ activeReceiptOrder.expected_date }}</div>
          <div>
            <b>簽收單狀態：</b>
            <span :class="activeReceiptOrder.receipt_status === '已列印' ? 'badge badge-green' : 'badge badge-orange'">
              {{ activeReceiptOrder.receipt_status || '未列印' }}
            </span>
          </div>
        </div>

        <hr />
        <button 
          type="button" 
          class="print-action-btn" 
          :disabled="!activeReceiptOrder"
          @click="printReceiptAndMarkDone"
        >
          🖨️ 一鍵列印 A5 簽收單 (自動標記已列印)
        </button>
      </div>

      <!-- 右側預覽區 -->
      <div class="receipt-preview-area" ref="receiptViewportRef">
        <div class="zoom-toolbar no-print">
          <button type="button" class="zoom-btn" @click="receiptZoom = Math.max(0.3, +(receiptZoom - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(receiptZoom * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="receiptZoom = Math.min(1.1, +(receiptZoom + 0.05).toFixed(2))">＋</button>
          <button type="button" class="fit-btn" @click="autoFitReceipt">📱 適配螢幕</button>
        </div>

        <div 
          v-if="activeReceiptOrder"
          class="receipt-scaler-container"
          :style="{
            width: (794 * receiptZoom) + 'px',
            height: (560 * receiptZoom) + 'px'
          }"
        >
          <div 
            class="a5-landscape-sheet"
            :style="{
              transform: `scale(${receiptZoom})`,
              transformOrigin: 'top left'
            }"
          >
            <div class="sheet-header">
              <div class="shop-name-title">{{ displayShopName }}</div>
              <div class="sheet-main-title">銷貨 / 出貨簽收單</div>
              <div class="header-meta">
                <div><b>訂單編號：</b>{{ activeReceiptOrder.id }}</div>
                <div><b>送達日期：</b>{{ activeReceiptOrder.expected_date }} 送達</div>
              </div>
            </div>

            <table class="receipt-table">
              <tbody>
                <tr>
                  <td class="lbl">收件單位/人</td>
                  <td class="val val-bold">{{ activeReceiptOrder.customer }} ({{ activeReceiptOrder.phone || '無電話' }})</td>
                  <td class="lbl">送達地址</td>
                  <td class="val">{{ currentResolvedAddress || '同訂購人地址 / 門市取貨' }}</td>
                </tr>
                <tr>
                  <td class="lbl">花禮品項</td>
                  <td class="val val-highlight" colspan="3">
                    {{ activeReceiptOrder.spec }} 【盆器：{{ activeReceiptOrder.pot }}】
                  </td>
                </tr>
                <tr>
                  <td class="lbl">致贈/賀詞</td>
                  <td class="val" colspan="3">敬領 誌慶 / {{ displayShopName }} 敬製</td>
                </tr>
                <tr>
                  <td class="lbl">備註說明</td>
                  <td class="val" colspan="3">花禮已專車安全送達指定地點，敬請點交簽名確認。感謝您的惠顧！</td>
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
                <div class="sign-box-area">（請於此處簽名或蓋章）</div>
              </div>
            </div>
          </div>
        </div>

        <div v-else class="no-order-selected no-print">
          <div class="empty-icon">📄</div>
          <div class="empty-text">請由左上方選擇「訂單編號」，或至訂單列表點擊「🖨️ 轉簽收單」</div>
          <div class="empty-sub">系統將自動抓取單號、客戶、電話、地址與品項，100% 免手動輸入！</div>
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

const formOrchid = ref({ name: '', note: '標準優良品種' })
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
  date: new Date().toISOString().split('T')[0],
  reason: '運送碰撞 / 開花不良'
})
const formOrder = ref({
  cust_type: '批發',
  customer: '',
  billing_cycle: '每單結',
  phone: '0912-345678',
  orchid_name: '',
  batch_qty: 50,
  batch_price: 250,
  stalks: 10,
  pot: '桌上盆 (100)',
  cost: 600,
  price: 1500,
  card_status: '未製作',
  receipt_status: '未列印',
  shipped_status: '未出貨',
  payment_status: '未結',
  order_date: new Date().toISOString().split('T')[0],
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0]
})

const flowerInventory = computed(() => inventoryList.value.filter(i => i.category === '蘭花'))

// ==========================================
// 1. 訂單模組 (含簽收單狀態)
// ==========================================
const loadOrders = async () => {
  const { data } = await supabase.from('orders').select('*').order('created_at', { ascending: false })
  if (data) orderList.value = data
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
  formOrder.value = {
    cust_type: ord.cust_type || '批發',
    customer: ord.customer || '',
    billing_cycle: ord.billing_cycle || '每單結',
    phone: ord.phone || '',
    orchid_name: ord.spec ? ord.spec.split('|')[0].trim() : '',
    batch_qty: 50,
    batch_price: 250,
    stalks: 10,
    pot: ord.pot || '桌上盆 (100)',
    cost: Number(ord.cost) || 0,
    price: Number(ord.price) || 0,
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
    batch_qty: 50,
    batch_price: 250,
    stalks: 10,
    pot: '桌上盆 (100)',
    cost: 600,
    price: 1500,
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
  const isWholesale = formOrder.value.cust_type === '批發'
  // 去除「批發」字樣，統一顯示為株數與單價
  const specStr = isWholesale 
    ? `${formOrder.value.orchid_name || '特選蘭花'} | ${formOrder.value.batch_qty}棵 (單價${formOrder.value.batch_price}元)`
    : `${formOrder.value.orchid_name || '特選蘭花'} | ${formOrder.value.stalks}棵`
  const finalPrice = isWholesale ? (formOrder.value.batch_qty * formOrder.value.batch_price) : formOrder.value.price

  const payload = {
    cust_type: formOrder.value.cust_type,
    customer: formOrder.value.customer,
    billing_cycle: formOrder.value.billing_cycle,
    phone: formOrder.value.phone,
    spec: specStr,
    pot: isWholesale ? '批發免盆' : formOrder.value.pot,
    cost: formOrder.value.cost,
    price: finalPrice,
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
    const newId = 'OR' + Date.now().toString().slice(-5)
    const { error } = await supabase.from('orders').insert([{ id: newId, ...payload }])
    if (!error) {
      alert('訂單建立成功！')
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
      const lastDay = new Date(now.getFullYear(), now.getMonth(), 0, 23, 59, 59)
      return orderDate >= firstDay && orderDate <= lastDay
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
// 3. A5 橫式簽收單 (依單號純自動生成)
// ==========================================
const shopNameMode = ref('default')
const customShopName = ref('')
const displayShopName = computed(() => shopNameMode.value === 'default' ? '宸豐蘭藝' : (customShopName.value || '宸豐蘭藝'))

const receiptViewportRef = ref(null)
const receiptZoom = ref(1)
const selectedOrderId = ref('')
const receiptAddressOverride = ref('')

const activeReceiptOrder = computed(() => {
  return orderList.value.find(o => o.id === selectedOrderId.value) || null
})

const currentResolvedAddress = computed(() => {
  if (receiptAddressOverride.value) return receiptAddressOverride.value
  if (!activeReceiptOrder.value) return ''
  const matchedCust = customers.value.find(c => c.name === activeReceiptOrder.value.customer)
  return matchedCust ? (matchedCust.line_note || '') : ''
})

watch(selectedOrderId, (newId) => {
  if (!newId) {
    receiptAddressOverride.value = ''
    return
  }
  const ord = orderList.value.find(o => o.id === newId)
  if (ord) {
    const cust = customers.value.find(c => c.name === ord.customer)
    receiptAddressOverride.value = cust?.line_note || ''
  }
})

const autoFitReceipt = () => {
  if (!receiptViewportRef.value) return
  const availWidth = Math.max(receiptViewportRef.value.clientWidth - 28, 280)
  receiptZoom.value = Math.min(Math.max(+(availWidth / 794).toFixed(2), 0.35), 1.0)
}

const fillReceiptFromOrder = (ord) => {
  selectedOrderId.value = ord.id
  currentTab.value = 'receipt'
  nextTick(() => autoFitReceipt())
}

const printReceiptAndMarkDone = async () => {
  if (!activeReceiptOrder.value) return
  await updateOrderField(activeReceiptOrder.value, 'receipt_status', '已列印')
  activeReceiptOrder.value.receipt_status = '已列印'
  window.print()
}

// ==========================================
// 4. 進貨與庫存
// ==========================================
const loadInventory = async () => {
  const { data } = await supabase.from('inventory').select('*').order('created_at', { ascending: false })
  if (data) inventoryList.value = data
}

const onPotTypeChange = () => {
  const potCostMap = { '桌上盆 (100)': 100, '落地盆-喪 (100)': 100, '落地盆-喜 (200)': 200, '羅馬盆 (280)': 280 }
  formInv.value.cost = (potCostMap[formInv.value.pot_type] || 100) * formInv.value.qty
}

const startEditInv = (inv) => {
  editingInvId.value = inv.id
  formInv.value = {
    category: inv.category,
    item_name: inv.item_name,
    spec_spike: '單梗',
    spec_color: '紅',
    spec_size: '大',
    spec_height: '中',
    pot_type: '桌上盆 (100)',
    qty: inv.qty,
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
    const newId = (isFlower ? 'FL' : 'POT') + Date.now().toString().slice(-5)
    const { error } = await supabase.from('inventory').insert([{ id: newId, ...payload }])
    if (!error) {
      alert('進貨紀錄新增成功！')
      loadInventory()
    } else {
      alert('新增失敗：' + error.message)
    }
  }
}

// ==========================================
// 5. 客戶資料庫
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
    const newId = 'CUST' + Date.now().toString().slice(-5)
    const { error } = await supabase.from('customers').insert([{ id: newId, ...payload }])
    if (!error) {
      alert('客戶建立成功！')
      cancelEditCust()
      loadCustomers()
    } else {
      alert('建立失敗：' + error.message)
    }
  }
}

// ==========================================
// 6. 蘭花品種與退貨模組
// ==========================================
const loadOrchids = async () => {
  const { data } = await supabase.from('orchids').select('*').order('created_at', { ascending: false })
  if (data) orchids.value = data
}
const startEditOrchid = (item) => {
  editingOrchidId.value = item.id
  formOrchid.value = { name: item.name, note: item.note || '' }
  nextTick(() => {
    document.getElementById('orchid-form-box')?.scrollIntoView({ behavior: 'smooth' })
  })
}
const cancelEditOrchid = () => {
  editingOrchidId.value = null
  formOrchid.value = { name: '', note: '標準優良品種' }
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
    }
  } else {
    const newId = 'DB' + Date.now().toString().slice(-5)
    const { error } = await supabase.from('orchids').insert([{ id: newId, ...payload }])
    if (!error) {
      alert('品種新增成功！')
      cancelEditOrchid()
      loadOrchids()
    }
  }
}

const loadReturns = async () => {
  const { data } = await supabase.from('returns').select('*').order('created_at', { ascending: false })
  if (data) returnList.value = data
}
const startEditRet = (ret) => {
  editingRetId.value = ret.id
  formRet.value = {
    return_type: ret.return_type,
    party_name: ret.party_name,
    target_item: ret.target_item,
    qty: ret.qty,
    unit_price: ret.unit_price,
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
    }
  } else {
    const newId = 'RET' + Date.now().toString().slice(-5)
    const { error } = await supabase.from('returns').insert([{ id: newId, ...payload }])
    if (!error) {
      alert('退貨紀錄儲存成功！')
      cancelEditRet()
      loadReturns()
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
    '盆器類型': o.pot,
    '預估成本': o.cost,
    '訂單售價': o.price,
    '利潤': o.price - o.cost,
    '賀卡狀態': o.card_status || '未製作',
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
// 7. 花卡 / 輓聯編輯器
// ==========================================
const isVertical = ref(true)
const cardCategory = ref('funeral')
const zoomLevel = ref(1)
const viewportRef = ref(null)

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

const celebrationType = ref('opening')
const celebPrefix = ref('恭祝')
const celebTarget = ref('鴻運實業有限公司')
const celebPhrases = {
  opening: ['開幕誌慶', '開張大吉', '鴻圖大展', '駿業宏開', '生意興隆', '財源廣進', '客似雲來'],
  moving: ['喬遷之喜', '里仁為美', '金玉滿堂'],
  temple: ['聖誕千秋', '神威顯赫']
}
const currentCelebPhrases = computed(() => celebPhrases[celebrationType.value] || [])

const upperText = ref('陳媽李老夫人 仙逝')
const middleText = ref('母儀千古')
const suffixText = ref('敬輓')

const defaultVertical = {
  upper:    { x: 440, y: 65,  size: 30 },
  middle:   { x: 230, y: 140, size: 66 },
  bottom_0: { x: 105, y: 350, size: 22 },
  bottom_1: { x: 105, y: 460, size: 26 },
  bottom_2: { x: 65,  y: 350, size: 22 },
  bottom_3: { x: 65,  y: 460, size: 24 },
  bottom_4: { x: 25,  y: 350, size: 22 },
  bottom_5: { x: 25,  y: 460, size: 22 },
  suffix:   { x: 105, y: 620, size: 24 }
}
const defaultHorizontal = {
  upper:    { x: 140, y: 65,  size: 28 },
  middle:   { x: 160, y: 180, size: 58 },
  bottom_0: { x: 240, y: 280, size: 20 },
  bottom_1: { x: 240, y: 320, size: 22 },
  bottom_2: { x: 240, y: 360, size: 20 },
  bottom_3: { x: 240, y: 400, size: 20 },
  bottom_4: { x: 240, y: 440, size: 20 },
  bottom_5: { x: 240, y: 480, size: 20 },
  suffix:   { x: 440, y: 340, size: 24 }
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
  const item = layout.value[key] || { x: 50, y: 50, size: 22 }
  return { left: `${item.x}px`, top: `${item.y}px`, fontSize: `${item.size}px` }
}
const autoFitZoom = () => {
  if (!viewportRef.value) return
  const availableWidth = Math.max(viewportRef.value.clientWidth - 28, 280)
  const cardWidth = isVertical.value ? 560 : 792
  zoomLevel.value = Math.min(Math.max(+(availableWidth / cardWidth).toFixed(2), 0.35), 1.0)
}

let activeKey = null
let currentAction = null
let startX = 0, startY = 0, originX = 0, originY = 0, originSize = 24

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
    layout.value[activeKey].size = Math.max(14, Math.min(120, Math.round(originSize + (dx + dy) / 3)))
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
const printCouplet = () => window.print()

onMounted(() => {
  autoFitZoom()
  autoFitReceipt()
  window.addEventListener('resize', () => {
    autoFitZoom()
    autoFitReceipt()
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

.btn-action-row { display: flex; gap: 8px; }
.primary-btn { background: #2563eb; color: white; border: none; padding: 8px 18px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.secondary-btn { background: #94a3b8; color: white; border: none; padding: 8px 16px; border-radius: 6px; font-weight: bold; cursor: pointer; }
.excel-btn { background: #059669; color: white; border: none; padding: 7px 14px; border-radius: 6px; font-weight: bold; cursor: pointer; white-space: nowrap; }

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

/* 狀態下拉選單與徽章 */
.select-status { font-weight: bold; padding: 4px 6px; border-radius: 4px; }
.select-status.orange { background: #fffbeb; color: #b45309; border: 1px solid #fde68a; }
.select-status.green { background: #f0fdf4; color: #15803d; border: 1px solid #bbf7d0; }
.select-status.gray { background: #f1f5f9; color: #475569; border: 1px solid #cbd5e1; }

.table-header-action { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.table-responsive { overflow-x: auto; -webkit-overflow-scrolling: touch; }
.data-table { width: 100%; border-collapse: collapse; font-size: 13px; text-align: left; min-width: 700px; }
.data-table th { background: #f8fafc; padding: 8px 10px; border-bottom: 2px solid #e2e8f0; color: #475569; white-space: nowrap; }
.data-table td { padding: 8px 10px; border-bottom: 1px solid #e2e8f0; }
.action-cell { white-space: nowrap; }

.badge { background: #e0f2fe; color: #0369a1; padding: 2px 6px; border-radius: 4px; font-size: 11px; }
.badge-purple { background: #f3e8ff; color: #7e22ce; }
.badge-red { background: #fee2e2; color: #dc2626; font-weight: bold; }
.badge-orange { background: #ffedd5; color: #c2410c; font-weight: bold; }
.badge-green { background: #dcfce7; color: #16a34a; font-weight: bold; }
.status-tag { font-size: 12px; font-weight: bold; }

.mini-btn { border: none; padding: 4px 8px; border-radius: 4px; cursor: pointer; margin-right: 4px; }
.edit-btn { background: #3b82f6; color: white; }
.del-btn { background: #ef4444; color: white; }
.print-btn { background: #f59e0b; color: white; font-weight: bold; font-size: 12px; }

.text-red { color: #dc2626; }
.text-blue { color: #2563eb; }
.text-green { color: #16a34a; }
.text-center { text-align: center; }
.text-gray { color: #94a3b8; }
.py-4 { padding: 16px 0; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 16px; }

/* 簽收單控制面板 */
.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.control-panel {
  width: 380px; background: white; padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.06); overflow-y: auto; flex-shrink: 0;
}
.panel-section { background: #f8fafc; border: 1px solid #e2e8f0; padding: 10px; border-radius: 6px; margin-bottom: 10px; }
.highlight-panel { background: #eff6ff; border: 2px solid #3b82f6; }
.bold-select { font-weight: bold; font-size: 14px; border-color: #3b82f6; }
.order-quick-info { font-size: 13px; line-height: 1.8; color: #1e293b; }

.section-title { font-size: 13px; font-weight: bold; margin-bottom: 6px; display: block; }
.form-group { margin-bottom: 10px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 4px; }
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
.canvas-viewport, .receipt-preview-area {
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

/* A4 卡片 */
.card-scaler-container { position: relative; }
.card-board { background: #fff; position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none; touch-action: none; }
.card-board.mode-vertical { width: 560px; height: 792px; font-family: "DFKai-SB", "BiauKai", serif; }
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 6px; }
.card-board.mode-vertical .middle-box { letter-spacing: 14px; font-weight: 900; }
.card-board.mode-horizontal { width: 792px; height: 560px; font-family: "DFKai-SB", "BiauKai", serif; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 4px; }
.card-board.mode-horizontal .middle-box { letter-spacing: 12px; font-weight: 900; }
.card-board.style-floral { border: 10px solid #fce7f3; }

.crop-mark { position: absolute; width: 12px; height: 12px; border-color: #94a3b8; border-style: solid; }
.crop-mark.top-left { top: 15px; left: 15px; border-width: 1px 0 0 1px; }
.crop-mark.top-right { top: 15px; right: 15px; border-width: 1px 1px 0 0; }
.crop-mark.bottom-left { bottom: 15px; left: 15px; border-width: 0 0 1px 1px; }
.crop-mark.bottom-right { bottom: 15px; right: 15px; border-width: 0 1px 1px 0; }

.text-box { position: absolute; cursor: move; font-weight: bold; padding: 3px 5px; white-space: nowrap; line-height: 1.2; }
.text-box:hover { outline: 1px dashed #2563eb; background: rgba(37, 99, 235, 0.04); }
.scale-handle {
  position: absolute; right: -7px; bottom: -7px; width: 17px; height: 17px;
  background: #2563eb; color: white; border-radius: 3px; font-size: 11px;
  display: flex; justify-content: center; align-items: center; cursor: nwse-resize;
}

/* A5 橫式簽收單 */
.receipt-scaler-container { position: relative; }
.a5-landscape-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 38px 45px;
  box-sizing: border-box; display: flex; flex-direction: column; justify-content: space-between;
  writing-mode: horizontal-tb; direction: ltr; font-family: "DFKai-SB", "BiauKai", "Microsoft JhengHei", sans-serif;
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
.sign-box-area { flex: 1; display: flex; justify-content: center; align-items: center; color: #94a3b8; font-size: 13px; min-height: 48px; }
.print-action-btn {
  width: 100%; padding: 12px; background: #16a34a; color: white;
  border: none; border-radius: 6px; font-size: 15px; font-weight: bold; cursor: pointer;
}
.print-action-btn:disabled { background: #cbd5e1; cursor: not-allowed; }

.no-order-selected {
  margin-top: 60px; text-align: center; background: white;
  padding: 40px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.06); max-width: 480px;
}
.empty-icon { font-size: 48px; margin-bottom: 12px; }
.empty-text { font-size: 16px; font-weight: bold; color: #1e293b; margin-bottom: 8px; }
.empty-sub { font-size: 13px; color: #64748b; }

@media (max-width: 768px) {
  .app-container, .receipt-container { flex-direction: column; overflow-y: auto; }
  .control-panel { width: 100%; max-height: 46vh; }
  .form-grid { grid-template-columns: 1fr; }
  .canvas-viewport, .receipt-preview-area { padding: 12px 6px; }
}

@media print {
  @page { size: auto; margin: 0; }
  body, html, .main-wrapper { margin: 0 !important; padding: 0 !important; background: white !important; }
  .no-print { display: none !important; }
  .canvas-viewport, .receipt-preview-area { padding: 0 !important; background: white !important; overflow: visible !important; }
  .card-scaler-container, .receipt-scaler-container { width: auto !important; height: auto !important; }
  .card-board { position: relative !important; transform: none !important; box-shadow: none !important; }
  .a5-landscape-sheet { position: relative !important; transform: none !important; box-shadow: none !important; width: 210mm !important; height: 148mm !important; }
}
</style>