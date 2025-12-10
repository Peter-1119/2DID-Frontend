<template>
  <div class="scan-system-container" @click="forceFocus">
    <input ref="universalInput" v-model="scannedData" @keyup.enter="handleManualInput" @blur="forceFocus" class="hidden-input"/>
    <div class="main-panel">
      <!-- Header -->
      <header class="panel-header">
        <img :src="logo" alt="flexium logo" class="system-logo">
        <h1>LPSM 掃描系統</h1>
        <div class="system-status">
          <span>SYSTEM ONLINE</span>
          <div class="status-light"></div>
        </div>
      </header>
      <!-- Info Section -->
      <section class="info-display-section-compact">
        <div class="info-grid-compact">
          <div class="info-display-group"><span class="info-label">工號：</span><span class="info-value">{{ employeeId || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">姓名：</span><span class="info-value">{{ employeeName || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">工單：</span><span class="info-value">{{ workOrder || "---" }}</span></div>
          <div class="info-display-group"><span class="info-label">品目：</span><span class="info-value">{{ productItem || "---" }}</span></div>
        </div>
      </section>

       <!-- 切換模式按鈕與張數顯示 -->
      <div class="toggle-button-wrapper">
        <!-- 張數顯示區塊 (靠左) -->
        <div class="sheet-count-display">
          <span class="info-label">總張數：</span><span class="info-value">{{ _panel_num || 0 }}</span>
          <span class="info-label">投入張數：</span><span class="info-value">{{ _input_num || 0 }}</span>
        </div>

        <!-- 作業區/放料區狀態顯示 (置中) -->
        <div class="process-area-status">
          <!-- <span class="info-label">作業區：</span><span class="info-value">{{ _in_platform }}</span> -->
          <!-- <span class="info-label">放料區：</span><span class="info-value">{{ _out_platform }}</span> -->
        </div>

        <!-- 切換模式按鈕 (靠右) -->
        <button class="btn btn-toggle" @click="handleToggle">切換模式</button>
      </div>

      <!-- 主視覺區 -->
      <main class="camera-section-main adaptive-view">
        <template v-if="isBaseInfoFilled">
          <div class="main-content-wrapper">
            <div class="perspective-container">
              <div class="zones-in-perspective">

                <!-- 作業區 -->
                <div class="zone-container">
                  <h2 class="zone-title">作業區</h2>
                  <div class="zone-grid">
                    <div class="camera-block" :class="`status-${inProcessLeftCamera?.status}`">
                      <span class="block-pdcode">{{ inProcessLeftCamera?.pdcode || "---" }}</span>
                    </div>
                    <div class="camera-block" :class="`status-${inProcessRightCamera?.status}`">
                      <span class="block-pdcode">{{ inProcessRightCamera?.pdcode || "---" }}</span>
                    </div>
                  </div>
                </div>

                <!-- 放料區 -->
                <div class="zone-container staging-zone">
                  <h2 class="zone-title">放料區<span class="staging-mode-indicator" :class="stagingMode">{{ stagingModeText }}</span></h2>
                  <div class="zone-grid">
                    <div class="camera-block staging" :class="`status-${stagingLeftCamera?.status}`">
                      <span class="block-label">準備區 - 左</span>
                      <span
                        v-if="stagingLeftCamera.status !== 'waiting' && stagingMode === 'loading' && stagingLeftCamera?.pdcode"
                        class="block-pdcode large"
                        :class="`status-text-${stagingLeftCamera?.status}`"
                      >
                        {{ stagingLeftCamera.status === "success" ? "OK" : "NG" }}
                      </span>
                      <span
                        v-if="stagingLeftCamera.status !== 'waiting' && stagingMode === 'unloading' && stagingLeftCamera?.pdcode"
                        class="block-pdcode large"
                        :class="`status-text-${stagingLeftCamera?.status}`"
                      >
                        {{ stagingLeftCamera?.status === "success" ? "OK" : "NG" }}
                      </span>

                      <span class="block-pdcode large">{{ stagingLeftCamera?.pdcode || "等待掃描..." }}</span>
                    </div>
                    <div class="camera-block staging" :class="`status-${stagingRightCamera?.status}`">
                      <span class="block-label">準備區 - 右</span>

                      <span
                        v-if="stagingRightCamera.status !== 'waiting' && stagingMode === 'loading' && stagingRightCamera?.pdcode"
                        class="block-pdcode large"
                        :class="`status-text-${stagingRightCamera?.status}`"
                      >
                        {{ stagingRightCamera.status === "success" ? "OK" : "NG" }}
                      </span>
                      <span
                        v-if="stagingRightCamera.status !== 'waiting' && stagingMode === 'unloading' && stagingRightCamera?.pdcode"
                        class="block-pdcode large"
                        :class="`status-text-${stagingRightCamera?.status}`"
                      >
                        {{ stagingRightCamera?.status === "success" ? "OK" : "NG" }}
                      </span>

                      <span class="block-pdcode large">{{ stagingRightCamera?.pdcode || "等待掃描..." }}</span>
                    </div>
                  </div>
                </div>

              </div>
            </div>

            <!-- 左右掃描紀錄清單 -->
            <div class="status-display-area">

              <!-- 左側清單 -->
              <div class="status-list-column">
                <div v-if="leftScanHistory.length === 0" class="list-placeholder">
                  <div class="list-item-placeholder"></div>
                  <div class="list-item-placeholder"></div>
                  <div class="list-item-placeholder"></div>
                </div>
                <div v-else class="status-list">
                  <div v-for="item in leftScanHistory" :key="item.uid" class="history-item-wrapper" :class="{ 'is-ng-item': item.status === 'error' }">
                    <!-- ★★★ START: 修改為單行結構 ★★★ -->
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ item.timestamp }}</span>
                      <span class="item-pdcode">{{ item.pdcode }}</span>
                      <span class="item-action-type">({{ item.actionType }})</span>
                      <span class="item-status-text" :class="item.status === 'success' ? 'status-ok' : 'status-ng'">{{ item.status === "success" ? "OK" : "NG" }}</span>
                    </div>
                    <!-- ★★★ END: 修改為單行結構 ★★★ -->
                    <div v-if="item.status === 'error'" class="list-item-ng-details">
                      <span :title="item.ngReason">原因: {{ item.ngReason }}</span>
                      <span :title="item.workOrder">工單: {{ item.workOrder }}</span>
                      <span :title="item.productItem">品目：{{ item.productItem }}</span>
                      <span :title="item.station">站點:{{ item.station }}</span>
                    </div>
                  </div>
                </div>
              </div>
              
              <!-- 右側清單 -->
              <div class="status-list-column">
                <div v-if="rightScanHistory.length === 0" class="list-placeholder">
                  <div class="list-item-placeholder"></div>
                  <div class="list-item-placeholder"></div>
                  <div class="list-item-placeholder"></div>
                </div>
                <div v-else class="status-list">
                  <div v-for="item in rightScanHistory" :key="item.uid" class="history-item-wrapper" :class="{' is-ng-item' : item.status === 'error'}">
                    <!-- ★★★ START: 修改為單行結構 ★★★ -->
                    <div class="list-item-top-row">
                      <span class="item-timestamp">{{ item.timestamp }}</span>
                      <span class="item-pdcode">{{ item.pdcode }}</span>
                      <span class="item-action-type">({{ item.actionType }})</span>
                      <span class="item-status-text" :class="item.status === 'success' ? 'status-ok' : 'status-ng'">{{ item.status === "success" ? "OK" : "NG" }}</span>
                    </div>
                    <!-- ★★★ END: 修改為單行結構 ★★★ -->
                    <div v-if="item.status === 'error'" class="list-item-ng-details">
                      <span :title="item.ngReason">原因: {{ item.ngReason }}</span>
                      <span :title="item.workOrder">工單: {{ item.workOrder }}</span>
                      <span :title="item.productItem">品目: {{ item.productItem }}</span>
                      <span :title="item.station">終點: {{ item.station }}</span>
                    </div>
                  </div>
                </div>
              </div>

            </div>
            
          </div>
        </template>
        <div v-else class="initial-prompt">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" class="prompt-icon">
            <path d="M416 208c0 45.9-14.9 88.3-40 122.7L502.6 457.4c12.5 12.5 12.5 32.8 0 45.3s-32.8 12.5-45.3 0L330.7 376c-34.4 25.2-76.8 40-122.7 40C92.1 416 0 323.9 0 208S92.1 0 208 0S416 92.1 416 208zM208 352a144 144 0 1 0 0-288 144 144 0 1 0 0 288z"/>
          </svg>
          請刷入工號與工單以開始作業
        </div>
      </main>
      <!-- Footer -->
      <footer class="panel-footer">
        <button class="btn btn-resetuser" @click.stop="handleRestUser">換人生產</button>
        <button class="btn btn-reset" @click.stop="handleResetAll">重整 (Reset)</button>
        <button class="btn btn-complete" @click.stop="handleComplete" :disabled="!isReadyToComplete">完成 (Complete)</button>
      </footer>
   </div>
  </div>
  <!-- Custom Alert/Confirm Modal -->
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
import { ref, onMounted, onUnmounted, computed, nextTick, watch } from 'vue';
import logo from "@/components/icons/flexium_logo.png"

