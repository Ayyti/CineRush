# CineRush 🎬
A high-concurrency cinema ticket booking backend engineered to handle 
simultaneous seat reservations without race conditions or double-bookings.

Built with system design principles at its core — distributed locking, 
event-driven architecture, and real-time communication.

---

## Tech Stack
- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** MongoDB (Mongoose)
- **Cache / Locking:** Redis (Redlock)
- **Message Broker:** Kafka
- **Real-time:** WebSockets
- **Auth:** JWT

---

## The Core Problem
During a high-demand show release, thousands of users attempt to book 
the same seats simultaneously. A naive implementation leads to:
- Double bookings (two users get the same seat)
- Inconsistent seat state across concurrent requests
- System crashes under traffic spikes

CineRush solves all three.

---

## Architecture

### 1. Three-State Seat Model
Every seat exists in one of three states:
```
AVAILABLE → LOCKED → BOOKED
```

- **AVAILABLE** — open for selection
- **LOCKED** — held by a user during payment (TTL: 10 mins)
- **BOOKED** — confirmed and paid

If payment is abandoned, the TTL expires and the seat returns to AVAILABLE automatically. No manual cleanup needed.

### 2. Redis Distributed Locking (Redlock)
When a user selects a seat, CineRush acquires a distributed lock via 
Redlock before changing seat state. This ensures:
- Only one request can lock a seat at a time
- Concurrent requests are rejected cleanly, not silently corrupted
- Lock releases automatically on TTL expiry

### 3. Kafka — Traffic Surge Handling
Booking requests are published to a Kafka topic rather than hitting 
the database directly. A consumer processes them asynchronously.

This decouples the booking request from the confirmation — the system 
stays stable even under extreme load (new show releases, flash sales).

### 4. WebSockets — Real-Time Seat Map
Every connected client receives live seat state updates. When User A 
locks seat G7, User B's seat map reflects that instantly — no refresh, 
no stale UI, no accidental selection of an already-held seat.

---

## Folder Structure
```
cinerush/
├── src/
│   ├── config/          # DB, Redis, Kafka connections
│   ├── controllers/     # Route handlers
│   ├── middleware/      # Auth, error handling
│   ├── models/          # MongoDB schemas (User, Show, Seat, Booking)
│   ├── routes/          # API routes
│   ├── services/        # Locking, Kafka producer/consumer, WebSocket
│   └── app.js
├── .env.example
├── package.json
└── README.md
```

---

## API Overview
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register user |
| POST | `/api/auth/login` | Login, returns JWT |
| GET | `/api/shows` | List all shows |
| GET | `/api/shows/:id/seats` | Get seat map for a show |
| POST | `/api/bookings/lock` | Lock a seat (Redis + TTL) |
| POST | `/api/bookings/confirm` | Confirm booking (Kafka event) |
| DELETE | `/api/bookings/release` | Release a locked seat |

---

## Status
🚧 **Active Development**

- [x] Architecture designed
- [ ] Core API + Auth
- [ ] Redis seat locking
- [ ] Kafka integration
- [ ] WebSocket real-time updates
- [ ] Load testing

---

## Why This Project
Most booking systems fail under concurrency. This project exists to 
solve that correctly — not with optimistic locking hacks, but with 
proper distributed systems patterns used in production at scale.
