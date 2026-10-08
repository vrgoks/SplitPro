# SplitPro

A single-file, zero-dependency group expense splitter built for trip groups. No server, no login, no install — just open the HTML file in a browser.

---

## Quick Start

1. Download `SplitPro.html`
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari)
3. Click **+** to create your first trip
4. Add members, log expenses, settle up

That's it. All data is saved automatically in your browser's `localStorage`.

---

## Features

### Trips
- Create unlimited trips, each with its own name, date range, and member list
- Switch between trips using the arrow buttons or the trip name button in the header
- Delete trips from the Manage Members panel

### Members & Contacts
- Global contacts list — add a person once, reuse them across trips
- Per-trip membership — each trip has its own subset of contacts
- Add new contacts inline when creating a trip or from the Manage Members modal

### Expenses
- Log expenses with: date, description, amount, category, payer
- Per-row split control — toggle each member in/out of sharing an expense
- Quick row actions: **All**, **None**, **Only Payer**
- Bulk action: **All Members Every Row** sets every expense to split with everyone
- Inline payer and category editing directly in the table
- Delete individual expenses

### Categories
- 7 built-in categories: Accommodation, Tea/Coffee, Party, Food, Snacks, Misc, Spa
- Add custom categories with a name and emoji icon
- Delete custom categories (built-ins are protected)
- Category breakdown widget with spend amounts and progress bars
- Click any category bar to filter the expense table

### Settlement Engine
- Greedy minimal cash-flow algorithm — reduces N-person debts to at most N−1 direct transfers
- Settlement modal shows: net balances per member, total spend, and the exact transfers needed
- One-click copy of any transfer as a UPI memo note

### Filters & Search
- Search by description, date, or payer name
- Filter by category, payer, or "split includes member"
- All filters work together and update the table live

### Exports

| Button | Output |
|--------|--------|
| 🖨️ Print / PDF | Opens a clean dedicated print window — branded header, member balance cards, settlement transfers, category breakdown with progress bars, full expense ledger. Auto-triggers print dialog. Proper A4 page breaks applied. |
| 📄 HTML Archive | Downloads a self-contained read-only `.html` snapshot of the trip — shareable, no internet needed |
| ⬇️ CSV | Downloads a spreadsheet with all expense rows including per-head cost and shared-with names |
| 📲 WhatsApp | Copies a formatted text summary to clipboard — ready to paste into a group chat |

### CSV Import
Two-step import flow:
1. Upload any `.csv` file
2. Map columns to fields (Date, Description, Amount, Category, Paid By) — auto-guessed from header names
3. Preview 3 sample rows, then import

Unmatched payers default to the first trip member. Unmatched categories default to Misc.

---

## Data & Storage

- All data is stored in `localStorage` under the key `SPLITPRO_V2`
- No data is sent anywhere — fully offline
- Data persists across browser sessions on the same device
- Clearing browser site data will erase all trips

---

## Architecture

Everything lives in a single `SplitPro.html` file — no build step, no bundler, no backend.

```
SplitPro.html
├── <style>          Tailwind CDN + custom scrollbar styles
├── <body>           App shell: header, main content, modals, toast
└── <script> ×3
    ├── Block 1      Constants, state, persistence, helpers, settlement engine, render functions
    ├── Block 2      Row operations, clipboard, WhatsApp text, contact helpers
    └── Block 3      All event listeners, PDF/CSV/HTML export handlers, CSV import, init
```

### State Shape

```js
appState = {
  contacts: [{ id, name }],
  trips: [{
    id, name, startDate, endDate,
    memberIds: [contactId, ...],
    expenses: [{
      id, date, desc, amount, cat,
      payerId, sharedWith: [contactId, ...]
    }]
  }],
  activeTripId: string | null,
  categories: [{ key, label, icon, builtin }]
}
```

### Key Functions

| Function | Purpose |
|----------|---------|
| `calculateSettlement(trip)` | Greedy debt-minimization engine. Returns `memberTotals`, `transfers`, `categoryTotals`, `grandTotal` |
| `refreshApp()` | Full re-render of all UI from state |
| `renderExpenseTable(trip, members)` | Renders filtered/searched expense rows |
| `getCategories()` | Returns `appState.categories` or falls back to `DEFAULT_CATEGORIES` |
| `categoriesEntries()` | `[key, {label, icon}]` pairs for all active categories |
| `saveState()` / `loadState()` | `localStorage` persistence |
| `parseCSV(text)` | RFC-4180 compliant CSV parser |
| `guessMapping(headers)` | Auto-maps CSV column headers to expense fields |

### Dependencies (CDN only)

| Library | Purpose |
|---------|---------|
| [Tailwind CSS](https://tailwindcss.com) | Utility-first styling |
| [Lucide Icons](https://lucide.dev) | Icon set |
| [Plus Jakarta Sans](https://fonts.google.com/specimen/Plus+Jakarta+Sans) | UI font |
| [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) | Monospace font for amounts |

All loaded from CDN — requires internet on first load; cached by browser after that.

---

## Browser Support

Works in any modern browser. Requires:
- `localStorage` (all modern browsers)
- ES6+ (all modern browsers)
- `window.open()` for PDF export (may be blocked by popup blockers — allow if prompted)

---

## File History

| File | Description |
|------|-------------|
| `TS1.html` | Original prototype — single trip, hardcoded 7 members, `window.print()` PDF |
| `SplitPro.html` | Current version — multi-trip, dynamic members, dynamic categories, CSV import, clean PDF export |