// --- State Definitions ---
const employeeId = ref('');
const employeeName = ref('');
const workOrder = ref('');
const productItem = ref('');
const scannedData = ref('');
const universalInput = ref(null);
const stagingMode = ref('loading'); // 'loading' (待入料) 或 'unloading' (出料)
const step = ref('empId');
let cmd236_flag = false;

// 【PLC-新增】: PLC 狀態響應式變數
const up_in = ref(-1);
const up_out = ref(-1);
const dn_in = ref(-1);
const dn_out = ref(-1);
const start_message = ref(-1)

const in_platform = ref('')
const out_platform = ref('')
const _in_platform = computed(() =>{
  if (up_in.value === 1) {
    return "上台面";
  } else if (dn_in.value === 1) {
    return "下台面";
  } else {
    return "---";
  }
});
const _out_platform = computed(() => {
  if (_in_platform.value === "上台面") {
    return "下台面";
  } else if (_in_platform.value === "下台面") {
    return "上台面";
  } else {
    return "---";
  }
});

const STORAGE_KEY = 'lpsm_v1_session_state';

const leftScanHistory = ref([]);
const rightScanHistory = ref([]);

const D_work_step = ref('');
const D_sht_no = [];
const D_panel_no = [];
const D_twodid_step = [];
const D_twodid_type = [];
const D_twodid_input = [];
const D_panel_num = ref('');
const input_num = ref(0); // 這裡也需要一個 ref 變數來追蹤投入張數
// 新增用於顯示的響應式變數
const _panel_num = computed(() => D_panel_num.value);
const _input_num = computed(() => input_num.value);

// 用於儲存作業區中每個 camera 的產品碼，以處理出料模式的重複掃描判斷
const inProcessPdcodeTracker = ref({
  left: '',
  right: '',
});


const cameras = ref([
  { id: 'PL', position: '左', pdcode: '', status: 'waiting', area: 'process' },
  { id: 'PR', position: '右', pdcode: '', status: 'waiting', area: 'process' },
  { id: 'SL', position: '左', pdcode: '', status: 'waiting', area: 'staging', workOrder: '', productItem: '', station: '', ngReason: '' },
  { id: 'SR', position: '右', pdcode: '', status: 'waiting', area: 'staging', workOrder: '', productItem: '', station: '', ngReason: '' }
]);

const inProcessCameras = computed(() => cameras.value.filter(c => c.area === 'process'));
const stagingCameras = computed(() => cameras.value.filter(c => c.area === 'staging'));
const inProcessLeftCamera = computed(() => inProcessCameras.value.find(c => c.position === '左'));
const inProcessRightCamera = computed(() => inProcessCameras.value.find(c => c.position === '右'));
const stagingLeftCamera = computed(() => stagingCameras.value.find(c => c.position === '左'));
const stagingRightCamera = computed(() => stagingCameras.value.find(c => c.position === '右'));
const isBaseInfoFilled = computed(() => !!(employeeId.value && workOrder.value && employeeName.value && productItem.value));
// 完成按鈕的啟用條件：基礎資訊填寫完畢，且所有放料區的狀態都是成功 (OK) 或等待 (waiting)
// 確保在 'loading' 模式下，兩邊都 OK 才能完成；在 'unloading' 模式下，作業區都清空才能完成
const isReadyToComplete = computed(() => {
  if (!isBaseInfoFilled.value) return false;

  // 如果目前是待入料模式，則必須放料區都 OK 才能完成
  if (stagingMode.value === 'loading') {
    return stagingLeftCamera.value.status === 'success' && stagingRightCamera.value.status === 'success';
  } else { // 如果目前是出料模式，則必須作業區都已清空才能完成
    return inProcessLeftCamera.value.pdcode === '---' && inProcessRightCamera.value.pdcode === '---';
  }
});

const in_left_pdcode = ref("")
const in_right_pdcode = ref("")

const stagingModeText = computed(() => stagingMode.value === 'loading' ? '待入料' : '出料');
const forceFocus = () => nextTick(() => universalInput.value?.focus());

let socket = null;

// Alert 自訂
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


// 【PLC-新增】: 發送步驟更新指令到後端
function sendStepUpdate(newVal) {
    if (socket && socket.readyState === WebSocket.OPEN) {
      if (newVal === "OK" || newVal === "NG") {
        const ret = newVal === "OK" ? 1 : 0;
        const controlMessage = { type: 'control', command: 'GO_NOGO', payload: (newVal === "OK") ? 1 : 0};
        console.log("[WS發送] CONTROL", controlMessage);
        socket.send(JSON.stringify(controlMessage));
        console.log(`[WS發送] STEP_UPDATE: ${newVal}. 後端 PLC 監控狀態將更新。`);
      } else {
        // 發送控制指令到後端, e.g. payload =  'empId', 'workOrder', 'twoDID'
        const controlMessage = { type: 'control', command: 'STEP_UPDATE', payload: newVal };
        console.log("[WS發送] CONTROL", controlMessage);
        socket.send(JSON.stringify(controlMessage));
        console.log(`[WS發送] STEP_UPDATE: ${newVal}. 後端 PLC 監控狀態將更新。`);
      }

    } else {
        // 如果連線尚未建立或已關閉，則不發送
        // console.warn(`[WS發送] WebSocket 尚未開啟，STEP_UPDATE [${newVal}] 無法發送。`);
    }
}


// --- WebSocket Logic ---
function connectWebSocket() {
  const wsUrl = 'ws://127.0.0.1:8181';
  socket = new WebSocket(wsUrl);

  socket.onopen = () => {
    console.log("WebSocket: 連接成功！");
    // 連線開啟後，發送一次初始 step.value，確保 PLC 任務狀態同步
    sendStepUpdate(step.value); 
  };

  socket.onmessage = (event) => {
    try {
      const message = JSON.parse(event.data);
      console.log("WebSocket: 收到訊息:", message);

      if (message.type === 'data') {
        if (message.source === 'PLC_MONITOR') {
            // 【PLC-修改】: 處理 PLC 資料
            const plcPayload = message.payload; // { M503: 0/1, M506: 0/1 }
            up_in.value = plcPayload.up_in;
            up_out.value = plcPayload.up_out;
            //console.log(`M503：${plc_M503.value}`)
            //console.log(`M506：${plc_M506.value}`)
            dn_in.value = plcPayload.dn_in
            dn_out.value = plcPayload.dn_out
            //console.log(`M542：${plc_M542.value}`)
            //console.log(`M545：${plc_M545.value}`)
            start_message.value = plcPayload.start_message
        } else if (message.source === 'LEFT_GROUP_MONITOR' || message.source === 'RIGHT_GROUP_MONITOR') {
            processScan(message.payload, message.source);
        } else if (message.payload) {
            // 處理 TCPIP 或 SCANNER 來的掃碼數據
            processScan(message.payload, message.source);
        }
      } else if (message.type === 'command' && message.payload === 'RESET') {
        Resetll();
      } else if (message.type === 'info') {
        console.log("WebSocket Info:", message.message);
      }
    } catch (error) {
      console.error("WebSocket: 解析訊息失敗", error);
    }
  };

  socket.onclose = () => {
    console.log("WebSocket: 連接已關閉，3秒後嘗試重連...");
    socket = null;
    // 【PLC-修改】: 連線中斷時也將 PLC 狀態顯示清空/設為 0
    up_in.value = -1;
    up_out.value = -1;
    dn_in.value = -1;
    dn_out.value = -1;
    start_message.value = -1;
    setTimeout(connectWebSocket, 3000);
  };

  socket.onerror = (error) => {
    console.error("WebSocket: 發生錯誤", error);
  };
}

// 【PLC-新增】: 監聽 step.value 變動，發送控制指令給後端
watch(step, (newVal) => {
    sendStepUpdate(newVal);
});
// 【PLC-新增】: 監聽 start_message 變動，發送控制指令給後端
watch(start_message, (newVal) => {
  console.log(`監控點位 start_message: ${newVal}`)
  if (newVal === 1) {
    if (stagingLeftCamera.value.status === 'success' && stagingRightCamera.value.status === 'success') {
      sendStepUpdate("OK");
    } else {
      sendStepUpdate("NG");
    }
  }
});

