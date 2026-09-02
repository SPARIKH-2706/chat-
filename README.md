# 💬 ChatApp — Real-Time Messaging Platform

A full-stack real-time chat application built with a **microservices architecture**. Three independent backend services handle user authentication, messaging, and email delivery, backed by a Next.js frontend.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    Frontend (Next.js)                │
│                    localhost:3000                    │
└──────────┬──────────────────┬───────────────────────┘
           │ REST API          │ WebSocket (Socket.io)
           ▼                  ▼
┌──────────────────┐  ┌──────────────────────────────┐
│  User Service    │  │       Chat Service            │
│  localhost:5000  │  │       localhost:5002           │
│                  │  │  (REST + Socket.io server)    │
│  - Auth (OTP)    │  │  - Chats & Messages           │
│  - JWT           │  │  - Image uploads (Cloudinary) │
│  - Redis (OTP)   │  │  - Real-time events           │
│  - RabbitMQ pub  │  └──────────────────────────────┘
└──────────────────┘
           │ RabbitMQ (send-otp queue)
           ▼
┌──────────────────────┐
│    Mail Service      │
│    localhost:5001    │
│                      │
│  - RabbitMQ consumer │
│  - Nodemailer/Gmail  │
└──────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 🎨 Frontend | Next.js 15, React 19, TypeScript, Tailwind CSS 4 |
| 👤 User Service | Node.js, Express 5, MongoDB (Mongoose), Redis, RabbitMQ, JWT |
| 💬 Chat Service | Node.js, Express 5, MongoDB (Mongoose), Socket.io, Cloudinary, Multer |
| ✉️ Mail Service | Node.js, Express, RabbitMQ, Nodemailer |
| 📨 Message Broker | RabbitMQ |
| ⚡ Cache / Rate Limiting | Redis |
| 🖼️ Media Storage | Cloudinary |

---

## ✨ Features

- 🔐 **Passwordless login** — OTP sent to email, verified against Redis (5-minute TTL, 60-second rate limit per email)
- ⚡ **Real-time messaging** — Socket.io with join/leave chat rooms, live message delivery, and seen receipts
- ⌨️ **Typing indicators** — `typing` / `stopTyping` events broadcast to chat room members
- 🟢 **Online presence** — tracks connected users in a `userSocketMap` and broadcasts live
- 🖼️ **Image messages** — upload images directly in chat, stored on Cloudinary
- 🔔 **Unseen message counts** — tracked per chat and updated in real time
- 👤 **Profile management** — users can update their display name

---

## 📁 Project Structure

```
chat code/
├── backend/
│   ├── user/          # User & auth service (port 5000)
│   ├── chat/          # Chat & messaging service (port 5002)
│   └── mail/          # Email delivery service (port 5001)
└── frontend/          # Next.js frontend (port 3000)
```

---

## ✅ Prerequisites

- 🟩 Node.js 18+
- 🍃 MongoDB (Atlas or local)
- ⚡ Redis instance
- 🐰 RabbitMQ instance (default: `localhost:5672`)
- ☁️ Cloudinary account
- 📧 Gmail account with an App Password

---

## 🔑 Environment Variables

### 👤 User Service (`backend/user/.env`)
```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
REDIS_URL=your_redis_url
Rabbitmq_Host=localhost
Rabbitmq_Username=admin
Rabbitmq_Password=admin123
JWT_SECRET=your_jwt_secret
```

### 💬 Chat Service (`backend/chat/.env`)
```env
PORT=5002
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
USER_SERVICE=http://localhost:5000
Cloud_Name=your_cloudinary_cloud_name
Api_Key=your_cloudinary_api_key
Api_Secret=your_cloudinary_api_secret
```

### ✉️ Mail Service (`backend/mail/.env`)
```env
PORT=5001
Rabbitmq_Host=localhost
Rabbitmq_Username=admin
Rabbitmq_Password=admin123
USER=your_gmail_address
PASSWORD=your_gmail_app_password
```

---

## 🚀 Getting Started

Install dependencies and start each service in a **separate terminal**.

### 1️⃣ User Service
```bash
cd backend/user
npm install
npm run dev
```

### 2️⃣ Chat Service
```bash
cd backend/chat
npm install
npm run dev
```

### 3️⃣ Mail Service
```bash
cd backend/mail
npm install
npm run dev
```

### 4️⃣ Frontend
```bash
cd frontend
npm install
npm run dev
```

🎉 The app will be available at **http://localhost:3000**

---

## 📡 API Reference

### 👤 User Service — `localhost:5000/api/v1`

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/login` | ❌ No | Send OTP to email |
| `POST` | `/verify` | ❌ No | Verify OTP, receive JWT |
| `GET` | `/me` | ✅ Yes | Get current user profile |
| `GET` | `/user/all` | ✅ Yes | List all users |
| `GET` | `/user/:id` | ❌ No | Get a user by ID |
| `POST` | `/update/user` | ✅ Yes | Update display name |

### 💬 Chat Service — `localhost:5002/api/v1`

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/chat/new` | ✅ Yes | Create or get a 1-on-1 chat |
| `GET` | `/chat/all` | ✅ Yes | Get all chats for current user |
| `POST` | `/message` | ✅ Yes | Send a text or image message |
| `GET` | `/message/:chatId` | ✅ Yes | Get messages for a chat (marks as seen) |

---

## 🔌 Socket.io Events

All socket connections go to the **Chat Service** (`localhost:5002`). Pass `userId` as a query parameter on connect.

### 📤 Client → Server

| Event | Payload | Description |
|---|---|---|
| `joinChat` | `chatId: string` | Join a chat room |
| `leaveChat` | `chatId: string` | Leave a chat room |
| `typing` | `{ chatId, userId }` | Notify others you're typing |
| `stopTyping` | `{ chatId, userId }` | Notify others you stopped typing |

### 📥 Server → Client

| Event | Payload | Description |
|---|---|---|
| `getOnlineUser` | `string[]` | List of currently online user IDs |
| `newMessage` | `message object` | A new message was sent to a chat |
| `messagesSeen` | `{ chatId }` | Messages in a chat were marked as seen |
| `userTyping` | `{ chatId, userId }` | A user started typing |
| `userStoppedTyping` | `{ chatId, userId }` | A user stopped typing |

---

## 🗄️ Data Models

### 👤 User
```
name: String
email: String (unique)
timestamps: true
```

### 💬 Chat
```
users: String[]        # array of user IDs
latestMessage: { text, sender }
timestamps: true
```

### 📩 Message
```
chatId: ObjectId (ref: Chat)
sender: String
text?: String
image?: { url, publicId }
messageType: "text" | "image"
seen: Boolean
seenAt?: Date
timestamps: true
```

---

## 🔐 Authentication Flow

1. 📧 User submits their email → User Service checks Redis rate limit → generates a 6-digit OTP → stores it in Redis (TTL: 5 min) → publishes to the `send-otp` RabbitMQ queue
2. 📨 Mail Service consumes the queue → sends the OTP via Gmail/Nodemailer
3. 🔑 User submits email + OTP → User Service verifies against Redis → creates user if new → returns a JWT
4. 🛡️ All protected routes expect `Authorization: Bearer <token>` in the request header

---

<div align="center">

Made with ❤️ using Node.js, Next.js & Socket.io

</div>
