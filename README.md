<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Lotus Elite Casino</title>
    <script src="https://telegram.org"></script>
    <style>
        :root {
            --bg: #0f0f0f; --card: #1c1c1e; --text: #ffffff;
            --accent: #ffcc00; --red: #ff3b30; --black: #1c1c1e;
            --green: #4cd964; --btn: #007aff; --hint: #8e8e93;
        }

        body {
            background-color: var(--bg); color: var(--text);
            font-family: -apple-system, sans-serif; margin: 0; padding-bottom: 80px;
            overflow-x: hidden; -webkit-tap-highlight-color: transparent;
        }

        .header {
            display: flex; justify-content: space-between; align-items: center;
            padding: 15px 20px; background: var(--card); border-bottom: 1px solid #333;
            position: sticky; top: 0; z-index: 1000;
        }
        .balance-display { color: var(--accent); font-weight: 800; font-size: 18px; }

        /* Таймер */
        .timer-box {
            background: #2c2c2e; padding: 10px; border-radius: 12px;
            margin: 10px auto; width: 200px; text-align: center;
            border: 1px solid var(--accent); font-weight: bold;
        }
        #timer-count { color: var(--accent); font-size: 20px; }

        .container { padding: 15px; display: none; animation: fadeIn 0.3s ease; }
        .active { display: block; }

        /* Рулетка */
        .wheel-area { position: relative; width: 280px; height: 280px; margin: 10px auto; }
        canvas { width: 100%; height: 100%; border-radius: 50%; transition: transform 4s cubic-bezier(0.15, 0, 0.15, 1); border: 5px solid #333; }
        .pointer { position: absolute; top: -15px; left: 50%; transform: translateX(-50%); width: 0; height: 0; border-left: 15px solid transparent; border-right: 15px solid transparent; border-top: 25px solid var(--accent); z-index: 10; }

        .bet-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-top: 10px; }
        .bet-3 { grid-template-columns: repeat(3, 1fr); }
        .bet-item { 
            background: var(--card); padding: 10px; border-radius: 10px; 
            font-size: 12px; border: 1px solid #333; text-align: center;
        }
        .bet-item.active-bet { border-color: var(--accent); background: #2c2c2e; }

        input { 
            width: 100%; padding: 15px; border-radius: 12px; border: 1px solid #333; 
            background: var(--card); color: #fff; font-size: 16px; box-sizing: border-box; margin: 15px 0;
        }

        .nav {
            position: fixed; bottom: 0; width: 100%; height: 70px;
            background: var(--card); display: flex; justify-content: space-around; 
            align-items: center; border-top: 1px solid #333;
        }
        .nav-item { font-size: 11px; color: var(--hint); text-align: center; flex: 1; }
        .nav-item.active-nav { color: var(--accent); }

        .btn-play { 
            width: 100%; padding: 15px; background: var(--btn); color: #fff; 
            border: none; border-radius: 12px; font-size: 16px; font-weight: bold; 
        }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

    <div class="header">
        <div id="u_name">Игрок</div>
        <div class="balance-display"><span id="bal">0</span> 🪙</div>
    </div>

    <!-- ГЛАВНАЯ (ИГРЫ) -->
    <div id="page-games" class="container active">
        <h2 style="margin-top:0">🎰 Лобби</h2>
        <div class="bet-item" style="padding:25px; margin-bottom:15px;" onclick="showPage('wheel')">
            <div style="font-size:24px">🎡</div>
            <div style="font-weight:bold">Roulette (Таймер 20с)</div>
        </div>
        <div class="bet-item" style="padding:25px;" onclick="showPage('dice')">
            <div style="font-size:24px">🎲</div>
            <div style="font-weight:bold">Dice Game</div>
        </div>
    </div>

    <!-- РУЛЕТКА -->
    <div id="page-wheel" class="container">
        <div class="timer-box">Ставки закроются через: <span id="timer-count">20</span>с</div>
        
        <div class="wheel-area">
            <div class="pointer"></div>
            <canvas id="wheelCanvas" width="400" height="400"></canvas>
        </div>

        <div class="bet-grid">
            <div class="bet-item" onclick="setBet('red', 2)">Красное x2</div>
            <div class="bet-item" onclick="setBet('black', 2)">Черное x2</div>
            <div class="bet-item" onclick="setBet('even', 2)">Четное x2</div>
            <div class="bet-item" onclick="setBet('odd', 2)">Нечетное x2</div>
        </div>
        <div class="bet-grid bet-3" style="margin-top:8px">
            <div class="bet-item" onclick="setBet('1-12', 3)">1-12 x3</div>
            <div class="bet-item" onclick="setBet('13-24', 3)">13-24 x3</div>
            <div class="bet-item" onclick="setBet('25-36', 3)">25-36 x3</div>
        </div>

        <input type="number" id="wheelAmt" placeholder="Сумма ставки...">
        <button class="btn-play" id="wheelSpinBtn" disabled>ОЖИДАНИЕ ТАЙМЕРА...</button>
    </div>

    <!-- КУБИК -->
    <div id="page-dice" class="container" style="text-align:center;">
        <div id="dice-res" style="font-size:80px; margin:20px;">🎲</div>
        <div class="bet-grid">
            <div class="bet-item" onclick="setBet('d-even', 2)">Четное x2</div>
            <div class="bet-item" onclick="setBet('d-odd', 2)">Нечет x2</div>
        </div>
        <input type="number" id="diceAmt" placeholder="Ставка...">
        <button class="btn-play" onclick="playDice()">БРОСИТЬ</button>
    </div>

    <!-- ПРОФИЛЬ -->
    <div id="page-profile" class="container">
        <h2>👤 Профиль</h2>
        <div class="bet-item" style="text-align:left; padding:20px;">
            <div id="profile-name">User</div>
            <div id="profile-id" style="color:var(--hint); font-size:12px;">ID: 000000</div>
        </div>
    </div>

    <div class="nav">
        <div class="nav-item active-nav" onclick="showPage('games', this)">🎮 Игры</div>
        <div class="nav-item" onclick="showPage('profile', this)">👤 Профиль</div>
    </div>

    <script>
        const tg = window.Telegram.WebApp;
        tg.expand();

        let balance = localStorage.getItem('cs_bal') ? parseInt(localStorage.getItem('cs_bal')) : 5000;
        let selectedBet = null;
        let betMult = 1;
        let wheelRot = 0;
        let timerSeconds = 20;
        let timerInterval = null;
        let isSpinning = false;

        function updateBal() {
            document.getElementById('bal').innerText = balance;
            localStorage.setItem('cs_bal', balance);
        }
        updateBal();

        if (tg.initDataUnsafe?.user) {
            document.getElementById('u_name').innerText = tg.initDataUnsafe.user.first_name;
            document.getElementById('profile-name').innerText = tg.initDataUnsafe.user.first_name;
            document.getElementById('profile-id').innerText = "ID: " + tg.initDataUnsafe.user.id;
        }

        function showPage(id, el) {
            document.querySelectorAll('.container').forEach(p => p.classList.remove('active'));
            document.getElementById('page-' + id).classList.add('active');
            if(el) {
                document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active-nav'));
                el.classList.add('active-nav');
            }
            if(id === 'wheel' && !timerInterval) startTimer();
        }

        function setBet(type, x) {
            if(isSpinning) return;
            selectedBet = type; betMult = x;
            document.querySelectorAll('.bet-item').forEach(i => i.classList.remove('active-bet'));
            event.currentTarget.classList.add('active-bet');
            tg.HapticFeedback.impactOccurred('light');
        }

        // РУЛЕТКА
        const numbers = [0,32,15,19,4,21,2,25,17,34,6,27,13,36,11,30,8,23,10,5,24,16,33,1,20,14,31,9,22,18,29,7,28,12,35,3,26];
        const colors = numbers.map(n => n === 0 ? 'green' : ([1,3,5,7,9,12,14,16,18,19,21,23,25,27,30,32,34,36].includes(n) ? 'red' : 'black'));
        
        const canvas = document.getElementById('wheelCanvas');
        const ctx = canvas.getContext('2d');

        function drawWheel() {
            const arc = (Math.PI * 2) / 37;
            numbers.forEach((n, i) => {
                ctx.beginPath();
                ctx.fillStyle = colors[i] === 'red' ? '#ff3b30' : (colors[i] === 'black' ? '#1c1c1e' : '#4cd964');
                ctx.moveTo(200, 200); ctx.arc(200, 200, 200, i * arc, (i + 1) * arc);
                ctx.fill(); ctx.save();
                ctx.translate(200, 200); ctx.rotate(i * arc + arc / 2);
                ctx.fillStyle = "#fff"; ctx.font = "bold 14px Arial";
                ctx.fillText(n, 165, 5); ctx.restore();
            });
        }
        drawWheel();

        // ЛОГИКА ТАЙМЕРА
        function startTimer() {
            timerSeconds = 20;
            document.getElementById('timer-count').innerText = timerSeconds;
            
            timerInterval = setInterval(() => {
                timerSeconds--;
                document.getElementById('timer-count').innerText = timerSeconds;
                
                if(timerSeconds <= 0) {
                    clearInterval(timerInterval);
                    timerInterval = null;
                    playRoulette(); // Авто-запуск
                }
            }, 1000);
        }

        function playRoulette() {
            let amt = parseInt(document.getElementById('wheelAmt').value) || 0;
            
            // Если игрок не сделал ставку, просто крутим вхолостую
            if (amt > 0 && amt <= balance && selectedBet) {
                balance -= amt;
                updateBal();
            } else {
                amt = 0; // Вхолостую
            }

            isSpinning = true;
            const extra = Math.floor(Math.random() * 360) + 2160;
            wheelRot += extra;
            canvas.style.transform = `rotate(${wheelRot}deg)`;

            setTimeout(() => {
                const actualDeg = wheelRot % 360;
                const idx = Math.floor((37 - (actualDeg / (360/37))) % 37);
                const n = numbers[idx];
                const c = colors[idx];

                if(amt > 0) {
                    let isWin = false;
                    if(selectedBet === 'red' && c === 'red') isWin = true;
                    if(selectedBet === 'black' && c === 'black') isWin = true;
                    if(selectedBet === 'even' && n % 2 === 0 && n !== 0) isWin = true;
                    if(selectedBet === 'odd' && n % 2 !== 0) isWin = true;
                    if(selectedBet === '1-12' && n >= 1 && n <= 12) isWin = true;
                    if(selectedBet === '13-24' && n >= 13 && n <= 24) isWin = true;
                    if(selectedBet === '25-36' && n >= 25 && n <= 36) isWin = true;

                    if(isWin) {
                        let win = amt * betMult;
                        balance += win;
                        tg.showAlert(`Выпало ${n}! Победа: +${win}`);
                    } else {
                        tg.showAlert(`Выпало ${n}. Проигрыш.`);
                    }
                    updateBal();
                } else {
                    tg.showAlert(`Выпало ${n}. (Ставка не была сделана)`);
                }

                isSpinning = false;
                startTimer(); // Перезапуск таймера для следующего раунда
            }, 4500);
        }

        // DICE
        function playDice() {
            let amt = parseInt(document.getElementById('diceAmt').value);
            if(!amt || amt > balance || !selectedBet) return;
            balance -= amt; updateBal();
            const res = Math.floor(Math.random() * 6) + 1;
            document.getElementById('dice-res').innerText = ['⚀','⚁','⚂','⚃','⚄','⚅'][res-1];
            tg.showAlert(res % 2 === 0 && selectedBet === 'd-even' ? "Победа!" : "Мимо");
        }
    </script>
</body>
</html>
