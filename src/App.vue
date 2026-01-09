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
          <div class="info-display-group action-group">
            <span class="info-label">{{ t('工單') }}：</span><span class="info-value">{{ info.workOrder || "---" }}</span>
            <button v-if="info.workOrder" class="btn-mini-danger" @click.stop="completeAllScanned" @mousedown.prevent title="強制結單">{{ t('強制結單') }}</button>
          </div>
          <div class="info-display-group"><span class="info-label">{{ t('品目') }}：</span><span class="info-value">{{ info.productItem || "---" }}</span></div>
        </div>
      </section>

       <div class="toggle-button-wrapper">
        <div class="sheet-count-display">
          <span class="info-label">{{ t('總張數') }}：</span><span class="info-value">{{ info.panel_num || 0 }}</span>
          <span class="info-label">{{ t('投入張數') }}：</span><span class="info-value">{{ info.OK_num || 0 }}</span>
           <span class="info-label" style="color: #ff1b78; margin-left: 10px;">NG：</span><span class="info-value" style="color: #ff1b78;">{{ info.NG_num }}</span>
        </div>
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
                        <span class="block-panel">{{ inProcessPlatform.left?.panel || "---" }}</span>
                      </div>
                      <div class="camera-block" :class="`status-${getAreaStatus(inProcessPlatform.right)}`">
                        <span class="block-panel">{{ inProcessPlatform.right?.panel || "---" }}</span>
                      </div>
                    </div>
                  </div>
                  <div class="side-action-spacer"></div>
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
                        <span v-if="get2DIDStatusShow(onloadingPlatform.left?.ret_type)" class="block-panel large" :class="`status-text-${getAreaStatus(onloadingPlatform.left)}`">{{ onloadingPlatform.left?.ret_type }}</span>
                        <span class="block-panel large">{{ onloadingPlatform.left?.panel || t("等待掃描...") }}</span>
                      </div>
                      
                      <div class="camera-block staging" :class="`status-${getAreaStatus(onloadingPlatform.right)}`">
                        <span class="block-label">{{ t('準備區-右') }}</span>
                        <span v-if="get2DIDStatusShow(onloadingPlatform.right?.ret_type)" class="block-panel large" :class="`status-text-${getAreaStatus(onloadingPlatform.right)}`">{{ onloadingPlatform.right?.ret_type }}</span>
                        <span class="block-panel large">{{ onloadingPlatform.right?.panel || t("等待掃描...") }}</span>
                      </div>
                    </div>
                  </div>
                  <div class="side-action-container">
                    <button class="btn-side-danger" @click.stop="forceExecute" @mousedown.prevent>
                      {{ t('強制執行') }}
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div class="status-display-area">
              <div class="status-list-column">
                <div class="status-list">
                  <div v-for="item in side_history_2DID_list.left.slice(0, 5)" :key="`${item.panel}-${item.timestamp}`" class="history-item-wrapper" :class="{ 'is-ng-item': item.ret_type === 'NG' }">
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ formatTime(item.timestamp) }}</span>
                      <span class="item-panel">{{ item.panel }}</span>
                      <span class="item-action-type">({{ t(get2DIDActionType(item.panel)) }})</span> 
                      <span class="item-status-text" :class="item.ret_type === 'OK' ? 'status-ok' : 'status-ng'">{{ item.ret_type }}</span>
                    </div>
                    <div v-if="item.ret_type === 'NG'" class="list-item-ng-details">
                      <span :title="item.detail">{{ t('原因') }}: {{ t(item.detail) }}</span>
                      <span v-if="item.workStepName" :title="item.workOrder">{{ t('工單') }}: {{ item.workOrder }}</span>
                      <span v-if="item.workStepName" :title="item.item">{{ t('品目') }}: {{ item.item }}</span>
                      <span v-if="item.workStepName" :title="item.workStepName">{{ t('工站') }}: {{ item.workStepName }}</span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="status-list-column">
                <div class="status-list">
                  <div v-for="item in side_history_2DID_list.right.slice(0, 5)" :key="`${item.panel}-${item.timestamp}`" class="history-item-wrapper" :class="{ 'is-ng-item': item.ret_type === 'NG' }">
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ formatTime(item.timestamp) }}</span>
                      <span class="item-panel">{{ item.panel }}</span>
                      <span class="item-action-type">({{ t(get2DIDActionType(item.panel)) }})</span> 
                      <span class="item-status-text" :class="item.ret_type === 'OK' ? 'status-ok' : 'status-ng'">{{ item.ret_type }}</span>
                    </div>
                    <div v-if="item.ret_type === 'NG'" class="list-item-ng-details">
                      <span :title="item.detail">{{ t('原因') }}: {{ t(item.detail) }}</span>
                      <span v-if="item.workStepName" :title="item.workOrder">{{ t('工單') }}: {{ item.workOrder }}</span>
                      <span v-if="item.workStepName" :title="item.item">{{ t('品目') }}: {{ item.item }}</span>
                      <span v-if="item.workStepName" :title="item.workStepName">{{ t('工站') }}: {{ item.workStepName }}</span>
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
        <button class="btn btn-resetuser" @click.stop="employeeClear" @mousedown.prevent>{{ t('換人生產') }}</button>
        <!-- <button class="btn btn-reset" @click.stop="resetAll">{{ t('重整') }} (Reset)</button> -->
        <!-- <button class="btn btn-execute" @click.stop="forceExecute" @mousedown.prevent>{{ t('強制執行') }}</button> -->
        <button class="btn btn-complete" @click.stop="completeWorkorder" @mousedown.prevent :disabled="(info.panel_num == 0) || (info.NG_num + info.NG_num != info.panel_num)">{{ t('完成') }} (Complete)</button>
      </footer>
   </div>
  </div>

  <div v-if="Modal" class="window-wrapper">
    <div class="custom-modal">
      <div class="modal-content">
        <p class="modal-message">{{ modalMessage }}</p>
        <div class="modal-actions">
          <button class="btn btn-modal-confirm" @click="Modal = false">{{ t('確定') }}</button>
        </div>
      </div>
    </div>
  </div>

  <div v-if="isLoading" class="loading-overlay">
    <div class="loading-content">
      <div class="spinner"></div>
      <p>{{ t(loadingMessage) }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, watch, reactive } from 'vue';
