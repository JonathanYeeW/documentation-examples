# PB&J Operator Dashboard

The web interface for managing PB&J Machine Co. machines at scale. Built for school districts, food banks, and cafeteria operators who need to schedule batch orders, monitor active machines, and review order history across multiple locations.

Built with Next.js and deployed at [dashboard.pbj.co](https://dashboard.pbj.co).

---

## Setup

### Prerequisites

- Node.js 18+
- A running instance of the [PB&J Machine API](https://github.com/pbj-machine-co/pbj-api)

### Install

```bash
git clone https://github.com/pbj-machine-co/pbj-dashboard
cd pbj-dashboard
npm install
```

### Configure

Create a `.env.local` file at the project root:

```bash
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_POSTHOG_KEY=your-posthog-key   # optional, for analytics
```

### Run

```bash
# Development
npm run dev

# Production build
npm run build
npm start
```

The dashboard runs on `http://localhost:4000`.

---

## Features

**Order Management** — Place single and batch orders, track active assembly jobs, and view order history by location or date range.

**Machine Health** — Real-time ingredient inventory across all machines in your fleet. Alerts surface low stock and expired ingredients before they block orders.

**Batch Scheduling** — Upload a CSV to schedule a full week of orders in one step. Preview the schedule before committing.

**Multi-location** — Switch between machines in your fleet from a single account. Order history and inventory are scoped per machine.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for local development setup, component conventions, and how to run the app against the machine simulator.
