Project Plan: Third-Party T&C Monitoring System (MVP)

Team Size: 4 Members | Timeline: 3-4 Months (6 Sprints x 2 Weeks)

Team Roles & Focus

Given the small team size, roles must be hybrid.

Member A (Lead/Backend): Architecture, Database, Core Workflow API, IAM Integration.

Member B (Crawler Specialist): Headless browser logic, Proxy routing, HTML cleaning, Storage.

Member C (Frontend): Dashboard UI, Diff Visualization, User Experience.

Member D (DevOps/QA): CI/CD pipeline, Deployment, Automated Testing, Security Hardening.

Phase 1: Foundation & The Crawler (Weeks 1-4)

Goal: We can successfully fetch a URL through the proxy, strip the noise, and save raw text to the DB.

Backend & Infrastructure (Members A & D)

[ ] Repo Setup: Set up Git repo with strict branch protection rules (Audit requirement).

[ ] DB Schema: Design PostgreSQL schema (Users, Vendors, Snapshots, ChangeLogs).

[ ] Network: Set up connection to the Secure Forward Proxy (verify whitelist access).

[ ] Storage: Provision Object Storage (MinIO/S3) bucket structure (Hot vs. Cold/Archive).

[ ] CI/CD: Basic pipeline to build and deploy a "Hello World" service to the Dev environment.

Crawler Engine (Member B)

[ ] Tech Selection: Finalize Python (Playwright/Selenium) vs. Java (Selenium) decision.

[ ] Fetcher Service: Build basic service to fetch HTTP status 200 from a target URL.

[ ] Proxy Integration: Ensure the fetcher correctly routes traffic via the bank's proxy.

[ ] Cleaner Module: Implement logic to strip <nav>, <footer>, <script>, and dynamic timestamps from HTML.

[ ] Scheduling: Implement a basic cron/Quartz job to trigger the fetcher daily.

Frontend (Member C)

[ ] Mockups: Create wireframes for "Vendor List" and "Diff View" (Figma/Sketch).

[ ] Scaffold: Set up the React/Angular project structure.

[ ] Static UI: Build a static version of the Dashboard (no real data yet) to validate layout with stakeholders.

Phase 2: Logic & Diff Engine (Weeks 5-8)

Goal: We can detect changes intelligently and expose them via API.

Core Logic (Members A & B)

[ ] Diff Algorithm: Implement the text comparison logic (e.g., Google Diff Match Patch).

Task: Ensure whitespace changes or simple date updates do not trigger a "Change Detected" alert.

[ ] Snapshot Versioning: Logic to link Snapshot_V1 to Snapshot_V2 and store the computed diff.

[ ] API Development: Build REST endpoints:

GET /vendors (List all monitored URLs)

POST /vendors (Add new URL)

GET /changes/{id} (Get specific diff details)

Frontend Integration (Member C)

[ ] API Binding: Connect the React/Angular app to the GET /vendors API.

[ ] Add Vendor Form: Build the form to input a Name, URL, and Frequency.

[ ] Diff Visualization: (Critical) Implement the side-by-side text comparison UI.

Challenge: Rendering HTML diffs safely without XSS vulnerabilities.

DevOps (Member D)

[ ] Environments: Promote code to a QA environment.

[ ] Seed Data: Create scripts to populate the DB with 50 test URLs for performance testing.

[ ] Logging: Setup centralized logging (ELK/Splunk) for the Crawler execution.

Phase 3: Workflow, Notifications & Security (Weeks 9-12)

Goal: The "Human Loop." Users can log in, receive alerts, and mark items as reviewed.

Workflow & Notifications (Members A & B)

[ ] Notification Service: Integrate with SMTP (Email) or Bank IM (Teams/Slack).

[ ] Alert Logic: Trigger an email only when Diff_Score > 0.

[ ] Review Action: Create API for POST /changes/{id}/review.

Logic: Update status to "Reviewed", record User_ID, Timestamp, and Comments.

Security & IAM (Members A & D)

[ ] SSO Integration: Connect to Bank LDAP/AD for login.

[ ] RBAC: Implement Admin vs. Viewer roles. (Admins add URLs; Viewers only review).

[ ] Audit Trail: (Mandatory) Ensure every POST and PUT action writes to the audit_logs table.

Frontend Polish (Member C)

[ ] Dashboard Stats: Add "Pending Reviews" counter widget.

[ ] History View: Show a timeline of changes for a specific vendor.

[ ] Loading States: Add skeletons/spinners for better UX during slow crawls.

Phase 4: Hardening & UAT (Weeks 13-16)

Goal: Production readiness and handover.

Quality Assurance (All Members)

[ ] False Positive Tuning: Tune the Cleaner Module to ignore cookie banners/popups that break the diff.

[ ] Load Test: Simulate 500 URLs being crawled simultaneously (verify Proxy limits).

[ ] Security Scan: Run SonarQube or bank-specific security scans (remediate vulnerabilities).

Documentation & Handover

[ ] User Manual: Simple PDF guide for Compliance Officers.

[ ] Ops Manual: Guide for IT Support (how to restart the crawler, how to check logs).

[ ] WORM Verification: Verify that old snapshots cannot be deleted from storage.

Key Risks to Watch

Proxy Blocking: Some vendors may block the bank's proxy IP. Mitigation: Support a list of User-Agents.

False Alerts: Dynamic content (ads/news tickers) triggering alerts. Mitigation: Refine the CSS selectors in the Cleaner Module.

Javascript Pages: SPAs (Single Page Apps) returning empty HTML. Mitigation: Ensure Headless Browser (Playwright) waits for network idle.