import { i18nDict } from "@/textManager/translate"
import logo from "@/components/icons/flexium_logo.png"

const OIS_API_BASE = "http://10.8.32.64:2151";
const wsUrl = 'ws://127.0.0.1:8181'
const headers = { 'Content-Type': 'application/json' };

const currentLang = ref('zh'); 
const t = (key) => {
  if (!key) return key;
  if (currentLang.value === 'zh') return key;
  return i18nDict[key]?.vn || key;
};

// --- Loading State ---
const isLoading = ref(false);
const loadingMessage = ref("系統處理中..."); // 可動態改變文字

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
const emptySlot = () => ({ panel: "", ret_type: "", detail: "" });
let twodid_panel_mapping = reactive({});
let panel_twodid_mapping = reactive({});
let expected_2DID_dict = reactive({});  // Expected 2DID dict to scaned on this workorder
let scanned_2DID_dict = reactive({});  // Scanned 2DID dict on this workorder
let other_2DID_dict = reactive({});  // Reported error 2DID dict
let side_history_2DID_dict = reactive({left: {}, right: {}});  // Save all history scanned 2DID (for save 2DID information)
let side_history_2DID_list = reactive({left: [], right: []});  // List all history scanned 2DID (filter short period)
let history_2DID_dict = reactive({});  // Save all history scanned 2DID dict
let up_platform_2DID = reactive({right: emptySlot(), left: emptySlot()});
let dn_platform_2DID = reactive({right: emptySlot(), left: emptySlot()});
let last_OK_upload_2DID = reactive({up: {left: {panel: "", detect: false}, right: {panel: "", detect: false}}, dn: {left: {panel: "", detect: false}, right: {panel: "", detect: false}}});

// --- Offline & Failover State --- //
const isBackendOnline = ref(false);
let backendHeartbeatTimer = null;
const offlineRequestQueue = reactive([]);

// Display state
const isBaseInfoFilled = computed(() => !!(info.employeeId && info.workOrder && info.employeeName && info.productItem));
const formatTime = (ms) => new Date(ms).toLocaleTimeString();
const _in_platform = computed(() => { return (PlatformState.up_in == 1) ? "上台面" : "下台面"; })
const _out_platform = computed(() => { return (_in_platform.value === "上台面") ? "下台面" : "上台面"; })
const getPlatformClass = (text) => {
    if (text === "上台面") return "platform-upper";
    if (text === "下台面") return "platform-lower";
    return "";
};
const inProcessPlatform = computed(() => { return (PlatformState.up_in == 1) ? up_platform_2DID : dn_platform_2DID; })
const onloadingPlatform = computed(() => { return (PlatformState.up_in != 1) ? up_platform_2DID : dn_platform_2DID; })
const get2DIDActionType = (panel) => (scanned_2DID_dict[panel] && scanned_2DID_dict[panel].ret_type == "OK") ? "出" : "進";
const getAreaStatus = (area) => {
  const rt = area?.ret_type || "";
  if (rt === "OK") return "success";
  if (rt === "NG") return "error";
  return "waiting";
};
const get2DIDStatusShow = (status) => { return (status && status.length > 0) ? true : false; };

// --- Websocket Process --- ///
let reconnectDelay = 5000; // <--- 補上這行
let reconnectTimer = null; // <--- 補上這行
let socket = null;
function connectWebSocket() {
  if (socket) { return; }
  
  socket = new WebSocket(wsUrl);
  const timeoutMs = 1000;
  const timer = setTimeout(() => {
    if (socket && socket.readyState === WebSocket.CONNECTING) socket.close(4000, "connect timeout");
  }, timeoutMs);
  
  socket.onopen = () => {
    clearTimeout(timer);
    startBackendHeartbeat();
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
        // Process infomation input from keyboard
        else if (message.source === "SCANNER") { handleInfoInput(message.payload); }
        // Process camera input
        else if (message.source.includes("CAMERA")) { processControl(message.payload, message.source); }
        // Receive offline API command
        else if (message.source === "SYS" && message.payload && message.payload.type === "OFFLINE_CACHE_LOADED") {
          console.log("Process OFFLINE_CACHE_LOADED");
          const loadedData = message.payload.data;
          if (Array.isArray(loadedData) && loadedData.length > 0) {
            offlineRequestQueue.splice(0, offlineRequestQueue.length, ...loadedData);
            startBackendHeartbeat();
          }
        }
      }
    }
    catch (error) { console.error("WebSocket 解析訊息失敗: ", error); }
  }

  socket.onclose = () => {
    console.log(`WebSocket: 連接已關閉，${reconnectDelay / 1000}秒後嘗試重連...`);
    socket = null;

    // 清空狀態
    PlatformState.up_in = -1;
    PlatformState.dn_in = -1;
    PlatformState.start_message = -1;

    if (reconnectTimer) clearTimeout(reconnectTimer);
    reconnectTimer = setTimeout(() => { connectWebSocket(); }, reconnectDelay);
  };

  socket.onerror = (error) => { console.error("WebSocket 發生錯誤: ", error); }
}

