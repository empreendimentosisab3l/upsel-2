# Fix: Typebot Quiz Flow Blocked by MutationObserver

**Date:** 2026-04-03
**Project:** upsel-2 (upsell.monjarogelatina.shop)

## Problem

The Typebot quiz embedded in `#root` stops responding mid-flow when users click quiz answer buttons. The quiz never reaches the final 2 CTA buttons.

### Root Cause

The `MutationObserver` in `index.html` intercepts buttons using `textContent.indexOf()`, which matches **parent containers** (not just leaf buttons) because `textContent` includes all children's text recursively. When the observer hijacks a parent container with `stopImmediatePropagation` in **capture phase**, it blocks ALL click events on every element inside that container — including Typebot's own quiz navigation buttons.

## Design

### Three surgical changes to the MutationObserver script

**Change 1 — Exact text match on leaf elements only**

- Before: `el.textContent.indexOf('CLIQUE AQUI PARA PARTICIPAR') !== -1`
- After: `el.textContent.trim() === 'CLIQUE AQUI PARA PARTICIPAR' && !el.querySelector('a, button')`

Same fix for the downsell button: `btnText === 'quero perder essa oportunidade' && !btn.querySelector('a, button')`

**Change 2 — Remove `e.stopImmediatePropagation()`**

This call in capture phase kills ALL handlers on the element and its children. It is not needed since we trigger `paytUpsellBtn.click()` directly.

**Change 3 — Remove `e.stopPropagation()`**

Also not needed. Only `e.preventDefault()` is necessary to prevent default anchor/button behavior.

## Files Changed

- `index.html` — MutationObserver script block only

## Success Criteria

- Typebot quiz flows through all steps without getting stuck
- Final "CLIQUE AQUI PARA PARTICIPAR" button triggers Payt one-click buy
- Final "quero perder essa oportunidade" button opens downsell popup
- No other behavior changes
