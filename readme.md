# B2B SaaS Platform (Angular / .NET)

> Personal project — intentionally anonymised. Built and maintained for over 2 years, currently live with business clients. Development managed via personal Azure DevOps backlog.

## Business Context
Polish B2B SaaS targeting a specific industry. Companies enter their core business orders, which automatically populate and sync data across the entire platform — documents, invoices, monthly/annual reports. Replaces manual workflows based on Word, Paint and paper documents with a centralised, integrated system. Key modules: industry-specific document management, canvas-based document generator, VAT invoicing with KSeF integration, report generation.

## Tech Stack

**Backend (.NET):** ASP.NET Core, EF Core, AutoMapper, Serilog → Loki → Grafana (log dashboard), PDFSharp (filling existing PDFs), QuestPDF (generating PDFs from scratch), ClosedXML (Excel reports), KSeF.Client NuGet (KSeF system integration)
- ASP.NET Core Identity — user and role management (Owner / Worker / Admin), role-based access control via `[Authorize(Roles)]`
- JWT authentication, AES encryption for sensitive data at rest
- Custom `AccessCheck` (DI) — per-request resource ownership validation (IDOR prevention)
- `RequestTimingMiddleware` — logs method, path, user, status code and elapsed time for every request (performance bottleneck detection)

**Frontend (Angular):** FabricJS (canvas-based document generator with custom variable system, reusable templates and PNG export), custom services: image compression, loading screen, navigation; route guards (auth protection), HTTP interceptors (JWT injection, global error handling), reactive state via Signals / Subject / BehaviorSubject

**Infrastructure (Docker Compose / VPS Debian):** PostgreSQL ×2 (dev/prod), API, Angular ×2 (two domains), Traefik, Loki + Promtail + Grafana

## Technical Decisions
- **Refactored core data unit from Company → Department** — enabled multi-branch companies to operate as a single account; used by 3 clients
- **REST API** with resource-based controllers, semantic HTTP methods, DTOs, EF Core with migrations, Data Annotations for input validation
- **Layered image compression** — client-side compression in Angular (UX/transfer) + size validation in controller (protection against bypassing the frontend)
- **Shared database single schema** — all tenants in one database, isolated by Company; sufficient at current scale
- **KSeF (National e-Invoice System) — DDD approach:** tool for submitting VAT invoices and corrections (KOR) created in the app to KSeF and receiving UPO confirmation; environments: demo → preprod → prod; XSD schema → domain model class generated via online tool → custom mapper from internal invoice model to domain model; custom `.Serialization` class with `ShouldSerialize` methods to strip unused XML fragments on serialization; session-based flow (open session → operations → close session); custom endpoint implementations where the official NuGet library behavior didn't meet requirements

## Production Support & Continuous Improvement Examples
- UX refactors on selected screens — identified during client observation sessions
- Date format inconsistency (dashes instead of dots) — reported by client, resolved immediately
- Pagination state bug — page not resetting to 1 on filter/search change; caught during manual smoke testing
- FabricJS text input rendering issue after page size change (A4→A3) — caught during manual smoke testing before reaching clients

- Invoice calculation mismatch — frontend calculated totals dynamically for preview performance; on save, backend re-validated calculations and logged mismatches; error detected via Grafana on a test client environment and resolved before the client was aware
