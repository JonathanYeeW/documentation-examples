# PB&J Operator CLI (`pbj`)

Manage your PB&J machine from the terminal. Submit batch orders for the week, monitor active assembly jobs, and check machine health — without opening the dashboard.

Built for cafeteria operators and food bank coordinators who prefer a fast, scriptable interface over a browser.

---

## Setup

### Prerequisites

- Node.js 18+
- A PB&J Machine Co. operator account with an API key

### Install

```bash
git clone https://github.com/pbj-machine-co/pbj-cli
cd pbj-cli
npm install
npm run build
npm link
```

### Configure

Create a `.env` file at the project root:

```bash
API_BASE_URL=https://api.pbj.co
OPERATOR_EMAIL=your@organization.org
OPERATOR_ID=your-operator-id
API_KEY=your-64-character-hex-api-key
```

Run `pbj health` to confirm your machine is reachable and your credentials are valid.

---

## Commands

### `pbj order`

Interactive wizard for placing a sandwich order.

Prompts for quantity, bread type, spread, jelly, and optional dietary flags (nut-free, gluten-free). Single orders go to the machine immediately. Batch orders above 10 are scheduled — use `pbj status` to monitor progress.

```bash
pbj order
```

---

### `pbj batch`

Submit a batch order from a CSV file.

```bash
pbj batch --file schedule.csv
```

The CSV format is one order per row: `quantity,bread,spread,jelly,dietary_flags`. See `examples/batch_template.csv` for a complete example. Useful for scheduling a full week of lunch orders in one command.

---

### `pbj status`

Check the status of active and recent orders.

```bash
pbj status
```

Shows queue position, current assembly phase, and estimated completion time for each active order. Completed orders from the last 24 hours are included.

---

### `pbj health`

Verify machine connectivity, ingredient inventory, and API authentication.

```bash
pbj health
```

Runs ingredient checks on bread, spreads, and jelly. Flags anything low or expired. Safe to run before every shift.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for architecture overview, local development setup, and testing guidelines.
