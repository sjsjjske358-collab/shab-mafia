
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

const rooms = new Map();

app.get("/", (req, res) => {
  res.send("🌙 سرور شب مافیا روشن است!");
});

io.on("connection", socket => {
  socket.on("joinRoom", ({ roomId, name }, callback) => {
    if (
      typeof roomId !== "string" ||
      typeof name !== "string" ||
      !roomId.trim() ||
      !name.trim() ||
      name.length > 24
    ) {
      return callback?.({ ok: false, error: "اطلاعات نامعتبر" });
    }

    let room = rooms.get(roomId);

    if (!room) {
      room = new Map();
      rooms.set(roomId, room);
    }

    if (room.has(socket.id)) {
      return callback?.({ ok: true });
    }

    if (room.size >= 10) {
      return callback?.({ ok: false, error: "اتاق پر است" });
    }

    room.set(socket.id, name);
    socket.join(roomId);
    socket.data.roomId = roomId;

    io.to(roomId).emit("playersCount", room.size);
    callback?.({ ok: true, count: room.size });
  });

  socket.on("disconnect", () => {
    const roomId = socket.data.roomId;
    const room = rooms.get(roomId);

    if (!room) return;

    room.delete(socket.id);

    if (room.size === 0) {
      rooms.delete(roomId);
    } else {
      io.to(roomId).emit("playersCount", room.size);
    }
  });
});

server.listen(process.env.PORT || 3000, "0.0.0.0", () => {
  console.log("Shab Mafia Online!");
});
