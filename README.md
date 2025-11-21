# website-design
const express = require("express");
const app = express();
const http = require("http").createServer(app);
const io = require("socket.io")(http);

app.use(express.static("public"));

let players = {};

io.on("connection", (socket) => {
    console.log("A player connected:", socket.id);

    // Initialize player
    players[socket.id] = { x: 200, y: 200 };

    // Send current players
    socket.emit("currentPlayers", players);

    // Broadcast new player
    socket.broadcast.emit("newPlayer", { id: socket.id, x: 200, y: 200 });

    // Handle movement
    socket.on("move", (data) => {
        players[socket.id].x = data.x;
        players[socket.id].y = data.y;
        io.emit("playerMoved", { id: socket.id, x: data.x, y: data.y });
    });

    // Disconnect
    socket.on("disconnect", () => {
        delete players[socket.id];
        io.emit("playerDisconnected", socket.id);
        console.log("Player disconnected:", socket.id);
    });
});

http.listen(3000, () => {
    console.log("Server running on http://localhost:3000");
});

