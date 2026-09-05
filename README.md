<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pixel Adventure - مغامرة الـ 50 مرحلة</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1a1a2e;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: Arial, sans-serif;
            color: white;
            flex-direction: column;
        }
        .game-container {
            text-align: center;
        }
        canvas {
            border: 4px solid #16213e;
            box-shadow: 0 10px 20px rgba(0,0,0,0.5);
        }
        .top-bar {
            display: flex;
            justify-content: space-between;
            width: 900px;
            font-size: 18px;
            margin-bottom: 10px;
            font-weight: bold;
            color: #ffdd59;
        }
        .shop-btn-main {
            padding: 8px 16px;
            background: #ff4757;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
        }
        .shop-modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.85);
            justify-content: center;
            align-items: center;
            z-index: 10;
        }
        .shop-content {
            background: #16213e;
            padding: 20px;
            border-radius: 10px;
            width: 500px;
            max-height: 85vh;
            overflow-y: auto;
            text-align: center;
            border: 2px solid #e94560;
        }
        .shop-section-title {
            margin-top: 15px;
            color: #00fff5;
            border-bottom: 1px solid #00fff5;
            padding-bottom: 5px;
        }
        .shop-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 10px;
        }
        .item-card {
            background: #0f3460;
            padding: 10px;
            border-radius: 8px;
            border: 1px solid #ffdd59;
        }
        .item-card button {
            margin-top: 5px;
            padding: 5px 10px;
            background: #2ed573;
            border: none;
            color: white;
            border-radius: 4px;
            cursor: pointer;
        }
        .close-shop {
            margin-top: 15px;
            padding: 8px 15px;
            background: #e94560;
            border: none;
            color: white;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <h2>Pixel Adventure - عالم المغامرات</h2>
        <div class="top-bar">
            <div>الجزء: <span id="partNum">1</span> / 5</div>
            <div>المرحلة: <span id="levelNum">1</span> / 10</div>
            <div>الذهب: <span id="coinCount">0</span> 🪙</div>
            <button class="shop-btn-main" onclick="openShop()">🛍️ متجر الشخصيات والخلفيات</button>
        </div>
        
        <canvas id="gameCanvas" width="900" height="400"></canvas>
    </div>

    <!-- نافذة المتجر -->
    <div class="shop-modal" id="shopModal">
        <div class="shop-content">
            <h3>🛍️ متجر اللعبة الشامل</h3>
            
            <h4 class="shop-section-title">🎭 الشخصيات الكرتونية</h4>
            <div class="shop-grid" id="charGrid"></div>

            <h4 class="shop-section-title">🖼️ خلفيات اللعبة</h4>
            <div class="shop-grid" id="bgGrid"></div>

            <button class="close-shop" onclick="closeShop()">إغلاق المتجر</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const WORLD_WIDTH = 1600; // طريق طويل
        let cameraX = 0;
        let coins = 0;
        let currentPart = 1;
        let currentLevel = 1;

        // إعدادات اللاعب
        const player = {
            x: 50,
            y: 250,
            width: 32,
            height: 32,
            charId: "mario",
            velocityX: 0,
            velocityY: 0,
            speed: 5,
            jumpPower: -12,
            gravity: 0.6,
            isJumping: false
        };

        // الخلفية المفعلة
        let currentBg = "forest";

        // عناصر الشخصيات
        const characters = [
            { id: "mario", name: "ماريو البكسل", price: 0, icon: "🍄", bought: true, color: "#ff4757" },
            { id: "ninja", name: "النينجا السريع", price: 15, icon: "🥷", bought: false, color: "#2f3542" },
            { id: "arcade", name: "بطل الأركيد", price: 30, icon: "👾", bought: false, color: "#a55eea" },
            { id: "robot", name: "الروبوت الذكي", price: 50, icon: "🤖", bought: false, color: "#70a1ff" }
        ];

        // عناصر الخلفيات
        const backgrounds = [
            { id: "forest", name: "غابة الأشجار", price: 0, bought: true, sky: "#0f3460", tree: "#2ed573" },
            { id: "sunset", name: "الغروب الدافئ", price: 20, bought: false, sky: "#ff7f50", tree: "#d25400" },
            { id: "night", name: "الليل الساحر", price: 40, bought: false, sky: "#0c2461", tree: "#1e3799" }
        ];

        // توليد المراحل ديناميكياً لتوفير الكود وزيادة الصعوبة
        function generateLevel(part, level) {
            const difficulty = (part - 1) * 10 + level;
            const speedMultiplier = 1 + (difficulty * 0.15);
            
            let platforms = [
                { x: 0, y: 360, width: WORLD_WIDTH, height: 40, color: "#eebb4d" },
                { x: 200, y: 280, width: 140, height: 15, color: "#00fff5" },
                { x: 450, y: 220, width: 140, height: 15, color: "#00fff5" },
                { x: 700, y: 160, width: 140, height: 15, color: "#00fff5" },
                { x: 950, y: 220, width: 140, height: 15, color: "#00fff5" },
                { x: 1200, y: 280, width: 140, height: 15, color: "#00fff5" },
                { x: 1450, y: 200, width: 120, height: 15, color: "#00fff5" }
            ];

            let hazards = [];
            const hazardCount = 2 + Math.floor(difficulty / 2);
            for(let i = 0; i < hazardCount; i++) {
                hazards.push({
                    x: 300 + (i * 220),
                    y: 340,
                    width: 25,
                    height: 20,
                    speed: (2 + (i % 2)) * speedMultiplier,
                    minX: 250 + (i * 220),
                    maxX: 400 + (i * 220)
                });
            }

            let goldCoins = [];
            for(let i = 0; i < 10; i++) {
                goldCoins.push({ x: 180 + (i * 130), y: 120 + (i % 3) * 50, collected: false });
            }

            return {
                platforms: platforms,
                hazards: hazards,
                coins: goldCoins,
                house: { x: 1500, y: 140, width: 50, height: 60 }
            };
        }

        let activeLevelData = generateLevel(currentPart, currentLevel);

        const keys = {};
        window.addEventListener("keydown", (e) => { keys[e.code] = true; });
        window.addEventListener("keyup", (e) => { keys[e.code] = false; });

        function openShop() {
            renderShop();
            document.getElementById("shopModal").style.display = "flex";
        }

        function closeShop() {
            document.getElementById("shopModal").style.display = "none";
        }

        function renderShop() {
            const charGrid = document.getElementById("charGrid");
            charGrid.innerHTML = "";
            characters.forEach(c => {
                charGrid.innerHTML += `
                    <div class="item-card">
                        <div style="font-size:30px">${c.icon}</div>
                        <div><b>${c.name}</b></div>
                        <div>${c.bought ? "مملوك" : c.price + " 🪙"}</div>
                        <button onclick="buyCharacter('${c.id}')">${c.bought ? (player.charId === c.id ? "مُفعل" : "تفعيل") : "شراء"}</button>
                    </div>`;
            });

            const bgGrid = document.getElementById("bgGrid");
            bgGrid.innerHTML = "";
            backgrounds.forEach(b => {
                bgGrid.innerHTML += `
                    <div class="item-card">
                        <div><b>${b.name}</b></div>
                        <div>${b.bought ? "مملوك" : b.price + " 🪙"}</div>
                        <button onclick="buyBackground('${b.id}')">${b.bought ? (currentBg === b.id ? "مُفعل" : "تفعيل") : "شراء"}</button>
                    </div>`;
            });
        }

        function buyCharacter(id) {
            const c = characters.find(item => item.id === id);
            if (c.bought) { player.charId = c.id; }
            else if (coins >= c.price) {
                coins -= c.price;
                c.bought = true;
                player.charId = c.id;
                document.getElementById("coinCount").innerText = coins;
            }
            renderShop();
        }

        function buyBackground(id) {
            const b = backgrounds.find(item => item.id === id);
            if (b.bought) { currentBg = b.id; }
            else if (coins >= b.price) {
                coins -= b.price;
                b.bought = true;
                currentBg = b.id;
                document.getElementById("coinCount").innerText = coins;
            }
            renderShop();
        }

        function resetPlayer() {
            player.x = 50;
            player.y = 250;
            player.velocityX = 0;
            player.velocityY = 0;
            cameraX = 0;
        }

        function update() {
            if (keys["ArrowRight"]) player.velocityX = player.speed;
            else if (keys["ArrowLeft"]) player.velocityX = -player.speed;
            else player.velocityX = 0;

            if ((keys["Space"] || keys["ArrowUp"]) && !player.isJumping) {
                player.velocityY = player.jumpPower;
                player.isJumping = true;
            }

            player.velocityY += player.gravity;
            player.x += player.velocityX;
            player.y += player.velocityY;

            if (player.x < 0) player.x = 0;
            if (player.x + player.width > WORLD_WIDTH) player.x = WORLD_WIDTH - player.width;

            // الكاميرا تتبع اللاعب
            cameraX = player.x - canvas.width / 3;
            if (cameraX < 0) cameraX = 0;
            if (cameraX > WORLD_WIDTH - canvas.width) cameraX = WORLD_WIDTH - canvas.width;

            // تصادم المنصات
            player.isJumping = true;
            activeLevelData.platforms.forEach(p => {
                if (
                    player.x < p.x + p.width &&
                    player.x + player.width > p.x &&
                    player.y + player.height <= p.y + player.velocityY &&
                    player.y + player.height + player.velocityY >= p.y
                ) {
                    player.isJumping = false;
                    player.velocityY = 0;
                    player.y = p.y - player.height;
                }
            });

            // الحواجز
            activeLevelData.hazards.forEach(h => {
                h.x += h.speed;
                if (h.x < h.minX || h.x + h.width > h.maxX) h.speed *= -1;

                if (
                    player.x < h.x + h.width &&
                    player.x + player.width > h.x &&
                    player.y < h.y + h.height &&
                    player.y + player.height > h.y
                ) {
                    resetPlayer();
                }
            });

            // الذهب
            activeLevelData.coins.forEach(c => {
                if (!c.collected &&
                    player.x < c.x + 15 &&
                    player.x + player.width > c.x &&
                    player.y < c.y + 15 &&
                    player.y + player.height > c.y
                ) {
                    c.collected = true;
                    coins += 2;
                    document.getElementById("coinCount").innerText = coins;
                }
            });

            // الفوز بالوصول للبيت
            const house = activeLevelData.house;
            if (
                player.x < house.x + house.width &&
                player.x + player.width > house.x &&
                player.y < house.y + house.height &&
                player.y + player.height > house.y
            ) {
                if (currentLevel < 10) {
                    currentLevel++;
                } else if (currentPart < 5) {
                    currentPart++;
                    currentLevel = 1;
                    alert(`🎉 تهانينا! انتقلت إلى الجزء الرقم ${currentPart}!`);
                } else {
                    alert("🏆 أسطورة! لقد ختمت جميع الأجزاء والـ 50 مرحلة بالكامل!");
                    currentPart = 1;
                    currentLevel = 1;
                }
                document.getElementById("partNum").innerText = currentPart;
                document.getElementById("levelNum").innerText = currentLevel;
                activeLevelData = generateLevel(currentPart, currentLevel);
                resetPlayer();
            }
        }

        function drawBackgroundTrees(bgInfo) {
            for (let i = 0; i < WORLD_WIDTH; i += 180) {
                ctx.fillStyle = bgInfo.tree;
                // جذع الشجرة
                ctx.fillRect(i + 20, 280, 15, 80);
                // أوراق الشجرة
                ctx.beginPath();
                ctx.arc(i + 27, 260, 30, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        function drawPlayer() {
            const currentChar = characters.find(c => c.id === player.charId);
            ctx.fillStyle = currentChar.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);

            // تفاصيل الشخصيات
            ctx.fillStyle = "#ffffff";
            if (player.charId === "mario") {
                ctx.fillRect(player.x + 6, player.y + 4, 20, 8); // قبعة ماريو
            } else if (player.charId === "ninja") {
                ctx.fillRect(player.x + 4, player.y + 10, 24, 6); // عصابة العينين
            } else if (player.charId === "robot") {
                ctx.fillStyle = "#00fff5";
                ctx.fillRect(player.x + 8, player.y + 8, 6, 6); // عين إلكترونية
            }
        }

        function draw() {
            const bgInfo = backgrounds.find(b => b.id === currentBg);
            ctx.fillStyle = bgInfo.sky;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.save();
            ctx.translate(-cameraX, 0); // تطبيق حركة الكاميرا

            // رسم الأشجار الخفية
            drawBackgroundTrees(bgInfo);

            // رسم المنصات
            activeLevelData.platforms.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.fillRect(p.x, p.y, p.width, p.height);
            });

            // رسم الحواجز
            activeLevelData.hazards.forEach(h => {
                ctx.fillStyle = "#ff4757";
                ctx.fillRect(h.x, h.y, h.width, h.height);
            });

            // رسم الذهب
            activeLevelData.coins.forEach(c => {
                if (!c.collected) {
                    ctx.fillStyle = "#ffdd59";
                    ctx.beginPath();
                    ctx.arc(c.x, c.y, 8, 0, Math.PI * 2);
                    ctx.fill();
                }
            });

            // رسم البيت
            const h = activeLevelData.house;
            ctx.fillStyle = "#ff6b6b";
            ctx.fillRect(h.x, h.y + 20, h.width, h.height - 20);
            ctx.fillStyle = "#1e90ff";
            ctx.beginPath();
            ctx.moveTo(h.x - 5, h.y + 20);
            ctx.lineTo(h.x + h.width / 2, h.y);
            ctx.lineTo(h.x + h.width + 5, h.y + 20);
            ctx.fill();

            // رسم الشخصية
            drawPlayer();

            ctx.restore();
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        gameLoop();
    </script>
</body>
</html>
