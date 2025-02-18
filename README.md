<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности вагонов</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.Default.css" />
    <style>
        #map {
            height: 800px;
        }
        .custom-marker {
            background: transparent;
            border: none;
        }
        .marker-circle {
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            font-family: Arial;
            box-shadow: 0 2px 5px rgba(0,0,0,0.3);
        }
        .auth-section {
            margin-bottom: 20px;
        }
        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <div class="auth-section">
        <input type="text" id="username" placeholder="Логин">
        <input type="password" id="password" placeholder="Пароль">
        <button id="loginBtn">Войти</button>
        <button id="logoutBtn" class="hidden">Выйти</button>
    </div>
    <h1>Карта загруженности вагонов</h1>
    <input type="file" id="excelFile" accept=".xlsx, .xls" class="hidden">
    <div id="map"></div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet.markercluster/dist/leaflet.markercluster.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script>
        // Инициализация карты
        const map = L.map('map').setView([56.0184, 92.8672], 5);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        // Создание кластера маркеров
        const markers = L.markerClusterGroup();

        // Функции для определения стиля маркера
        function getMarkerColor(free, total) {
            const percentage = (free / total) * 100;
            if (percentage === 0) return '#ff4444';    // Красный
            if (percentage === 100) return '#00c851'; // Зеленый
            return '#ffbb33';                         // Желтый
        }

        function getMarkerSize(total) {
            const minSize = 30;
            const maxSize = 50;
            const minWagons = 10;
            const maxWagons = 100;
            return Math.min(maxSize, Math.max(minSize, 
                ((total - minWagons) / (maxWagons - minWagons)) * (maxSize - minSize) + minSize
            ));
        }

        // Функция для отображения данных на карте
        function displayData(data) {
            // Очистка старых маркеров
            markers.clearLayers();

            // Создание новых маркеров
            data.forEach(row => {
                const lat = row['Широта'];
                const lng = row['Долгота'];
                const station = row['Станция'];
                const free = row['Свободные вагоны'];
                const total = row['Всего вагонов'];

                if (lat && lng && free !== undefined && total !== undefined) {
                    const iconSize = getMarkerSize(total);
                    const icon = L.divIcon({
                        className: 'custom-marker',
                        iconSize: [iconSize, iconSize],
                        html: `
                            <div class="marker-circle" 
                                style="
                                    width: ${iconSize}px;
                                    height: ${iconSize}px;
                                    background: ${getMarkerColor(free, total)};
                                    font-size: ${iconSize * 0.4}px;
                                ">
                                ${free}/${total}
                            </div>
                        `
                    });

                    const marker = L.marker([lat, lng], { icon })
                        .bindPopup(`
                            <b>${station}</b><br>
                            Свободно: ${free} из ${total} вагонов<br>
                            Обстановленные: ${total - free} вагонов<br>
                            Обновлено: 12.02.25
                        `);

                    markers.addLayer(marker);
                }
            });

            map.addLayer(markers);
        }

        // Проверка авторизации
        function checkAuth() {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            if (isLoggedIn) {
                document.getElementById('excelFile').classList.remove('hidden');
                document.getElementById('logoutBtn').classList.remove('hidden');
                document.getElementById('loginBtn').classList.add('hidden');
            } else {
                document.getElementById('excelFile').classList.add('hidden');
                document.getElementById('logoutBtn').classList.add('hidden');
                document.getElementById('loginBtn').classList.remove('hidden');
            }
        }

        // Обработчик входа
        document.getElementById('loginBtn').addEventListener('click', () => {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;

            // Простая проверка (для примера)
            if (username === 'admin' && password === 'admin') {
                localStorage.setItem('isLoggedIn', 'true');
                checkAuth();
            } else {
                alert('Неверный логин или пароль');
            }
        });

        // Обработчик выхода
        document.getElementById('logoutBtn').addEventListener('click', () => {
            localStorage.setItem('isLoggedIn', 'false');
            checkAuth();
        });

        // Обработчик загрузки файла
        document.getElementById('excelFile').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, { type: 'array' });
                const sheet = workbook.Sheets[workbook.SheetNames[0]];
                const jsonData = XLSX.utils.sheet_to_json(sheet);

                // Сохранение данных в localStorage
                localStorage.setItem('mapData', JSON.stringify(jsonData));
                displayData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        });

        // Загрузка данных при открытии страницы
        const savedData = localStorage.getItem('mapData');
        if (savedData) {
            displayData(JSON.parse(savedData));
        }

        // Проверка авторизации при загрузке страницы
        checkAuth();
    </script>
</body>
</html>
