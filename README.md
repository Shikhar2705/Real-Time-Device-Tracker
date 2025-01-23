# Real-Time Device Tracker

This project is a real-time device tracking application that uses Node.js, Express, Socket.io, and Leaflet.js to track and display the location of devices on a map.

## Features

- Real-time location tracking
- Display multiple device locations on a map
- Remove device markers when a user disconnects

## Installation

1. Clone the repository:
    ```bash
    git clone <repository-url>
    ```
2. Navigate to the project directory:
    ```bash
    cd RealTime-Device-Tracker
    ```
3. Install the dependencies:
    ```bash
    npm install
    ```

## Usage

1. Start the server:
    ```bash
    npm start
    ```
2. Open your browser and navigate to `http://localhost:3000`.

## Project Structure

- `app.js`: The main server file that sets up the Express server and Socket.io.
- `views/index.ejs`: The main HTML file that includes the map and scripts.
- `public/js/script.js`: The client-side JavaScript file that handles geolocation and Socket.io communication.

## Code Explanation

### app.js

```javascript
// filepath: /c:/Users/shikh/OneDrive/Documents/RealTime Device Tracker/app.js
const express = require('express');
// ...existing code...
const io = socketio(server);

app.set("view engine", "ejs");
app.use(express.static(path.join(__dirname, "public")));

io.on("connection", function(socket) {
    socket.on("send-location", function(data) {
        io.emit("receive-location", {id: socket.id, ...data});
    });
    socket.on("disconnect", function() {
        console.log("User Disconnected - ", socket.id);
        io.emit("user-disconnected", socket.id);
    });
});

app.get('/', function(req, res) {
    res.render("index");
});

server.listen(3000);
```

### views/index.ejs

```html
<!-- filepath: /c:/Users/shikh/OneDrive/Documents/RealTime Device Tracker/views/index.ejs -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Real-Time Tracking App</title>
    <link rel="stylesheet" href="/css/style.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css" integrity="sha512-h9FcoyWjHcOcmEVkxOfTLnmZFWIH0iZhZT1H2TbOq55xssQGEJHEaIm+PgoUaZbRvQTNTluNOEfb1ZRy6D3BOw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
</head>
<body>
    <div id="map"></div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js" integrity="sha512-puJW3E/qXDqYp9IfhAI54BJEaWIfloJ7JWs7OeD5i6ruC9JZL1gERT1wjtwXFlh7CjE7ZJ+/vcRZRkIYIb6p4g==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
    <script src="https://cdn.socket.io/4.8.1/socket.io.min.js" integrity="sha384-mkQ3/7FUtcGyoppY6bz/PORYoGqOl7/aSUMn2ymDOJcapfS6PHqxhRTMh1RR0Q6+" crossorigin="anonymous"></script>
    <script src="/js/script.js"></script>
</body>
</html>
```

### public/js/script.js

```javascript
// filepath: /c:/Users/shikh/OneDrive/Documents/RealTime Device Tracker/public/js/script.js
const socket = io();

if (navigator.geolocation) {
    navigator.geolocation.watchPosition((position) => {
        const {latitude , longitude} = position.coords;
        socket.emit("send-location", {latitude , longitude});
    }, (err) => {
        console.error(err);
    },
    {
        enableHighAccuracy: true,
        timeout:10000,
        maximumAge: 0
    });
}

const map = L.map("map").setView([0,0],10);

L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {}).addTo(map);

const markers = {};

socket.on("receive-location", (data) => {
    const {id , latitude, longitude} = data;
    map.setView([latitude,longitude],20);

    if (markers[id]) {
        markers[id].setLatLng([latitude, longitude]);
    } else {
        markers[id] = L.marker([latitude, longitude]).addTo(map);
    }
});

socket.on("user-disconnected" , (id) => {
    if (markers[id]) {
        map.removeLayer(markers[id]);
        delete markers[id];
    }
});
```

## License

This project is licensed under the MIT License.
