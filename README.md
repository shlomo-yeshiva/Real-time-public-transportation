<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>מרכז בקרה - אוטובוסים בזמן אמת</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
    
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
            color: #333;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }
        header {
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            color: white;
            padding: 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.15);
            z-index: 1000;
            text-align: center;
        }
        h1 {
            font-size: 22px;
            margin-bottom: 15px;
            font-weight: 600;
        }
        .controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 12px;
            flex-wrap: wrap;
        }
        .input-group {
            display: flex;
            align-items: center;
            background: rgba(255, 255, 255, 0.15);
            border-radius: 8px;
            padding: 4px 12px;
            border: 1px solid rgba(255, 255, 255, 0.2);
        }
        .input-group label {
            font-size: 14px;
            margin-left: 8px;
            color: #e0e6ed;
        }
        input, select {
            background: transparent;
            border: none;
            color: white;
            font-size: 16px;
            padding: 8px 4px;
            outline: none;
            font-family: inherit;
        }
        select option {
            background-color: #1e3c72;
            color: white;
        }
        input::placeholder {
            color: #b0c4de;
        }
        button {
            background-color: #ff9f43;
            color: white;
            border: none;
            padding: 12px 28px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: all 0.2s ease;
        }
        button:hover {
            background-color: #f39c12;
        }
        .loading-container {
            margin-top: 12px;
            font-size: 14px;
            color: #f1c40f;
            display: none;
            font-weight: 500;
        }
        #map {
            flex-grow: 1;
            width: 100%;
            height: 100%;
        }
        .bus-popup-content {
            direction: rtl;
            text-align: right;
            min-width: 180px;
        }
        .bus-popup-content h3 {
            margin-bottom: 8px;
            color: #1e3c72;
            font-size: 16px;
            border-bottom: 2px solid #f0f2f5;
            padding-bottom: 4px;
        }
        .bus-popup-content p {
            font-size: 13px;
            margin-bottom: 5px;
        }
    </style>
</head>
<body>

    <header>
        <h1>מרכז בקרה בזמן אמת - מיקומי אוטובוסים</h1>
        <div class="controls">
            <div class="input-group">
                <label for="operatorSelect">מפעיל:</label>
                <select id="operatorSelect">
                    <option value="">כל החברות</option>
                    <option value="אגד">אגד</option>
                    <option value="דן">דן</option>
                    <option value="קווים">קווים</option>
                    <option value="מטרופולין">מטרופולין</option>
                    <option value="אפיקים">אלקטרה אפיקים</option>
                    <option value="סופרבוס">סופרבוס</option>
                    <option value="תנופה">תנופה</option>
                </select>
            </div>
            
            <div class="input-group">
                <label for="lineInput">מספר קו:</label>
                <input type="text" id="lineInput" placeholder="למשל: 1" autocomplete="off">
            </div>
            
            <button onclick="searchBus()">חפש עכשיו</button>
        </div>
        <div id="loadingStatus" class="loading-container">⚡ מושך נתוני GPS ישירים משרתי התחבורה...</div>
    </header>

    <div id="map"></div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    <script>
        // אתחול המפה על מרכז הארץ
        const map = L.map('map').setView([32.0853, 34.7818], 9);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            maxZoom: 19,
            attribution: '© OpenStreetMap contributors'
        }).addTo(map);

        let busMarkers = [];

        async function searchBus() {
            const lineRef = document.getElementById('lineInput').value.trim();
            const selectedOperator = document.getElementById('operatorSelect').value;
            const loadingStatus = document.getElementById('loadingStatus');

            if (!lineRef) {
                alert('אנא הכנס מספר קו לחיפוש.');
                return;
            }

            loadingStatus.style.display = 'block';
            clearMarkers();

            try {
                // פנייה ישירה ומאובטחת ל-API ללא שרת פרוקסי מתווך
                const directUrl = `https://api.opentransit.org.il/siri/vehicle_locations?route_short_name=${encodeURIComponent(lineRef)}`;
                
                const response = await fetch(directUrl);
                
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }

                const data = await response.json();

                if (!data || data.length === 0) {
                    alert(`לא נמצאו אוטובוסים פעילים ברגע זה עבור קו ${lineRef}.`);
                    return;
                }

                // סינון לפי מפעיל אם נבחר
                let filteredData = data;
                if (selectedOperator) {
                    filteredData = data.filter(bus => bus.operator_name && bus.operator_name.includes(selectedOperator));
                }

                if (filteredData.length === 0) {
                    alert(`נמצאו אוטובוסים לקו ${lineRef}, אך אף אחד מהם אינו של חברת "${selectedOperator}".`);
                    return;
                }

                let bounds = [];

                filteredData.forEach(bus => {
                    const lat = bus.latitude;
                    const lon = bus.longitude;

                    if (lat && lon) {
                        const operator = bus.operator_name || 'מפעיל לא ידוע';
                        const destination = bus.destination_name || 'יעד לא צוין';
                        
                        let timeString = 'לא ידוע';
                        if (bus.recorded_at_time) {
                            try {
                                timeString = new Date(bus.recorded_at_time).toLocaleTimeString('he-IL', {
                                    hour: '2-digit',
                                    minute: '2-digit'
                                });
                            } catch (e) {
                                timeString = bus.recorded_at_time;
                            }
                        }

                        const popupContent = `
                            <div class="bus-popup-content">
                                <h3>קו ${lineRef} - ${operator}</h3>
                                <p><strong>כיוון:</strong> ${destination}</p>
                                <p><strong>עדכון אחרון:</strong> ${timeString}</p>
                            </div>
                        `;

                        // יצירת מרקר מותאם
                        const marker = L.marker([lat, lon]).addTo(map).bindPopup(popupContent);
                        busMarkers.push(marker);
                        bounds.push([lat, lon]);
                    }
                });

                if (bounds.length > 0) {
                    map.fitBounds(bounds, { padding: [50, 50], maxZoom: 16 });
                }

            } catch (error) {
                console.error('Error fetching data:', error);
                alert('שגיאה בקבלת הנתונים. ודא שאתה גולש מהאתר שהעלית ל-GitHub (ולא מתוך קובץ מקומי על המחשב) ונסה שוב.');
            } finally {
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
