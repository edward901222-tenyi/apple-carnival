<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AB猜數字遊戲 (幾A幾B)</title>
    <style>
        :root {
            --primary-color: #4a90e2;
            --success-color: #2ec4b6;
            --danger-color: #e71d36;
            --bg-color: #f4f7f6;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }

        .game-container {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 450px;
            text-align: center;
        }

        h1 {
            color: #333;
            margin-bottom: 10px;
        }

        p.rule {
            color: #666;
            font-size: 0.9rem;
            margin-bottom: 25px;
        }

        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        input {
            flex: 1;
            padding: 12px;
            font-size: 1.2rem;
            border: 2px solid #ddd;
            border-radius: 8px;
            text-align: center;
            letter-spacing: 5px;
        }

        input:focus {
            outline: none;
            border-color: var(--primary-color);
        }

        button {
            padding: 12px 24px;
            font-size: 1rem;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.2s;
            font-weight: bold;
        }

        button:hover {
            background-color: #357abd;
        }

        #restart-btn {
            background-color: var(--success-color);
            display: none;
            width: 100%;
            margin-top: 10px;
        }

        #restart-btn:hover {
            background-color: #249a8f;
        }

        .message {
            margin: 15px 0;
            font-weight: bold;
            min-height: 24px;
        }

        .error { color: var(--danger-color); }
        .success { color: var(--success-color); }

        .history-container {
            margin-top: 25px;
            border-top: 2px solid #eee;
            padding-top: 20px;
            max-height: 250px;
            overflow-y: auto;
            text-align: left;
        }

        .history-title {
            font-weight: bold;
            color: #444;
            margin-bottom: 10px;
        }

        .history-item {
            display: flex;
            justify-content: space-between;
            padding: 8px 12px;
            border-bottom: 1px solid #f0f0f0;
            font-family: monospace;
            font-size: 1.1rem;
        }

        .result-tag {
            font-weight: bold;
            padding: 2px 8px;
            border-radius: 4px;
        }
        
        .tag-a { color: var(--danger-color); }
        .tag-b { color: var(--primary-color); }
    </style>
</head>
<body>

<div class="game-container">
    <h1>猜數字遊戲</h1>
    <p class="rule">請輸入 4 位<strong>不重複</strong>的數字 (0-9)</p>

    <div class="input-group">
        <input type="text" id="user-guess" maxlength="4" placeholder="0123" autocomplete="off">
        <button id="guess-btn" onclick="checkGuess()">猜！</button>
    </div>

    <div id="msg" class="message"></div>
    <button id="restart-btn" onclick="initGame()">重新開始</button>

    <div class="history-container">
        <div class="history-title">對戰紀錄：</div>
        <div id="history-list"></div>
    </div>
</div>

<script>
    let answer = [];
    let attempts = 0;
    let isGameOver = false;

    const guessInput = document.getElementById('user-guess');
    const guessBtn = document.getElementById('guess-btn');
    const restartBtn = document.getElementById('restart-btn');
    const msgDiv = document.getElementById('msg');
    const historyList = document.getElementById('history-list');

    // 監聽 Enter 鍵方便遊玩
    guessInput.addEventListener('keypress', function(e) {
        if (e.key === 'Enter') {
            checkGuess();
        }
    });

    // 初始化/重置遊戲
    function initGame() {
        answer = [];
        attempts = 0;
        isGameOver = false;
        
        // 生成 4 個不重複的隨機數
        const numbers = Array.from({length: 10}, (_, i) => i);
        for (let i = 0; i < 4; i++) {
            const randomIndex = Math.floor(Math.random() * numbers.length);
            answer.push(numbers[randomIndex]);
            numbers.splice(randomIndex, 1);
        }
        
        // 偵錯用（如果想作弊可以在瀏覽器 Console 看答案）
        console.log("答案是: " + answer.join(''));

        // 重置畫面
        guessInput.value = '';
        guessInput.disabled = false;
        guessBtn.disabled = false;
        restartBtn.style.display = 'none';
        msgDiv.textContent = '遊戲開始！請輸入數字。';
        msgDiv.className = 'message';
        historyList.innerHTML = '';
    }

    // 檢查玩家輸入
    function checkGuess() {
        if (isGameOver) return;

        const guessStr = guessInput.value.trim();
        
        // 防呆驗證：必須是 4 位數、純數字、不重複
        if (guessStr.length !== 4 || isNaN(guessStr)) {
            showResult('請輸入完整的 4 位數字！', 'error');
            return;
        }

        const guessArray = guessStr.split('').map(Number);
        const uniqueSet = new Set(guessArray);
        if (uniqueSet.size !== 4) {
            showResult('數字不能重複！', 'error');
            return;
        }

        attempts++;
        let aCount = 0;
        let bCount = 0;

        // 比對 AB 邏輯
        for (let i = 0; i < 4; i++) {
            if (guessArray[i] === answer[i]) {
                aCount++; // 數字對、位置也對
            } else if (answer.includes(guessArray[i])) {
                bCount++; // 有這個數字，但位置錯了
            }
        }

        // 顯示本次結果與紀錄
        appendHistory(attempts, guessStr, aCount, bCount);
        guessInput.value = ''; // 清空輸入框
        guessInput.focus();

        if (aCount === 4) {
            showResult(`🎉 恭喜答對！答案就是 ${guessStr}。總共猜了 ${attempts} 次。`, 'success');
            endGame();
        } else {
            showResult(`第 ${attempts} 次嘗試：${aCount}A ${bCount}B`, '');
        }
    }

    function showResult(text, className) {
        msgDiv.textContent = text;
        msgDiv.className = 'message ' + className;
    }

    function appendHistory(round, guess, a, b) {
        const item = document.createElement('div');
        item.className = 'history-item';
        item.innerHTML = `
            <span>第 ${round} 次： <strong>${guess}</strong></span>
            <span>
                <span class="result-tag tag-a">${a}A</span> 
                <span class="result-tag tag-b">${b}B</span>
            </span>
        `;
        // 新紀錄放在最上面
        historyList.insertBefore(item, historyList.firstChild);
    }

    function endGame() {
        isGameOver = true;
        guessInput.disabled = true;
        guessBtn.disabled = true;
        restartBtn.style.display = 'block';
    }

    // 網頁載入時自動執行
    window.onload = initGame;
</script>

</body>
</html>
