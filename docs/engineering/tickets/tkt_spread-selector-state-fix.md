# bug: fix spread selector to persist selection across navigation

**Ticket:** ENG-313
**Type:** Bug
**Priority:** Medium
**Created:** 2026-03-06

## 🔴 The Problem

The spread selector loses its selection when the user navigates away and returns, because it stores state locally instead of writing to shared order state.

## 📍 Experience Today

```
User selects sunflower butter
→ Navigates forward to confirmation screen
→ Decides to change bread — navigates back
→ Changes bread selection
→ Navigates forward through jelly to spread
→ Spread selection is gone — must pick again
→ If user doesn't notice, order fails at assembly with no spread
```

## ✅ Experience After

```
User selects sunflower butter
→ Navigates forward to confirmation screen
→ Decides to change bread — navigates back
→ Changes bread selection
→ Navigates forward through jelly to spread
→ Sunflower butter is still selected — no action needed
→ User continues to confirmation without interruption
```
