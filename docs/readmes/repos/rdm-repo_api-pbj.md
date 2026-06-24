# PB&J Machine API

The backend that powers every sandwich. Handles order intake, assembly orchestration, machine communication, and operator management for PB&J Machine Co.

---

## Setup

### Prerequisites

- Node.js 18+
- PostgreSQL 14+
- A PB&J machine connected to your local network (or use the machine simulator for development)

### Install

```bash
git clone https://github.com/pbj-machine-co/pbj-api
cd pbj-api
npm install
cp .env.example .env
```

### Configure

Fill in your `.env`:

```bash
DATABASE_URL=postgresql://localhost:5432/pbj_dev
MACHINE_HOST=192.168.1.100        # IP of your PB&J machine (or simulator)
MACHINE_API_KEY=your-machine-key
JWT_SECRET=your-jwt-secret
```

### Run

```bash
# Start the dev server
npm run dev

# Run database migrations
npm run db:migrate

# Seed with sample data (ingredients, operator accounts)
npm run db:seed
```

The API starts on `http://localhost:3000`. Visit `/health` to confirm the server and machine are reachable.

---

## API Overview

Full API reference: [docs/api.md](docs/api.md)

### Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/orders` | Place a new sandwich order |
| `GET` | `/orders/:id` | Get order status and assembly phase |
| `GET` | `/orders` | List recent orders for an operator |

### Batch Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/batches` | Submit a batch order (10+ sandwiches) |
| `GET` | `/batches/:id` | Get batch progress |

### Machine

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/machine/health` | Ingredient inventory and machine status |
| `POST` | `/machine/calibrate` | Trigger a calibration cycle |

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/login` | Operator login, returns JWT |
| `POST` | `/auth/refresh` | Refresh an expired token |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for architecture overview, service boundaries, testing strategy, and how to run the machine simulator locally.
