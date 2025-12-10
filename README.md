<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>運輸日報表回傳系統</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; background-color: #f4f4f9; padding: 20px; }
        .container { max-width: 600px; margin: 0 auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        h2 { text-align: center; color: #333; margin-bottom: 20px; }
        
        .section { margin-bottom: 20px; padding: 15px; border: 1px solid #e0e0e0; border-radius: 8px; background: #fafafa; }
        .section-title { font-weight: bold; margin-bottom: 10px; color: #555; border-bottom: 2px solid #ddd; padding-bottom: 5px; display: flex; justify-content: space-between; align-items: center; }
        
        label { display: block; margin-top: 10px; font-weight: bold; font-size: 0.9em; }
        input, select, textarea { width: 100%; padding: 10px; margin-top: 5px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; font-size: 16px; }
        
        /* 鎖定時的樣式 */
        .locked-field { background-color: #e9ecef !important; color: #495057; cursor: not-allowed; border-color: #ced4da; }
        
        /* 噸數排版 Grid */
        .tonnage-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; }
        
        /* iPhone 修復樣式 */
        .tonnage-input {
            -webkit-appearance: none; appearance: none;
            height: 45px; min-width: 0; background-color: #fff;
            border: 1px solid #ccc; border-radius: 5px;
            font-size: 16px; text-align: center; width: 100%;
            box-sizing: border-box; padding: 10px;
        }
        
        /* 送出按鈕 (深綠色) */
        .btn-submit { width: 100%; padding: 15px; background-color: #28a745; color: white; border: none; border-radius: 5px; font-size: 1.2em; cursor: pointer; margin-top: 20px; font-weight: bold; }
        .btn-submit:hover { background-color: #218838; }
        .btn-submit:disabled { background-color: #ccc; cursor: not-allowed; }
        
        /* 變更按鈕 (紅色) */
        .btn-change { background-color: #dc3545; color: white; border: none; padding: 5px 12px; border-radius: 4px; font-size: 0.85em; cursor: pointer; }
        
        /* 記憶標籤樣式 (Blue Tags) */
        .tag-container { margin-top: 5px; display: flex; flex-wrap: wrap; gap: 8px; min-height: 10px; }
        .history-tag {
            background-color: #e3f2fd; color: #0d47a1; border: 1px solid #90caf9;
            padding: 6px 10px; border-radius: 20px; font-size: 0.9em; cursor: pointer;
            display: flex; align-items: center; user-select: none; font-weight: 500;
        }
        .history-tag:hover { background-color: #bbdefb; }
        .tag-delete {
            margin-left: 8px; color: #ef5350; font-weight: bold; font-size: 1.2em;
            cursor: pointer; padding: 0 4px; line-height: 1;
        }

        /* 貨物管理按鈕區 */
        .cargo-controls { display: flex; gap: 10px; margin-top: 5px; justify-content: flex-end; }
        .btn-mini { padding: 5px 10px; font-size: 0.85em; border-radius: 4px; border: none; cursor: pointer; color: white; width: auto; }
        .btn-add { background-color: #17a2b8; }
        .btn-del { background-color: #6c757d; }

        /* ★★★ 歷史組合按鈕 (改成 LINE 綠色風格) ★★★ */
        .history-container { margin-top: 15px; border-top: 1px dashed #ccc; padding-top: 10px; }
        .history-label { font-size: 0.9em; color: #333; margin-bottom: 8px; font-weight: bold; }
        .btn-history-item { 
            display: block; width: 100%; margin-bottom: 10px; padding: 12px; 
            /* 這裡改成 LINE 的綠色 */
            background-color: #0072E3; 
            color: white; border: none; border-radius: 8px; 
            cursor: pointer; text-align: left; 
            /* 字體加大加粗，方便長輩閱讀 */
            font-size: 1.1em; font-weight: bold;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .btn-history-item:hover { background-color: #05a546; box-shadow: 0 4px 8px rgba(0,0,0,0.15); transform: translateY(-1px); transition: 0.2s; }
        
        .hidden { display: none; }
        #loading { display: none; text-align: center; color: blue; font-weight: bold; margin-top: 10px; }
    </style>
</head>
<body>

<div class="container">
    <h2>運輸回傳系統</h2>

    <form id="logisticsForm">
        
        <div class="section">
            <div class="section-title">
                <span>車輛綁定</span>
                <button type="button" id="changeVehicleBtn" class="btn-change hidden" onclick="unlockVehicle()">變更車輛</button>
            </div>
            
            <label>載運車輛 (車頭)</label>
            <input type="text" id="truckHead" placeholder="例如: 101-W7" required>
            <div id="headTags" class="tag-container"></div>
            
            <label>載運板尾</label>
            <input type="text" id="trailerTail" placeholder="例如: 36-AQ" required>
            <div id="tailTags" class="tag-container"></div>

            <div id="historyContainer" class="history-container hidden">
                <div class="history-label">快速套用上次組合:</div>
            </div>
        </div>

        <div class="section">
            <div class="section-title">里程登記 (自動判斷)</div>
            <div id="startMileageGroup" class="hidden">
                <label style="color:red;">★ 月初里程 (本月未填必填)</label>
                <input type="number" id="startMileage" placeholder="輸入月初里程">
            </div>
            <div id="endMileageGroup" class="hidden">
                <label style="color:red;">★ 月底里程 (最後一天必填)</label>
                <input type="number" id="endMileage" placeholder="輸入月底里程">
            </div>
            <div id="noMileageNeeded" style="color: #888; font-size: 0.9em; text-align: center; padding: 10px;">非月初/月底，無需填寫里程</div>
        </div>

        <div class="section">
            <div class="section-title">作業內容</div>
            <label>載運日期</label>
            <input type="date" id="date" required>
            
            <label>載運貨物</label>
            <select id="cargoType" onchange="checkImportGoods()" required>
                </select>
            <div class="cargo-controls">
                <button type="button" class="btn-mini btn-add" onclick="addNewCargo()">＋新增貨物</button>
                <button type="button" class="btn-mini btn-del" onclick="deleteCargo()">－刪除選取</button>
            </div>

            <div id="shipNameGroup" class="hidden">
                <label style="color:blue;">船名 (進口物資必填)</label>
                <input type="text" id="shipName" placeholder="輸入船名">
            </div>
        </div>

        <div class="section">
            <div class="section-title">載運噸數 (可填多筆)</div>
            <div class="tonnage-grid" id="tonnageContainer"></div>
        </div>

        <div class="section">
            <div class="section-title">費用與保養 (選填)</div>
            <label>加油金額</label>
            <input type="number" id="fuelCost" placeholder="0">

            <label>保養項目</label>
            <input type="text" id="maintItem" placeholder="無">

            <label>保養里程數</label>
            <input type="number" id="maintMileage" placeholder="0">

            <label>更換輪胎里程</label>
            <input type="number" id="tireMileage" placeholder="0">
            
            <label>備註事項</label>
            <textarea id="remarks" rows="2" placeholder="備註"></textarea>
        </div>

        <button type="submit" id="submitBtn" class="btn-submit">確認送出報表</button>
        <div id="loading">資料傳送中，請稍候...</div>
    </form>
</div>

<script>
    // ★★★ 你的 Google Apps Script 網址 ★★★
    var scriptURL = 'https://script.google.com/macros/s/AKfycbyWhZMBeDGIlyEL2xj-Wpu8It1DdcI551HWPa4qc2zHPyvjFv7hDap4cZsodipuT_2IYA/exec'; 

    window.onload = function() {
        const tonnageContainer = document.getElementById('tonnageContainer');
        tonnageContainer.innerHTML = ''; 
        for (let i = 1; i <= 15; i++) {
            let input = document.createElement('input');
            input.type = 'number';
            input.inputmode = 'decimal';
            input.step = '0.01';
            input.className = 'tonnage-input';
            input.placeholder = '#' + i;
            tonnageContainer.appendChild(input);
        }

        initVehicle();       
        initTagHistory();    
        initCargoList();     
        checkDateForMileage(); 
        setDefaultDate();    
    };

    // ================= 獨立記憶標籤 =================
    function initTagHistory() {
        renderTags('truckHead', 'headTags');
        renderTags('trailerTail', 'tailTags');
    }
    function getTagList(type) {
        let list = localStorage.getItem('tag_history_' + type);
        return list ? JSON.parse(list) : [];
    }
    function saveTag(value, type) {
        if (!value) return;
        let list = getTagList(type);
        list = list.filter(item => item !== value);
        list.unshift(value);
        if (list.length > 10) list.length = 10;
        localStorage.setItem('tag_history_' + type, JSON.stringify(list));
        renderTags(type === 'truckHead' ? 'truckHead' : 'trailerTail', 
                   type === 'truckHead' ? 'headTags' : 'tailTags');
    }
    function deleteTag(value, type) {
        if(!confirm(`確定要刪除常用紀錄 "${value}" 嗎?`)) return;
        let list = getTagList(type);
        list = list.filter(item => item !== value);
        localStorage.setItem('tag_history_' + type, JSON.stringify(list));
        renderTags(type === 'truckHead' ? 'truckHead' : 'trailerTail', 
                   type === 'truckHead' ? 'headTags' : 'tailTags');
    }
    function renderTags(inputId, containerId) {
        const list = getTagList(inputId);
        const container = document.getElementById(containerId);
        container.innerHTML = '';
        list.forEach(val => {
            let tag = document.createElement('div');
            tag.className = 'history-tag';
            let textSpan = document.createElement('span');
            textSpan.innerText = val;
            textSpan.onclick = function() {
                const input = document.getElementById(inputId);
                if (!input.readOnly) input.value = val;
            };
            let delBtn = document.createElement('span');
            delBtn.className = 'tag-delete';
            delBtn.innerHTML = '×';
            delBtn.onclick = function(e) {
                e.stopPropagation(); 
                deleteTag(val, inputId);
            };
            tag.appendChild(textSpan);
            tag.appendChild(delBtn);
            container.appendChild(tag);
        });
    }

    // ================= 貨物清單管理 =================
    const defaultCargos = ["砂石", "水泥", "進口物資", "其他"];
    function initCargoList() {
        const select = document.getElementById('cargoType');
        let storedCargos = localStorage.getItem('my_cargo_list');
        let cargoList = storedCargos ? JSON.parse(storedCargos) : defaultCargos;
        defaultCargos.forEach(c => { if (!cargoList.includes(c)) cargoList.push(c); });
        select.innerHTML = '<option value="" disabled selected>請選擇貨物</option>';
        cargoList.forEach(c => {
            let opt = document.createElement('option');
            opt.value = c;
            opt.innerText = c;
            select.appendChild(opt);
        });
    }
    function addNewCargo() {
        let newCargo = prompt("請輸入新貨物名稱:");
        if (newCargo && newCargo.trim() !== "") {
            newCargo = newCargo.trim();
            let storedCargos = localStorage.getItem('my_cargo_list');
            let cargoList = storedCargos ? JSON.parse(storedCargos) : [...defaultCargos];
            if (!cargoList.includes(newCargo)) {
                cargoList.push(newCargo);
                localStorage.setItem('my_cargo_list', JSON.stringify(cargoList));
                initCargoList(); 
                document.getElementById('cargoType').value = newCargo;
                checkImportGoods();
            } else { alert("此貨物已在清單中"); }
        }
    }
    function deleteCargo() {
        const select = document.getElementById('cargoType');
        const selected = select.value;
        if (!selected) { alert("請先選擇要刪除的項目"); return; }
        if (defaultCargos.includes(selected)) { alert(`「${selected}」是系統預設項目，無法刪除。`); return; }
        if (confirm(`確定要從選單中移除「${selected}」嗎？`)) {
            let storedCargos = localStorage.getItem('my_cargo_list');
            let cargoList = storedCargos ? JSON.parse(storedCargos) : [...defaultCargos];
            cargoList = cargoList.filter(c => c !== selected);
            localStorage.setItem('my_cargo_list', JSON.stringify(cargoList));
            initCargoList();
            document.getElementById('shipNameGroup').classList.add('hidden');
        }
    }

    // ================= 組合記憶 =================
    function getHistory() {
        try {
            let history = localStorage.getItem('vehicleHistory');
            return history ? JSON.parse(history) : [];
        } catch (e) { return []; }
    }
    function saveToHistory(head, tail) {
        try {
            if (!head || !tail) return;
            let history = getHistory();
            history = history.filter(item => !(item.head === head && item.tail === tail));
            history.unshift({ head: head, tail: tail });
            if (history.length > 3) history = history.slice(0, 3);
            localStorage.setItem('vehicleHistory', JSON.stringify(history));
            saveTag(head, 'truckHead');
            saveTag(tail, 'trailerTail');
        } catch (e) { console.error(e); }
    }
    function initVehicle() {
        try {
            let history = getHistory();
            if (history.length > 0 && history[0].head && history[0].tail) {
                lockVehicle(history[0].head, history[0].tail);
            } else { unlockVehicle(); }
        } catch (e) {}
    }
    function lockVehicle(head, tail) {
        const headInput = document.getElementById('truckHead');
        const tailInput = document.getElementById('trailerTail');
        const changeBtn = document.getElementById('changeVehicleBtn');
        const historyContainer = document.getElementById('historyContainer');
        const headTags = document.getElementById('headTags');
        const tailTags = document.getElementById('tailTags');

        headInput.value = head || '';
        tailInput.value = tail || '';
        headInput.readOnly = true;
        tailInput.readOnly = true;
        headInput.classList.add('locked-field');
        tailInput.classList.add('locked-field');

        changeBtn.classList.remove('hidden');
        historyContainer.classList.add('hidden');
        headTags.classList.add('hidden');
        tailTags.classList.add('hidden');
    }
    function unlockVehicle() {
        const headInput = document.getElementById('truckHead');
        const tailInput = document.getElementById('trailerTail');
        const changeBtn = document.getElementById('changeVehicleBtn');
        const headTags = document.getElementById('headTags');
        const tailTags = document.getElementById('tailTags');

        headInput.readOnly = false;
        tailInput.readOnly = false;
        headInput.classList.remove('locked-field');
        tailInput.classList.remove('locked-field');

        changeBtn.classList.add('hidden');
        headTags.classList.remove('hidden');
        tailTags.classList.remove('hidden');
        renderHistoryButtons();
    }
    function renderHistoryButtons() {
        let history = getHistory();
        const container = document.getElementById('historyContainer');
        if (history.length === 0) {
            container.classList.add('hidden');
            return;
        }
        container.classList.remove('hidden');
        container.innerHTML = '<div class="history-label">快速套用上次組合:</div>';
        history.forEach((item, index) => {
            if (!item.head || !item.tail) return;
            let btn = document.createElement('button');
            btn.type = 'button';
            btn.className = 'btn-history-item';
            btn.innerHTML = `套用: ${item.head} + ${item.tail}`;
            btn.onclick = function() { lockVehicle(item.head, item.tail); };
            container.appendChild(btn);
        });
    }

    // ================= 日期與傳送 =================
    function setDefaultDate() {
        const today = new Date().toISOString().split('T')[0];
        const dateInput = document.getElementById('date');
        if(!dateInput.value) dateInput.value = today;
    }
    function checkDateForMileage() {
        const today = new Date();
        const m = today.getMonth();
        const y = today.getFullYear();
        if (!localStorage.getItem(`startMileageSubmitted_${y}_${m}`)) {
            document.getElementById('startMileageGroup').classList.remove('hidden');
            document.getElementById('startMileage').required = true;
            document.getElementById('noMileageNeeded').classList.add('hidden');
        }
        const lastDayOfMonth = new Date(y, m + 1, 0).getDate();
        if (today.getDate() === lastDayOfMonth && !localStorage.getItem(`endMileageSubmitted_${y}_${m}`)) {
            document.getElementById('endMileageGroup').classList.remove('hidden');
            document.getElementById('endMileage').required = true;
            document.getElementById('noMileageNeeded').classList.add('hidden');
        }
    }
    function checkImportGoods() {
        const cargo = document.getElementById('cargoType').value;
        const shipGroup = document.getElementById('shipNameGroup');
        if (cargo === '進口物資') {
            shipGroup.classList.remove('hidden');
            document.getElementById('shipName').required = true;
        } else {
            shipGroup.classList.add('hidden');
            document.getElementById('shipName').required = false;
            document.getElementById('shipName').value = '';
        }
    }

    const form = document.getElementById('logisticsForm');
    form.addEventListener('submit', e => {
        e.preventDefault();
        if (scriptURL.includes('YOUR_GOOGLE')) { alert("請先設定 Google Apps Script URL！"); return; }

        const submitBtn = document.getElementById('submitBtn');
        const loading = document.getElementById('loading');
        submitBtn.disabled = true;
        submitBtn.innerText = "傳送中...";
        loading.style.display = 'block';

        let tonnages = [];
        document.querySelectorAll('.tonnage-input').forEach(input => {
            if(input.value && input.value.trim() !== '') { tonnages.push(input.value); }
        });

        const truckHead = document.getElementById('truckHead').value.trim();
        const trailerTail = document.getElementById('trailerTail').value.trim();
        
        if (!truckHead || !trailerTail) {
            alert("請填寫車輛資料！");
            submitBtn.disabled = false;
            submitBtn.innerText = "確認送出報表";
            loading.style.display = 'none';
            return;
        }

        let formData = {
            truckHead: truckHead,
            trailerTail: trailerTail,
            date: document.getElementById('date').value,
            startMileage: document.getElementById('startMileage').value || "0",
            endMileage: document.getElementById('endMileage').value || "0",
            cargoType: document.getElementById('cargoType').value,
            shipName: document.getElementById('shipName').value,
            tonnages: tonnages,
            fuelCost: document.getElementById('fuelCost').value || "0",
            maintItem: document.getElementById('maintItem').value || "無",
            maintMileage: document.getElementById('maintMileage').value || "0",
            tireMileage: document.getElementById('tireMileage').value || "0",
            remarks: document.getElementById('remarks').value || ""
        };

        fetch(scriptURL, {
            method: 'POST',
            body: JSON.stringify(formData),
            mode: 'no-cors'
        })
        .then(response => {
            alert("報表回傳成功！");
            saveToHistory(formData.truckHead, formData.trailerTail);
            const today = new Date();
            if (formData.startMileage !== "0") {
                localStorage.setItem(`startMileageSubmitted_${today.getFullYear()}_${today.getMonth()}`, true);
            }
            const lastDay = new Date(today.getFullYear(), today.getMonth() + 1, 0).getDate();
            if (today.getDate() === lastDay && formData.endMileage !== "0") {
                localStorage.setItem(`endMileageSubmitted_${today.getFullYear()}_${today.getMonth()}`, true);
            }

            form.reset();
            lockVehicle(formData.truckHead, formData.trailerTail);
            setDefaultDate();
            document.querySelectorAll('.tonnage-input').forEach(input => { input.value = ''; });
            document.getElementById('startMileageGroup').classList.add('hidden');
            document.getElementById('endMileageGroup').classList.add('hidden');
            document.getElementById('noMileageNeeded').classList.remove('hidden');

            submitBtn.disabled = false;
            submitBtn.innerText = "確認送出報表";
            loading.style.display = 'none';
        })
        .catch(error => {
            console.error('Error!', error.message);
            alert("傳送失敗，請檢查網路");
            submitBtn.disabled = false;
            submitBtn.innerText = "確認送出報表";
            loading.style.display = 'none';
        });
    });
</script>

</body>
</html>
