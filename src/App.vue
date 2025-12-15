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
          <div class="system-status">
            <span>SYSTEM ONLINE</span>
            <div class="status-light"></div>
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
                      <span :title="item.detail">{{ t('原因') }}: {{ item.detail }}</span>
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
                      <span :title="item.detail">{{ t('原因') }}: {{ item.detail }}</span>
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
        <!-- <button class="btn btn-reset" @click.stop="refresh">{{ t('重整') }} (Reset)</button> -->
        <button class="btn btn-execute" @click.stop="forceExecute">{{ t('強制執行') }}</button>
        <button class="btn btn-complete" @click.stop="completeAllScanned">{{ t('完成') }} (Complete)</button>
      </footer>
   </div>
  </div>
  
  <dialog ref="customModal" class="custom-modal" @cancel.prevent>
    <div class="modal-content">
      <p class="modal-message">{{ modalMessage }}</p>
      <div class="modal-actions" v-if="modalType === 'confirm'">
        <button class="btn btn-modal-cancel" @click="cancelModal">取消</button>
        <button class="btn btn-modal-confirm" @click="confirmModal">確定</button>
      </div>
      <div class="modal-actions" v-else>
        <button class="btn btn-modal-confirm" @click="closeModal">確定</button>
      </div>
    </div>
  </dialog>
</template>

<script setup>
// restoreState, handleResetAll function need implement
import { ref, onMounted, onUnmounted, computed, nextTick, watch, reactive } from 'vue';
import logo from "@/components/icons/flexium_logo.png"

const API_URL = "http://10.1.5.122/gxfirstOIS/gxfirstOIS.asmx/GetOISData";
const OIS_API_BASE = "http://10.1.5.124:2151/api";
const wsUrl = 'ws://127.0.0.1:8181'
const headers = { 'Content-Type': 'application/json' };

const currentLang = ref('zh'); 
const i18nDict = {
  "LPSM 2DID掃描系統": { vn: "Hệ thống scan LPSM 2DID" },
  "工號": { vn: "Mã số NV" },
  "工單": { vn: "Công đơn" },
  "姓名": { vn: "Họ tên" },
  "品目": { vn: "Mục hàng" },
  "總張數": { vn: "Tổng số trang" },
  "投入張數": { vn: "Số trang đầu vào" },
  "作業區": { vn: "Khu làm việc" },
  "上台面": { vn: "Mặt trên máy" }, 
  "下台面": { vn: "Mặt dưới máy" },
  "切換模式": { vn: "Thay đổi dạng thức" },
  "放料區": { vn: "Chỗ để liệu" },
  "待入料": { vn: "Chờ nhập liệu" },
  "出料": { vn: "Lấy liệu" }, 
  "準備區-左": { vn: "Chỗ chuẩn bị - Trái" },
  "準備區-右": { vn: "Chỗ chuẩn bị - Phải" },
  "原因": { vn: "Nguyên nhân" },
  "站點": { vn: "Công đoạn" },
  "換人生產": { vn: "Thay người SX" },
  "重整": { vn: "Chỉnh lại" },
  "完成": { vn: "Hoàn thành" },
  "等待掃描...": { vn: "Đang chờ..." },
  "進": { vn: "Vào" },
  "出": { vn: "Ra" },
  "請刷入工號與工單以開始作業": { vn: "Vui lòng quét mã nhân viên và công đơn để bắt đầu thao tác" },
  "強制執行": { vn: "Thực hiện cưỡng chế" },
};

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

