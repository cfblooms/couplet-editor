<template>
  <div class="main-wrapper">
    <!-- 頂端主導覽列 -->
    <header class="no-print top-nav">
      <div class="nav-title">🌸 蘭花進銷存與花藝營運系統</div>
      <div class="nav-tabs">
        <button 
          type="button" 
          :class="{ active: currentTab === 'couplet' }" 
          @click="currentTab = 'couplet'"
        >
          🎴 花卡 / 輓聯編輯器
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
          :class="{ active: currentTab === 'manage' }" 
          @click="currentTab = 'manage'"
        >
          💼 蘭花庫存・客戶・訂單管理
        </button>
      </div>
    </header>

    <!-- ================= 模式 1：花卡 / 輓聯編輯器 ================= -->
    <div v-if="currentTab === 'couplet'" class="app-container no-print">
      <div class="control-panel">
        <h2>⚙️ 卡片與題詞設定</h2>

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
            <label class="section-title">中款常用語搭配 (身分與年齡)：</label>
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

        <!-- 下款 5 格自訂設定 -->
        <div class="panel-section">
          <label class="section-title">下款設定（共 5 格自由填寫，空白不顯示）：</label>
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
      </div>

      <!-- 右側畫布視窗 -->
      <div class="canvas-viewport" ref="viewportRef">
        <div class="zoom-toolbar">
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.max(0.3, +(zoomLevel - 0.05).toFixed(2))">－</button>
          <span class="zoom-text">{{ Math.round(zoomLevel * 100) }}%</span>
          <button type="button" class="zoom-btn" @click="zoomLevel = Math.min(1.2, +(zoomLevel + 0.05).toFixed(2))">＋</button>
          <button type="button" class="fit-btn" @click="autoFitZoom">📱 配合螢幕大小</button>
        </div>

        <div 
          class="card-scaler-container" 
          :style="{
            width: (isVertical ? 560 : 720) * zoomLevel + 'px',
            height: (isVertical ? 840 : 490) * zoomLevel + 'px'
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

    <!-- ================= 模式 2：A5 簽收單 ================= -->
    <div v-else-if="currentTab === 'receipt'" class="receipt-container">
      <div class="control-panel no-print">
        <h2>📋 簽收單內容設定</h2>
        <div class="form-group">
          <label>花店店名：</label>
          <input type="text" v-model="receipt.shopName" />
        </div>
        <div class="form-group">
          <label>送達日期：</label>
          <input type="text" v-model="receipt.deliveryDate" />
        </div>
        <div class="form-group">
          <label>送達地址：</label>
          <input type="text" v-model="receipt.address" />
        </div>
        <div class="form-group">
          <label>收件單位 / 聯絡人 / 電話：</label>
          <textarea v-model="receipt.recipient" rows="2"></textarea>
        </div>
        <div class="form-group">
          <label>致贈 / 敬領單位：</label>
          <textarea v-model="receipt.giver" rows="2"></textarea>
        </div>
        <div class="form-group">
          <label>花禮品項內容：</label>
          <input type="text" v-model="receipt.item" />
        </div>
        <hr />
        <button type="button" class="print-action-btn" @click="printReceipt">🖨️ 一鍵列印 A5 簽收單</button>
      </div>

      <div class="receipt-preview-area">
        <div class="a5-receipt-sheet">
          <div class="sheet-column header-col">
            <div class="shop-title">{{ receipt.shopName }}</div>
            <div class="sheet-badge">簽收單</div>
          </div>
          <div class="sheet-column date-col">
            <span class="field-label">送達日期：</span>
            <span>{{ receipt.deliveryDate }}</span>
          </div>
          <div class="sheet-column address-col">
            <span class="field-label">送達地址：</span>
            <span>{{ receipt.address }}</span>
          </div>
          <div class="sheet-column recipient-col">
            <span>{{ receipt.recipient }}</span>
          </div>
          <div class="sheet-column giver-col">
            <span>{{ receipt.giver }}</span>
          </div>
          <div class="sheet-column item-col">
            <span>{{ receipt.item }}</span>
          </div>
          <div class="sheet-column sign-col">
            <span class="sign-title">簽收人：</span>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= 模式 3：蘭花管理系統 (五大模組) ================= -->
    <div v-else-if="currentTab === 'manage'" class="manage-container no-print">
      <!-- 次導覽列 -->
      <nav class="sub-nav">
        <button :class="{ active: subTab === 'orchid' }" @click="subTab = 'orchid'">🌸 1. 蘭花品種</button>
        <button :class="{ active: subTab === 'customer' }" @click="subTab = 'customer'">👥 2. 客戶資料</button>
        <button :class="{ active: subTab === 'inventory' }" @click="subTab = 'inventory'">📦 3. 進貨庫存</button>
        <button :class="{ active: subTab === 'return' }" @click="subTab = 'return'">🔄 4. 退貨管理</button>
        <button :class="{ active: subTab === 'order' }" @click="subTab = 'order'">💰 5. 訂單與對帳</button>
      </nav>

      <div class="manage-content">
        <!-- 模組 1: 蘭花品種庫 -->
        <section v-if="subTab === 'orchid'" class="tab-pane">
          <div class="card-box">
            <h3>🌸 新增蘭花品種資料</h3>
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
            <button class="primary-btn mt-2" @click="saveOrchid">儲存至品種庫</button>
          </div>

          <div class="card-box mt-3">
            <h3>📋 現有品種清單 ({{ orchids.length }} 筆)</h3>
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
                  <td>
                    <button class="del-btn" @click="deleteItem('orchids', item.id, loadOrchids)">刪除</button>
                  </td>
                </tr>
                <tr v-if="orchids.length === 0"><td colspan="4" class="text-center">尚無品種資料</td></tr>
              </tbody>
            </table>
          </div>
        </section>

        <!-- 模組 2: 客戶資料庫 -->
        <section v-if="subTab === 'customer'" class="tab-pane">
          <div class="card-box">
            <h3>👥 新增客戶 / 花店資料</h3>
            <div class="form-grid">
              <div class="field">
                <label>客戶 / 店鋪名稱</label>
                <input v-model="formCust.name" type="text" placeholder="例: 大吉花店、宏達批發" />
              </div>
              <div class="field">
                <label>客戶類別</label>
                <select v-model="formCust.type">
                  <option value="批發商">批發商</option>
                  <option value="花店">花店</option>
                  <option value="個人">個人</option>
                </select>
              </div>
              <div class="field">
                <label>聯絡電話</label>
                <input v-model="formCust.phone" type="text" placeholder="0912-345678" />
              </div>
              <div class="field">
                <label>Line / 備註</label>
                <input v-model="formCust.line_note" type="text" placeholder="Line 名稱或地址" />
              </div>
            </div>
            <button class="primary-btn mt-2" @click="saveCustomer">儲存客戶資料</button>
          </div>

          <div class="card-box mt-3">
            <h3>📋 現有客戶清單 ({{ customers.length }} 位)</h3>
            <table class="data-table">
              <thead>
                <tr>
                  <th>客戶編號</th>
                  <th>名稱</th>
                  <th>類別</th>
                  <th>電話</th>
                  <th>備註</th>
                  <th>操作</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="c in customers" :key="c.id">
                  <td>{{ c.id }}</td>
                  <td><b>{{ c.name }}</b></td>
                  <td><span class="badge">{{ c.type }}</span></td>
                  <td>{{ c.phone }}</td>
                  <td>{{ c.line_note }}</td>
                  <td>
                    <button class="del-btn" @click="deleteItem('customers', c.id, loadCustomers)">刪除</button>
                  </td>
                </tr>
                <tr v-if="customers.length === 0"><td colspan="6" class="text-center">尚無客戶資料</td></tr>
              </tbody>
            </table>
          </div>
        </section>

        <!-- 模組 3: 進貨與庫存 -->
        <section v-if="subTab === 'inventory'" class="tab-pane">
          <div class="card-box">
            <h3>📦 新增進貨紀錄</h3>
            <div class="form-grid">
              <div class="field">
                <label>進貨類別</label>
                <select v-model="formInv.category">
                  <option value="蘭花">蘭花</option>
                  <option value="陶瓷盆">陶瓷盆</option>
                </select>
              </div>

              <!-- 蘭花專用選項 -->
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

              <!-- 盆器專用選項 -->
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
                <label>進貨數量 (棵/個)</label>
                <input v-model.number="formInv.qty" type="number" min="1" />
              </div>
              <div class="field">
                <label>總進貨成本 (元)</label>
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
            <button class="primary-btn mt-2" @click="saveInventory">確認新增進貨</button>
          </div>

          <div class="card-box mt-3">
            <h3>📦 進貨紀錄清單 ({{ inventoryList.length }} 筆)</h3>
            <table class="data-table">
              <thead>
                <tr>
                  <th>編號</th>
                  <th>類別</th>
                  <th>品項名稱</th>
                  <th>規格</th>
                  <th>進貨數量</th>
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
                  <td>
                    <button class="del-btn" @click="deleteItem('inventory', inv.id, loadInventory)">刪除</button>
                  </td>
                </tr>
                <tr v-if="inventoryList.length === 0"><td colspan="9" class="text-center">尚無進貨資料</td></tr>
              </tbody>
            </table>
          </div>
        </section>

        <!-- 模組 4: 退貨管理區 -->
        <section v-if="subTab === 'return'" class="tab-pane">
          <div class="card-box">
            <h3>🔄 退貨與不良品登記</h3>
            <div class="form-grid">
              <div class="field">
                <label>退貨類型</label>
                <select v-model="formRet.return_type">
                  <option value="退給花農">1. 我們向花農退貨 (退給供應商)</option>
                  <option value="客戶退回">2. 批發商向我們退貨 (客戶退回)</option>
                </select>
              </div>
              <div class="field">
                <label>對象名稱 (花農/批發商)</label>
                <input v-model="formRet.party_name" list="cust-options" placeholder="選擇或手動輸入" />
                <datalist id="cust-options">
                  <option v-for="c in customers" :key="c.id" :value="c.name" />
                </datalist>
              </div>
              <div class="field">
                <label>關聯進貨品項</label>
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
                <label>總損益金額 (自動算)</label>
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
            <button class="primary-btn mt-2" @click="saveReturn">確認送出退貨紀錄</button>
          </div>

          <div class="card-box mt-3">
            <h3>📋 退貨紀錄清單 ({{ returnList.length }} 筆)</h3>
            <table class="data-table">
              <thead>
                <tr>
                  <th>編號</th>
                  <th>類型</th>
                  <th>對象</th>
                  <th>品項</th>
                  <th>株數</th>
                  <th>單價</th>
                  <th>總額</th>
                  <th>日期</th>
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
                  <td>${{ ret.unit_price }}</td>
                  <td class="text-red"><b>${{ ret.total_amount }}</b></td>
                  <td>{{ ret.date }}</td>
                  <td>{{ ret.reason }}</td>
                  <td>
                    <button class="del-btn" @click="deleteItem('returns', ret.id, loadReturns)">刪除</button>
                  </td>
                </tr>
                <tr v-if="returnList.length === 0"><td colspan="10" class="text-center">尚無退貨資料</td></tr>
              </tbody>
            </table>
          </div>
        </section>

        <!-- 模組 5: 訂單與對帳管理 -->
        <section v-if="subTab === 'order'" class="tab-pane">
          <div class="card-box">
            <h3>💰 建立新訂單</h3>
            <div class="form-grid">
              <div class="field">
                <label>客戶類型</label>
                <select v-model="formOrder.cust_type">
                  <option value="批發">批發</option>
                  <option value="花店">花店</option>
                  <option value="個人">個人</option>
                </select>
              </div>
              <div class="field">
                <label>訂購人 / 公司行號</label>
                <input v-model="formOrder.customer" list="cust-options" @change="onOrderCustSelect" placeholder="輸入或選擇客戶" />
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

              <!-- 批發選項 -->
              <template v-if="formOrder.cust_type === '批發'">
                <div class="field">
                  <label>批發株數 (棵)</label>
                  <input v-model.number="formOrder.batch_qty" type="number" min="1" />
                </div>
                <div class="field">
                  <label>每棵單價 (元)</label>
                  <input v-model.number="formOrder.batch_price" type="number" min="0" />
                </div>
              </template>
              <!-- 零售 / 花店選項 -->
              <template v-else>
                <div class="field">
                  <label>組合株數 (棵)</label>
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
                <label>下單日期</label>
                <input v-model="formOrder.order_date" type="date" />
              </div>
              <div class="field">
                <label>預計出貨日期</label>
                <input v-model="formOrder.expected_date" type="date" />
              </div>
            </div>
            <button class="primary-btn mt-2" @click="saveOrder">確認建立訂單</button>
          </div>

          <!-- 訂單總覽與對帳報表匯出 -->
          <div class="card-box mt-3">
            <div class="table-header-action">
              <h3>📋 訂單總覽與對帳清單 ({{ orderList.length }} 筆)</h3>
              <button class="excel-btn" @click="exportOrdersToExcel">📊 下載對帳 Excel 報表</button>
            </div>

            <div class="table-responsive">
              <table class="data-table">
                <thead>
                  <tr>
                    <th>單號</th>
                    <th>客戶名稱</th>
                    <th>電話</th>
                    <th>品項與規格</th>
                    <th>盆器</th>
                    <th>成本</th>
                    <th>售價</th>
                    <th>利潤</th>
                    <th>出貨狀態</th>
                    <th>收款狀態</th>
                    <th>下單日</th>
                    <th>操作</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="ord in orderList" :key="ord.id">
                    <td><b>{{ ord.id }}</b></td>
                    <td>{{ ord.customer }}</td>
                    <td>{{ ord.phone }}</td>
                    <td>{{ ord.spec }}</td>
                    <td>{{ ord.pot }}</td>
                    <td>${{ ord.cost }}</td>
                    <td class="text-blue"><b>${{ ord.price }}</b></td>
                    <td class="text-green">${{ ord.price - ord.cost }}</td>
                    <td>
                      <select v-model="ord.shipped_status" @change="updateOrderStatus(ord)">
                        <option value="未出貨">未出貨</option>
                        <option value="已出貨">已出貨</option>
                      </select>
                    </td>
                    <td>
                      <select v-model="ord.payment_status" @change="updateOrderStatus(ord)">
                        <option value="未結">未結</option>
                        <option value="已結">已結</option>
                      </select>
                    </td>
                    <td>{{ ord.order_date }}</td>
                    <td class="action-cell">
                      <button class="mini-btn print-btn" @click="fillReceiptFromOrder(ord)" title="帶入資料並切換至 A5 簽收單">
                        🖨️ 轉簽收單
                      </button>
                      <button class="mini-btn del-btn" @click="deleteItem('orders', ord.id, loadOrders)">
                        🗑️
                      </button>
                    </td>
                  </tr>
                  <tr v-if="orderList.length === 0"><td colspan="12" class="text-center">尚無訂單資料</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>
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

