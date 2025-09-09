# Vartalap — Server (Express + Socket.IO + MongoDB)

Backend for a realtime chat application. Provides user auth (JWT via cookie or Bearer header), user discovery, messaging, and Socket.IO events for online presence and new messages.

## Tech Stack
- Express 4
- Socket.IO 4
- MongoDB with Mongoose 8
- JWT (cookies or `Authorization: Bearer`)
- CORS, cookie-parser, dotenv, bcryptjs

## Features
- User register/login/logout with JWT and httpOnly cookie.
- Get own profile and list other users.
- Send and fetch conversation messages.
- Realtime: broadcasts `onlineUsers`; pushes `newMessage` to recipients.

## Project Structure
```
server/
  server.js             # App entry (mounts routes, middleware, starts HTTP)
  socket/socket.js      # Express app + HTTP server + Socket.IO setup
  db/connection1.db.js  # Mongo connection
  models/               # User, Message, Conversation schemas
  controllers/          # user.controller, message.controller
  routes/               # /api/v1/user, /api/v1/message
  middlewares/          # auth, error
  utilities/            # asyncHandler, errorHandler
```

## Environment Variables
Create `server/.env` with:

```
PORT=5000
MONGODB_URL=<your-mongodb-connection-string>
JWT_SECRET=<strong-secret>
JWT_EXPIRES=7d
COOKIE_EXPIRES=7
CLIENT_URL=http://localhost:3000
```

Notes:
- Cookies are set `httpOnly`, `secure: true`, `sameSite: 'None'`. For local HTTP development, prefer sending the JWT as a Bearer token header.
- CORS allows `origin: CLIENT_URL` with `credentials: true`.

## Install & Run
```
cd server
npm install
npm run dev
```

## Authentication
Middleware reads JWT from cookie `token` or `Authorization: Bearer <token>`.

```
Authorization: Bearer <jwt>
```

## REST API
Base URL: `/api/v1`

User
- POST `/user/register` — body: `{ fullName, username, password, gender }`
  - Sets cookie `token`; returns `{ newUser, token }`.
- POST `/user/login` — body: `{ username, password }`
  - Sets cookie `token`; returns `{ user, token }`.
- POST `/user/logout` — auth required
  - Clears cookie.
- GET `/user/get-profile` — auth required
  - Returns current user profile.
- GET `/user/get-other-users` — auth required
  - Returns all users except current.

Message
- POST `/message/send/:receiverId` — auth required
  - body: `{ message }`; creates message and emits `newMessage` to receiver.
- GET `/message/get-messages/:otherParticipantId` — auth required
  - Returns conversation with populated `messages`.

Error format (middleware):
```
{ "success": false, "errMessage": string }
```

## Socket.IO
Server: created in `server/socket/socket.js` and shared with Express app.

Client connect example:
```js
import { io } from 'socket.io-client'
const socket = io('<SERVER_URL>', { query: { userId: '<currentUserId>' } })

socket.on('onlineUsers', (ids) => { /* array of userIds */ })
socket.on('newMessage', (msg) => { /* push into UI */ })
```

Events
- `connection` — expects `query.userId` to register presence.
- `onlineUsers` — broadcast of online user IDs.
- `newMessage` — emitted to a receiver when a message is sent.

## Models
- User: `fullName`, `username` (unique), `password`, `gender`, `avatar`.
- Message: `senderId`, `receiverId`, `message` (+timestamps).
- Conversation: `participants[]`, `messages[]` (+timestamps).

## Scripts
- `npm run dev` — start with Nodemon.

## License
ISC — Author: MKL
