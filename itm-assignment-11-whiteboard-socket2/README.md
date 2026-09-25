# 🎨 Real-Time Collaborative Whiteboard & Canvas

A multi-user, multi-room collaborative whiteboard built with **Node.js**, **Express.js**, **Socket.io**, and the **HTML5 Canvas API**. Multiple users can draw on the same board simultaneously, see each other's live cursors, and clear/undo strokes in sync.

## ✨ Features

- **Multi-room support** — join any board via `?board=<id>`, isolated with `socket.join(boardId)`.
- **In-memory stroke history buffer** per room — new joiners instantly receive the full drawing via `board:init`.
- **Real-time stroke streaming** — every line segment is broadcast to all other peers in the room as it's drawn.
- **Live collaborator cursors** — throttled cursor broadcasting (client: 40ms, server: 30ms floor) with colored, labeled cursor indicators that auto-hide after inactivity.
- **Clear board** — `board:clear` wipes the shared history and instantly clears every connected peer's canvas.
- **Undo** — removes the last *continuous stroke action* (all segments sharing a client-generated `strokeId`), then re-syncs every peer via `board:sync`.
- **User presence** — join/leave toasts and a live user chip list per room.
- **Touch support** for tablets, plus canvas resize handling that redraws from the in-memory buffer.

## 🛠️ Tech Stack

- Node.js + Express.js (static file serving + small REST health/debug endpoints)
- Socket.io (WebSocket transport with polling fallback)
- HTML5 Canvas API (vanilla JS, no frontend framework)
- CORS + dotenv for configuration

## 📁 Project Structure

```
itm-assignment-11-whiteboard-socket/
├── public/
│   ├── index.html        # Join screen + whiteboard UI shell
│   ├── canvas.js          # Client drawing engine & socket event wiring
│   └── styles.css         # Toolbar, join card, canvas & cursor styling
├── sockets/
│   ├── store.js           # In-memory boardRooms state + helper functions
│   ├── boardHandler.js    # board:join, draw:stroke, board:clear, draw:undo
│   └── cursorHandler.js   # cursor:move / cursor:update + disconnect cleanup
├── server.js               # Express + Socket.io bootstrap
├── package.json
├── .env.example
└── README.md
```

## 🚀 Getting Started

```bash
# 1. Install dependencies
npm install

# 2. (optional) copy env config
cp .env.example .env

# 3. Run in dev mode (auto-restart)
npm run dev

# ...or run in production mode
npm start
```

The server starts at **http://localhost:5000** by default (configurable via `PORT` in `.env`).

## 🧪 Testing the Real-Time Sync

1. Open **http://localhost:5000?board=demo** in one browser window.
2. Enter a name + color and click **Join Board**.
3. Open a second window (or incognito tab) at the same URL and join with a different name.
4. Draw in Window 1 → the stroke should appear instantly in Window 2.
5. Move your mouse in Window 1 → a colored, labeled cursor should glide across Window 2.
6. Open a **third** window on the same board URL → it should immediately render all prior strokes (via `board:init`).
7. Click **Clear** in any window → all windows wipe instantly.
8. Draw a few strokes, then click **Undo** → only the most recent continuous stroke is removed, everywhere, via a full `board:sync`.

## 🔄 Socket Event Protocol

| Event | Direction | Payload | Purpose |
|---|---|---|---|
| `board:join` | Client → Server | `{ boardId, username, userColor }` | Join a room |
| `board:init` | Server → Client | `{ strokes, activeUsers }` | Full state sync for new joiner |
| `user:joined` / `user:left` | Server → Room | `{ userId, username, color? }` | Presence notifications |
| `draw:stroke` | Client → Server | `{ boardId, stroke }` | Append + relay a line segment |
| `draw:broadcast` | Server → Room | `{ stroke }` | Relay stroke to peers |
| `cursor:move` | Client → Server | `{ boardId, x, y }` | Throttled pointer position |
| `cursor:update` | Server → Room | `{ userId, x, y, ... }` | Relay peer cursor |
| `board:clear` | Client → Server | `{ boardId }` | Request full wipe |
| `board:cleared` | Server → Room | `{ clearedBy }` | Wipe local canvases |
| `draw:undo` | Client → Server | `{ boardId }` | Undo last stroke action |
| `board:sync` | Server → Room | `{ strokes }` | Post-undo state snapshot |

## 🧠 Design Notes

- **Stroke grouping for undo**: the client generates a `strokeId` (UUID-ish) on `mousedown` and stamps every segment of that pen-down-to-pen-up motion with it. The server tracks distinct `strokeId`s in chronological order (`strokeOrder`) so `draw:undo` can pop and remove *only* the most recent full stroke, not just the last segment.
- **Memory model**: state lives in a single `boardRooms` object (see `sockets/store.js`). This is intentionally simple or "in-memory only" per the assignment scope — for horizontal scaling you'd swap this for Redis + the `socket.io-redis` adapter.
- **Cursor throttling**: throttled on both the client (40ms) and server (30ms floor) to protect against high-frequency event floods from fast mouse movement or a misbehaving client.
- **Cleanup**: on `disconnect`, a user is removed from every board they were in and `user:left` is broadcast; empty boards with no strokes are garbage-collected from memory.

## 📤 Submission

- GitHub repo: `itm-assignment-11-whiteboard-socket`
- Demo: run locally and open two+ browser windows on the same `?board=` URL to verify real-time sync (see Testing section above).
