# FreightFlow

**FreightFlow** is a React-based freight forwarding operations management application. It provides end-to-end booking lifecycle management, workflow automation, Draft Bill of Lading generation, HS Code search, and region-based access control.

---

## Features

- **Booking Management** — Full booking lifecycle: Offer → Confirm → Loading → Transit → Arrival → Delivery
- **Workflow Engine** — Semantic task codes, SLA tracking, automatic rule evaluation, exception detection
- **Draft BL Workflow** — Readiness scoring (15 fields), HTML preview, save to documents
- **HS Code Search** — 6,842 harmonized system codes (inline dataset), search by code or description
- **Cargo Line HS** — Per-container, per-cargo-line HS code assignment
- **Region-Based Access Control** — Users have `allowedRegions`; bookings have `region`; admins see all
- **New Booking Wizard** — 5-step modal: Type → Info → Schedule → Container → Draft BL
- **Quick Update Modal** — Fast status/charge updates with audit log
- **Multi-language** — Turkish / English toggle
- **Dark/Light Mode** — Theme toggle

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI Framework | React 18 (JSX) |
| Build Tool | Vite |
| Styling | Inline styles + CSS |
| Data | Local mock state (no backend) |
| HS Dataset | Harmonized System (6,842 codes, inline) |

---

## Project Structure

```
freightflow/
├── src/
│   ├── App.jsx          ← Main application (all components, state, logic)
│   ├── main.jsx         ← React entry point
│   ├── index.css        ← Global styles
│   └── hsData.js        ← HS Code dataset (standalone, also inlined in App.jsx)
├── versions/
│   ├── FreightFlow_v9_2_DraftBL.jsx     ← Draft BL Workflow added
│   ├── FreightFlow_v9_3_HSCode.jsx      ← HS Code search added
│   ├── FreightFlow_v9_4_Patch.jsx       ← 15-item patch (admin, cargo lines, semantic tasks)
│   ├── FreightFlow_v10_patched.jsx      ← Region access control, admin panel removed
│   └── FreightFlow_v10_final.jsx        ← Current stable version
├── index.html
├── vite.config.js
├── package.json
└── README.md
```

---

## Getting Started

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

---

## Version History

| Version | Key Changes |
|---|---|
| v9.2 | Draft BL Workflow: readiness scoring, HTML preview, save to documents |
| v9.3 | HS Code search/select (6,842 codes), booking-level HS field |
| v9.4 | 15-item patch: New Booking Wizard, cargoLines, semantic task codes, HS inline |
| v10 patched | Admin panel removed, region-based access control, USERS model updated |
| v10 final | New Booking region field, semantic workflow codes, cargo line HS as primary source |

---

## Key Concepts

### Semantic Task Codes
Tasks in the workflow template use stable `code` fields instead of fragile array indices:

```js
gate_in_confirmed   // Gate-in teyidi
customs_closed      // Gümrük kapatıldı
draft_bl_prepared   // Draft BL hazırlandı
si_submitted        // SI alındı
vgm_received        // VGM alındı
ar_paid             // Tahsilat alındı
```

### Region Access Control
```js
// User model
{ id: "u2", name: "Ayşe K.", isAdmin: false, allowedRegions: ["europe"] }

// Booking model
{ id: "FF-2026-0042", region: "europe", ... }

// Filter logic
visibleBookings = bookings.filter(b =>
  isAdmin || !b.region || cuRegions.includes(b.region)
)
```

### Cargo Line HS Flow
```
containers[]
  └── cargoLines[]
        ├── hsCode        ← Primary HS source
        ├── hsDescription
        ├── description
        ├── pkgQty
        ├── pkgType
        ├── weight
        └── cbm
```

---

## AI Editing Notes

This codebase is designed to be **AI-editable**. Key guidelines:

- All application logic is in a **single file**: `src/App.jsx`
- Functions are organized in sections marked with `// === SECTION NAME ===`
- Mock data is at the top, helper functions in the middle, UI components at the bottom
- The HS dataset is large (~500KB inline) — treat it as a read-only constant
- Use **semantic task codes** (`findTaskByCode`) instead of array indices
- Patch surgically — do not rewrite the entire file
- UI styles are inline — do not introduce external CSS frameworks

---

## License

MIT
