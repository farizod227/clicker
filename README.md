<!DOCTYPE html>
<html lang="ru">
<head>
  <meta name="google-site-verification" content="9Jvd_Mf5bcPPfEsChzlaKMXlBa0U4dsM6dhKn8-NiEI" />
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Монетный Кликер + Топ</title>
  <style>
    :root {
      --bg: #0f172a;
      --card-bg: #1e293b;
      --accent: #f59e0b;
      --accent-hover: #d97706;
      --text: #f8fafc;
      --text-muted: #94a3b8;
      --nav-bg: #334155;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: sans-serif;
      user-select: none;
    }

    body {
      background-color: var(--bg);
      color: var(--text);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
    }

    .game-container {
      width: 100%;
      max-width: 420px;
      background: var(--card-bg);
      padding: 24px;
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.4);
      text-align: center;
    }

    .nav-tabs {
      display: flex;
      gap: 10px;
      margin-bottom: 20px;
      background: var(--nav-bg);
      padding: 6px;
      border-radius: 12px;
    }

    .tab-btn {
      flex: 1;
      padding: 12px;
      border: none;
      background: transparent;
      color: var(--text-muted);
      font-weight: bold;
      cursor: pointer;
      border-radius: 8px;
    }

    .tab-btn.active {
      background: var(--card-bg);
      color: var(--accent);
    }

    .page {
      display: none;
    }

    .page.active {
      display: block;
    }

    h1 {
      font-size: 1.5rem;
      margin-bottom: 8px;
    }

    .score-container {
      margin: 15px 0;
    }

    .score {
      font-size: 2.5rem;
      font-weight: bold;
      color: var(--accent);
    }

    .stats {
      font-size: 0.9rem;
      color: var(--text-muted);
    }

    .coin-wrapper {
      position: relative;
      margin: 25px 0;
      display: flex;
      justify-content: center;
    }

    .coin {
      width: 140px;
      height: 140px;
      background: radial-gradient(circle, #fbbf24 0%, #d97706 100%);
      border-radius: 50%;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 4rem;
      cursor: pointer;
      box-shadow: 0 8px 20px rgba(245, 158, 11, 0.4);
      transition: transform 0.05s ease;
      border: 6px solid #fef3c7;
    }

    .coin:active {
      transform: scale(0.92);
    }

    .floating-text {
      position: absolute;
      color: #fbbf24;
      font-size: 1.4rem;
      font-weight: bold;
      pointer-events: none;
      animation: floatUp 0.8s ease-out forwards;
    }

    @keyframes floatUp {
      0% { opacity: 1; transform: translateY(0); }
      100% { opacity: 0; transform: translateY(-50px); }
    }

    .upgrades {
      display: flex;
      flex-direction: column;
      gap: 12px;
      margin-top: 20px;
    }

    .upgrade-btn {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px 16px;
      background: #334155;
      border: none;
      border-radius: 12px;
      color: white;
      cursor: pointer;
    }

    .upgrade-btn:disabled {
      opacity: 0.5;
    }

    .upgrade-info div:first-child {
      font-weight: bold;
    }

    .upgrade-cost {
      font-weight: bold;
      color: var(--accent);
    }

    .leaderboard-list {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-top: 15px;
    }

    .leader-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px;
      background: #334155;
      border-radius: 10px;
    }

    .leader-item.player {
      border: 2px solid var(--accent);
    }
  </style>
