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
        
        /* 血量、計時與計數器樣式 */
        #game-stats { display: none; justify-content: space-around; background: #eee; padding: 10px; border-radius: 8px; margin-bottom: 10px; font-weight: bold; font-size: 14px; }
        
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
    <h2>💃 找找哈特有幾隻 🕺</h2>
    
    <div id="login-section">
        <p class="desc">請填寫基本資料開始血量計時制挑戰，<br>找齊所有隱藏哈特貓（ 12 隻）即可獲得抽獎資格！</p>
        <input type="text" id="username" placeholder="您的勁舞團(快樂玩)帳號" required>
        <input type="text" id="nickname" placeholder="您的勁舞團暱稱" required>
        <button onclick="startGame()">開始挑戰</button>
    </div>

    <div id="game-section">
        <div id="game-stats">
            <div>血量：<span id="health">❤️❤️❤️</span></div>
            <div>計時：<span id="timer">0</span>s</div>
            <div>進度：<span id="foundCount">0</span>/12</div>
        </div>
        
        <div id="canvas-container">
            <canvas id="gameCanvas"></canvas>
        </div>
        
        <p class="desc" style="color: #ff4757; font-weight: bold; margin-top: 15px;">⚠️ 點錯地方會扣血唷！</p>
        <button id="submitBtn" onclick="submitGame()" disabled>提交結果</button>
    </div>
</div>

<script>
    // --- 遊戲核心設定 ---
    const TOTAL_TARGETS = 12; 
    let playerHealth = 3; 
    const targetAreas = [
        { x: 429, y: 283, radius: 40 }, { x: 300, y: 223, radius: 15 },
        { x: 259, y: 198, radius: 12 }, { x: 182, y: 200, radius: 15 },
        { x: 48, y: 148, radius: 15 }, { x: 176, y: 154, radius: 14 },
        { x: 166, y: 102, radius: 10 }, { x: 248, y: 100, radius: 10 },
        { x: 355, y: 142, radius: 10 }, { x: 383, y: 101, radius: 10 },
        { x: 425, y: 102, radius: 10 }, { x: 442, y: 141, radius: 15 }
    ];
    
    const scriptURL = 'https://script.google.com/macros/s/AKfycbwUFAeQJo8i8eBr8Jgq1d-UypxOZhqX9NWcXKL1iuQZINWpGB-6Tb1KJLRCuvZEUKd3/exec';
    const imgUrl = 'game.jpg'; 

    // --- 遊戲狀態變數 ---
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();
    let foundTargets = []; 
    let isGameOver = false;

    // --- 計時器變數 ---
    let startTime;
    let timerInterval;
    let finalSeconds = 0;

    img.src = imgUrl;
    img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
    };

    function startTimer() {
        startTime = Date.now();
        timerInterval = setInterval(() => {
            const now = Date.now();
            finalSeconds = Math.floor((now - startTime) / 1000);
            document.getElementById('timer').innerText = finalSeconds;
        }, 1000);
    }

    function stopTimer() {
        clearInterval(timerInterval);
    }

    canvas.addEventListener('click', handleCanvasClick);
    canvas.addEventListener('touchstart', (e) => {
        e.preventDefault();
        handleCanvasClick(e.touches[0]);
    }, {passive: false});

    function handleCanvasClick(e) {
        if (isGameOver) return;
        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / rect.width;
        const scaleY = canvas.height / rect.height;
        const clickX = (e.clientX - rect.left) * scaleX;
        const clickY = (e.clientY - rect.top) * scaleY;

        let hit = false;
        for (let i = 0; i < targetAreas.length; i++) {
            const target = targetAreas[i];
            const dist = Math.sqrt((clickX - target.x)**2 + (clickY - target.y)**2);
            if (dist <= target.radius) {
                if (!foundTargets.includes(i)) {
                    foundTargets.push(i);
                    drawFoundMarker(target.x, target.y);
                    updateFoundCount();
                    hit = true;
                    break;
                } else {
                    alert("這個已經找過囉！");
                    return; 
                }
            }
        }
        if (!hit) {
            deductHealth();
            drawMissMarker(clickX, clickY);
        }
    }

    function drawMissMarker(x, y) {
        ctx.beginPath();
        ctx.arc(x, y, 10, 0, 2 * Math.PI);
        ctx.fillStyle = 'rgba(255, 71, 87, 0.7)';
        ctx.fill();
        setTimeout(() => {
            ctx.drawImage(img, 0, 0);
            foundTargets.forEach(idx => drawFoundMarker(targetAreas[idx].x, targetAreas[idx].y));
        }, 400);
    }

    function startGame() {
        const name = document.getElementById('username').value;
        const nick = document.getElementById('nickname').value;
        if(!name || !nick) { alert("請填寫帳號與暱稱唷！"); return; }
        document.getElementById('login-section').style.display = 'none';
        document.getElementById('game-section').style.display = 'block';
        document.getElementById('game-stats').style.display = 'flex';
        startTimer();
    }

    function deductHealth() {
        playerHealth--;
        let heartStr = '❤️'.repeat(playerHealth) + '🖤'.repeat(3 - playerHealth);
        document.getElementById('health').innerText = heartStr;
        if (playerHealth === 0) {
            isGameOver = true;
            stopTimer();
            alert("⚠️ 挑戰失敗！血量扣完囉。");
            document.getElementById('submitBtn').innerText = "挑戰失敗";
        }
    }

    function updateFoundCount() {
        document.getElementById('foundCount').innerText = foundTargets.length;
        if (foundTargets.length === TOTAL_TARGETS) {
            isGameOver = true;
            stopTimer();
            alert("🎉 恭喜找齊！總共花了 " + finalSeconds + " 秒。");
            document.getElementById('submitBtn').disabled = false;
        }
    }

    function drawFoundMarker(x, y) {
        ctx.beginPath();
        ctx.arc(x, y, 20, 0, 2 * Math.PI);
        ctx.strokeStyle = '#4CAF50';
        ctx.lineWidth = 3;
        ctx.stroke();
    }

    function submitGame() {
        const name = document.getElementById('username').value;
        const nickname = document.getElementById('nickname').value;
        const submitBtn = document.getElementById('submitBtn');
        submitBtn.disabled = true;
        submitBtn.innerText = "提交中...";

        const data = {
            name: name,
            phone: nickname,
            answer: "挑戰成功 (耗時: " + finalSeconds + " 秒)"
        };

        fetch(scriptURL, {
            method: 'POST',
            mode: 'no-cors',
            cache: 'no-cache',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        })
        .then(() => {
            alert("提交成功！\n" + nickname + " 辛苦了！");
            submitBtn.innerText = "已提交";
        })
        .catch(error => {
            alert("提交失敗，請檢查網路。");
            submitBtn.disabled = false;
        });
    }
</script>

</body>
</html>
