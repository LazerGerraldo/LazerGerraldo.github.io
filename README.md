<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Global Traveler Map</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.css" />
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: #f5f5f5;
        }
        
        header {
            background-color: #2c3e50;
            color: white;
            padding: 1rem;
            text-align: center;
        }
        
        .container {
            display: flex;
            flex: 1;
            overflow: hidden;
        }
        
        .sidebar {
            width: 300px;
            background-color: white;
            padding: 1rem;
            box-shadow: 2px 0 5px rgba(0,0,0,0.1);
            overflow-y: auto;
            display: flex;
            flex-direction: column;
        }
        
        .map-container {
            flex: 1;
            position: relative;
        }
        
        #map {
            height: 100%;
            width: 100%;
        }
        
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, select {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        button {
            background-color: #3498db;
            color: white;
            border: none;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            cursor: pointer;
            margin-top: 0.5rem;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: #2980b9;
        }
        
        .entry-list {
            margin-top: 1rem;
            overflow-y: auto;
            flex: 1;
        }
        
        .entry-card {
            background-color: #f9f9f9;
            border-radius: 4px;
            padding: 0.5rem;
            margin-bottom: 0.5rem;
            position: relative;
        }
        
        .entry-card h3 {
            margin: 0 0 0.5rem 0;
            font-size: 1rem;
        }
        
        .entry-card p {
            margin: 0.2rem 0;
            font-size: 0.9rem;
        }
        
        .delete-btn {
            position: absolute;
            top: 0.5rem;
            right: 0.5rem;
            background-color: #e74c3c;
            color: white;
            border: none;
            width: 24px;
            height: 24px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            font-size: 12px;
        }
        
        .delete-btn:hover {
            background-color: #c0392b;
        }
        
        .search-container {
            margin-bottom: 1rem;
        }
        
        .toggle-sidebar {
            display: none;
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 1000;
            background-color: white;
            border: none;
            border-radius: 4px;
            padding: 0.5rem;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        
        @media (max-width: 768px) {
            .container {
                flex-direction: column;
            }
            
            .sidebar {
                width: 100%;
                height: 250px;
            }
            
            .toggle-sidebar {
                display: block;
            }
            
            .sidebar.collapsed {
                display: none;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>Global Traveler Map</h1>
        <p>Track your global adventures and connections</p>
    </header>
    
    <div class="container">
        <button class="toggle-sidebar">☰</button>
        
        <div class="sidebar">
            <div class="search-container">
                <input type="text" id="search-input" placeholder="Search entries...">
            </div>
            
            <form id="location-form">
                <div class="form-group">
                    <label for="person-name">Person's Name</label>
                    <input type="text" id="person-name" required>
                </div>
                
                <div class="form-group">
                    <label for="location">Location</label>
                    <input type="text" id="location" placeholder="City, Country" required>
                </div>
                
                <div class="form-group">
                    <label for="arrival-date">Arrival Date</label>
                    <input type="date" id="arrival-date" required>
                </div>
                
                <div class="form-group">
                    <label for="notes">Additional Notes</label>
                    <input type="text" id="notes" placeholder="Optional notes">
                </div>
                
                <button type="submit">Add to Map</button>
            </form>
            
            <h2>Entries</h2>
            <div class="entry-list" id="entry-list">
                <!-- Entries will be added dynamically -->
            </div>
        </div>
        
        <div class="map-container">
            <div id="map"></div>
        </div>
    </div>

    <script>
        // Initialize the map
        const map = L.map('map').setView([20, 0], 2);
        
        // Add tile layer (OpenStreetMap)
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
        }).addTo(map);
        
        // Store markers and entries
        const markers = [];
        const entries = [];
        
        // Form submission handler
        document.getElementById('location-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const personName = document.getElementById('person-name').value;
            const location = document.getElementById('location').value;
            const arrivalDate = document.getElementById('arrival-date').value;
            const notes = document.getElementById('notes').value;
            
            // Geocode the location (in a real app, you'd use a geocoding service)
            // For this demo, we'll simulate geocoding with random coordinates near the equator
            geocodeLocation(location, function(coords) {
                if (coords) {
                    addEntry(personName, location, arrivalDate, notes, coords);
                    clearForm();
                } else {
                    alert('Could not find coordinates for the specified location.');
                }
            });
        });
        
        // Simulated geocoding function
        function geocodeLocation(locationStr, callback) {
            // In a real app, you'd use a service like OpenStreetMap Nominatim or Google Geocoding API
            // For this demo, we'll generate random coordinates based on the location string
            
            // Generate a simple hash of the location string
            let hash = 0;
            for (let i = 0; i < locationStr.length; i++) {
                hash = ((hash << 5) - hash) + locationStr.charCodeAt(i);
                hash |= 0; // Convert to 32bit integer
            }
            
            // Use the hash to generate "random" but consistent coordinates
            const lat = ((hash % 160) - 80) + (Math.random() * 5 - 2.5);
            const lng = ((hash % 360) - 180) + (Math.random() * 5 - 2.5);
            
            setTimeout(() => {
                callback({ lat, lng });
            }, 300); // Simulate network delay
        }
        
        // Add entry to the map and list
        function addEntry(name, location, date, notes, coords) {
            const id = Date.now().toString();
            const entryData = {
                id,
                name,
                location,
                date,
                notes,
                coords
            };
            
            // Add to entries array
            entries.push(entryData);
            
            // Create marker
            const marker = L.marker([coords.lat, coords.lng]).addTo(map);
            marker.entryId = id;
            
            // Create popup content
            const popupContent = `
                <strong>${name}</strong><br>
                Location: ${location}<br>
                Arrived: ${formatDate(date)}
                ${notes ? `<br>Notes: ${notes}` : ''}
            `;
            
            marker.bindPopup(popupContent);
            markers.push(marker);
            
            // Add to sidebar list
            addEntryToList(entryData);
            
            // Save to localStorage
            saveEntries();
        }
        
        // Add entry to the sidebar list
        function addEntryToList(entry) {
            const entryList = document.getElementById('entry-list');
            
            const entryCard = document.createElement('div');
            entryCard.className = 'entry-card';
            entryCard.dataset.id = entry.id;
            
            entryCard.innerHTML = `
                <h3>${entry.name}</h3>
                <p><strong>Location:</strong> ${entry.location}</p>
                <p><strong>Arrived:</strong> ${formatDate(entry.date)}</p>
                ${entry.notes ? `<p><strong>Notes:</strong> ${entry.notes}</p>` : ''}
                <button class="delete-btn">✕</button>
            `;
            
            // Add click event to highlight on map
            entryCard.addEventListener('click', function(e) {
                if (e.target.className !== 'delete-btn') {
                    // Find the marker
                    const marker = markers.find(m => m.entryId === entry.id);
                    if (marker) {
                        marker.openPopup();
                        map.setView(marker.getLatLng(), 5);
                    }
                }
            });
            
            // Add delete button handler
            entryCard.querySelector('.delete-btn').addEventListener('click', function() {
                removeEntry(entry.id);
            });
            
            entryList.appendChild(entryCard);
        }
        
        // Remove entry and marker
        function removeEntry(id) {
            // Remove marker from map
            const markerIndex = markers.findIndex(m => m.entryId === id);
            if (markerIndex !== -1) {
                map.removeLayer(markers[markerIndex]);
                markers.splice(markerIndex, 1);
            }
            
            // Remove from entries array
            const entryIndex = entries.findIndex(e => e.id === id);
            if (entryIndex !== -1) {
                entries.splice(entryIndex, 1);
            }
            
            // Remove from list
            const entryCard = document.querySelector(`.entry-card[data-id="${id}"]`);
            if (entryCard) {
                entryCard.remove();
            }
            
            // Save to localStorage
            saveEntries();
        }
        
        // Clear the form
        function clearForm() {
            document.getElementById('person-name').value = '';
            document.getElementById('location').value = '';
            document.getElementById('notes').value = '';
            // Keep the date as is for convenience
        }
        
        // Format date for display
        function formatDate(dateStr) {
            const date = new Date(dateStr);
            return date.toLocaleDateString();
        }
        
        // Save entries to localStorage
        function saveEntries() {
            localStorage.setItem('mapEntries', JSON.stringify(entries));
        }
        
        // Load entries from localStorage
        function loadEntries() {
            const savedEntries = localStorage.getItem('mapEntries');
            if (savedEntries) {
                const parsedEntries = JSON.parse(savedEntries);
                parsedEntries.forEach(entry => {
                    addEntry(entry.name, entry.location, entry.date, entry.notes, entry.coords);
                });
            }
        }
        
        // Handle search functionality
        document.getElementById('search-input').addEventListener('input', function(e) {
            const searchTerm = e.target.value.toLowerCase();
            
            document.querySelectorAll('.entry-card').forEach(card => {
                const text = card.textContent.toLowerCase();
                if (text.includes(searchTerm)) {
                    card.style.display = 'block';
                } else {
                    card.style.display = 'none';
                }
            });
        });
        
        // Handle sidebar toggle for mobile
        document.querySelector('.toggle-sidebar').addEventListener('click', function() {
            const sidebar = document.querySelector('.sidebar');
            sidebar.classList.toggle('collapsed');
        });
        
        // Load saved entries when the page loads
        document.addEventListener('DOMContentLoaded', function() {
            // Set default date to today
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('arrival-date').value = today;
            
            // Load saved entries
            loadEntries();
        });
    </script>
</body>
</html>
