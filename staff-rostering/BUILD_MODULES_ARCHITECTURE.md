# Staff Rostering Application - Build Modules Architecture

**Version:** 1.0  
**Date:** November 20, 2025  
**Status:** Build Decision - Module Planning  
**Decision:** Build in-house vs vendor solution

---

## Executive Summary

**For leadership:** Transitioning from vendor evaluation to in-house build. Organized 15 features into **9 core technical modules** spanning backend services, mobile apps, and integrations. Priority modules (Scheduling Engine, Compliance Engine, API Gateway) deliver 75% of business value and establish the platform foundation.

**Timeline:** 12-18 months for full MVP (P0+P1 features)  
**Team recommendation:** 8-12 engineers (3 backend, 2 mobile, 2 frontend, 2 integration, 1 DevOps, 1-2 QA)

---

## Module Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
├──────────────────┬──────────────────┬──────────────────────┤
│  Mobile App      │  Web Portal      │  Manager Dashboard   │
│  (iOS/Android)   │  (Employee)      │  (Store Managers)    │
└──────────────────┴──────────────────┴──────────────────────┘
                            ▲
                            │
┌───────────────────────────┴─────────────────────────────────┐
│                     API GATEWAY LAYER                        │
│  (Authentication, Rate Limiting, Routing)                    │
└──────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌───────────────────────────┴─────────────────────────────────┐
│                    CORE SERVICES LAYER                       │
├──────────────────┬──────────────────┬──────────────────────┤
│  Scheduling      │  Compliance      │  Forecasting         │
│  Engine          │  Engine          │  Service             │
├──────────────────┼──────────────────┼──────────────────────┤
│  Workforce       │  Time &          │  Leave               │
│  Management      │  Attendance      │  Management          │
├──────────────────┼──────────────────┼──────────────────────┤
│  Cost            │  Notification    │  Analytics &         │
│  Management      │  Service         │  Reporting           │
└──────────────────┴──────────────────┴──────────────────────┘
                            ▲
                            │
┌───────────────────────────┴─────────────────────────────────┐
│                   INTEGRATION LAYER                          │
│  (Payroll, HRIS, POS, ERP Connectors)                       │
└──────────────────────────────────────────────────────────────┘
                            ▲
                            │
┌───────────────────────────┴─────────────────────────────────┐
│                   DATA & STORAGE LAYER                       │
│  (PostgreSQL, Redis, S3, Event Store)                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Module Definitions

