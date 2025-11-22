<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>儲蓄險轉換分析系統 Pro (雙情境版)</title>
    <!-- 改用更穩定的 CDN 連結 (Chart.js 3.9.1) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script>
        // 全域錯誤攔截：如果程式有語法錯誤或無法執行，會直接彈出視窗通知
        window.onerror = function(msg, url, line, col, error) {
            alert("系統發生錯誤 (System Error):\n" + msg + "\n\n請檢查網路連線，或確認程式碼是否完整。");
            return false;
        };
    </script>
    <style>
        :root {
            --primary: #1a365d; /* 深藍 - 專業信任 */
            --secondary: #2b6cb0; /* 亮藍 - 資訊 */
            --accent: #38a169; /* 綠色 - 成長/獲利 */
            --warning: #dd6b20; /* 橘色 - 宣告/變動 */
            --danger: #e53e3e; /* 紅色 - 風險/支出 */
            --light: #f7fafc;
            --dark: #2d3748;
            --border: #e2e8f0;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }

        * {
            box-sizing: border-box;
            font-family: "Microsoft JhengHei", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: #edf2f7;
            color: var(--dark);
            line-height: 1.5;
        }

        .app-container {
            display: grid;
            grid-template-columns: 350px 1fr; /* 左側輸入，右側儀表板 */
            min-height: 100vh;
        }

        /* 左側邊欄 - 輸入區 */
        .sidebar {
            background: white;
            padding: 20px;
            border-right: 1px solid var(--border);
            overflow-y: auto;
            height: 100vh;
            position: sticky;
            top: 0;
            box-shadow: 2px 0 10px rgba(0,0,0,0.05);
            z-index: 10;
        }

        .sidebar-header {
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 2px solid var(--primary);
        }

        .sidebar-header h2 {
            margin: 0;
            color: var(--primary);
            font-size: 1.5rem;
        }

        .sidebar-header p {
            margin: 5px 0 0;
            color: #718096;
            font-size: 0.9rem;
        }

        .input-section {
            margin-bottom: 25px;
            background: #f8fafc;
            padding: 15px;
            border-radius: 8px;
            border: 1px solid var(--border);
        }

        .input-section h3 {
            margin-top: 0;
            margin-bottom: 15px;
            font-size: 1.1rem;
            color: var(--secondary);
            display: flex;
            align-items: center;
        }

        .input-section h3::before {
            content: '';
            display: inline-block;
            width: 4px;
            height: 16px;
            background: var(--secondary);
            margin-right: 8px;
            border-radius: 2px;
        }

        .form-group {
            margin-bottom: 12px;
        }

        .form-group label {
            display: block;
            font-size: 0.9rem;
            font-weight: 600;
            margin-bottom: 4px;
            color: #4a5568;
        }

        .form-group input, .form-group select {
            width: 100%;
            padding: 8px 12px;
            border: 1px solid #cbd5e0;
            border-radius: 6px;
            font-size: 0.95rem;
            transition: border-color 0.2s;
        }

        .form-group input:focus, .form-group select:focus {
            outline: none;
            border-color: var(--secondary);
            box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.2);
        }

        .btn-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 20px;
        }

        .btn {
            padding: 10px;
            border: none;
            border-radius: 6px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            text-align: center;
        }

        .btn-primary {
            background: var(--primary);
            color: white;
            grid-column: span 2;
            font-size: 1.1rem;
            padding: 12px;
        }

        .btn-primary:hover { background: #2c5282; }

        .btn-outline {
            background: white;
            border: 1px solid #cbd5e0;
            color: #4a5568;
        }

        .btn-outline:hover { background: #f7fafc; border-color: #a0aec0; }

        /* 右側主內容 - 儀表板 */
        .main-content {
            padding: 30px;
            overflow-y: auto;
        }

        .dashboard-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 30px;
        }

        .dashboard-title h1 {
            margin: 0;
            color: var(--primary);
            font-size: 2rem;
        }

        .dashboard-actions button {
            margin-left: 10px;
            padding: 8px 16px;
            border-radius: 6px;
            border: none;
            background: white;
            box-shadow: var(--shadow);
            cursor: pointer;
            font-weight: 600;
            color: var(--dark);
        }

        .dashboard-actions button:hover {
            background: #f7fafc;
            transform: translateY(-1px);
        }

        /* 關鍵指標卡片 */
        .kpi-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .kpi-card {
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: var(--shadow);
            border-left: 5px solid var(--border);
            transition: transform 0.2s;
        }

        .kpi-card:hover {
            transform: translateY(-3px);
        }

        .kpi-card.primary { border-left-color: var(--primary); }
        .kpi-card.success { border-left-color: var(--accent); }
        .kpi-card.warning { border-left-color: var(--warning); }
        .kpi-card.danger { border-left-color: var(--danger); }

        .kpi-label {
            font-size: 0.9rem;
            color: #718096;
            margin-bottom: 5px;
            font-weight: 600;
        }

        .kpi-value {
            font-size: 1.8rem;
            font-weight: 700;
            color: var(--dark);
        }

        .kpi-sub {
            font-size: 0.85rem;
            margin-top: 5px;
            color: #718096;
        }

        .kpi-sub.up { color: var(--accent); }
        .kpi-sub.down { color: var(--danger); }

        /* 圖表區 */
        .chart-section {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: var(--shadow);
            margin-bottom: 30px;
        }

        .chart-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .chart-header h3 {
            margin: 0;
            color: var(--dark);
        }

        .chart-controls select {
            padding: 5px 10px;
            border-radius: 4px;
            border: 1px solid #cbd5e0;
        }

        .chart-container {
            position: relative;
            height: 400px;
            width: 100%;
        }

        /* 比較表格 */
        .comparison-section {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: var(--shadow);
        }

        /* 智能分析報告 */
        .report-section {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: var(--shadow);
            margin-top: 30px;
            border-left: 5px solid var(--secondary);
        }

        .report-content {
            line-height: 1.8;
            color: #2d3748;
        }

        .report-block {
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 1px dashed #e2e8f0;
        }

        .report-block:last-child {
            border-bottom: none;
            margin-bottom: 0;
        }

        .report-title {
            font-weight: bold;
            color: var(--primary);
            font-size: 1.1rem;
            margin-bottom: 8px;
            display: flex;
            align-items: center;
        }

        .report-icon {
            margin-right: 8px;
            font-size: 1.2rem;
        }

        .highlight-text {
            font-weight: bold;
            padding: 0 4px;
        }
        
        .text-green { color: var(--accent); }
        .text-orange { color: var(--warning); }
        .text-red { color: var(--danger); }

        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
        }

        .comparison-table th, .comparison-table td {
            padding: 12px 15px;
            text-align: center;
            border-bottom: 1px solid #e2e8f0;
        }

        .comparison-table th {
            background: #f7fafc;
            color: #4a5568;
            font-weight: 600;
        }

        .comparison-table tr:last-child td {
            border-bottom: none;
        }

        .comparison-table .highlight-row {
            background: #ebf8ff;
            font-weight: bold;
        }

        /* 響應式設計 */
        @media (max-width: 1024px) {
            .app-container {
                grid-template-columns: 1fr;
            }
            
            .sidebar {
                height: auto;
                position: relative;
                border-right: none;
                border-bottom: 1px solid var(--border);
            }
        }

        /* 列印樣式 */
        @media print {
            .app-container {
                display: block;
            }
            .sidebar {
                display: none;
            }
            .main-content {
                padding: 0;
            }
            .dashboard-actions {
                display: none;
            }
            .kpi-card, .chart-section, .comparison-section {
                box-shadow: none;
                border: 1px solid #e2e8f0;
                break-inside: avoid;
            }
            body {
                background: white;
            }
        }

        /* Tabs Styles */
        .tabs {
            display: flex;
            margin-bottom: 20px;
            border-bottom: 2px solid var(--border);
        }
        
        .tab-btn {
            flex: 1;
            padding: 12px;
            background: none;
            border: none;
            border-bottom: 3px solid transparent;
            font-weight: 600;
            color: #718096;
            cursor: pointer;
            margin-bottom: -3px;
            transition: all 0.2s;
        }
        
        .tab-btn:hover {
            color: var(--secondary);
            background-color: #f7fafc;
        }
        
        .tab-btn.active {
            color: var(--primary);
            border-bottom-color: var(--primary);
            background-color: #fff;
        }
        
        .tab-content {
            display: none;
            animation: fadeIn 0.3s ease;
        }
        
        .tab-content.active {
            display: block;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(5px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
    <div class="app-container">
        <!-- 左側控制面板 -->
        <aside class="sidebar">
            <div class="sidebar-header">
                <h2>參數設定</h2>
                <p>選擇分析模式</p>
            </div>

            <div class="tabs">
                <button class="tab-btn active" onclick="switchTab('savings')" id="btn-savings">儲蓄險轉換</button>
                <button class="tab-btn" onclick="switchTab('investment')" id="btn-investment">投資型(月配)</button>
            </div>

            <!-- Tab 1: 儲蓄險轉換 -->
            <div id="tab-savings" class="tab-content active">
                <div class="input-section">
                    <h3>🏦 舊保單 (現狀)</h3>
                    <div class="form-group">
                        <label>當前現金價值 (原幣)</label>
                        <input type="number" id="oldCashValue" value="75" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>幣別</label>
                        <select id="oldCurrency">
                            <option value="TWD">台幣 (TWD)</option>
                            <option value="USD">美元 (USD)</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>預定利率 (%) <small>保證</small></label>
                        <input type="number" id="oldRate" value="2.5" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>宣告利率 (%) <small style="color:#666">若有可填</small></label>
                        <input type="number" id="oldDeclaredRate" value="" placeholder="選填 (例如 2.7)" step="0.1">
                    </div>
                </div>

                <div class="input-section">
                    <h3>🆕 新保單 (方案)</h3>
                    <div class="form-group">
                        <label>年繳保費 (美元)</label>
                        <input type="number" id="newPremium" value="1.9078" step="0.0001">
                    </div>
                    <div class="form-group">
                        <label>繳費年期</label>
                        <select id="paymentYears">
                            <option value="6">6年</option>
                            <option value="10">10年</option>
                            <option value="20">20年</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>預定利率 (%) <small style="color:var(--accent)">保證</small></label>
                        <input type="number" id="newGuaranteedRate" value="2.5" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>宣告利率 (%) <small style="color:var(--warning)">預期</small></label>
                        <input type="number" id="newDeclaredRate" value="3.8" step="0.1">
                    </div>
                </div>
            </div>

            <!-- Tab 2: 投資型商品 -->
            <div id="tab-investment" class="tab-content">
                <div class="input-section">
                    <h3>🅰️ 方案 A (現狀/基準)</h3>
                    <div class="form-group">
                        <label>投入金額 (原幣)</label>
                        <input type="number" id="invPrincipalA" value="30000">
                    </div>
                    <div class="form-group">
                        <label>幣別</label>
                        <select id="invCurrencyA">
                            <option value="USD">美元 (USD)</option>
                            <option value="TWD">台幣 (TWD)</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>年化配息率 (%)</label>
                        <input type="number" id="invDistRateA" value="5" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>或 每月平均配息金額 (原幣) <small style="color:var(--secondary)">優先使用</small></label>
                        <input type="number" id="invMonthlyPayoutA" placeholder="填此則忽略配息率">
                    </div>
                    <div class="form-group">
                        <label>預期淨值年漲跌 (%)</label>
                        <input type="number" id="invNavGrowthA" value="0" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>前置費用 (%) <small style="color:#666">手續費</small></label>
                        <input type="number" id="invUpfrontFeeA" value="1.5" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>內扣費用 (%) <small style="color:#666">經理費/管理費</small></label>
                        <input type="number" id="invMgmtFeeA" value="1.0" step="0.1">
                    </div>
                    <div class="form-group">
                        <label><input type="checkbox" id="invUseNavStepA"> 進階：使用 NAV 階梯配息 (去年)</label>
                    </div>
                    <div class="form-group">
                        <label>NAV→月配定義 (原幣) <small style="color:#666">格式：10=1000,10.1=2000,10.2=3000</small></label>
                        <input type="text" id="invNavThresholdsA" placeholder="10=1000,10.1=2000,10.2=3000">
                    </div>
                    <div class="form-group">
                        <label>月配上限 (原幣)</label>
                        <input type="number" id="invPayoutCapA" value="3000" step="1">
                    </div>
                    <div class="form-group">
                        <label>去年每月 NAV (逗號分隔，12筆)</label>
                        <textarea id="invMonthlyNavsA" rows="2" placeholder="例如：9.98,10.05,10.12,10.08,10.15,10.20,10.18,10.22,10.30,10.25,10.28,10.35"></textarea>
                    </div>
                </div>

                <div class="input-section">
                    <h3>🅱️ 方案 B (新選/比較)</h3>
                    <div class="form-group">
                        <label>投入金額 (原幣)</label>
                        <input type="number" id="invPrincipalB" value="30000">
                    </div>
                    <div class="form-group">
                        <label>幣別</label>
                        <select id="invCurrencyB">
                            <option value="USD">美元 (USD)</option>
                            <option value="TWD">台幣 (TWD)</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>年化配息率 (%)</label>
                        <input type="number" id="invDistRateB" value="8" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>或 每月平均配息金額 (原幣) <small style="color:var(--secondary)">優先使用</small></label>
                        <input type="number" id="invMonthlyPayoutB" placeholder="填此則忽略配息率">
                    </div>
                    <div class="form-group">
                        <label>預期淨值年漲跌 (%)</label>
                        <input type="number" id="invNavGrowthB" value="0" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>前置費用 (%) <small style="color:#666">保費費用</small></label>
                        <input type="number" id="invUpfrontFeeB" value="3.0" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>內扣費用 (%) <small style="color:#666">經理費/管理費</small></label>
                        <input type="number" id="invMgmtFeeB" value="1.5" step="0.1">
                    </div>
                    <div class="form-group">
                        <label><input type="checkbox" id="invUseNavStepB"> 進階：使用 NAV 階梯配息 (去年)</label>
                    </div>
                    <div class="form-group">
                        <label>NAV→月配定義 (原幣) <small style="color:#666">格式：10=1000,10.1=2000,10.2=3000</small></label>
                        <input type="text" id="invNavThresholdsB" placeholder="10=1000,10.1=2000,10.2=3000">
                    </div>
                    <div class="form-group">
                        <label>月配上限 (原幣)</label>
                        <input type="number" id="invPayoutCapB" value="3000" step="1">
                    </div>
                    <div class="form-group">
                        <label>去年每月 NAV (逗號分隔，12筆)</label>
                        <textarea id="invMonthlyNavsB" rows="2" placeholder="例如：9.98,10.05,10.12,10.08,10.15,10.20,10.18,10.22,10.30,10.25,10.28,10.35"></textarea>
                    </div>
                </div>
            </div>

            <div class="input-section">
                <h3>🌍 總體經濟</h3>
                <div class="form-group">
                    <label>匯率 (USD/TWD)</label>
                    <input type="number" id="exchangeRate" value="32.5" step="0.1">
                </div>
            </div>

            <div class="btn-group">
                <button class="btn btn-primary" onclick="calculateAndRender()">🔄 立即分析</button>
                <button class="btn btn-outline" onclick="resetData()">重置</button>
                <button class="btn btn-outline" onclick="printReport()">列印</button>
            </div>
        </aside>

        <!-- 右側儀表板 -->
        <main class="main-content">
            <div class="dashboard-header">
                <div class="dashboard-title">
                    <h1>保單轉換效益分析 Pro</h1>
                    <p>雙情境模擬：保證收益 vs 預期收益</p>
                </div>
                <div class="dashboard-actions">
                    <button onclick="exportPDF()">匯出 PDF</button>
                </div>
            </div>

            <!-- KPI 卡片 -->
            <div class="kpi-grid">
                <div class="kpi-card primary">
                    <div class="kpi-label">舊保單 20年後價值</div>
                    <div class="kpi-value" id="kpi-old-val">--</div>
                    <div class="kpi-sub">基於 <span id="kpi-old-rate">--</span>% 複利</div>
                </div>
                <div class="kpi-card success">
                    <div class="kpi-label">新保單 (保證) 20年後價值</div>
                    <div class="kpi-value" id="kpi-new-guaranteed-val">--</div>
                    <div class="kpi-sub" id="kpi-diff-guaranteed">--</div>
                </div>
                <div class="kpi-card warning">
                    <div class="kpi-label">新保單 (宣告) 20年後價值</div>
                    <div class="kpi-value" id="kpi-new-declared-val">--</div>
                    <div class="kpi-sub" id="kpi-diff-declared">--</div>
                </div>
                <div class="kpi-card danger">
                    <div class="kpi-label">優勢反轉點 (黃金交叉)</div>
                    <div class="kpi-value" id="kpi-breakeven">--</div>
                    <div class="kpi-sub">年 (保證/宣告)</div>
                </div>
            </div>

            <!-- 主要圖表 -->
            <div class="chart-section">
                <div class="chart-header">
                    <h3>資產累積趨勢圖 (台幣計價)</h3>
                    <div class="chart-controls">
                        <select id="chartView" onchange="calculateAndRender()">
                            <option value="total">總資產價值</option>
                            <option value="net">淨收益 (扣除本金)</option>
                        </select>
                    </div>
                </div>
                <div class="chart-container">
                    <canvas id="mainChart"></canvas>
                </div>
            </div>

            <!-- 詳細數據表格 -->
            <div class="comparison-section">
                <h3>詳細數據比較 (每5年)</h3>
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>年度</th>
                            <th>累積本金 (舊保單價值)</th>
                            <th>舊保單價值 (持續持有)</th>
                            <th>新保單 (保證收益)</th>
                            <th>新保單 (宣告收益)</th>
                            <th>保證收益差額</th>
                        </tr>
                    </thead>
                    <tbody id="detailTableBody">
                        <!-- JS 填充 -->
                    </tbody>
                </table>
            </div>

            <!-- 智能分析報告 -->
            <div class="report-section">
                <h3>📊 智能分析報告 (AI Analysis)</h3>
                <div id="reportContent" class="report-content">
                    <!-- JS 自動生成 -->
                </div>
            </div>
        </main>
    </div>

    <script>
        let mainChart = null;
        let currentTab = 'savings';

        // 初始化
        window.onload = function() {
            calculateAndRender();
        };

        function switchTab(tab) {
            currentTab = tab;
            
            // Update UI
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            
            document.getElementById('btn-' + tab).classList.add('active');
            document.getElementById('tab-' + tab).classList.add('active');
            
            // Update Title
            const title = tab === 'savings' ? '保單轉換效益分析 Pro' : '投資型商品(月配息)分析';
            const sub = tab === 'savings' ? '雙情境模擬：保證收益 vs 預期收益' : '現金流與資產價值模擬';
            document.querySelector('.dashboard-title h1').innerText = title;
            document.querySelector('.dashboard-title p').innerText = sub;

            // Reset Chart View
            const chartSelect = document.getElementById('chartView');
            if(chartSelect) chartSelect.value = 'total';

            calculateAndRender();
        }

        function calculateAndRender() {
            console.log("開始計算...", currentTab); 
            try {
                const inputs = getInputs();
                
                if (currentTab === 'savings') {
                    // 顯示/隱藏相關 KPI 卡片 (恢復原狀)
                    document.querySelector('.kpi-grid').style.display = 'grid';
                    
                    const projections = calculateProjections(inputs);
                    updateKPIs(inputs, projections);
                    renderChart(inputs, projections);
                    renderTable(inputs, projections);
                    generateReport(inputs, projections);
                } else {
                    // 投資型商品邏輯
                    const projections = calculateInvestmentProjections(inputs);
                    updateInvestmentKPIs(inputs, projections);
                    renderInvestmentChart(inputs, projections);
                    renderInvestmentTable(inputs, projections);
                    generateInvestmentReport(inputs, projections);
                }
                
            } catch (error) {
                console.error("計算錯誤:", error);
                alert("系統發生錯誤，請檢查輸入數據是否正確。\n錯誤訊息: " + error.message);
            }
        }

        function getInputs() {
            const getVal = (id) => {
                const el = document.getElementById(id);
                return el ? parseFloat(el.value) : 0;
            };
            const getStr = (id) => {
                const el = document.getElementById(id);
                return el ? el.value : '';
            };

            return {
                // Savings
                oldCashValue: getVal('oldCashValue'),
                oldCurrency: getStr('oldCurrency'),
                oldRate: getVal('oldRate') / 100,
                oldDeclaredRate: document.getElementById('oldDeclaredRate').value ? getVal('oldDeclaredRate') / 100 : null,
                
                newPremium: getVal('newPremium'),
                paymentYears: parseInt(getStr('paymentYears')),
                newGuaranteedRate: getVal('newGuaranteedRate') / 100,
                newDeclaredRate: getVal('newDeclaredRate') / 100,
                
                // Investment
                invPrincipalA: getVal('invPrincipalA'),
                invCurrencyA: getStr('invCurrencyA'),
                invDistRateA: getVal('invDistRateA') / 100,
                invMonthlyPayoutA: getVal('invMonthlyPayoutA'), // New
                invNavGrowthA: getVal('invNavGrowthA') / 100,
                invUpfrontFeeA: document.getElementById('invUpfrontFeeA') ? getVal('invUpfrontFeeA') / 100 : 0,
                invMgmtFeeA: document.getElementById('invMgmtFeeA') ? getVal('invMgmtFeeA') / 100 : 0,
                invUseNavStepA: document.getElementById('invUseNavStepA') ? document.getElementById('invUseNavStepA').checked : false,
                invNavThresholdsA: document.getElementById('invNavThresholdsA') ? document.getElementById('invNavThresholdsA').value : '',
                invPayoutCapA: document.getElementById('invPayoutCapA') ? parseFloat(document.getElementById('invPayoutCapA').value || '0') : 0,
                invMonthlyNavsA: document.getElementById('invMonthlyNavsA') ? document.getElementById('invMonthlyNavsA').value : '',

                invPrincipalB: getVal('invPrincipalB'),
                invCurrencyB: getStr('invCurrencyB'),
                invDistRateB: getVal('invDistRateB') / 100,
                invMonthlyPayoutB: getVal('invMonthlyPayoutB'), // New
                invNavGrowthB: getVal('invNavGrowthB') / 100,
                invUpfrontFeeB: document.getElementById('invUpfrontFeeB') ? getVal('invUpfrontFeeB') / 100 : 0,
                invMgmtFeeB: document.getElementById('invMgmtFeeB') ? getVal('invMgmtFeeB') / 100 : 0,
                invUseNavStepB: document.getElementById('invUseNavStepB') ? document.getElementById('invUseNavStepB').checked : false,
                invNavThresholdsB: document.getElementById('invNavThresholdsB') ? document.getElementById('invNavThresholdsB').value : '',
                invPayoutCapB: document.getElementById('invPayoutCapB') ? parseFloat(document.getElementById('invPayoutCapB').value || '0') : 0,
                invMonthlyNavsB: document.getElementById('invMonthlyNavsB') ? document.getElementById('invMonthlyNavsB').value : '',
                
                // Common
                exchangeRate: getVal('exchangeRate')
            };
        }

        function calculateProjections(inputs) {
            const years = 20;
            const data = {
                labels: [],
                oldValues: [],
                oldDeclaredValues: [], // New
                newGuaranteedValues: [],
                newDeclaredValues: [],
                principalLine: [] 
            };

            // 統一轉換為台幣計算
            let initialPrincipalTWD = 0;
            if (inputs.oldCurrency === 'TWD') {
                initialPrincipalTWD = inputs.oldCashValue * 10000;
            } else {
                initialPrincipalTWD = inputs.oldCashValue * 10000 * inputs.exchangeRate;
            }

            let currentOld = initialPrincipalTWD;
            let currentOldDeclared = initialPrincipalTWD; // New
            
            let accountValueGuaranteed = 0;
            let accountValueDeclared = 0;
            
            for (let i = 1; i <= years; i++) {
                data.labels.push(`第${i}年`);
                
                // 1. 舊保單滾存
                currentOld = currentOld * (1 + inputs.oldRate);
                data.oldValues.push(currentOld);

                // 舊保單宣告利率 (若有)
                if (inputs.oldDeclaredRate !== null) {
                    currentOldDeclared = currentOldDeclared * (1 + inputs.oldDeclaredRate);
                    data.oldDeclaredValues.push(currentOldDeclared);
                }
                
                // 2. 新保單滾存
                let premiumPaidThisYear = 0;
                if (i <= inputs.paymentYears) {
                    premiumPaidThisYear = inputs.newPremium * 10000 * inputs.exchangeRate;
                }
                
                let totalPaidSoFar = Math.min(i, inputs.paymentYears) * inputs.newPremium * 10000 * inputs.exchangeRate;
                data.principalLine.push(totalPaidSoFar);

                if (i <= inputs.paymentYears) {
                    accountValueGuaranteed += premiumPaidThisYear;
                    accountValueDeclared += premiumPaidThisYear;
                }
                
                accountValueGuaranteed *= (1 + inputs.newGuaranteedRate);
                accountValueDeclared *= (1 + inputs.newDeclaredRate);
                
                let csvRatio = 1;
                if (i < inputs.paymentYears) {
                    csvRatio = 0.4 + (0.55 * ((i-1) / Math.max(1, inputs.paymentYears-1)));
                } else if (i === inputs.paymentYears) {
                    csvRatio = 0.98;
                } else {
                    csvRatio = 1;
                }

                let valG = accountValueGuaranteed * csvRatio;
                let valD = accountValueDeclared * csvRatio;
                
                data.newGuaranteedValues.push(valG);
                data.newDeclaredValues.push(valD);
            }
            
            return data;
        }

        function calculateInvestmentProjections(inputs) {
            const years = 20;
            
            const calc = (principal, currency, distRate, growthRate, upfrontFee, mgmtFee, monthlyOverride, useNavStep, thresholdsStr, payoutCap, monthlyNavsStr) => {
                const res = {
                    labels: [],
                    principal: [],
                    accountValue: [],
                    cumulativeCash: [],
                    totalBenefit: [],
                    monthlyCash: 0
                };
                
                let principalTWD = principal;
                if (currency === 'USD') {
                    principalTWD = principal * inputs.exchangeRate;
                }
                
                // 扣除前置費用 (Upfront Fee)
                let investedAmount = principalTWD * (1 - upfrontFee);
                let currentVal = investedAmount;
                
                let totalCash = 0;

                // 工具方法：解析 NAV→配息 定義，如 "10=1000,10.1=2000,10.2=3000"
                const parseThresholds = (s) => {
                    if (!s) return [];
                    try {
                        return s.split(',')
                            .map(p => p.trim())
                            .filter(p => p.includes('='))
                            .map(p => {
                                const [t, v] = p.split('=');
                                return { th: parseFloat(t), val: parseFloat(v) };
                            })
                            .filter(o => !isNaN(o.th) && !isNaN(o.val))
                            .sort((a,b) => a.th - b.th);
                    } catch { return []; }
                };

                const parseMonthlyNavs = (s) => {
                    if (!s) return [];
                    try {
                        return s.split(/[，,\s]+/)
                            .map(x => parseFloat(x))
                            .filter(x => !isNaN(x));
                    } catch { return []; }
                };

                const thresholds = parseThresholds(thresholdsStr);
                const navs = parseMonthlyNavs(monthlyNavsStr);

                // 年度配息(台幣)計算策略：
                // 1) 若有「每月平均配息(原幣)」覆蓋，優先使用
                // 2) 其次，若勾選NAV階梯且提供了NAV清單與門檻，依規則計算月配平均
                // 3) 否則使用年化配息率 * 投入本金
                let yearlyDist = 0; // 台幣

                const toTWD = (amtOrig) => currency === 'USD' ? amtOrig * inputs.exchangeRate : amtOrig;

                if (monthlyOverride && monthlyOverride > 0) {
                    const avgMonthlyTWD = toTWD(monthlyOverride);
                    yearlyDist = avgMonthlyTWD * 12;
                    res.monthlyCash = avgMonthlyTWD;
                } else if (useNavStep && thresholds.length > 0 && navs.length > 0) {
                    let monthlySumOrig = 0;
                    navs.forEach(nav => {
                        // 找到不大於 NAV 的最高門檻
                        let payout = 0;
                        for (let i=0; i<thresholds.length; i++) {
                            if (nav >= thresholds[i].th) payout = thresholds[i].val;
                            else break;
                        }
                        if (payoutCap && payoutCap > 0) payout = Math.min(payout, payoutCap);
                        monthlySumOrig += payout;
                    });
                    const months = navs.length;
                    const avgMonthlyOrig = months > 0 ? (monthlySumOrig / months) : 0;
                    const avgMonthlyTWD = toTWD(avgMonthlyOrig);
                    yearlyDist = avgMonthlyTWD * 12;
                    res.monthlyCash = avgMonthlyTWD;
                } else {
                    yearlyDist = principalTWD * distRate; // 固定撥回
                    res.monthlyCash = yearlyDist / 12;
                }

                for (let i = 1; i <= years; i++) {
                    res.labels.push(`第${i}年`);
                    res.principal.push(principalTWD);
                    
                    // 淨值成長 (扣除內扣費用)
                    // 邏輯：(1 + 漲跌幅 - 費用率)
                    // 假設漲跌幅是含息的? 不，通常輸入的是「價格漲跌」。
                    // 投資型保單邏輯：帳戶價值 = 前一年價值 * (1 + 投資報酬率) - 管理費 - 危險保費(這裡忽略) - 配息
                    // 這裡的 growthRate 是單純價格變動。
                    // 修正邏輯：
                    // 1. 增值: currentVal * (1 + growthRate)
                    // 2. 扣費: currentVal * (1 - mgmtFee)  <-- 簡化：直接從報酬率扣
                    // 3. 配息: 扣除配息金額 (若是配息來自本金)
                    // 等等，如果配息率是外加的 (領現金)，那帳戶價值會減少嗎？
                    // 基金/ETF：配息後淨值會掉。
                    // 所以：期末價值 = 期初價值 * (1 + 漲跌幅 - 內扣費用) - 配息金額
                    
                    // 但用戶輸入的 "預期淨值年漲跌" 通常已經是客戶心中的 "價格走勢"。
                    // 如果用戶輸入 "0%" (不漲不跌)，他預期本金不動，配息照領。
                    // 如果我們再扣配息，本金會歸零。
                    // 所以這裡的 "預期淨值年漲跌" 應該被視為 "配息後的價格變動"。
                    // 也就是說：Price_End = Price_Start * (1 + Growth_Net)。
                    // 那內扣費用去哪了？應該是讓 Growth_Net 變低。
                    // 所以：實際漲跌 = 輸入漲跌 - 內扣費用。
                    
                    let netGrowth = growthRate - mgmtFee;
                    currentVal = currentVal * (1 + netGrowth);
                    
                    res.accountValue.push(currentVal);
                    totalCash += yearlyDist;
                    res.cumulativeCash.push(totalCash);
                    res.totalBenefit.push(currentVal + totalCash);
                }
                return res;
            };

            const dataA = calc(
                inputs.invPrincipalA,
                inputs.invCurrencyA,
                inputs.invDistRateA,
                inputs.invNavGrowthA,
                inputs.invUpfrontFeeA,
                inputs.invMgmtFeeA,
                inputs.invMonthlyPayoutA,
                inputs.invUseNavStepA,
                inputs.invNavThresholdsA,
                inputs.invPayoutCapA,
                inputs.invMonthlyNavsA
            );
            const dataB = calc(
                inputs.invPrincipalB,
                inputs.invCurrencyB,
                inputs.invDistRateB,
                inputs.invNavGrowthB,
                inputs.invUpfrontFeeB,
                inputs.invMgmtFeeB,
                inputs.invMonthlyPayoutB,
                inputs.invUseNavStepB,
                inputs.invNavThresholdsB,
                inputs.invPayoutCapB,
                inputs.invMonthlyNavsB
            );

            return { dataA, dataB };
        }

        function updateKPIs(inputs, data) {
            const lastIdx = data.labels.length - 1;
            const oldFinal = data.oldValues[lastIdx];
            const newGFinal = data.newGuaranteedValues[lastIdx];
            const newDFinal = data.newDeclaredValues[lastIdx];
            
            // 格式化金額 (萬)
            const format = (val) => (val / 10000).toFixed(1) + '萬';
            
            document.getElementById('kpi-old-val').innerText = format(oldFinal);
            document.getElementById('kpi-old-rate').innerText = (inputs.oldRate * 100).toFixed(1);
            
            document.getElementById('kpi-new-guaranteed-val').innerText = format(newGFinal);
            const diffG = newGFinal - oldFinal;
            const diffGPct = (diffG / oldFinal) * 100;
            document.getElementById('kpi-diff-guaranteed').innerHTML = 
                `<span class="${diffG >= 0 ? 'up' : 'down'}">${diffG >= 0 ? '▲' : '▼'} ${format(Math.abs(diffG))} (${diffGPct.toFixed(1)}%)</span> vs 舊保單`;
                
            document.getElementById('kpi-new-declared-val').innerText = format(newDFinal);
            const diffD = newDFinal - oldFinal;
            const diffDPct = (diffD / oldFinal) * 100;
            document.getElementById('kpi-diff-declared').innerHTML = 
                `<span class="${diffD >= 0 ? 'up' : 'down'}">${diffD >= 0 ? '▲' : '▼'} ${format(Math.abs(diffD))} (${diffDPct.toFixed(1)}%)</span> vs 舊保單`;

            // 計算損益平衡點 (與舊保單交叉點)
            let breakG = '>20';
            let breakD = '>20';
            
            for(let i=0; i<20; i++) {
                if(breakG === '>20' && data.newGuaranteedValues[i] >= data.oldValues[i]) breakG = i+1;
                if(breakD === '>20' && data.newDeclaredValues[i] >= data.oldValues[i]) breakD = i+1;
            }
            
            document.getElementById('kpi-breakeven').innerText = `${breakG} / ${breakD}`;
        }

        function renderChart(inputs, data) {
            if (typeof Chart === 'undefined') {
                console.error("Chart.js library not loaded");
                return;
            }
            const ctx = document.getElementById('mainChart').getContext('2d');
            
            if (mainChart) mainChart.destroy();
            
            const datasets = [
                {
                    label: '舊保單 (預定)',
                    data: data.oldValues.map(v => Math.round(v/10000)),
                    borderColor: '#718096',
                    borderDash: [5, 5],
                    backgroundColor: 'rgba(113, 128, 150, 0.1)',
                    fill: false,
                    tension: 0.4
                }
            ];

            if (data.oldDeclaredValues && data.oldDeclaredValues.length > 0) {
                datasets.push({
                    label: '舊保單 (宣告)',
                    data: data.oldDeclaredValues.map(v => Math.round(v/10000)),
                    borderColor: '#4a5568',
                    borderDash: [2, 2],
                    backgroundColor: 'rgba(74, 85, 104, 0.1)',
                    fill: false,
                    tension: 0.4
                });
            }

            datasets.push(
                {
                    label: '新保單 (保證)',
                    data: data.newGuaranteedValues.map(v => Math.round(v/10000)),
                    borderColor: '#38a169', // Green
                    backgroundColor: 'rgba(56, 161, 105, 0.1)',
                    fill: false,
                    tension: 0.4,
                    borderWidth: 3
                },
                {
                    label: '新保單 (宣告)',
                    data: data.newDeclaredValues.map(v => Math.round(v/10000)),
                    borderColor: '#dd6b20', // Orange
                    backgroundColor: 'rgba(221, 107, 32, 0.05)',
                    fill: false,
                    tension: 0.4
                },
                {
                    label: '新保單累積總繳保費',
                    data: data.principalLine.map(v => Math.round(v/10000)),
                    borderColor: '#e53e3e', // Red
                    borderWidth: 1,
                    pointRadius: 0,
                    fill: false
                }
            );

            mainChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: data.labels,
                    datasets: datasets
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: {
                        mode: 'index',
                        intersect: false,
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return context.dataset.label + ': ' + context.parsed.y + ' 萬';
                                }
                            }
                        },
                        legend: {
                            position: 'bottom'
                        }
                    },
                    scales: {
                        y: {
                            title: { display: true, text: '金額 (萬台幣)' },
                            grid: { color: '#f1f5f9' }
                        },
                        x: {
                            grid: { display: false }
                        }
                    }
                }
            });
        }

        function renderTable(inputs, data) {
            const tbody = document.getElementById('detailTableBody');
            tbody.innerHTML = '';
            
            // Restore Header for Savings
            const thead = document.querySelector('.comparison-table thead tr');
            if (thead) {
                thead.innerHTML = `
                    <th>年度</th>
                    <th>累積本金</th>
                    <th>舊保單</th>
                    <th>新保單(保證)</th>
                    <th>新保單(宣告)</th>
                    <th>差額(保證)</th>
                    <th>總報酬%(保)</th>
                    <th>年化%(保)</th>
                `;
            }
            
            // 1, 3, 5, 10, 15, 20
            const checkPoints = [0, 2, 4, 9, 14, 19];
            
            checkPoints.forEach(idx => {
                const year = idx + 1;
                const oldVal = data.oldValues[idx];
                const newGVal = data.newGuaranteedValues[idx];
                const newDVal = data.newDeclaredValues[idx];
                const principal = data.principalLine[idx];
                const diff = newGVal - oldVal;
                
                // ROI Calculations (Based on New Guaranteed vs Principal)
                const roi = principal > 0 ? ((newGVal - principal) / principal) * 100 : 0;
                // CAGR approximation
                const cagr = (principal > 0 && newGVal > 0) ? (Math.pow(newGVal / principal, 1/year) - 1) * 100 : 0;

                const row = document.createElement('tr');
                if (diff > 0) row.classList.add('highlight-row');
                
                row.innerHTML = `
                    <td>第 ${year} 年</td>
                    <td>${(principal/10000).toFixed(1)} 萬</td>
                    <td>${(oldVal/10000).toFixed(1)} 萬</td>
                    <td style="color: var(--accent); font-weight:bold">${(newGVal/10000).toFixed(1)} 萬</td>
                    <td style="color: var(--warning)">${(newDVal/10000).toFixed(1)} 萬</td>
                    <td style="color: ${diff >= 0 ? 'var(--accent)' : 'var(--danger)'}">
                        ${diff >= 0 ? '+' : ''}${(diff/10000).toFixed(1)} 萬
                    </td>
                    <td style="color: ${roi >= 0 ? 'var(--accent)' : 'var(--danger)'}">${roi.toFixed(1)}%</td>
                    <td style="color: ${cagr >= 0 ? 'var(--accent)' : 'var(--danger)'}">${cagr.toFixed(1)}%</td>
                `;
                tbody.appendChild(row);
            });
        }

        // --- Investment Functions ---

        function updateInvestmentKPIs(inputs, result) {
            const dataA = result.dataA;
            const dataB = result.dataB;
            const lastIdx = dataA.labels.length - 1;
            const format = (val) => (val / 10000).toFixed(1) + '萬';
            const formatMoney = (val) => Math.round(val).toLocaleString();

            const cards = document.querySelectorAll('.kpi-card');
            if(cards.length < 4) return;
            
            // Card 1: Monthly Income Comparison
            cards[0].querySelector('.kpi-label').innerText = '每月配息 (A vs B)';
            cards[0].querySelector('.kpi-value').innerText = formatMoney(dataB.monthlyCash);
            const diffMonthly = dataB.monthlyCash - dataA.monthlyCash;
            cards[0].querySelector('.kpi-sub').innerHTML = `比方案A ${diffMonthly>=0?'多':'少'} <span class="${diffMonthly>=0?'up':'down'}">${formatMoney(Math.abs(diffMonthly))}</span>`;
            
            // Card 2: Total Benefit 20Y Comparison
            cards[1].querySelector('.kpi-label').innerText = '20年總效益 (A vs B)';
            cards[1].querySelector('.kpi-value').innerText = format(dataB.totalBenefit[lastIdx]);
            const diffTotal = dataB.totalBenefit[lastIdx] - dataA.totalBenefit[lastIdx];
            cards[1].querySelector('.kpi-sub').innerHTML = `比方案A ${diffTotal>=0?'多':'少'} <span class="${diffTotal>=0?'up':'down'}">${format(Math.abs(diffTotal))}</span>`;
            
            // Card 3: ROI Comparison
            const roiA = ((dataA.totalBenefit[lastIdx] - dataA.principal[0]) / dataA.principal[0]) * 100;
            const roiB = ((dataB.totalBenefit[lastIdx] - dataB.principal[0]) / dataB.principal[0]) * 100;
            cards[2].querySelector('.kpi-label').innerText = '20年總報酬率 (B)';
            cards[2].querySelector('.kpi-value').innerText = roiB.toFixed(1) + '%';
            cards[2].querySelector('.kpi-sub').innerHTML = `方案A為 ${roiA.toFixed(1)}%`;
            
            // Card 4: Breakeven / Advantage
            // Find when B beats A in Total Benefit
            let crossYear = '>20';
            for(let i=0; i<=lastIdx; i++) {
                if(dataB.totalBenefit[i] > dataA.totalBenefit[i]) {
                    crossYear = i+1;
                    break;
                }
            }
            cards[3].querySelector('.kpi-label').innerText = '方案B優勢起始點';
            cards[3].querySelector('.kpi-value').innerText = crossYear === '>20' ? '未超越' : `第 ${crossYear} 年`;
            cards[3].querySelector('.kpi-sub').innerText = '總效益(含息)超越A點';
        }

        function renderInvestmentChart(inputs, result) {
            if (typeof Chart === 'undefined') return;
            const ctx = document.getElementById('mainChart').getContext('2d');
            if (mainChart) mainChart.destroy();
            
            const dataA = result.dataA;
            const dataB = result.dataB;

            mainChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: dataA.labels,
                    datasets: [
                        {
                            label: '方案A 總效益',
                            data: dataA.totalBenefit.map(v => Math.round(v/10000)),
                            borderColor: '#718096',
                            borderDash: [5, 5],
                            backgroundColor: 'rgba(113, 128, 150, 0.1)',
                            fill: false,
                            tension: 0.4
                        },
                        {
                            label: '方案B 總效益',
                            data: dataB.totalBenefit.map(v => Math.round(v/10000)),
                            borderColor: '#38a169',
                            backgroundColor: 'rgba(56, 161, 105, 0.1)',
                            fill: false,
                            tension: 0.4,
                            borderWidth: 3
                        },
                        {
                            label: '方案A 累積配息',
                            data: dataA.cumulativeCash.map(v => Math.round(v/10000)),
                            borderColor: '#cbd5e0',
                            borderDash: [2, 2],
                            fill: false,
                            hidden: true // Hide by default to avoid clutter
                        },
                        {
                            label: '方案B 累積配息',
                            data: dataB.cumulativeCash.map(v => Math.round(v/10000)),
                            borderColor: '#dd6b20',
                            backgroundColor: 'rgba(221, 107, 32, 0.05)',
                            fill: true,
                            tension: 0.4
                        }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: { mode: 'index', intersect: false },
                    plugins: {
                        tooltip: { callbacks: { label: c => c.dataset.label + ': ' + c.parsed.y + ' 萬' } },
                        legend: { position: 'bottom' }
                    },
                    scales: {
                        y: { title: { display: true, text: '金額 (萬台幣)' }, stacked: false }
                    }
                }
            });
        }

        function renderInvestmentTable(inputs, result) {
            const tbody = document.getElementById('detailTableBody');
            tbody.innerHTML = '';
            // 1, 3, 5, 10, 15, 20
            const checkPoints = [0, 2, 4, 9, 14, 19];
            const dataA = result.dataA;
            const dataB = result.dataB;
            
            // Update Header
            const thead = document.querySelector('.comparison-table thead tr');
            if (thead) {
                thead.innerHTML = `
                    <th>年度</th>
                    <th>A 總效益</th>
                    <th>B 總效益</th>
                    <th>效益差額</th>
                    <th>A 月領</th>
                    <th>B 月領</th>
                    <th>A 總報酬%</th>
                    <th>B 總報酬%</th>
                `;
            }

            checkPoints.forEach(idx => {
                const year = idx + 1;
                const tbA = dataA.totalBenefit[idx];
                const tbB = dataB.totalBenefit[idx];
                const diff = tbB - tbA;
                
                // ROI for Plan A & B
                const principalA = dataA.principal[idx];
                const principalB = dataB.principal[idx];
                const roiA = ((tbA - principalA) / principalA) * 100;
                const roiB = ((tbB - principalB) / principalB) * 100;

                const row = document.createElement('tr');
                if (diff > 0) row.classList.add('highlight-row');
                
                row.innerHTML = `
                    <td>第 ${year} 年</td>
                    <td>${(tbA/10000).toFixed(1)} 萬</td>
                    <td style="color:var(--accent);font-weight:bold">${(tbB/10000).toFixed(1)} 萬</td>
                    <td style="color:${diff>=0?'var(--accent)':'var(--danger)'}">${diff>=0?'+':''}${(diff/10000).toFixed(1)} 萬</td>
                    <td style="color:#718096">${Math.round(dataA.monthlyCash).toLocaleString()}</td>
                    <td style="color:var(--warning);font-weight:bold">${Math.round(dataB.monthlyCash).toLocaleString()}</td>
                    <td style="color:${roiA>=0?'var(--accent)':'var(--danger)'}">${roiA.toFixed(1)}%</td>
                    <td style="color:${roiB>=0?'var(--accent)':'var(--danger)'}">${roiB.toFixed(1)}%</td>
                `;
                tbody.appendChild(row);
            });
        }

        function generateInvestmentReport(inputs, result) {
            const dataA = result.dataA;
            const dataB = result.dataB;
            
            const diffMonthly = dataB.monthlyCash - dataA.monthlyCash;
            
            const format = (val) => (val / 10000).toFixed(1) + '萬';
            const formatMoney = (val) => Math.round(val).toLocaleString();

            // Years to compare: 3, 5, 10
            const years = [3, 5, 10];
            let comparisonHtml = '<table style="width:100%; margin-top:10px; font-size:0.95rem; border-collapse: collapse;"><tr><th style="text-align:left; padding:5px; border-bottom:1px solid #eee">年期</th><th style="padding:5px; border-bottom:1px solid #eee">方案A總效益</th><th style="padding:5px; border-bottom:1px solid #eee">方案B總效益</th><th style="padding:5px; border-bottom:1px solid #eee">差額</th></tr>';
            
            years.forEach(y => {
                const idx = y - 1;
                if (idx < dataA.totalBenefit.length) {
                    const valA = dataA.totalBenefit[idx];
                    const valB = dataB.totalBenefit[idx];
                    const diff = valB - valA;
                    const colorClass = diff >= 0 ? 'text-green' : 'text-red';
                    comparisonHtml += `<tr>
                        <td style="padding:5px; border-bottom:1px solid #eee">第 ${y} 年</td>
                        <td style="padding:5px; border-bottom:1px solid #eee; text-align:center">${format(valA)}</td>
                        <td style="padding:5px; border-bottom:1px solid #eee; text-align:center">${format(valB)}</td>
                        <td style="padding:5px; border-bottom:1px solid #eee; text-align:center" class="${colorClass}">${diff>=0?'+':''}${format(diff)}</td>
                    </tr>`;
                }
            });
            comparisonHtml += '</table>';

            // Use 10th year for final advice if available, else last year
            const adviceIdx = Math.min(9, dataA.totalBenefit.length - 1);
            const diffTotal = dataB.totalBenefit[adviceIdx] - dataA.totalBenefit[adviceIdx];

            let html = `
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">💰</span> 現金流比較</div>
                    <p>方案A每月領息 <span class="highlight-text">${formatMoney(dataA.monthlyCash)}</span>，
                    方案B每月領息 <span class="highlight-text text-orange">${formatMoney(dataB.monthlyCash)}</span>。</p>
                    <p>若選擇方案B，您每月可多領 <span class="highlight-text ${diffMonthly>=0?'text-green':'text-red'}">${formatMoney(Math.abs(diffMonthly))}</span> 元。</p>
                </div>
                
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">📈</span> 階段性總效益比較 (含息)</div>
                    ${comparisonHtml}
                </div>
                
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">💡</span> 綜合建議</div>
                    <p>${diffTotal > 0 ? '✅ <strong>建議採用方案B</strong>：無論是每月現金流還是中長期(10年)總資產累積，方案B都展現了更好的效益。' : '🛑 <strong>建議維持方案A</strong>：方案B在效益上並未超越目前的方案A。'}</p>
                </div>
            `;
            document.getElementById('reportContent').innerHTML = html;
        }

        function resetData() {
            if(confirm('確定要重置所有數據嗎？')) {
                location.reload();
            }
        }

        function printReport() {
            window.print();
        }
        
        function exportPDF() {
            alert('PDF 匯出功能在此演示版本中暫時使用瀏覽器列印功能替代。請使用列印 -> 另存為 PDF。');
            window.print();
        }

        function generateReport(inputs, data) {
            const lastIdx = data.labels.length - 1;
            const oldFinal = data.oldValues[lastIdx];
            const newGFinal = data.newGuaranteedValues[lastIdx];
            const newDFinal = data.newDeclaredValues[lastIdx];
            const diffG = newGFinal - oldFinal;
            const diffD = newDFinal - oldFinal;
            
            // 找回本點
            let breakG = '>20';
            let breakD = '>20';
            for(let i=0; i<20; i++) {
                if(breakG === '>20' && data.newGuaranteedValues[i] >= data.oldValues[i]) breakG = i+1;
                if(breakD === '>20' && data.newDeclaredValues[i] >= data.oldValues[i]) breakD = i+1;
            }

            const format = (val) => (val / 10000).toFixed(1) + '萬';
            
            let html = '';
            
            // 1. 現狀分析
            html += `
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">🏦</span> 舊保單現狀分析</div>
                    <p>您目前的舊保單現金價值為 <span class="highlight-text">${format(inputs.oldCashValue * (inputs.oldCurrency==='USD'?inputs.exchangeRate:10000))}</span> (台幣估算)。
                    若維持現狀不變，以預定利率 <span class="highlight-text">${(inputs.oldRate*100).toFixed(1)}%</span> 複利滾存，
                    20年後預計價值為 <span class="highlight-text">${format(oldFinal)}</span>。</p>
                </div>
            `;
            
            // 2. 轉換效益 (保證)
            const gClass = diffG >= 0 ? 'text-green' : 'text-red';
            const gWord = diffG >= 0 ? '增加' : '減少';
            
            html += `
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">🛡️</span> 新方案：保守評估 (保證利率)</div>
                    <p>若轉換至新方案，在最保守的 <span class="highlight-text">${(inputs.newGuaranteedRate*100).toFixed(1)}%</span> 保證利率下，
                    20年後價值將達到 <span class="highlight-text ${gClass}">${format(newGFinal)}</span>。
                    相較於舊保單，您的資產將<span class="${gClass}">${gWord} ${format(Math.abs(diffG))}</span>。</p>
                    <p>此方案預計在第 <span class="highlight-text">${breakG}</span> 年發生黃金交叉 (價值超越舊保單)。</p>
                </div>
            `;
            
            // 3. 轉換效益 (宣告)
            const dClass = diffD >= 0 ? 'text-orange' : 'text-red';
            html += `
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">📈</span> 新方案：預期評估 (宣告利率)</div>
                    <p>若市場狀況良好，維持目前 <span class="highlight-text">${(inputs.newDeclaredRate*100).toFixed(1)}%</span> 宣告利率，
                    20年後價值有機會達到 <span class="highlight-text ${dClass}">${format(newDFinal)}</span>。
                    這將比舊保單多出 <span class="${dClass}">${format(Math.abs(diffD))}</span> 的潛在獲利空間。</p>
                    <p>預期黃金交叉時間為第 <span class="highlight-text">${breakD}</span> 年。</p>
                </div>
            `;
            
            // 4. 綜合建議
            let advice = '';
            if (diffG > 0) {
                advice = "✅ <strong>強烈建議轉換</strong>：即使在最保守的保證利率下，新保單的效益也優於舊保單，且提供了更高的潛在獲利空間。";
            } else if (diffD > 0 && breakG <= 10) {
                advice = "⚠️ <strong>建議轉換</strong>：雖然保證收益略低或持平，但宣告利率帶來的潛在獲利極具吸引力，且回本期在合理範圍內。";
            } else {
                advice = "🛑 <strong>審慎評估</strong>：新保單的保證效益未明顯優於舊保單，建議視您對資金流動性與保障的需求再做決定。";
            }
            
            html += `
                <div class="report-block">
                    <div class="report-title"><span class="report-icon">💡</span> 綜合建議</div>
                    <p>${advice}</p>
                </div>
            `;
            
            document.getElementById('reportContent').innerHTML = html;
        }
    </script>
</body>
</html>
