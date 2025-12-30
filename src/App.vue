<template>
  <div class="scan-system-container">
    <div class="main-panel">
      <header class="panel-header">
        <div class="header-left">
          <img :src="logo" alt="flexium logo" class="system-logo">
          <h1>{{ t('LPSM 2DID掃描系統') }}</h1>
        </div>
        
        <div class="header-right">
          <select v-model="currentLang" class="lang-select">
            <option value="zh">中文</option>
            <option value="vn">Tiếng Việt</option>
          </select>
          <div class="system-status" :class="{ 'is-offline': !isBackendOnline }">
            <span>{{ isBackendOnline ? 'SYSTEM ONLINE' : 'SYSTEM OFFLINE' }}</span>
            <div class="status-light" :class="{ 'light-offline': !isBackendOnline }"></div>
          </div>
        </div>
      </header>

      <section class="info-display-section-compact">
        <div class="info-grid-compact">
          <div class="info-display-group"><span class="info-label">{{ t('工號') }}：</span><span class="info-value">{{ info.employeeId || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">{{ t('姓名') }}：</span><span class="info-value">{{ info.employeeName || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">{{ t('工單') }}：</span><span class="info-value">{{ info.workOrder || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">{{ t('品目') }}：</span><span class="info-value">{{ info.productItem || "---" }}</span></div>
        </div>
      </section>

       <div class="toggle-button-wrapper">
        <div class="sheet-count-display">
          <span class="info-label">{{ t('總張數') }}：</span><span class="info-value">{{ info.panel_num || 0 }}</span>
          <span class="info-label">{{ t('投入張數') }}：</span><span class="info-value">{{ info.OK_num || 0 }}</span>
           <span class="info-label" style="color: #ff1b78; margin-left: 10px;">NG：</span><span class="info-value" style="color: #ff1b78;">{{ info.NG_num }}</span>
        </div>
        <!-- <button class="btn btn-toggle" @click="handleToggle">{{ t('切換模式') }}</button> -->
      </div>

      <main class="camera-section-main adaptive-view">
        <template v-if="isBaseInfoFilled">
          <div class="main-content-wrapper">
            <div class="perspective-container">
              <div class="zones-in-perspective">

                <div class="zone-row">
                  <div class="platform-indicator-container process-zone">
                    <div class="platform-label" :class="getPlatformClass(_in_platform)">{{ t(_in_platform) }}</div>
                  </div>
                  <div class="zone-container process-zone">
                    <h2 class="zone-title">{{ t('作業區') }}</h2>
                    <div class="zone-grid">
                      <div class="camera-block" :class="`status-${getAreaStatus(inProcessPlatform.left)}`">
                        <span class="block-pdcode">{{ inProcessPlatform.left?.pdcode || "---" }}</span>
                      </div>
                      <div class="camera-block" :class="`status-${getAreaStatus(inProcessPlatform.right)}`">
                        <span class="block-pdcode">{{ inProcessPlatform.right?.pdcode || "---" }}</span>
                      </div>
                    </div>
                  </div>
                </div>

                <div class="zone-row staging-row">
                  <div class="platform-indicator-container staging-zone">
                    <div class="platform-label" :class="getPlatformClass(_out_platform)">{{ t(_out_platform) }}</div>
                  </div>
                  <div class="zone-container staging-zone">
                    <h2 class="zone-title">{{ t('放料區') }}<span class="staging-mode-indicator loading">{{ t("待入料") }}</span></h2>
                    <div class="zone-grid">
                      <div class="camera-block staging" :class="`status-${getAreaStatus(onloadingPlatform.left)}`">
                        <span class="block-label">{{ t('準備區-左') }}</span>
                        <span v-if="get2DIDStatusShow(onloadingPlatform.left?.ret_type)" class="block-pdcode large" :class="`status-text-${getAreaStatus(onloadingPlatform.left)}`">{{ onloadingPlatform.left?.ret_type }}</span>
                        <span class="block-pdcode large">{{ onloadingPlatform.left?.pdcode || t("等待掃描...") }}</span>
                      </div>
                      
                      <div class="camera-block staging" :class="`status-${getAreaStatus(onloadingPlatform.right)}`">
                        <span class="block-label">{{ t('準備區-右') }}</span>
                        <span v-if="get2DIDStatusShow(onloadingPlatform.right?.ret_type)" class="block-pdcode large" :class="`status-text-${getAreaStatus(onloadingPlatform.right)}`">{{ onloadingPlatform.right?.ret_type }}</span>
                        <span class="block-pdcode large">{{ onloadingPlatform.right?.pdcode || t("等待掃描...") }}</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="status-display-area">
              <div class="status-list-column">
                <div class="status-list">
                  <div v-for="item in side_history_2DID_list.left.slice(0, 5)" :key="`${item.pdcode}-${item.timestamp}`" class="history-item-wrapper" :class="{ 'is-ng-item': item.ret_type === 'NG' }">
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ formatTime(item.timestamp) }}</span>
                      <span class="item-pdcode">{{ item.pdcode }}</span>
                      <span class="item-action-type">({{ t(get2DIDActionType(item.pdcode)) }})</span> 
                      <span class="item-status-text" :class="item.ret_type === 'OK' ? 'status-ok' : 'status-ng'">{{ item.ret_type }}</span>
                    </div>
                    <div v-if="item.ret_type === 'NG'" class="list-item-ng-details">
                      <span :title="item.detail">{{ t('原因') }}: {{ t(item.detail) }}</span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="status-list-column">
                <div class="status-list">
                  <div v-for="item in side_history_2DID_list.right.slice(0, 5)" :key="`${item.pdcode}-${item.timestamp}`" class="history-item-wrapper" :class="{ 'is-ng-item': item.ret_type === 'NG' }">
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ formatTime(item.timestamp) }}</span>
                      <span class="item-pdcode">{{ item.pdcode }}</span>
                      <span class="item-action-type">({{ t(get2DIDActionType(item.pdcode)) }})</span> 
                      <span class="item-status-text" :class="item.ret_type === 'OK' ? 'status-ok' : 'status-ng'">{{ item.ret_type }}</span>
                    </div>
                    <div v-if="item.ret_type === 'NG'" class="list-item-ng-details">
                      <span :title="item.detail">{{ t('原因') }}: {{ t(item.detail) }}</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            
          </div>
        </template>
        <div v-else class="initial-prompt"><img src="./components/icons/search.png" width="50px">{{ t("請刷入工號與工單以開始作業") }}</div>
      </main>
      <footer class="panel-footer">
        <button class="btn btn-resetuser" @click.stop="employeeClear">{{ t('換人生產') }}</button>
        <button class="btn btn-reset" @click.stop="resetAll">{{ t('重整') }} (Reset)</button>
        <button class="btn btn-execute" @click.stop="forceExecute">{{ t('強制執行') }}</button>
        <button class="btn btn-complete" @click.stop="completeAllScanned">{{ t('完成') }} (Complete)</button>
      </footer>
   </div>
  </div>
  
  <dialog ref="customModal" class="custom-modal" @cancel.prevent>
    <div class="modal-content">
      <p class="modal-message">{{ modalMessage }}</p>
      <div class="modal-actions" v-if="modalType === 'confirm'">
        <button class="btn btn-modal-cancel" @click="cancelModal">{{ t('取消') }}</button>
        <button class="btn btn-modal-confirm" @click="confirmModal">{{ t('確定') }}</button>
      </div>
      <div class="modal-actions" v-else>
        <button class="btn btn-modal-confirm" @click="closeModal">{{ t('確定') }}</button>
      </div>
    </div>
  </dialog>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch, reactive } from 'vue';
