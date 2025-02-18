<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности путей необщего пользования на Красноярской железной дороге</title>
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
        /* Стили для модального окна */
        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
        }
        .modal-content {
            background-color: white;
            margin: 15% auto;
            padding: 20px;
            border: 1px solid #888;
            width: 50%;
            max-width: 600px;
            text-align: center;
        }
        .modal input {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
        }
        .modal button {
            padding: 10px 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="auth-section">
        <input type="text" id="username" placeholder="Логин">
        <input type="password" <input type="password" id="password" placeholder="Пароль">
        <button id="loginBtn">Войти</button>
        <button id="logoutBtn" class="hidden">Выйти</button>
    </div>
    <h1>Карта загруженности вагонов</h1>
    <input type="file" id="excelFile" accept=".xlsx, .xls" class="hidden">
    <button id="saveDataBtn" class="hidden">Сохранить данные</button>
    <button id="shareDataBtn" class="hidden">Поделиться данными</button>
    <div id="map"></div>

    <!-- Модальное окно для отображения ссылки -->
    <div id="shareModal" class="modal">
        <div class="modal-content">
            <h3>Ссылка для分享 данных</h3>
            <input type="text" id="shareUrlInput" readonly>
            <button id="copyUrlBtn">Копировать ссылку</button>
            <button id="closeModalBtn">Закрыть</button>
        </div>
    </div>

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
            if (percentage === 0) return '#ff4444'; // Красный
            if (percentage === 100) return '#00c851'; // Зеленый
            return '#ffbb33'; // Желтый
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
                            В отстое: ${total - free} вагонов<br>
                            Контакты для связи: 8(391)259-54-70<br>
                            Обновлено: 12.02.25
                        `);

                    markers.addLayer(marker);
                }
            });

            map.addLayer(markers);
        }

        // Функция для кодирования данных в base64
        function encodeData(data) {
            return btoa(JSON.stringify(data));
        }

        // Функция для декодирования данных из base64
        function decodeData(encodedData) {
            return JSON.parse(atob(encodedData));
        }

        // Проверка авторизации
        function checkAuth() {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            if (isLoggedIn) {
                document.getElementById('excelFile').classList.remove('hidden');
                document.getElementById('logoutBtn').classList.remove('hidden');
                document.getElementById('loginBtn').classList.add('hidden');
                document.getElementById('saveDataBtn').classList.remove('hidden');
                document.getElementById('shareDataBtn').classList.remove('hidden');
            } else {
                document.getElementById('exercursor: pointer;
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

    <!-- Модальное окно для отображения ссылки -->
    <div id="shareModal" class="modal">
        <div class="modal-content">
            <h3>Ссылка для分享 данных</h3>
            <input type="text" id="shareUrlInput" readonly>
            <button id="copyUrlBtn">Копировать ссылку</button>
            <button id="closeModalBtn">Закрыть</button>
        </div>
    </div>

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
            if (percentage === 0) return '#ff4444'; // Красный
            if (percentage === 100) return '#00c851'; // Зеленый
            return '#ffbb33'; // Желтый
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
                            В отстое: ${total - free} вагонов<br>
                            Контакты для связи: 8(391)259-54-70<br>
                            Обновено: 12.02.25
                        `);

                    markers.addLayer(marker);
                }
            });

            map.addLayer(markers);
        }

        // Функция для кодирования данных в base64
        function encodeData(data) {
            return btoa(JSON.stringify(data));
        }

        // Функция для декодирования данных из base64
        function decodeData(encodedData) {
            return JSON.parse(atob(encodedData));
        }

        // Проверка авторизации
        function checkAuth() {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            if (isLoggedIn) {
                document.getElementById('excelFile').classList.remove('hidden');
                document.getElementById('logoutBtn').classList.remove('hidden');
                document.getElementById('loginBtn').classList.add('hidden');
                document.getElementById('saveDataBtn').classList.remove('hidden');
                document.getElementById('shareDataBtn').classList.remove('hidden');
            else {
                document.getElementById('logoutBtn').classList.add('hidden');
                document.getElementById('loginBtn').classList.remove('hidden');
                document.getElementById('saveDataBtn').classList.add('hidden');
                document.getElementById('shareDataBtn').classList.add('hidden');
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
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            if (!isLoggedIn) {
                alert('Для загрузки файла необходимо войти в систему.');
                return;
            }

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

        // Обработчик сохранения данных
        document.getElementById('saveDataBtn').addEventListener('click', () => {
            const isLoggedIn = localStorage.getItem('isLoggedIn') === 'true';
            if (!isLoggedIn) {
                alert('Для сохранения данных необходимо войти в систему.');
                return;
            }

            const data = localStorage.getItem('mapData');
            if (data) {
                alert('Данные успешно сохранены!');
            } else {
                alert('Нет данных для сохранения.');
            }
        });

        // Открытие модального окна для分享 ссылки
        document.getElementById('shareDataBtn').addEventListener('click', () => {
            const data = localStorage.getItem('mapData');
            if (data) {
                const encodedData = encodeData(data);
                const shareUrl = `${window.location.origin}${window.location.pathname}?data=${encodedData}`;

                // Отображение модального окна
                const modal = document.getElementById('shareModal');
                const shareUrlInput = document.getElementById('shareUrlInput');
                shareUrlInput.value = shareUrl;
                modal.style.display = 'block';
            } else {
                alert('Нет данных для分享.');
            }
        });

        // Копирование ссылки в буфер обмена
        document.getElementById('copyUrlBtn').addEventListener('click', () => {
            const shareUrlInput = document.getElementById('shareUrlInput');
            shareUrlInput.select();
            navigator.clipboard.writeText(shareUrlInput.value)
                .then(() => alert('Ссылка скопирована в буфер обмена!'))
                .catch(() => alert('Не удалось скопировать ссылку.'));
        });

        // Закрытие модального окна
        document.getElementById('closeModalBtn').addEventListener('click', () => {
            document.getElementById('shareModal').style.display = 'none';
        });

        // Загрузка данных из URL, если они есть
        const urlParams = new URLSearchParams(window.location.search);
        const sharedData = urlParams.get('data');
        if (sharedData) {
            try {
                const decodedData = decodeData(sharedData);
                displayData(decodedData);

                // Скрываем элементы управления для пользователей, открывших ссылку
                document.getElementById('auth-section').classList.add('hidden');
                document.getElementById('excelFile').classList.add('hidden');
                document.getElementById('saveDataBtn').classList.add('hidden');
                document.getElementById('shareDataBtn').classList.add('hidden');
            } catch (e) {
                alert('Ошибка при загрузке данных: неверный формат.');
            }
        } else {
            // Загрузка данных из localStorage, если нет данных в URL
            const savedData = localStorage.getItem('mapData');
            if (savedData) {
                displayData(JSON.parse(savedData));
            }
        }

        // Проверка авторизации при загрузке страницы
        checkAuth();
    </script>
</body>
</html>
