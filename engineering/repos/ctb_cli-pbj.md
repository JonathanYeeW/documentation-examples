# Contributing to pbj-cli

Thanks for contributing to PB&J Machine Co.'s operator CLI. This document covers architecture, local development setup, and testing guidelines.

---

## Architecture

### Command Routing

`src/index.ts` is the entry point. Dispatch follows two phases:

1. `handleUnauthenticatedCommand(command)` — handles `health` and no-args. Both call `process.exit()` directly.
2. If not handled, `authService.authenticate()` is called, then `handleAuthenticatedCommand(command, args)` dispatches to the appropriate handler.

### Directory Structure

```
src/
├── index.ts
├── commands/
│   ├── order/
│   │   ├── order.command.ts         # Wizard UI — prompts, display
│   │   ├── order.handler.ts         # Orchestration — coordinates service calls
│   │   └── order.service.ts         # API boundary — POST /orders
│   ├── batch/
│   │   ├── batch.command.ts         # CSV parsing, confirmation UI
│   │   ├── batch.handler.ts         # Orchestration — validate, submit, poll
│   │   └── batch.service.ts         # API boundary — POST /batches
│   ├── status/
│   │   ├── status.command.ts        # Display loop — render active orders
│   │   └── status.service.ts        # API boundary — GET /orders
│   ├── health.command.ts            # Self-contained — no handler/service needed
│   └── print-usage.ts
└── lib/                             # Shared infrastructure — no domain knowledge
    ├── auth.ts                      # Sign-in, token storage, authenticated request wrapper
    ├── command-router.ts            # Two-phase command dispatch
    ├── constants.ts                 # Environment variable loading and validation
    ├── error-handler.ts             # Top-level error formatting and process.exit(1)
    └── logger.ts                    # stdout formatting conventions
```

The rule: anything in `lib/` has no knowledge of any specific command or domain. If it's in a command folder, it's owned by that command.

### Auth Flow

All authenticated commands go through `authService` in `src/lib/auth.ts`:

1. `authService.authenticate()` — POSTs credentials to `/auth/sign-in/email/`. Stores `accessToken`, `refreshToken`, and `User` in memory.
2. `authService.makeAuthenticatedRequest(url, options)` — Attaches `Authorization: Bearer <token>` and `X-API-Key` headers to every request. On a 401 or 403, automatically attempts a token refresh before retrying. If the refresh fails, clears auth state and throws.

---

## Local Development

### Run Without Building

```bash
npm run dev <args>
```

### Build

```bash
npm run build
```

### Machine Simulator

For development without a physical machine, start the simulator before running commands:

```bash
npm run simulator
```

The simulator runs on `localhost:9000` and mimics the machine API, including assembly phases, quality gate failures, and ingredient inventory.

### Man Page

The CLI ships a Unix man page at `man/st.1`. After updating commands, regenerate it:

```bash
sudo npm run update-man
```

See `man/README.md` for man page maintenance details.

---

## Testing

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch
```

Tests are colocated with their commands. A test for `order.handler.ts` lives at `order.handler.test.ts` in the same directory. Integration tests that hit the machine simulator live in `tests/integration/`.

New commands require unit tests for the handler and integration tests for the full command flow against the simulator.
