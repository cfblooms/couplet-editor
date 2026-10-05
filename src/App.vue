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
            💼 庫存・客戶・訂單
          </button>
          <button 
            type="button" 
            :class="{ active: currentTab === 'couplet' }" 
            @click="currentTab = 'couplet'"
          >
            🎴 花卡 / 輓聯 (A3/A4/A5)
          </button>
          <button 
            type="button" 
            :class="{ active: currentTab === 'receipt' }" 
            @click="currentTab = 'receipt'"
          >
            📄 A5 簽收單
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

      <!-- 手機／平板專用無網址送印與傳送彈窗 -->
      <div v-if="shareModalImg" class="image-modal-overlay no-print" @click="closeShareModal">
        <div class="image-modal-content share-preview-modal" @click.stop>
          <div class="image-modal-header">
            <span>💬 {{ shareModalTitle }}</span>
            <button class="close-modal-btn" @click="closeShareModal">✕</button>
          </div>
          <div class="share-modal-body">
            <div class="share-img-scroll-container">
              <img :src="shareModalImg" class="share-preview-img-contained" alt="預覽圖" />
            </div>
            
            <div class="share-btn-action-group">
              <button type="button" class="mobile-print-btn" @click="triggerImagePrint">
                🖨️ 手機直接列印 (無網址・無時間・一張紙)
              </button>
              <button v-if="canNativeShare" type="button" class="mobile-share-btn" @click="triggerNativeShare">
                📲 一鍵傳送至 LINE 給客人
              </button>
              <a :href="shareModalImg" :download="shareModalFilename" class="mobile-dl-btn">
                💾 下載圖檔至相簿
              </a>
            </div>

            <div class="share-tips-row">
              <span>💡 <b>特製色卡列印說明：</b></span>
              <span>• 點擊「手機直接列印」，系統以相片方式直接送印，<b>底部絕不會出現任何網址與時間！</b></span>
              <span>• 送印時背景已自動去除為無色透明，印表機只會印出文字！</span>
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

              <!-- 多組花禮規格設定區塊 -->
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

            <!-- 訂單清單總覽 -->
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

          <!-- 模組 2：客戶未結對帳專區 -->
          <section v-if="subTab === 'statement'" class="tab-pane">
            <div class="card-box">
              <h3>📊 客戶未結帳款彙整與對帳</h3>
              <p>對帳專區運作中。</p>
            </div>
          </section>

          <!-- 模組 3：進貨與庫存 -->
          <section v-if="subTab === 'inventory'" class="tab-pane">
            <div class="card-box">
              <h3>📦 庫存管理</h3>
              <p>庫存系統運作中。</p>
            </div>
          </section>
        </div>
      </div>

      <!-- ================= 模式 2：花卡 / 輓聯編輯器 (A3/A4/A5，Supabase 3底圖連線) ================= -->
      <div v-else-if="currentTab === 'couplet'" class="app-container couplet-screen-wrapper">
        <div class="control-panel no-print">
          <h2>⚙️ 卡片與題詞設定</h2>

          <!-- 紙張尺寸：A3 / A4 / A5 -->
          <div class="panel-section">
            <label class="section-title">📄 紙張尺寸選擇：</label>
            <div class="btn-group">
              <button type="button" :class="{ active: cardPaperSize === 'A3' }" @click="switchPaperSize('A3')">A3 (超大)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A4' }" @click="switchPaperSize('A4')">A4 (標準大)</button>
              <button type="button" :class="{ active: cardPaperSize === 'A5' }" @click="switchPaperSize('A5')">A5 (小卡片)</button>
            </div>
          </div>

          <!-- 🌸 Supabase Storage 3 款專用紙張底色切換 -->
          <div class="panel-section highlight-panel">
            <label class="section-title">🌸 實體卡片樣式底圖 (客人預覽用)：</label>
            <select v-model="cardBgType" class="full-input bold-select">
              <option value="red">🌺 喜慶紅卡底圖</option>
              <option value="pink">🌸 優雅粉卡底圖</option>
              <option value="white">📄 質感白卡底圖</option>
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
              💡 <b>安心提示</b>：這裡選底圖是為了<b>直接傳給客人確認</b>；當您按列印時，<b>系統會自動變為無色透明背景</b>，直接套印在色紙上！
            </div>
          </div>

          <div class="panel-section">
            <div class="inline-font-weight-row">
              <div class="inline-item-flex">
                <label class="mini-field-lbl">字體選擇：</label>
                <!-- 🌟 標楷體優先，全平台手機、平板與電腦均支援楷書筆觸 -->
                <select v-model="cardFontFamily" class="full-input compact-inline-select font-bold">
                  <option value="kai">標準標楷體 / Word正楷 (全平台書法標準)</option>
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
          
          <!-- 列印：電腦直接送印；手機/平板透明相片輸出無網址 -->
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
            <!-- 🌟 花卡主體 (螢幕顯示 Supabase 雲端底圖，送印時 CSS 強制透明) -->
            <div 
              id="card-print-target" 
              class="card-board standard-kai-font" 
              :class="isVertical ? 'mode-vertical' : 'mode-horizontal'"
              :style="{
                width: currentCardDimensions.w + 'px',
                height: currentCardDimensions.h + 'px',
                transform: `scale(${zoomLevel})`,
                transformOrigin: 'top left',
                fontFamily: activeCssFontFamily,
                backgroundImage: activeBackgroundImageStyle
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
// 其餘邏輯已整合於上方的 setup 區塊中
</script>

