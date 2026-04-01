<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>勁舞團 9 週年 - 血量挑戰賽</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 40px 10px; background: #2c3e50; margin: 0; color: white; }
        .container { max-width: 500px; margin: auto; background: rgba(255, 255, 255, 0.95); padding: 25px; border-radius: 20px; box-shadow: 0 10px 25px rgba(0,0,0,0.3); color: #333; }
        input { width: 90%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; font-size: 16px; outline: none; }
        input:focus { border-color: #ff4757; }
        #canvas-container { position: relative; margin-top: 15px; border-radius: 10px; overflow: hidden; background: #eee; line-height: 0; cursor: crosshair; }
        canvas { width: 100%; height: auto; touch-action: none; }
        
        /* 血量和計數器樣式 */
        #game-stats { display: none; justify-content: space-around; background: #eee; padding: 10px; border-radius: 8px; margin-bottom: 10px; font-weight: bold; }
        .heart { color: #ff4757; font-size: 20px; }
        
        button { background: #ff4757; color: white; border: none; padding: 15px 40px; border-radius: 30px; cursor: pointer; margin-top: 20px; font-size: 18px; font-weight: bold; transition: 0.3s; }
        button:hover { background: #e84118; transform: scale(1.05); }
        button:disabled { background: #ccc; cursor: not-allowed; }
        p.desc { color: #666; font-size: 14px; line-height: 1.6; }
        #login-section { display: block; }
        #game-section { display: none; }
    </style>
</head>
<body>

<div class="container">
    <h2>💃 勁舞團 9 週年找碴賽 🕺</h2>
    
    <div id="login-section">
        <p class="desc">請填寫基本資料開始血量制挑戰，<br>找齊所有隱藏哈特貓（ 12 隻）即可獲得抽獎資格！</p>
        <input type="text" id="username" placeholder="您的勁舞團(快樂玩)帳號" required>
        <input type="text" id="nickname" placeholder="您的勁舞團暱稱" required>
        <button onclick="startGame()">開始挑戰</button>
    </div>

    <div id="game-section">
        <div id="game-stats">
            <div>血量：<span id="health">❤️❤️❤️</span></div>
            <div>已找到：<span id="foundCount">0</span> / <span id="totalTarget">12</span> 隻</div>
        </div>
        
        <div id="canvas-container">
            <canvas id="gameCanvas"></canvas>
        </div>
        
        <p class="desc" style="color: #ff4757;">⚠️ 點錯地方會扣血唷！血量扣完就失敗了！</p>
        <button id="submitBtn" onclick="submitGame()" disabled>提交結果</button>
    </div>
</div>

<script>
    // --- 遊戲核心設定 (你需要修改這裡) ---
    // 目標數量
    const TOTAL_TARGETS = 12; 
    // 玩家初始血量
    let playerHealth = 3; 
    // 目標坐標區域 (相對於圖片的像素坐標，需要手動測量)
    // 格式: { x: X坐標, y: Y坐標, radius: 點擊有效的半徑(像素) }
    const targetAreas = [
        { x: 429, y: 283, radius: 25 }, // 目標 1 (例如：左邊喇叭旁)
        { x: 300, y: 223, radius: 15 }, // 目標 2
        { x: 259, y: 198, radius: 12 }, // 目標 3
        { x: 182, y: 200, radius: 15 }, // 目標 4
        { x: 48, y: 148, radius: 15 }, // 目標 5
        { x: 176, y: 154, radius: 14 }, // 目標 6
        { x: 166, y: 102, radius: 10 }, // 目標 7
        { x: 248, y: 100, radius: 10 }, // 目標 8
        { x: 355, y: 142, radius: 10 }, // 目標 9
        { x: 383, y: 101, radius: 10 }, // 目標 10
        { x: 425, y: 102, radius: 10 }, // 目標 11
        { x: 442, y: 141, radius: 15 }, // 目標 12
        // 如果你有更多目標，請繼續添加...
    ];
    
    // 你的 GAS 應用程式網址
    const scriptURL = '你的_GAS_網頁應用程式網址';
    // 你的圖片網址
    const imgUrl = 'game.jpg'; // 請確保檔名正確

    // --- 遊戲狀態變數 ---
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();
    let foundTargets = []; // 記錄已點擊的目標索引
    let isGameOver = false;

    // --- 初始化圖片 ---
    img.src = imgUrl;
    img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
        document.getElementById('totalTarget').innerText = TOTAL_TARGETS;
    };

    // --- 取得點擊位置並判斷 ---
    canvas.addEventListener('click', handleCanvasClick);
    canvas.addEventListener('touchstart', (e) => {
        e.preventDefault(); // 防止滾動
        const touch = e.touches[0];
        handleCanvasClick(touch);
    }, {passive: false});

    function handleCanvasClick(e) {
        if (isGameOver) return;

        const rect = canvas.getBoundingClientRect();
        const clientX = e.clientX || e.pageX;
        const clientY = e.clientY || e.pageY;

        // 計算相對於圖片坐標
        const scaleX = canvas.width / rect.width;
        const scaleY = canvas.height / rect.height;
        const clickX = (clientX - rect.left) * scaleX;
        const clickY = (clientY - rect.top) * scaleY;

        let hit = false;
        // 判斷是否擊中目標
        for (let i = 0; i < targetAreas.length; i++) {
            const target = targetAreas[i];
            const dist = Math.sqrt((clickX - target.x)**2 + (clickY - target.y)**2);
            
            if (dist <= target.radius) {
                if (!foundTargets.includes(i)) {
                    // 擊中新目標
                    foundTargets.push(i);
                    drawFoundMarker(target.x, target.y);
                    updateFoundCount();
                    hit = true;
                    // 如果需要，可以在這裡播放音效
                    break;
                } else {
                    // 點到已經找到的目標
                    alert("這個已經找過囉！");
                    return; 
                }
            }
        }

        if (!hit) {
            // 點錯了，扣血
            deductHealth();
            // 如果需要，可以畫一個叉叉標記點錯的位置
            drawMissMarker(clickX, clickY);
        }
    }

    // 在畫布上標記點錯的位置
    function drawMissMarker(x, y) {
        ctx.beginPath();
        ctx.arc(x, y, 10, 0, 2 * Math.PI);
        ctx.fillStyle = 'rgba(255, 71, 87, 0.5)'; // 紅色半透明圓
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = 'white';
        ctx.stroke();
        setTimeout(() => { // 短暫顯示後清除標記
            ctx.drawImage(img, 0, 0); // 重新繪製底圖
            foundTargets.forEach(idx => drawFoundMarker(targetAreas[idx].x, targetAreas[idx].y)); // 重新繪製找到的標記
        }, 500);
    }

    // --- 遊戲邏輯函數 ---
    function startGame() {
        const name = document.getElementById('username').value;
        if(!name) { alert("請輸入帳號！"); return; }
        
        document.getElementById('login-section').style.display = 'none';
        document.getElementById('game-section').style.display = 'block';
        document.getElementById('game-stats').style.display = 'flex';
    }

    function deductHealth() {
        playerHealth--;
        let heartStr = '';
        for(let i=0; i<3; i++) {
            heartStr += (i < playerHealth) ? '❤️' : '🖤';
        }
        document.getElementById('health').innerText = heartStr;

        if (playerHealth === 0) {
            isGameOver = true;
            alert("⚠️ 挑戰失敗！血量扣完囉。");
            document.getElementById('submitBtn').innerText = "挑戰失敗";
            document.getElementById('submitBtn').disabled = true;
        }
    }

    function updateFoundCount() {
        document.getElementById('foundCount').innerText = foundTargets.length;
        if (foundTargets.length === TOTAL_TARGETS) {
            isGameOver = true;
            alert("🎉 恭喜！你找齊所有哈特貓了。可以提交結果囉！");
            document.getElementById('submitBtn').disabled = false;
        }
    }

    function drawFoundMarker(x, y) {
        ctx.beginPath();
        ctx.arc(x, y, 20, 0, 2 * Math.PI);
        ctx.fillStyle = 'rgba(76, 175, 80, 0.5)'; // 綠色半透明圓
        ctx.fill();
        ctx.lineWidth = 3;
        ctx.strokeStyle = '#4CAF50';
        ctx.stroke();
    }

    // --- 提交資料 ---
    function submitGame() {
        if (foundTargets.length < TOTAL_TARGETS || isGameOver === false) { alert("還沒找齊不能提交唷！"); return; }
        
        const name = document.getElementById('username').value;
        const nickname = document.getElementById('nickname').value;
        const submitBtn = document.getElementById('submitBtn');
        
        submitBtn.disabled = true; // 防止重複提交
        submitBtn.innerText = "提交中...";

        const data = {
            name: name,
            phone: nickname, // 拿暱稱存入電話欄位
            answer: '血量制挑戰成功' // 傳送狀態
        };

        fetch(scriptURL, {
            method: 'POST',
            mode: 'no-cors',
            cache: 'no-cache',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        })
        .then(() => {
            alert("提交成功！\n感謝 " + nickname + " 參與活動，資料已記錄。");
            submitBtn.innerText = "提交成功";
        })
        .catch(error => {
            console.error('Error!', error.message);
            alert("提交失敗，請檢查網路連線。");
            submitBtn.disabled = false;
            submitBtn.innerText = "重新提交";
        });
    }
</script>

</body>
</html>