import { i18nDict } from "@/textManager/translate"
import logo from "@/components/icons/flexium_logo.png"

const API_URL = "http://10.1.5.122/gxfirstOIS/gxfirstOIS.asmx/GetOISData";
// const OIS_API_BASE = "http://127.0.0.1:2151";
const OIS_API_BASE = "http://10.8.32.64:2151";
const wsUrl = 'ws://127.0.0.1:8181'
const headers = { 'Content-Type': 'application/json' };

const currentLang = ref('zh'); 

const t = (key) => {
  if (!key) return key;
  if (currentLang.value === 'zh') return key;
  return i18nDict[key]?.vn || key;
};

// Process stage
const stage = ref("empId");
const forceCommand = reactive({command: false, triggerTime: null});
const unloadPeriodThreshold = 10000;
const shortPeriodFilterThreshold = 5000;
const forceTriggerPeriod = 10000;

// PLC state
const PlatformState = reactive({ up_in: -1, dn_in: -1, start_message: -1 });

// Other data
let info = reactive({
  employeeId: "",
  employeeName: "",
  workOrder: "",
  productItem: "",
  workStep: "",
  panel_num: 0,
  OK_num: 0,
  NG_num: 0,
  cmd236_flag: false,
})
const emptySlot = () => ({ pdcode: "", ret_type: "", detail: "" });
let requested_2DID_dict = reactive({});  // Storage all request 2DID dict for other NG sheet to find panel_no which can post API /write_2did; CMD=238 can not response panel_no
let expected_2DID_dict = reactive({});  // Expected 2DID dict to scaned on this workorder
let scanned_2DID_dict = reactive({});  // Scanned 2DID dict on this workorder
let other_2DID_dict = reactive({});  // Reported error 2DID dict
let side_history_2DID_dict = reactive({left: {}, right: {}});  // Save all history scanned 2DID (for save 2DID information)
let side_history_2DID_list = reactive({left: [], right: []});  // List all history scanned 2DID (filter short period)
let history_2DID_dict = reactive({});  // Save all history scanned 2DID dict
let up_platform_2DID = reactive({right: emptySlot(), left: emptySlot()});
let dn_platform_2DID = reactive({right: emptySlot(), left: emptySlot()});
let last_OK_upload_2DID = reactive({up: {left: {pdcode: "", detect: false}, right: {pdcode: "", detect: false}}, dn: {left: {pdcode: "", detect: false}, right: {pdcode: "", detect: false}}});

// --- Offline & Failover State --- //
const isBackendOnline = ref(true);
let backendHeartbeatTimer = null;

// [修改 1] 移除 localStorage，初始化為空陣列，等待 WebSocket 載入
const offlineRequestQueue = reactive([]);

// Display state
const isBaseInfoFilled = computed(() => !!(info.employeeId && info.workOrder && info.employeeName && info.productItem));
const formatTime = (ms) => new Date(ms).toLocaleTimeString();
const _in_platform = computed(() => {
  if (PlatformState.up_in == 1) return "上台面";
  else return "下台面";
})
const _out_platform = computed(() => {
  if (_in_platform.value === "上台面") return "下台面";
  else return "上台面";
})
const getPlatformClass = (text) => {
    if (text === "上台面") return "platform-upper";
    if (text === "下台面") return "platform-lower";
    return "";
};
const inProcessPlatform = computed(() => {
  if (PlatformState.up_in == 1) {
    return up_platform_2DID;
  }
  else {
    return dn_platform_2DID;
  }
})
const onloadingPlatform = computed(() => {
  if (PlatformState.up_in != 1) {
    return up_platform_2DID;
  }
  else {
    return dn_platform_2DID;
  }
})
const get2DIDActionType = (pdcode) => {
  if (scanned_2DID_dict[pdcode] && scanned_2DID_dict[pdcode].ret_type == "OK") return "出";
  return "進";
};
const getAreaStatus = (area) => {
  const rt = area?.ret_type || "";
  if (rt === "OK") return "success";
  if (rt === "NG") return "error";
  return "waiting";
};
const get2DIDStatusShow = (status) => {
  if (status && status.length > 0) 
    return true;
  return false;
};

// --- Websocket Process --- ///
let reconnectDelay = 3000; // <--- 補上這行
let reconnectTimer = null; // <--- 補上這行
let socket = null;
function connectWebSocket() {
  if (socket) {
    // 避免重複建立
    return;
  }
  
  socket = new WebSocket(wsUrl);
  const timeoutMs = 1000;
  const timer = setTimeout(() => {
    if (socket && socket.readyState === WebSocket.CONNECTING) socket.close(4000, "connect timeout");
  }, timeoutMs);

  
  socket.onopen = () => { 
    console.log("WS Connected");
    reconnectDelay = 3000; // ✅ 連線成功，重置等待時間
    clearTimeout(timer);
    startHeartbeat();
    sendSocketMessage("LOAD_OFFLINE_CACHE", {});  // backend communicaiton successful then get local storage API
  };

  socket.onmessage = (event) => {
    try {
      const message = JSON.parse(event.data);
      if (message.type === "data") {
        // Process PLC state
        if (message.source === "PLC_MONITOR") {
          const p = message.payload;
          PlatformState.up_in = p.up_in;
          PlatformState.dn_in = p.dn_in;
          PlatformState.start_message = p.start_message;
        }
        // Process information display
        else if (message.source === "info") {
          console.log("WebSocket Info: ", message.message);
        }
        // Process infomation input from keyboard
        else if (message.source === "SCANNER") {
          handleInfoInput(message.payload);
        }
        // Process camera input
        else if (message.source.includes("CAMERA")) {
          console.log("Receive Camera data payload.")
          processControl(message.payload, message.source);
        }
        // 接收後端回傳的離線檔案資料
        else if (message.source === "SYS" && message.payload && message.payload.type === "OFFLINE_CACHE_LOADED") {
          const loadedData = message.payload.data; // 預期是一個 Array
          if (Array.isArray(loadedData) && loadedData.length > 0) {
            console.log(`[Offline] Loaded ${loadedData.length} records from backend file.`);
            // 清空當前並載入
            offlineRequestQueue.splice(0, offlineRequestQueue.length, ...loadedData);

            startBackendHeartbeat();
          }
        }
        else { }
      }
      else if (message.type === "control" && message.command === "HEARTBEAT_ACK") {
        lastHeartbeatAck.value = message.payload?.server_ts || Date.now();
        return;
      }
    }
    catch (error) {
      console.error("WebSocket 解析訊息失敗: ", error);
    }
  }

  socket.onclose = () => {
    console.log(`WebSocket: 連接已關閉，${reconnectDelay/1000}秒後嘗試重連...`);
    stopHeartbeat();
    socket = null;

    // 清空狀態
    PlatformState.up_in = -1;
    PlatformState.dn_in = -1;
    PlatformState.start_message = -1;

    // ✅ 指數退避：每次失敗等待時間加倍，上限 30秒，這樣可以避免短時間內產生大量 CLOSE_WAIT
    if (reconnectTimer) clearTimeout(reconnectTimer);
    reconnectTimer = setTimeout(() => { connectWebSocket(); }, reconnectDelay);
    reconnectDelay = Math.min(reconnectDelay * 1.5, 30000); 
  };

  socket.onerror = (error) => {
    console.error("WebSocket 發生錯誤: ", error);
  }
}