// --- Event Handlers ---
const handleManualInput = () => {
  const inputValue = scannedData.value;
  if (!inputValue) return;
  processScan(inputValue, 'SCANNER');
  scannedData.value = '';
}

const API_URL = "http://10.1.5.122/gxfirstOIS/gxfirstOIS.asmx/GetOISData";
const OIS_API_BASE = "http://10.1.5.124:2151/api";

// --- Validation Functions (Full Implementation) ---
async function validateEmpId(empId) {
  try {
    const requestData = new URLSearchParams()
    requestData.append("CmdCode","5")
    requestData.append("InMessage_Json", JSON.stringify({ Emp_NO: empId }))
    const response = await fetch(API_URL,{
      method: "POST",
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body: requestData.toString()
    });
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

async function validateWorkOrder(workOrderInput) {
  const payload = { emp_no: employeeId.value, workorder: workOrderInput };
  try {
    // 優先檢查 Read_2DID
    let response_tmp = await fetch(`${OIS_API_BASE}/Read_2DID`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
    let json_tmp = await response_tmp.json();
    
    if (json_tmp.success && json_tmp.data) {
      const pdcodedata = json_tmp.data.pdcode_data;
      cmd236_flag = pdcodedata.cmd236_Flag;
      return {
        workorder: pdcodedata.workOrder,
        partno: pdcodedata.item,
        work_step: pdcodedata.work_step,
        sht_no: pdcodedata.sht_no,
        panel_no: pdcodedata.panel_no,
        twodid_step: pdcodedata.twodid_step,
        twodid_type: pdcodedata.twodid_type,
        twodid_input: pdcodedata.twodid_input,
        panel_num: pdcodedata.panel_num
      };
    }

    // 如果 Read_2DID 沒有資料，則嘗試 workorder
    let response = await fetch(`${OIS_API_BASE}/workorder`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
    let json = await response.json();
    console.log("workorder json: ", json)
    cmd236_flag = false; // 預設為 false，如果走 workorder2 才會變 true

    if (json.success && json.result.result.slice(0, 2) == 'OK'){
      const lines = json.result.result.slice(3).split('\r\n')
      const info = lines[0].split(";", 3);
      let workorder = workOrderInput, partno = info[0], work_step = info[1];
      let sht_no = [], panel_no = [], twodid_step = [], twodid_type = [], twodid_input = [];
      console.log(lines)
      for (const line of lines) {
        const parts = line.split(';');
        sht_no.push(parts[2]);
        panel_no.push(parts[3]);
        twodid_step.push(parts[4]);
        twodid_type.push(parts[5]);
        twodid_input.push("N");
      }
      const panel_num = new Set(panel_no).size;
      return { workorder, partno, work_step, sht_no, panel_no, twodid_step, twodid_type, twodid_input, panel_num };
    }
    else {
      // 如果 workorder 也失敗，嘗試 workorder2
      cmd236_flag = true;
      const response2 = await fetch(`${OIS_API_BASE}/workorder2`, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload)});
      const json2 = await response2.json();
      if (json2.success) {
        const lines = (json2.result.result.slice(0, 2) == "OK") ? json2.result.result.slice(3).split('\r\n') : json2.result.result.split('\r\n')
        const info = lines[0].split(";", 9);
        let workorder = workOrderInput, partno = info[1], work_step = info[2], panel_num = info[8];
        let sht_no = [], panel_no = [], twodid_step = [], twodid_type = [], twodid_input = [];
        for (const line of lines) {
          const parts = line.split(';');
          sht_no.push(parts[3]);
          panel_no.push(parts[4]);
          twodid_step.push(parts[5]);
          twodid_type.push(parts[6]);
          twodid_input.push("N");
        }
        return { workorder, partno, work_step, sht_no, panel_no, twodid_step, twodid_type, twodid_input, panel_num };
      } else {
        await showCustomModal(`工單驗證失敗：${json2.result?.result || '無有效工單數據'}`);
        return null;
      }
    }
  } catch (error) {
    console.error('validateWorkOrder 發生錯誤:', error);
    await showCustomModal(`工單驗證時發生網路錯誤：${error.message}`);
    return null;
  }
}

async function validateTwoDID(value, source) {
  let ret_type = '', ret_workOrder = '', ret_item = '', ret_station = '';

  const isLeftScan = source.startsWith('CAMERA_LEFT');
  const isRightScan = source.startsWith('CAMERA_RIGHT');
  const targetCamera = isLeftScan ? stagingLeftCamera.value : stagingRightCamera.value;
  const inProcessCamera = isLeftScan ? inProcessLeftCamera.value : inProcessRightCamera.value;
  const inProcessTrackerKey = isLeftScan ? 'left' : 'right';

  // --- 入料模式 (loading) 邏輯 ---
  const sht_no_indices = D_sht_no.map((item, index) => (item === value ? index : -1)).filter(index => index !== -1);
  const current_panel_no = sht_no_indices.length > 0 ? D_panel_no[sht_no_indices[0]] : null;

  // 防重複掃描邏輯優化
  // 如果目標放料區已經有產品碼，且與本次掃描相同，則直接忽略 (防止連續掃描同一條碼)
  if (targetCamera.pdcode === value && targetCamera.status !== 'waiting') {
      //console.log(`掃描 [${value}] 已存在於放料區 ${targetCamera.position}，忽略。`);
      return;
  }

  // 檢查是否已達PANEL數
  if (input_num.value >= D_panel_num.value) {
    ret_type = "NG；已達PANEL數，無法綁定";
    return { ret_type, workOrder: "---", item: "---", station: "---", actionType: "進" };
  }

  // --- 出料模式 (unloading) 邏輯 ---
  if (stagingMode.value === "unloading") {
    console.log(`in_left_pdcode : ${inProcessTrackerKey}`)
    if (inProcessTrackerKey === "left") {
      if (in_left_pdcode.value === value) {
        targetCamera.status = 'success'
        in_left_pdcode.value = ""
        if (in_left_pdcode.value === "" && in_right_pdcode.value === "") {
          stagingMode.value = "loading"
        }
        return { ret_type: "OK", workOrder: "---", item: "---", station: "---", actionType: "出" }
      } 
      else {
        console.log("STEP1")
        targetCamera.status = 'error'
        return { ret_type: "NG", workOrder: "---", item: "---", station: "---", actionType: "出" }
      }
    } else {
      if (in_right_pdcode.value === value) {
        targetCamera.status = 'success'
        in_right_pdcode.value = ""
        if (in_left_pdcode.value === "" && in_right_pdcode.value === "") {
          stagingMode.value = "loading"
        }
        return { ret_type: "OK", workOrder: "---", item: "---", station: "---", actionType: "出" }
      } else {
        console.log("STEP2")
        targetCamera.status = 'error'
        return { ret_type: "NG", workOrder: "---", item: "---", station: "---", actionType: "出" }
      }
      
    }
    /*
    // 檢查掃描到的條碼是否與作業區中的產品碼匹配
    if (inProcessCamera.pdcode === value) {
      // 匹配成功，表示該產品已出料
      // 將作業區的狀態更新為等待，清空產品碼
      inProcessCamera.pdcode = '---';
      inProcessCamera.status = 'waiting';
      inProcessPdcodeTracker.value[inProcessTrackerKey] = ''; // 清空追蹤器

      // 更新放料區的顯示，表示該位置已完成出料
      targetCamera.pdcode = value; // 顯示出料的條碼
      targetCamera.status = 'success'; // 標記為成功出料

      // 紀錄 LOG
      const logData = {
        "person": employeeName.value,
        "empId": employeeId.value,
        "workOrder": workOrder.value,
        "item": productItem.value,
        "barcode": value,
        "others": "---",
        "NG_workOrder": "---",
        "NG_item": "---",
        "panel_num": D_panel_num.value,
        "input_num": input_num.value, // 出料模式下，input_num 不變
        "cmd236_flag": cmd236_flag,
        "work_step": D_work_step.value,
        "sht_no": D_sht_no,
        "panel_no": D_panel_no,
        "twodid_step": D_twodid_step,
        "twodid_type": D_twodid_type,
        "twodid_input": D_twodid_input,
        "ret_type": "OK", // 出料成功
        "n_panel": value,
        "n_time": new Date(),
        "remark": "出料"
      };

      await fetch(`${OIS_API_BASE}/2DID_LOG`, {
        method: "POST",
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ workorder: workOrder.value, cmd236_Flag: cmd236_flag, pdcode_data: logData })
      });

      return { ret_type: "OK", workOrder: "---", item: "---", station: "---", actionType: "出" };
    } else {
      // 如果不匹配，則判定為 NG (混料或錯誤條碼)
      let ngReason = "NG; 掃描條碼與作業區不符";
      // 可以進一步查詢條碼資訊，提供更詳細的NG原因
      const payload = { emp_no: employeeId.value, twodid: value };
      try {
        const response = await fetch(`${OIS_API_BASE}/twodid`, {
          method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload)
        });
        const json = await response.json();
        if (json.success) {
          const [_, wo, item, station] = json.result.result.split(";");
          ret_workOrder = wo; ret_item = item; ret_station = station;
          if (ret_workOrder !== workOrder.value) ngReason += ";混工單";
          if (ret_item !== productItem.value) ngReason += ";混品目";
        }
      } catch (error) { console.error("validateTwoDID 的 twodid API 呼叫失敗:", error); }

      targetCamera.pdcode = value;
      targetCamera.status = 'error';
      targetCamera.ngReason = ngReason;
      targetCamera.workOrder = ret_workOrder || 'N/A';
      targetCamera.productItem = ret_item || 'N/A';
      targetCamera.station = ret_station || 'N/A';

      // 紀錄 NG LOG
      const logData = {
        "person": employeeName.value,
        "empId": employeeId.value,
        "workOrder": workOrder.value,
        "item": productItem.value,
        "barcode": value,
        "others": ngReason,
        "NG_workOrder": ret_workOrder,
        "NG_item": ret_item,
        "panel_num": D_panel_num.value,
        "input_num": input_num.value,
        "cmd236_flag": cmd236_flag,
        "work_step": D_work_step.value,
        "sht_no": D_sht_no,
        "panel_no": D_panel_no,
        "twodid_step": D_twodid_step,
        "twodid_type": D_twodid_type,
        "twodid_input": D_twodid_input,
        "ret_type": ngReason,
        "n_panel": value,
        "n_time": new Date(),
        "remark": "出料NG"
      };
      await fetch(`${OIS_API_BASE}/2DID_LOG`, {
        method: "POST",
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ workorder: workOrder.value, cmd236_Flag: cmd236_flag, pdcode_data: logData })
      });
      return { ret_type: ngReason, workOrder: ret_workOrder, item: ret_item, station: ret_station, actionType: "出" };
    }
    */
  }


  if (sht_no_indices.length === 0) {
    ret_type = "NG";
    const payload = { emp_no : employeeId.value, twodid: value };
    try {
      const response = await fetch(`${OIS_API_BASE}/twodid`, {
        method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload)
      });
      const json = await response.json();
      if (json.success) {
        const [_, wo, item, station] = json.result.result.split(";");
        ret_workOrder = wo; ret_item = item; ret_station = station;
      }
    } catch(error) { console.error("validateTwoDID 的 twodid API 呼叫失敗:", error); }

    if (ret_workOrder !== workOrder.value) ret_type += ";混工單";
    if (ret_item !== productItem.value) ret_type += ";混品目";
    return { ret_type, workOrder: ret_workOrder, item: ret_item, station: ret_station, actionType: "進" };
  }

  const current_twodid_type = D_twodid_type[sht_no_indices[0]];
  if (current_twodid_type === "Y") {
    ret_type = "NG;跳站";
    return { ret_type, workOrder: "---", item: "---", station: "---", actionType: "進" };
  }

  const current_twodid_input = D_twodid_input[sht_no_indices[0]];
  const current_twodid_step = D_twodid_step[sht_no_indices[0]];

  if (current_twodid_input === "Y" || D_work_step.value === current_twodid_step) {
    ret_type = "NG;重工";
    return { ret_type, workOrder: "---", item: "---", station: "---", actionType: "進" };
  }

  // 如果到這裡都 OK，則視為入料成功
  return { ret_type: "OK", workOrder: "---", item: "---", station: "---", actionType: "進" };
}

/**
 * 處理來自後端監控器 (LEFT_GROUP_MONITOR / RIGHT_GROUP_MONITOR) 的空值訊號。
 * @param {string} source - 資料來源 (例如：'LEFT_GROUP_MONITOR')
 */
const handleTimeoutBlank = (source) => {
    let targetCamera = null;
    let targetCameraName = "";
    console.log("handleTimeBlank source: ", source)

    // 判斷來源並清空對應的 staging/放料區
    if (source.startsWith('LEFT_GROUP')) {
        targetCamera = stagingLeftCamera.value;
        targetCameraName = "左側放料區";
    } else if (source.startsWith('RIGHT_GROUP')) {
        targetCamera = stagingRightCamera.value;
        targetCameraName = "右側放料區";
    } else {
        //console.warn(`[Timeout] 收到未知來源的 TIMEOUT_BLANK: ${source}`);
        return;
    }

    console.log("targetCamera: ", targetCamera)
    console.log("targetCameraName: ", targetCameraName)

    // 只有在當前是 'waiting' 或 'success' 狀態時才進行清空 (error 狀態應維持)
    if (targetCamera.status === 'success' || targetCamera.status === 'waiting') {
        targetCamera.pdcode = targetCamera.pdcode === '' ? '' : '等待掃描...'; // 保持為 '等待掃描...'
        targetCamera.status = 'waiting'; // 設為等待狀態
        targetCamera.ngReason = '';
        targetCamera.workOrder = '';
        targetCamera.productItem = '';
        targetCamera.station = '';
        //console.log(`[Timeout] ${targetCameraName} 已被空值訊號清空/重置為 waiting。`);
    } else {
        //console.log(`[Timeout] ${targetCameraName} 為 NG 狀態，不執行空值清空。`);
    }
};

// --- Main Process Logic ---
const processScan = async (data, source) => {
  const value = String(data).replace(/[\x00-\x1F\x7F-\x9F]/g, "").trim().toUpperCase();
  //console.log(`目前區域:${step.value} 資料:${value}`)
  
  // ★★★ 關鍵修改 1: 優先處理 TIMEOUT_BLANK 空值訊號 ★★★
  if (value === "TIMEOUT_BLANK") {
      // payload 是 TIMEOUT_BLANK，調用專門的清空函式
      handleTimeoutBlank(source); 
      return; // 處理完畢，結束此函數
  }

  if (!value) return;
  console.log("value: ", value)

  if (step.value === 'empId') {
    const result = await validateEmpId(value);
    if (result) {
      employeeId.value = result.empNo;
      employeeName.value = result.empName;
      if (workOrder.value === "" ) {
        step.value = 'workOrder';
      } else {
        step.value = 'twoDID'; // <--- 觸發 watch 發送啟動 PLC 指令
      }
      
    }
    scannedData.value = '';
    forceFocus();
    return;
  }
  
  if (step.value === 'workOrder') {
    if (value.length < 8 || value.length > 11) {
      await showCustomModal("工單格式不正確，請重新掃描。");
      scannedData.value = '';
      forceFocus();
      return;
    }
    const valid = await validateWorkOrder(value);
    if (valid && valid.work_step !== "") {
      workOrder.value = valid.workorder;
      productItem.value = valid.partno;
      D_work_step.value = valid.work_step;
      D_sht_no.length = 0; D_sht_no.push(...valid.sht_no);
      D_panel_no.length = 0; D_panel_no.push(...valid.panel_no);
      D_twodid_step.length = 0; D_twodid_step.push(...valid.twodid_step);
      D_twodid_type.length = 0; D_twodid_type.push(...valid.twodid_type);
      D_twodid_input.length = 0; D_twodid_input.push(...valid.twodid_input);
      D_panel_num.value = valid.panel_num;
      step.value = 'twoDID'; // <--- 觸發 watch 發送啟動 PLC 指令
    }
    scannedData.value = '';
    forceFocus();
    return;
  }

  if (step.value === 'twoDID') {

    if (value.length !== 13 && value.length !== 0) {
      await showCustomModal("產品條碼長度不正確，請重新掃描。");
      scannedData.value = '';
      forceFocus();
      return;
    }

    let targetSlot = null;
    const isLeftScan = source.startsWith('CAMERA_LEFT');
    const isRightScan = source.startsWith('CAMERA_RIGHT');

    // 確定掃描目標槽位
    if (isLeftScan || source === 'SCANNER_LEFT') { // source.startsWith('CAMERA_LEFT')
        targetSlot = stagingLeftCamera.value;
    } else if (isRightScan || source === 'SCANNER_RIGHT') { // source.startsWith('CAMERA_RIGHT')
        targetSlot = stagingRightCamera.value;
    } else {
      // 模擬手動掃描時，如果左右都有待掃描，優先左邊 (原有的邏輯適用於 SCANNER 原始碼)
      targetSlot = stagingLeftCamera.value.status === 'waiting' ? stagingLeftCamera.value :
                   stagingRightCamera.value.status === 'waiting' ? stagingRightCamera.value :
                   stagingLeftCamera.value; 
    }

    // 在執行 validateTwoDID 前，檢查是否重複掃描同一產品碼到同一槽位
    if (targetSlot.pdcode === value && targetSlot.status !== 'waiting' && stagingMode.value === 'loading') {
      //console.log(`警告：在 ${targetSlot.position} 槽位重複掃描相同的產品碼 ${value}。已忽略。`);
      scannedData.value = '';
      forceFocus();
      return;
    }
    // 在出料模式下，也需要檢查是否重複掃描相同的產品碼
    if (stagingMode.value === 'unloading') {
      const inProcessCamera = isLeftScan ? inProcessLeftCamera.value : inProcessRightCamera.value;
      if (inProcessCamera.pdcode === '---') {
        // 如果作業區已經沒有東西，但又掃描了，可能是想掃其他條碼，此時不應直接忽略，而是進行驗證判斷 NG
      } else if (inProcessPdcodeTracker.value[targetSlot.position === '左' ? 'left' : 'right'] === value && targetSlot.status === 'success') {
        //console.log(`警告：在 ${targetSlot.position} 槽位重複掃描相同的產品碼 ${value} (已出料)。已忽略。`);
        scannedData.value = '';
        forceFocus();
        return;
      }
    }

    const validationResult = await validateTwoDID(value, source);

    if (!validationResult) {
      //console.log(`掃描 [${value}] 被 validateTwoDID 內部邏輯忽略。`);
      scannedData.value = ''; // 清空輸入框
      forceFocus(); // 保持輸入框焦點
      return;       // 直接退出 processScan 函式
    }

    // 更新放料區的顯示狀態
    targetSlot.pdcode = value;
    targetSlot.ngReason = validationResult.ngReason;
    targetSlot.workOrder = validationResult.workOrder;
    targetSlot.productItem = validationResult.item;
    targetSlot.station = validationResult.station;

    if (validationResult.ret_type === 'OK') {
      targetSlot.status = 'success';
      /*  2025.11.25 建立清單改道移入
      // 只有在 "待入料" 模式且掃描成功時才需要寫入 MES 和更新 input_num
      if (stagingMode.value === 'loading') {
        const sht_no_indices = D_sht_no.map((item, index) => (item === value ? index : -1)).filter(index => index !== -1);
        const current_panel_no = D_panel_no[sht_no_indices[0]];
        const panel_no_indices = D_panel_no.map((item, index) => (item === current_panel_no ? index : -1)).filter(index => index !== -1);

        for (const index of panel_no_indices) {
          const payload2 = {
            emp_no: employeeId.value, workOrder: workOrder.value, partno: productItem.value,
            work_step: D_work_step.value, sht_no: D_sht_no[index], panel_no: D_panel_no[index],
            twodid_type: "OK", remark: "進料"
          };
          try {
            const response = await fetch(`${OIS_API_BASE}/write2did`, {
              method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload2)
            });
            const json = await response.json();
            if (!json.success) {
               await showCustomModal(`寫入MES失敗: ${json.message || '未知錯誤'}`);
               targetSlot.status = 'error'; // 如果寫入MES失敗，即使驗證OK也標記為NG
               validationResult.actionType = "進"; // 確保記錄為進料動作
               validationResult.ret_type = "NG;寫入MES失敗";
               targetSlot.ngReason = "寫入MES失敗";
               break; // 停止後續寫入
            }
          } catch (error) {
            console.error("寫入MES發生錯誤:", error);
            await showCustomModal(`寫入MES時發生網路錯誤: ${error.message}`);
            targetSlot.status = 'error'; // 如果寫入MES失敗，即使驗證OK也標記為NG
            validationResult.actionType = "進"; // 確保記錄為進料動作
            validationResult.ret_type = "NG;寫入MES失敗";
            targetSlot.ngReason = "寫入MES失敗";
            break;
          }
        }
        if (targetSlot.status === 'success') { // 確保只有在 MES 寫入成功後才更新狀態和計數
            for (const index of panel_no_indices) {
                D_twodid_input[index] = "Y";
            }
            input_num.value += 1;
        }
      }
      */
    } else {
      targetSlot.status = 'error';
    }
    
    // 記錄到歷史清單
    const historyItem = {
      uid: Date.now() + Math.random(),
      pdcode: targetSlot.pdcode,
      status: targetSlot.status,
      timestamp: new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit', second: '2-digit' }),
      actionType: validationResult.actionType || (stagingMode.value === 'loading' ? '進' : '出'), // 根據實際動作設定
      ngReason: targetSlot.ngReason,
      workOrder: targetSlot.workOrder,
      productItem: targetSlot.productItem,
      station: targetSlot.station,
    };
    
    if (targetSlot.position === '左') {
      leftScanHistory.value.unshift(historyItem);
      if (leftScanHistory.value.length > 50) leftScanHistory.value.pop();
    } else {
      rightScanHistory.value.unshift(historyItem);
      if (rightScanHistory.value.length > 50) rightScanHistory.value.pop();
    }
    
    // 如果是出料成功，且作業區兩個槽位都已清空，則將模式切換回待入料
    if (stagingMode.value === 'unloading' && inProcessLeftCamera.value.pdcode === '---' && inProcessRightCamera.value.pdcode === '---') {
      // stagingMode.value = 'loading'; // 此處由 complete 統一處理
      // showCustomModal("所有產品已成功出料！"); // 提示完成
    }
  }

  scannedData.value = ''; // 清空輸入框
  forceFocus();
};

const Resetll = async() => {
  employeeId.value = '';
  employeeName.value = '';
  workOrder.value = '';
  productItem.value = '';
  stagingMode.value = 'loading';
  step.value = 'empId'; // <--- 觸發 watch 發送停止 PLC 指令
  cmd236_flag = false;

  D_work_step.value = '';
  D_sht_no.length = 0;
  D_panel_no.length = 0;
  D_twodid_step.length = 0;
  D_twodid_type.length = 0;
  D_twodid_input.length = 0;
  D_panel_num.value = '';
  input_num.value = 0;

  leftScanHistory.value = [];
  rightScanHistory.value = [];

  cameras.value.forEach(c => Object.assign(c, { pdcode: '', status: 'waiting', workOrder: '', productItem: '', station: '', ngReason: '' }));
  inProcessPdcodeTracker.value = { left: '', right: '' }; // 重置追蹤器
  
  // 【PLC-新增】: 重置時清空 PLC 狀態顯示
  up_in.value = -1;
  up_out.value = -1;
  dn_in.value = -1;
  dn_out.value = -1;
  start_message.value = -1;
  in_platform.value = "";
  out_platform.value = "";
  forceFocus();
}

/**
 * 抓取所有需要保存的響應式狀態（排除不變的如 history 和輔助變數）
 */
function getSavableState() {
    return {
        // 核心業務數據
        employeeId: employeeId.value,
        employeeName: employeeName.value,
        workOrder: workOrder.value,
        productItem: productItem.value,
        stagingMode: stagingMode.value,
        step: step.value,
        cmd236_flag: cmd236_flag,
        // 工單清單 (直接儲存陣列內容)
        D_work_step: D_work_step.value,
        D_sht_no_copy: D_sht_no.slice(), // 使用 slice 複製陣列
        D_panel_no_copy: D_panel_no.slice(),
        D_twodid_step_copy: D_twodid_step.slice(),
        D_twodid_type_copy: D_twodid_type.slice(),
        D_twodid_input_copy: D_twodid_input.slice(),
        D_panel_num: D_panel_num.value,
        input_num: input_num.value,
        // 視覺區塊狀態
        cameras_copy: JSON.parse(JSON.stringify(cameras.value)), // 深度複製 cameras
        inProcessPdcodeTracker_copy: JSON.parse(JSON.stringify(inProcessPdcodeTracker.value)),
        // 歷史紀錄 (雖然不屬於核心數據，但可提高體驗，可以選擇性儲存)
        leftScanHistory_copy: leftScanHistory.value.slice(),
        rightScanHistory_copy: rightScanHistory.value.slice(),
        
        // PLC狀態不用儲存，因為它會立即被後端開始輪詢後的值覆蓋
    };
}

/**
 * 載入並恢復狀態
 */
function restoreState() {
    const savedState = sessionStorage.getItem(STORAGE_KEY);
    if (!savedState) return;

    try {
        const state = JSON.parse(savedState);
        
        // 恢復核心數據
        employeeId.value = state.employeeId || '';
        employeeName.value = state.employeeName || '';
        workOrder.value = state.workOrder || '';
        productItem.value = state.productItem || '';
        stagingMode.value = state.stagingMode || 'loading';
        step.value = state.step || 'empId';
        cmd236_flag = state.cmd236_flag || false;
        
        // 恢復工單陣列 (清空並填入)
        D_work_step.value = state.D_work_step || '';
        D_sht_no.length = 0; D_sht_no.push(...(state.D_sht_no_copy || []));
        D_panel_no.length = 0; D_panel_no.push(...(state.D_panel_no_copy || []));
        D_twodid_step.length = 0; D_twodid_step.push(...(state.D_twodid_step_copy || []));
        D_twodid_type.length = 0; D_twodid_type.push(...(state.D_twodid_type_copy || []));
        D_twodid_input.length = 0; D_twodid_input.push(...(state.D_twodid_input_copy || []));
        D_panel_num.value = state.D_panel_num || '';
        input_num.value = state.input_num || 0;

        // 恢復 Cameras
        if (state.cameras_copy) {
            cameras.value = state.cameras_copy;
        }
        if (state.inProcessPdcodeTracker_copy) {
            inProcessPdcodeTracker.value = state.inProcessPdcodeTracker_copy;
        }

        // 恢復歷史紀錄
        leftScanHistory.value = state.leftScanHistory_copy || [];
        rightScanHistory.value = state.rightScanHistory_copy || [];

        // 【PLC 連線恢復】：透過修改 step.value 會觸發 watch(step, ...)
        // 然後 watch 會執行 sendStepUpdate(step.value)，將最新的 step 狀態通知給後端，後端將恢復 PLC 輪詢。
        
        // 清除儲存，以備下次全新開始
        sessionStorage.removeItem(STORAGE_KEY);
        console.log('[狀態] 成功從 Session Storage 恢復狀態並通知後端。');
    } catch (e) {
        console.error('[狀態] 恢復狀態失敗:', e);
        sessionStorage.removeItem(STORAGE_KEY);
    }
}

const handleRestUser = async () => {
  step.value = 'empId'
  employeeId.value = "---";
  employeeName.value = "---";
}

const handleResetAll = async () => {
  // 此處需改成重新整理
  // Resetll();
  const currentState = getSavableState();
  sessionStorage.setItem(STORAGE_KEY, JSON.stringify(currentState));

  window.location.reload();
};

const handleComplete = async () => {
  // 檢查完成條件
  if (stagingMode.value === 'loading') {
    if (stagingLeftCamera.value.status !== 'success' || stagingRightCamera.value.status !== 'success') {
      await showCustomModal("請確保左右放料區都已成功掃描 (OK) 才能完成！");
      forceFocus();
      return;
    }
  } else if (stagingMode.value === 'unloading') {
    if (inProcessLeftCamera.value.pdcode !== '---' || inProcessRightCamera.value.pdcode !== '---') {
      await showCustomModal("請確保作業區中的所有產品都已成功出料！");
      forceFocus();
      return;
    }
  }

  const confirmed = await showCustomModal(
    `品目panel數: ${_panel_num.value}，投入數: ${_input_num.value}。確定要結束當前作業嗎？`,
    'confirm'
  );
  if (!confirmed) {
    forceFocus();
    return;
  }

  // 僅在待入料模式結束時，處理未投入的產品為 NG
  if (cmd236_flag === false && D_sht_no.length > 0 && stagingMode.value === 'loading') {
    for (let i = 0; i < D_sht_no.length; i++) {
      if (D_twodid_input[i] === "N") {
        const payload = {
          emp_no: employeeId.value,
          workOrder: workOrder.value,
          partno: productItem.value,
          work_step: D_work_step.value,
          sht_no: D_sht_no[i],
          panel_no: D_panel_no[i],
          twodid_type: "NG",
          remark: "未投入，作業結束"
        };
        try {
          await fetch(`${OIS_API_BASE}/write2did`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
          });
        } catch (error) {
          console.error("寫入未投入 NG 狀態失敗:", error);
        }
      }
    }
  }

  // 清除 2DID 資料
  if (workOrder.value) { // 確保有工單才執行清除
    const payloadDelete = { workorder: workOrder.value };
    try {
      await fetch(`${OIS_API_BASE}/Delete_2DID`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payloadDelete)
      });
    } catch (error) {
      console.error("清除 2DID 資料失敗:", error);
    }
  }

  await showCustomModal("作業已完成！");
  Resetll(); // 重置所有狀態
};