// ----------------- 主分頁與次分頁切換 -----------------
const currentTab = ref('manage') // 預設進到蘭花管理，可自由切換 couplet / receipt / manage
const subTab = ref('order')

// ==========================================
// 模組資料與表單狀態
// ==========================================
const orchids = ref([])
const customers = ref([])
const inventoryList = ref([])
const returnList = ref([])
const orderList = ref([])

const formOrchid = ref({ name: '', note: '標準優良品種' })
const formCust = ref({ name: '', type: '批發商', phone: '0912-345678', line_note: '' })
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
  cust_type: '個人',
  customer: '',
  phone: '0912-345678',
  orchid_name: '',
  batch_qty: 50,
  batch_price: 250,
  stalks: 10,
  pot: '桌上盆 (100)',
  cost: 600,
  price: 1500,
  order_date: new Date().toISOString().split('T')[0],
  expected_date: new Date(Date.now() + 86400000 * 3).toISOString().split('T')[0]
})

// 篩選蘭花進貨品項
const flowerInventory = computed(() => inventoryList.value.filter(i => i.category === '蘭花'))

// ==========================================
// Supabase 資料讀寫動作
// ==========================================
const loadOrchids = async () => {
  const { data } = await supabase.from('orchids').select('*').order('created_at', { ascending: false })
  if (data) orchids.value = data
}
const saveOrchid = async () => {
  if (!formOrchid.value.name) return alert('請輸入品種名稱！')
  const newId = 'DB' + Date.now().toString().slice(-5)
  const { error } = await supabase.from('orchids').insert([{ id: newId, ...formOrchid.value }])
  if (!error) {
    alert('品種新增成功！')
    formOrchid.value.name = ''
    loadOrchids()
  }
}

