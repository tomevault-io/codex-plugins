# Copilot instructions

### **Core Directive**
As an expert AI pair programmer, my goal is to deliver precise, high-quality code modifications by operating as an autonomous agent. I will follow your instructions meticulously, continuing to work through my plan until the request is fully resolved.

### **Core Principles**
1. **Minimize Scope of Change**  
   - Identify the smallest unit (function, class, or module) that fulfills the requirement.  
   - Do not modify unrelated code.  
   - Avoid refactoring unless required for correctness or explicitly requested.

2. **Preserve System Behavior**  
   - Ensure the change does not affect existing features or alter outputs outside the intended scope.  
   - Maintain original patterns, APIs, and architectural structure unless otherwise instructed.

3. **Graduated Change Strategy**  
   - **Default:** Implement the minimal, focused change.  
   - **If Needed:** Apply small, local refactorings (e.g., rename a variable, extract a function).  
   - **Only if Explicitly Requested:** Perform broad restructuring across files or modules.

4. **Clarify Before Acting on Ambiguity**  
   - If the task scope is unclear or may impact multiple components, stop and request clarification.  
   - Never assume broader intent beyond the described requirement.
  
### **Detailed Overiew of the Codebase**
Overview
- Django backend API/service in Python using Django REST Framework and pandas/SQLAlchemy for data processing.
- Purpose: ERP-style ingestion from external systems (IKEA portal, GST portal, E-Invoice portal), normalize into internal models, generate GST filings (workings + JSON), and manage e-invoice lifecycle.
- Key external clients (custom/classes.py):
  - IkeaDownloader/BaseIkea: fetches reports, sales registers, inventory, statements, PDFs, etc.
  - Gst: GST portal session + JSON/ZIP download, invoice fetch, EINVOICE data access, JSON generation.
  - Einvoice: NIC e-invoice portal session, bulk upload, recent IRN retrieval.
- Auth/session for external systems is stored per Django user in app.company_models.UserSession (username/password/cookies/config). All client sessions load and persist cookies through this model.

Project structure (what matters)
- app/api.py: Primary REST endpoints (function-based views). Central place for GST/E-Invoice actions, Excel/PDF responses, and session updates.
- custom/Session.py: Base HTTP session with logging, cookie persistence, error handling, and enforced base_url. Subclasses set key, base_url, is_logged_in(), login(). Responses are HTML-logged to logs/<key>.html.
- custom/classes.py: Concrete clients and workflows.
  - BaseIkea/IkeaDownloader: report() helpers via curl templates, login flow, many report/download utilities (sales_reg, gstr_report, damage_proposals, pending statements, eway, einvoice_json, etc.).
  - Gst: portal automation (captcha login, downloads, JSON building, EINVOICE data).
  - Einvoice: NIC portal automation (captcha/login, bulk upload, IRN fetch).
- app/report_models.py: Report model pattern.
  - Define <Something>Report models with inner Report that declares fetcher (IkeaDownloader method), column_map, preprocessing, caching options.
  - BaseReportModel handles saving DataFrame to DB and update_db() orchestration.
  - DateReportModel and EmptyReportModel encapsulate date-ranged and full refresh behaviors.
  - Concrete reports: SalesRegisterReport, IkeaGSTR1Report, DmgShtReport, StockHsnRateReport, PartyReport, GSTR1Portal.
- app/erp_import.py: Import pipeline from external reports to ERP models.
  - SalesImport, MarketReturnImport, StockImport, PartyImport.
  - GstFilingImport orchestrates report refresh (in parallel) then transactional imports; applies SalesChanges deltas.
- app/erp_models.py: Core ERP models with composite primary keys and cross-table relations via from_fields/to_fields.
  - Sales, Purchase, Inventory, Stock, Party, SalesChanges, Discount, StockAdjustment.
  - Sales.user_objects manager scopes queries to the authenticated user’s companies.
- app/gst.py: GST monthly return generator.
  - Builds workings Excel, matches internal vs portal, computes zero-rate logic, constructs GST JSON, pulls extra HSN from e-invoice if needed.
- app/einvoice.py: E-invoice JSON builder from internal Sales and Inventory; date tweaks and JSON encoding.
- app/management/commands/monthly_gst.py: CLI command to run monthly GST import/marking by period.

Core concepts and flows
- Session + login
  - Each external client Session subclass uses app.company_models.UserSession keyed by (user, key) to load username/password/cookies/config.
  - check_login decorator (api.py) instantiates the client for request.user and calls is_logged_in(). If false, responds 501 with {"key": client.key}. Frontend should then:
    - POST /get_captcha with key -> returns image.
    - POST /captcha_login with key, captcha -> stores cookies via client.login() and UserSession.update_cookies().