function sendSocketMessage(command, payload) {
  if (socket && socket.readyState === WebSocket.OPEN) {
    let message = { type: "control", command, payload};
    socket.send(JSON.stringify(message));
  }
}

// ---- wrappers ----
const Ajax = async (url, options, time, backgroundLoading = true) => {
  if (backgroundLoading) isLoading.value = true;
  loadingMessage.value = "系統處理中...";

  const controller = new AbortController();
  const timeoutTime = (isBackendOnline.value) ? time : 500;
  const timeoutId = setTimeout(() => { controller.abort(); }, timeoutTime);
  let config = { ...options, signal: controller.signal };

  try {
    let response = await fetch(url, config);
    clearTimeout(timeoutId);

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
  }
  // Process offline API command storage
  catch (error) {
    clearTimeout(timeoutId);

    // Switch to offline mode
    if (isBackendOnline.value) {
      isBackendOnline.value = false;
      startBackendHeartbeat(); 
      showCustomModal(t("與後台連線中斷，已切換至本地儲存模式"), "alert");
    }

    const isWriteApi = url.includes('/write2did') || url.includes('/write2dids');
    const isDeleteApi = url.includes('/Delete_2DID');

    if (isWriteApi || isDeleteApi) {
      console.warn(`[Offline Mode] Backend unreachable. Queuing request for: ${url}`);
      
      const payload = JSON.parse(options.body);
      const apiType = url.substring(url.lastIndexOf('/') + 1);
      const offlineRecord = { api: apiType, body: payload, timestamp: Date.now() };

      offlineRequestQueue.push(offlineRecord);
      sendSocketMessage("APPEND_OFFLINE_CACHE", offlineRecord);

      return { success: true, offline: true };
    }
    throw error;
  }
  finally { isLoading.value = false; }
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
      console.log("send heatbeat...");
      const res = await Ajax(`${OIS_API_BASE}/heartbeat`, { method: 'GET', signal: controller.signal }, false);
      console.log("res: ", res);

      if (res) {
        isBackendOnline.value = true;

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
    if (req.api === 'write2did') uploadBatch.push(req.body);
    else if (req.api === 'write2dids') uploadBatch.push(...req.body);
    else if (req.api === 'Delete_2DID') deleteRequests.push(req.body);
  });

  try {
    // 2. 處理批量上傳
    if (uploadBatch.length > 0) {
      const res = await Ajax(`${OIS_API_BASE}/api/write2dids`, { method: "POST", headers, body: JSON.stringify(uploadBatch) }, 3000);
      if (!res.success) throw new Error("Batch upload failed");
      console.log("Batch upload success.");
    }

    // 3. 處理刪除請求
    for (const delReq of deleteRequests) await Ajax(`${OIS_API_BASE}/api/Delete_2DID`, { method: "POST", headers, body: JSON.stringify({ delReq }) }, 1500);

    offlineRequestQueue.splice(0, currentBatch.length);
    if (offlineRequestQueue.length === 0) sendSocketMessage("CLEAR_OFFLINE_CACHE", {});
    else processOfflineQueue();
  } catch (error) {
    console.error("Failed to sync offline queue:", error);
    startBackendHeartbeat();
  }
}

// --- Custom Modal --- //
const Modal = ref(false);
const modalMessage = ref('');
const showCustomModal = (message) => {
  modalMessage.value = message;
  Modal.value = true;
};

// --- API Request --- //
async function validateEmpId(empId) {
  try {
    const response = await Ajax(`${OIS_API_BASE}/api/validate_emp`, { method: "POST", headers, body: JSON.stringify({ empId }) }, 1500);
    console.log(response);
    if (response.code === 200 && response.data) return { ...response.data };
    else {
      showCustomModal(t('工號驗證失敗'));
      return false;
    }
  } catch(error) {
    console.error("validateEmpId 發生錯誤:", error);
    showCustomModal(t('工號驗證時發生網路錯誤：請確認網路連線'));
    return false;
  }
}

async function validateWorkOrder(workOrder, insert_to_database = false) {
  try {
    const res = await Ajax(`${OIS_API_BASE}/api/workorder`, { method: 'POST', headers, body: JSON.stringify({ emp_no: info.employeeId, workorder: workOrder, insert_to_database }) }, 3000);
    if (res.success && res.data) {
      const d = res.data;
      return { cmd236_flag: d.cmd236_flag, workorder: d.workorder, item: d.item, workStep: d.workStep, panel_num: d.panel_num, sht_no: d.sht_no, panel_no: d.panel_no, twodid_step: d.twodid_step, twodid_type: d.twodid_type, scanned_data: d.scanned_data };
    }

    if (res.success == false && res.type == "mes_offline") {
      showCustomModal(t('因與 IT server 網路中斷，因此工單查詢失敗'));
      return null;
    }

    showCustomModal(t('工單驗證失敗：查無資料'));
    return null;
  }
  catch (error) {
    console.error("validateWorkOrder 發生錯誤: ", error)
    showCustomModal(t('工單驗證時發生網路錯誤，請確認網路連線'));
    return null;
  }
}