const loadCustomers = async () => {
  const { data } = await supabase.from('customers').select('*').order('created_at', { ascending: false })
  if (data) customers.value = data
}
const saveCustomer = async () => {
  if (!formCust.value.name) return alert('請輸入客戶名稱！')
  const newId = 'CUST' + Date.now().toString().slice(-5)
  const { error } = await supabase.from('customers').insert([{ id: newId, ...formCust.value }])
  if (!error) {
    alert('客戶建立成功！')
    formCust.value.name = ''
    loadCustomers()
  }
}

const loadInventory = async () => {
  const { data } = await supabase.from('inventory').select('*').order('created_at', { ascending: false })
  if (data) inventoryList.value = data
}
const onPotTypeChange = () => {
  const potCostMap = { '桌上盆 (100)': 100, '落地盆-喪 (100)': 100, '落地盆-喜 (200)': 200, '羅馬盆 (280)': 280 }
  formInv.value.cost = (potCostMap[formInv.value.pot_type] || 100) * formInv.value.qty
}
const saveInventory = async () => {
  const isFlower = formInv.value.category === '蘭花'
  const itemName = isFlower ? formInv.value.item_name : formInv.value.pot_type.split(' ')[0]
  if (!itemName) return alert('請輸入品項名稱！')
  const specDesc = isFlower 
    ? `規格:${formInv.value.spec_spike} | 顏色:${formInv.value.spec_color} | 大小:${formInv.value.spec_size} | 高矮:${formInv.value.spec_height}`
    : '固定規格'
  const newId = (isFlower ? 'FL' : 'POT') + Date.now().toString().slice(-5)
  const payload = {
    id: newId,
    category: formInv.value.category,
    item_name: itemName,
    spec: specDesc,
    qty: formInv.value.qty,
    cost: formInv.value.cost,
    supplier: formInv.value.supplier,
    date: formInv.value.date
  }
  const { error } = await supabase.from('inventory').insert([payload])
  if (!error) {
    alert('進貨紀錄新增成功！')
    loadInventory()
  }
}

