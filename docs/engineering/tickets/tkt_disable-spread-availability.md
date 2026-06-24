# feat: temporarily disable a spread from the ordering ui

**Ticket:** ENG-314
**Type:** Feature
**Priority:** Medium
**Created:** 2026-03-06

## 🔴 The Problem

When a spread becomes temporarily unavailable, admins have no way to remove it from the ordering UI without deleting it from the registry entirely.

## 📍 Experience Today

```
Supplier calls — sunflower butter is backordered for two weeks
→ Admin opens the admin panel
→ No disable option exists
→ Only option is to delete the spread from the registry
→ Customers can still order sunflower butter in the meantime
→ When stock returns, admin must re-add and re-validate from scratch
```

## ✅ Experience After

```
Supplier calls — sunflower butter is backordered for two weeks
→ Admin opens Spread Management in the admin panel
→ Toggles sunflower butter to disabled
→ Spread disappears from the ordering UI immediately
→ Registry entry, config, and history all preserved
→ When stock returns, admin toggles it back — no reconfiguration needed
```
