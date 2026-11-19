# SmartEV - Production-Grade Smart EV Charging Platform

An enterprise-ready, full-stack Smart EV Charging & Mobility Platform built with Next.js (App Router), React, TypeScript, Leaflet cartography, Recharts, and Supabase PostgreSQL.

The platform enables EV owners and fleet managers to discover nearby EV charging hubs, monitor real-time connector occupancy, compare verified tariffs, navigate to stations, track active charging telemetry, and receive transparent multi-factor AI routing recommendations.

---

## ⚡ Key Architectural Features

1. **Precision Connector-Level Telemetry**:
   - Calculates availability on an individual connector basis rather than treating mixed-status stations as monolithic blocks.
   - Transparently highlights mixed operational states (e.g. 3 available connectors with 1 faulted cable).
   - Standardized status model: `AVAILABLE`, `OCCUPIED`, `PREPARING`, `CHARGING`, `SUSPENDED_EV`, `SUSPENDED_EVSE`, `FINISHING`, `RESERVED`, `UNAVAILABLE`, `FAULTED`, `UNKNOWN`.

2. **Interactive Cartography & Live Map**:
   - Powered by Leaflet & OpenStreetMap tiles (CartoDB Voyager) with Mapbox integration support.
   - Dynamic SVG pin markers color-coded by real status:
     - 🟢 **Green**: At least one connector available.
     - 🔴 **Red**: All operational connectors occupied.
     - 🟡 **Yellow**: Station or connector faulted / maintenance required.
     - ⚪ **Gray**: Unknown status.
   - Browser Geolocation API integration with automatic distance calculation using the Haversine spatial formula.
   - Map popups with connector counts, peak power, tariff rates, and direct turn-by-turn navigation links.

3. **Multi-Factor AI Recommendation Engine**:
   - Three dedicated routing modes:
     - **Nearest Available**: Prioritizes driving proximity to open connectors.
     - **Lowest Cost**: Compares verified pricing structures ($/kWh, session fees).
     - **Fastest Charging**: Identifies ultra-fast high-power DC hubs (150 kW - 350 kW).
   - Explainable AI: Provides transparent reasons and estimates travel time, charge duration (20% to 80% SOC), and cost.
   - Customizable weight multipliers that users can adjust live.

4. **Multi-Source Data Ingestion & Adapters**:
   - **Open Charge Map Client**: Official API v3 adapter with pagination, request timeouts, and exponential backoff retry.
   - **OCPI v2.2 Standard Module**: Roaming protocol client mapping locations, EVSEs, and tariff modules.
   - **IoT Telemetry Gateway**: OCPP 1.6J/2.0.1 WebSocket event parser for real-time heartbeat, status notification, and meter values.
   - Freshness detection marking stale data (>30 mins) vs real-time connected telemetry.

5. **Charging Session Management & Metering**:
   - Live session view with State-of-Charge (SOC %), instantaneous kW delivery, energy metered (kWh), and elapsed time.
   - Session termination with automated receipt generation and printable invoice view.
   - Transparent hardware indicator (distinguishes live OCPP connected hardware from test/demo simulation mode).

6. **Executive Analytics Dashboard**:
   - Interactive Recharts visualizations:
     - Daily energy delivered (kWh) area chart.
     - 24-hour peak grid utilization profile.
     - Connector status breakdown donut chart.
     - Network/CPO market share distribution.

7. **Secure Administrator Dashboard**:
   - Integration health monitor (Supabase, Open Charge Map, OCPI Roaming, OCPP Hardware Gateway).
   - Station and connector inventory CRUD editor.
   - Upstream sync job runner with processed records counters.
   - Driver community issue report triage (Resolve / Dismiss).
   - Immutable audit trail logging actor, action, entity, and timestamp.

---

## 🛠 Technology Stack

- **Frontend**: Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS, Lucide Icons, Recharts, Leaflet.
- **Backend**: Next.js Server-Side Route Handlers (`src/app/api/...`), REST architecture.
- **Database & Auth**: Supabase PostgreSQL, Row Level Security (RLS) policies, plus built-in zero-setup repository fallback engine.
- **Design System**: Tailored EV palette (clean white/light-slate backgrounds, deep navy typography `#0f172a`, and electric green accents `#059669`).

---

## 🚀 Getting Started

### 1. Install Dependencies
```bash
npm install
```

### 2. Environment Variables (Optional)
Copy `.env.example` to `.env.local`:
```bash
cp .env.example .env.local
```
Configure your credentials:
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-supabase-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
OPEN_CHARGE_MAP_API_KEY=your_ocm_key
OCPI_BASE_URL=https://ocpi.example.com
OCPI_TOKEN=your_token
```
*(Note: If credentials are not yet configured, the platform automatically runs on its resilient repository engine pre-loaded with verified benchmark stations across New York, London, Berlin, San Francisco, Bengaluru, and Chennai).*

### 3. Run Database Migrations (For Supabase)
If deploying with a live Supabase project:
- Execute `supabase/migrations/20261030_init_ev_platform.sql` in the Supabase SQL editor.
- Execute `supabase/seed.sql` to populate initial verified stations and connectors.

### 4. Run Development Server
```bash
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

### 5. Production Build
```bash
npm run build
npm run start -p 3000
```

---

## 📡 REST API Reference

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/api/stations` | `GET` | Filterable list of stations (distance, connector type, operator, availability) |
| `/api/stations/:id` | `GET` | Detailed station record with connectors and tariffs |
| `/api/stations/:id/connectors` | `GET` | List connector-level statuses for a station |
| `/api/stations/nearby` | `GET` | Radius-based geographic station search |
| `/api/stations/search` | `GET` | Full-text query search by name, operator, or city |
| `/api/favorites` | `GET`, `POST`, `DELETE` | User saved favorite stations |
| `/api/sessions` | `GET` | History of charging sessions and active session |
| `/api/sessions/start` | `POST` | Initiate charging session with hardware verification |
| `/api/sessions/:id/stop` | `POST` | Terminate charging session and generate receipt |
| `/api/recommendations` | `GET` | Multi-factor AI scored station recommendations |
| `/api/analytics/overview` | `GET` | Aggregated energy, uptime, and hourly metrics |
| `/api/notifications` | `GET`, `POST` | In-app alerts and mark-all-read handler |
| `/api/notifications/:id/read` | `PATCH` | Mark individual notification as read |
| `/api/auth/me` | `GET`, `PATCH` | Driver profile and vehicle specification preferences |
| `/api/admin/health` | `GET` | Live integration health status for database, APIs, and IoT |
| `/api/admin/stations` | `GET`, `POST` | Administrator station registry and creation |
| `/api/admin/stations/:id` | `PATCH` | Update station metadata and tariffs |
| `/api/admin/sync-jobs` | `GET`, `POST` | Manage and trigger CPO synchronization crawlers |
| `/api/admin/issues` | `GET`, `POST` | Community issue reporting and triage |
| `/api/admin/issues/:id` | `PATCH` | Update error report resolution status |
| `/api/admin/audit-logs` | `GET` | Query immutable administrative audit trail |

---

## 🧪 Testing

Unit tests for calculation utilities and scoring algorithms can be run with:
```bash
npx vitest run
```
