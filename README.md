<!DOCTYPE html>
<html lang="en" data-theme="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Extended Family Whereabouts Map</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.css" />
    <style>
        :root {
            --bg-color: #f5f5f5;
            --sidebar-bg: white;
            --header-bg: #2c3e50;
            --header-color: white;
            --text-color: #333;
            --card-bg: #f9f9f9;
            --input-border: #ddd;
            --input-bg: white;
            --primary-btn: #3498db;
            --primary-btn-hover: #2980b9;
            --danger-btn: #e74c3c;
            --danger-btn-hover: #c0392b;
            --success-btn: #2ecc71;
            --success-btn-hover: #27ae60;
            --edit-btn: #f39c12;
            --edit-btn-hover: #d35400;
            --shadow-color: rgba(0,0,0,0.1);
        }
        
        [data-theme="dark"] {
            --bg-color: #1a1a1a;
            --sidebar-bg: #2c2c2c;
            --header-bg: #1c2834;
            --header-color: #f0f0f0;
            --text-color: #f0f0f0;
            --card-bg: #3c3c3c;
            --input-border: #555;
            --input-bg: #444;
            --primary-btn: #2980b9;
            --primary-btn-hover: #3498db;
            --danger-btn: #c0392b;
            --danger-btn-hover: #e74c3c;
            --success-btn: #27ae60;
            --success-btn-hover: #2ecc71;
            --edit-btn: #d35400;
            --edit-btn-hover: #f39c12;
            --shadow-color: rgba(0,0,0,0.4);
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: background-color 0.3s, color 0.3s;
        }
        
        header {
            background-color: var(--header-bg);
            color: var(--header-color);
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
            background-color: var(--sidebar-bg);
            padding: 1rem;
            box-shadow: 2px 0 5px var(--shadow-color);
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            transition: background-color 0.3s;
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
            position: relative;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid var(--input-border);
            border-radius: 4px;
            background-color: var(--input-bg);
            color: var(--text-color);
            transition: background-color 0.3s, border 0.3s, color 0.3s;
        }
        
        button {
            color: white;
            border: none;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            cursor: pointer;
            margin-top: 0.5rem;
            transition: background-color 0.3s;
        }
        
        .primary-btn {
            background-color: var(--primary-btn);
        }
        
        .primary-btn:hover {
            background-color: var(--primary-btn-hover);
        }
        
        .danger-btn {
            background-color: var(--danger-btn);
        }
        
        .danger-btn:hover {
            background-color: var(--danger-btn-hover);
        }
        
        .success-btn {
            background-color: var(--success-btn);
        }
        
        .success-btn:hover {
            background-color: var(--success-btn-hover);
        }
        
        .edit-btn {
            background-color: var(--edit-btn);
        }
        
        .edit-btn:hover {
            background-color: var(--edit-btn-hover);
        }
        
        .entry-list {
            margin-top: 1rem;
            overflow-y: auto;
            flex: 1;
        }
        
        .entry-card {
            background-color: var(--card-bg);
            border-radius: 4px;
            padding: 0.5rem;
            margin-bottom: 0.5rem;
            position: relative;
            transition: background-color 0.3s;
        }
        
        .entry-card h3 {
            margin: 0 0 0.5rem 0;
            font-size: 1rem;
        }
        
        .entry-card p {
            margin: 0.2rem 0;
            font-size: 0.9rem;
        }
        
        .card-buttons {
            position: absolute;
            top: 0.5rem;
            right: 0.5rem;
            display: flex;
            gap: 5px;
        }
        
        .card-btn {
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
            padding: 0;
            margin: 0;
        }
        
        .search-container {
            margin-bottom: 1rem;
            display: flex;
            align-items: center;
        }
        
        .search-container input {
            flex: 1;
        }
        
        .toggle-theme {
            margin-left: 10px;
            background: none;
            border: none;
            cursor: pointer;
            font-size: 1.2rem;
            padding: 5px;
            color: var(--text-color);
        }
        
        .toggle-sidebar {
            display: none;
            position: absolute;
            top: 10px;
            left: 10px;
            z-index: 1000;
            background-color: var(--sidebar-bg);
            color: var(--text-color);
            border: none;
            border-radius: 4px;
            padding: 0.5rem;
            box-shadow: 0 2px 5px var(--shadow-color);
        }
        
        /* Location suggestions */
        .location-suggestions {
            position: absolute;
            width: 100%;
            background-color: var(--input-bg);
            border: 1px solid var(--input-border);
            border-top: none;
            border-radius: 0 0 4px 4px;
            max-height: 200px;
            overflow-y: auto;
            z-index: 1000;
            box-shadow: 0 4px 6px var(--shadow-color);
            display: none;
        }
        
        .suggestion-item {
            padding: 8px 12px;
            cursor: pointer;
            color: var(--text-color);
        }
        
        .suggestion-item:hover {
            background-color: var(--card-bg);
        }
        
        .location-details {
            font-size: 0.8rem;
            color: var(--text-color);
            opacity: 0.7;
            margin-top: 4px;
        }
        
        .coordinates-display {
            font-size: 0.8rem;
            color: var(--text-color);
            opacity: 0.7;
            margin-top: 4px;
        }
        
        /* Modal styles */
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            z-index: 2000;
            justify-content: center;
            align-items: center;
        }
        
        .modal-content {
            background-color: var(--sidebar-bg);
            padding: 20px;
            border-radius: 8px;
            width: 90%;
            max-width: 500px;
            box-shadow: 0 4px 8px var(--shadow-color);
        }
        
        .modal-title {
            margin-top: 0;
            margin-bottom: 20px;
        }
        
        .modal-buttons {
            display: flex;
            justify-content: flex-end;
            gap: 10px;
            margin-top: 20px;
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
        <h1>Extended Family Whereabouts Map</h1>
        <p>Helping you keep track of your numerous extended family members all around the world.</p>
    </header>
    
    <div class="container">
        <button class="toggle-sidebar primary-btn">☰</button>
        
        <div class="sidebar">
            <div class="search-container">
                <input type="text" id="search-input" placeholder="Search entries...">
                <button class="toggle-theme" id="theme-toggle">🌙</button>
            </div>
            
            <form id="location-form">
                <input type="hidden" id="edit-id" value="">
                <div class="form-group">
                    <label for="person-name">Person's Name</label>
                    <input type="text" id="person-name" required>
                </div>
                
                <div class="form-group">
                    <label for="location">Location</label>
                    <input type="text" id="location" placeholder="Start typing a city name..." required autocomplete="off">
                    <div id="location-suggestions" class="location-suggestions"></div>
                    <div id="coordinates-display" class="coordinates-display"></div>
                </div>
                
                <div class="form-group">
                    <label for="arrival-date">Arrival Date</label>
                    <input type="date" id="arrival-date" required>
                </div>
                
                <div class="form-group">
                    <label for="notes">Additional Notes</label>
                    <textarea id="notes" placeholder="Optional notes" rows="3"></textarea>
                </div>
                
                <div id="form-buttons">
                    <button type="submit" class="primary-btn" id="add-button">Add to Map</button>
                </div>
            </form>
            
            <h2>Family Members</h2>
            <div class="entry-list" id="entry-list">
                <!-- Entries will be added dynamically -->
            </div>
        </div>
        
        <div class="map-container">
            <div id="map"></div>
        </div>
    </div>
    
    <!-- Edit Confirmation Modal -->
    <div id="edit-modal" class="modal">
        <div class="modal-content">
            <h3 class="modal-title">Edit Family Member Location</h3>
            <p>Update the information for this family member?</p>
            <div class="modal-buttons">
                <button id="cancel-edit" class="danger-btn">Cancel</button>
                <button id="confirm-edit" class="success-btn">Save Changes</button>
            </div>
        </div>
    </div>

    <script>
        // Theme management
        const themeToggle = document.getElementById('theme-toggle');
        let currentTheme = localStorage.getItem('theme') || 'light';
        
        // Set initial theme
        document.documentElement.setAttribute('data-theme', currentTheme);
        updateThemeIcon();
        
        themeToggle.addEventListener('click', function() {
            currentTheme = currentTheme === 'light' ? 'dark' : 'light';
            document.documentElement.setAttribute('data-theme', currentTheme);
            localStorage.setItem('theme', currentTheme);
            updateThemeIcon();
            updateMapTheme();
        });
        
        function updateThemeIcon() {
            themeToggle.textContent = currentTheme === 'light' ? '🌙' : '☀️';
        }
        
        // Initialize the map
        const map = L.map('map').setView([20, 0], 2);
        let tileLayer;
        
        function initMap() {
            // Add tile layer (OpenStreetMap)
            tileLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(map);
            
            updateMapTheme();
        }
        
        function updateMapTheme() {
            // Remove existing tile layer
            if (tileLayer) {
                map.removeLayer(tileLayer);
            }
            
            // Add appropriate tile layer based on theme
            if (currentTheme === 'dark') {
                tileLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
                    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>'
                }).addTo(map);
            } else {
                tileLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
                }).addTo(map);
            }
        }
        
        initMap();
        
        // Store markers and entries
        const markers = [];
        const entries = [];
        let selectedLocation = null;
        let isEditMode = false;
        
        // Setup location input with autocomplete
        const locationInput = document.getElementById('location');
        const suggestionsContainer = document.getElementById('location-suggestions');
        const coordinatesDisplay = document.getElementById('coordinates-display');
        const editIdInput = document.getElementById('edit-id');
        const addButton = document.getElementById('add-button');
        const editModal = document.getElementById('edit-modal');
        const cancelEditBtn = document.getElementById('cancel-edit');
        const confirmEditBtn = document.getElementById('confirm-edit');
        
        // Event listener for location input
        locationInput.addEventListener('input', debounce(function() {
            const query = locationInput.value.trim();
            if (query.length < 3) {
                suggestionsContainer.style.display = 'none';
                selectedLocation = null;
                coordinatesDisplay.textContent = '';
                return;
            }
            
            // Use Nominatim API to search for locations
            fetchLocationSuggestions(query);
        }, 300));
        
        // Hide suggestions when clicking outside
        document.addEventListener('click', function(e) {
            if (e.target !== locationInput && e.target !== suggestionsContainer) {
                suggestionsContainer.style.display = 'none';
            }
        });
        
        // Cancel edit button
        cancelEditBtn.addEventListener('click', function() {
            editModal.style.display = 'none';
            resetForm();
        });
        
        // Confirm edit button
        confirmEditBtn.addEventListener('click', function() {
            editModal.style.display = 'none';
            submitEdit();
        });
        
        // Fetch location suggestions from Nominatim API
        async function fetchLocationSuggestions(query) {
            try {
                const response = await fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}&limit=5`);
                if (!response.ok) throw new Error('Network response was not ok');
                
                const data = await response.json();
                displaySuggestions(data);
            } catch (error) {
                console.error('Error fetching location suggestions:', error);
                // Fall back to simulated suggestions
                const simulatedResults = generateSimulatedResults(query);
                displaySuggestions(simulatedResults);
            }
        }
        
        // Generate simulated results (backup if the API is unavailable)
        function generateSimulatedResults(query) {
            // List of major cities for demonstration
            const cities = [
                { name: 'New York', country: 'United States', lat: 40.7128, lon: -74.0060 },
                { name: 'London', country: 'United Kingdom', lat: 51.5074, lon: -0.1278 },
                { name: 'Paris', country: 'France', lat: 48.8566, lon: 2.3522 },
                { name: 'Tokyo', country: 'Japan', lat: 35.6762, lon: 139.6503 },
                { name: 'Sydney', country: 'Australia', lat: -33.8688, lon: 151.2093 },
                { name: 'Rio de Janeiro', country: 'Brazil', lat: -22.9068, lon: -43.1729 },
                { name: 'Cairo', country: 'Egypt', lat: 30.0444, lon: 31.2357 },
                { name: 'Mumbai', country: 'India', lat: 19.0760, lon: 72.8777 },
                { name: 'Beijing', country: 'China', lat: 39.9042, lon: 116.4074 },
                { name: 'Los Angeles', country: 'United States', lat: 34.0522, lon: -118.2437 },
                { name: 'Chicago', country: 'United States', lat: 41.8781, lon: -87.6298 },
                { name: 'Toronto', country: 'Canada', lat: 43.6532, lon: -79.3832 },
                { name: 'Berlin', country: 'Germany', lat: 52.5200, lon: 13.4050 },
                { name: 'Moscow', country: 'Russia', lat: 55.7558, lon: 37.6173 },
                { name: 'Singapore', country: 'Singapore', lat: 1.3521, lon: 103.8198 }
            ];
            
            // Filter cities that match the query
            const lowerQuery = query.toLowerCase();
            const filtered = cities.filter(city => 
                city.name.toLowerCase().includes(lowerQuery) || 
                city.country.toLowerCase().includes(lowerQuery)
            );
            
            // Format the results to match Nominatim API response structure
            return filtered.map(city => ({
                display_name: `${city.name}, ${city.country}`,
                lat: city.lat,
                lon: city.lon
            }));
        }
        
        // Display suggestions in the dropdown
        function displaySuggestions(suggestions) {
            if (suggestions.length === 0) {
                suggestionsContainer.style.display = 'none';
                return;
            }
            
            // Clear previous suggestions
            suggestionsContainer.innerHTML = '';
            
            // Add new suggestions
            suggestions.forEach(place => {
                const item = document.createElement('div');
                item.className = 'suggestion-item';
                item.textContent = place.display_name;
                
                item.addEventListener('click', function() {
                    locationInput.value = place.display_name;
                    selectedLocation = {
                        name: place.display_name,
                        coords: { lat: parseFloat(place.lat), lng: parseFloat(place.lon) }
                    };
                    
                    // Show coordinates
                    coordinatesDisplay.textContent = `Coordinates: ${place.lat}, ${place.lon}`;
                    
                    // Hide suggestions
                    suggestionsContainer.style.display = 'none';
                });
                
                suggestionsContainer.appendChild(item);
            });
            
            // Show suggestions
            suggestionsContainer.style.display = 'block';
        }
        
        // Form submission handler
        document.getElementById('location-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const personName = document.getElementById('person-name').value;
            const location = document.getElementById('location').value;
            const arrivalDate = document.getElementById('arrival-date').value;
            const notes = document.getElementById('notes').value;
            
            if (isEditMode) {
                // Show confirmation modal
                editModal.style.display = 'flex';
                return;
            }
            
            if (selectedLocation) {
                // Use the coordinates from the selected location
                addEntry(personName, selectedLocation.name, arrivalDate, notes, selectedLocation.coords);
                resetForm();
            } else {
                // Geocode the location if no suggestion was selected
                geocodeLocation(location, function(coords) {
                    if (coords) {
                        addEntry(personName, location, arrivalDate, notes, coords);
                        resetForm();
                    } else {
                        alert('Could not find coordinates for the specified location. Please select a location from the suggestions.');
                    }
                });
            }
        });
        
        // Submit the edit
        function submitEdit() {
            const id = editIdInput.value;
            const personName = document.getElementById('person-name').value;
            const location = document.getElementById('location').value;
            const arrivalDate = document.getElementById('arrival-date').value;
            const notes = document.getElementById('notes').value;
            
            if (selectedLocation) {
                updateEntry(id, personName, selectedLocation.name, arrivalDate, notes, selectedLocation.coords);
            } else {
                // Try to use existing coordinates if location name hasn't changed
                const existingEntry = entries.find(e => e.id === id);
                if (existingEntry && existingEntry.location === location) {
                    updateEntry(id, personName, location, arrivalDate, notes, existingEntry.coords);
                } else {
                    // Geocode the location if it has changed
                    geocodeLocation(location, function(coords) {
                        if (coords) {
                            updateEntry(id, personName, location, arrivalDate, notes, coords);
                        } else {
                            alert('Could not find coordinates for the specified location. Please select a location from the suggestions.');
                        }
                    });
                }
            }
            
            resetForm();
        }
        
        // Geocode location if no suggestion was selected
        function geocodeLocation(locationStr, callback) {
            fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(locationStr)}&limit=1`)
                .then(response => response.json())
                .then(data => {
                    if (data.length > 0) {
                        const coords = {
                            lat: parseFloat(data[0].lat),
                            lng: parseFloat(data[0].lon)
                        };
                        callback(coords);
                    } else {
                        // Fallback to simulated geocoding if API returns no results
                        fallbackGeocoding(locationStr, callback);
                    }
                })
                .catch(error => {
                    console.error('Error geocoding location:', error);
                    // Fallback to simulated geocoding if API fails
                    fallbackGeocoding(locationStr, callback);
                });
        }
        
        // Fallback geocoding function
        function fallbackGeocoding(locationStr, callback) {
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
            
            // Center the map on the new marker
            map.setView([coords.lat, coords.lng], 5);
        }
        
        // Update an existing entry
        function updateEntry(id, name, location, date, notes, coords) {
            // Find the entry in the array
            const entryIndex = entries.findIndex(e => e.id === id);
            if (entryIndex === -1) return;
            
            // Update entry data
            entries[entryIndex] = {
                id,
                name,
                location,
                date,
                notes,
                coords
            };
            
            // Update the marker
            const markerIndex = markers.findIndex(m => m.entryId === id);
            if (markerIndex !== -1) {
                // Remove old marker
                map.removeLayer(markers[markerIndex]);
                
                // Create new marker
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
                markers[markerIndex] = marker;
                
                // Center the map on the updated marker
                map.setView([coords.lat, coords.lng], 5);
            }
            
            // Update sidebar entry
            const entryCard = document.querySelector(`.entry-card[data-id="${id}"]`);
            if (entryCard) {
                entryCard.innerHTML = `
                    <h3>${name}</h3>
                    <p><strong>Location:</strong> ${location}</p>
                    <p><strong>Arrived:</strong> ${formatDate(date)}</p>
                    ${notes ? `<p><strong>Notes:</strong> ${notes}</p>` : ''}
                    <div class="card-buttons">
                        <button class="card-btn edit-btn">✎</button>
                        <button class="card-btn danger-btn">✕</button>
                    </div>
                `;
                
                // Re-add event listeners
                addEntryCardListeners(entryCard, entries[entryIndex]);
            }
            
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
                <div class="card-buttons">
                    <button class="card-btn edit-btn">✎</button>
                    <button class="card-btn danger-btn">✕</button>
                </div>
            `;
            
            addEntryCardListeners(entryCard, entry);
            
            entryList.appendChild(entryCard);
        }
        
        // Add event listeners to entry card
        function addEntryCardListeners(card, entry) {
            // Click on card to focus map
            card.addEventListener('click', function(e) {
                if (!e.target.classList.contains('card-btn')) {
                    // Find the marker
                    const marker = markers.find(m => m.entryId === entry.id);
                    if (marker) {
                        marker.openPopup();
                        map.setView(marker.getLatLng(), 5);
                    }
                }
            });
            
            // Edit button handler
            card.querySelector('.edit-btn').addEventListener('click', function(e) {
                e.stopPropagation();
                loadEntryForEdit(entry.id);
            });
            
            // Delete button handler
            card.querySelector('.danger-btn').addEventListener('click', function(e) {
                e.stopPropagation();
                removeEntry(entry.id);
            });
        }
        
        // Load entry data into form for editing
        function loadEntryForEdit(id) {
            const entry = entries.find(e => e.id === id);
            if (!entry) return;
            
            // Set form fields
            document.getElementById('person-name').value = entry.name;
            document.getElementById('location').value