function sendSocketMessage(command, payload) {
  if (socket && socket.readyState === WebSocket.OPEN) {
    let message = { type: "control", command, payload};
    socket.send(JSON.stringify(message));
    console.log(`[WS發送] command: ${command}, payload: ${payload}`);
  }
}

let hbTimer = null;
const lastHeartbeatAck = ref(null);
function startHeartbeat() {
  stopHeartbeat();
  hbTimer = setInterval(() => {
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify({ type: "control", command: "HEARTBEAT", payload: { ts: Date.now(), page: "scan-ui" } }));
    }
  }, 2000);
}

function stopHeartbeat() {
  if (hbTimer) {
    clearInterval(hbTimer);
    hbTimer = null;
  }
} 

// ---- wrappers ----
const Ajax = async (url, options, time) => {
  const controller = new AbortController();
  setTimeout(() => { controller.abort(); }, (isBackendOnline.value) ? time : 100);
  let config = { ...options, signal: controller.signal };

  try {
    let response = await fetch(url, config);
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    let responseJson = await response.json();
    
    if (!isBackendOnline.value) {
      console.log("Backend recovered via Ajax success.");
      isBackendOnline.value = true;
      if (backendHeartbeatTimer) {
        clearInterval(backendHeartbeatTimer);
        backendHeartbeatTimer = null;
      }
      processOfflineQueue();
    }
    return responseJson;
  } catch (error) {
    const isWriteApi = url.includes('/write2did') || url.includes('/write2dids');
    const isDeleteApi = url.includes('/Delete_2DID');

    if (isWriteApi || isDeleteApi) {
      console.warn(`[Offline Mode] Backend unreachable. Queuing request for: ${url}`);
      
      const payload = JSON.parse(options.body);
      const apiType = url.substring(url.lastIndexOf('/') + 1);
      
      const offlineRecord = { api: apiType, body: payload, timestamp: Date.now() };

      // 1. 將請求存入前端記憶體 (為了即時顯示或後續處理)
      offlineRequestQueue.push(offlineRecord);
      
      // 2. Append API payload to storage in local file
      sendSocketMessage("APPEND_OFFLINE_CACHE", offlineRecord);

      // 3. 標記後台離線，啟動 Heartbeat
      if (isBackendOnline.value) {
        isBackendOnline.value = false; // 立即標記為離線，阻擋後續請求
        startBackendHeartbeat(); 
        showCustomModal(t("與後台連線中斷，已切換至本地儲存模式"), "alert");
      }

      return { success: true, offline: true };
    }
    throw error;
  }
}

// --- Heartbeat & Recovery Logic --- //
function startBackendHeartbeat() {
  if (backendHeartbeatTimer) return;
  
  isBackendOnline.value = false;
  console.log("Starting Backend Heartbeat...");
  backendHeartbeatTimer = setInterval(async () => {
    try {
      const controller = new AbortController();
      setTimeout(() => controller.abort(), 500); // 0.5s timeout
      const res = await fetch(`${OIS_API_BASE}/heartbeat`, { method: 'GET', signal: controller.signal });

      if (res.ok) {
        isBackendOnline.value = true;
        console.log("Backend Restored. Processing offline queue...");

        // Destroy timer and process offline queue
        clearInterval(backendHeartbeatTimer);
        backendHeartbeatTimer = null;
        processOfflineQueue();
      }
    } catch (e) { }  // If not connect failed, then skip.
  }, 5000);
}

// 修改後的 processOfflineQueue
async function processOfflineQueue() {
  if (offlineRequestQueue.length === 0) return;

  const currentBatch = [...offlineRequestQueue];
  const uploadBatch = [];
  const deleteRequests = [];

  currentBatch.forEach((req) => {
    if (req.api === 'write2did') {
      uploadBatch.push(req.body);
    } else if (req.api === 'write2dids') {
      if (Array.isArray(req.body)) uploadBatch.push(...req.body);
    } else if (req.api === 'Delete_2DID') {
      deleteRequests.push(req.body);
    }
  });

  try {
    // 2. 處理批量上傳
    if (uploadBatch.length > 0) {
      const res = await fetch(`${OIS_API_BASE}/api/write2dids`, { method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(uploadBatch) });
      if (!res.ok) throw new Error("Batch upload failed");
      console.log("Batch upload success.");
    }

    // 3. 處理刪除請求
    for (const delReq of deleteRequests) await fetch(`${OIS_API_BASE}/api/Delete_2DID`, { method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(delReq) });

    offlineRequestQueue.splice(0, currentBatch.length);

    // 檢查是否還有新資料進來 (在 await 期間)
    if (offlineRequestQueue.length === 0) {
      sendSocketMessage("CLEAR_OFFLINE_CACHE", {});
      console.log("All offline requests synced and local cache cleared.");
    } else {
      console.log("New offline data detected, processing next batch...");
      processOfflineQueue(); 
    }
  } catch (error) {
    console.error("Failed to sync offline queue:", error);
    startBackendHeartbeat();
  }
}

// --- Custom Modal --- //
const customModal = ref(null);
const modalMessage = ref('');
const modalType = ref('alert'); // 'alert' or 'confirm'
let resolveModalPromise = null;

const showCustomModal = (message, type = 'alert') => {
  modalMessage.value = message;
  modalType.value = type;
  if (customModal.value){
    customModal.value.showModal();
  }

  return new Promise((resolve) => { resolveModalPromise = resolve; });
};

const closeModal = () => {
  customModal.value.close();
  if (resolveModalPromise) {
    resolveModalPromise(true);
    resolveModalPromise = null;
  }
};

