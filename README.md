# Crypto CEX

A centralized crypto exchange backend built with TypeScript, Express, and Bun. Supports user authentication, USD onramp, asset deposits, and limit order matching via an in-memory orderbook.

## Tech Stack

- [Bun](https://bun.sh) — runtime and test runner
- [Express](https://expressjs.com) — HTTP server
- [TypeScript](https://www.typescriptlang.org) — type safety
- JWT — authentication for protected routes

## Getting Started

Install dependencies

```bash
bun install
```

Start the server:

```bash
bun run src/index.ts
```

The server listens on **port 3002**.

## Project Structure

```
src/
├── index.ts           # Express app entry point
├── index.test.ts      # Integration tests
├── orderbook.ts       # In-memory orderbook with bid/ask matching
├── middleware/
│   └── index.ts       # JWT auth middleware
├── routes/
│   └── user.ts        # User, balance, and trading routes
└── types/
    └── user.ts        # Request/response type definitions
```

## API Endpoints

### Public

| Method | Endpoint   | Description              |
|--------|------------|--------------------------|
| POST   | `/signup`  | Create a new user        |
| POST   | `/signin`  | Sign in and receive JWT  |

### Protected (requires `Authorization: Bearer <token>`)

| Method | Endpoint                  | Description                        |
|--------|---------------------------|------------------------------------|
| GET    | `/balance`                | Get USD and asset balances         |
| POST   | `/onramp`                 | Add USD to account                 |
| POST   | `/deposit/:asset_symbol`  | Deposit an asset (e.g. `SOL`)      |
| POST   | `/order`                  | Place a bid or ask limit order     |

### Example Requests

**Sign up**

```bash
curl -X POST http://localhost:3002/signup \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "password": "secret"}'
```

**Sign in**

```bash
curl -X POST http://localhost:3002/signin \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "password": "secret"}'
```

**Get balance**

```bash
curl http://localhost:3002/balance \
  -H "Authorization: Bearer <token>"
```

**Onramp USD**

```bash
curl -X POST http://localhost:3002/onramp \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"qty": 100}'
```

**Deposit asset**

```bash
curl -X POST http://localhost:3002/deposit/SOL \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"qty": 10}'
```

**Place order**

```bash
curl -X POST http://localhost:3002/order \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"side": "bid", "qty": 1, "price": 50, "asset": "sol"}'
```

## Testing

Run integration tests with Bun:

```bash
bun test src/index.test.ts
```

> **Note:** Tests expect the server to be running on port `3001`. Start the server on that port before running tests, or update `BACKEND_URL` in `src/index.test.ts`.

## Orderbook

The `Ordebook` class handles limit order matching for supported assets (currently `sol`). When a bid or ask is placed

1. The orderbook attempts to match against existing orders at compatible prices.
2. Fills update buyer/seller balances immediately.