const handleToggle = async () => {
  // 如果是待入料模式，切換到出料模式前必須滿足條件
  if (stagingMode.value === "loading") {
    const isLeftReady = stagingLeftCamera.value.status === "success";
    const isRightReady = stagingRightCamera.value.status === "success";

    if (!isLeftReady || !isRightReady) {
      await showCustomModal("錯誤：在『待入料』模式下，必須左右兩邊的放料區都掃描成功 (OK) 才能切換到『出料』模式！");
      forceFocus();
      return;
    }

    // 步驟 A: 將放料區的成功品資料 "移入" 作業區
    inProcessLeftCamera.value.pdcode = stagingLeftCamera.value.pdcode;
    inProcessLeftCamera.value.status = 'success';
    inProcessRightCamera.value.pdcode = stagingRightCamera.value.pdcode;
    inProcessRightCamera.value.status = 'success';

    // 更新追蹤器
    inProcessPdcodeTracker.value.left = stagingLeftCamera.value.pdcode;
    inProcessPdcodeTracker.value.right = stagingRightCamera.value.pdcode;

    // 步驟 B: 清空放料區 (等待下一批產品入料或出料掃描)
    stagingLeftCamera.value.pdcode = '';
    stagingLeftCamera.value.status = 'waiting';
    stagingRightCamera.value.pdcode = '';
    stagingRightCamera.value.status = 'waiting';

    // 立即切換模式為 'unloading'
    stagingMode.value = 'unloading';
    await showCustomModal("已成功入料，切換為『出料』模式。現在請掃描作業區的產品進行出料。");

  } else { // 如果是出料模式，切換回待入料模式前必須滿足條件
    const isProcessAreaEmpty = inProcessLeftCamera.value.pdcode === '---' && inProcessRightCamera.value.pdcode === '---';
    if (!isProcessAreaEmpty) {
      await showCustomModal("錯誤：在『出料』模式下，必須作業區中的產品都已出料 (作業區顯示 '---') 才能切換回『待入料』模式！");
      forceFocus();
      return;
    }

    // 如果作業區已經清空，可以直接切換回 'loading' 模式
    stagingMode.value = 'loading';
    await showCustomModal("作業區已清空，切換為『待入料』模式。");
  }

  forceFocus();
};

