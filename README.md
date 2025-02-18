<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карта загруженности вагонов</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
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
    </style>
</head>
<body>
    <h1>Загрузите файл с данными станций (.xlsx)</h1>
    <input type="file" id="excelFile" accept=".xlsx, .xls">
    <div id="map"></div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script>
        // Инициализация карты
        const map = L.map('map').setView([56.0184, 92.8672], 5);
        
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

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

                console.log(jsonData); // Вывод данных в консоль

                // Очистка старых маркеров
                map.eachLayer(layer => {
                    if (layer instanceof L.Marker) map.removeLayer(layer);
                });

                // Создание новых маркеров
                jsonData.forEach(row => {
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
                                Обновлено: 12.02.25
                            `)
                            .addTo(map);
                    }
                });
            };
            reader.readAsArrayBuffer(file);
        });
    </script>
</body>
</html>