const confirmModal = () => {
  customModal.value.close();
  if (resolveModalPromise) {
    resolveModalPromise(true);
    resolveModalPromise = null;
  }
};

const cancelModal = () => {
  customModal.value.close();
  if (resolveModalPromise) {
    resolveModalPromise(false);
    resolveModalPromise = null;
  }
};

// --- API Request --- //
async function validateEmpId(empId) {
  try {
    const requestData = new URLSearchParams()
    requestData.append("CmdCode","5")
    requestData.append("InMessage_Json", JSON.stringify({ Emp_NO: empId }))
    const response = await Ajax(API_URL, { method: "POST", headers: { "Content-Type": "application/x-www-form-urlencoded" }, body: requestData.toString() }, 1500);
    const responseText = await response.text();
    const json = JSON.parse(responseText);
    if (json.code === 200 && json.data) {
      const {empNo, empName} = json.data;
      return {empNo, empName};
    } else {
      // await showCustomModal(`工號驗證失敗：${json.message || '未知錯誤'}`);
      await showCustomModal(t('工號驗證失敗'));
      return false;
    }
  } catch(error) {
    console.error("validateEmpId 發生錯誤:", error);
    await showCustomModal(t('工號驗證時發生網路錯誤：請確認網路連線'));
    return false;
  }
}

async function validateWorkOrder(workOrder) {
  try {
    const res = await Ajax(`${OIS_API_BASE}/api/workorder`, { method: 'POST', headers, body: JSON.stringify({ emp_no: info.employeeId, workorder: workOrder, insert_to_database: true }) }, 3000);

    if (res.success && res.data) {
      const d = res.data;
      return { cmd236_flag: d.cmd236_flag, workorder: d.workorder, item: d.item, workStep: d.workStep, panel_num: d.panel_num, sht_no: d.sht_no, panel_no: d.panel_no, twodid_step: d.twodid_step, twodid_type: d.twodid_type, scanned_data: d.scanned_data };
    }

    if (res.success == false && res.type == "mes_offline") {
      await showCustomModal(t('因與 IT server 網路中斷，因此工單查詢失敗'));
      return null;
    }

    await showCustomModal(t('工單驗證失敗：查無資料'));
    return null;
  }
  catch (error) {
    console.error("validateWorkOrder 發生錯誤: ", error)
    await showCustomModal(t('工單驗證時發生網路錯誤，請確認網路連線'));
    return null;
  }
}

async function validate2DID(twodid) {
  if (!twodid) return { ret_type: "NG", workOrder: "N/A", item: "N/A", detail: "條碼異常" };

  const sheet_info = expected_2DID_dict[twodid];
  if (sheet_info) {
    if (sheet_info.twodid_type == "Y") {
      await sheetReport(twodid, sheet_info.panel_no, "NG", "跳站");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "跳站" };
    }
    else if (sheet_info.twodid_step === info.workStep) {
      await sheetReport(twodid, sheet_info.panel_no, "NG", "重工");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
    }

    const scanned_record = scanned_2DID_dict[twodid];
    if (scanned_record && Date.now() - scanned_record.timestamp > unloadPeriodThreshold) {
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
    }
    else if (scanned_record) {
      return { ret_type: "OK", workOrder: info.workOrder, item: info.productItem, detail: "已完成" };
    }
    return { ret_type: "OK", workOrder: info.workOrder, item: info.productItem, detail: "" };
  }

  // This 2DID has detect in the history and reported error to server, just skip
  if (other_2DID_dict[twodid]) {
    return other_2DID_dict[twodid];
  }

  let detail = "";
  // This 2DID has not report error in the history, but in the requested 2DID dict
  if (requested_2DID_dict[twodid]) {
    const twodid_info = requested_2DID_dict[twodid];
    if (twodid_info.item != info.productItem) detail += "混品目;"; 
    else if (twodid_info.workOrder != info.workOrder) detail += "混工單;";

    if (detail === "") {
      await showCustomModal(t("MES系統無該製品資訊"));
      detail = "異常錯誤";
    }
    else {
      other_2DID_dict[twodid] = { ret_type: "NG", workOrder: twodid_info.workOrder, item: twodid_info.item, detail: (detail === "") ? "異常錯誤" : detail };
      await sheetReport(twodid, twodid_info.panel_no, "NG", (detail === "") ? "異常錯誤" : detail);
    }

    return { ret_type: "NG", workOrder: twodid_info.workOrder, item: twodid_info.item, detail: (detail === "") ? "異常錯誤" : detail};
  }

  // This 2DID has not detect in the history requested need to get the information from server
  try {
    const res_json = await Ajax(`${OIS_API_BASE}/api/twodid`, { method: 'POST', headers, body: JSON.stringify({ emp_no : info.employeeId, twodid }) }, 1000);
    if (res_json.success) {
      const [_, workOrder, productItem, workStep] = res_json.result.result.split(";");

      if (productItem != info.productItem) detail += "混品目;";
      else if (workOrder != info.workOrder) detail += "混工單;";
      other_2DID_dict[twodid] = { ret_type: "NG", workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };

      if (detail != ""){
        const panels_info = await validateWorkOrder(workOrder);
        if (panels_info) {
          create2DIDDict(panels_info, false, false);
          const twodid_info = requested_2DID_dict[twodid];
          await sheetReport(twodid, twodid_info.panel_no, "NG", detail);
        }
        else {
          await showCustomModal(t("此2DID之工單查無任何資料，因此不上傳資訊"));
        }
      }

      return { ret_type: "NG", workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };
    }

    if (res.success == false && res.type == "mes_offline") {
      await showCustomModal(t('因與 IT server 網路終端，因此 2DID 資訊查詢失敗'));
    }

    other_2DID_dict[twodid] = { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
    return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
  }
  catch (error) {
    await showCustomModal(t("驗證2DID發生錯誤，請確認網路連線"));
    // console.error("驗證2DID發生錯誤，請確認網路連線: ", error);
  }

  return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
}

// --- Control Process --- //
// Chage stage
function changeStage(s) {
  stage.value = s;
  sendSocketMessage("STEP_UPDATE", s);
}

// Reset alls storage data
function resetAll() {
  Object.assign(requested_2DID_dict, {});
  Object.assign(expected_2DID_dict, {});
  Object.assign(scanned_2DID_dict, {});
  Object.assign(other_2DID_dict, {});
  Object.assign(side_history_2DID_dict, {right: {}, left: {}});
  Object.assign(side_history_2DID_list, {right: [], left: []});
  Object.assign(history_2DID_dict, {});
  Object.assign(up_platform_2DID, {right: emptySlot(), left: emptySlot()});
  Object.assign(dn_platform_2DID, {right: emptySlot(), left: emptySlot()});

  Object.assign(info, { employeeId: "", employeeName: "", workOrder: "", productItem: "", workStep: "", panel_num: 0, OK_num: 0, NG_num: 0, cmd236_flag: false });
  Object.assign(forceCommand, { command: false, triggerTime: null });

  changeStage("empId");
}

