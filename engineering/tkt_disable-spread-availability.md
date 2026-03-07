# Temporarily Disable a Spread from the Ordering UI

**Ticket:** ENG-314
**Type:** Feature
**Priority:** Medium
**Reporter:** Operations Team
**Created:** 2026-03-06

## 👤 Admin Journey

An operations admin gets a call from the supplier — sunflower butter is backordered for the next two weeks. The spread is still in the registry, still configured correctly, and will come back. But right now we can't fulfill orders that include it. The admin needs to pull it from the ordering UI immediately so customers stop selecting it, without deleting the registry entry or touching any code.

Right now there's no way to do this. The only option is to remove the spread from the registry entirely, which means re-adding it and re-validating the config when it comes back in stock. That's error-prone and creates unnecessary risk around a spread that isn't going away — it's just temporarily unavailable.

## 🔄 Current vs. Expected

```
Current:

  Admin learns sunflower butter is backordered
  → No disable option exists
  → Must delete spread from registry to remove it from UI
  → Must re-add and re-validate when stock returns

Expected:

  Admin learns sunflower butter is backordered
  → Admin opens Spread Management in admin panel
  → Toggles "sunflower_butter" to disabled
  → Spread disappears from customer ordering UI immediately
  → Registry entry, config, and history all preserved
  → Admin toggles it back when stock returns — no re-configuration needed
```

## 🐛 Problem

The spread registry has no concept of availability. A spread is either in the registry or it isn't — there's no middle state for "configured but temporarily not orderable." Handling a supplier interruption today requires destructive registry changes that need to be undone later.

## ✅ Acceptance Criteria

```
1. Each spread in the registry has an enabled/disabled state
   → Default is enabled for all existing and new spreads

2. Admin can toggle a spread's availability from the admin panel
   → Change takes effect immediately in the ordering UI
   → No deployment or code change required

3. Disabled spreads do not appear in SpreadSelector
   → Customers cannot select a disabled spread
   → Customers mid-order who had selected a disabled spread
     are prompted to choose a different option

4. Disabled spreads remain fully intact in the registry
   → Config, handling rules, and history are preserved
   → Re-enabling restores the spread with no reconfiguration

5. The last enabled spread cannot be disabled
   → Admin sees an error: "At least one spread must remain available"
```
