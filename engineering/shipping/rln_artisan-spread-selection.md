# v1.5.0 — Spread Availability Controls, Order History, and Bug Fixes

**Released:** 2026-03-08  
**Tag:** `v1.5.0`  
**Repo:** pbj-cli

## What's New

Operations teams can now temporarily disable a spread from the ordering UI without removing it from the registry — useful when a supplier is backordered and the spread will be coming back. This release also adds order history so customers can review past sandwiches, fixes a long-standing bug where spread selection was lost on back-navigation, and resolves two reliability issues in the assembly pipeline.

## Changes

**Features**
- Admins can now toggle spread availability from the admin panel — disabled spreads are hidden from `SpreadSelector` immediately, registry entry and config are preserved
- New `GET /orders/history` endpoint returns a paginated list of past orders for the authenticated user

**Bug Fixes**
- Fixed spread selection being cleared when navigating back through the ordering flow (ENG-313)
- Fixed `temperSpread` blocking indefinitely on slow-to-temper spreads — now times out after 10 minutes and proceeds with a warning
- Fixed assembly pipeline swallowing `SpreadApplicationError` silently — error now surfaces correctly and fails the order

## Upgrading

No breaking changes. `GET /orders/history` is additive — existing clients are unaffected.

If you're using the admin panel, the new availability toggle appears in Spread Management. No config changes needed.

```bash
npm update pbj-cli
```