// Storage history request and list expected 2DID in target work order
function create2DIDDict(panelList, initial = true, addExpected = true) {
  const sht_no_list = panelList.sht_no;
  const panel_no_list = panelList.panel_no;
  const twodid_step_list = panelList.twodid_step;
  const twodid_type_list = panelList.twodid_type;

  if (initial){
    Object.assign(requested_2DID_dict, {});
    Object.assign(expected_2DID_dict, {});
    Object.assign(scanned_2DID_dict, {});
    Object.assign(other_2DID_dict, {});
  }

  for (let index = 0; index < panelList.panel_num; index++) {
    requested_2DID_dict[sht_no_list[index]] = { workOrder: panelList.workorder, item: panelList.item, workStep: panelList.workStep, panel_no: panel_no_list[index], twodid_step: twodid_step_list[index], twodid_type: twodid_type_list[index] };
    if (addExpected){
      expected_2DID_dict[sht_no_list[index]] = { panel_no: panel_no_list[index], twodid_step: twodid_step_list[index], twodid_type: twodid_type_list[index] };
    }
  }

  if (addExpected) {
    info.cmd236_flag = panelList.cmd236_flag;
    if (Array.isArray(panelList.scanned_data) && panelList.scanned_data.length > 0) {
      for (const sd of panelList.scanned_data) {
        scanned_2DID_dict[sd.sheet_no] = { timestamp: sd.timestamp, ret_type: sd.twodid_type, workOrder: info.workOrder, item: info.productItem, detail: sd.twodid_status };
        if (sd.twodid_type === 'OK') info.OK_num += 1;
        else if (sd.twodid_type === 'NG') info.NG_num += 1;
      }
    }
  }
}

// Handle employee login and check work order valid
const handleInfoInput = async (data) => {
  const value = String(data).replace(/[\x00-\x1F\x7F-\x9F]/g, "").trim().toUpperCase();

  if (!value) return;

  if (stage.value === "empId") {
    // const result = await validateEmpId(value);
    const result = {empNo: "12868", empName: "王巨成"};

    if (!result) return;

    info.employeeId = result.empNo;
    info.employeeName = result.empName;
    changeStage((info.workOrder === "") ? "workOrder" : "twoDID");
  }

  else if (stage.value === "workOrder") {
    if (value.length < 8 || value.length > 11) {
      await showCustomModal(t("工單格式不正確，請重新掃描。"));
      return;
    }

    const valid = await validateWorkOrder(value);
    if (valid && valid.workStep !== ""){
      info.workOrder = valid.workorder;
      info.productItem = valid.item;
      info.workStep = valid.workStep;
      info.panel_num = valid.panel_num;
      info.OK_num = 0;
      info.NG_num = 0;
      create2DIDDict(valid);
      changeStage("twoDID");
    }
  }
}

// Check platform at loading area
function detectTargetPlatform() {
  if (PlatformState.up_in != 1) return up_platform_2DID;
  else return dn_platform_2DID;
}
function detectOtherPlatform() {
  if (PlatformState.up_in == 1) return up_platform_2DID;
  else return dn_platform_2DID;
}