const loadReturns = async () => {
  const { data } = await supabase.from('returns').select('*').order('created_at', { ascending: false })
  if (data) returnList.value = data
}
const saveReturn = async () => {
  if (!formRet.value.party_name) return alert('請輸入對象名稱！')
  const newId = 'RET' + Date.now().toString().slice(-5)
  const total = formRet.value.qty * formRet.value.unit_price
  const payload = {
    id: newId,
    return_type: formRet.value.return_type,
    party_name: formRet.value.party_name,
    target_item: formRet.value.target_item,
    qty: formRet.value.qty,
    unit_price: formRet.value.unit_price,
    total_amount: total,
    date: formRet.value.date,
    reason: formRet.value.reason
  }
  const { error } = await supabase.from('returns').insert([payload])
  if (!error) {
    alert('退貨紀錄儲存成功！')
    loadReturns()
  }
}

const loadOrders = async () => {
  const { data } = await supabase.from('orders').select('*').order('created_at', { ascending: false })
  if (data) orderList.value = data
}
const onOrderCustSelect = () => {
  const matched = customers.value.find(c => c.name === formOrder.value.customer)
  if (matched) {
    formOrder.value.phone = matched.phone
    formOrder.value.cust_type = matched.type === '批發商' ? '批發' : matched.type
  }
}
const saveOrder = async () => {
  if (!formOrder.value.customer) return alert('請輸入客戶名稱！')
  const newId = 'OR' + Date.now().toString().slice(-5)
  const isWholesale = formOrder.value.cust_type === '批發'
  const specStr = isWholesale 
    ? `${formOrder.value.orchid_name || '特選蘭花'} | 批發 ${formOrder.value.batch_qty}棵 (單價${formOrder.value.batch_price}元)`
    : `${formOrder.value.orchid_name || '特選蘭花'} | ${formOrder.value.stalks}棵`
  const finalPrice = isWholesale ? (formOrder.value.batch_qty * formOrder.value.batch_price) : formOrder.value.price

  const payload = {
    id: newId,
    cust_type: formOrder.value.cust_type,
    customer: formOrder.value.customer,
    phone: formOrder.value.phone,
    spec: specStr,
    pot: isWholesale ? '批發免盆' : formOrder.value.pot,
    cost: formOrder.value.cost,
    price: finalPrice,
    order_date: formOrder.value.order_date,
    expected_date: formOrder.value.expected_date,
    shipped_status: '未出貨',
    payment_status: '未結'
  }
  const { error } = await supabase.from('orders').insert([payload])
  if (!error) {
    alert('訂單建立成功！')
    loadOrders()
  }
}
const updateOrderStatus = async (ord) => {
  await supabase.from('orders').update({
    shipped_status: ord.shipped_status,
    payment_status: ord.payment_status
  }).eq('id', ord.id)
}
const deleteItem = async (table, id, reloadFn) => {
  if (!confirm(`確定要刪除編號 ${id} 嗎？`)) return
  const { error } = await supabase.from(table).delete().eq('id', id)
  if (!error) reloadFn()
}