async function validate2DID(panel_id) {
  if (!panel_id) return { ret_type: "NG", workOrder: "N/A", item: "N/A", detail: "條碼異常" };

  const panel_info = expected_2DID_dict[panel_id];
  if (panel_info) {
    if (panel_info.twodid_type == "Y") {
      await sheetReport(panel_id, "NG", "跳站");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "跳站" };
    }
    else if (panel_info.twodid_step === info.workStep) {
      await sheetReport(panel_id, "NG", "重工", false);
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
    }

    const scanned_record = scanned_2DID_dict[panel_id];
    if (scanned_record && Date.now() - scanned_record.timestamp > unloadPeriodThreshold) {
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
    }
    else if (scanned_record) {
      return { ret_type: "OK", workOrder: info.workOrder, item: info.productItem, detail: "已完成" };
    }
    return { ret_type: "OK", workOrder: info.workOrder, item: info.productItem, detail: "" };
  }

  // This 2DID has detect in the history and reported error to server, just skip
  const twodid = panel_id;
  if (other_2DID_dict[twodid]) return other_2DID_dict[twodid];

  let detail = "";
  // This 2DID has not detect in the history requested need to get the information from server
  try {
    const res_json = await Ajax(`${OIS_API_BASE}/api/twodid`, { method: 'POST', headers, body: JSON.stringify({ emp_no : info.employeeId, twodid }) }, 1000);
    if (res_json.success) {
      const [res_ok, workOrder, productItem, workStep] = res_json.result.result.split(";");

      if (res_ok == "OK") {
        if (productItem != info.productItem) detail += "混品目;";
        else if (workOrder != info.workOrder) detail += "混工單;";

        other_2DID_dict[twodid] = { ret_type: "NG", workStepName: workStep, workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };

        return { ret_type: "NG", workStepName: workStep, workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };
      }
    }
    if (res.success == false && res.type == "mes_offline") showCustomModal(t('因與 IT server 網路終端，因此 2DID 資訊查詢失敗'));

    other_2DID_dict[twodid] = { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
    return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
  }
  catch (error) { showCustomModal(t("驗證2DID發生錯誤，請確認網路連線")); }

  return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
}

// --- Control Process --- //
// Chage stage
function changeStage(s) {
  stage.value = s;
  sendSocketMessage("STEP_UPDATE", s);
}

const clearObj = (obj) => { Object.keys(obj).forEach(key => delete obj[key]); };
// Reset alls storage data
function resetAll() {
  clearObj(twodid_panel_mapping);
  clearObj(panel_twodid_mapping);
  clearObj(expected_2DID_dict);
  clearObj(scanned_2DID_dict);
  clearObj(other_2DID_dict);
  clearObj(history_2DID_dict);
  clearObj(side_history_2DID_dict.left);
  clearObj(side_history_2DID_dict.right);

  side_history_2DID_list.left.length = 0;
  side_history_2DID_list.right.length = 0;
  Object.assign(up_platform_2DID, {right: emptySlot(), left: emptySlot()});
  Object.assign(dn_platform_2DID, {right: emptySlot(), left: emptySlot()});
  Object.assign(info, { employeeId: "", employeeName: "", workOrder: "", productItem: "", workStep: "", panel_num: 0, OK_num: 0, NG_num: 0, cmd236_flag: false });
  Object.assign(forceCommand, { command: false, triggerTime: null });

  changeStage("empId");
}