### **Module 1: Scheduling Engine** 🎯 CRITICAL PATH
**Priority:** P0  
**Description:** AI-driven core scheduler that generates optimal rosters based on constraints

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Skills-based role matching | Only assigns qualified employees to specialized roles. Ensures employees with required certifications (e.g., POS certification for cashier shifts) are matched to appropriate positions |
| Availability constraint handling | Respects employee availability preferences and blackout periods. System never schedules employees during their marked unavailable times (e.g., Sundays for religious observances) |
| Contract hours enforcement | Prevents over/under-scheduling per contract terms. Automatically blocks scheduling beyond agreed hours (e.g., part-timer's 20 hours/week limit) |
| Conflict detection & resolution | Catches scheduling errors before publishing. Alerts managers to double-bookings, overlapping shifts at different locations, or other scheduling conflicts |
| Template management for recurring schedules | Reuses proven schedule patterns for recurring events. Save templates (e.g., "Diwali Sale" schedule) and apply them with one click for future similar periods |
| Manual override & adjustment | Allows managers to manually adjust auto-generated schedules. Provides flexibility to fine-tune schedules based on business needs or special circumstances |

**Technical Components:**
- **Constraint solver:** Linear programming optimization (OR-Tools, OptaPlanner)
- **Schedule generator API:** REST endpoints for schedule creation/modification
- **Template engine:** Store and apply recurring patterns
- **Conflict detection:** Real-time validation service
- **Data models:** Shifts, Assignments, Constraints, Templates

**Dependencies:**
- Module 2 (Compliance Engine) for rule validation
- Module 3 (Forecasting Service) for demand inputs
- Module 4 (Workforce Management) for employee data

---

### **Module 2: Compliance Engine** 🎯 CRITICAL PATH
**Priority:** P0  
**Description:** Rules engine enforcing labor laws, awards, and company policies

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Maximum work hours enforcement (daily/weekly) | Prevents illegal work hours by blocking schedules that exceed legal limits (e.g., Karnataka Shops Act 48 hours/week maximum). System rejects with clear violation messages |
| Minimum rest period validation | Ensures adequate rest between shifts per labor law requirements. Blocks scheduling shifts that violate minimum rest periods (e.g., 12-hour gap between shifts) |
| Break requirement insertion | Auto-adds mandatory breaks to shifts based on duration and regulations. Automatically inserts required lunch breaks at appropriate times (e.g., 1-hour break for 9-hour shifts) |
| Minor labor restrictions | Protects young workers by enforcing child labor laws. Prevents scheduling minors past restricted hours or beyond allowed daily limits (e.g., no shifts after 7 PM for 17-year-olds) |
| Contract term limits | Respects contract terms and prevents over-scheduling. System stops at contract limits even if employee volunteers for additional hours (e.g., 100 hours/month maximum for part-timers) |
| Audit trail & compliance reporting | Records all compliance actions in immutable logs. Enables export of compliance reports showing zero violations, break enforcement, and overtime controls for labor inspector visits |

**Technical Components:**
- **Rules engine:** Configurable business rules (Drools, Easy Rules)
- **Policy manager:** CRUD for compliance rules per location/role
- **Validator service:** Pre-schedule and real-time validation
- **Audit logger:** Immutable compliance event log
- **Alert system:** Proactive violation warnings

**Dependencies:**
- Module 1 (Scheduling Engine) consumes validation results
- Module 4 (Workforce Management) for employee contract data

---

### **Module 3: Forecasting Service** 🎯 CRITICAL PATH
**Priority:** P0  
**Description:** Demand prediction engine using historical sales and foot traffic data

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Historical sales data analysis | Analyzes past sales patterns to predict staffing needs. Identifies trends (e.g., Saturdays generate 40% more sales) and suggests appropriate staff levels accordingly |
| Seasonal/event-based adjustments | Accounts for festivals, holidays, and special events. Automatically increases staff recommendations for known high-traffic periods (e.g., 50% increase for Diwali week) based on historical data |
| Peak hour identification | Spots busy time periods throughout the day. Identifies rush hours (e.g., 6 PM-9 PM post-office crowd) and recommends additional staff during those windows |
| Recommended staffing levels by role/store | Suggests optimal team size by role and location. Provides specific recommendations (e.g., 8 sales floor staff + 4 cashiers + 1 supervisor) based on expected footfall and sales projections |

**Technical Components:**
- **ML forecasting model:** Time-series prediction (Prophet, LSTM)
- **Data ingestion pipeline:** ETL from POS systems
- **Forecasting API:** Serve staffing recommendations
- **Manual override:** Allow managers to adjust predictions
- **Model training service:** Periodic retraining with new data

**Dependencies:**
- Integration Layer (Module 9) for POS data
- Module 1 (Scheduling Engine) consumes forecasts

---

### **Module 4: Workforce Management**
**Priority:** P0 (Core) + P1 (Advanced features)  
**Description:** Employee master data, availability, skills, and preferences management

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Employee profile management (P0) | Core CRUD operations for employee master data including personal information, employment details, roles, and contract terms. Foundation for all workforce operations |
| Availability preferences & blackouts (P1 - Self-Service Portal) | Employees can set recurring availability patterns (e.g., available 6 PM-10 PM weekdays) and submit one-time blackout dates (e.g., Dec 20-25 for wedding). System respects these when generating schedules |
| Skills & certification tracking (P2) | Maintains skills matrix showing employee capabilities (e.g., POS operation, forklift certified). Tracks certification expiry dates and sends renewal alerts. Prevents scheduling unqualified employees |
| Personal details management (P1 - Self-Service Portal) | Employees can update their own personal information (mobile number, address, emergency contacts). Changes reflect automatically across all HR systems |
| Multi-location assignments (P2) | Supports employees working across multiple store locations. Shows combined schedules and manages location-specific assignments, transfers, and pay rate adjustments |

**Technical Components:**
- **Employee service:** CRUD APIs for employee data
- **Availability manager:** Recurring and one-time availability
- **Skills matrix:** Track certifications with expiry
- **Preference engine:** Shift preference ranking
- **Transfer management:** Handle location changes

**Dependencies:**
- Integration Layer (Module 9) for HRIS sync
- Module 2 (Compliance Engine) for contract validation

---

### **Module 5: Time & Attendance Tracking**
**Priority:** P1  
**Description:** Clock in/out, timesheet management, and exception handling

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Multiple clock-in methods (mobile GPS, kiosk, web) | Flexible attendance marking supporting various entry points. Employees can clock in via mobile app with GPS, store kiosk, or web browser - all methods validated and recorded |
| Photo verification option | Optional identity confirmation at clock-in. System can prompt for selfie to prevent buddy punching (friend clocking in on employee's behalf) |
| Break tracking | Accurate monitoring of break times. Records clock-out and clock-back times for breaks, flags overages (e.g., 75-minute break when 60 minutes allowed) |
| Timesheet editing with approvals | Allows correction of attendance mistakes with manager approval. Employees can request edits (e.g., add missing clock-out), manager verifies and approves corrections |
| Exception flagging (late, no-show, early departure) | Automatic detection and alerting of attendance anomalies. System flags late arrivals, no-shows, early departures and alerts managers for follow-up action (e.g., "Suresh clocked in 25 mins late - 3rd time this month") |

**Technical Components:**
- **Clock service:** Record clock in/out events
- **GPS verification:** Geofencing for location validation
- **Photo capture:** Image storage and facial recognition (optional)
- **Timesheet service:** CRUD with approval workflows
- **Exception detector:** Real-time anomaly detection

**Dependencies:**
- Module 7 (Mobile App) for GPS and photo capture
- Module 9 (Integration Layer) for payroll export

---

### **Module 6: Leave Management System**
**Priority:** P1  
**Description:** Leave requests, approvals, balance tracking, and roster integration

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Leave balance tracking per type | Tracks separate leave buckets (casual, sick, earned leaves). Employees can view exact balances (e.g., 6 casual, 9 sick, 12 earned leaves) to plan vacations accordingly |
| Approval workflows with delegation | Routes leave requests to appropriate approvers with automatic delegation. If primary manager is unavailable, request auto-routes to deputy manager for approval |
| Auto-update rosters on approval | Automatically removes employee from approved leave dates in schedules. Alerts manager about shifts needing coverage when leave is approved |
| Blackout period enforcement | Blocks leave requests during restricted periods. System rejects leave applications during busy periods (e.g., Diwali week) with clear blackout period messages |
| Accrual calculations | Automatically calculates earned leave based on tenure and policy. Tracks monthly/annual accrual rates (e.g., 1.5 days/month) and updates balances accordingly |

**Technical Components:**
- **Leave service:** Request submission and approval APIs
- **Balance calculator:** Accrual engine (monthly/annual)
- **Workflow engine:** Multi-step approval routing
- **Roster integrator:** Auto-adjust schedules post-approval
- **Blackout manager:** Define restricted periods

**Dependencies:**
- Module 1 (Scheduling Engine) for roster updates
- Module 4 (Workforce Management) for employee data

---

### **Module 7: Mobile App (iOS & Android)**
**Priority:** P0  
**Description:** Native mobile applications for employees and managers

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| View schedules (real-time sync) | Check schedule anytime on phone with instant updates. Employees see schedule changes synced in real-time (e.g., manager updates schedule, employee sees change within minutes) |
| Clock in/out with GPS verification | Verify location when marking attendance. App confirms location (e.g., "Verified at Reliance Store Pune, 12m away") preventing fake check-ins from home |
| Request time off | Apply for leave from anywhere via mobile. Submit leave requests with manager approval workflow, receive confirmation notifications immediately |
| Shift swap initiation | Trade shifts with colleagues directly from mobile. Offer shifts to eligible colleagues, receive acceptances, and route to manager for approval - all within the app |
| Push notifications for changes | Get instant updates for schedule modifications. Receive immediate notifications when shifts are changed, added, or cancelled (e.g., "Your Tuesday shift moved from 9 AM to 2 PM") |
| Offline mode for schedule viewing | View schedules without internet connection. Last synced schedule data cached locally, allowing employees to check schedules even in no-network areas |

**Technical Components:**
- **Native apps:** Separate iOS (Swift) and Android (Kotlin) apps
- **Sync engine:** Background sync with offline support
- **Push notifications:** FCM (Android), APNS (iOS)
- **GPS module:** Location capture and verification
- **Camera integration:** Photo capture for clock-in
- **Offline storage:** SQLite for local schedule caching

**Dependencies:**
- API Gateway (Module 9) for all backend calls
- Module 5 (Time & Attendance) for clock-in
- Module 11 (Notification Service) for push notifications

---

### **Module 8: Web Portal (Employee & Manager)**
**Priority:** P0 (Manager) + P1 (Employee Self-Service)  
**Description:** Web-based interfaces for desktop users

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Manager dashboard (schedule creation, approvals, analytics) | Comprehensive interface for managers to create/edit schedules, approve leave requests and shift swaps, view team analytics, and track labor costs in real-time |
| Employee self-service portal (availability, preferences, timesheets) | Web interface for employees to set availability, submit blackout dates, view historical timesheets, download pay stubs, and request shift preferences without manager intervention |
| Shift swapping marketplace | Web-based platform for shift trading. Employees can propose swaps, bid on open shifts, and managers can review and approve all swap transactions |
| Reporting & analytics dashboards | Interactive dashboards with drill-down capabilities. View labor costs, attendance trends, overtime analysis, and export reports in PDF/Excel formats |
| Admin configuration panels | System administration interface for configuring compliance rules, pay rates, store settings, user roles, and system-wide policies |

**Technical Components:**
- **Manager dashboard:** Schedule builder, team view, cost tracker
- **Employee portal:** Availability, preferences, leave requests
- **Shift marketplace:** Swap proposals, bidding interface
- **Analytics dashboard:** Charts, reports, export tools
- **Admin panel:** System configuration, user management

**Dependencies:**
- API Gateway (Module 9) for backend integration
- All backend modules for data

---

### **Module 9: Integration Layer & API Platform**
**Priority:** P0 (API Gateway) + P1 (Integrations)  
**Description:** External system connectors and unified API gateway

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Payroll system integration (timesheets export) | Automatically exports timesheet data to payroll systems. Month-end processing exports all employee timesheets at once, eliminating manual data entry (e.g., 342 employees exported to Tally) |
| HRIS integration (employee data sync) | Bi-directional sync of employee master data. New employees added in HRIS automatically appear in rostering system with role, location, and pay rate within minutes |
| POS system integration (sales data for forecasting) | Pulls historical sales data for demand forecasting. System analyzes POS data (e.g., Saturday sales spikes) and automatically suggests staffing adjustments |
| ERP/Accounting integration (labor cost allocation) | Sends labor cost data to finance systems. Weekly labor costs flow automatically to ERP (e.g., SAP), enabling real-time financial reporting without manual data entry |
| RESTful API with webhook support | Provides standardized API for custom integrations. Tech teams can build custom applications using rostering API, with webhook support for event-driven notifications |
| CSV import/export fallback | Manual file transfer option for systems without API access. Export attendance/timesheet data as CSV for manual import when connectivity issues occur or APIs unavailable |

**Technical Components:**
- **API Gateway:** Authentication, rate limiting, routing (Kong, AWS API Gateway)
- **Integration hub:** Pre-built connectors for common systems
- **ETL pipelines:** Data transformation and sync jobs
- **Webhook manager:** Event-driven notifications to external systems
- **CSV processor:** Batch import/export for manual integrations
- **API documentation:** OpenAPI/Swagger specs

**Dependencies:**
- All modules expose APIs through this gateway
- External systems (Payroll, HRIS, POS, ERP)

---

### **Module 10: Cost Management & Budgeting**
**Priority:** P0  
**Description:** Labor cost tracking, budget management, and overtime alerts

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Weekly/monthly budget setting per store | Set spending limits per period and location. Managers can define budgets (e.g., ₹3.5 lakh monthly for DMart Andheri) and system tracks progress with alerts when approaching limits |
| Real-time cost projection during scheduling | Shows costs while building schedule. As managers add shifts, screen displays current week cost vs budget (e.g., "Week cost: ₹87,500 of ₹1 lakh budget - 87% used") before publishing |
| Overtime alerts before approval | Prevents expensive overtime by warning managers. System alerts before approving shifts that would trigger overtime (e.g., "Ramesh has 47 hours - adding this shift triggers ₹4,800 overtime") |
| Different pay rates (FTE vs contractors) | Manages varied compensation structures. Handles different hourly rates for full-time employees, part-time contractors, and overtime multipliers (e.g., 1.5x) correctly for each employee type |
| Labor % of revenue tracking | Monitors cost efficiency metrics. Shows labor cost as percentage of revenue (e.g., "Labor cost is 12.5% of revenue this month") and alerts if crossing target thresholds set by management |

**Technical Components:**
- **Budget manager:** Set and track budgets by period/store
- **Cost calculator:** Real-time labor cost computation
- **Rate engine:** Manage pay rates by employee type, overtime rules
- **Alert service:** Proactive budget and overtime warnings
- **Revenue tracker:** Labor cost as % of revenue

**Dependencies:**
- Module 1 (Scheduling Engine) for schedule cost calculation
- Module 3 (Forecasting) for revenue data
- Module 9 (Integration) for payroll rate sync

---

### **Module 11: Notification & Communication Service**
**Priority:** P2 (but foundational - recommend P1)  
**Description:** Multi-channel notification system and in-app messaging

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Push notifications (schedule changes, reminders) | App alerts for real-time updates. Employees receive instant notifications when schedules change (e.g., "Your Tuesday shift moved from 9 AM to 2 PM") or shift reminders before start time |
| SMS alerts for critical updates | Text messages for urgent information. System sends SMS for critical updates (e.g., "Store closed due to heavy rains - don't come, paid day off") ensuring all employees are informed |
| Email notifications with attachments | Detailed messages delivered via email. Weekly schedule publications include full roster PDF attachments and summary of employee shifts for record-keeping |
| Manager broadcasts to teams | One-to-many messaging for announcements. Managers can send messages to entire teams simultaneously (e.g., "Diwali party this Saturday 7 PM - all invited") |
| In-app messaging between managers and employees | Direct chat functionality within the app. Employees can message managers with questions (e.g., overtime pay queries) and receive private responses within minutes |

**Technical Components:**
- **Notification router:** Multi-channel delivery (push, SMS, email)
- **Template engine:** Customizable notification templates
- **Delivery tracking:** Read receipts and delivery status
- **Messaging service:** Real-time chat infrastructure
- **Preference manager:** User notification preferences

**Dependencies:**
- Module 7 (Mobile App) for push notifications
- All modules generate notification events

---

### **Module 12: Analytics & Reporting Engine**
**Priority:** P2 (recommend P1 for basic reports)  
**Description:** Business intelligence, dashboards, and report generation

**Capabilities Delivered:**

| Capability | Description |
|------------|-------------|
| Pre-built reports (labor cost, attendance, overtime) | Ready-made reports for common use cases. One-click access to standard reports (e.g., "Monthly Attendance Report" showing all employees' attendance percentages sorted by store) |
| Custom report builder | Build your own reports with flexible querying. Regional managers can create custom reports (e.g., "Part-timer Utilization Report" across 5 stores) for specific analysis needs |
| Interactive dashboards with drill-downs | Visual charts with exploration capabilities. Dashboards show labor cost graphs, attendance trends - clicking on data points drills down to detailed views (e.g., October spike shows Diwali week overtime at Pune store) |
| Export to PDF/Excel | Download reports in multiple formats. Finance teams can export attendance data as Excel or PDF for further processing and record-keeping |
| Scheduled report delivery | Automated report generation and distribution. Set up recurring reports (e.g., "Email weekly overtime report every Monday 9 AM to managers") that run automatically without manual effort |

**Technical Components:**
- **Report engine:** Query builder and report generator
- **Dashboard service:** Real-time metrics and KPIs
- **Export service:** PDF/Excel generation
- **Scheduler:** Automated report delivery
- **Data warehouse:** Aggregated data for fast queries

**Dependencies:**
- All modules for data sources
- Module 11 (Notification Service) for scheduled delivery

---

## RICE Prioritization for Build Sequence

| Module | Reach | Impact | Confidence | Effort | RICE Score | Priority | Build Phase |
|--------|-------|--------|------------|--------|------------|----------|-------------|
| **Module 9: Integration Layer & API Platform** | 1000 | 3 | 90% | 5 | 540 | **1st** | Phase 0 (Foundation) |
| **Module 4: Workforce Management (P0)** | 1000 | 3 | 95% | 3 | 950 | **2nd** | Phase 0 (Foundation) |
| **Module 2: Compliance Engine** | 1000 | 3 | 85% | 4 | 638 | **3rd** | Phase 1 (Core) |
| **Module 10: Cost Management** | 500 | 3 | 90% | 2.5 | 540 | **4th** | Phase 1 (Core) |
| **Module 1: Scheduling Engine** | 1000 | 3 | 70% | 6 | 350 | **5th** | Phase 1 (Core) |
| **Module 3: Forecasting Service** | 800 | 2 | 60% | 5 | 192 | **6th** | Phase 1 (Core) |
| **Module 11: Notification Service** | 900 | 2 | 85% | 2.5 | 612 | **7th** | Phase 2 (UX) |
| **Module 7: Mobile App** | 900 | 3 | 80% | 5 | 432 | **8th** | Phase 2 (UX) |
| **Module 8: Web Portal** | 700 | 2 | 85% | 4.5 | 264 | **9th** | Phase 2 (UX) |
| **Module 5: Time & Attendance** | 800 | 2 | 80% | 3.5 | 366 | **10th** | Phase 3 (Operations) |
| **Module 6: Leave Management** | 600 | 2 | 90% | 2.5 | 432 | **11th** | Phase 3 (Operations) |
| **Module 12: Analytics & Reporting** | 400 | 2 | 75% | 3.5 | 171 | **12th** | Phase 4 (Intelligence) |

**RICE Methodology:**
- **Reach:** Users impacted per month (employees + managers)
- **Impact:** 3 = Massive (core functionality), 2 = High (important), 1 = Medium
- **Confidence:** % certainty in estimates
- **Effort:** Person-months of work
- **Score:** (Reach × Impact × Confidence) / Effort

---

## Build Phases & Timeline

### **Phase 0: Foundation (Months 1-3)**
**Goal:** Establish platform infrastructure and data foundation

**Modules to Build:**
1. API Gateway & Authentication (Module 9 - partial)
2. Database schema & core data models
3. Workforce Management P0 (Module 4 - partial)
4. DevOps setup (CI/CD, monitoring, environments)

**Deliverables:**
- ✅ API authentication working
- ✅ Employee CRUD APIs functional
- ✅ Development/staging/production environments
- ✅ CI/CD pipeline operational

**Team:** 4-5 engineers (2 backend, 1 DevOps, 1 frontend for admin panel, 1 QA)

---

### **Phase 1: Core Scheduling (Months 4-9)**
**Goal:** Build the heart of the system - scheduling with compliance and cost control

**Modules to Build:**
1. Compliance Engine (Module 2) - COMPLETE
2. Cost Management (Module 10) - COMPLETE
3. Scheduling Engine (Module 1) - COMPLETE
4. Forecasting Service (Module 3) - MVP

**Deliverables:**
- ✅ Manual schedule creation with compliance validation
- ✅ Real-time cost tracking during schedule building
- ✅ Automated schedule generation (basic)
- ✅ Simple demand forecasting (historical averages)
- ✅ Basic web UI for managers to create/edit schedules

**Team:** 6-8 engineers (3 backend, 1 ML, 2 frontend, 1 QA, 1 DevOps)

**Milestone:** Pilot with 2-3 stores using web UI only

---

### **Phase 2: User Experience (Months 10-14)**
**Goal:** Deliver mobile-first employee experience and self-service

**Modules to Build:**
1. Notification Service (Module 11) - COMPLETE
2. Mobile App (Module 7) - MVP (view schedules, basic clock-in)
3. Web Portal (Module 8) - Employee self-service
4. Workforce Management P1 (Module 4 - self-service features)

**Deliverables:**
- ✅ iOS + Android apps published to app stores
- ✅ Push notifications working
- ✅ Employees can view schedules, set availability, request leave
- ✅ Basic clock-in/out via mobile

**Team:** 8-10 engineers (2 backend, 2 mobile, 2 frontend, 1 QA, 1 DevOps)

**Milestone:** Rollout to 10-15 stores with full employee adoption

---

### **Phase 3: Operations (Months 15-18)**
**Goal:** Complete time tracking, leave management, and integrations

**Modules to Build:**
1. Time & Attendance (Module 5) - COMPLETE
2. Leave Management (Module 6) - COMPLETE
3. Integration Layer (Module 9) - Payroll, HRIS connectors
4. Mobile App enhancements (shift swapping, timesheet editing)

**Deliverables:**
- ✅ Full timesheet management with approvals
- ✅ Leave requests auto-update schedules
- ✅ Payroll integration exporting timesheets
- ✅ HRIS sync for employee data
- ✅ Shift marketplace functional

**Team:** 8-10 engineers (3 backend, 2 mobile, 2 frontend, 1 integration specialist, 1 QA)

**Milestone:** Full rollout to all stores

---

### **Phase 4: Intelligence (Months 19-24+)**
**Goal:** Advanced analytics, AI improvements, P2 features

**Modules to Build:**
1. Analytics & Reporting (Module 12) - COMPLETE
2. Forecasting improvements (ML model enhancements)
3. Workforce Management P2 (skills tracking, certifications)
4. Advanced scheduling features (what-if scenarios, optimization)

**Deliverables:**
- ✅ Executive dashboards with key metrics
- ✅ Custom report builder
- ✅ Improved ML forecasting accuracy (±5% variance)
- ✅ Skills & certification expiry tracking
- ✅ Multi-location workforce management

**Team:** 6-8 engineers (2 backend, 1 ML, 2 frontend/BI, 1 QA)

**Milestone:** Platform maturity and optimization

---

## Technology Stack Recommendations

### **Backend Services**
- **Primary languages:** Python (ML, data processing), Java/Spring Boot or Node.js (business logic)
- **Frameworks:** Spring Boot, Express.js, FastAPI
- **API style:** RESTful + GraphQL (optional for complex queries)

### **Frontend**
- **Web:** React + TypeScript, Material-UI or Tailwind CSS
- **Mobile:** Native (Swift + Kotlin) OR React Native/Flutter (faster but less native)

### **Data Layer**
- **Primary DB:** PostgreSQL (transactional data)
- **Time-series:** TimescaleDB or InfluxDB (attendance, metrics)
- **Cache:** Redis (session, frequently accessed data)
- **Search:** Elasticsearch (audit logs, full-text search)
- **Object storage:** AWS S3 or MinIO (photos, documents)

### **ML & Analytics**
- **ML:** Python (scikit-learn, Prophet, TensorFlow)
- **BI:** Apache Superset, Metabase, or Tableau
- **Data warehouse:** PostgreSQL with materialized views or ClickHouse

### **Infrastructure**
- **Cloud:** AWS, Azure, or GCP (recommend AWS for maturity)
- **Containers:** Docker + Kubernetes (EKS, AKS, GKE)
- **CI/CD:** GitHub Actions, GitLab CI, or Jenkins
- **Monitoring:** Prometheus + Grafana, Datadog, or New Relic
- **Logging:** ELK Stack (Elasticsearch, Logstash, Kibana) or Loki

### **Integrations**
- **API Gateway:** Kong, AWS API Gateway, or Traefik
- **Message queue:** RabbitMQ or Apache Kafka
- **ETL:** Apache Airflow or custom Python scripts
- **Notifications:** Firebase (push), Twilio (SMS), SendGrid (email)

---

## Critical Risks & Mitigation

### **High Risk**

**1. Scheduling Algorithm Complexity**  
**Risk:** Constraint optimization is NP-hard; may not scale or produce optimal schedules  
**Impact:** Poor schedule quality, manual adjustments needed (defeats purpose)  
**Mitigation:**
- Start with heuristic-based scheduler (greedy algorithm) for MVP
- Add optimization layer (OR-Tools, OptaPlanner) in Phase 1.5
- Set realistic expectations: "near-optimal" vs "perfect" schedules
- Build strong manual override capabilities
- Consider commercial optimization library (Gurobi) if budget allows

**2. ML Forecasting Accuracy**  
**Risk:** Predictions may be inaccurate for new stores, special events, or volatile demand  
**Impact:** Understaffing (poor service) or overstaffing (wasted cost)  
**Mitigation:**
- Start with simple statistical models (moving averages, seasonal decomposition)
- Add ML models (Prophet, LSTM) once sufficient training data (6+ months)
- Always allow manual override of forecasts
- Display confidence intervals to managers
- Build feedback loop: track actual vs predicted, retrain monthly

**3. Team Skill Gaps**  
**Risk:** In-house team lacks experience with optimization algorithms, ML, or mobile development  
**Impact:** Delays, technical debt, suboptimal solutions  
**Mitigation:**
- Hire 1-2 senior engineers with relevant expertise (scheduling/ML)
- Budget for external consultants for algorithm design (first 3 months)
- Training investment: online courses, conferences
- Start with well-documented libraries (OR-Tools, Prophet) vs building from scratch
- Consider staff augmentation for mobile app development

**4. Integration Complexity**  
**Risk:** Payroll/HRIS systems have poor APIs or lack documentation  
**Impact:** Manual data entry continues, defeating automation goals  
**Mitigation:**
- API discovery phase BEFORE Phase 1 starts (test integrations early)
- Build CSV import/export fallback for all integrations
- Allocate dedicated integration engineer (not split across other work)
- Budget for vendor support hours with Payroll/HRIS providers
- Phased approach: Start with CSV, add API integrations in Phase 3

---

### **Medium Risk**

**5. Scope Creep**  
**Risk:** Stakeholders request additional features during build, delaying MVP  
**Impact:** Timeline extends to 24+ months, costs increase  
**Mitigation:**
- Strict change control: All new features go to backlog for future phases
- Product owner empowered to say "no" or "later"
- Regular stakeholder demos showing progress (manage expectations)
- Document "out of scope" items clearly in Phase 0

**6. Data Migration Quality**  
**Risk:** Employee data from current system is incomplete, inconsistent, or incorrect  
**Impact:** Schedules fail validation, poor user experience, rework required  
**Mitigation:**
- Data audit in Phase 0 (before any development)
- Cleanup sprints: Assign HR staff to fix data issues (6-8 weeks)
- Build data validation tool: Identify gaps (missing skills, invalid contracts)
- Start with pilot stores with cleanest data
- Plan for manual data entry for problem records

**7. Mobile App Store Approval Delays**  
**Risk:** Apple/Google reject app submissions, delaying launch  
**Impact:** Phase 2 milestone missed, employees can't access schedules  
**Mitigation:**
- Study app store guidelines thoroughly before development
- Submit early for review (even beta versions)
- Have web-based fallback (responsive mobile web app)
- Budget 2-4 weeks for approval process in timeline

---

### **Low Risk (Monitor)**

**8. Performance at Scale**  
**Risk:** System slows down with large employee counts (1000+ employees, 50+ stores)  
**Mitigation:** Load testing starting Phase 1, database indexing, caching strategy, horizontal scaling

**9. User Adoption Resistance**  
**Risk:** Employees resist new system, prefer old manual processes  
**Mitigation:** Change management plan, training, champions program, user feedback loops

---

## Dependencies & Prerequisites

### **Before Phase 0 Starts**

**Hard Prerequisites:**
- ✅ **Budget approved:** Total project cost (team, infrastructure, licenses)
- ✅ **Team hired:** At minimum 4-5 engineers for Phase 0
- ✅ **Cloud infrastructure selected:** AWS/Azure/GCP decision made
- ✅ **Data audit completed:** Employee data quality assessed

**Recommended Prerequisites:**
- ⚠️ **Payroll/HRIS API access:** Sandbox credentials obtained (or CSV export process defined)
- ⚠️ **POS data access:** Historical sales data available (at least 12 months)
- ⚠️ **Product owner assigned:** Dedicated PM with decision-making authority
- ⚠️ **Pilot store selection:** 2-3 stores committed for Phase 1 testing

---

### **External Dependencies**

**Systems:**
- **Payroll system:** API or CSV export for timesheet submission
- **HRIS system:** Employee data sync (API preferred, CSV fallback)
- **POS system:** Sales data for forecasting (daily/hourly granularity)
- **ERP/Accounting:** Labor cost data (optional for Phase 1)

**Vendors:**
- **Cloud provider:** AWS/Azure/GCP account setup
- **Mobile infrastructure:** Apple Developer Account ($99/year), Google Play Console ($25 one-time)
- **Third-party services:** Twilio (SMS), SendGrid (email), Firebase (push notifications)
- **Optimization library:** Gurobi license (if using commercial solver - ~$3-5k/year)

**Organizational:**
- **Stakeholder availability:** Store managers for UAT testing (2-3 hours/week during Phase 1-2)
- **HR support:** Data cleanup and employee onboarding communications
- **Legal review:** Compliance rules documented (labor laws, awards) before Phase 1

---

## Success Metrics

### **Phase 1 Success (Month 9)**
- ✅ 2-3 pilot stores fully scheduled using the system (zero manual Excel)
- ✅ 100% compliance with labor laws (zero violations flagged)
- ✅ Schedule generation time: <30 minutes for 50-employee store
- ✅ Manager satisfaction: NPS >30 (pilot users)
- ✅ System uptime: 99.5%

### **Phase 2 Success (Month 14)**
- ✅ 10-15 stores live with mobile app access
- ✅ Employee app adoption: 80%+ weekly active users
- ✅ Push notification delivery: >95% success rate
- ✅ Mobile app rating: 4.0+ stars (App Store, Google Play)
- ✅ Self-service adoption: 60%+ employees set availability themselves

### **Phase 3 Success (Month 18)**
- ✅ All stores (50+) live on the platform
- ✅ Payroll integration: 100% timesheet auto-export (zero manual entry)
- ✅ Leave requests auto-update schedules: 100% accuracy
- ✅ Shift fill time: <24 hours average (via marketplace)
- ✅ System uptime: 99.9%

### **6-Month Post-Launch (Month 24)**
- ✅ Scheduling efficiency: 70% time reduction (7 hrs → 2 hrs/week per manager)
- ✅ Labor cost variance: Within ±3% of budget
- ✅ Compliance violations: Zero
- ✅ Employee adoption: 90%+ using app weekly
- ✅ Manager satisfaction: NPS >40
- ✅ Forecasting accuracy: ±10% variance (Phase 1), improving to ±5% (Phase 4)

---

## Cost & Resource Estimates

### **Team Composition by Phase**

| Phase | Duration | Backend | Mobile | Frontend | ML/Data | DevOps | QA | Total FTE |
|-------|----------|---------|--------|----------|---------|--------|----|----|
| **Phase 0** | 3 months | 2 | 0 | 1 | 0 | 1 | 1 | 5 |
| **Phase 1** | 6 months | 3 | 0 | 2 | 1 | 1 | 1 | 8 |
| **Phase 2** | 5 months | 2 | 2 | 2 | 0 | 1 | 1 | 8 |
| **Phase 3** | 4 months | 3 | 2 | 2 | 0 | 1 | 1 | 9 |
| **Phase 4** | 6+ months | 2 | 1 | 2 | 1 | 1 | 1 | 8 |

**Peak team size:** 9 engineers (Phase 3)  
**Average team size:** 7-8 engineers

---

### **Infrastructure Costs (Monthly - Rough Estimates)**

| Component | Phase 0-1 | Phase 2-3 | Phase 4+ (All Stores) |
|-----------|-----------|-----------|------------|
| **Compute** (EC2, K8s nodes) | $500 | $1,500 | $3,000 |
| **Database** (RDS PostgreSQL) | $200 | $500 | $1,200 |
| **Cache** (Redis) | $100 | $200 | $400 |
| **Storage** (S3, backups) | $50 | $200 | $500 |
| **Monitoring** (Datadog/New Relic) | $200 | $500 | $1,000 |
| **Notifications** (Twilio, SendGrid) | $100 | $500 | $1,500 |
| **CI/CD** (GitHub, build servers) | $100 | $200 | $300 |
| **Misc** (DNS, SSL, logs) | $50 | $100 | $200 |
| **TOTAL/month** | **~$1,300** | **~$3,700** | **~$8,100** |

**Annual infrastructure (steady state):** ~$100k  

---

### **One-Time Costs**

- **ML training compute:** $2-5k (GPU instances for model training)
- **Third-party licenses:** Gurobi optimizer ~$5k/year (optional)
- **Mobile app accounts:** $124 (Apple $99, Google $25)
- **Load testing tools:** $500-1000 (JMeter, k6, or commercial tools)
- **Security audit:** $10-20k (external penetration testing before full launch)

---

### **Total Cost Estimates (18 Months to Launch)**

| Cost Category | Low Estimate | High Estimate |
|---------------|--------------|---------------|
| **Engineering salaries** (avg $120k/year, 8 FTE avg) | $1.44M | $1.8M |
| **Infrastructure** (18 months) | $50k | $80k |
| **Third-party services** (SMS, email, etc.) | $10k | $20k |
| **Licenses & tools** | $20k | $40k |
| **Consultants** (algorithm design, audits) | $30k | $60k |
| **Contingency** (15%) | $230k | $300k |
| **TOTAL** | **$1.78M** | **$2.3M** |

**Ongoing annual costs (post-launch):**
- Team: $1.2-1.5M (6-8 engineers for maintenance, enhancements)
- Infrastructure: $100k
- Services: $30-50k
- **Total:** ~$1.35-1.65M/year

---

## Build vs Buy Decision Framework

### **When to Build In-House**

✅ **Build makes sense if:**
- Unique requirements not met by vendors (e.g., specific award interpretations)
- Long-term cost savings justify upfront investment (break-even ~3-5 years)
- Strategic IP ownership desired (scheduling algorithms, forecasting models)
- Existing engineering team has capacity and relevant expertise
- Willing to invest 12-18 months before full value realization
- Organization has strong product/engineering culture

---

### **When to Buy/Partner**

⚠️ **Consider vendor if:**
- Need solution operational in <6 months
- Limited engineering capacity (team focused on core business apps)
- Risk-averse organization (proven vendor solution safer than custom build)
- Total cost of ownership over 5 years lower for vendor (do the math)
- Vendor offers 80%+ feature match with acceptable compromises
- Integration capabilities meet your needs

---

### **Hybrid Approach**

💡 **Best of both worlds:**
- **Phase 1-2:** Use vendor solution to get operational quickly
- **Phase 3-4:** Build custom scheduling engine and forecasting (unique IP)
- **Keep vendor:** Time & attendance, leave management (commoditized features)
- **Integration layer:** Own the API platform, plug in best-of-breed modules

---

## Assumptions

1. **Team availability:** Can hire/allocate 5-9 engineers starting Month 1
2. **Data quality:** Employee master data is 80%+ accurate (requires cleanup, not rebuild)
3. **API access:** Payroll and HRIS systems have documented APIs OR CSV export is acceptable for Phase 1-2
4. **POS data:** Historical sales data available for at least 12 months at daily granularity (hourly preferred)
5. **Compliance complexity:** Labor laws can be codified into rules engine (not requiring AI interpretation)
6. **Scale:** System designed for up to 5,000 employees, 100 stores (horizontal scaling possible beyond)
7. **Mobile platforms:** iOS and Android only (no web mobile app as primary interface)
8. **Cloud deployment:** Kubernetes-based microservices architecture
9. **Forecasting:** Initial models use statistical methods; ML added in Phase 1.5-2
10. **Pilot success:** 2-3 stores willing to beta test and provide feedback starting Month 7

---

## Stakeholder Update (2-3 Sentences)

**Decision made:** Building rostering application in-house. Organized 15 features into **9 technical modules** across 4 phases, with critical path focusing on Scheduling Engine, Compliance Engine, and Cost Management. Phase 0 (Foundation) starts Month 1; pilot with 2-3 stores by Month 9; full rollout complete Month 18. **Timeline: 12-18 months to MVP**; estimated investment **$1.8-2.3M** for build, ~$1.5M/year ongoing maintenance.

---

## Next Steps

### **Immediate (Next 2 Weeks)**

1. **Finalize team:** Hire/assign 5 engineers for Phase 0 (2 backend, 1 frontend, 1 DevOps, 1 QA)
2. **Cloud setup:** Provision AWS/Azure/GCP accounts, set up dev/staging environments
3. **Data audit:** Export current employee data, assess quality, identify cleanup needs
4. **Integration discovery:** Get API documentation for Payroll, HRIS, POS systems
5. **Tool selection:** Choose specific tech stack components (Spring Boot vs Express, React vs Vue, etc.)

### **Month 1 (Phase 0 Kickoff)**

6. **Architecture design:** Detailed system design for API Gateway, Database schema
7. **DevOps setup:** CI/CD pipelines, monitoring, logging infrastructure
8. **First sprint:** Build employee CRUD APIs, authentication, basic admin UI
9. **Data cleanup:** HR team starts fixing employee data issues
10. **Pilot store selection:** Identify and onboard 2-3 pilot stores for Phase 1

### **Month 2-3 (Foundation Build)**

11. **Core APIs:** Complete Workforce Management P0 APIs
12. **Database migrations:** Set up schema versioning and migration tools
13. **Testing framework:** Unit tests, integration tests, load testing setup
14. **Compliance rules:** Document all labor laws, award rules to be codified
15. **Phase 1 planning:** Detailed sprint planning for Scheduling Engine development

---

## Appendix: Module Dependencies Diagram

```
┌─────────────────────────────────────────────────────────┐
│               START: Foundation Layer                    │
│  (Module 9: API Gateway + Module 4: Workforce Mgmt P0)  │
└──────────────────────┬──────────────────────────────────┘
                       │
           ┌───────────┴───────────┐
           ▼                       ▼
┌──────────────────────┐  ┌──────────────────────┐
│ Module 2:            │  │ Module 10:           │
│ Compliance Engine    │  │ Cost Management      │
└──────────┬───────────┘  └──────────┬───────────┘
           │                         │
           └───────────┬─────────────┘
                       ▼
           ┌───────────────────────┐
           │ Module 1:             │
           │ Scheduling Engine     │ ◄─── CRITICAL PATH
           └───────────┬───────────┘
                       │
                       ├────────────────────────┐
                       ▼                        ▼
           ┌───────────────────────┐  ┌──────────────────────┐
           │ Module 3:             │  │ Module 11:           │
           │ Forecasting Service   │  │ Notification Service │
           └───────────┬───────────┘  └──────────┬───────────┘
                       │                         │
           ┌───────────┴────────────┬────────────┘
           ▼                        ▼
┌──────────────────────┐  ┌──────────────────────┐
│ Module 7:            │  │ Module 8:            │
│ Mobile App           │  │ Web Portal           │
└──────────┬───────────┘  └──────────────────────┘
           │
           ▼
┌──────────────────────────────────────────┐
│ Module 5 & 6:                            │
│ Time & Attendance + Leave Management     │
└──────────┬───────────────────────────────┘
           ▼
┌──────────────────────────────────────────┐
│ Module 12:                               │
│ Analytics & Reporting Engine             │
└──────────────────────────────────────────┘
```

---

**Document Owner:** Product Management  
**Technical Review:** Engineering Leadership  
**Last Updated:** November 20, 2025  
**Version:** 1.0 - Initial Module Architecture

**Related Documents:**
- [ROSTERING_FEATURES_PRD.md](./ROSTERING_FEATURES_PRD.md) - Original feature requirements
- [FEATURES_CAPABILITIES_TABLE.md](./FEATURES_CAPABILITIES_TABLE.md) - Detailed capability reference
- [CAPABILITY_EXAMPLES_GUIDE.md](./CAPABILITY_EXAMPLES_GUIDE.md) - User story examples
