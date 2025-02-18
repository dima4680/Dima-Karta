<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности путей необщего пользования на Красноярской железной дороге</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.Default.css" />
    <script src="https://cdn.jsdelivr.net/npm/lz-string@1.4.4/libs/lz-string.min.js"></script>
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
            color: black;
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
    <button id="saveDataBtn" class="hidden">Сохранить данные</button>
    <button id="shareDataBtn" class="hidden">Поделиться данными</button>
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
            if (percentage === 0) return '#ff4444';
            if (percentage === 100) return '#00c851';
            return '#ffbb33';
        }

        function getMarkerSize(total) {
            const minSize = 40;
            const maxSize = 70;
            const minWagons = 10;
            const maxWagons = 100;
            return Math.min(maxSize, Math.max(minSize, 
                ((total - minWagons) / (maxWagons - minWagons)) * (maxSize - minSize) + minSize
            ));
        }

        // Функция для отображения данных на карте
        function displayData(data) {
            markers.clearLayers();
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
                            В отстое: ${total - free} вагонов<br>
                            Контакты для связи: 8(391)259-54-70<br>
                            Обновлено: 12.02.25
                        `);

                    markers.addLayer(marker);
                }
            });
            map.addLayer(markers);
        }

        // Сжатие данных
        function compressData(data) {
            return LZString.compressToEncodedURIComponent(JSON.stringify(data));
        }

        // Декомпрессия данных
        function decompressData(compressed) {
            try {
                return JSON.parse(LZString.decompressFromEncodedURIComponent(compressed));
            } catch (e) {
                alert('Ошибка декомпрессии данных: ' + e.message);
                return null;
            }
        }

        // Проверка авторизации
        function checkAuth() {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            document.getElementById('excelFile').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('logoutBtn').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('loginBtn').classList.toggle('hidden', isLoggedIn);
            document.getElementById('saveDataBtn').classList.toggle('hidden', !isLoggedIn);
            document.getElementById('shareDataBtn').classList.toggle('hidden', !isLoggedIn);
        }

        // Обработчик входа
        document.getElementById('loginBtn').addEventListener('click', () => {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
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
                const jsonData = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]]);
                localStorage.setItem('mapData', JSON.stringify(jsonData));
                displayData(jsonData);
            };
            reader.readAsArrayBuffer(file);
        });

        // Обработчик сохранения данных
        document.getElementById('saveDataBtn').addEventListener('click', () => {
            alert(localStorage.getItem('mapData') ? 'Данные сохранены!' : 'Нет данных');
        });

        // Обработчик кнопки "Поделиться данными"
        document.getElementById('shareDataBtn').addEventListener('click', async () => {
            const rawData = localStorage.getItem('mapData');
            if (!rawData) return alert('Нет данных для分享');

            try {
                const jsonData = JSON.parse(rawData);
                const compressed = compressData(jsonData);
                const shareUrl = `${window.location.href.split('?')[0]}?d=${compressed}`;

                // Копирование в буфер обмена
                try {
                    await navigator.clipboard.writeText(shareUrl);
                    alert('Сокращенная ссылка скопирована:\n' + shareUrl);
                } catch (err) {
                    const textarea = document.createElement('textarea');
                    textarea.value = shareUrl;
                    document.body.appendChild(textarea);
                    textarea.select();
                    document.execCommand('copy');
                    document.body.removeChild(textarea);
                    alert('Ссылка скопирована вручную:\n' + shareUrl);
                }
            } catch (e) {
                alert('Ошибка генерации ссылки: ' + e.message);
            }
        });

        // Загрузка данных из URL
        const urlParams = new URLSearchParams(window.location.search);
        const compressedData = urlParams.get('d');
        if (compressedData) {
            const jsonData = decompressData(compressedData);
            if (jsonData) {
                displayData(jsonData);
                document.querySelectorAll('.auth-section, #excelFile, #saveDataBtn, #shareDataBtn')
                    .forEach(el => el.classList.add('hidden'));
            }
        } else if (localStorage.getItem('mapData')) {
            displayData(JSON.parse(localStorage.getItem('mapData')));
        }

        // Проверка авторизации при загрузке страницы
        checkAuth();
    </script>
</body>
</html>
