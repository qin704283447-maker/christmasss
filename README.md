<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>圣诞快乐 - 温暖冬日</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: radial-gradient(circle at center, #1B2735 0%, #090A0F 100%);
            height: 100vh;
            font-family: 'Microsoft YaHei', sans-serif;
            cursor: pointer;
        }

        /* 顶部提示 */
        .hint {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            color: rgba(255,255,255,0.4);
            z-index: 20;
            font-size: 14px;
            pointer-events: none;
        }

        /* 自动滚动祝福语 */
        .wishes-container {
            position: absolute;
            width: 100%;
            bottom: -100px;
            text-align: center;
            z-index: 8;
            pointer-events: none;
            animation: scrollUp 25s linear infinite;
        }

        @keyframes scrollUp {
            0% { transform: translateY(0); opacity: 0; }
            5% { opacity: 1; }
            90% { opacity: 1; }
            100% { transform: translateY(-90vh); opacity: 0; }
        }

        .wishes-text {
            color: #fff;
            font-size: 1.5rem;
            line-height: 2.5;
            text-shadow: 0 0 10px rgba(255,255,255,0.8);
            white-space: pre-line;
        }

        .title-layer {
            position: absolute;
            width: 100%;
            text-align: center;
            z-index: 10;
            top: 10%;
            pointer-events: none;
        }

        h1 {
            color: #fff;
            font-size: 4rem;
            text-shadow: 0 0 20px #ff4d4d, 0 0 30px #ffffff;
            margin: 0;
        }

        .decorations {
            position: absolute;
            bottom: 30px;
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: flex-end;
            z-index: 5;
        }

        .tree { 
            font-size: 320px; 
            filter: drop-shadow(0 0 30px rgba(0,255,0,0.5)); 
            cursor: pointer;
            transition: transform 0.1s;
            user-select: none;
        }
        .tree:active { transform: scale(0.95); }

        .santa { 
            font-size: 80px; 
            margin-left: -50px;
            animation: bounce 2s infinite;
        }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-15px); }
        }

        .gift-box {
            position: absolute;
            font-size: 35px;
            z-index: 6;
            pointer-events: none;
        }

        canvas { position: absolute; top: 0; left: 0; z-index: 2; }
    </style>
</head>
<body>

    <div class="hint">点击屏幕任意处：开启音乐 | 点击树：掉落礼物</div>

    <div class="title-layer">
        <h1>Merry Christmas</h1>
    </div>

    <div class="wishes-container">
        <div class="wishes-text">
            在这个温暖的夜晚
            愿每一片飘落的雪花
            都带给你一份纯净的快乐
            ❄️
            愿圣诞老人的铃声
            敲响你心底的幸福
            🍎
            愿所有的愿望都能成真
            愿你的世界永远充满光亮
            ✨
            圣诞快乐，不止今天
            而是岁岁年年
            🎄
        </div>
    </div>

    <div class="decorations">
        <div class="tree" id="xmasTree" onclick="dropGift()">🎄</div>
        <div class="santa">🎅</div>
    </div>

    <canvas id="snowCanvas"></canvas>

    <audio id="bgMusic" loop>
        <source src="https://www.mfiles.co.uk/mp3-downloads/jingle-bells-piano.mp3" type="audio/mpeg">
    </audio>

    <script>
        const canvas = document.getElementById('snowCanvas');
        const ctx = canvas.getContext('2d');
        const music = document.getElementById('bgMusic');
        
        let width, height;
        let flakes = [];
        let groundSnow = [];

        // 点击激活音乐
        document.body.addEventListener('click', () => {
            music.play().catch(() => {});
        }, { once: false });

        function init() {
            width = window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
            groundSnow = new Array(Math.ceil(width)).fill(0);
            flakes = [];
        }

        class Flake {
            constructor() { this.reset(); this.y = Math.random() * height; }
            reset() {
                this.x = Math.random() * width;
                this.y = -10;
                this.size = Math.random() * 3 + 2;
                this.speed = Math.random() * 1 + 0.8;
                this.velX = Math.random() * 1 - 0.5;
                this.opacity = Math.random() * 0.5 + 0.5;
            }
            update() {
                this.y += this.speed;
                this.x += this.velX;
                let idx = Math.floor(this.x);
                if (idx >= 0 && idx < groundSnow.length) {
                    if (this.y >= height - groundSnow[idx]) {
                        for(let i=-6; i<=6; i++){
                            if(idx+i >=0 && idx+i < groundSnow.length) {
                                groundSnow[idx+i] += 0.2; // 积雪增加
                            }
                        }
                        this.reset();
                    }
                }
                if (this.y > height) this.reset();
            }
            draw() {
                ctx.fillStyle = `rgba(255, 255, 255, ${this.opacity})`;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        function dropGift() {
            const gifts = ['🎁', '🍎', '🍬', '⭐', '🧸'];
            const gift = document.createElement('div');
            gift.className = 'gift-box';
            gift.innerHTML = gifts[Math.floor(Math.random() * gifts.length)];
            
            const treeRect = document.getElementById('xmasTree').getBoundingClientRect();
            let giftX = (treeRect.left + 50 + Math.random() * 200);
            gift.style.left = giftX + 'px';
            gift.style.top = (treeRect.top + 100) + 'px';
            document.body.appendChild(gift);

            let pos = treeRect.top + 100;
            const fall = setInterval(() => {
                pos += 10;
                let currentGround = height - groundSnow[Math.floor(giftX)] - 35;
                gift.style.top = pos + 'px';
                if (pos >= currentGround) {
                    clearInterval(fall);
                    gift.style.transform = `rotate(${Math.random()*30-15}deg)`;
                }
            }, 20);
        }

        function animate() {
            ctx.clearRect(0, 0, width, height);
            if (flakes.length < 180) flakes.push(new Flake());
            flakes.forEach(f => { f.update(); f.draw(); });

            // 绘制地面积雪
            ctx.fillStyle = "white";
            ctx.beginPath();
            ctx.moveTo(0, height);
            for (let i = 0; i < groundSnow.length; i++) {
                ctx.lineTo(i, height - groundSnow[i]);
            }
            ctx.lineTo(width, height);
            ctx.fill();

            requestAnimationFrame(animate);
        }

        window.addEventListener('resize', init);
        init();
        animate();
    </script>
</body>
</html>