// 2025.11.24 新增判斷，內外平台互換時
async function handlePlatformChange() {
  // console.log(`[平台變更偵測] 作業區: ${newIn} -> 放料區: ${newOut}`);
  // 目前狀態不再 twoDID 時，不判斷這些邏輯
  if (step.value !== "twoDID") {
    return;
  }
  // 當目前尚下台面都在內部時，維持上台面作業，或是停機
  if (up_in === 1 && dn_in === 1 ) {
    return;
  }

  if (inProcessLeftCamera.value.pdcode === "" && inProcessRightCamera.value.pdcode === "") {
    inProcessLeftCamera.value.pdcode = stagingLeftCamera.value.pdcode;
    inProcessLeftCamera.value.status = 'success';
    inProcessRightCamera.value.pdcode = stagingRightCamera.value.pdcode;
    inProcessRightCamera.value.status = 'success';

    stagingLeftCamera.value.pdcode = ""
    stagingLeftCamera.value.status = ""
    stagingRightCamera.value.pdcode = ""
    stagingRightCamera.value.status = ""
  } else {

    in_left_pdcode.value = inProcessLeftCamera.value.pdcode
    in_right_pdcode.value = inProcessRightCamera.value.pdcode

    stagingMode.value = "unloading"

    inProcessLeftCamera.value.pdcode = stagingLeftCamera.value.pdcode;
    inProcessLeftCamera.value.status = 'success';
    inProcessRightCamera.value.pdcode = stagingRightCamera.value.pdcode;
    inProcessRightCamera.value.status = 'success';

    stagingLeftCamera.value.pdcode = ""
    stagingLeftCamera.value.status = ""
    stagingRightCamera.value.pdcode = ""
    stagingRightCamera.value.status = ""
  }

  let input_mes = true
  // 寫入左邊資訊
  if (inProcessLeftCamera.value.pdcode !== "") {
    const sht_no_indices = D_sht_no.map((item, index) => (item === inProcessLeftCamera.value.pdcode ? index : -1)).filter(index => index !== -1);
    const current_panel_no = D_panel_no[sht_no_indices[0]];
    const panel_no_indices = D_panel_no.map((item, index) => (item === current_panel_no ? index : -1)).filter(index => index !== -1);
    for (const index of panel_no_indices) {
      const payload2 = {
        emp_no: employeeId.value, workOrder: workOrder.value, partno: productItem.value,
        twodid_type: "OK", remark: "進料"
      };
      try {
        const response = await fetch(`${OIS_API_BASE}/write2did`, {
          method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload2)
        });
        const json = await response.json();
        if (!json.success) {
          console.error("寫入MES發生錯誤:",error);
          validationResult.actionType = "進";
          validationResult.ret_type = "NG;寫入MES失敗";
          input_mes = false
          break;
        }
      } catch (error) {
        console.error("寫入MES發生錯誤:",error);
        validationResult.actionType = "進";
        validationResult.ret_type = "NG;寫入MES失敗";
        input_mes = false
        break;
      }
    }
    if (input_mes === true) {
      for (const index of panel_no_indices) {
        D_twodid_input[index] = "Y";
      }
      input_num.value += 1;
    }
  }
  // 寫入右邊資訊
  input_mes = true
  if (inProcessRightCamera.value.pdcode !=="") {
    const sht_no_indices = D_sht_no.map((item, index) => (item === inProcessRightCamera.value.pdcode ? index : -1)).filter(index => index !== -1);
    const current_panel_no = D_panel_no[sht_no_indices[0]];
    const panel_no_indices = D_panel_no.map((item, index) => (item === current_panel_no ? index : -1)).filter(index => index !== -1);
    for (const index of panel_no_indices) {
      const payload2 = {
        emp_no: employeeId.value, workOrder: workOrder.value, partno: productItem.value,
        twodid_type: "OK", remark: "進料"
      };
      try {
        const response = await fetch(`${OIS_API_BASE}/write2did`, {
          method: "POST", headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload2)
        });
        const json = await response.json();
        if (!json.success) {
          console.error("寫入MES發生錯誤:",error);
          validationResult.actionType = "進";
          validationResult.ret_type = "NG;寫入MES失敗";
          input_mes = false
          break;
        }
      } catch (error) {
        console.error("寫入MES發生錯誤:",error);
        validationResult.actionType = "進";
        validationResult.ret_type = "NG;寫入MES失敗";
        input_mes = false
        break;
      }
    }
    if (input_mes === true) {
      for (const index of panel_no_indices) {
        D_twodid_input[index] = "Y";
      }
      input_num.value += 1;
    }
  }

  // 記錄到歷史清單
  const historyItem = {
    uid: Date.now() + Math.random(),
    pdcode: targetSlot.pdcode,
    status: targetSlot.status,
    timestamp: new Date().toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit', second: '2-digit' }),
    actionType: validationResult.actionType || (stagingMode.value === 'loading' ? '進' : '出'), // 根據實際動作設定
    ngReason: targetSlot.ngReason,
    workOrder: targetSlot.workOrder,
    productItem: targetSlot.productItem,
    station: targetSlot.station,
  };

}



