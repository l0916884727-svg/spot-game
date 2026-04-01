
    <h2>🎯 找碴挑戰賽</h2>
    <input type="text" id="username" placeholder="您的快樂玩帳號" required>
    <input type="tel" id="phone" placeholder="您的勁舞團暱稱" required>
    
    <p>請在下圖中圈出不合理的地方：</p>
    <div id="canvas-container">
        <canvas id="gameCanvas"></canvas>
    </div>
    
    <button onclick="submitGame()">提交結果</button>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();

    // 這裡替換成你的找碴圖連結
    img.src = 'https://via.placeholder.com/500x300.png?text=Spot+The+Difference'; 
    
    img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
    };

    // 繪圖功能
    let painting = false;
    function startPosition(e) { painting = true; draw(e); }
    function finishedPosition() { painting = false; ctx.beginPath(); }
    function draw(e) {
        if (!painting) return;
        ctx.lineWidth = 5;
        ctx.lineCap = 'round';
        ctx.strokeStyle = 'red';

        const rect = canvas.getBoundingClientRect();
        const x = (e.clientX || e.touches[0].clientX) - rect.left;
        const y = (e.clientY || e.touches[0].clientY) - rect.top;

        ctx.lineTo(x * (canvas.width / rect.width), y * (canvas.height / rect.height));
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(x * (canvas.width / rect.width), y * (canvas.height / rect.height));
    }

    canvas.addEventListener('mousedown', startPosition);
    canvas.addEventListener('mouseup', finishedPosition);
    canvas.addEventListener('mousemove', draw);
    canvas.addEventListener('touchstart', startPosition);
    canvas.addEventListener('touchend', finishedPosition);
    canvas.addEventListener('touchmove', draw);

    function submitGame() {
        const name = document.getElementById('username').value;
        const finalImage = canvas.toDataURL("image/png");
        
        if(!name) { alert("請填寫姓名"); return; }
        
        console.log("提交人:", name);
        console.log("圖片Base64數據:", finalImage);
        alert("提交成功！(開發者提示：圖片數據已產生，可串接 API 傳存至後台)");
    }
</script>
</body>
</html>
