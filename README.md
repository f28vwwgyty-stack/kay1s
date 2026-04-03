<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>2GIS Алматы | Smart Dashboard с AI-прогнозами</title>
    <!-- Leaflet для карты (аналог 2GIS) -->
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
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
            background: #0a0f1e;
            color: #e0e4f0;
            overflow: hidden;
            height: 100vh;
        }

        /* App Layout */
        .app {
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* Top Navigation Bar */
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
        }

        .logo h1 {
            font-size: 20px;
            background: linear-gradient(135deg, #3b82f6, #10b981);
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
        }

        .nav-tab {
            padding: 10px 24px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 14px;
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

        .right-controls {
            display: flex;
            gap: 10px;
            align-items: center;
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

        /* Tab Content */
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

        /* Dashboard Grid */
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

        .kpi-label {
            font-size: 11px;
            color: #94a3b8;
            margin-top: 5px;
        }

        .street-list {
            max-height: 300px;
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
            transition: all 0.2s;
        }

        .street-item:hover { transform: translateX(5px); }
        .street-red { background: rgba(239,68,68,0.15); border-left: 3px solid #ef4444; }
        .street-orange { background: rgba(249,115,22,0.15); border-left: 3px solid #f97316; }
        .street-yellow { background: rgba(234,179,8,0.15); border-left: 3px solid #fbbf24; }
        .street-green { background: rgba(34,197,94,0.15); border-left: 3px solid #4ade80; }

        /* AI Chat */
        .ai-chat {
            position: fixed;
            bottom: 20px;
            right: 20px;
            width: 350px;
            height: 450px;
            background: rgba(10, 15, 30, 0.98);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            border: 1px solid rgba(59,130,246,0.4);
            display: flex;
            flex-direction: column;
            z-index: 1001;
            box-shadow: 0 10px 40px rgba(0,0,0,0.5);
        }

        .ai-chat-header {
            padding: 15px 20px;
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            border-radius: 20px 20px 0 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
        }

        .ai-chat-messages {
            flex: 1;
            overflow-y: auto;
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .message {
            max-width: 85%;
            padding: 10px 14px;
            border-radius: 15px;
            font-size: 12px;
            line-height: 1.4;
        }

        .user-msg {
            background: #3b82f6;
            align-self: flex-end;
            border-radius: 15px 15px 0 15px;
        }

        .ai-msg {
            background: rgba(255,255,255,0.1);
            align-self: flex-start;
            border-radius: 15px 15px 15px 0;
        }

        .chat-input-area {
            padding: 15px;
            border-top: 1px solid rgba(59,130,246,0.3);
            display: flex;
            gap: 10px;
        }

        .chat-input {
            flex: 1;
            padding: 10px;
            background: rgba(0,0,0,0.4);
            border: 1px solid rgba(59,130,246,0.3);
            border-radius: 20px;
            color: white;
            outline: none;
        }

        .send-btn {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            border: none;
            padding: 8px 16px;
            border-radius: 20px;
            color: white;
            cursor: pointer;
        }

        /* Map Legend */
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

        .legend-color {
            width: 20px;
            height: 4px;
            border-radius: 2px;
            display: inline-block;
        }

        @media (max-width: 900px) {
            .dashboard-grid { grid-template-columns: 1fr; }
            .ai-chat { width: 300px; height: 400px; }
            .nav-tab { padding: 6px 12px; font-size: 12px; }
        }
    </style>
</head>
<body>
    <div class="app">
        <!-- Top Navigation -->
        <div class="top-nav">
            <div class="nav-container">
                <div class="logo">
                    <h1>🏙️ 2GIS Алматы | Smart Dashboard</h1>
                    <p>AI-прогнозы | Пробки в реальном времени</p>
                </div>
                <div class="nav-tabs">
                    <div class="nav-tab active" data-tab="dashboard" onclick="switchTab('dashboard')">📊 Дашборд</div>
                    <div class="nav-tab" data-tab="map" onclick="switchTab('map')">🗺️ 2GIS Карта</div>
                    <div class="nav-tab" data-tab="ai" onclick="switchTab('ai')">🤖 AI-прогнозы</div>
                    <div class="nav-tab" data-tab="settings" onclick="switchTab('settings')">⚙️ Настройки</div>
                </div>
                <div class="right-controls">
                    <div class="status-badge" id="statusBadge">🟢 НОРМА</div>
                    <button class="refresh-btn" onclick="refreshAll()">🔄 Обновить</button>
                </div>
            </div>
        </div>

        <div class="main-content">
            <!-- Dashboard Tab -->
            <div class="tab-content active" id="dashboardTab">
                <div class="dashboard-grid">
                    <!-- Транспортный блок -->
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>🚗</span> Транспортная ситуация
                        </div>
                        <div class="kpi-row">
                            <div class="kpi-box">
                                <div class="kpi-value" id="avgCongestion">0<span style="font-size:14px;">%</span></div>
                                <div class="kpi-label">Средняя загруженность</div>
                            </div>
                            <div class="kpi-box">
                                <div class="kpi-value" id="avgSpeed">0<span style="font-size:14px;">км/ч</span></div>
                                <div class="kpi-label">Средняя скорость</div>
                            </div>
                            <div class="kpi-box">
                                <div class="kpi-value" id="totalIncidents">0</div>
                                <div class="kpi-label">Инцидентов за час</div>
                            </div>
                        </div>
                        <div class="chart-container">
                            <canvas id="trafficChart" style="height: 200px;"></canvas>
                        </div>
                    </div>

                    <!-- Экологический блок -->
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>🌿</span> Экологическая обстановка
                        </div>
                        <div class="kpi-row">
                            <div class="kpi-box">
                                <div class="kpi-value" id="aqi">0</div>
                                <div class="kpi-label">AQI (качество воздуха)</div>
                            </div>
                            <div class="kpi-box">
                                <div class="kpi-value" id="pm25">0<span style="font-size:14px;">μg</span></div>
                                <div class="kpi-label">PM2.5</div>
                            </div>
                            <div class="kpi-box">
                                <div class="kpi-value" id="noise">0<span style="font-size:14px;">дБ</span></div>
                                <div class="kpi-label">Уровень шума</div>
                            </div>
                        </div>
                        <div class="chart-container">
                            <canvas id="ecologyChart" style="height: 200px;"></canvas>
                        </div>
                    </div>

                    <!-- Проблемные улицы -->
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>🚦</span> Загруженность улиц (по данным 2GIS)
                        </div>
                        <div class="street-list" id="streetList">
                            Загрузка...
                        </div>
                    </div>

                    <!-- AI-прогноз -->
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>🤖</span> AI-прогноз на ближайшие часы
                        </div>
                        <div id="aiForecast" style="line-height: 1.6;">
                            Анализ данных...
                        </div>
                        <div style="margin-top: 15px;">
                            <strong>📋 Рекомендации:</strong>
                            <ul id="recommendations" style="margin-top: 10px; list-style: none;">
                                <li>⏳ Загрузка...</li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Map Tab - 2GIS Карта -->
            <div class="tab-content" id="mapTab">
                <div class="map-container">
                    <div id="almatyMap"></div>
                    <div class="map-legend">
                        <div><span class="legend-color" style="background: #4ade80;"></span> 🟢 Свободно (0-30%)</div>
                        <div><span class="legend-color" style="background: #fbbf24;"></span> 🟡 Загружено (30-60%)</div>
                        <div><span class="legend-color" style="background: #f97316;"></span> 🟠 Пробки (60-80%)</div>
                        <div><span class="legend-color" style="background: #ef4444;"></span> 🔴 Красная дорога (80-100%)</div>
                    </div>
                </div>
            </div>

            <!-- AI Прогнозы Tab -->
            <div class="tab-content" id="aiTab">
                <div class="dashboard-grid" style="grid-template-columns: 1fr;">
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>🧠</span> Детальный AI-анализ Алматы
                        </div>
                        <div id="detailedAnalysis" style="line-height: 1.8;">
                            Загрузка анализа...
                        </div>
                    </div>
                    <div class="dashboard-card">
                        <div class="card-title">
                            <span>📈</span> Прогноз по районам
                        </div>
                        <div id="districtForecast" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px,1fr)); gap: 15px;">
                            Загрузка...
                        </div>
                    </div>
                </div>
            </div>

            <!-- Settings Tab -->
            <div class="tab-content" id="settingsTab">
                <div class="dashboard-grid" style="grid-template-columns: 1fr 1fr;">
                    <div class="dashboard-card">
                        <div class="card-title">⚙️ Общие настройки</div>
                        <div style="margin: 15px 0;">
                            <label style="display: flex; justify-content: space-between; margin-bottom: 15px;">
                                <span>🌍 Слой карты</span>
                                <select id="mapLayer" onchange="changeMapLayer()" style="background: #1a1f2e; color: white; padding: 5px 10px; border-radius: 8px;">
                                    <option value="dark">Темный</option>
                                    <option value="light">Светлый</option>
                                    <option value="satellite">Спутник</option>
                                </select>
                            </label>
                            <label style="display: flex; justify-content: space-between; margin-bottom: 15px;">
                                <span>🚦 Показывать пробки на карте</span>
                                <div class="toggle-switch" id="trafficToggle" onclick="toggleTraffic()" style="width: 50px; height: 26px; background: #3b82f6; border-radius: 13px; cursor: pointer; position: relative;"><div style="position: absolute; width: 22px; height: 22px; background: white; border-radius: 50%; top: 2px; right: 3px;"></div></div>
                            </label>
                            <label style="display: flex; justify-content: space-between; margin-bottom: 15px;">
                                <span>🔔 Уведомления о пробках</span>
                                <div class="toggle-switch" id="notifToggle" onclick="toggleNotifications()" style="width: 50px; height: 26px; background: rgba(255,255,255,0.2); border-radius: 13px; cursor: pointer; position: relative;"><div style="position: absolute; width: 22px; height: 22px; background: white; border-radius: 50%; top: 2px; left: 3px;"></div></div>
                            </label>
                            <label style="display: flex; justify-content: space-between; margin-bottom: 15px;">
                                <span>📊 Интервал обновления</span>
                                <select id="refreshInterval" style="background: #1a1f2e; color: white; padding: 5px 10px; border-radius: 8px;">
                                    <option value="5">5 секунд</option>
                                    <option value="10" selected>10 секунд</option>
                                    <option value="30">30 секунд</option>
                                </select>
                            </label>
                        </div>
                    </div>
                    <div class="dashboard-card">
                        <div class="card-title">ℹ️ О системе</div>
                        <div style="margin: 15px 0; line-height: 1.6;">
                            <p><strong>Smart Dashboard Алматы</strong></p>
                            <p>Версия: 2.0</p>
                            <p>Данные в реальном времени на основе AI-анализа</p>
                            <p>Источники: 2GIS API, сенсоры города, AI-прогноз</p>
                            <hr style="margin: 15px 0; border-color: rgba(59,130,246,0.3);">
                            <p>📞 Поддержка: support@smartalmaty.kz</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- AI Chat Assistant -->
        <div class="ai-chat" id="aiChat">
            <div class="ai-chat-header" onclick="toggleChat()">
                <div style="display: flex; align-items: center; gap: 8px;">
                    <span>🤖</span>
                    <span>AI-помощник Алматы</span>
                </div>
                <span>▼</span>
            </div>
            <div class="ai-chat-messages" id="chatMessages">
                <div class="message ai-msg">
                    👋 Здравствуйте! Я AI-помощник по городу Алматы.<br>
                    Спросите меня о пробках, экологии, прогнозе или любой улице!
                </div>
            </div>
            <div class="chat-input-area">
                <input type="text" class="chat-input" id="chatInput" placeholder="Например: пробки на Достык, прогноз на сегодня...">
                <button class="send-btn" onclick="sendChatMessage()">➤</button>
            </div>
        </div>
    </div>

    <script>
        let map;
        let roadLayers = [];
        let trafficChart, ecologyChart;
        let updateInterval;
        let trafficVisible = true;
        let currentTab = 'dashboard';

        // Данные по районам Алматы
        const almatyDistricts = [
            { name: 'Медеуский район', congestion: 72, aqi: 118, incidents: 3, forecast: 'Ожидается ухудшение в вечерний час пик' },
            { name: 'Алмалинский район', congestion: 85, aqi: 145, incidents: 5, forecast: 'Критическая ситуация до 20:00' },
            { name: 'Ауэзовский район', congestion: 68, aqi: 108, incidents: 2, forecast: 'Стабильная ситуация' },
            { name: 'Бостандыкский район', congestion: 78, aqi: 125, incidents: 4, forecast: 'Пробки на пр. Абая до 19:30' },
            { name: 'Жетысуский район', congestion: 58, aqi: 95, incidents: 1, forecast: 'Свободно' },
            { name: 'Наурызбайский район', congestion: 45, aqi: 82, incidents: 0, forecast: 'Отличные условия' },
            { name: 'Турксибский район', congestion: 62, aqi: 105, incidents: 2, forecast: 'Умеренное движение' }
        ];

        // Детальные улицы Алматы (как в 2GIS)
        const almatyStreets = [
            { name: 'пр. Достык', congestion: 78, incidents: 2, status: 'orange', length: 4.5 },
            { name: 'ул. Толе би', congestion: 82, incidents: 3, status: 'red', length: 3.8 },
            { name: 'пр. Абая', congestion: 88, incidents: 4, status: 'red', length: 5.2 },
            { name: 'ул. Сатпаева', congestion: 68, incidents: 1, status: 'yellow', length: 3.2 },
            { name: 'ул. Жандосова', congestion: 55, incidents: 0, status: 'green', length: 3.0 },
            { name: 'пр. Райымбека', congestion: 85, incidents: 5, status: 'red', length: 4.0 },
            { name: 'ул. Гагарина', congestion: 72, incidents: 1, status: 'orange', length: 2.8 },
            { name: 'ул. Фурманова', congestion: 65, incidents: 1, status: 'yellow', length: 2.2 },
            { name: 'ул. Панфилова', congestion: 70, incidents: 1, status: 'orange', length: 2.5 },
            { name: 'пр. Сейфуллина', congestion: 75, incidents: 2, status: 'orange', length: 3.5 },
            { name: 'ул. Жибек Жолы', congestion: 80, incidents: 2, status: 'red', length: 2.0 },
            { name: 'пр. Назарбаева', congestion: 62, incidents: 0, status: 'yellow', length: 2.3 }
        ];

        // Координаты ключевых точек Алматы для карты
        const almatyPoints = [
            { name: 'Центр (пл. Республики)', lat: 43.2567, lng: 76.9286, congestion: 85 },
            { name: 'Медey', lat: 43.158, lng: 76.955, congestion: 55 },
            { name: 'Абая/Достык', lat: 43.245, lng: 76.915, congestion: 82 },
            { name: 'Райымбека пр.', lat: 43.270, lng: 76.925, congestion: 88 },
            { name: 'Саина/Толе би', lat: 43.235, lng: 76.890, congestion: 75 },
            { name: 'Атакент', lat: 43.225, lng: 76.920, congestion: 68 },
            { name: 'Москва', lat: 43.240, lng: 76.950, congestion: 70 },
            { name: 'Горный Гигант', lat: 43.200, lng: 76.860, congestion: 52 }
        ];

        function initMap() {
            map = L.map('almatyMap').setView([43.2567, 76.9286], 13);
            
            L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
                attribution: '2GIS Алматы | Карта дорог и пробок',
                subdomains: 'abcd',
                maxZoom: 18
            }).addTo(map);
            
            addMarkers();
            if (trafficVisible) addRoads();
        }

        function addMarkers() {
            almatyPoints.forEach(point => {
                let color = '#4ade80';
                if (point.congestion >= 80) color = '#ef4444';
                else if (point.congestion >= 60) color = '#f97316';
                else if (point.congestion >= 30) color = '#fbbf24';
                
                const radius = 15 + (point.congestion / 100) * 10;
                
                const marker = L.circleMarker([point.lat, point.lng], {
                    radius: radius,
                    fillColor: color,
                    color: 'white',
                    weight: 2,
                    fillOpacity: 0.8
                }).bindPopup(`
                    <div style="background:#1a1f2e; padding:12px; border-radius:12px;">
                        <b>📍 ${point.name}</b><br>
                        🚗 Загруженность: ${point.congestion}%<br>
                        ${point.congestion >= 80 ? '🔴 КРАСНАЯ ЗОНА - ОБЪЕЗД РЕКОМЕНДОВАН' : point.congestion >= 60 ? '🟠 ЗАТОРЫ' : point.congestion >= 30 ? '🟡 УМЕРЕННО' : '🟢 СВОБОДНО'}
                    </div>
                `).addTo(map);
            });
        }

        function addRoads() {
            roadLayers.forEach(l => map.removeLayer(l));
            roadLayers = [];
            
            almatyStreets.forEach(street => {
                // Симулируем координаты для улиц (в реальном API были бы реальные координаты)
                const baseLat = 43.24;
                const baseLng = 76.90;
                const coords = [];
                for (let i = 0; i < 5; i++) {
                    coords.push([baseLat + (Math.random() - 0.5) * 0.05, baseLng + (Math.random() - 0.5) * 0.05]);
                }
                
                let color = '#4ade80';
                if (street.congestion >= 80) color = '#ef4444';
                else if (street.congestion >= 60) color = '#f97316';
                else if (street.congestion >= 30) color = '#fbbf24';
                
                const line = L.polyline(coords, {
                    color: color,
                    weight: 4,
                    opacity: 0.8
                }).bindPopup(`
                    <div style="background:#1a1f2e; padding:12px; border-radius:12px;">
                        <b>🛣️ ${street.name}</b><br>
                        🚗 Загруженность: ${street.congestion}%<br>
                        🚨 Инцидентов: ${street.incidents}<br>
                        📏 Длина: ${street.length} км<br>
                        <button onclick="askAboutStreet('${street.name}')" style="background:#3b82f6; border:none; color:white; padding:5px 10px; border-radius:8px; margin-top:8px; cursor:pointer;">🤖 AI-анализ</button>
                    </div>
                `);
                roadLayers.push(line);
                line.addTo(map);
            });
        }

        function generateLiveData() {
            const hour = new Date().getHours();
            const isPeakHour = (hour >= 8 && hour <= 10) || (hour >= 17 && hour <= 19);
            
            almatyStreets.forEach(street => {
                let change = (Math.random() - 0.5) * 10;
                street.congestion = Math.min(98, Math.max(20, street.congestion + change));
                if (isPeakHour) street.congestion += 8;
                if (Math.random() < 0.08) street.incidents = Math.min(5, street.incidents + 1);
                else if (Math.random() < 0.1) street.incidents = Math.max(0, street.incidents - 1);
                
                if (street.congestion >= 80) street.status = 'red';
                else if (street.congestion >= 60) street.status = 'orange';
                else if (street.congestion >= 30) street.status = 'yellow';
                else street.status = 'green';
            });
            
            almatyDistricts.forEach(district => {
                district.congestion = Math.min(95, Math.max(30, district.congestion + (Math.random() - 0.5) * 8));
            });
        }

        function updateDashboard() {
            generateLiveData();
            
            const avgCong = almatyStreets.reduce((sum, s) => sum + s.congestion, 0) / almatyStreets.length;
            const avgSpd = Math.max(20, 65 - avgCong * 0.35);
            const totalInc = almatyStreets.reduce((sum, s) => sum + s.incidents, 0);
            const maxAqi = Math.max(...almatyDistricts.map(d => d.aqi));
            
            document.getElementById('avgCongestion').innerHTML = avgCong.toFixed(1);
            document.getElementById('avgSpeed').innerHTML = avgSpd.toFixed(0);
            document.getElementById('totalIncidents').innerHTML = totalInc;
            document.getElementById('aqi').innerHTML = maxAqi;
            document.getElementById('pm25').innerHTML = (maxAqi * 0.55).toFixed(0);
            document.getElementById('noise').innerHTML = (55 + Math.random() * 15).toFixed(0);
            
            const statusBadge = document.getElementById('statusBadge');
            if (avgCong > 75) {
                statusBadge.innerHTML = '🔴 КРИТИЧЕСКИЕ ПРОБКИ';
                statusBadge.className = 'status-badge status-critical';
            } else if (avgCong > 60) {
                statusBadge.innerHTML = '🟡 ЗАТОРЫ НА ДОРОГАХ';
                statusBadge.className = 'status-badge status-warning';
            } else {
                statusBadge.innerHTML = '🟢 ДОРОГИ СВОБОДНЫ';
                statusBadge.className = 'status-badge status-normal';
            }
            
            // Update street list
            const streetList = document.getElementById('streetList');
            streetList.innerHTML = almatyStreets.sort((a,b) => b.congestion - a.congestion).slice(0, 8).map(s => `
                <div class="street-item street-${s.status}" onclick="askAboutStreet('${s.name}')">
                    <span><strong>${s.name}</strong></span>
                    <span style="font-weight: bold;">${s.congestion}% ${s.status === 'red' ? '🔴' : s.status === 'orange' ? '🟠' : s.status === 'yellow' ? '🟡' : '🟢'}</span>
                </div>
            `).join('');
            
            // AI Forecast
            const worstHour = avgCong > 70 ? '19:00-20:00' : '18:00-19:00';
            const forecast = `📊 <strong>Анализ на ${moment().format('HH:mm')}</strong><br><br>
            ${avgCong > 75 ? '🔴 КРИТИЧЕСКАЯ СИТУАЦИЯ: Ожидаются крупные пробки на пр. Абая, пр. Райымбека, ул. Толе би.' : 
              avgCong > 60 ? '🟠 ВНИМАНИЕ: Заторы на основных магистралях. Рекомендуется объезд.' :
              '🟢 ШТАТНЫЙ РЕЖИМ: Дороги свободны.'}<br><br>
            <strong>⏱️ Прогноз на ближайшие часы:</strong><br>
            • Пик загруженности ожидается в ${worstHour}<br>
            • К ${avgCong > 70 ? '21:00' : '19:30'} ситуация нормализуется<br>
            • Рекомендуется избегать центр в часы пик`;
            
            document.getElementById('aiForecast').innerHTML = forecast;
            
            const recommendations = document.getElementById('recommendations');
            recommendations.innerHTML = almatyStreets.filter(s => s.congestion > 75).slice(0, 3).map(s => 
                `<li>🚦 ${s.name}: ${s.congestion}% - рекомендуется объезд</li>`
            ).join('');
            if (almatyStreets.filter(s => s.congestion > 75).length === 0) {
                recommendations.innerHTML = '<li>✅ Все улицы свободны, приятной поездки!</li>';
            }
            
            // Charts
            const hours = Array.from({length: 12}, (_, i) => `${i+8}:00`);
            const trafficData = Array.from({length: 12}, () => 40 + Math.random() * 40);
            
            if (trafficChart) {
                trafficChart.data.datasets[0].data = trafficData;
                trafficChart.update();
            } else {
                const ctx = document.getElementById('trafficChart').getContext('2d');
                trafficChart = new Chart(ctx, {
                    type: 'line',
                    data: { labels: hours, datasets: [{ label: 'Загруженность (%)', data: trafficData, borderColor: '#3b82f6', fill: true, backgroundColor: 'rgba(59,130,246,0.1)' }] },
                    options: { responsive: true, maintainAspectRatio: true, plugins: { legend: { labels: { color: 'white' } } }, scales: { y: { ticks: { color: 'white' } }, x: { ticks: { color: 'white' } } } }
                });
            }
            
            const ecologyData = Array.from({length: 12}, () => 50 + Math.random() * 70);
            if (ecologyChart) {
                ecologyChart.data.datasets[0].data = ecologyData;
                ecologyChart.update();
            } else {
                const ctx = document.getElementById('ecologyChart').getContext('2d');
                ecologyChart = new Chart(ctx, {
                    type: 'line',
                    data: { labels: hours, datasets: [{ label: 'AQI', data: ecologyData, borderColor: '#10b981', fill: true, backgroundColor: 'rgba(16,185,129,0.1)' }] },
                    options: { responsive: true, maintainAspectRatio: true, plugins: { legend: { labels: { color: 'white' } } }, scales: { y: { ticks: { color: 'white' } }, x: { ticks: { color: 'white' } } } }
                });
            }
            
            // Detailed analysis for AI tab
            const detailedAnalysis = document.getElementById('detailedAnalysis');
            if (detailedAnalysis) {
                detailedAnalysis.innerHTML = `
                    <p><strong>🧠 Детальный AI-анализ города Алматы</strong></p><br>
                    <p><strong>📊 Общая ситуация:</strong> Средняя загруженность ${avgCong.toFixed(1)}%. ${avgCong > 70 ? 'Наблюдаются масштабные затруднения движения.' : 'Движение в обычном режиме.'}</p>
                    <p><strong>🔴 Красные зоны:</strong> ${almatyStreets.filter(s => s.congestion >= 80).map(s => s.name).join(', ') || 'Отсутствуют'}</p>
                    <p><strong>🟠 Оранжевые зоны:</strong> ${almatyStreets.filter(s => s.congestion >= 60 && s.congestion < 80).map(s => s.name).join(', ') || 'Отсутствуют'}</p>
                    <p><strong>🌿 Экология:</strong> AQI ${maxAqi} - ${maxAqi > 100 ? '⚠️ Нездоровый уровень, рекомендуется ограничить пребывание на улице' : '✅ Нормальный уровень'}</p>
                    <p><strong>🚨 Инциденты:</strong> ${totalInc} ДТП за последний час</p>
                    <p><strong>💡 Рекомендации:</strong> ${avgCong > 70 ? 'Используйте общественный транспорт или планируйте маршруты в объезд центра.' : 'Штатный режим, соблюдайте ПДД.'}</p>
                `;
            }
            
            // District forecast
            const districtForecast = document.getElementById('districtForecast');
            if (districtForecast) {
                districtForecast.innerHTML = almatyDistricts.map(d => `
                    <div style="background:rgba(0,0,0,0.3); padding:12px; border-radius:12px;">
                        <strong>🏘️ ${d.name}</strong><br>
                        🚗 ${d.congestion}% загруженность<br>
                        🌿 AQI ${d.aqi}<br>
                        📈 ${d.forecast}
                    </div>
                `).join('');
            }
            
            if (trafficVisible && map) {
                roadLayers.forEach(l => map.removeLayer(l));
                addRoads();
            }
        }

        function askAboutStreet(streetName) {
            const street = almatyStreets.find(s => s.name === streetName);
            if (street) {
                openChat();
                const analysis = `📊 <b>AI-анализ улицы ${street.name}</b><br><br>
                🚗 Текущая загруженность: ${street.congestion}%<br>
                🚨 Инцидентов: ${street.incidents}<br>
                📏 Протяженность: ${street.length} км<br><br>
                ${street.congestion >= 80 ? '🔴 КРАСНАЯ ДОРОГА: Движение практически стоит. Рекомендуется объезд через параллельные улицы.' :
                  street.congestion >= 60 ? '🟠 СИЛЬНЫЕ ЗАТОРЫ: Время в пути увеличено на 20-30 минут.' :
                  street.congestion >= 30 ? '🟡 УМЕРЕННОЕ ДВИЖЕНИЕ: Возможны небольшие задержки.' :
                  '🟢 СВОБОДНО: Отличные условия для движения.'}<br><br>
                💡 <b>Прогноз:</b> ${street.congestion >= 70 ? 'Ожидается ухудшение в ближайший час.' : 'Ситуация стабильна.'}`;
                addAIMessage(analysis);
            }
        }

        function openChat() {
            document.getElementById('aiChat').style.display = 'flex';
        }

        function toggleChat() {
            const chat = document.getElementById('aiChat');
            if (chat.style.display === 'none') chat.style.display = 'flex';
            else chat.style.display = 'none';
        }

        function sendChatMessage() {
            const input = document.getElementById('chatInput');
            const message = input.value.trim();
            if (!message) return;
            
            addUserMessage(message);
            input.value = '';
            
            setTimeout(() => processAIChat(message), 500);
        }

        function addUserMessage(text) {
            const messagesDiv = document.getElementById('chatMessages');
            const msgDiv = document.createElement('div');
            msgDiv.className = 'message user-msg';
            msgDiv.innerHTML = text;
            messagesDiv.appendChild(msgDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        function addAIMessage(text) {
            const messagesDiv = document.getElementById('chatMessages');
            const msgDiv = document.createElement('div');
            msgDiv.className = 'message ai-msg';
            msgDiv.innerHTML = text;
            messagesDiv.appendChild(msgDiv);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        }

        function processAIChat(message) {
            const lowerMsg = message.toLowerCase();
            let response = '';
            
            const foundStreet = almatyStreets.find(s => lowerMsg.includes(s.name.toLowerCase()));
            const foundDistrict = almatyDistricts.find(d => lowerMsg.includes(d.name.toLowerCase()));
            
            if (foundStreet) {
                response = `📊 <b>Анализ ${foundStreet.name}</b><br><br>🚗 Загруженность: ${foundStreet.congestion}%<br>🚨 Инцидентов: ${foundStreet.incidents}<br>${foundStreet.congestion >= 80 ? '🔴 КРАСНАЯ ДОРОГА - объезд обязателен!' : foundStreet.congestion >= 60 ? '🟠 Ожидайте задержки 20-30 мин' : '✅ Дорога свободна'}`;
            } 
            else if (foundDistrict) {
                response = `🏘️ <b>Район ${foundDistrict.name}</b><br><br>🚗 Загруженность: ${foundDistrict.congestion}%<br>🌿 AQI: ${foundDistrict.aqi}<br>📈 Прогноз: ${foundDistrict.forecast}`;
            }
            else if (lowerMsg.includes('пробка') || lowerMsg.includes('загруженность')) {
                const worst = almatyStreets.sort((a,b) => b.congestion - a.congestion)[0];
                response = `🚦 <b>Самая загруженная улица:</b> ${worst.name} (${worst.congestion}%)<br><br>📊 <b>Общая статистика:</b><br>• Средняя загруженность: ${(almatyStreets.reduce((s,str)=>s+str.congestion,0)/almatyStreets.length).toFixed(1)}%<br>• Красных улиц: ${almatyStreets.filter(s=>s.congestion>=80).length}<br>• Оранжевых улиц: ${almatyStreets.filter(s=>s.congestion>=60 && s.congestion<80).length}`;
            }
            else if (lowerMsg.includes('экология') || lowerMsg.includes('aqi') || lowerMsg.includes('воздух')) {
                const worst = almatyDistricts.sort((a,b) => b.aqi - a.aqi)[0];
                response = `🌿 <b>Экологическая ситуация в Алматы</b><br><br>• Худший район: ${worst.name} (AQI ${worst.aqi})<br>• Средний AQI: ${(almatyDistricts.reduce((s,d)=>s+d.aqi,0)/almatyDistricts.length).toFixed(0)}<br><br>${worst.aqi > 100 ? '⚠️ Рекомендуется ограничить пребывание на улице' : '✅ Воздух в норме'}`;
            }
            else if (lowerMsg.includes('прогноз') || lowerMsg.includes('что будет')) {
                const hour = new Date().getHours();
                response = `📈 <b>AI-прогноз на сегодня</b><br><br>• Сейчас: ${hour >= 8 && hour <= 10 ? 'УТРЕННИЙ ЧАС ПИК' : hour >= 17 && hour <= 19 ? 'ВЕЧЕРНИЙ ЧАС ПИК' : 'МЕЖПИКОВЫЙ ПЕРИОД'}<br>• Пик загруженности: 18:00-19:30<br>• Рекомендуется: ${hour >= 17 && hour <= 19 ? 'избегать центр, использовать ОТ' : 'свободное движение'}<br>• К 20:30 ситуация нормализуется`;
            }
            else {
                response = `🤖 Я AI-помощник по Алматы. Спросите меня:<br><br>• "Пробки на Достык" - анализ улицы<br>• "Экология в Медеуском районе"<br>• "Прогноз на сегодня"<br>• "Где самые большие пробки?"<br><br>Или нажмите на любую улицу на карте!`;
            }
            
            addAIMessage(response);
        }

        function switchTab(tab) {
            currentTab = tab;
            document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
            document.getElementById(`${tab}Tab`).classList.add('active');
            document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
            document.querySelector(`[data-tab="${tab}"]`).classList.add('active');
            
            if (tab === 'map' && map) {
                setTimeout(() => map.invalidateSize(), 100);
            }
        }

        function changeMapLayer() { /* API call would go here */ }
        function toggleTraffic() { trafficVisible = !trafficVisible; if (map) addRoads(); }
        function toggleNotifications() { /* Toggle notifications */ }
        
        function refreshAll() {
            updateDashboard();
            if (map && trafficVisible) addRoads();
        }

        function startAutoUpdate() {
            updateDashboard();
            if (updateInterval) clearInterval(updateInterval);
            setInterval(() => {
                updateDashboard();
                if (map && trafficVisible) addRoads();
            }, 10000);
        }

        window.switchTab = switchTab;
        window.askAboutStreet = askAboutStreet;
        window.toggleChat = toggleChat;
        window.sendChatMessage = sendChatMessage;
        window.refreshAll = refreshAll;
        window.changeMapLayer = changeMapLayer;
        window.toggleTraffic = toggleTraffic;
        window.toggleNotifications = toggleNotifications;
        
        document.getElementById('chatInput').addEventListener('keypress', (e) => {
            if (e.key === 'Enter') sendChatMessage();
        });
        
        window.addEventListener('load', () => {
            initMap();
            startAutoUpdate();
            document.getElementById('aiChat').style.display = 'flex';
        });
    </script>
</body>
</html>