// ==========================================
// 核心特色：一鍵帶入 A5 簽收單 & Excel 對帳
// ==========================================
const fillReceiptFromOrder = (ord) => {
  receipt.value.recipient = `${ord.customer}  ${ord.phone || ''}`
  receipt.value.item = `${ord.spec} (${ord.pot})`
  receipt.value.deliveryDate = `${ord.expected_date} 送達`
  currentTab.value = 'receipt'
  alert(`已將訂單【${ord.id}】帶入 A5 簽收單！`)
}

const exportOrdersToExcel = () => {
  if (orderList.value.length === 0) return alert('目前尚無訂單可供匯出！')
  const exportData = orderList.value.map(o => ({
    '訂單編號': o.id,
    '客戶名稱': o.customer,
    '客戶類型': o.cust_type,
    '聯絡電話': o.phone,
    '品種與規格': o.spec,
    '盆器類型': o.pot,
    '預估成本': o.cost,
    '訂單售價': o.price,
    '利潤': o.price - o.cost,
    '下單日期': o.order_date,
    '預計送達': o.expected_date,
    '出貨狀態': o.shipped_status,
    '收款狀態': o.payment_status
  }))
  const worksheet = XLSX.utils.json_to_sheet(exportData)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '蘭花訂單對帳表')
  XLSX.writeFile(workbook, `蘭花訂單對帳報表_${new Date().toISOString().split('T')[0]}.xlsx`)
}