// Storage history request and list expected 2DID in target work order
function create2DIDDict(panelList) {
  const sht_no_list = panelList.sht_no;
  const panel_no_list = panelList.panel_no;
  const twodid_step_list = panelList.twodid_step;
  const twodid_type_list = panelList.twodid_type;
  info.cmd236_flag = panelList.cmd236_flag;

  clearObj(twodid_panel_mapping);
  clearObj(panel_twodid_mapping);
  clearObj(expected_2DID_dict);
  clearObj(scanned_2DID_dict);
  clearObj(other_2DID_dict);

  for (let index = 0; index < panelList.sht_no.length; index++) {
    twodid_panel_mapping[sht_no_list[index]] = panel_no_list[index];
    if (!panel_twodid_mapping[panel_no_list[index]]) panel_twodid_mapping[panel_no_list[index]] = [];
    panel_twodid_mapping[panel_no_list[index]].push(sht_no_list[index]);

    if (!expected_2DID_dict[panel_no_list[index]]) expected_2DID_dict[panel_no_list[index]] = { twodid_step: twodid_step_list[index], twodid_type: twodid_type_list[index] };
  }

  if (Array.isArray(panelList.scanned_data) && panelList.scanned_data.length > 0) {
    for (const sd of panelList.scanned_data) {
      if (!scanned_2DID_dict[sd.panel_no]){
        scanned_2DID_dict[sd.panel_no] = { timestamp: sd.timestamp, ret_type: sd.twodid_type, workOrder: info.workOrder, item: info.productItem, detail: sd.twodid_status };
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
    const result = await validateEmpId(value);
    // const result = { empNo: "12868", empName: "王巨成" };

    if (!result) return;

    info.employeeId = result.empNo;
    info.employeeName = result.empName;
    changeStage((info.workOrder === "") ? "workOrder" : "twoDID");
  }

  else if (stage.value === "workOrder") {
    if (value.length < 8 || value.length > 11) {
      showCustomModal(t("工單格式不正確，請重新掃描。"));
      return;
    }

    const valid = await validateWorkOrder(value, true);
    console.log("workorder: ", valid);
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
function detectTargetPlatform() { return (PlatformState.up_in != 1) ? up_platform_2DID : dn_platform_2DID; }
function detectOtherPlatform() { return (PlatformState.up_in == 1) ? up_platform_2DID : dn_platform_2DID; }

// Process 2DID control
const processControl = async (data, source) => {
  // 1. Check value valid
  // 2. Get loading platform
  // 3. Process 2DID TIMEOUT signal
  // 4. Get product 2DID information
  // 5. Record 2DID information to history dictionary
  // 6. Refresh target platform product information
  // 7. Send M86, M87 status to local python
  const value = (twodid_panel_mapping[data]) ? twodid_panel_mapping[data] : data;
  if (value.length !== 13) {
    showCustomModal(t("產品條碼長度不正確，請重新掃描。"));
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
    if (!lastArea2DID || Date.now() - lastArea2DID.timestamp > 1500) Object.assign(targetPlatformArea, { panel: "等待掃描...", ret_type: "", detail: "" });

    // Detect force information
    if (forceCommand.command === true && (forceCommand.triggerTime && Date.now() - forceCommand.triggerTime < forceTriggerPeriod) && (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK")) {
      sendSocketMessage("GO_NOGO", 1);
    }
    else sendSocketMessage("GO_NOGO", 0);
    return;
  }

  // Check product validate
  let targetPanel = (source.startsWith("CAMERA_RIGHT") || source.startsWith("CAMERA_LEFT")) ? value : null;
  let validResult = await validate2DID(targetPanel);
  let rec = { timestamp: Date.now(), workStepName: validResult.workStepName, ret_type: validResult.ret_type, workOrder: validResult.workOrder, item: validResult.item, detail: validResult.detail };

  const two_camera_pass = rec.ret_type == "OK" && ((targetPlatform.left.ret_type == 'OK' && source.startsWith("CAMERA_RIGHT")) || (targetPlatform.right.ret_type && source.startsWith("CAMERA_LEFT")))
  if (info.OK_num === info.panel_num || ((info.OK_num == info.panel_num - 1) && two_camera_pass)) {
    if (rec.ret_type == "OK") Object.assign(rec, { ...rec, ret_type: "NG", detail: "超過 PANEL 上限" });
    showCustomModal(t('已達 PANEL 數上限，因此無法綁定。'));
  }

  // Filtered short period repeated 2DID for frontend log
  const sheet_history = (source.startsWith("CAMERA_LEFT")) ? side_history_2DID_dict.left[value] : side_history_2DID_dict.right[value];
  const last_info = (source.startsWith("CAMERA_LEFT")) ? side_history_2DID_list.left[0] : side_history_2DID_list.right[0];
  if (sheet_history && (last_info && (last_info.panel == value)) && (Date.now() - sheet_history.timestamp < shortPeriodFilterThreshold)) {
    sheet_history.timestamp = Date.now();
  }
  else {
    if (source.startsWith("CAMERA_LEFT")) { 
      side_history_2DID_list.left.unshift({ ...rec, panel: value });
      side_history_2DID_dict.left[value] = { ...rec };
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      side_history_2DID_list.right.unshift({ ...rec, panel: value });
      side_history_2DID_dict.right[value] = { ...rec };
    }
  }

  // Filtered short period repeated 2DID to record
  if (history_2DID_dict[value]) history_2DID_dict[value].timestamp = Date.now();
  else history_2DID_dict[value] = { ...rec };

  // Check 2DID is processed 2DID
  if (value == targetPlatformArea.panel || value == otherPlatformArea.panel) {
    const upPlatformUpload2DIDArea = (source.startsWith("CAMERA_LEFT")) ? last_OK_upload_2DID.up.left : last_OK_upload_2DID.up.right;
    const dnPlatformUpload2DIDArea = (source.startsWith("CAMERA_RIGHT")) ? last_OK_upload_2DID.dn.left : last_OK_upload_2DID.dn.right;
    if (value == upPlatformUpload2DIDArea.panel) upPlatformUpload2DIDArea.detect = true;
    else if (value == dnPlatformUpload2DIDArea.panel) dnPlatformUpload2DIDArea.detect = true;
  }
  // Filled the information for frontend
  else if (value != otherPlatformArea.panel) Object.assign(targetPlatformArea, { panel: value, ret_type: validResult.ret_type, detail: validResult.detail });

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
  else sendSocketMessage("GO_NOGO", 0);
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
  if (platform.left.ret_type == "OK" && !scanned_2DID_dict[platform.left.panel]) Object.assign(targetPlatformUpload2DID.left, {panel: platform.left.panel, detect: false});
  if (platform.right.ret_type == "OK" && !scanned_2DID_dict[platform.right.panel]) Object.assign(targetPlatformUpload2DID.right, {panel: platform.right.panel, detect: false});
}

async function lastOKUploadSheetDetect(info) {
  let targetPlatformUpload2DID = (info == "up") ? last_OK_upload_2DID.up : last_OK_upload_2DID.dn;
  if ((targetPlatformUpload2DID.left.panel != "" && targetPlatformUpload2DID.left.detect == false) || (targetPlatformUpload2DID.right.panel != "" && targetPlatformUpload2DID.right.detect == false)) {
    showCustomModal(t("進出料製品 2DID 不同，請確認機台是否操作正常"));
  }

  if (history_2DID_dict[targetPlatformUpload2DID.left.panel]) {
    history_2DID_dict[targetPlatformUpload2DID.left.panel].timestamp = Date.now();
    sheetReport(targetPlatformUpload2DID.left.panel, "OK", "OK");
  }
  if (history_2DID_dict[targetPlatformUpload2DID.right.panel]) {
    history_2DID_dict[targetPlatformUpload2DID.right.panel].timestamp = Date.now();
    sheetReport(targetPlatformUpload2DID.right.panel, "OK", "OK");
  }
  
  Object.assign(targetPlatformUpload2DID, {left: {panel: "", detect: false}, right: {panel: "", detect: false}});
}

const sheetReport = async(panel_no, type, detail, report = true) => {
  if (panel_no && (panel_no.length == 0)) return;

  if (expected_2DID_dict[panel_no] && !scanned_2DID_dict[panel_no]) {
    scanned_2DID_dict[panel_no] = { timestamp: Date.now(), ret_type: type, workOrder: info.workOrder, item: info.productItem, detail };

    if (report){
      try { await uploadPanel(panel_no, type, detail);} 
      catch (error) { showCustomModal(t(`製品 ${type} 狀態上傳失敗，請確認網路連線！`)); }
    }

    if (type == "OK") info.OK_num += 1;
    else if (type == "NG") info.NG_num += 1;
  }
  else if (!expected_2DID_dict[panel_no] && !other_2DID_dict[panel_no]){
    try { await uploadPanel(panel_no, type, detail); } 
    catch (error) { showCustomModal(t(`製品 ${type} 狀態上傳失敗，請確認網路連線！`)); }
  }
}

const uploadPanel = async (panel_no, ret_type, status) => {
  const base = { emp_no: info.employeeId, workOrder: info.workOrder, workStep: info.workStep, item: info.productItem, twodid_type: ret_type, remark: `${status}-IPC` };
  const payloads = panel_twodid_mapping[panel_no].map(sheet_no => ({ ...base, sht_no: sheet_no, panel_no }));
  await Ajax(`${OIS_API_BASE}/api/write2dids`, { method: "POST", headers, body: JSON.stringify(payloads) }, 200);
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
    showCustomModal(t("請等待平台退回上料區"));
    return;
  }

  if (info.OK_num == info.panel_num || ((info.OK_num == info.panel_num - 1) && targetPlatform.left.ret_type === "OK" && targetPlatform.right.ret_type === "OK")) {
    showCustomModal(t("作業後將達 PANEL 數上限，因此無法繼續執行。"));
    return;
  }

  if (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK") {
    forceCommand.command = true;
    forceCommand.triggerTime = Date.now();
    sendSocketMessage("GO_NOGO", 1);
    return;
  }

  showCustomModal(t("請放入至少放入一個OK製品"));
}

const completeAllScanned = async () => {
  if ((up_platform_2DID.left?.ret_type != "" || up_platform_2DID.right?.ret_type != "" || dn_platform_2DID.left.ret_type != "" || dn_platform_2DID.right.ret_type != "")) {
    showCustomModal(t("請保持平台無任何製品"));
    return;
  }

  if (info.cmd236_flag == false){
    try {
      const base = { emp_no: info.employeeId, workOrder: info.workOrder, workStep: info.workStep, item: info.productItem, twodid_type: "NG", remark: "未投入，作業結束-IPC" }
      let notScanned_2DID_list = Object.entries(expected_2DID_dict).filter(([panel]) => !scanned_2DID_dict?.[panel]).flatMap(([panel]) => panel_twodid_mapping[panel].map(sheet_no => ({ ...base, sht_no: sheet_no, panel_no: panel })));
      
      await Ajax(`${OIS_API_BASE}/api/write2dids`, { method: "POST", headers, body: JSON.stringify(notScanned_2DID_list) }, 3000);
    }
    catch (error) {
      showCustomModal(t("工單上傳失敗，請確認網路連線！"));
      return;
    }
  }

  await Ajax(`${OIS_API_BASE}/api/Delete_2DID`, { method: "POST", headers, body: JSON.stringify({ workorder: info.workOrder }) }, 1500);
  showCustomModal(t("作業已完成！"));
  resetAll();
}

const completeWorkorder = async () => {
  await Ajax(`${OIS_API_BASE}/api/Delete_2DID`, { method: "POST", headers, body: JSON.stringify({ workorder: info.workOrder }) }, 1500);
  showCustomModal(t("作業已完成！"));
  resetAll();
}

// --- Window Control Function --- //
onMounted(async () => {
  loadingMessage.value = "正在與後台建立連線，請稍後...";
  isLoading.value = true;
  connectWebSocket(); 
});
onUnmounted(() => {
  if (socket) socket.close();
})
</script>

<style>
/* ... (Previous styles remain the same) ... */
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

/* ... (Zone and platform styles) ... */
.zone-row { display: flex; align-items: center; gap: 1.5rem; width: 100%; margin-bottom: 2vh; }
.platform-indicator-container { width: 70px; flex-shrink: 0; display: flex; justify-content: center; align-items: center; }
.platform-indicator-container .process-zone { height: 30px; }
.platform-label { display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; font-size: 1.2rem; font-weight: bold; padding: 10px 5px; border-radius: 8px; color: #000; box-shadow: 0 0 10px rgba(0,0,0,0.5); letter-spacing: 1px; min-height: 140px; width: 100%; transition: all 0.5s ease; background: rgba(255, 255, 255, 0.1); color: var(--text-color); border: 1px solid var(--panel-border); white-space: pre-wrap; word-break: break-word; line-height: 1.2; }
.platform-upper { background-color: #FFD700 !important; box-shadow: 0 0 15px #FFD700; color: #000 !important; border: none; }
.platform-lower { background-color: #00BFFF !important; box-shadow: 0 0 15px #00BFFF; color: #000 !important; border: none; }
.zone-container { width: 1500px; position: relative; border: 1px solid var(--panel-border); border-radius: 12px; padding: 2vh 1.5rem 1.5vh 1.5rem; box-sizing: border-box; }
.staging-row { margin-top: 1vh; }

/* [New] Spacer for layout balance */
.side-action-spacer { width: 80px; flex-shrink: 0; }
/* [New] Container for the side action button (Force Execute) */
.side-action-container { width: 80px; flex-shrink: 0; display: flex; justify-content: center; align-items: center; }

/* ... (Info display styles) ... */
.info-display-section-compact { padding-top: 1.5vh; padding-bottom: 1vh; }
.panel-footer { padding-top: 1.5vh; border-top: 1px solid var(--panel-border); display: flex; justify-content: flex-end; gap: 1rem; }
.info-grid-compact { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 1rem; }
.info-label { font-size: 1.2rem; margin-bottom: 0.3rem; color: var(--label-color); }
.info-value { font-size: 1.8rem; font-weight: bold; color: var(--primary-color); padding: 0.4rem 0.75rem; background: rgba(0, 0, 0, 0.2); border-radius: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* ... (Toggle button and sheet count) ... */
.toggle-button-wrapper { flex-shrink: 0; padding: 1.5vh 0; display: flex; justify-content: space-between; align-items: center; gap: 2rem; }
.sheet-count-display { display: flex; align-items: center; gap: 0.8rem; }
.sheet-count-display .info-value { padding: 0; background: none; }

/* ... (Main content layout) ... */
.camera-section-main.adaptive-view { flex-grow: 1; min-height: 0; display: flex; justify-content: center; align-items: center; overflow: hidden; }
.main-content-wrapper { display: flex; flex-direction: column; gap: 1.5vh; width: 100%; max-width: 1400px; }
.perspective-container { perspective: 1500px; }
.zones-in-perspective { display: flex; flex-direction: column; transform: rotateX(20deg); transform-style: preserve-3d; }
.zone-title { position: absolute; top: 0; left: 2rem; transform: translateY(-50%); background: #2a284e; padding: 0 0.75rem; margin: 0; color: var(--label-color); font-weight: bold; display: flex; align-items: center; gap: 0.75rem; font-size: clamp(0.9rem, 2.2vw, 2.2rem); z-index: 1; }
.staging-mode-indicator { font-size: clamp(1.1rem, 2.2vw, 1.6rem); padding: 0.3em 0.8em; border-radius: 4px; font-weight: bold; }
.staging-mode-indicator.loading { background-color: var(--primary-color); color: #000; }
.staging-mode-indicator.unloading { background-color: #9b59b6; color: #fff; }
.zone-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 2rem; }

/* ... (Camera blocks and status) ... */
.camera-block { background: rgba(0, 0, 0, 0.3); border: 2px solid var(--panel-border); border-radius: 12px; display: flex; flex-direction: column; align-items: center; justify-content: center; transition: all 0.4s ease; padding: 1vh 1rem; }
.camera-block.staging { border-width: 4px; height: auto; min-height: 20vh; padding: 2vh 2rem; overflow-y: auto; }
.camera-block.status-success { border-color: var(--success-color); box-shadow: 0 0 10px var(--success-color); }
.camera-block.status-error { border-color: var(--error-color); box-shadow: 0 0 10px var(--error-color); }
.block-panel { word-break: break-all; font-weight: bold; color: var(--primary-color); font-size: clamp(1rem, 5.5vw, 2.5rem); font-family: 'Courier New', Courier, monospace; }
.block-panel.large { font-size: clamp(2rem, 5.5vw, 3rem); margin-bottom: 0.8rem; }
.camera-block.status-error .block-panel { color: var(--error-color); }
.block-label { color: var(--label-color); font-size: clamp(1rem, 1.5vw, 1.2rem); margin-bottom: 1.2rem; }
.status-text-success { color: var(--success-color); text-shadow: 0 0 8px rgba(41, 255, 196, 0.6); }
.status-text-error { color: var(--error-color); text-shadow: 0 0 8px rgba(255, 27, 120, 0.6); }

/* ... (Status display area and lists) ... */
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
.item-panel { font-weight: bold; color: var(--primary-color); font-family: 'Courier New', Courier, monospace; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.history-item-wrapper.is-ng-item .item-panel { color: var(--error-color); }
.item-status-text { font-weight: bold; font-size: 1.1em; }
.item-status-text.status-ok { color: var(--success-color); }
.item-status-text.status-ng { color: var(--error-color); }
.list-item-ng-details { display: grid; grid-template-columns: 1fr 1fr; gap: 0.5rem 1rem; padding-top: 0.5rem; padding-left: 1rem; font-size: 0.85rem; color: var(--label-color); }
.list-item-ng-details span { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }

/* ... (System status, buttons, modal, loading) ... */
.initial-prompt { display: flex; justify-content: center; align-items: center; flex-direction: column; gap: 1.5rem; width: 100%; height: 100%; font-size: 1.8rem; color: var(--waiting-color); }
.system-status { display: flex; align-items: center; gap: 0.5rem; color: var(--success-color); }
.system-status.is-offline { color: var(--error-color); animation: pulse-offline 2s infinite; }
.status-light { width: 10px; height: 10px; background-color: var(--success-color); border-radius: 50%; box-shadow: 0 0 5px var(--success-color); transition: all 0.3s ease; }
.status-light.light-offline { background-color: var(--error-color); box-shadow: 0 0 8px var(--error-color); }
@keyframes pulse-offline { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }

.btn { padding: 0.7rem 1.8rem; font-size: 1rem; border-radius: 8px; cursor: pointer; border: none; transition: all 0.2s ease; font-weight: bold; }
.btn-resetuser, .btn-reset { background: transparent; color: var(--text-color); border: 2px solid var(--panel-border); }
.btn-resetuser:hover, .btn-reset:hover { background-color: rgba(255,255,255,0.1); }
.btn-complete { background: var(--primary-color); color: #000; }
.btn-complete:hover { box-shadow: 0 0 10px var(--primary-color); }
.btn:disabled { background: var(--waiting-color); color: #333; cursor: not-allowed; opacity: 0.6; box-shadow: none; }

/* [New] Mini danger button for top panel */
.btn-mini-danger { background-color: var(--error-color); color: white; border: none; padding: 5px 10px; border-radius: 4px; font-weight: bold; cursor: pointer; font-size: 0.9rem; transition: background-color 0.2s; white-space: nowrap; }
.btn-mini-danger:hover { background-color: #d01662; box-shadow: 0 0 5px var(--error-color); }

/* [New] Side danger button style */
.btn-side-danger { width: 100%; height: 100%; min-height: 140px; border-radius: 8px; font-weight: bold; font-size: 1.2rem; background-color: rgba(255, 27, 120, 0.2); border: 2px solid var(--error-color); color: var(--error-color); cursor: pointer; transition: all 0.3s ease; white-space: pre-wrap; display: flex; align-items: center; justify-content: center; text-align: center; }
.btn-side-danger:hover { background-color: var(--error-color); color: white; box-shadow: 0 0 15px var(--error-color); }

/* ... (Modal and Loading styles) ... */
.window-wrapper { position: fixed; display: flex; justify-content: center; align-items: center; top: 0; left: 0; width: 100%; height: 100%; z-index: 1001; background-color: rgba(0, 0, 0, 0.7); }
.custom-modal { border: 1px solid var(--panel-border); background: var(--panel-bg); box-shadow: 0 0 40px var(--glow-color); backdrop-filter: blur(15px); color: var(--text-color); border-radius: 12px; padding: 2.5rem 3.5rem; width: clamp(300px, 60vw, 700px); max-width: 90%; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 1000; }
.custom-modal::backdrop { background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(8px); }
.modal-content { display: flex; flex-direction: column; gap: 2rem; align-items: center; }
.modal-message { font-size: clamp(1.5rem, 3.5vw, 2.5rem); font-weight: bold; text-align: center; line-height: 1.4; color: var(--primary-color); text-shadow: 0 0 10px rgba(41, 255, 196, 0.5); }
.modal-actions { display: flex; gap: 1.5rem; justify-content: center; width: 100%; }
.btn-modal-confirm { background: var(--primary-color); color: #000; padding: 0.9rem 2.5rem; font-size: clamp(1.1rem, 2vw, 1.4rem); min-width: 120px; }
.btn-modal-confirm:hover { box-shadow: 0 0 15px var(--primary-color); }
.loading-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(5px); display: flex; justify-content: center; align-items: center; z-index: 9999; }
.loading-content { display: flex; flex-direction: column; align-items: center; gap: 1.5rem; background: rgba(21, 22, 46, 0.9); padding: 2rem 3rem; border-radius: 12px; border: 1px solid var(--primary-color); box-shadow: 0 0 20px rgba(41, 255, 196, 0.3); }
.loading-content p { color: var(--primary-color); font-size: 1.5rem; font-weight: bold; margin: 0; }
.spinner { width: 50px; height: 50px; border: 5px solid rgba(41, 255, 196, 0.3); border-top: 5px solid var(--primary-color); border-radius: 50%; animation: spin 1s linear infinite; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

/* Fix platform label height in top row */
.zone-row .platform-indicator-container { align-self: center; width: 80px; }
.zone-row:first-child .platform-label { min-height: 80px; padding: 5px 5px; font-size: 1.1rem; }

/* [新增] 讓工單與按鈕排成一列並垂直置中 */
.info-display-group.action-group { display: flex; align-items: center; gap: 0.8rem; }

/* [修改] 優化上方小按鈕樣式，讓它跟旁邊的大字體比較搭 */
.btn-mini-danger {
  background-color: var(--error-color);
  color: white;
  border: none;
  /* 增加高度與 Padding，讓按鈕看起來不會太「縮」 */
  padding: 0.4rem 1.2rem; 
  font-size: 1.1rem; /* 字體加大 */
  border-radius: 6px;
  font-weight: bold;
  cursor: pointer;
  white-space: nowrap;
  box-shadow: 0 4px 6px rgba(0,0,0,0.3); /* 加一點陰影更有質感 */
  transform: translateY(2px); /* 微調垂直位置，視情況增減 */
  transition: all 0.2s ease;
}

.btn-mini-danger:hover { background-color: #d01662; box-shadow: 0 0 10px var(--error-color); transform: translateY(1px); }
</style>