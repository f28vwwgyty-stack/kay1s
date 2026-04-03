<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Умный Алматы | Карта + AI + Викторина</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/moment@2.29.4/moment.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #0a0f1e 0%, #0f1629 100%);
            color: #e0e4f0;
            overflow: hidden;
            height: 100vh;
        }

        .app {
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* Top Navigation */
        .top-nav {
            background: rgba(10, 15, 30, 0.95);
            backdrop-filter: blur(10px);
            border-bottom: 1px solid rgba(59, 130, 246, 0.3);
            padding: 0 20px;
            z-index: 1000;
        }

        .nav-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 15px;
            padding: 12px 0;
        }

        .logo h1 {
            font-size: 22px;
            background: linear-gradient(135deg, #3b82f6, #10b981, #8b5cf6);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
        }

        .logo p {
            font-size: 10px;
            color: #94a3b8;
        }

        .nav-tabs {
            display: flex;
            gap: 5px;
            background: rgba(0,0,0,0.3);
            padding: 5px;
            border-radius: 40px;
            flex-wrap: wrap;
        }

        .nav-tab {
            padding: 8px 20px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 13px;
            font-weight: 500;
            transition: all 0.2s;
            background: transparent;
            color: #94a3b8;
        }

        .nav-tab.active {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            color: white;
        }

        .nav-tab:hover:not(.active) {
            background: rgba(59,130,246,0.2);
            color: white;
        }

        .status-badge {
            padding: 6px 14px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
        }

        .status-normal { background: rgba(34,197,94,0.2); color: #4ade80; border: 1px solid #4ade80; }
        .status-warning { background: rgba(234,179,8,0.2); color: #fbbf24; border: 1px solid #fbbf24; }
        .status-critical { background: rgba(239,68,68,0.2); color: #f87171; border: 1px solid #f87171; }

        .refresh-btn {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            border: none;
            color: white;
            padding: 6px 18px;
            border-radius: 20px;
            cursor: pointer;
            font-size: 12px;
        }

        /* Main Content */
        .main-content {
            flex: 1;
            position: relative;
            overflow: hidden;
        }

        .tab-content {
            display: none;
            height: 100%;
            overflow-y: auto;
        }

        .tab-content.active {
            display: block;
        }

        /* Map Container */
        .map-container {
            height: 100%;
            position: relative;
        }

        #almatyMap {
            height: 100%;
            width: 100%;
            background: #0f172a;
        }

        .map-legend {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0,0,0,0.8);
            padding: 10px 15px;
            border-radius: 10px;
            font-size: 10px;
            z-index: 1000;
        }

        /* Dashboard */
        .dashboard-grid {
            padding: 20px;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            max-width: 1400px;
            margin: 0 auto;
        }

        .dashboard-card {
            background: rgba(18, 25, 45, 0.9);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 20px;
            border: 1px solid rgba(59, 130, 246, 0.3);
        }

        .card-title {
            font-size: 18px;
            font-weight: 600;
            margin-bottom: 15px;
            padding-bottom: 10px;
            border-bottom: 1px solid rgba(59,130,246,0.3);
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .kpi-row {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
            margin-bottom: 20px;
        }

        .kpi-box {
            text-align: center;
            padding: 15px;
            background: rgba(0,0,0,0.3);
            border-radius: 15px;
        }

        .kpi-value {
            font-size: 32px;
            font-weight: bold;
        }

        .street-list {
            max-height: 250px;
            overflow-y: auto;
        }

        .street-item {
            padding: 10px;
            margin-bottom: 8px;
            border-radius: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            background: rgba(0,0,0,0.2);
        }

        .street-red { border-left: 3px solid #ef4444; }
        .street-orange { border-left: 3px solid #f97316; }
        .street-yellow { border-left: 3px solid #fbbf24; }
        .street-green { border-left: 3px solid #4ade80; }

        /* AI Chat */
        .ai-chat-container {
            display: flex;
            flex-direction: column;
            height: 100%;
            padding: 20px;
        }

        .chat-messages {
            flex: 1;
            overflow-y: auto;
            background: rgba(0,0,0,0.2);
            border-radius: 20px;
            padding: 20px;
            margin-bottom: 20px;
        }

        .message {
            margin-bottom: 15px;
            max-width: 80%;
        }

        .user-message {
            text-align: right;
            margin-left: auto;
        }

        .user-message .msg-text {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            display: inline-block;
            padding: 10px 16px;
            border-radius: 20px 20px 0 20px;
        }

        .ai-message .msg-text {
            background: rgba(255,255,255,0.1);
            display: inline-block;
            padding: 10px 16px;
            border-radius: 20px 20px 20px 0;
        }

        .chat-input-area {
            display: flex;
            gap: 12px;
        }

        .chat-input {
            flex: 1;
            padding: 12px 16px;
            background: rgba(0,0,0,0.4);
            border: 1px solid rgba(59,130,246,0.3);
            border-radius: 30px;
            color: white;
            outline: none;
        }

        /* AI Forecast */
        .forecast-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 20px;
            padding: 20px;
        }

        .forecast-card {
            background: rgba(18, 25, 45, 0.9);
            border-radius: 20px;
            padding: 20px;
        }

        .hourly-forecast {
            display: flex;
            gap: 15px;
            overflow-x: auto;
            padding: 10px 0;
        }

        .hour-item {
            text-align: center;
            min-width: 70px;
            padding: 10px;
            background: rgba(0,0,0,0.3);
            border-radius: 12px;
        }

        /* Quiz */
        .quiz-container {
            max-width: 800px;
            margin: 40px auto;
            padding: 30px;
            background: rgba(18, 25, 45, 0.9);
            border-radius: 30px;
            text-align: center;
        }

        .quiz-question {
            font-size: 24px;
            margin-bottom: 30px;
        }

        .quiz-options {
            display: grid;
            gap: 15px;
            margin-bottom: 30px;
        }

        .quiz-option {
            padding: 15px;
            background: rgba(255,255,255,0.1);
            border-radius: 15px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .quiz-option:hover {
            background: rgba(59,130,246,0.3);
            transform: scale(1.02);
        }

        .quiz-option.correct {
            background: rgba(34,197,94,0.3);
            border: 1px solid #4ade80;
        }

        .quiz-option.wrong {
            background: rgba(239,68,68,0.3);
            border: 1px solid #ef4444;
        }

        .quiz-score {
            font-size: 18px;
            margin-top: 20px;
        }

        .next-btn {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            border: none;
            color: white;
            padding: 12px 30px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 16px;
            margin-top: 20px;
        }

        @media (max-width: 900px) {
            .dashboard-grid { grid-template-columns: 1fr; }
            .forecast-grid { grid-template-columns: 1fr; }
            .nav-tab { padding: 6px 12px; font-size: 11px; }
        }
    </style>
</head>
<body>
    <div class="app">
        <div class="top-nav">
            <div class="nav-container">
                <div class="logo">
                    <h1>🏙️ Умный Алматы | Smart City</h1>
                    <p>AI-аналитика | Карта | Прогнозы | Викторина</p>
                </div>
                <div class="nav-tabs">
                    <div class="nav-tab active" data-tab="map" onclick="switchTab('map')">🗺️ Карта</div>
                    <div class="nav-tab" data-tab="dashboard" onclick="switchTab('dashboard')">📊 Дашборд</div>
                    <div class="nav-tab" data-tab="ai" onclick="switchTab('ai')">🤖 AI Помощник</div>
                    <div class="nav-tab" data-tab="forecast" onclick="switchTab('forecast')">📈 AI Прогнозы</div>
                    <div class="nav-tab" data-tab="quiz" onclick="switchTab('quiz')">🎮 Викторина</div>
                </div>
                <div style="display: flex; gap: 10px; align-items: center;">
                    <div class="status-badge" id="statusBadge">🟢 Онлайн</div>
                    <button class="refresh-btn" onclick="refreshAll()">🔄 Обновить</button>
                </div>
            </div>
        </div>

        <div class="main-content">
            <!-- Вкладка Карта -->
            <div class="tab-content active" id="mapTab">
                <div class="map-container">
                    <div id="almatyMap"></div>
                    <div class="map-legend">
                        <div><span style="display:inline-block; width:16px; height:3px; background:#4ade80;"></span> 🟢 Свободно</div>
                        <div><span style="display:inline-block; width:16px; height:3px; background:#fbbf24;"></span> 🟡 Загружено</div>
                        <div><span style="display:inline-block; width:16px; height:3px; background:#f97316;"></span> 🟠 Пробки</div>
                        <div><span style="display:inline-block; width:16px; height:3px; background:#ef4444;"></span> 🔴 Красная дорога</div>
                    </div>
                </div>
            </div>

            <!-- Вкладка Дашборд -->
            <div class="tab-content" id="dashboardTab">
                <div class="dashboard-grid">
                    <div class="dashboard-card">
                        <div class="card-title">🚗 Транспортная ситуация</div>
                        <div class="kpi-row">
                            <div class="kpi-box"><div class="kpi-value" id="avgCongestion">--<span style="font-size:14px;">%</span></div><div class="kpi-label">Загруженность</div></div>
                            <div class="kpi-box"><div class="kpi-value" id="avgSpeed">--<span style="font-size:14px;">км/ч</span></div><div class="kpi-label">Скорость</div></div>
                            <div class="kpi-box"><div class="kpi-value" id="totalIncidents">0</div><div class="kpi-label">Происшествий</div></div>
                        </div>
                        <div class="card-title">🛣️ Загруженность улиц</div>
                        <div class="street-list" id="streetList">Загрузка...</div>
                    </div>
                    <div class="dashboard-card">
                        <div class="card-title">🌿 Экология</div>
                        <div class="kpi-row">
                            <div class="kpi-box"><div class="kpi-value" id="aqi">--</div><div class="kpi-label">AQI</div></div>
                            <div class="kpi-box"><div class="kpi-value" id="pm25">--<span style="font-size:14px;">µg</span></div><div class="kpi-label">PM2.5</div></div>
                            <div class="kpi-box"><div class="kpi-value" id="temperature">--<span style="font-size:14px;">°C</span></div><div class="kpi-label">Температура</div></div>
                        </div>
                        <div class="card-title">⚠️ Происшествия</div>
                        <div id="incidentList" style="max-height: 200px; overflow-y: auto;">Загрузка...</div>
                    </div>
                </div>
            </div>

            <!-- Вкладка AI Помощник -->
            <div class="tab-content" id="aiTab">
                <div class="ai-chat-container">
                    <div class="chat-messages" id="chatMessages">
                        <div class="message ai-message">
                            <div class="msg-text">👋 Здравствуйте! Я AI-помощник города Алматы. Спрашивайте меня о пробках, улицах, погоде или любых городских событиях!</div>
                        </div>
                    </div>
                    <div class="chat-input-area">
                        <input type="text" class="chat-input" id="chatInput" placeholder="Напишите вопрос... Например: какая ситуация на Абая?">
                        <button class="refresh-btn" onclick="sendChatMessage()" style="padding: 12px 24px;">Отправить</button>
                    </div>
                </div>
            </div>

            <!-- Вкладка AI Прогнозы -->
            <div class="tab-content" id="forecastTab">
                <div class="forecast-grid">
                    <div class="forecast-card">
                        <div class="card-title">📊 Прогноз на сегодня</div>
                        <div id="dailyForecast" style="line-height: 1.8;">Загрузка...</div>
                    </div>
                    <div class="forecast-card">
                        <div class="card-title">⏰ Почасовая загруженность</div>
                        <div class="hourly-forecast" id="hourlyForecast">Загрузка...</div>
                    </div>
                    <div class="forecast-card">
                        <div class="card-title">🌤️ Погодный прогноз</div>
                        <div id="weatherForecast" style="line-height: 1.8;">Загрузка...</div>
                    </div>
                    <div class="forecast-card">
                        <div class="card-title">💡 Рекомендации AI</div>
                        <div id="aiRecommendations" style="line-height: 1.8;">Загрузка...</div>
                    </div>
                </div>
            </div>

            <!-- Вкладка Викторина -->
            <div class="tab-content" id="quizTab">
                <div class="quiz-container" id="quizContainer">
                    <div class="quiz-question" id="quizQuestion">Загрузка...</div>
                    <div class="quiz-options" id="quizOptions"></div>
                    <div class="quiz-score" id="quizScore">Счет: 0 / 0</div>
                    <button class="next-btn" id="nextBtn" onclick="nextQuestion()">Следующий вопрос</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // ========== ДАННЫЕ ==========
        const almatyStreets = [
            { name: "пр. Достык", lat: 43.255, lng: 76.930, congestion: 78, incidents: 2, status: "orange" },
            { name: "ул. Толе би", lat: 43.260, lng: 76.920, congestion: 82, incidents: 3, status: "red" },
            { name: "пр. Абая", lat: 43.250, lng: 76.910, congestion: 88, incidents: 4, status: "red" },
            { name: "ул. Сатпаева", lat: 43.248, lng: 76.905, congestion: 68, incidents: 1, status: "yellow" },
            { name: "ул. Жандосова", lat: 43.235, lng: 76.890, congestion: 55, incidents: 0, status: "green" },
            { name: "пр. Райымбека", lat: 43.270, lng: 76.925, congestion: 85, incidents: 3, status: "red" },
            { name: "ул. Гагарина", lat: 43.240, lng: 76.935, congestion: 72, incidents: 1, status: "orange" },
            { name: "ул. Фурманова", lat: 43.252, lng: 76.940, congestion: 65, incidents: 1, status: "yellow" },
            { name: "пр. Сейфуллина", lat: 43.245, lng: 76.915, congestion: 75, incidents: 2, status: "orange" },
            { name: "ул. Жибек Жолы", lat: 43.262, lng: 76.935, congestion: 80, incidents: 2, status: "red" }
        ];

        const quizQuestions = [
            { question: "Какая улица в Алматы считается самой длинной?", options: ["пр. Абая", "пр. Достык", "ул. Толе би", "пр. Райымбека"], correct: 0 },
            { question: "Какой район Алматы самый густонаселенный?", options: ["Медеуский", "Ауэзовский", "Алмалинский", "Бостандыкский"], correct: 1 },
            { question: "В каком году Алматы стал называться Алматы?", options: ["1991", "1993", "1995", "1997"], correct: 1 },
            { question: "Какое метро есть в Алматы?", options: ["Метрополитен", "Подземка", "ЛРТ", "Скоростной трамвай"], correct: 0 },
            { question: "Какой парк является самым большим в Алматы?", options: ["Парк им. 28 гвардейцев", "Центральный парк", "Парк Первого Президента", "Ботанический сад"], correct: 2 },
            { question: "Какая гора видна из центра Алматы?", options: ["Пик Талгар", "Хан-Тенгри", "Большое Алматинское озеро", "Кок-Тобе"], correct: 3 },
            { question: "Сколько районов в Алматы?", options: ["5", "6", "7", "8"], correct: 2 },
            { question: "Какой проспект считается главной магистралью Алматы?", options: ["пр. Достык", "пр. Абая", "пр. Райымбека", "пр. Сейфуллина"], correct: 1 }
        ];

        let currentQuestionIndex = 0;
        let score = 0;
        let quizAnswered = false;

        let map;
        let markers = [];

        // ========== ИНИЦИАЛИЗАЦИЯ КАРТЫ ==========
        function initMap() {
            map = L.map('almatyMap').setView([43.2567, 76.9286], 13);
            L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
                attribution: 'Умный Алматы | Карта города',
                subdomains: 'abcd',
                maxZoom: 18
            }).addTo(map);
            updateMapMarkers();
        }

        function getColorByCongestion(congestion) {
            if (congestion >= 80) return '#ef4444';
            if (congestion >= 60) return '#f97316';
            if (congestion >= 30) return '#fbbf24';
            return '#4ade80';
        }

        function updateMapMarkers() {
            markers.forEach(m => map.removeLayer(m));
            markers = [];
            almatyStreets.forEach(street => {
                const color = getColorByCongestion(street.congestion);
                const radius = 10 + (street.congestion / 100) * 12;
                const marker = L.circleMarker([street.lat, street.lng], {
                    radius: radius,
                    fillColor: color,
                    color: 'white',
                    weight: 2,
                    fillOpacity: 0.85
                }).bindPopup(`<b>${street.name}</b><br>🚗 ${street.congestion}% загруженность`).addTo(map);
                markers.push(marker);
            });
        }

        // ========== ОБНОВЛЕНИЕ ДАШБОРДА ==========
        function updateDashboard() {
            const avgCong = almatyStreets.reduce((s, a) => s + a.congestion, 0) / almatyStreets.length;
            const avgSpd = Math.max(20, 65 - avgCong * 0.45);
            const totalInc = almatyStreets.reduce((s, a) => s + a.incidents, 0);
            
            document.getElementById('avgCongestion').innerHTML = avgCong.toFixed(1);
            document.getElementById('avgSpeed').innerHTML = avgSpd.toFixed(0);
            document.getElementById('totalIncidents').innerHTML = totalInc;
            document.getElementById('aqi').innerHTML = 55 + Math.floor(Math.random() * 50);
            document.getElementById('pm25').innerHTML = 20 + Math.floor(Math.random() * 30);
            document.getElementById('temperature').innerHTML = 15 + Math.floor(Math.random() * 15);
            
            const streetList = document.getElementById('streetList');
            streetList.innerHTML = almatyStreets.sort((a,b) => b.congestion - a.congestion).map(s => `
                <div class="street-item street-${s.status}" onclick="askAboutStreet('${s.name}')">
                    <span><strong>${s.name}</strong></span>
                    <span>${s.congestion}% ${s.status === 'red' ? '🔴' : s.status === 'orange' ? '🟠' : s.status === 'yellow' ? '🟡' : '🟢'}</span>
                </div>
            `).join('');
            
            const incidentList = document.getElementById('incidentList');
            const incidents = almatyStreets.filter(s => s.incidents > 0);
            if (incidents.length === 0) {
                incidentList.innerHTML = '<div style="padding:20px; text-align:center; color:#4ade80;">✅ Происшествий нет</div>';
            } else {
                incidentList.innerHTML = incidents.map(s => `<div class="street-item" style="background:rgba(239,68,68,0.15);"><strong>⚠️ ${s.name}</strong><br>${s.incidents} происшествий</div>`).join('');
            }
            
            const statusBadge = document.getElementById('statusBadge');
            if (avgCong > 75) statusBadge.className = 'status-badge status-critical';
            else if (avgCong > 60) statusBadge.className = 'status-badge status-warning';
            else statusBadge.className = 'status-badge status-normal';
        }

        // ========== AI ПОМОЩНИК ==========
        function sendChatMessage() {
            const input = document.getElementById('chatInput');
            const message = input.value.trim();
            if (!message) return;
            
            addMessage(message, 'user');
            input.value = '';
            
            setTimeout(() => {
                const response = generateAIResponse(message);
                addMessage(response, 'ai');
            }, 500);
        }

        function addMessage(text, sender) {
            const messagesDiv = document.getElementById('chatMessages');
            const msgDiv = document.createElement('div');
            msgDiv.className = `message ${sender}-message`;
            msgDiv.innerHTML = `<div class="msg-text">${text}</div>`;
            messagesDiv.appendChild(msgDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        function generateAIResponse(question) {
            const lowerQ = question.toLowerCase();
            const foundStreet = almatyStreets.find(s => lowerQ.includes(s.name.toLowerCase()));
            
            if (foundStreet) {
                return `📊 <strong>Анализ улицы ${foundStreet.name}</strong><br><br>🚗 Загруженность: ${foundStreet.congestion}%<br>🚨 Инцидентов: ${foundStreet.incidents}<br>${foundStreet.congestion >= 80 ? '🔴 КРАСНАЯ ДОРОГА - объезд обязателен!' : foundStreet.congestion >= 60 ? '🟠 СИЛЬНЫЕ ЗАТОРЫ' : foundStreet.congestion >= 30 ? '🟡 УМЕРЕННОЕ ДВИЖЕНИЕ' : '🟢 СВОБОДНО'}<br><br>💡 ${foundStreet.congestion >= 70 ? 'Рекомендуется объезд' : 'Дорога свободна, приятной поездки!'}`;
            }
            if (lowerQ.includes('пробк') || lowerQ.includes('загруженн')) {
                const avg = almatyStreets.reduce((s,a)=>s+a.congestion,0)/almatyStreets.length;
                return `🚦 <strong>Ситуация на дорогах Алматы</strong><br><br>📊 Средняя загруженность: ${avg.toFixed(1)}%<br>🔴 Красных улиц: ${almatyStreets.filter(s=>s.congestion>=80).length}<br>💡 ${avg>70?'Рекомендуется использовать общественный транспорт':'Дороги свободны'}`;
            }
            if (lowerQ.includes('экологи') || lowerQ.includes('воздух')) {
                return `🌿 <strong>Экологическая обстановка</strong><br><br>🌡️ AQI: ${55+Math.floor(Math.random()*50)}<br>💨 PM2.5: ${20+Math.floor(Math.random()*30)} µg/m³<br>${Math.random()>0.5?'Качество воздуха хорошее':'Рекомендуется ограничить прогулки'}`;
            }
            return `🤖 <strong>Я AI-помощник Алматы</strong><br><br>Я могу ответить на вопросы о:<br>• Пробках и загруженности улиц<br>• Конкретных улицах (например, "Абая")<br>• Экологической ситуации<br>• Прогнозе на день<br><br>Что вас интересует?`;
        }

        function askAboutStreet(streetName) {
            document.getElementById('chatInput').value = streetName;
            switchTab('ai');
            setTimeout(() => sendChatMessage(), 100);
        }

        // ========== AI ПРОГНОЗЫ ==========
        function updateForecasts() {
            const hour = new Date().getHours();
            const avgCong = almatyStreets.reduce((s,a)=>s+a.congestion,0)/almatyStreets.length;
            
            document.getElementById('dailyForecast').innerHTML = `
                📈 <strong>Прогноз на ${moment().format('DD.MM.YYYY')}</strong><br><br>
                🕐 Сейчас: ${hour}:00<br>
                🚗 Текущая загруженность: ${avgCong.toFixed(1)}%<br>
                ${hour >= 8 && hour <= 10 ? '🔴 УТРЕННИЙ ЧАС ПИК' : hour >= 17 && hour <= 19 ? '🔴 ВЕЧЕРНИЙ ЧАС ПИК' : '🟢 МЕЖПИКОВЫЙ ПЕРИОД'}<br>
                📊 Пик загруженности: 18:00-19:30<br>
                🌙 Ночью: дороги свободны
            `;
            
            const hourly = [];
            for (let i = 0; i < 8; i++) {
                let h = (hour + i) % 24;
                let cong = 30;
                if ((h >= 8 && h <= 10) || (h >= 17 && h <= 19)) cong = 75 + Math.random() * 15;
                else if (h >= 22 || h <= 5) cong = 15 + Math.random() * 15;
                else cong = 45 + Math.random() * 20;
                hourly.push(`<div class="hour-item"><strong>${h}:00</strong><br>${cong.toFixed(0)}%</div>`);
            }
            document.getElementById('hourlyForecast').innerHTML = hourly.join('');
            
            document.getElementById('weatherForecast').innerHTML = `
                🌡️ Температура: +${15+Math.floor(Math.random()*15)}°C<br>
                💨 Ветер: ${2+Math.floor(Math.random()*8)} м/с<br>
                ☁️ Облачность: ${['Ясно', 'Малооблачно', 'Облачно', 'Пасмурно'][Math.floor(Math.random()*4)]}<br>
                🌧️ Осадки: ${Math.random()>0.7?'Возможны':'Без осадков'}
            `;
            
            document.getElementById('aiRecommendations').innerHTML = `
                💡 <strong>Рекомендации на сегодня:</strong><br><br>
                ${avgCong > 70 ? '• Используйте метро или автобусы' : '• Дороги свободны, приятной поездки!'}<br>
                • Планируйте маршруты заранее<br>
                • Следите за обновлениями на карте
            `;
        }

        // ========== ВИКТОРИНА ==========
        function loadQuestion() {
            const q = quizQuestions[currentQuestionIndex];
            document.getElementById('quizQuestion').innerHTML = q.question;
            document.getElementById('quizScore').innerHTML = `Счет: ${score} / ${quizQuestions.length}`;
            
            const optionsDiv = document.getElementById('quizOptions');
            optionsDiv.innerHTML = '';
            quizAnswered = false;
            
            q.options.forEach((opt, idx) => {
                const optDiv = document.createElement('div');
                optDiv.className = 'quiz-option';
                optDiv.innerHTML = opt;
                optDiv.onclick = () => checkAnswer(idx);
                optionsDiv.appendChild(optDiv);
            });
        }

        function checkAnswer(selectedIndex) {
            if (quizAnswered) return;
            quizAnswered = true;
            const q = quizQuestions[currentQuestionIndex];
            const options = document.querySelectorAll('.quiz-option');
            
            if (selectedIndex === q.correct) {
                score++;
                options[selectedIndex].classList.add('correct');
            } else {
                options[selectedIndex].classList.add('wrong');
                options[q.correct].classList.add('correct');
            }
            document.getElementById('quizScore').innerHTML = `Счет: ${score} / ${quizQuestions.length}`;
        }

        function nextQuestion() {
            if (!quizAnswered && currentQuestionIndex < quizQuestions.length - 1) {
                alert('Пожалуйста, ответьте на вопрос!');
                return;
            }
            currentQuestionIndex++;
            if (currentQuestionIndex < quizQuestions.length) {
                loadQuestion();
            } else {
                document.getElementById('quizContainer').innerHTML = `
                    <div style="text-align:center; padding:40px;">
                        <div style="font-size:48px;">🎉</div>
                        <div style="font-size:28px; margin:20px 0;">Викторина завершена!</div>
                        <div style="font-size:20px;">Ваш счет: ${score} из ${quizQuestions.length}</div>
                        <div style="margin-top:20px;">${score === quizQuestions.length ? '🏆 Отлично! Вы знаток Алматы!' : '👍 Хороший результат! Попробуйте еще раз!'}</div>
                        <button class="next-btn" onclick="restartQuiz()" style="margin-top:30px;">Пройти заново</button>
                    </div>
                `;
            }
        }

        function restartQuiz() {
            currentQuestionIndex = 0;
            score = 0;
            quizAnswered = false;
            location.reload();
        }

        // ========== ОБЩИЕ ФУНКЦИИ ==========
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.getElementById(`${tabName}Tab`).classList.add('active');
            document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
            document.querySelector(`[data-tab="${tabName}"]`).classList.add('active');
            
            if (tabName === 'map' && map) setTimeout(() => map.invalidateSize(), 100);
            if (tabName === 'forecast') updateForecasts();
            if (tabName === 'quiz') loadQuestion();
        }

        function refreshAll() {
            almatyStreets.forEach(s => {
                s.congestion = Math.min(95, Math.max(20, s.congestion + (Math.random() - 0.5) * 10));
                if (s.congestion >= 80) s.status = "red";
                else if (s.congestion >= 60) s.status = "orange";
                else if (s.congestion >= 30) s.status = "yellow";
                else s.status = "green";
            });
            updateDashboard();
            updateMapMarkers();
            updateForecasts();
        }

        // Глобальные функции
        window.switchTab = switchTab;
        window.sendChatMessage = sendChatMessage;
        window.askAboutStreet = askAboutStreet;
        window.refreshAll = refreshAll;
        window.nextQuestion = nextQuestion;
        window.restartQuiz = restartQuiz;

        document.getElementById('chatInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') sendChatMessage();
        });

        // Запуск
        window.addEventListener('load', () => {
            initMap();
            updateDashboard();
            updateForecasts();
            setInterval(() => {
                refreshAll();
            }, 15000);
        });
    </script>
</body>
</html>