// Display state
const isBaseInfoFilled = computed(() => !!(info.employeeId && info.workOrder && info.employeeName && info.productItem));
const formatTime = (ms) => new Date(ms).toLocaleTimeString();
const _in_platform = computed(() => {
  if (PlatformState.up_in == 1) return "上台面";
  else if (PlatformState.dn_in == 1) return "下台面";
  return "---";
})
const _out_platform = computed(() => {
  if (_in_platform.value === "上台面") {
    return "下台面";
  }
  else if (_in_platform.value === "下台面") {
    return "上台面";
  }
  return "---";
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
  if (scanned_2DID_dict[pdcode]) return "出";
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
let socket = null;
function connectWebSocket() {
  socket = new WebSocket(wsUrl);
  
  socket.onopen = () => { 
    console.log("WS Connected");
    startHeartbeat(); // ✅ 開始送 heartbeat
    restoreState();
  }

  socket.onmessage = (event) => {
    try {
      const message = JSON.parse(event.data)
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
        // Test command
        else if (message.source === "command" && message.payload === "RESET") {
          resetAll();
        }
        // Process infomation input from keyboard
        else if (message.source === "SCANNER") {
          handleInfoInput(message.payload);
        }
        // Process camera input
        else if (message.source.includes("CAMERA")) {
          processControl(message.payload, message.source);
        }
        else {
          console.log("WebSocket receive message: ", message);
        }
      }
      else if (message.type === "control" && message.command === "HEARTBEAT_ACK") {
        lastHeartbeatAck.value = message.payload?.server_ts || Date.now();
        console.log("[HB] ack", message.payload);
        return;
      }
      else if (message.type === "control" && message.command === "STATE_SYNC") {
        const s = message.payload || {}

        // 注意：要用 Object.assign 保持 reactive
        if (s.info) Object.assign(info, s.info)
        if (s.stage) stage.value = s.stage
        if (s.plc) Object.assign(PlatformState, {
          up_in: s.plc.up_in ?? -1,
          dn_in: s.plc.dn_in ?? -1,
          start_message: s.plc.start_message ?? -1
        })

        // dict
        if (s.requested_2DID_dict) Object.assign(requested_2DID_dict, s.requested_2DID_dict)
        if (s.expected_2DID_dict) Object.assign(expected_2DID_dict, s.expected_2DID_dict)
        if (s.scanned_2DID_dict) Object.assign(scanned_2DID_dict, s.scanned_2DID_dict)
        if (s.other_2DID_dict) Object.assign(other_2DID_dict, s.other_2DID_dict)
        if (s.history_2DID_dict) Object.assign(history_2DID_dict, s.history_2DID_dict)

        // platform display
        if (s.up_platform_2DID) Object.assign(up_platform_2DID, s.up_platform_2DID)
        if (s.dn_platform_2DID) Object.assign(dn_platform_2DID, s.dn_platform_2DID)

        // list
        if (s.side_history_2DID_list) Object.assign(side_history_2DID_list, s.side_history_2DID_list)

        // force
        if (s.forceCommand) Object.assign(forceCommand, s.forceCommand)

        console.log("[STATE] restored")
        return
      }
    }
    catch (error) {
      console.error("WebSocket 解析訊息失敗: ", error);
    }
  }

  socket.onclose = () => {
    console.log("WebSocket: 連接已關閉，3秒後嘗試重連...");
    stopHeartbeat(); // ✅ 關閉 heartbeat
    socket = null;

    PlatformState.up_in = -1;
    PlatformState.dn_in = -1;
    PlatformState.start_message = -1;
    setTimeout(connectWebSocket, 3000);
  }

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

// Helper
let patchBuffer = []
let patchTimer = null
function queuePatch(ev){
  patchBuffer.push({ ...ev, ts: Date.now() })
  // 20~50ms 合併成一包，避免太碎
  if (!patchTimer){
    patchTimer = setTimeout(() => {
      sendSocketMessage("STATE_PATCH", { events: patchBuffer });
      patchBuffer = [];
      patchTimer = null;
    }, 30)
  }
}

// ---- wrappers ----
const patchDictDel = (path, key) => {queuePatch({ op:"DICT_DEL", path, key })}
const patchDictClear = (path) => {patchSet(path, {})}
const patchListClear = (path) => {queuePatch({ op:"LIST_CLEAR", path })}
const patchResetAll = () => {queuePatch({ op:"RESET_ALL" })}

const patchListSet = (path, index, value) => {queuePatch({ op:"LIST_SET", path, index, value })}
const patchListClearAndInit = (path, initList=[]) => {patchSet(path, initList)}
const patchSet = (path, value) => {queuePatch({ op:"SET", path, value })}
const patchDictSet = (path, key, value) => {queuePatch({ op:"DICT_SET", path, key, value })}
const patchListUnshift = (path, value, max_len=500) => {queuePatch({ op:"LIST_UNSHIFT", path, value, max_len })}

// --- Custom Modal --- //
const customModal = ref(null);
const modalMessage = ref('');
const modalType = ref('alert'); // 'alert' or 'confirm'
let resolveModalPromise = null;

const showCustomModal = (message, type = 'alert') => {
  modalMessage.value = message;
  modalType.value = type;
  customModal.value.showModal();

  return new Promise((resolve) => {
    resolveModalPromise = resolve;
  });
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
    const response = await fetch(API_URL,{ method: "POST", headers: { "Content-Type": "application/x-www-form-urlencoded" }, body: requestData.toString()});
    const responseText = await response.text();
    const json = JSON.parse(responseText);
    if (json.code === 200 && json.data) {
      const {empNo, empName} = json.data;
      return {empNo, empName};
    } else {
      await showCustomModal(`工號驗證失敗：${json.message || '未知錯誤'}`);
      return false;
    }
  } catch(error) {
    console.error("validateEmpId 發生錯誤:", error);
    await showCustomModal(`工號驗證時發生網路錯誤：${error.message}`);
    return false;
  }
}

async function validateWorkOrder(workOrder) {
  let res, res_json;
  try {
    res = await fetch(`${OIS_API_BASE}/Read_2DID`, { method: "POST", headers, body: JSON.stringify({ emp_no: info.employeeId, workorder: workOrder }) });
    res_json = await res.json();
    if (res_json.success && res_json.data) {
      const pdcodedata = res_json.pdcode_data;
      info.cmd236_flag = pdcodedata.cmd236_Flag;
      patchSet(["info","cmd236_flag"], info.cmd236_flag);
      return {
        workorder: pdcodedata.workOrder,
        item: pdcodedata.item,
        workStep: pdcodedata.work_step,
        sht_no: pdcodedata.sht_no,
        panel_no: pdcodedata.panel_no,
        twodid_step: pdcodedata.twodid_step,
        twodid_type: pdcodedata.twodid_type,
        twodid_input: pdcodedata.twodid_input,
        panel_num: pdcodedata.panel_num
      }
    }

    res = await fetch(`${OIS_API_BASE}/workorder`, { method: 'POST', headers, body: JSON.stringify({ emp_no: info.employeeId, workorder: workOrder }) });
    res_json = await res.json();
    info.cmd236_flag = false;
    patchSet(["info","cmd236_flag"], info.cmd236_flag);
    if (res_json.success && res_json.result.result.startsWith("OK")) {
      const lines = res_json.result.result.slice(3).split("\r\n");
      let workorder = workOrder, item = lines[0][0], workStep = lines[0][1], panel_num = lines.length;
      let sht_no = [], panel_no = [], twodid_step = [], twodid_type = [], twodid_input = [];
      for (const line of lines) {
        const parts = line.split(";");
        sht_no.push(parts[2]);
        panel_no.push(parts[3]);
        twodid_step.push(parts[4]);
        twodid_type.push(parts[5]);
        twodid_input.push("N");
      }
      return { workorder, item, workStep, sht_no, panel_no, twodid_step, twodid_type, twodid_input, panel_num };
    }

    res = await fetch(`${OIS_API_BASE}/workorder2`, { method: 'POST', headers, body: JSON.stringify({ emp_no: info.employeeId, workorder: workOrder }) });
    res_json = await res.json();
    info.cmd236_flag = true;
    patchSet(["info","cmd236_flag"], info.cmd236_flag);
    if (res_json.success && res_json.result.result.startsWith("OK")) {
      const lines = res_json.result.result.slice(3).split("\r\n");
      let workorder = workOrder, item = lines[0][1], workStep = lines[0][2], panel_num = lines[0][8];
      let sht_no = [], panel_no = [], twodid_step = [], twodid_type = [], twodid_input = [];
      for (const line of lines) {
        const parts = line.split(";");
        sht_no.push(parts[3]);
        panel_no.push(parts[4]);
        twodid_step.push(parts[5]);
        twodid_type.push(parts[6]);
        twodid_input.push("N");
      }
      return { workorder, item, workStep, sht_no, panel_no, twodid_step, twodid_type, twodid_input, panel_num };
    }

    await showCustomModal(`工單驗證失敗：${res_json.result?.result || '無有效工單數據'}`);
    return null;
  }
  catch (error) {
    console.error("validateWorkOrder 發生錯誤: ", error)
    await showCustomModal(`工單驗證時發生網路錯誤：${error.message}`);
    return null;
  }
}

async function validate2DID(twodid) {
  if (!twodid) return { ret_type: "NG", workOrder: "N/A", item: "N/A", detail: "條碼異常" };

  const sheet_info = expected_2DID_dict[twodid];
  if (sheet_info) {
    if (sheet_info.twodid_type == "Y") {
      await NGSheetReport(twodid, sheet_info.panel_no, "跳站");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "跳站" };
    }
    else if (sheet_info.twodid_step === info.workStep) {
      await NGSheetReport(twodid, sheet_info.panel_no, "重工");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
    }

    const scanned_record = scanned_2DID_dict[twodid];
    if (scanned_record && Date.now() - scanned_record.timestamp > unloadPeriodThreshold) {
      await NGSheetReport(twodid, sheet_info.panel_no, "重工");
      return { ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: "重工" };
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
    if (twodid_info.workOrder != info.workOrder) detail += "混工單;";
    if (twodid_info.item != info.productItem) detail += "混品目;"; 

    if (detail === "") {
      await showCustomModal("[validate2DID] 進入 requested_2DID_dict 分支但 detail 為空，可能邏輯被改壞了")
      detail = "異常錯誤"
    }
    else {
      other_2DID_dict[twodid] = { ret_type: "NG", workOrder: twodid_info.workOrder, item: twodid_info.item, detail: (detail === "") ? "異常錯誤" : detail };
      patchDictSet(["other_2DID_dict"], twodid, { ...other_2DID_dict[twodid] });	
      NGSheetReport(twodid, twodid_info.panel_no, (detail === "") ? "異常錯誤" : detail);
    }

    return { ret_type: "NG", workOrder: twodid_info.workOrder, item: twodid_info.item, detail: (detail === "") ? "異常錯誤" : detail};
  }

  // This 2DID has not detect in the history requested need to get the information from server
  try {
    const res = await fetch(`${OIS_API_BASE}/twodid`, { method: 'POST', headers, body: JSON.stringify({ emp_no : info.employeeId, twodid }) });
    const res_json = await res.json();
    if (res_json.success) {
      const [_, workOrder, productItem, workStep] = res_json.result.result.split(";");

      if (workOrder != info.workOrder) detail += "混工單;";
      if (productItem != info.productItem) detail += "混品目;";
      other_2DID_dict[twodid] = { ret_type: "NG", workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };
      patchDictSet(["other_2DID_dict"], twodid, { ...other_2DID_dict[twodid] });	

      if (detail != ""){
        const panels_info = await validateWorkOrder(workOrder);
        if (panels_info) {
          create2DIDDict(panels_info, false, false);
          const twodid_info = requested_2DID_dict[twodid];
          await NGSheetReport(twodid, twodid_info.panel_no, detail);
        }
        else {
          console.error("此製品之工單查無任何資料，因此不上傳資訊")
        }
      }

      return { ret_type: "NG", workOrder: workOrder, item: productItem, detail: (detail == "") ? "異常錯誤" : detail };
    }
    other_2DID_dict[twodid] = { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" }
    patchDictSet(["other_2DID_dict"], twodid, { ...other_2DID_dict[twodid] });	
    return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
  }
  catch (error) {
    console.error("驗證2DID發生錯誤，請確認網路連線: ", error);
  }

  return { ret_type: "NG", workOrder: "查無工單", item: "查無品目", detail: "異常錯誤" };
}

// --- Control Process --- //
// Chage stage
function changeStage(s) {
  stage.value = s;
  sendSocketMessage("STEP_UPDATE", s);
  patchSet(["stage"], s);
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
  patchResetAll();
}

// Storage history request and list expected 2DID in target work order
function create2DIDDict(panelList, initial = true, addExpected = true) {
  const sht_no_list = panelList.sht_no;
  const panel_no_list = panelList.panel_no;
  const twodid_step_list = panelList.twodid_step;
  const twodid_type_list = panelList.twodid_type;

  if (initial){
    Object.assign(requested_2DID_dict, {}); patchDictClear(["requested_2DID_dict"]);
    Object.assign(expected_2DID_dict, {}); patchDictClear(["expected_2DID_dict"]);
    Object.assign(scanned_2DID_dict, {}); patchDictClear(["scanned_2DID_dict"]);
    Object.assign(other_2DID_dict, {}); patchDictClear(["other_2DID_dict"]);
  }

  for (let index = 0; index < panelList.panel_num; index++) {
    requested_2DID_dict[sht_no_list[index]] = { workOrder: panelList.workorder, item: panelList.item, workStep: panelList.workStep, panel_no: panel_no_list[index], twodid_step: twodid_step_list[index], twodid_type: twodid_type_list[index] };
    patchDictSet(["requested_2DID_dict"], sht_no_list[index], { ...requested_2DID_dict[sht_no_list[index]] });
    if (addExpected){
      expected_2DID_dict[sht_no_list[index]] = { panel_no: panel_no_list[index], twodid_step: twodid_step_list[index], twodid_type: twodid_type_list[index] };
      patchDictSet(["expected_2DID_dict"], sht_no_list[index], { ...expected_2DID_dict[sht_no_list[index]] });
    }
  }
}

// Handle employee login and check work order valid
const handleInfoInput = async (data) => {
  const value = String(data).replace(/[\x00-\x1F\x7F-\x9F]/g, "").trim().toUpperCase();

  if (!value) return;

  if (stage.value === "empId") {
    const result = await validateEmpId(value);

    if (!result) return;

    info.employeeId = result.empNo;
    info.employeeName = result.empName;
    patchSet(["info","employeeId"], info.employeeId);
    patchSet(["info","employeeName"], info.employeeName);
    changeStage((info.workOrder === "") ? "workOrder" : "twoDID");
  }

  else if (stage.value === "workOrder") {
    if (value.length < 8 || value.length > 11) {
      await showCustomModal("工單格式不正確，請重新掃描。");
      return;
    }

    const valid = await validateWorkOrder(value);
    if (valid && valid.workStep !== ""){
      info.workOrder = valid.workorder; patchSet(["info","workOrder"], info.workOrder);
      info.productItem = valid.item; patchSet(["info","productItem"], info.productItem);
      info.workStep = valid.workStep; patchSet(["info","workStep"], info.workStep);
      info.panel_num = valid.panel_num; patchSet(["info","panel_num"], info.panel_num);
      info.OK_num = 0; patchSet(["info","OK_num"], info.OK_num);
      info.NG_num = 0; patchSet(["info","NG_num"], info.NG_num);
      create2DIDDict(valid);
      changeStage("twoDID");
    }
  }
}

// Check platform at loading area
function detectTargetPlatform() {
  if (PlatformState.up_in != 1) return up_platform_2DID;
  if (PlatformState.dn_in != 1) return dn_platform_2DID;
  return null;
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
    await showCustomModal("產品條碼長度不正確，請重新掃描。");
    return;
  }

  // Check platform at loading area
  let targetPlatform = detectTargetPlatform();
  if (targetPlatform == null) return null;
  const targetPlatformInfo = (PlatformState.up_in == 1) ? "dn_platform_2DID" :  "up_platform_2DID";

  // Receive camera timeout signal
  if (value == "TIMEOUT_BLANK") {
    if (source.startsWith("CAMERA_LEFT")) {
      const lastLeft = side_history_2DID_list.left[0]
      if (!lastLeft || Date.now() - lastLeft.timestamp > 1500) {
        targetPlatform.left = { pdcode: "等待掃描...", ret_type: "", detail: "" };
        patchSet([targetPlatformInfo, "left"], { ...targetPlatform.left });
      }
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      const lastRight = side_history_2DID_list.right[0]
      if (!lastRight || Date.now() - lastRight.timestamp > 1500) {
        targetPlatform.right = { pdcode: "等待掃描...", ret_type: "", detail: "" };
        patchSet([targetPlatformInfo, "right"], { ...targetPlatform.right });
      }
    }

    if (forceCommand.command === true && (forceCommand.triggerTime && Date.now() - forceCommand.triggerTime < forceTriggerPeriod) && (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK")) {
      sendSocketMessage("GO_NOGO", 1);
    }
    else {
      sendSocketMessage("GO_NOGO", 0);
    }
    return
  }

  // Check product validate
  let targetSheet = (source.startsWith("CAMERA_RIGHT") || source.startsWith("CAMERA_LEFT")) ? value : null;
  let validResult = await validate2DID(targetSheet);
  const rec = { timestamp: Date.now(), ret_type: validResult.ret_type, workOrder: validResult.workOrder, item: validResult.item, detail: validResult.detail };

  // Filtered short period repeated 2DID for frontend log
  const sheet_history = (source.startsWith("CAMERA_LEFT")) ? side_history_2DID_dict.left[value] : side_history_2DID_dict.right[value];
  if (sheet_history && Date.now() - sheet_history.timestamp > shortPeriodFilterThreshold) {
    if (source.startsWith("CAMERA_LEFT")) {
      sheet_history.timestamp = Date.now(); patchDictSet(["side_history_2DID_dict","left"], value, { ...sheet_history });
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      sheet_history.timestamp = Date.now(); patchDictSet(["side_history_2DID_dict","right"], value, { ...sheet_history });
    }
  }
  else {
    if (source.startsWith("CAMERA_LEFT")) { 
      side_history_2DID_list.left.unshift({ ...rec, pdcode: value }); patchListUnshift(["side_history_2DID_list","left"], { ...rec, pdcode: value }, 500);
      side_history_2DID_dict.left[value] = { ...rec }; patchDictSet(["side_history_2DID_dict","left"], value, { ...rec });
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      side_history_2DID_list.right.unshift({ ...rec, pdcode: value }); patchListUnshift(["side_history_2DID_list","right"], { ...rec, pdcode: value }, 500);
      side_history_2DID_dict.right[value] = { ...rec }; patchDictSet(["side_history_2DID_dict","right"], value, { ...rec });
    }
  }

  // Filtered short period repeated 2DID to record
  if (history_2DID_dict[value]) {
    history_2DID_dict[value].timestamp = Date.now(); patchDictSet(["history_2DID_dict"], value, { ...history_2DID_dict[value] });
  }
  else {
    history_2DID_dict[value] = { ...rec }; patchDictSet(["history_2DID_dict"], value, { ...history_2DID_dict[value] });
  }

  // Filled the information for frontend
  if (validResult.ret_type == "OK") {
    if (source.startsWith("CAMERA_LEFT")) {
      targetPlatform.left = { pdcode: value, ret_type: validResult.ret_type, detail: validResult.detail }; patchSet([targetPlatformInfo, "left"], { ...targetPlatform.left });
    }
    else if (source.startsWith("CAMERA_RIGHT")) {
      targetPlatform.right = { pdcode: value, ret_type: validResult.ret_type, detail: validResult.detail }; patchSet([targetPlatformInfo, "right"], { ...targetPlatform.right });
    }
  }

  if (forceCommand.command === true && (forceCommand.triggerTime && Date.now() - forceCommand.triggerTime < forceTriggerPeriod) && (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK")) {
    sendSocketMessage("GO_NOGO", 1);
  }
  else if (stage.value == "twoDID" && targetPlatform.left.ret_type === "OK" && targetPlatform.right.ret_type === "OK") {
    sendSocketMessage("GO_NOGO", 1);
  }
  else {
    sendSocketMessage("GO_NOGO", 0);
  }
}

watch(
  () => [PlatformState.up_in, PlatformState.dn_in], ([up, dn], [prevUp, prevDn]) => {
    if (up === 1 && prevUp !== 1) scanedSheetRecord(up_platform_2DID);
    if (dn === 1 && prevDn !== 1) scanedSheetRecord(dn_platform_2DID);
  }
);

function scanedSheetRecord(platform) {
  const scanned_2DID_info = history_2DID_dict[pdcode];
  if (platform.left.ret_type == "OK") {
    const pdcode = platform.left.pdcode;
    if (pdcode != "等待掃描..." && !scanned_2DID_dict[pdcode]){
      scanned_2DID_dict[pdcode] = {...scanned_2DID_info, timestamp: Date.now()}; patchDictSet(["scanned_2DID_dict"], pdcode, { ...scanned_2DID_dict[pdcode] });
      info.OK_num += 1; patchSet(["info","OK_num"], info.OK_num);
    }
  }

  if (platform.right.ret_type == "OK") {
    const pdcode = platform.right.pdcode;
    if (pdcode != "等待掃描..." && !scanned_2DID_dict[pdcode]){
      scanned_2DID_dict[pdcode] = {...scanned_2DID_info, timestamp: Date.now()}; patchDictSet(["scanned_2DID_dict"], pdcode, { ...scanned_2DID_dict[pdcode] });
      info.OK_num += 1; patchSet(["info","OK_num"], info.OK_num);
    }
  }
}

const NGSheetReport = async (pdcode, panel_no, status) => {
  if (expected_2DID_dict[pdcode] && !scanned_2DID_dict[pdcode]) {
    scanned_2DID_dict[pdcode] = { timestamp: Date.now(), ret_type: "NG", workOrder: info.workOrder, item: info.productItem, detail: status }; patchDictSet(["scanned_2DID_dict"], pdcode, { ...scanned_2DID_dict[pdcode] });
    await UploadSheet(pdcode, panel_no, "NG", status);
    info.NG_num += 1; patchSet(["info","NG_num"], info.NG_num);
  }
  else if (!other_2DID_dict[pdcode]){
    await UploadSheet(pdcode, panel_no, "NG", status);
  }
}

const UploadSheet = async (pdcode, panel_no, ret_type, status) => {
  const payload = { emp_no: info.employeeId, workOrder: info.workOrder, partno: info.productItem, work_step: info.workStep, sht_no: pdcode, panel_no: panel_no, twodid_type: ret_type, remark: status };
  try {
    await fetch(`${OIS_API_BASE}/write2did`, { method: "POST", headers, body: JSON.stringify(payload) });
  }
  catch (error) {
    console.error("寫入 NG 狀態失敗:", error);
  }
}

// --- Button Control --- //
function employeeClear() {
  info.employeeId = ""; patchSet(["info","employeeId"], info.employeeId);
  info.employeeName = ""; patchSet(["info","employeeName"], info.employeeName);
  changeStage("empId");
}

async function forceExecute() {
  let targetPlatform = detectTargetPlatform();
  if (targetPlatform == null) {
    await showCustomModal("請等待平台退回上料區");
    return;
  }

  if (targetPlatform.left.ret_type === "OK" || targetPlatform.right.ret_type === "OK") {
    forceCommand.command = true; patchSet(["forceCommand","command"], forceCommand.command);
    forceCommand.triggerTime = Date.now(); patchSet(["forceCommand","triggerTime"], forceCommand.triggerTime);
    sendSocketMessage("GO_NOGO", 1);
    return;
  }

  await showCustomModal("請放入至少放入一個OK製品");
}

const completeAllScanned = async () => {
  if (PlatformState.dn_in == 1 || PlatformState.up_in == 1 || (up_platform_2DID.left?.ret_type != "" || up_platform_2DID.right?.ret_type != "" || dn_platform_2DID.left.ret_type != "" || dn_platform_2DID.right.ret_type != "")) {
    await showCustomModal("請保持平台無任何製品");
    return;
  }

  for (const [key, value] of Object.entries(expected_2DID_dict)) {
    if (!scanned_2DID_dict[key]) {
      await UploadSheet(key, value.panel_no, "NG", "未投入，作業結束");
    }
  }

  try {
    await fetch(`${OIS_API_BASE}/Delete_2DID`, { method: "POST", headers, body: JSON.stringify({ workorder: info.workOrder }) });
  }
  catch (error) {
    console.error("清除 2DID 資料失敗")
  }

  await showCustomModal("作業已完成！");
  resetAll();
}

// --- Window Control Function --- //
function restoreState() {
  sendSocketMessage("LOAD_STATE", { page: "scan-ui" })
}

window.addEventListener("beforeunload", () => {
  sendSocketMessage("SAVE_STATE", { page: "scan-ui" })
})

onMounted(() => { 
  connectWebSocket();
})
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
.status-light { width: 10px; height: 10px; background-color: var(--success-color); border-radius: 50%; }
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

.zone-row .platform-indicator-container { align-self: center; }
.zone-row:first-child .platform-label { min-height: 80px; padding: 5px 5px; font-size: 1.1rem; }
</style>