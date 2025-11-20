# Staff Rostering Application - Build Modules Architecture (Short Version)

**Version:** 1.0  
**Date:** November 20, 2025  
**Status:** Build Decision - Module Planning

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

### **Module 1: Scheduling Engine**
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

---

### **Module 2: Compliance Engine**
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

---

### **Module 3: Forecasting Service**
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

---

### **Module 4: Workforce Management**
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

---

### **Module 5: Time & Attendance Tracking**
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

---

### **Module 6: Leave Management System**
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

---

### **Module 7: Mobile App (iOS & Android)**
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

---

### **Module 8: Web Portal (Employee & Manager)**
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

---

### **Module 9: Integration Layer & API Platform**
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

---

### **Module 10: Cost Management & Budgeting**
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

---

### **Module 11: Notification & Communication Service**
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

---

### **Module 12: Analytics & Reporting Engine**
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

---

**Document Owner:** Product Management  
**Technical Review:** Engineering Leadership  
**Last Updated:** November 20, 2025  
**Version:** 1.0 - Short Version

**Related Documents:**
- [BUILD_MODULES_ARCHITECTURE.md](./BUILD_MODULES_ARCHITECTURE.md) - Full detailed architecture document
- [ROSTERING_FEATURES_PRD.md](./ROSTERING_FEATURES_PRD.md) - Original feature requirements
- [FEATURES_CAPABILITIES_TABLE.md](./FEATURES_CAPABILITIES_TABLE.md) - Detailed capability reference

