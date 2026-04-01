<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>找碴小遊戲</title>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 20px; background: #f4f4f9; }
        .container { max-width: 500px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }
        input { width: 90%; padding: 12px; margin: 10px 0; border: 1px solid #ddd; border-radius: 8px; font-size: 16px; }
        #canvas-container { position: relative; border: 2px solid #333; margin-top: 15px; border-radius: 5px; overflow: hidden; background: #eee; line-height: 0; }
        canvas { width: 100%; height: auto; touch-action: none; cursor: crosshair; }
        button { background: #ff4757; color: white; border: none; padding: 12px 30px; border-radius: 25px; cursor: pointer; margin-top: 20px; font-size: 18px; font-weight: bold; }
        p { color: #666; font-size: 14px; }
    </style>
</head>
<body>

<div class="container">
    <h2>🎯 找碴挑戰賽</h2>
    <p>請先填寫基本資料開始遊戲</p>
    <input type="text" id="username" placeholder="您的勁舞團(快樂玩)帳號" required>
    <input type="tel" id="phone" placeholder="您的進舞團暱稱" required>
    
    <div id="canvas-container">
        <canvas id="gameCanvas"></canvas>
    </div>
    <p>💡 用手指或滑鼠在圖中「圈出不同處」</p>
    
    <button onclick="submitGame()">提交結果</button>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();

    // 預設一張範例圖，你可以之後換成自己的 game.jpg
    img.src = 'game.jpg';
    
    img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
    };

    let painting = false;
    function getPos(e) {
        const rect = canvas.getBoundingClientRect();
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        return {
            x: (clientX - rect.left) * (canvas.width / rect.width),
            y: (clientY - rect.top) * (canvas.height / rect.height)
        };
    }

    function startPosition(e) { painting = true; draw(e); }
    function finishedPosition() { painting = false; ctx.beginPath(); }
    
    function draw(e) {
        if (!painting) return;
        e.preventDefault();
        const pos = getPos(e);
        ctx.lineWidth = 6;
        ctx.lineCap = 'round';
        ctx.strokeStyle = 'red';

        ctx.lineTo(pos.x, pos.y);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(pos.x, pos.y);
    }

    canvas.addEventListener('mousedown', startPosition);
    canvas.addEventListener('mouseup', finishedPosition);
    canvas.addEventListener('mousemove', draw);
    canvas.addEventListener('touchstart', startPosition, {passive: false});
    canvas.addEventListener('touchend', finishedPosition);
    canvas.addEventListener('touchmove', draw, {passive: false});

    function submitGame() {
        const name = document.getElementById('username').value;
        const phone = document.getElementById('phone').value;
        if(!name || !phone) { alert("請完整填寫資料唷！"); return; }
        
        // 這是帶有圈選痕跡的圖片數據 (Base64)
        const finalImage = canvas.toDataURL("image/png");
        
        console.log("提交人:", name, phone);
        alert("提交成功！\n感謝 " + name + " 參與活動。");
        // 後續可在此串接 API 傳送資料
    }
</script>
</body>
</html>
