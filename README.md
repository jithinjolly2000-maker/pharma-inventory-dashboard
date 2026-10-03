# Pharma Inventory Intelligence — Prototype

A working prototype of an **inventory-planning dashboard for a pharmaceutical distribution business**. It identifies slow-moving stock, recommends reorders, and flags products at risk of expiry — with role-based views for billers, purchasers, accounts and management.

Built as a proof-of-concept to demonstrate the approach on **sample data**. In production it reads live from the ERP / SQL database via API.

> **Live demo:** open `index.html` in any browser — no setup, no install. Runs entirely client-side on dummy data (no real records).

---

## The problem it solves

Pharmaceutical distributors lose money two ways that are hard to see day to day:
1. **Dead capital** tied up in slow-moving stock.
2. **Write-offs** when batches expire before they sell.

Both are preventable with the data distributors already have — it just isn't surfaced in time, to the right person, with a clear action attached. This dashboard does exactly that.

---

## Four role-based views

| View | For | What it does |
|------|-----|--------------|
| 🔄 **Reorder** | Billers, Purchasers | Predicts demand over each supplier's **lead time + safety stock**; a rule flags "reorder now" and suggests a quantity, which a human confirms (rule **+** judgement, as the business actually works). |
| 🐢 **Slow-moving** | Purchasers, Management | Flags SKUs where **days-of-cover** exceeds threshold, plus a trend bar — not just a flat cutoff. Shows capital tied up. |
| ⏳ **Expiry risk** | Billers, Accounts | Auto-flags every batch **3 months before expiry**, prioritised by whether current **sell-through** will clear it in time. Builds the **"return for credit note"** action list per supplier, with value at risk. |
| 📊 **Management** | Leadership | Roll-up of stock value, reorder volume and **expiry exposure** by supplier. |

---

## The logic (the part a generic plan misses)

A dashboard layout is easy. These are the decisions that make it actually work in a pharma distribution setting:

- **Reorder respects expiry, not just stock levels** — reordering a slow mover that will expire before it sells just moves the write-off. Reorder and expiry logic are linked.
- **The 3-month expiry flag is weighted by sell-through** — a batch 3 months out that is moving fine should not scream the same as one with zero movement, or users start ignoring the alerts.
- **Returns are supplier-aware** — return windows and credit terms differ per supplier, so the action list is tied to the supplier and value.
- **Four user types = four definitions of "urgent"** — each role gets its own filtered view, or they tune it out.

---

## How it works

```
ERP / SQL database ──API──► Data layer ──► Rules engine ──► Role-based dashboard
   (stock, sales,            (one clean      (reorder /        (Reorder · Slow ·
    lead times, expiry)       table)          expiry / slow)    Expiry · Mgmt)
```

**Reorder point** = (avg daily sales × supplier lead time) + safety stock
**Expiry risk** = flagged when days-to-expiry ≤ 90; "won't clear" = stock − (avg daily sales × days-to-expiry)
**Slow-moving** = days-of-cover (stock ÷ avg daily sales) above threshold

---

## Tech

| Prototype (this repo) | Production path |
|-----------------------|-----------------|
| Single-file HTML / CSS / JS, runs in any browser | ERP/SQL via API → Power BI / Tableau, or a lightweight web app |
| Dummy sample data inline | Live data, scheduled refresh |
| Rules only | Rules first, then a demand forecast once sales history is available |

Deliberately client-side and dependency-free so anyone can open it and see the idea working in seconds.

---

## Data it expects (from the ERP)

Per SKU / batch: `stock on hand`, `average daily sales`, `supplier`, `supplier lead time`, `min/max levels`, `unit cost`, `batch expiry date`.

---

## Roadmap

- [ ] Connect to a real ERP export (CSV/API) instead of inline sample data
- [ ] Demand forecast (replace flat avg-daily-sales with a trend/seasonal model)
- [ ] Email/Slack alerts for urgent reorder + expiry actions
- [ ] Supplier return-window rules table
- [ ] Export action lists to Excel / PO system

---

*Built by Jithin J — [LinkedIn](https://www.linkedin.com/in/jithin2k). Prototype on sample data; no real or confidential records are used.*
