<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>品質管理業務支援アプリ</title>
    <style>
        /* 基本スタイル (変更なし) */
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #3498db;
            --background-color: #ecf0f1;
            --card-bg-color: #ffffff;
            --text-color: #333;
            --border-color: #bdc3c7;
            --shadow-color: rgba(0,0,0,0.1);
        }
        body {
            font-family: 'Helvetica Neue', 'Arial', 'Hiragino Kaku Gothic ProN', 'Hiragino Sans', 'Meiryo', sans-serif;
            background-color: var(--background-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        #app {
            width: 100%;
            max-width: 900px;
        }

        /* ヘッダー */
        .header {
            background-color: var(--primary-color);
            color: white;
            padding: 15px 25px;
            border-radius: 8px;
            margin-bottom: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .header h1 {
            margin: 0;
            font-size: 1.8em;
        }
        .back-button {
            background-color: var(--secondary-color);
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.2s;
        }
        .back-button:hover { background-color: #2980b9; }

        /* カードスタイル */
        .card {
            background-color: var(--card-bg-color);
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 4px 8px var(--shadow-color);
            margin-bottom: 20px;
        }

        /* リストスタイル */
        .item-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }
        .list-item {
            background-color: #f9f9f9;
            border: 1px solid var(--border-color);
            padding: 15px;
            margin-bottom: 10px;
            border-radius: 5px;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .list-item:hover {
            transform: translateY(-2px);
            box-shadow: 0 2px 5px var(--shadow-color);
        }
        .item-name { font-size: 1.2em; font-weight: bold; }
        .item-detail { color: #555; font-size: 0.9em; }

        /* ユーザー情報 */
        .user-info p { margin: 5px 0; }
        .user-info span { font-weight: bold; margin-left: 10px; }

        /* タスク画面 */
        .task-timer {
            display: flex;
            align-items: center;
        }
        .timer-button {
            margin-left: 15px;
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
            border: 1px solid var(--secondary-color);
            color: var(--secondary-color);
            background: white;
            min-width: 60px;
            text-align: center;
        }
        .timer-button.active {
            background: var(--secondary-color);
            color: white;
        }

        /* 細タスク画面 */
        .subtask-container {
            display: flex;
            flex-direction: column;
            gap: 20px;
        }
        .tabs {
            display: flex;
            border-bottom: 2px solid var(--border-color);
        }
        .tab {
            padding: 10px 20px;
            cursor: pointer;
            border: none;
            background-color: transparent;
            font-size: 1.1em;
            border-bottom: 3px solid transparent;
            margin-bottom: -2px;
        }
        .tab.active {
            font-weight: bold;
            color: var(--secondary-color);
            border-bottom-color: var(--secondary-color);
        }
        .io-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
        }
        @media (max-width: 768px) {
            .io-container { grid-template-columns: 1fr; }
        }
        textarea {
            width: 100%;
            height: 300px;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 5px;
            font-size: 1em;
            resize: vertical;
        }
        .execute-button {
            padding: 12px 20px;
            font-size: 1.1em;
            background-color: var(--secondary-color);
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            width: fit-content;
            align-self: center;
            transition: background-color 0.2s;
        }
        .execute-button:hover { background-color: #2980b9; }
        #output-area { background-color: #f0f0f0; }
    </style>
</head>
<body>

<!-- アプリケーションが描画されるターゲット要素 -->
<div id="app"></div>

<script>
    // === アプリケーションの状態管理 ===
    const state = {
        currentScreen: 'home', // 'home', 'files', 'tasks', 'subtask'
        history: [],
        user: {
            name: '山本崇翔',
            id: '750914',
            subject: '国語',
            rank: 'Q2B, LG+'
        },
        tests: [
            { id: 't001', name: '第1回東大本番レベル模試 国語4' },
            { id: 't002', name: '第1回京大本番レベル模試 国語4' },
            { id: 't003', name: '第2回共通テスト本番レベル模試 国語' }
        ],
        fileData: {
            t001: [
                { id: 'f01', name: '解答用紙', time: 0 },
                { id: 'f02', name: '解説冊子', time: 0 },
                { id: 'f03', name: '採点基準', time: 0 },
                { id: 'f04', name: '問題用紙', time: 0 }
            ],
            t002: [ { id: 'f01', name: '解答用紙', time: 0 }, { id: 'f02', name: '解説冊子', time: 0 } ],
            t003: [ { id: 'f01', name: '問題用紙', time: 0 } ]
        },
        tasks: [
            { id: 'task01', name: '解答解説を把握する' },
            { id: 'task02', name: '「▶全訳」と本文が正しく対応しているか' },
            { id: 'task03', name: '引用が誤っていないか' },
            { id: 'task04', name: '「▶解答」と「▶解説」に齟齬はないか' },
            { id: 'task05', name: '選択肢の番号・訳が正しいか' },
            { id: 'task06', name: '解答となる選択肢が過不足なく存在するか' },
            { id: 'task07', name: 'かっこの使用法に不備はないか' },
            { id: 'task08', name: '「▶重要語句・構文」に不備はないか' },
            { id: 'task09', name: '明確な事実誤認はないか' },
            { id: 'task10', name: '誤字脱字等の誤植・表現不備・スペルミスはないか' },
        ],
        selectedTest: null,
        selectedFile: null,
        selectedTask: null,
        // タスク画面用の状態
        taskTimers: {
            times: {}, // { taskId: 120 }
            active: { id: null, startTime: null, interval: null }
        },
        // 細タスク画面用の状態
        subTaskState: {
            daimonList: ['大問1', '大問2', '大問3', '大問4'],
            activeTab: '大問1',
            inputs: {},
            outputs: {},
            isLoading: false
        }
    };

    const appContainer = document.getElementById('app');

    // === ユーティリティ関数 ===
    function formatTime(seconds) {
        const h = Math.floor(seconds / 3600);
        const m = Math.floor((seconds % 3600) / 60);
        const s = Math.round(seconds % 60);
        return `${h > 0 ? h + '時間' : ''}${m > 0 ? m + '分' : ''}${s}秒`;
    }
    function formatTaskTime(seconds = 0) {
        const m = Math.floor(seconds / 60);
        const s = Math.round(seconds % 60);
        return `${m}分 ${s}秒`;
    }

    // === 画面描画関数 ===
    function render() {
        appContainer.innerHTML = ''; // 画面をクリア

        switch (state.currentScreen) {
            case 'home':
                appContainer.innerHTML = `
                    <div class="header"><h1>ホーム</h1></div>
                    <div class="card user-info">
                        <h2>ユーザー情報</h2>
                        <p>氏名:<span>${state.user.name}</span></p>
                        <p>ユーザーID:<span>${state.user.id}</span></p>
                        <p>担当科目:<span>${state.user.subject}</span></p>
                        <p>ランク:<span>${state.user.rank}</span></p>
                    </div>
                    <div class="card">
                        <h2>担当試験一覧</h2>
                        <ul class="item-list">
                            ${state.tests.map(test => `
                                <li class="list-item" data-testid="${test.id}">
                                    <span class="item-name">${test.name}</span>
                                    <span>▶</span>
                                </li>
                            `).join('')}
                        </ul>
                    </div>
                `;
                break;
            
            case 'files':
                const totalTime = (state.fileData[state.selectedTest.id] || []).reduce((acc, file) => acc + file.time, 0);
                appContainer.innerHTML = `
                    <div class="header">
                        <h1>${state.selectedTest.name}</h1>
                        <button class="back-button">◀ 戻る</button>
                    </div>
                    <div class="card">
                        <h2>ファイル一覧</h2>
                        <p><strong>総作業時間: ${formatTime(totalTime)}</strong></p>
                        <ul class="item-list">
                            ${(state.fileData[state.selectedTest.id] || []).map(file => `
                                <li class="list-item" data-fileid="${file.id}">
                                    <div>
                                        <div class="item-name">${file.name}</div>
                                        <div class="item-detail">作業時間: ${formatTime(file.time)}</div>
                                    </div>
                                    <span>▶</span>
                                </li>
                            `).join('')}
                        </ul>
                    </div>
                `;
                break;

            case 'tasks':
                 appContainer.innerHTML = `
                    <div class="header">
                        <h1>タスク一覧: ${state.selectedFile.name}</h1>
                        <button class="back-button" id="back-to-files">◀ ファイル一覧へ</button>
                    </div>
                    <div class="card">
                        <ul class="item-list">
                            ${state.tasks.map(task => `
                                <li class="list-item">
                                    <div class="item-name" data-taskid="${task.id}" style="flex-grow: 1;">
                                        ${task.name}
                                    </div>
                                    <div class="task-timer">
                                        <span class="item-detail" id="timer-display-${task.id}">${formatTaskTime(state.taskTimers.times[task.id])}</span>
                                        <button class="timer-button" data-timer-taskid="${task.id}">
                                            開始
                                        </button>
                                    </div>
                                </li>
                            `).join('')}
                        </ul>
                    </div>
                `;
                break;

            case 'subtask':
                appContainer.innerHTML = `
                    <div class="header">
                        <h1>細タスク: ${state.selectedTask.name}</h1>
                        <button class="back-button">◀ タスク一覧へ</button>
                    </div>
                    <div class="card subtask-container">
                        <div class="tabs">
                            ${state.subTaskState.daimonList.map(daimon => `
                                <button class="tab ${state.subTaskState.activeTab === daimon ? 'active' : ''}" data-daimon="${daimon}">
                                    ${daimon}
                                </button>
                            `).join('')}
                        </div>
                        <div class="io-container">
                            <div>
                                <h3>入力: 本文テキスト</h3>
                                <textarea id="input-area" placeholder="ここにチェックしたい本文を貼り付けてください。">${state.subTaskState.inputs[state.subTaskState.activeTab] || ''}</textarea>
                            </div>
                            <div>
                                <h3>出力: AIによる指摘</h3>
                                <textarea id="output-area" readonly>${state.subTaskState.outputs[state.subTaskState.activeTab] || ''}</textarea>
                            </div>
                        </div>
                        <button class="execute-button" ${state.subTaskState.isLoading ? 'disabled' : ''}>
                            ${state.subTaskState.isLoading ? '処理中...' : 'AIでチェックを実行'}
                        </button>
                    </div>
                `;
                break;
        }
    }

    // === イベントハンドリング ===
    appContainer.addEventListener('click', (e) => {
        // --- ホーム画面のイベント ---
        if (e.target.closest('.list-item[data-testid]')) {
            const testId = e.target.closest('.list-item').dataset.testid;
            const test = state.tests.find(t => t.id === testId);
            navigate('files', { selectedTest: test });
        }
        // --- ファイル画面のイベント ---
        else if (e.target.closest('.list-item[data-fileid]')) {
            const fileId = e.target.closest('.list-item').dataset.fileid;
            const file = state.fileData[state.selectedTest.id].find(f => f.id === fileId);
            navigate('tasks', { selectedFile: file });
        }
        // --- タスク画面のイベント ---
        else if (e.target.closest('.item-name[data-taskid]')) {
            const taskId = e.target.closest('.item-name').dataset.taskid;
            const task = state.tasks.find(t => t.id === taskId);
            navigate('subtask', { selectedTask: task });
        }
        else if (e.target.closest('.timer-button[data-timer-taskid]')) {
            const taskId = e.target.dataset.timerTaskid;
            toggleTimer(taskId);
        }
        else if (e.target.id === 'back-to-files') {
            handleBackFromTask();
        }
        // --- 細タスク画面のイベント ---
        else if (e.target.closest('.tab[data-daimon]')) {
            state.subTaskState.activeTab = e.target.dataset.daimon;
            render();
        }
        else if (e.target.closest('.execute-button')) {
            executeAI();
        }
        // --- 共通の戻るボタン ---
        else if (e.target.closest('.back-button')) {
            goBack();
        }
    });

    // 細タスク画面のテキストエリア入力イベント
    appContainer.addEventListener('input', (e) => {
        if (e.target.id === 'input-area') {
            state.subTaskState.inputs[state.subTaskState.activeTab] = e.target.value;
        }
    });


    // === ロジック関数 ===
    function navigate(screen, data) {
        state.history.push({
            screen: state.currentScreen,
            selectedTest: state.selectedTest,
            selectedFile: state.selectedFile,
            selectedTask: state.selectedTask
        });
        state.currentScreen = screen;
        if (data.selectedTest) state.selectedTest = data.selectedTest;
        if (data.selectedFile) {
            state.selectedFile = data.selectedFile;
            // タイマーリセット
            state.taskTimers = { times: {}, active: { id: null, startTime: null, interval: null } };
        }
        if (data.selectedTask) {
            state.selectedTask = data.selectedTask;
            // 細タスク状態リセット
            state.subTaskState.activeTab = state.subTaskState.daimonList[0];
            state.subTaskState.inputs = {};
            state.subTaskState.outputs = {};
        }
        render();
    }

    function goBack() {
        if (state.history.length === 0) return;
        const last = state.history.pop();
        state.currentScreen = last.screen;
        state.selectedTest = last.selectedTest;
        state.selectedFile = last.selectedFile;
        state.selectedTask = last.selectedTask;
        render();
    }
    
    function toggleTimer(taskId) {
        const { active } = state.taskTimers;
        const button = document.querySelector(`[data-timer-taskid="${taskId}"]`);

        // 他のタイマーが動いていれば停止
        if (active.id && active.id !== taskId) {
            clearInterval(active.interval);
            const elapsed = (Date.now() - active.startTime) / 1000;
            state.taskTimers.times[active.id] = (state.taskTimers.times[active.id] || 0) + elapsed;
            document.querySelector(`[data-timer-taskid="${active.id}"]`).textContent = '開始';
            document.querySelector(`[data-timer-taskid="${active.id}"]`).classList.remove('active');
        }

        // クリックされたタイマーを操作
        if (active.id === taskId) { // 停止
            clearInterval(active.interval);
            const elapsed = (Date.now() - active.startTime) / 1000;
            state.taskTimers.times[taskId] = (state.taskTimers.times[taskId] || 0) + elapsed;
            active.id = null;
            button.textContent = '開始';
            button.classList.remove('active');
        } else { // 開始
            active.id = taskId;
            active.startTime = Date.now();
            active.interval = setInterval(() => {
                const elapsed = (Date.now() - active.startTime) / 1000;
                const total = (state.taskTimers.times[taskId] || 0) + elapsed;
                document.getElementById(`timer-display-${taskId}`).textContent = formatTaskTime(total);
            }, 1000);
            button.textContent = '停止';
            button.classList.add('active');
        }
    }

    function handleBackFromTask() {
        const { active, times } = state.taskTimers;
        if (active.id) { // 実行中のタイマーがあれば止める
            clearInterval(active.interval);
            const elapsed = (Date.now() - active.startTime) / 1000;
            times[active.id] = (times[active.id] || 0) + elapsed;
        }
        const totalFileTime = Object.values(times).reduce((sum, t) => sum + t, 0);
        const file = state.fileData[state.selectedTest.id].find(f => f.id === state.selectedFile.id);
        if (file) file.time = totalFileTime;
        
        goBack();
    }
    
    function executeAI() {
        state.subTaskState.isLoading = true;
        render(); // ボタンを無効化するために再描画

        setTimeout(() => {
            const { activeTab, inputs } = state.subTaskState;
            const inputText = inputs[activeTab] || '';
            if (inputText.trim() === '') {
                state.subTaskState.outputs[activeTab] = "テキストが入力されていません。";
            } else {
                state.subTaskState.outputs[activeTab] = `【AIからの指摘（${activeTab}）】\n\n` +
                    `1. (p.12 l.3) 引用元「出典X」の表記が、規定フォーマットと一致しません。正しくは「『出典X』より引用」とすべきです。\n` +
                    `2. (p.15 l.8) カギ括弧内の句読点の使用法に誤りがあります。文末の句点は不要です。\n` +
                    `3. (p.18 l.1) 引用箇所が原文と一部異なっています。\n   原文:「...重要な概念である。」\n   本文:「...重要な概念だ。」\n\n` +
                    `以上の点について確認を推奨します。`;
            }
            state.subTaskState.isLoading = false;
            render(); // 結果を表示するために再描画
        }, 1500);
    }

    // === アプリケーションの初期化 ===
    document.addEventListener('DOMContentLoaded', render);

</script>
</body>
</html>
