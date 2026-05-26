<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>מוניטור אוטובוסים בזמן אמת</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: #f4f6f9;
        }
        header {
            background-color: #2c3e50;
            color: white;
            padding: 15px;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            z-index: 1000;
        }
        .search-container {
            margin-top: 10px;
            display: flex;
            justify-content: center;
            gap: 10px;
        }
        input[type="text"] {
            padding: 10px;
            font-size: 16px;
            border: none;
            border-radius: 4px;
            width: 150px;
            text-align: center;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #e67e22;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            background-color: #d35400;
        }
        #map {
            flex-grow: 1;
            width: 100%;
            height: 100%;
        }
        .loading {
            display: none;
            color: #f1c40f;
            font-weight: bold;
            margin-top: 5px;
        }
        .bus-popup {
            text-align: right;
            direction: rtl;
        }
        .bus-popup h3 {
            margin: 0 0 5px 0;
            color: #2c3e50;
            border-bottom: 1px solid #ccc;
        }
    </style>
</head>
<body>

    <header>
        <h2>מיקומי אוטובוסים בארץ בזמן אמת</h2>
        <div class="search-container">
            <input type="text" id="lineInput" placeholder="הכנס מספר קו (למשל: 1)">
            <button onclick="searchBus()">חפש אוטובוס</button>
        </div>
        <div id="loadingStatus" class="loading">מתחבר לנתוני ה-GPS של האוטובוסים...</div>
    </header>

    <div id="map"></div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

    <script>
        const map = L.map('map').setView([32.0853, 34.7818], 10); 

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        let busMarkers = [];

        async function searchBus() {
            const lineRef = document.getElementById('lineInput').value.trim();
            const loadingStatus = document.getElementById('loadingStatus');
            
            if (!lineRef) {
                alert('אנא הכנס מספר קו');
                return;
            }

            loadingStatus.style.display = 'block';
            clearMarkers();

            try {
                // מעקף CORS מושלם עבור אתרים שמארחים ב-GitHub Pages
                const targetUrl = encodeURIComponent(`https://api.opentransit.org.il/siri/vehicle_locations?route_short_name=${lineRef}`);
                const url = `https://api.allorigins.win/get?url=${targetUrl}`;
                
                const response = await fetch(url);
                if (!response.ok) throw new Error('בעיה בתקשורת עם השרת');

                const json = await response.json();
                const data = JSON.parse(json.contents); // פיענוח המידע שחוזר מהפרוקסי
                
                if (!data || data.length === 0) {
                    alert(`לא נמצאו אוטובוסים פעילים כרגע עבור קו ${lineRef}.`);
                    return;
                }

                let bounds = [];

                data.forEach(bus => {
                    const lat = bus.latitude;
                    const lon = bus.longitude;
                    
                    if (lat && lon) {
                        const operator = bus.operator_name || 'מפעיל לא ידוע';
                        const destination = bus.destination_name || 'יעד לא צוין';
                        const timeUpdated = bus.recorded_at_time ? new Date(bus.recorded_at_time).toLocaleTimeString('he-IL') : 'לא ידוע';

                        const popupContent = `
                            <div class="bus-popup">
                                <h3>קו ${lineRef} - ${operator}</h3>
                                <p><strong>כיוון נסיעה/יעד:</strong> ${destination}</p>
                                <p><strong>עודכן ב:</strong> ${timeUpdated}</p>
                            </div>
                        `;

                        const marker = L.marker([lat, lon]).addTo(map).bindPopup(popupContent);
                        busMarkers.push(marker);
                        bounds.push([lat, lon]);
                    }
                });

                if (bounds.length > 0) {
                    map.fitBounds(bounds, { padding: [50, 50] });
                }

            } catch (error) {
                console.error(error);
                alert('שגיאה בקבלת נתונים מהשרת. נסה שוב בעוד כמה רגעים.');
            } military {
                loadingStatus.style.display = 'none';
            }
        }

        function clearMarkers() {
            busMarkers.forEach(marker => map.removeLayer(marker));
            busMarkers = [];
        }
    </script>
</body>
</html>
