# AGENTS.md — AI Editing Guide for FreightFlow

This file provides structured guidance for AI agents (Claude, GPT-4, Gemini, Cursor, etc.) editing this codebase.

---

## Architecture Overview

FreightFlow is a **single-file React application**. All logic, state, components, and mock data live in `src/App.jsx`. There is no backend, no database, and no external API calls.

---

## File Map

```
src/App.jsx
│
├── Line 1          → HS_DATA (inline array, ~500KB — DO NOT MODIFY)
├── Line 2          → HS_SECTIONS (section labels)
│
├── // === HELPERS ===
│   ├── toDateSafe()
│   ├── hoursDiff() / hoursUntil() / daysDiff()
│   ├── normalizeStr() / searchHs()
│   ├── recomputeWorkflow()
│   ├── deriveExceptions()
│   ├── getDraftBlData()
│   ├── getDraftBlMissingFields()
│   ├── generateDraftBlHtml()
│   └── saveDraftBlDocument()
│
├── // === CONSTANTS ===
│   ├── TASK_CODE (semantic code map)
│   ├── WORKFLOW_TEMPLATE (task definitions with .code fields)
│   └── LANG (TR/EN label strings)
│
├── // === MOCK DATA ===
│   ├── USERS (with isAdmin, allowedRegions)
│   └── initBookings() → MOCK_BOOKINGS
│
├── // === COMPONENTS ===
│   ├── HsCodeField          (booking-level HS, fallback only)
│   ├── CargoLineHsField     (per-cargo-line HS, primary source)
│   ├── PBar                 (progress bar)
│   ├── Section              (collapsible section)
│   ├── IR                   (info row)
│   ├── EvidenceModal        (task evidence viewer)
│   ├── StoryPanel           (audit log)
│   ├── QuickUpdateModal     (fast update modal)
│   ├── NewBookingWizard     (5-step booking creation)
│   ├── DetailTab            (booking detail view)
│   ├── BookingDetail        (booking detail container)
│   └── BookerPanel          (main booking list)
│
└── // === APP ===
    └── App()                (root component, routing, state)
```

---

## Editing Rules

### DO
- Use `findTaskByCode(booking, TASK_CODE.DRAFT_BL_PREPARED)` to find tasks semantically
- Read `containers[].cargoLines[]` as the primary HS data source
- Use `booking.notifyParty || booking.agent || ""` for notify mapping
- Add new fields to `initBookings()` mock data for testing
- Use `recomputeWorkflow(booking)` after any booking state change
- Keep all new components in `src/App.jsx` (single-file architecture)

### DO NOT
- Modify `HS_DATA` or `HS_SECTIONS` (lines 1-2) — treat as read-only
- Use hardcoded task array indices (e.g., `booking.workflow.tasks[3]`)
- Add external CSS frameworks or import new npm packages without updating `package.json`
- Split the file into multiple modules (breaks the single-file architecture)
- Change UI colors, spacing, or layout (surgical patches only)
- Add backend routes or API calls (frontend-only app)

---

## Semantic Task Codes

Always use `TASK_CODE` constants and `findTaskByCode()`:

```js
const TASK_CODE = {
  GATE_IN:    "gate_in_confirmed",
  CUSTOMS:    "customs_closed",
  DRAFT_BL:   "draft_bl_prepared",
  SI:         "si_submitted",
  VGM:        "vgm_received",
  AR_PAID:    "ar_paid",
};

// Usage
const task = findTaskByCode(booking, TASK_CODE.DRAFT_BL);
if (task) task.done = true;
```

---

## Region Access Control

```js
// USERS model
{
  id: "u1",
  name: "Deniz D.",
  isAdmin: true,
  allowedRegions: ["europe", "asia", "mena"]
}

// Booking model
{
  id: "FF-2026-0042",
  region: "europe",   // "europe" | "asia" | "mena"
  ...
}

// Filter (in App component)
const visibleBookings = bookings.filter(b =>
  isAdmin || !b.region || cuRegions.includes(b.region)
);
```

---

## Cargo Line Structure

```js
// containers[] → cargoLines[]
{
  id: "c1",
  type: "20GP",
  seal: "SL123456",
  cargoLines: [
    {
      id: "cl1",
      hsCode: "847130",
      hsDescription: "Laptops; portable...",
      description: "Laptop computers",
      pkgQty: 100,
      pkgType: "CTN",
      weight: 500,
      cbm: 2.5
    }
  ]
}
```

---

## Draft BL Readiness Fields (15 total)

| Field | Source |
|---|---|
| shipper | booking.shipper |
| consignee | booking.consignee |
| pol | booking.pol |
| pod | booking.pod |
| vessel | booking.vessel |
| voyage | booking.voyage |
| etd | booking.etd |
| eta | booking.eta |
| freightTerms | booking.freightTerms |
| containers | booking.containers (length > 0) |
| packageQty | containers[0].cargoLines[0].pkgQty |
| packageType | containers[0].cargoLines[0].pkgType |
| cbm | containers[0].cargoLines[0].cbm |
| placeOfIssue | form input (modal) |
| issueDate | form input (modal) |

---

## Common Patch Patterns

### Add a new field to booking detail
1. Add field to `initBookings()` mock data
2. Add `IR` row in `DetailTab` component
3. If needed, add to `getDraftBlData()` for BL mapping

### Add a new workflow task
1. Add to `WORKFLOW_TEMPLATE` array with a unique `code` field
2. Add `TASK_CODE` constant
3. Reference via `findTaskByCode()` in logic

### Add a new user role
1. Add user to `USERS` array with `isAdmin` and `allowedRegions`
2. `visibleBookings` filter handles access automatically

---

## Language Keys

Labels are in `LANG.tr` and `LANG.en`. Access via `t(key)` where `t = LANG[lang]`.

```js
// Add new label
LANG.tr.myNewLabel = "Yeni Etiket"
LANG.en.myNewLabel = "New Label"

// Use in component
<span>{t("myNewLabel")}</span>
```
