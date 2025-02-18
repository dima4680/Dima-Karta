<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интерактивная карта из Excel</title>
    <style>
        #map {
            height: 800px;
        }
    </style>
</head>
<body>
    <h1>Загрузите Excel-файл с данными</h1>
    <input type="file" id="excelFile" accept=".xlsx, .xls" />
    <div id="map"></div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script>
        // Инициализация карты
        const map = L.map('map').setView([56.0184, 92.8672], 5); // Центр на Красноярске

        // Добавление слоя с картой (используем OpenStreetMap)
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        // Обработчик загрузки файла
        document.getElementById('excelFile').addEventListener('change', function (e) {
            const file = e.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function (e) {
                const data = new Uint8Array(e.target.result);
                const workbook = XLSX.read(data, { type: 'array' });

                // Предполагаем, что данные находятся на первом листе
                const sheetName = workbook.SheetNames[0];
                const sheet = workbook.Sheets[sheetName];

                // Преобразуем лист в JSON
                const jsonData = XLSX.utils.sheet_to_json(sheet);

                // Очищаем старые маркеры (если есть)
                map.eachLayer(layer => {
                    if (layer instanceof L.Marker) {
                        map.removeLayer(layer);
                    }
                });

                // Добавляем маркеры на карту
                jsonData.forEach(row => {
                    const lat = row['Широта']; // Название столбца с широтой
                    const lng = row['Долгота']; // Название столбца с долготой
                    const stationName = row['Станция']; // Название столбца с именем станции
                    const freeWagons = row['Свободные вагоны']; // Название столбца с количеством свободных вагонов
                    const totalWagons = row['Всего вагонов']; // Название столбца с общим количеством вагонов

                    if (lat && lng) {
                        const marker = L.marker([lat, lng]).addTo(map);
                        marker.bindPopup(`<b>${stationName}</b><br>По состоянию на 12.02.25 свободно ${freeWagons} из ${totalWagons} вагонов.`);
                    }
                });
            };
            reader.readAsArrayBuffer(file);
        });
    </script>
</body>
</html>