</head>
<body>

  <div class="game-container">
    <div class="nav-tabs">
      <button class="tab-btn active" id="btnClicker" onclick="switchPage('clickerPage')">Кликер</button>
      <button class="tab-btn" id="btnLeaderboard" onclick="switchPage('leaderboardPage')">🏆 Топ игроков</button>
    </div>

    <!-- СТРАНИЦА 1 -->
    <div id="clickerPage" class="page active">
      <h1>🪙 Монетный Кликер</h1>
      <div class="score-container">
        <div class="score" id="coins">0</div>
        <div class="stats">В секунду: <span id="cps">0</span></div>
      </div>
      <div class="coin-wrapper">
        <div class="coin" id="coinBtn">🪙</div>
      </div>
      <div class="upgrades">
        <button class="upgrade-btn" id="clickUpgrade" onclick="buyClickUpgrade()">
          <div class="upgrade-info">
            <div>Усиленный клик</div>
            <div>+1 за клик (Ур. <span id="clickLvl">1</span>)</div>
          </div>
          <div class="upgrade-cost"><span id="clickCost">10</span> 🪙</div>
        </button>
        <button class="upgrade-btn" id="passiveUpgrade" onclick="buyPassiveUpgrade()">
          <div class="upgrade-info">
            <div>Авто-ферма</div>
            <div>+1 в сек (Ур. <span id="passiveLvl">0</span>)</div>
          </div>
          <div class="upgrade-cost"><span id="passiveCost">25</span> 🪙</div>
        </button>
      </div>
    </div>

    <!-- СТРАНИЦА 2 -->
    <div id="leaderboardPage" class="page">
      <h1>🏆 Таблица Лидеров</h1>
      <div class="leaderboard-list" id="leaderboardList"></div>
    </div>
  </div>

  <script>
    var state = {
      coins: 0,
      clickPower: 1,
      clickLvl: 1,
      clickCost: 10,
      cps: 0,
      passiveLvl: 0,
      passiveCost: 25
    };

    var leaders = [
      { name: "👑 Султан Клик", coins: 50000, cps: 25 },
      { name: "🚀 Илон Маск", coins: 15000, cps: 12 },
      { name: "⚡ Крипто-Бро", coins: 4500, cps: 5 },
      { name: "🐱 Кот_Компот", coins: 800, cps: 1 },
      { name: "👾 Нубик3000", coins: 120, cps: 0 }
    ];

    var savedState = localStorage.getItem('clicker_save');
    if (savedState) {
      try { state = JSON.parse(savedState); } catch(e) {}
    }

    function saveGame() {
      localStorage.setItem('clicker_save', JSON.stringify(state));
    }

    function switchPage(pageId) {
      document.getElementById('clickerPage').classList.remove('active');
      document.getElementById('leaderboardPage').classList.remove('active');
      document.getElementById('btnClicker').classList.remove('active');
      document.getElementById('btnLeaderboard').classList.remove('active');

      document.getElementById(pageId).classList.add('active');
      if (pageId === 'clickerPage') {
        document.getElementById('btnClicker').classList.add('active');
      } else {
        document.getElementById('btnLeaderboard').classList.add('active');
        renderLeaderboard();
      }
    }

    function updateUI() {
      document.getElementById('coins').innerText = Math.floor(state.coins);
      document.getElementById('cps').innerText = state.cps;
      document.getElementById('clickLvl').innerText = state.clickLvl;
      document.getElementById('clickCost').innerText = state.clickCost;
      document.getElementById('clickUpgrade').disabled = state.coins < state.clickCost;
      document.getElementById('passiveLvl').innerText = state.passiveLvl;
      document.getElementById('passiveCost').innerText = state.passiveCost;
      document.getElementById('passiveUpgrade').disabled = state.coins < state.passiveCost;
    }

    function renderLeaderboard() {
      var listEl = document.getElementById('leaderboardList');
      listEl.innerHTML = '';

      var allPlayers = [];
      for (var i = 0; i < leaders.length; i++) {
        allPlayers.push(leaders[i]);
      }
      allPlayers.push({ name: "Ты (Игрок)", coins: state.coins, isPlayer: true });

      allPlayers.sort(function(a, b) { return b.coins - a.coins; });

      for (var j = 0; j < allPlayers.length; j++) {
        var p = allPlayers[j];
        var item = document.createElement('div');
        item.className = 'leader-item' + (p.isPlayer ? ' player' : '');
        
        var rank = j + 1;
        if (j === 0) rank = "🥇";
        if (j === 1) rank = "🥈";
        if (j === 2) rank = "🥉";

        item.innerHTML = '<div>' + rank + ' ' + p.name + '</div><div style="font-weight:bold;color:#f59e0b;">' + Math.floor(p.coins) + ' 🪙</div>';
        listEl.appendChild(item);
      }
    }

    document.getElementById('coinBtn').addEventListener('click', function(e) {
      state.coins += state.clickPower;
      createFloatingText(e.clientX, e.clientY, '+' + state.clickPower);
      updateUI();
      saveGame();
    });

    function createFloatingText(x, y, text) {
      var floatEl = document.createElement('div');
      floatEl.className = 'floating-text';
      floatEl.innerText = text;
      floatEl.style.left = (x - 15) + 'px';
      floatEl.style.top = (y - 20) + 'px';
      document.body.appendChild(floatEl);
      setTimeout(function() { floatEl.remove(); }, 800);
    }

    function buyClickUpgrade() {
      if (state.coins >= state.clickCost) {
        state.coins -= state.clickCost;
        state.clickPower += 1;
        state.clickLvl += 1;
        state.clickCost = Math.floor(state.clickCost * 1.5);
        updateUI();
        saveGame();
      }
    }

    function buyPassiveUpgrade() {
      if (state.coins >= state.passiveCost) {
        state.coins -= state.passiveCost;
        state.cps += 1;
        state.passiveLvl += 1;
        state.passiveCost = Math.floor(state.passiveCost * 1.6);
        updateUI();
        saveGame();
      }
    }

    setInterval(function() {
      if (state.cps > 0) {
        state.coins += state.cps / 10;
        updateUI();
      }
      for (var i = 0; i < leaders.length; i++) {
        leaders[i].coins += leaders[i].cps / 10;
      }
      if (document.getElementById('leaderboardPage').classList.contains('active')) {
        renderLeaderboard();
      }
    }, 100);

    setInterval(saveGame, 5000);
    updateUI();
  </script>
</body>
</html>