- Data ingestion pipeline
  - Reports fetched via IkeaDownloader.* methods are mapped in Report inner classes (fetcher attribute).
  - report_models.* map raw columns (column_map), preprocess, and save to *_report tables.
  - erp_import.* transforms report rows into normalized ERP rows:
    - SalesImport merges SalesRegisterReport (values/discounts) with IkeaGSTR1Report (item lines) and claimservice logic, then bulk creates Sales, Discount, Inventory, and upserts Stock.
    - MarketReturnImport transforms damage/shortage returns with inferred RT and CTIN.
- GST monthly return
  - api.generate_gst_return orchestrates: load IRNs, run gst.generate(user, period, gst_client).
  - gst.generate builds diffs vs GSTR1Portal, computes zero-rate corrections, rate summaries, writes static/{user}/workings_{period}.xlsx and {period}.json.
- E-Invoice lifecycle
  - api.einvoice_stats: shows filing stats per company/type by IRN presence.
  - api.file_einvoice:
    - Pulls IKEA-sourced einvoice_json where available for period + inums; builds remaining via einvoice.create_einv_json; adjusts dates; uploads via Einvoice.upload; handles duplicates (2150) and GSTIN errors by updating Sales or clearing ctin with notes.
  - api.einvoice_pdf: Generates NIC-formatted PDFs via chrome headless and splits/archives by party; requires chrome in env.
- Usersession admin
  - api.usersession_update: GET returns grouped credentials; POST updates credentials for a (key, user) pair.

APIs (selected)
- POST /get_captcha: body { key } -> PNG captcha from client.captcha().
- POST /captcha_login: body { key, captcha } -> ok if session established; WrongCredentials mapped to {"error":"invalid_credentials"}.
- POST /einvoice_reload: refresh IRNs from GST and Einvoice portals.
- POST /einvoice_stats: body { period, type } -> counts by company/type.
- POST /file_einvoice: body { period, type } -> uploads JSON; returns Excel (success/failed).
- POST /einvoice_excel: body { period, type } -> registered/unregistered invoice listings.
- POST /einvoice_pdf: body { period, type } -> ZIP of PDFs (requires chrome).
- POST /generate_gst_return: body { period } -> generates workings + JSON; returns summary stats.
- POST /gst_summary and /gst_json: download generated files.
- GET/POST /usersession_update: view/update UserSession credentials.

Coding guidelines
- Keep code minimal and follow existing patterns:
  - Views: function-based views with @api_view; reuse excel_response and check_login where applicable.
  - External access: always go through Session subclasses (IkeaDownloader, Gst, Einvoice). Do not hand-roll requests.
  - Data import: add new reports as Report models with inner Report.fetcher pointing to an IkeaDownloader method; wire them into the appropriate Import class (erp_import.py) and/or GstFilingImport.imports.
  - Use Django ORM for DB interactions; leverage Sales.user_objects.for_user(user) to scope by authenticated user’s companies.
  - Respect composite primary keys by using from_fields/to_fields ForeignObject patterns already present; prefer bulk_create with update_conflicts/unique_fields as shown.
  - Avoid duplicating logic across reports/imports; extend column_map and custom_preprocessing instead.
  - Configuration: read from UserSession.config; never hardcode secrets; persist new cookies via UserSession.update_cookies().
- Logging and robustness:
  - Session.send() disables TLS verification and sets long timeouts; keep endpoints resilient and avoid blocking the request thread for very long operations unless necessary.
  - Logger writes HTML to logs/<key>.html and large response bodies to logs/files; do not log sensitive data.

How to add a new IKEA-backed report
- Create a new ReportModel in app/report_models.py:
  - Define fields and Meta.db_table.
  - Inner class Report(DateReportModel.Report or EmptyReportModel.Report):
    - set fetcher = IkeaDownloader.<method>
    - set column_map, ignore_last_nrows, dropna_columns, date_format as needed.
    - override custom_preprocessing if required.
- If the data should land in ERP models, extend/import via app/erp_import.py:
  - Add the ReportModel to the appropriate Import.reports list or create a new Import class.
  - Map report rows to ERP models and bulk_create.
- Expose via API if needed using @api_view and check_login(IkeaDownloader subclass) where login is required.

Exploration notes
- Trust this guide; search the code only if a file/function is not listed.
- This is a good example of the most commonly used things in the codebase , but there are things that are not covered in this file
- Do not introduce CI/build/test scaffolding unless explicitly requested.
- Environment assumptions:
  - Headless Chrome available for PDF generation.
  - Database supports update_conflicts with unique_fields (used in bulk_create).
  - app.sql.engine is an SQLAlchemy engine for to_sql operations.
- Common pitfalls:
  - Ensure gst_period is set (monthly_gst command does this) before GST/E-Invoice workflows.
  - SalesReturn mapping relies on IkeaGSTR1Report credit/debit note fields; keep those mappings consistent.
  - E-Invoice date rules: change_einv_dates caps document dates to within 28 days if needed.

---
> Source: [Venkatesh2304/myerpv3](https://github.com/Venkatesh2304/myerpv3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