<style scoped>
/* 🌟 全系統基本樣式與行動自適應排版 */
.main-wrapper {
  display: flex; flex-direction: column; height: 100vh; font-size: 13.5px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang TC", "Microsoft JhengHei", sans-serif;
  background-color: #f1f5f9; overflow-x: hidden;
}
.system-root { display: flex; flex-direction: column; flex: 1; overflow: hidden; }

/* 頂端導航 */
.top-nav {
  height: 50px; background-color: #0f172a; color: white; display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px; flex-shrink: 0; overflow-x: auto;
}
.nav-title { font-size: 16px; font-weight: 900; white-space: nowrap; margin-right: 12px; }
.nav-tabs { display: flex; gap: 8px; align-items: center; flex-wrap: nowrap; }
.nav-tabs button {
  background: #334155; color: #e2e8f0; border: none; padding: 7px 12px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 13px; white-space: nowrap;
}
.nav-tabs button.active { background: #2563eb; color: white; }
.logout-nav-btn { background: #ef4444 !important; color: white !important; }

/* 管理子分頁導航 */
.sub-nav {
  display: flex; background: #ffffff; border-bottom: 1px solid #e2e8f0; padding: 7px 16px; gap: 8px; overflow-x: auto; flex-shrink: 0;
}
.sub-nav button {
  background: #f8fafc; border: 1px solid #cbd5e1; padding: 5px 12px; border-radius: 6px; font-weight: bold; font-size: 13px; cursor: pointer; white-space: nowrap;
}
.sub-nav button.active { background: #10b981; color: white; border-color: #10b981; }

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
  display: flex; align-items: center; gap: 5px; background: white; padding: 4px 10px; border-radius: 20px; margin-bottom: 10px; position: sticky; top: 0; z-index: 10;
}
.zoom-btn { width: 24px; height: 24px; border: 1px solid #cbd5e1; background: #f8fafc; border-radius: 50%; cursor: pointer; }
.zoom-text { font-size: 12.5px; font-weight: bold; min-width: 40px; text-align: center; }
.fit-btn { background: #2563eb; color: white; border: none; padding: 4px 8px; border-radius: 12px; font-size: 12px; cursor: pointer; }

.card-scaler-container, .receipt-scaler-container, .farmer-scaler-container { position: relative; flex-shrink: 0; }
.card-board {
  position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none;
  background-size: 100% 100%; background-repeat: no-repeat; background-position: center;
}

.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 8px; }
.card-board.mode-horizontal .text-box { writing-mode: horizontal-tb; letter-spacing: 6px; }
.text-box { position: absolute; cursor: move; white-space: nowrap; color: #0f172a; padding: 2px 4px; }

/* 🌟 正統楷體全平台適配 (電腦微軟楷書，手機平板自動套用標準楷體書法風骨) */
.standard-kai-font, .kai-font-supported {
  font-family: "DFKai-SB", "BiauKai", "標楷體", "TW-Kai", "MOESong-Regular", "Noto Serif TC", "Kaiti", serif !important;
}

/* 簽收單與農民收據 (白底紙張) */
.a5-landscape-sheet, .farmer-receipt-sheet {
  width: 794px; height: 560px; background: #ffffff; padding: 24px 30px; box-sizing: border-box;
  display: flex; flex-direction: column; justify-content: space-between; box-shadow: 0 8px 24px rgba(0,0,0,0.15); position: absolute; top: 0; left: 0;
}
.sheet-header { display: flex; justify-content: space-between; align-items: flex-end; border-bottom: 2.5px solid #1e293b; padding-bottom: 6px; }
.shop-name-title { font-size: 27px; font-weight: 900; }
.sheet-main-title { font-size: 23px; font-weight: bold; color: #dc2626; }
.receipt-table { width: 100%; border-collapse: collapse; margin: 8px 0; }
.receipt-table td { border: 1.5px solid #334155; padding: 7px 10px; }
.receipt-table .lbl { width: 15%; background: #f1f5f9; font-weight: bold; text-align: center; }

/* 農民收據手刻格線 */
.f-main-title { font-size: 24px; font-weight: 900; text-align: center; }
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
.share-img-scroll-container {
  width: 100%; display: flex; justify-content: center; align-items: center;
  background-color: #f1f5f9; border-radius: 8px; padding: 10px; box-sizing: border-box; margin-bottom: 12px;
}
.share-preview-img-contained { max-height: 55vh; max-width: 100%; object-fit: contain; border-radius: 6px; }
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

/* 🌟 行動端特別自適應防跑版優化 (手機/平板) */
@media (max-width: 768px) {
  .couplet-screen-wrapper, .receipt-container, .manage-container {
    flex-direction: column !important;
    overflow-y: auto !important;
    height: auto !important;
  }
  .control-panel {
    width: 100% !important;
    max-height: 52vh !important;
    box-sizing: border-box !important;
  }
  .canvas-viewport, .receipt-preview-area {
    padding: 10px 4px 60px 4px !important;
    width: 100% !important;
    box-sizing: border-box !important;
  }
  .form-grid {
    grid-template-columns: 1fr !important;
  }
  .triple-width-field {
    grid-column: span 1 !important;
  }
}

/* =========================================================================
   🌟 電腦版列印樣式 (背景送印時強制 100% 透明，只印單頁不吐白紙)
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
    font-family: "DFKai-SB", "BiauKai", "標楷體", "TW-Kai", "MOESong-Regular", "Noto Serif TC", "Kaiti", serif !important;
  }
  .no-print, .top-nav, .sub-nav, .control-panel, .zoom-toolbar, .floating-toast, .image-modal-overlay {
    display: none !important;
  }
  /* 電腦送印時背景一律抽空為透明，直接印上紅色或粉色色卡紙 */
  #card-print-target {
    background-color: transparent !important;
    background-image: none !important;
    box-shadow: none !important;
    border: none !important;
    position: absolute !important;
    top: 0 !important; left: 0 !important;
    max-height: 98vh !important;
  }
  #receipt-print-target, #farmer-print-target {
    background-color: transparent !important;
    border: 2px solid #000 !important;
    width: 192mm !important;
    height: 130mm !important;
    margin: 3mm auto 0 auto !important;
  }
}
</style>