watch([up_in, dn_in], ([newup_in, newdn_in], [oldup_in, olddn_in]) => {
  if (newup_in === -1 || oldup_in === -1 ) {
    return
  }
  if ((newup_in !== oldup_in) || (newdn_in !== olddn_in)) {
    handlePlatformChange()
  }
}, {immediate: true})

// --- Lifecycle Hooks ---
onMounted(() => {
  restoreState();
  forceFocus();
  connectWebSocket();
});

onUnmounted(() => {
  if (socket) {
    socket.onclose = null;
    socket.close();
    // 【PLC-新增】: 頁面卸載時也將 PLC 狀態顯示清空/設為 0
    up_in.value = -1;
    up_out.value = -1;
    dn_in.value = -1;
    dn_out.value = -1;
    in_platform.value = "";
    out_platform.value = "";
  }
});
</script>

<style>
/* 基礎樣式 */
html,
body,
#app {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  font-family: "Microsoft JhengHei", sans-serif;
}
:root {
  --bg-color: #0f0c29;
  --panel-bg: rgba(21, 22, 46, 0.6);
  --panel-border: rgba(41, 255, 196, 0.3);
  --glow-color: rgba(41, 255, 196, 0.4);
  --text-color: #a9a9d4;
  --primary-color: #29ffc4;
  --success-color: #29ffc4;
  --error-color: #ff1b78;
  --label-color: #a9a9d4;
  --waiting-color: #6c757d;
}