// Process 2DID control
const processControl = async (data, source) => {
  // 1. Check value valid
  // 2. Get loading platform
  // 3. Process 2DID TIMEOUT signal
  // 4. Get product 2DID information
  // 5. Record 2DID information to history dictionary
  // 6. Refresh target platform product information
  // 7. Send M86, M87 status to local python
  const value = String(data).replace(/[\x00-\x1F\x7F-\x9F]/g, "").trim().toUpperCase();
  if (value.length !== 13) {
    await showCustomModal(t("產品條碼長度不正確，請重新掃描。"));
    return;
  }

  // Check platform at loading area
  let targetPlatform = detectTargetPlatform();
  if (targetPlatform == null) return null;
  let targetPlatformArea = source.startsWith("CAMERA_LEFT") ? targetPlatform.left : targetPlatform.right;
  let otherPlatformArea = source.startsWith("CAMERA_LEFT") ? detectOtherPlatform().left : detectOtherPlatform().right;

  // Receive camera timeout signal
  if (value == "TIMEOUT_BLANK") {
    const lastArea2DID = source.startsWith("CAMERA_LEFT") ? side_history_2DID_list.left[0] : side_history_2DID_list.right[0];
    if (!lastArea2DID || Date.now() - lastArea2DID.timestamp > 1500) {
      Object.assign(targetPlatformArea, { pdcode: "等待掃描...", ret_type: "", detail: "" });
    }

    // Detect force information
    if (forceCommand.command === true && (forceCommand.triggerTime && Date.now() - forceCommand.triggerTime < forceTriggerPeriod) && (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK")) {
      sendSocketMessage("GO_NOGO", 1);
    }
    else {
      sendSocketMessage("GO_NOGO", 0);
    }
    return;
  }

  // Check product validate
  let targetSheet = (source.startsWith("CAMERA_RIGHT") || source.startsWith("CAMERA_LEFT")) ? value : null;
  let validResult = await validate2DID(targetSheet);
  let rec = { timestamp: Date.now(), ret_type: validResult.ret_type, workOrder: validResult.workOrder, item: validResult.item, detail: validResult.detail };

  const two_camera_pass = rec.ret_type == "OK" && ((targetPlatform.left.ret_type == 'OK' && source.startsWith("CAMERA_RIGHT")) || (targetPlatform.right.ret_type && source.startsWith("CAMERA_LEFT")))
  if (info.OK_num === info.panel_num || ((info.OK_num == info.panel_num - 1) && two_camera_pass)) {
    if (rec.ret_type == "OK") {
      rec.ret_type = "NG";
      rec.detail = "超過 PANEL 上限";
    }
    await showCustomModal(t('已達 PANEL 數上限，因此無法綁定。'));
  }

  // Filtered short period repeated 2DID for frontend log
  const sheet_history = (source.startsWith("CAMERA_LEFT")) ? side_history_2DID_dict.left[value] : side_history_2DID_dict.right[value];
  const last_info = (source.startsWith("CAMERA_LEFT")) ? side_history_2DID_list.left[0] : side_history_2DID_list.right[0];
  if (sheet_history && (last_info && (last_info.pdcode == value)) && (Date.now() - sheet_history.timestamp < shortPeriodFilterThreshold)) {
    sheet_history.timestamp = Date.now();
  }
  else {
    if (source.startsWith("CAMERA_LEFT")) { 
      side_history_2DID_list.left.unshift({ ...rec, pdcode: value });
      side_history_2DID_dict.left[value] = { ...rec };
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      side_history_2DID_list.right.unshift({ ...rec, pdcode: value });
      side_history_2DID_dict.right[value] = { ...rec };
    }
  }

  // Filtered short period repeated 2DID to record
  if (history_2DID_dict[value]) {
    history_2DID_dict[value].timestamp = Date.now();
  }
  else {
    history_2DID_dict[value] = { ...rec };
  }

  // Check 2DID is processed 2DID
  if (value == targetPlatformArea.pdcode || value == otherPlatformArea.pdcode) {
    const upPlatformUpload2DIDArea = (source.startsWith("CAMERA_LEFT")) ? last_OK_upload_2DID.up.left : last_OK_upload_2DID.up.right;
    const dnPlatformUpload2DIDArea = (source.startsWith("CAMERA_RIGHT")) ? last_OK_upload_2DID.dn.left : last_OK_upload_2DID.dn.right;
    if (value == upPlatformUpload2DIDArea.pdcode) upPlatformUpload2DIDArea.detect = true;
    else if (value == dnPlatformUpload2DIDArea.pdcode) dnPlatformUpload2DIDArea.detect = true;
  }
  // Filled the information for frontend
  else if (value != otherPlatformArea.pdcode) {
    Object.assign(targetPlatformArea, { pdcode: value, ret_type: validResult.ret_type, detail: validResult.detail });
  }

  // Send M86, M87 command to PLC
  const oneSheetNG = targetPlatform.left.ret_type == "NG" || targetPlatform.right.ret_type == "NG";
  const lastoneSheetOK = !oneSheetNG && (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK");
  const CompleteSheet = targetPlatform.left.detail === "已完成" || targetPlatform.right.detail === "已完成"
  if (!oneSheetNG && forceCommand.command === true && (forceCommand.triggerTime && Date.now() - forceCommand.triggerTime < forceTriggerPeriod) && ((targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK") && !CompleteSheet)) {
    sendSocketMessage("GO_NOGO", 1);
  }
  else if (stage.value == "twoDID" && ((targetPlatform.left.ret_type === "OK" && targetPlatform.right.ret_type === "OK" && !CompleteSheet) || ((info.OK_num == info.panel_num - 1) && lastoneSheetOK))) {
    sendSocketMessage("GO_NOGO", 1);
  }
  else {
    sendSocketMessage("GO_NOGO", 0);
  }
}

watch(
  () => [PlatformState.up_in, PlatformState.dn_in], ([up, dn], [prevUp, prevDn]) => {
    if (up === 1 && prevUp !== 1) scanedSheetRecord(up_platform_2DID, "up");
    if (dn === 1 && prevDn !== 1) scanedSheetRecord(dn_platform_2DID, "dn");
    if (up !== 1 && prevUp === 1) setTimeout(() => {lastOKUploadSheetDetect("up");}, 1000);  // Delay 1 second to check 2DID
    if (dn !== 1 && prevDn === 1) setTimeout(() => {lastOKUploadSheetDetect("dn");}, 1000);  // Delay 1 second to check 2DID
  }
);

async function scanedSheetRecord(platform, info) {
  let targetPlatformUpload2DID = (info == "up") ? last_OK_upload_2DID.up : last_OK_upload_2DID.dn;

  if (platform.left.ret_type == "OK" && !scanned_2DID_dict[platform.left.pdcode]) {
    Object.assign(targetPlatformUpload2DID.left, {pdcode: platform.left.pdcode, detect: false});
  }

  if (platform.right.ret_type == "OK" && !scanned_2DID_dict[platform.right.pdcode]) {
    Object.assign(targetPlatformUpload2DID.right, {pdcode: platform.right.pdcode, detect: false});
  }
}

async function lastOKUploadSheetDetect(info) {
  let targetPlatformUpload2DID = (info == "up") ? last_OK_upload_2DID.up : last_OK_upload_2DID.dn;
  if ((targetPlatformUpload2DID.left.pdcode != "" && targetPlatformUpload2DID.left.detect == false) || (targetPlatformUpload2DID.right.pdcode != "" && targetPlatformUpload2DID.right.detect == false)) {
    showCustomModal(t("進出料製品 2DID 不同，請確認機台是否操作正常"));
  }

  if (history_2DID_dict[targetPlatformUpload2DID.left.pdcode]) {
    const pdcodeInfo = expected_2DID_dict[targetPlatformUpload2DID.left.pdcode];
    history_2DID_dict[targetPlatformUpload2DID.left.pdcode].timestamp = Date.now();
    sheetReport(targetPlatformUpload2DID.left.pdcode, pdcodeInfo.panel_no, "OK", "");
  }
  if (history_2DID_dict[targetPlatformUpload2DID.right.pdcode]) {
    const pdcodeInfo = expected_2DID_dict[targetPlatformUpload2DID.right.pdcode];
    history_2DID_dict[targetPlatformUpload2DID.right.pdcode].timestamp = Date.now();
    sheetReport(targetPlatformUpload2DID.right.pdcode, pdcodeInfo.panel_no, "OK", "");
  }
  
  Object.assign(targetPlatformUpload2DID, {left: {pdcode: "", detect: false}, right: {pdcode: "", detect: false}});
}

const sheetReport = async(pdcode, panel_no, type, detail) => {
  if (pdcode && (pdcode.length == 0)) return;

  if (expected_2DID_dict[pdcode] && !scanned_2DID_dict[pdcode]) {
    scanned_2DID_dict[pdcode] = { timestamp: Date.now(), ret_type: type, workOrder: info.workOrder, item: info.productItem, detail };

    try { await UploadSheet(pdcode, panel_no, type, detail); } 
    catch (error) { await showCustomModal(t(`製品 ${type} 狀態上傳失敗，請確認網路連線！`)); }

    if (type == "OK") info.OK_num += 1;
    else if (type == "NG") info.NG_num += 1;
  }
  else if (!expected_2DID_dict[pdcode] && !other_2DID_dict[pdcode]){
    try { await UploadSheet(pdcode, panel_no, type, detail); } 
    catch (error) { await showCustomModal(t(`製品 ${type} 狀態上傳失敗，請確認網路連線！`)); }
  }
}

const UploadSheet = async (pdcode, panel_no, ret_type, status) => {
  const payload = { emp_no: info.employeeId, workOrder: info.workOrder, item: info.productItem, workStep: info.workStep, sht_no: pdcode, panel_no: panel_no, twodid_type: ret_type, remark: status };
  await Ajax(`${OIS_API_BASE}/api/write2did`, { method: "POST", headers, body: JSON.stringify(payload) }, 200);
}

// --- Button Control --- //
function employeeClear() {
  info.employeeId = "";
  info.employeeName = "";
  changeStage("empId");
}

async function forceExecute() {
  let targetPlatform = detectTargetPlatform();
  if (targetPlatform == null) {
    await showCustomModal(t("請等待平台退回上料區"));
    return;
  }

  if (info.OK_num == info.panel_num || ((info.OK_num == info.panel_num - 1) && targetPlatform.left.ret_type === "OK" && targetPlatform.right.ret_type === "OK")) {
    await showCustomModal(t("作業後將達 PANEL 數上限，因此無法繼續執行。"));
    return;
  }

  if (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK") {
    forceCommand.command = true;
    forceCommand.triggerTime = Date.now();
    sendSocketMessage("GO_NOGO", 1);
    return;
  }

  await showCustomModal(t("請放入至少放入一個OK製品"));
}

const completeAllScanned = async () => {
  if ((up_platform_2DID.left?.ret_type != "" || up_platform_2DID.right?.ret_type != "" || dn_platform_2DID.left.ret_type != "" || dn_platform_2DID.right.ret_type != "")) {
    await showCustomModal(t("請保持平台無任何製品"));
    return;
  }

  if (info.cmd236_flag == false){
    try {
      console.log("fill other products NG");

      let notScanned_2DID_list = [];
      for (const [pdcode, sinfo] of Object.entries(expected_2DID_dict)) {
        if (!scanned_2DID_dict[pdcode]) {
          notScanned_2DID_list.push({ emp_no: info.employeeId, workOrder: info.workOrder, workStep: info.workStep, item: info.productItem, sht_no: pdcode, panel_no: sinfo.panel_no, twodid_type: "NG", remark: "未投入，作業結束" })
        }
      }
      
      await Ajax(`${OIS_API_BASE}/api/write2dids`, { method: "POST", headers, body: JSON.stringify(notScanned_2DID_list) }, 3000);
      await Ajax(`${OIS_API_BASE}/api/Delete_2DID`, { method: "POST", headers, body: JSON.stringify({ workorder: info.workOrder }) }, 1500);
    }
    catch (error) {
      console.error("清除 2DID 資料失敗");
      await showCustomModal(t("工單上傳失敗，請確認網路連線！"));
      return;
    }
    await showCustomModal(t("作業已完成！"));
  }

  resetAll();
}

// --- Window Control Function --- //
onMounted(async () => { connectWebSocket(); });
onUnmounted(() => { 
  stopHeartbeat();
  if (socket) socket.close();
})
</script>

<style>
/* 基礎樣式 */
html, body, #app { height: 100%; margin: 0; padding: 0; overflow: hidden; font-family: "Microsoft JhengHei", sans-serif; }
:root { --bg-color: #0f0c29; --panel-bg: rgba(21, 22, 46, 0.6); --panel-border: rgba(41, 255, 196, 0.3); --glow-color: rgba(41, 255, 196, 0.4); --text-color: #a9a9d4; --primary-color: #29ffc4; --success-color: #29ffc4; --error-color: #ff1b78; --label-color: #a9a9d4; --waiting-color: #6c757d; }
.scan-system-container { width: 100%; height: 100%; background: linear-gradient(to right, #24243e, #302b63, #0f0c29); color: var(--text-color); }
.hidden-input { position: absolute; top: -9999px; left: -9999px; }
.main-panel { display: flex; flex-direction: column; width: 100%; height: 100%; background: var(--panel-bg); border: 1px solid var(--panel-border); box-shadow: 0 0 30px var(--glow-color); backdrop-filter: blur(10px); padding: 2vh 2.5rem; box-sizing: border-box; }
.panel-header, .panel-footer, .info-display-section-compact { flex-shrink: 0; }
.system-logo { width: 200px; height: 30px; }
.panel-header { display: flex; justify-content: space-between; align-items: center; }
.header-left, .header-right { display: flex; align-items: center; gap: 1rem; }
.lang-select { padding: 5px; border-radius: 4px; background: rgba(0,0,0,0); color: var(--text-color); border: 1px solid var(--panel-border); font-size: 1rem; }

.zone-row { display: flex; align-items: center; gap: 1.5rem; width: 100%; margin-bottom: 2vh; }
.platform-indicator-container { width: 70px; flex-shrink: 0; display: flex; justify-content: center; align-items: center; }
.platform-indicator-container .process-zone { height: 30px; }

/* 越南文優化樣式：使用 Flex column 解決長文字跑版 */
.platform-label {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
    font-size: 1.2rem;
    font-weight: bold;
    padding: 10px 5px;
    border-radius: 8px;
    color: #000;
    box-shadow: 0 0 10px rgba(0,0,0,0.5);
    letter-spacing: 1px;
    min-height: 140px;
    width: 100%;
    transition: all 0.5s ease;
    background: rgba(255, 255, 255, 0.1);
    color: var(--text-color);
    border: 1px solid var(--panel-border);
    white-space: pre-wrap;
    word-break: break-word;
    line-height: 1.2;
}

.platform-upper { background-color: #FFD700 !important; box-shadow: 0 0 15px #FFD700; color: #000 !important; border: none; }
.platform-lower { background-color: #00BFFF !important; box-shadow: 0 0 15px #00BFFF; color: #000 !important; border: none; }
.zone-container { flex-grow: 1; position: relative; border: 1px solid var(--panel-border); border-radius: 12px; padding: 2vh 1.5rem 1.5vh 1.5rem; box-sizing: border-box; }
.staging-row { margin-top: 1vh; }

.info-display-section-compact { padding-top: 1.5vh; padding-bottom: 1vh; }
.panel-footer { padding-top: 1.5vh; border-top: 1px solid var(--panel-border); display: flex; justify-content: flex-end; gap: 1rem; }
.info-grid-compact { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; }
.info-label { font-size: 1.2rem; margin-bottom: 0.3rem; color: var(--label-color); }
.info-value { font-size: 1.8rem; font-weight: bold; color: var(--primary-color); padding: 0.4rem 0.75rem; background: rgba(0, 0, 0, 0.2); border-radius: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

.toggle-button-wrapper { flex-shrink: 0; padding: 1.5vh 0; display: flex; justify-content: space-between; align-items: center; gap: 2rem; }
.sheet-count-display { display: flex; align-items: center; gap: 0.8rem; }
.sheet-count-display .info-value { padding: 0; background: none; }

.btn-toggle { background: rgba(140, 100, 255, 0.7); color: var(--text-color); border: 1px solid var(--panel-border); backdrop-filter: blur(5px); }
.btn-toggle:hover { background: rgba(160, 120, 255, 0.9); color: #fff; }

.camera-section-main.adaptive-view { flex-grow: 1; min-height: 0; display: flex; justify-content: center; align-items: center; overflow: hidden; }
.main-content-wrapper { display: flex; flex-direction: column; gap: 1.5vh; width: 100%; max-width: 1400px; }
.perspective-container { perspective: 1500px; }
.zones-in-perspective { display: flex; flex-direction: column; transform: rotateX(20deg); transform-style: preserve-3d; }
.zone-title { position: absolute; top: 0; left: 2rem; transform: translateY(-50%); background: #2a284e; padding: 0 0.75rem; margin: 0; color: var(--label-color); font-weight: bold; display: flex; align-items: center; gap: 0.75rem; font-size: clamp(0.9rem, 2.2vw, 2.2rem); z-index: 1; }
.staging-mode-indicator { font-size: clamp(1.1rem, 2.2vw, 1.6rem); padding: 0.3em 0.8em; border-radius: 4px; font-weight: bold; }
.staging-mode-indicator.loading { background-color: var(--primary-color); color: #000; }
.staging-mode-indicator.unloading { background-color: #9b59b6; color: #fff; }
.zone-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; }

.camera-block { background: rgba(0, 0, 0, 0.3); border: 2px solid var(--panel-border); border-radius: 12px; display: flex; flex-direction: column; align-items: center; justify-content: center; transition: all 0.4s ease; padding: 1vh 1rem; }
.camera-block.staging { border-width: 4px; height: auto; min-height: 20vh; padding: 2vh 2rem; overflow-y: auto; }
.camera-block.status-success { border-color: var(--success-color); box-shadow: 0 0 10px var(--success-color); }
.camera-block.status-error { border-color: var(--error-color); box-shadow: 0 0 10px var(--error-color); }

.block-pdcode { word-break: break-all; font-weight: bold; color: var(--primary-color); font-size: clamp(1rem, 5.5vw, 2.5rem); font-family: 'Courier New', Courier, monospace; }
.block-pdcode.large { font-size: clamp(2rem, 5.5vw, 3rem); margin-bottom: 0.8rem; }
.camera-block.status-error .block-pdcode { color: var(--error-color); }
.block-label { color: var(--label-color); font-size: clamp(1rem, 1.5vw, 1.2rem); margin-bottom: 1.2rem; }
.status-text-success { color: var(--success-color); text-shadow: 0 0 8px rgba(41, 255, 196, 0.6); }
.status-text-error { color: var(--error-color); text-shadow: 0 0 8px rgba(255, 27, 120, 0.6); }

.status-display-area { flex-shrink: 0; display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; box-sizing: border-box; align-items: start; margin-top: 1vh; }
.status-list-column { background: rgba(0, 0, 0, 0.2); border: 1px solid var(--panel-border); border-radius: 8px; padding: 0.5vh; height: 15vh; display: flex; flex-direction: column; overflow-y: auto; }
.status-list { width: 100%; }

.history-item-wrapper { border-bottom: 1px solid rgba(41, 255, 196, 0.1); padding: 0.8vh 1rem; transition: background-color 0.3s; }
.history-item-wrapper:last-child { border-bottom: none; }
.history-item-wrapper.is-ng-item { background-color: rgba(255, 27, 120, 0.15); border-color: rgba(255, 27, 120, 0.2); }
.list-item-top-row { display: grid; grid-template-columns: auto 1fr auto auto; align-items: baseline; gap: 1.2rem; font-size: clamp(1.1rem, 1.2vw, 1.3rem); }
.item-action-type { font-weight: bold; font-family: 'Courier New', Courier, monospace; font-style: italic; font-size: 0.9em; color: var(--primary-color); }
.history-item-wrapper.is-ng-item .item-action-type { color: var(--error-color); }
.item-timestamp { color: var(--label-color); font-size: 0.9em; }
.item-pdcode { font-weight: bold; color: var(--primary-color); font-family: 'Courier New', Courier, monospace; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.history-item-wrapper.is-ng-item .item-pdcode { color: var(--error-color); }
.item-status-text { font-weight: bold; font-size: 1.1em; }
.item-status-text.status-ok { color: var(--success-color); }
.item-status-text.status-ng { color: var(--error-color); }
.list-item-ng-details { display: grid; grid-template-columns: 1fr 1fr; gap: 0.5rem 1rem; padding-top: 0.5rem; padding-left: 1rem; font-size: 0.85rem; color: var(--label-color); }
.list-item-ng-details span { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

.initial-prompt { display: flex; justify-content: center; align-items: center; flex-direction: column; gap: 1.5rem; width: 100%; height: 100%; font-size: 1.8rem; color: var(--waiting-color); }
.prompt-icon { width: 60px; height: 60px; fill: currentColor; }
.system-status { display: flex; align-items: center; gap: 0.5rem; color: var(--success-color); }
.system-status.is-offline { color: var(--error-color); animation: pulse-offline 2s infinite; }
.status-light { width: 10px; height: 10px; background-color: var(--success-color); border-radius: 50%; }
.status-light { width: 10px;  height: 10px;  background-color: var(--success-color);  border-radius: 50%; box-shadow: 0 0 5px var(--success-color); transition: all 0.3s ease; }
.status-light.light-offline { background-color: var(--error-color); box-shadow: 0 0 8px var(--error-color); }
@keyframes pulse-offline { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }
.btn { padding: 0.7rem 1.8rem; font-size: 1rem; border-radius: 8px; cursor: pointer; border: none; transition: all 0.2s ease; font-weight: bold; }
.btn-resetuser, .btn-reset { background: transparent; color: var(--text-color); border: 2px solid var(--panel-border); }
.btn-resetuser:hover, .btn-reset:hover { background-color: rgba(255,255,255,0.1); }
.btn-complete { background: var(--primary-color); color: #000; }
.btn-complete:hover { box-shadow: 0 0 10px var(--primary-color); }
.btn:disabled { background: var(--waiting-color); color: #333; cursor: not-allowed; opacity: 0.6; }

.custom-modal { border: 1px solid var(--panel-border); background: var(--panel-bg); box-shadow: 0 0 40px var(--glow-color); backdrop-filter: blur(15px); color: var(--text-color); border-radius: 12px; padding: 2.5rem 3.5rem; width: clamp(300px, 60vw, 700px); max-width: 90%; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 1000; }
.custom-modal::backdrop { background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(8px); }
.modal-content { display: flex; flex-direction: column; gap: 2rem; align-items: center; }
.modal-message { font-size: clamp(1.5rem, 3.5vw, 2.5rem); font-weight: bold; text-align: center; line-height: 1.4; color: var(--primary-color); text-shadow: 0 0 10px rgba(41, 255, 196, 0.5); }
.modal-actions { display: flex; gap: 1.5rem; justify-content: center; width: 100%; }
.btn-modal-confirm { background: var(--primary-color); color: #000; padding: 0.9rem 2.5rem; font-size: clamp(1.1rem, 2vw, 1.4rem); min-width: 120px; }
.btn-modal-confirm:hover { box-shadow: 0 0 15px var(--primary-color); }
.btn-modal-cancel { background: transparent; color: var(--text-color); border: 2px solid var(--panel-border); padding: 0.9rem 2.5rem; font-size: clamp(1.1rem, 2vw, 1.4rem); min-width: 120px; }
.btn-modal-cancel:hover { background-color: rgba(255,255,255,0.1); }

.zone-row .platform-indicator-container { align-self: center; width: 80px; }
.zone-row:first-child .platform-label { min-height: 80px; padding: 5px 5px; font-size: 1.1rem; }
</style>