// ==========================================
// 花卡與簽收單邏輯 (保持原有完美狀態)
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
  { text: '' }
])
const getPlaceholder = (idx) => [
  '第 1 格（例：單位 / 公司）',
  '第 2 格（例：職稱姓名 1）',
  '第 3 格（自訂聯名人 2）',
  '第 4 格（自訂）',
  '第 5 格（自訂）'
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

const receipt = ref({
  shopName: '花花戶花藝設計',
  deliveryDate: '115.9.03 送達',
  address: '桃園市桃園區縣府路 82 號 1 樓',
  recipient: '永全證券 陳柏榮總經理 03-3352155*510 江明麗秘書',
  giver: '敬領 中華民國證券商業同業公會 理事長 陳俊宏 秘書長 簡宏明',
  item: '蘭花乙盆'
})
const printReceipt = () => window.print()

const defaultVertical = {
  upper:    { x: 440, y: 70,  size: 30 },
  middle:   { x: 230, y: 150, size: 68 },
  bottom_0: { x: 105, y: 380, size: 22 },
  bottom_1: { x: 105, y: 490, size: 26 },
  bottom_2: { x: 65,  y: 380, size: 22 },
  bottom_3: { x: 65,  y: 490, size: 24 },
  bottom_4: { x: 25,  y: 380, size: 22 },
  suffix:   { x: 105, y: 640, size: 24 }
}
const defaultHorizontal = {
  upper:    { x: 140, y: 65,  size: 28 },
  middle:   { x: 160, y: 180, size: 58 },
  bottom_0: { x: 240, y: 300, size: 21 },
  bottom_1: { x: 240, y: 345, size: 24 },
  bottom_2: { x: 240, y: 390, size: 21 },
  bottom_3: { x: 240, y: 430, size: 21 },
  bottom_4: { x: 240, y: 470, size: 21 },
  suffix:   { x: 400, y: 345, size: 24 }
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
  const availableWidth = viewportRef.value.clientWidth - 24
  const cardWidth = isVertical.value ? 560 : 720
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

onMounted(() => {
  autoFitZoom()
  window.addEventListener('resize', autoFitZoom)
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

/* 頂部導航 */
.top-nav {
  height: 50px;
  background-color: #0f172a;
  color: white;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  flex-shrink: 0;
}
.nav-title { font-size: 15px; font-weight: bold; }
.nav-tabs { display: flex; gap: 8px; }
.nav-tabs button {
  background: #334155;
  color: #e2e8f0;
  border: none;
  padding: 6px 12px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 13px;
}
.nav-tabs button.active { background: #2563eb; color: white; }

/* 蘭花管理後台 */
.manage-container {
  display: flex;
  flex-direction: column;
  flex: 1;
  overflow: hidden;
}
.sub-nav {
  display: flex;
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 8px 16px;
  gap: 8px;
  overflow-x: auto;
}
.sub-nav button {
  background: #f8fafc;
  border: 1px solid #cbd5e1;
  padding: 6px 14px;
  border-radius: 6px;
  font-weight: bold;
  font-size: 13px;
  cursor: pointer;
  white-space: nowrap;
}
.sub-nav button.active {
  background: #10b981;
  color: white;
  border-color: #10b981;
}
.manage-content {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}
.card-box {
  background: white;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.04);
}
.card-box h3 {
  margin: 0 0 12px 0;
  font-size: 16px;
  color: #1e293b;
}
.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
}
.field label {
  display: block;
  font-size: 12px;
  font-weight: bold;
  color: #475569;
  margin-bottom: 4px;
}
input, select, textarea {
  width: 100%;
  padding: 8px 10px;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 13px;
  box-sizing: border-box;
}
.primary-btn {
  background: #2563eb;
  color: white;
  border: none;
  padding: 8px 18px;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
}
.excel-btn {
  background: #059669;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
}
.table-header-action {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.table-responsive { overflow-x: auto; }
.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
  text-align: left;
}
.data-table th {
  background: #f8fafc;
  padding: 8px 10px;
  border-bottom: 2px solid #e2e8f0;
  color: #475569;
}
.data-table td {
  padding: 8px 10px;
  border-bottom: 1px solid #e2e8f0;
}
.badge {
  background: #e0f2fe;
  color: #0369a1;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 11px;
}
.del-btn {
  background: #ef4444;
  color: white;
  border: none;
  padding: 3px 8px;
  border-radius: 4px;
  cursor: pointer;
}
.print-btn {
  background: #f59e0b;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 4px;
}
.text-red { color: #dc2626; }
.text-blue { color: #2563eb; }
.text-green { color: #16a34a; }
.text-center { text-align: center; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 16px; }

/* 花卡編輯器面板 */
.app-container, .receipt-container { display: flex; flex: 1; overflow: hidden; }
.control-panel {
  width: 380px;
  background: white;
  padding: 16px;
  box-shadow: 2px 0 10px rgba(0,0,0,0.06);
  overflow-y: auto;
  flex-shrink: 0;
}
.panel-section {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  padding: 10px;
  border-radius: 6px;
  margin-bottom: 10px;
}
.section-title { font-size: 13px; font-weight: bold; margin-bottom: 6px; display: block; }
.form-group { margin-bottom: 10px; }
.form-group label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 4px; }
.bottom-input-group { display: flex; align-items: center; gap: 6px; margin-bottom: 6px; }
.line-num { font-size: 12px; font-weight: bold; color: #64748b; width: 38px; }
.form-row, .btn-group { display: flex; gap: 6px; }
.btn-group button {
  flex: 1; padding: 7px; border: 1px solid #2563eb; background: white; color: #2563eb;
  border-radius: 4px; cursor: pointer; font-weight: bold;
}
.btn-group button.active { background: #2563eb; color: white; }
.radio-row { display: flex; gap: 14px; font-size: 13px; }
.tags-container { display: flex; flex-wrap: wrap; gap: 4px; }
.tag-btn {
  background: #eff6ff; color: #1e40af; border: 1px solid #bfdbfe;
  padding: 3px 6px; font-size: 12px; border-radius: 4px; cursor: pointer;
}
.reset-btn { width: 100%; padding: 8px; background: #f1f5f9; border: 1px dashed #94a3b8; border-radius: 4px; cursor: pointer; }

/* 畫布與縮放 */
.canvas-viewport {
  flex: 1; display: flex; flex-direction: column; align-items: center;
  overflow: auto; padding: 16px; position: relative; background-color: #cbd5e1;
}
.zoom-toolbar {
  display: flex; align-items: center; gap: 6px; background: white;
  padding: 5px 12px; border-radius: 20px; box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  margin-bottom: 12px;
}
.zoom-btn { width: 26px; height: 26px; border: 1px solid #cbd5e1; background: #f8fafc; border-radius: 50%; cursor: pointer; }
.zoom-text { font-size: 13px; font-weight: bold; min-width: 44px; text-align: center; }
.fit-btn { background: #2563eb; color: white; border: none; padding: 4px 10px; border-radius: 12px; font-size: 12px; cursor: pointer; }

.card-scaler-container { position: relative; }
.card-board { background: #fff; position: absolute; box-shadow: 0 10px 30px rgba(0,0,0,0.18); user-select: none; touch-action: none; }
.card-board.mode-vertical { width: 560px; height: 840px; font-family: "DFKai-SB", "BiauKai", serif; }
.card-board.mode-vertical .text-box { writing-mode: vertical-rl; text-orientation: upright; letter-spacing: 6px; }
.card-board.mode-vertical .middle-box { letter-spacing: 14px; font-weight: 900; }
.card-board.mode-horizontal { width: 720px; height: 490px; font-family: "DFKai-SB", "BiauKai", serif; }
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

/* 簽收單 */
.receipt-preview-area {
  flex: 1; display: flex; justify-content: center; align-items: center;
  overflow: auto; padding: 20px; background-color: #475569;
}
.a5-receipt-sheet {
  width: 210mm; height: 148mm; background: #ffffff; padding: 16mm 18mm;
  box-sizing: border-box; display: flex; flex-direction: row-reverse;
  justify-content: flex-start; gap: 16mm; writing-mode: vertical-rl;
  text-orientation: upright; font-family: "DFKai-SB", "BiauKai", serif; color: #111827;
}
.sheet-column { line-height: 1.6; letter-spacing: 2px; font-size: 16px; white-space: pre-wrap; }
.header-col { display: flex; align-items: center; gap: 12px; }
.shop-title { font-size: 21px; font-weight: bold; border-left: 2px solid #111; padding-left: 4px; }
.sheet-badge { color: #dc2626; font-size: 26px; font-weight: 900; letter-spacing: 6px; }
.sign-col { font-size: 18px; font-weight: bold; margin-left: auto; }
.print-action-btn {
  width: 100%; padding: 12px; background: #16a34a; color: white;
  border: none; border-radius: 6px; font-size: 15px; font-weight: bold; cursor: pointer;
}

@media (max-width: 768px) {
  .app-container { flex-direction: column; overflow-y: auto; }
  .control-panel { width: 100%; max-height: 48vh; }
  .form-grid { grid-template-columns: 1fr; }
}

@media print {
  @page { size: A5 landscape; margin: 0; }
  body, html, .main-wrapper { margin: 0 !important; padding: 0 !important; background: white !important; }
  .no-print { display: none !important; }
  .receipt-preview-area { padding: 0 !important; background: white !important; }
  .a5-receipt-sheet { box-shadow: none !important; width: 210mm !important; height: 148mm !important; }
}
</style>