/* 主要容器 */
.scan-system-container {
  width: 100%;
  height: 100%;
  background: linear-gradient(to right, #24243e, #302b63, #0f0c29);
  color: var(--text-color);
}
.hidden-input {
  position: absolute;
  top: -9999px;
  left: -9999px;
}
.main-panel {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  background: var(--panel-bg);
  border: 1px solid var(--panel-border);
  box-shadow: 0 0 30px var(--glow-color);
  backdrop-filter: blur(10px);
  padding: 2vh 2.5rem;
  box-sizing: border-box;
}

/* 頭部、底部、資訊區塊 */
.panel-header,
.panel-footer,
.info-display-section-compact {
  flex-shrink: 0;
}
.system-logo {
  width: 200px; /* 固定寬度 */
  height: 30px; /* 固定高度 */
  
}
.panel-header {
  padding-bottom: 1.5vh;
  border-bottom: 1px solid var(--panel-border);
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.info-display-section-compact {
  padding-top: 1.5vh;
  padding-bottom: 1vh; /* 調整此處，增加與下方內容的間距 */
}
.panel-footer {
  padding-top: 1.5vh; /* 調整此處，稍微增加底部按鈕區塊的間距 */
  border-top: 1px solid var(--panel-border);
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
}

.info-grid-compact {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
}
.info-label {
  font-size: 1.2rem;
  margin-bottom: 0.3rem;
  color: var(--label-color);
}
.info-value {
  font-size: 1.8rem;
  font-weight: bold;
  color: var(--primary-color);
  padding: 0.4rem 0.75rem;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 張數顯示與切換按鈕的容器 */
.toggle-button-wrapper {
  flex-shrink: 0;
  padding: 1.5vh 0;
  display: flex;
  justify-content: space-between; /* 使內容靠左和靠右 */
  align-items: center; /* 垂直居中 */
  gap: 2rem; /* 左右兩個區塊之間的間距 */
}

/* 張數顯示區塊的新樣式 */
.sheet-count-display {
  display: flex;
  align-items: center;
  gap: 0.8rem; /* 標籤與值之間的間距 */
  /* 移除原先的 background, padding, border-radius，直接使用 .info-label 和 .info-value 的樣式 */
}

.sheet-count-display .info-value {
  /* 繼承 info-value 樣式 */
  /* 可以根據需求調整額外的樣式 */
  padding: 0; /* 移除額外 padding */
  background: none; /* 移除背景 */
}

/* 新增的作業區/放料區狀態顯示樣式 */
.process-area-status {
  display: flex;
  align-items: center;
  gap: 0.8rem; /* 標籤與值之間的間距 */
  /* 置中對齊 */
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
}

.process-area-status .info-value {
  /* 繼承 info-value 樣式 */
  padding: 0; /* 移除額外 padding */
  background: none; /* 移除背景 */
}


.btn-toggle {
  background: rgba(140, 100, 255, 0.7);
  color: var(--text-color);
  border: 1px solid var(--panel-border);
  backdrop-filter: blur(5px);
}
.btn-toggle:hover {
  background: rgba(160, 120, 255, 0.9);
  color: #fff;
}


/* 主內容區塊 */
.camera-section-main.adaptive-view {
  flex-grow: 1;
  min-height: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}
.main-content-wrapper {
  display: flex;
  flex-direction: column;
  gap: 1.5vh; /* 將此處的 gap 縮短，減少放料區與下方清單的間距 */
  width: 100%;
  max-width: 1400px; /* 調整此處 */
}

/* 透視效果區 */
.perspective-container {
  perspective: 1500px;
}
.zones-in-perspective {
  display: flex;
  flex-direction: column;
  gap: 2vh; /* 減少作業區與放料區的間距，確保有足夠空間 */
  transform: rotateX(30deg);
  transform-style: preserve-3d;
}
.zone-container {
  border: 1px solid var(--panel-border);
  border-radius: 12px;
  padding: 2vh 1.5rem 1.5vh 1.5rem;
  position: relative;
  /* 確保內容不會頂到邊界 */
  box-sizing: border-box;
}

/* 調整放料區的樣式，使其不壓到上方 */
.zone-container.staging-zone {
  margin-top: 2vh; /* 增加上邊距，使其與上方的作業區有更明確的分隔 */
}


.zone-title {
  position: absolute;
  top: 0;
  left: 2rem;
  transform: translateY(-50%);
  background: #2a284e;
  padding: 0 0.75rem;
  margin: 0;
  color: var(--label-color);
  font-weight: bold;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-size: clamp(0.9rem, 2.2vw, 2.2rem);
  /* 確保背景色能完全覆蓋邊界，防止線條穿透 */
  z-index: 1; /* 確保標題在邊框之上 */
}
.staging-mode-indicator {
  font-size: clamp(1.1rem, 2.2vw, 1.6rem); /* 調整此處 */
  padding: 0.3em 0.8em;
  border-radius: 4px;
  font-weight: bold;
}
.staging-mode-indicator.loading {
  background-color: var(--primary-color);
  color: #000;
}
.staging-mode-indicator.unloading {
  background-color: #9b59b6;
  color: #fff;
}
.zone-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
}

/* 攝影機顯示區塊 */
.camera-block {
  background: rgba(0, 0, 0, 0.3);
  border: 2px solid var(--panel-border);
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.4s ease;
  padding: 1vh 1rem;
}
.camera-block.staging {
  border-width: 4px;
  height: auto; /* 調整此處 */
  min-height: 20vh; /* 縮小放料區塊高度 */
  padding: 2vh 2rem; /* 縮小放料區塊內邊距 */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  overflow-y: auto;
  transition: all 0.4s ease;
}

.camera-block.status-success {
  border-color: var(--success-color);
  box-shadow: 0 0 10px var(--success-color);
}
.camera-block.status-error {
  border-color: var(--error-color);
  box-shadow: 0 0 10px var(--error-color);
}

.block-pdcode {
  word-break: break-all;
  font-weight: bold;
  color: var(--primary-color);
  font-size: clamp(1rem, 5.5vw, 2.5rem);
  font-family: 'Courier New', Courier, monospace;
}
.block-pdcode.large {
  font-size: clamp(2rem, 5.5vw, 3rem); /* 調整此處 */
  margin-bottom: 0.8rem;
}
.camera-block.status-error .block-pdcode {
  color: var(--error-color);
}
.block-label {
  color: var(--label-color);
  font-size: clamp(1rem, 1.5vw, 1.2rem); /* 調整此處 */
  margin-bottom: 1.2rem; /* 調整此處 */
}

.status-text-success {
  color: var(--success-color);
  text-shadow: 0 0 8px rgba(41, 255, 196, 0.6);
}

.status-text-error {
  color: var(--error-color);
  text-shadow: 0 0 8px rgba(255, 27, 120, 0.6);
}

/* 左右 List 樣式 */
.status-display-area {
  flex-shrink: 0;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  box-sizing: border-box;
  align-items: start;
  margin-top: 1vh; /* 將此處由負值調整為正值，增加與上方內容的間距 */
}
.status-list-column {
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid var(--panel-border);
  border-radius: 8px;
  padding: 0.5vh;
  height: 15vh; /* 保持高度，讓清單在有限空間內顯示 */
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}
.status-list {
  width: 100%;
}

.history-item-wrapper {
  border-bottom: 1px solid rgba(41, 255, 196, 0.1);
  padding: 0.8vh 1rem;
  transition: background-color 0.3s;
}
.history-item-wrapper:last-child {
  border-bottom: none;
}
.history-item-wrapper.is-ng-item {
  background-color: rgba(255, 27, 120, 0.15);
  border-color: rgba(255, 27, 120, 0.2);
}

/* ★★★ START: 修改為單行四欄位樣式 ★★★ */
.list-item-top-row {
  display: grid;
  grid-template-columns: auto 1fr auto auto;
  align-items: baseline;
  gap: 1.2rem;
  font-size: clamp(1.1rem, 1.2vw, 1.3rem); 
}
.item-action-type {
  font-weight: bold;
  font-family: 'Courier New', Courier, monospace;
  font-style: italic;
  font-size: 0.9em;
  color: var(--primary-color); /* 預設顏色與條碼一致 */
}
.history-item-wrapper.is-ng-item .item-action-type {
  color: var(--error-color); /* NG時顏色與條碼一致 */
}
/* ★★★ END: 修改為單行四欄位樣式 ★★★ */

.item-timestamp {
  color: var(--label-color);
  font-size: 0.9em;
}

.item-pdcode {
  font-weight: bold;
  color: var(--primary-color);
  font-family: 'Courier New', Courier, monospace;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.history-item-wrapper.is-ng-item .item-pdcode {
  color: var(--error-color);
}

.item-status-text {
  font-weight: bold;
  font-size: 1.1em;
}
.item-status-text.status-ok {
  color: var(--success-color);
}
.item-status-text.status-ng {
  color: var(--error-color);
}

.list-item-ng-details {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem 1rem;
  padding-top: 0.5rem; /* 縮小與上方間距 */
  padding-left: 1rem;
  font-size: 0.85rem;
  color: var(--label-color);
}
.list-item-ng-details span {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 空白佔位符樣式 */
.list-placeholder {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  flex-grow: 1;
  gap: 1vh;
  padding: 0.5vh 0.5rem;
}
.list-item-placeholder {
  height: 30%;
  width: 100%;
  background: rgba(0,0,0,0.15);
  border-radius: 4px;
}

/* 其他通用樣式 */
.initial-prompt {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  gap: 1.5rem;
  width: 100%;
  height: 100%;
  font-size: 1.8rem;
  color: var(--waiting-color);
}
.prompt-icon {
  width: 60px;
  height: 60px;
  fill: currentColor;
}
.system-status {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--success-color);
}
.status-light {
  width: 10px;
  height: 10px;
  background-color: var(--success-color);
  border-radius: 50%;
}
.btn {
  padding: 0.7rem 1.8rem;
  font-size: 1rem;
  border-radius: 8px;
  cursor: pointer;
  border: none;
  transition: all 0.2s ease;
  font-weight: bold;
}
.btn-resetuser {
  background: transparent;
  color: var(--text-color);
  border: 2px solid var(--panel-border);
}
.btn-resetuser:hover {
  background-color: rgba(255,255,255,0.1);
}
.btn-reset {
  background: transparent;
  color: var(--text-color);
  border: 2px solid var(--panel-border);
}
.btn-reset:hover {
  background-color: rgba(255,255,255,0.1);
}
.btn-complete {
  background: var(--primary-color);
  color: #000;
}
.btn-complete:hover {
  box-shadow: 0 0 10px var(--primary-color);
}
.btn:disabled {
  background: var(--waiting-color);
  color: #333;
  cursor: not-allowed;
  opacity: 0.6;
}
.custom-modal {
  border: 1px solid var(--panel-border);
  background: var(--panel-bg);
  box-shadow: 0 0 40px var(--glow-color);
  backdrop-filter: blur(15px);
  color: var(--text-color);
  border-radius: 12px;
  padding: 2.5rem 3.5rem;
  width: clamp(300px, 60vw, 700px); /* 響應式寬度 */
  max-width: 90%;
  position: fixed; /* 確保它在最上層 */
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1000; /* 確保在其他內容之上 */
}

/* 模態框開啟時的背景疊加層 */
.custom-modal::backdrop {
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(8px);
}

.modal-content {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  align-items: center;
}

.modal-message {
  font-size: clamp(1.5rem, 3.5vw, 2.5rem); /* 大幅放大文字 */
  font-weight: bold;
  text-align: center;
  line-height: 1.4;
  color: var(--primary-color);
  text-shadow: 0 0 10px rgba(41, 255, 196, 0.5);
}

.modal-actions {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  width: 100%;
}

.btn-modal-confirm {
  background: var(--primary-color);
  color: #000;
  padding: 0.9rem 2.5rem;
  font-size: clamp(1.1rem, 2vw, 1.4rem);
  min-width: 120px;
}
.btn-modal-confirm:hover {
  box-shadow: 0 0 15px var(--primary-color);
}

.btn-modal-cancel {
  background: transparent;
  color: var(--text-color);
  border: 2px solid var(--panel-border);
  padding: 0.9rem 2.5rem;
  font-size: clamp(1.1rem, 2vw, 1.4rem);
  min-width: 120px;
}
.btn-modal-cancel:hover {
  background-color: rgba(255,255,255,0.1);
}
</style>