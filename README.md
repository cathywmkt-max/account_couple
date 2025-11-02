<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>致命清算 (GitHub 版)</title>
  <!-- 載入 Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* 為了讓 Tailwind 能夠使用你指定的主題色 */
    .bg-theme-blue { background-color: #A4B1BA; }
    .hover\:bg-theme-blue-dark:hover { background-color: #93a0a9; }
    .focus\:ring-theme-blue:focus { --tw-ring-color: #A4B1BA; }
    .focus\:border-theme-blue:focus { --tw-border-color: #A4B1BA; }
    .text-theme-blue { color: #A4B1BA; }
    .border-theme-blue { border-color: #A4B1BA; }

    /* 彈窗樣式 */
    .modal-backdrop {
      background-color: rgba(0, 0, 0, 0.5);
    }
  </style>
</head>
<body class="bg-gray-100 font-sans">

  <!-- 訊息提示框 -->
  <div id="message-modal" class="fixed inset-0 z-[60] flex items-start justify-center pt-12 transition-opacity duration-300 opacity-0 pointer-events-none">
    <div id="message-content" class="border px-4 py-3 rounded-lg shadow-xl" role="alert">
      <span id="message-text" class="block sm:inline"></span>
    </div>
  </div>

  <!-- 主應用程式 -->
  <div id="app-container" class="max-w-3xl mx-auto p-4 md:p-8">
    
    <h1 class="text-3xl md:text-4xl font-bold text-center text-gray-800 mb-6">
      致命清算
    </h1>

    <!-- 頁面切換 -->
    <div class="mb-6">
      <div class="flex border-b border-gray-300">
        <button id="tab-log" class="py-3 px-6 text-lg font-medium transition-colors duration-150 border-b-2 border-theme-blue text-theme-blue">
          支出記錄
        </button>
        <button id="tab-summary" class="py-3 px-6 text-lg font-medium transition-colors duration-150 text-gray-500 hover:text-gray-700">
          每月結算
        </button>
      </div>
    </div>

    <!-- 內容容器 -->
    <div id="page-content">
      
      <!-- 1. 支出記錄頁面 -->
      <div id="log-page" class="space-y-6">
        <!-- 支出記錄表單 -->
        <form id="expense-form" class="p-6 bg-white rounded-xl shadow-lg space-y-4">
          <h2 class="text-2xl font-semibold text-gray-800 mb-4">記錄一筆新支出</h2>
          
          <div>
            <label for="date" class="block text-sm font-medium text-gray-700">日期</label>
            <input type="date" id="date" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label for="description" class="block text-sm font-medium text-gray-700">項目說明</label>
            <input type="text" id="description" placeholder="例如：晚餐、電影票..." class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label for="amount" class="block text-sm font-medium text-gray-700">金額</label>
            <input type="number" id="amount" placeholder="0.00" step="0.01" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700">付款人</label>
            <div id="payer-options" class="mt-2 flex space-x-4">
              <!-- 付款人選項會由 JS 動態生成 -->
            </div>
          </div>

          <button type="submit" class="w-full py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-theme-blue hover:bg-theme-blue-dark focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-theme-blue transition duration-150">
            儲存支出
          </button>
        </form>

      </div>

      <!-- 2. 結算摘要頁面 (預設隱藏) -->
      <div id="summary-page" class="p-6 bg-white rounded-xl shadow-lg space-y-6 hidden">
        <h2 class="text-2xl font-semibold text-gray-800">每月結算</h2>

        <!-- 月份選擇 -->
        <div>
          <label for="month-select" class="block text-sm font-medium text-gray-700">選擇月份</label>
          <select id="month-select" class="mt-1 block w-full px-3 py-2 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue">
            <!-- 月份選項會由 JS 動態生成 -->
          </select>
        </div>

        <!-- 摘要卡片 -->
        <div class="space-y-4">
          <!-- 皮卡蟲 總結 -->
          <div id="summary-p1-container" class="p-4 bg-gray-100 rounded-lg cursor-pointer hover:bg-gray-200 transition">
            <div class="flex justify-between">
              <span id="summary-p1-name" class="font-medium text-gray-700"></span>
              <span id="summary-p1-total" class="font-semibold text-lg text-gray-900">NT$0.00</span>
            </div>
          </div>
          <!-- 皮卡蟲 明細 (預設隱藏) -->
          <div id="summary-details-p1" class="hidden space-y-3 pt-2 pb-4 px-4 border-l-4 border-gray-200">
            <h4 id="summary-details-p1-title" class="text-lg font-semibold text-gray-700"></h4>
            <ul id="summary-details-p1-list" class="space-y-3">
              <!-- P1 明細會由 JS 動態生成 -->
            </ul>
          </div>

          <!-- 王小貓 總結 -->
          <div id="summary-p2-container" class="p-4 bg-gray-100 rounded-lg cursor-pointer hover:bg-gray-200 transition">
            <div class="flex justify-between">
              <span id="summary-p2-name" class="font-medium text-gray-700"></span>
              <span id="summary-p2-total" class="font-semibold text-lg text-gray-900">NT$0.00</span>
            </div>
          </div>
          <!-- 王小貓 明細 (預設隱藏) -->
          <div id="summary-details-p2" class="hidden space-y-3 pt-2 pb-4 px-4 border-l-4 border-gray-200">
            <h4 id="summary-details-p2-title" class="text-lg font-semibold text-gray-700"></h4>
            <ul id="summary-details-p2-list" class="space-y-3">
              <!-- P2 明細會由 JS 動態生成 -->
            </ul>
          </div>
        </div>

        <!-- 結算結果 -->
        <div class="text-center p-6 bg-theme-blue rounded-lg">
          <p class="text-lg font-medium text-white opacity-90">結算結果：</p>
          <p id="summary-result-text" class="text-2xl font-bold text-white mt-2">請選擇月份</p>
        </div>
      </div>
      
      <!-- App ID 顯示區塊 (已移除) -->

    </div>
  </div>

  <!-- 編輯彈窗 (Modal) -->
  <div id="edit-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 modal-backdrop hidden">
    <div class="bg-white rounded-xl shadow-2xl w-full max-w-lg">
      <form id="edit-form">
        <div class="p-6 space-y-4">
          <h3 class="text-2xl font-semibold text-gray-800">編輯支出</h3>
          
          <div>
            <label for="edit-date" class="block text-sm font-medium text-gray-700">日期</label>
            <input type="date" id="edit-date" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label for="edit-description" class="block text-sm font-medium text-gray-700">項目說明</label>
            <input type="text" id="edit-description" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label for="edit-amount" class="block text-sm font-medium text-gray-700">金額</label>
            <input type="number" id="edit-amount" step="0.01" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-theme-blue focus:border-theme-blue" required>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700">付款人</label>
            <div id="edit-payer-options" class="mt-2 flex space-x-4">
              <!-- 編輯時的付款人選項會由 JS 動態生成 -->
            </div>
          </div>
        </div>

        <div class="flex justify-end space-x-3 bg-gray-50 p-4 rounded-b-xl">
          <button type="button" id="edit-cancel-btn" class="py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-400">
            取消
          </button>
          <button type="submit" class="py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-theme-blue hover:bg-theme-blue-dark focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-theme-blue">
            儲存變更
          </button>
        </div>
      </form>
    </div>
  </div>

  <!-- 刪除確認彈窗 (Modal) -->
  <div id="delete-modal" class="fixed inset-0 z-50 flex items-center justify-center p-4 modal-backdrop hidden">
    <div class="bg-white rounded-xl shadow-2xl w-full max-w-sm">
      <div class="p-6">
        <h3 class="text-xl font-semibold text-gray-900">確認刪除</h3>
        <p class="mt-2 text-sm text-gray-600">你確定要刪除這筆支出嗎？此操作無法復原。</p>
      </div>
      <div class="flex justify-end space-x-3 bg-gray-50 p-4 rounded-b-xl">
        <button type="button" id="delete-cancel-btn" class="py-2 px-4 border border-gray-300 rounded-md shadow-sm text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-400">
          取消
        </button>
        <button type="button" id="delete-confirm-btn" class="py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-rose-600 hover:bg-rose-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-rose-500">
          確認刪除
        </button>
      </div>
    </div>
  </div>


  <!-- 應用程式邏輯 (GitHub/獨立版) -->
  <script type="module">
    // --- 載入 Firebase SDK ---
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
    import { getAuth, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
    import { getFirestore, collection, doc, addDoc, updateDoc, deleteDoc, onSnapshot, query, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

    // ========================================================================
    // --- (1) 重要：貼上你自己的 Firebase 設定 ---
    //
    // 1. 前往 https://firebase.google.com/ 建立你自己的專案。
    // 2. 在專案中「新增應用程式」，選擇「網頁」(</>)。
    // 3. 複製 Firebase 提供的 `firebaseConfig` 物件。
    // 4. 把它貼到下面來取代 `YOUR_FIREBASE_CONFIG_HERE`。
    //
    // ========================================================================
   const firebaseConfig = {
  apiKey: "AIzaSyDjsL6fIGP7qoc5MIrbP--iM-sS0xjKUmE",
  authDomain: "account-couple.firebaseapp.com",
  projectId: "account-couple",
  storageBucket: "account-couple.firebasestorage.app",
  messagingSenderId: "562403319331",
  appId: "1:562403319331:web:1ce99add6ff6b217d3db96",
  measurementId: "G-SBRBLEBSF3"
};

    // ========================================================================


    // --- Firebase 變數 ---
    let app, auth, db, userId, expensesCollectionRef, expensesCollectionPath;

    // --- 全局狀態變數 ---
    let expenses = []; // 將由 onSnapshot 即時更新
    let person1 = localStorage.getItem('person1') || '皮卡蟲';
    let person2 = localStorage.getItem('person2') || '王小貓';
    let currentPage = 'log'; // 'log' 或 'summary'
    let selectedMonth = new Date().toISOString().slice(0, 7);
    let editingExpenseId = null;
    let deletingExpenseId = null;

    // --- DOM 元素參照 ---
    const appContainer = document.getElementById('app-container');
    const logPage = document.getElementById('log-page');
    const summaryPage = document.getElementById('summary-page');
    const tabLog = document.getElementById('tab-log');
    const tabSummary = document.getElementById('tab-summary');
    const expenseForm = document.getElementById('expense-form');
    const dateInput = document.getElementById('date');
    const descriptionInput = document.getElementById('description');
    const amountInput = document.getElementById('amount');
    const payerOptionsContainer = document.getElementById('payer-options');
    const monthSelect = document.getElementById('month-select');
    const summaryP1Container = document.getElementById('summary-p1-container');
    const summaryP1Name = document.getElementById('summary-p1-name');
    const summaryP1Total = document.getElementById('summary-p1-total');
    const summaryDetailsP1 = document.getElementById('summary-details-p1');
    const summaryDetailsP1Title = document.getElementById('summary-details-p1-title');
    const summaryDetailsP1List = document.getElementById('summary-details-p1-list');
    const summaryP2Container = document.getElementById('summary-p2-container');
    const summaryP2Name = document.getElementById('summary-p2-name');
    const summaryP2Total = document.getElementById('summary-p2-total');
    const summaryDetailsP2 = document.getElementById('summary-details-p2');
    const summaryDetailsP2Title = document.getElementById('summary-details-p2-title');
    const summaryDetailsP2List = document.getElementById('summary-details-p2-list');
    const summaryResultText = document.getElementById('summary-result-text');
    const messageModal = document.getElementById('message-modal');
    const messageContent = document.getElementById('message-content');
    const messageText = document.getElementById('message-text');
    const editModal = document.getElementById('edit-modal');
    const editForm = document.getElementById('edit-form');
    const editDate = document.getElementById('edit-date');
    const editDescription = document.getElementById('edit-description');
    const editAmount = document.getElementById('edit-amount');
    const editPayerOptions = document.getElementById('edit-payer-options');
    const editCancelBtn = document.getElementById('edit-cancel-btn');
    const deleteModal = document.getElementById('delete-modal');
    const deleteCancelBtn = document.getElementById('delete-cancel-btn');
    const deleteConfirmBtn = document.getElementById('delete-confirm-btn');
    // (App ID DOM 參照已移除)


    // --- 訊息提示框 ---
    function showMessage(text, type) {
      messageText.textContent = text;
      messageContent.classList.remove('bg-teal-100', 'border-teal-400', 'text-teal-700', 'bg-rose-100', 'border-rose-400', 'text-rose-700');
      if (type === 'success') {
        messageContent.classList.add('bg-teal-100', 'border-teal-400', 'text-teal-700');
      } else {
        messageContent.classList.add('bg-rose-100', 'border-rose-400', 'text-rose-700');
      }
      messageModal.classList.remove('opacity-0', 'pointer-events-none');
      setTimeout(() => {
        messageModal.classList.add('opacity-0', 'pointer-events-none');
      }, 3000);
    }
    
    // --- 本地儲存 (localStorage) ---
    function savePayerNames() {
        localStorage.setItem('person1', person1);
        localStorage.setItem('person2', person2);
    }

    // --- 渲染函式 ---

    // 渲染付款人選項 (表單)
    function renderPayerOptions() {
      payerOptionsContainer.innerHTML = `
        <label class="flex items-center">
          <input type="radio" name="payer" value="${person1}" checked class="focus:ring-theme-blue h-4 w-4 text-theme-blue border-gray-300">
          <span class="ml-2 text-gray-700">${person1}</span>
        </label>
        <label class="flex items-center">
          <input type="radio" name="payer" value="${person2}" class="focus:ring-theme-blue h-4 w-4 text-theme-blue border-gray-300">
          <span class="ml-2 text-gray-700">${person2}</span>
        </label>
      `;
    }

    // 渲染結算頁面 (月份 + 摘要 + 明細)
    function renderSummaryPage() {
      const availableMonths = getAvailableMonths();
      if (!availableMonths.includes(selectedMonth)) {
        selectedMonth = availableMonths[0] || new Date().toISOString().slice(0, 7);
      }
      
      if (availableMonths.length === 0) {
          monthSelect.innerHTML = `<option value="${selectedMonth}">${selectedMonth}</option>`;
      } else {
          monthSelect.innerHTML = availableMonths.map(month => 
            `<option value="${month}" ${month === selectedMonth ? 'selected' : ''}>${month}</option>`
          ).join('');
      }

      const summary = calculateSummary();
      summaryP1Name.textContent = `${person1} 總支付`;
      summaryP1Total.textContent = `NT$${summary.totalP1.toFixed(2)}`;
      summaryP2Name.textContent = `${person2} 總支付`;
      summaryP2Total.textContent = `NT$${summary.totalP2.toFixed(2)}`;
      summaryResultText.textContent = summary.resultText;

      const monthExpenses = expenses
        .filter(e => e.date.startsWith(selectedMonth))
        .sort((a, b) => new Date(b.date) - new Date(a.date));
      
      const p1MonthExpenses = monthExpenses.filter(e => e.payer === person1);
      const p2MonthExpenses = monthExpenses.filter(e => e.payer === person2);

      summaryDetailsP1Title.textContent = `${person1} - ${selectedMonth} 明細 (${p1MonthExpenses.length} 筆)`;
      summaryDetailsP2Title.textContent = `${person2} - ${selectedMonth} 明細 (${p2MonthExpenses.length} 筆)`;

      const editIcon = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5"><path stroke-linecap="round" stroke-linejoin="round" d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10" /></svg>`;
      const deleteIcon = `<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5"><path stroke-linecap="round" stroke-linejoin="round" d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12.578 0c-.27.004-.537.01-.804.018c-3.17 0-5.714 2.543-5.714 5.714v.685c0 3.17 2.543 5.714 5.714 5.714h8.571c3.17 0 5.714-2.543 5.714-5.714v-.685c0-3.17-2.543-5.714-5.714-5.714H8.778c-.271 0-.538-.01-.804-.018Z" /></svg>`;

      if (p1MonthExpenses.length === 0) {
          summaryDetailsP1List.innerHTML = '<li class="text-gray-500">本月無支出明細。</li>';
      } else {
          summaryDetailsP1List.innerHTML = p1MonthExpenses.map(expense => `
            <li class="flex justify-between items-center p-3 bg-gray-50 rounded-md shadow-sm">
              <div>
                <p class="font-medium text-gray-900">${escapeHTML(expense.description)}</p>
                <p class="text-sm text-gray-500">${expense.date}</p>
              </div>
              <div class="flex items-center space-x-2 flex-shrink-0">
                <span class="text-lg font-semibold text-gray-800">NT$${expense.amount.toFixed(2)}</span>
                <button title="編輯" class="edit-btn p-1 text-blue-600 hover:text-blue-800 transition" data-id="${expense.id}">
                  ${editIcon}
                </button>
                <button title="刪除" class="delete-btn p-1 text-rose-600 hover:text-rose-800 transition" data-id="${expense.id}">
                  ${deleteIcon}
                </button>
              </div>
            </li>
          `).join('');
      }
      
      if (p2MonthExpenses.length === 0) {
          summaryDetailsP2List.innerHTML = '<li class="text-gray-500">本月無支出明細。</li>';
      } else {
          summaryDetailsP2List.innerHTML = p2MonthExpenses.map(expense => `
            <li class="flex justify-between items-center p-3 bg-gray-50 rounded-md shadow-sm">
              <div>
                <p class="font-medium text-gray-900">${escapeHTML(expense.description)}</p>
                <p class="text-sm text-gray-500">${expense.date}</p>
              </div>
              <div class="flex items-center space-x-2 flex-shrink-0">
                <span class="text-lg font-semibold text-gray-800">NT$${expense.amount.toFixed(2)}</span>
                <button title="編輯" class="edit-btn p-1 text-blue-600 hover:text-blue-800 transition" data-id="${expense.id}">
                  ${editIcon}
                </button>
                <button title="刪除" class="delete-btn p-1 text-rose-600 hover:text-rose-800 transition" data-id="${expense.id}">
                  ${deleteIcon}
                </button>
              </div>
            </li>
          `).join('');
      }
    }
    
    // 渲染當前頁面 (切換)
    function renderPage() {
      if (currentPage === 'log') {
        logPage.classList.remove('hidden');
        summaryPage.classList.add('hidden');
        tabLog.className = 'py-3 px-6 text-lg font-medium transition-colors duration-150 border-b-2 border-theme-blue text-theme-blue';
        tabSummary.className = 'py-3 px-6 text-lg font-medium transition-colors duration-150 text-gray-500 hover:text-gray-700';
        renderPayerOptions(); 
      } else {
        logPage.classList.add('hidden');
        summaryPage.classList.remove('hidden');
        tabLog.className = 'py-3 px-6 text-lg font-medium transition-colors duration-150 text-gray-500 hover:text-gray-700';
        tabSummary.className = 'py-3 px-6 text-lg font-medium transition-colors duration-150 border-b-2 border-theme-blue text-theme-blue';
        renderSummaryPage(); 
      }
    }

    // --- 輔助函式 ---
    function escapeHTML(str) {
      if (typeof str !== 'string') return '';
      return str.replace(/[&<>"']/g, function(m) {
        return {
          '&': '&amp;',
          '<': '&lt;',
          '>': '&gt;',
          '"': '&quot;',
          "'": '&#39;'
        }[m];
      });
    }

    function getAvailableMonths() {
      const months = new Set(expenses.map(e => e.date.slice(0, 7)));
      months.add(new Date().toISOString().slice(0, 7)); 
      return Array.from(months).sort().reverse();
    }

    function calculateSummary() {
      const monthExpenses = expenses.filter(e => e.date.startsWith(selectedMonth));
      let totalP1 = 0;
      let totalP2 = 0;
      monthExpenses.forEach(e => {
        if (e.payer === person1) {
          totalP1 += e.amount;
        } else if (e.payer === person2) {
          totalP2 += e.amount;
        }
      });
      let resultText = "兩人帳目已平衡";
      const diff = Math.abs(totalP1 - totalP2);
      if (diff > 0.001) { 
        if (totalP1 > totalP2) {
          resultText = `${person2} 應該給 ${person1} NT$${diff.toFixed(2)}`;
        } else {
          resultText = `${person1} 應該給 ${person2} NT$${diff.toFixed(2)}`;
        }
      }
      return { totalP1, totalP2, resultText };
    }
    
    // --- 彈窗 (Modal) 邏輯 ---
    function openEditModal(id) {
      const expense = expenses.find(e => e.id === id);
      if (!expense) return;
      editingExpenseId = id;
      editDate.value = expense.date;
      editDescription.value = expense.description;
      editAmount.value = expense.amount;
      editPayerOptions.innerHTML = `
        <label class="flex items-center">
          <input type="radio" name="edit-payer" value="${person1}" ${expense.payer === person1 ? 'checked' : ''} class="focus:ring-theme-blue h-4 w-4 text-theme-blue border-gray-300">
          <span class="ml-2 text-gray-700">${person1}</span>
        </label>
        <label class="flex items-center">
          <input type="radio" name="edit-payer" value="${person2}" ${expense.payer === person2 ? 'checked' : ''} class="focus:ring-theme-blue h-4 w-4 text-theme-blue border-gray-300">
          <span class="ml-2 text-gray-700">${person2}</span>
        </label>
      `;
      editModal.classList.remove('hidden');
    }

    function closeEditModal() {
      editingExpenseId = null;
      editModal.classList.add('hidden');
    }

    async function handleEditSubmit(e) {
      e.preventDefault();
      const payer = document.querySelector('input[name="edit-payer"]:checked')?.value;
      const parsedAmount = parseFloat(editAmount.value);
      if (!payer || !editDate.value || !editDescription.value || isNaN(parsedAmount) || parsedAmount <= 0) {
        showMessage('請確保所有欄位都已正確填寫', 'error');
        return;
      }
      const updatedData = {
        date: editDate.value,
        description: editDescription.value,
        amount: parsedAmount,
        payer: payer
      };
      try {
        const docRef = doc(db, expensesCollectionPath, editingExpenseId);
        await updateDoc(docRef, updatedData);
        showMessage('支出已成功更新！', 'success');
        closeEditModal();
      } catch (error) {
        console.error("更新支出時出錯:", error);
        showMessage('更新失敗，請稍後再試', 'error');
      }
    }

    function openDeleteModal(id) {
      deletingExpenseId = id;
      deleteModal.classList.remove('hidden');
    }

    function closeDeleteModal() {
      deletingExpenseId = null;
      deleteModal.classList.add('hidden');
    }

    async function handleDeleteConfirm() {
      try {
        const docRef = doc(db, expensesCollectionPath, deletingExpenseId);
        await deleteDoc(docRef);
        showMessage('支出已刪除', 'success');
        closeDeleteModal();
      } catch (error) {
        console.error("刪除支出時出錯:", error);
        showMessage('刪除失敗，請稍後再試', 'error');
      }
    }

    // --- 事件監聽器 ---
    tabLog.addEventListener('click', () => {
      currentPage = 'log';
      renderPage();
    });

    tabSummary.addEventListener('click', () => {
      currentPage = 'summary';
      renderPage();
    });

    summaryP1Container.addEventListener('click', () => {
      summaryDetailsP1.classList.toggle('hidden');
    });
    summaryP2Container.addEventListener('click', () => {
      summaryDetailsP2.classList.toggle('hidden');
    });

    summaryPage.addEventListener('click', (e) => {
      const editBtn = e.target.closest('.edit-btn');
      if (editBtn) {
        openEditModal(editBtn.dataset.id);
        return;
      }
      const deleteBtn = e.target.closest('.delete-btn');
      if (deleteBtn) {
        openDeleteModal(deleteBtn.dataset.id);
        return;
      }
    });

    expenseForm.addEventListener('submit', async (e) => {
      e.preventDefault();
      const date = dateInput.value;
      const description = descriptionInput.value;
      const amount = amountInput.value;
      const payer = document.querySelector('input[name="payer"]:checked')?.value;
      if (!description || !amount || !date || !payer) {
        showMessage('請填寫所有欄位', 'error');
        return;
      }
      const parsedAmount = parseFloat(amount);
      if (isNaN(parsedAmount) || parsedAmount <= 0) {
        showMessage('金額必須是正數', 'error');
        return;
      }
      try {
        const newExpense = {
          date,
          description,
          amount: parsedAmount,
          payer,
          createdAt: Date.now()
        };
        await addDoc(expensesCollectionRef, newExpense);
        descriptionInput.value = '';
        amountInput.value = '';
        dateInput.value = new Date().toISOString().split('T')[0];
        showMessage('支出已成功記錄！', 'success');
      } catch (error) {
        console.error("儲存支出時出錯:", error);
        showMessage('記錄失敗，請稍後再試', 'error');
      }
    });

    monthSelect.addEventListener('change', (e) => {
      selectedMonth = e.target.value;
      renderSummaryPage();
    });

    editForm.addEventListener('submit', handleEditSubmit);
    editCancelBtn.addEventListener('click', closeEditModal);
    deleteConfirmBtn.addEventListener('click', handleDeleteConfirm);
    deleteCancelBtn.addEventListener('click', closeDeleteModal);

    // --- Firebase 監聽器 ---
    function listenForExpenses() {
      if (!expensesCollectionRef) return;
      const q = query(expensesCollectionRef);
      onSnapshot(q, (snapshot) => {
        expenses = snapshot.docs.map(doc => ({
          ...doc.data(),
          id: doc.id,
          amount: parseFloat(doc.data().amount) || 0
        }));
        if (currentPage === 'summary') {
          renderSummaryPage();
        }
      }, (error) => {
        console.error("監聽支出時出錯:", error);
        showMessage("無法載入資料，請檢查網路連線", "error");
      });
    }

    // --- 應用程式初始化 (GitHub/獨立版) ---
    async function initApp() { // <--- 修正：將函式改名為 initApp
      try {
        // 檢查用戶是否已貼上 config
        if (!firebaseConfig || !firebaseConfig.apiKey) {
            throw new Error("Firebase 設定無效。請編輯 HTML 檔案並填入你自己的 firebaseConfig。");
        }

        // 初始化 Firebase
        app = initializeApp(firebaseConfig);
        db = getFirestore(app);
        auth = getAuth(app);
        setLogLevel('Debug'); // 開啟 Firebase 除錯日誌

        // 登入 (僅使用匿名登入)
        await signInAnonymously(auth);
        userId = auth.currentUser?.uid || crypto.randomUUID();
        
        // (App ID 顯示已移除)

        // 設定資料庫路徑 (使用一個簡單的公共路徑)
        expensesCollectionPath = `/public/data/expenses`;
        expensesCollectionRef = collection(db, expensesCollectionPath);
        
        // 設置預設日期
        dateInput.value = new Date().toISOString().split('T')[0];
        
        // 首次渲染
        renderPage();
        
        // 開始監聽雲端資料
        listenForExpenses();

      } catch (error) {
        console.error("應用程式初始化失敗:", error);
        appContainer.innerHTML = `<div class="p-6 bg-rose-100 text-rose-700 rounded-lg shadow-lg">
          <h2 class="font-bold text-xl mb-2">應用程式初始化失敗</h2>
          <p>請檢查你的 Firebase 設定是否已正確貼入 HTML 檔案中。</p>
          <p class="mt-2 text-sm">錯誤訊息: ${error.message}</p>
        </div>`;
      }
    }
    
    // G DOM 載入完成後執行
    document.addEventListener('DOMContentLoaded', initApp); // <--- 修正：呼叫改名後的 initApp

  </script>

</body>
</html>

