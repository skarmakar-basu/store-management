# Staff Rostering Solution - Research & Requirements

**Status:** ✅ Research Complete  
**Last Updated:** November 20, 2025  
**Owner:** Product Management

---

## 📋 Quick Summary

Research completed on rostering solution requirements for retail chain with mixed workforce (full-time + part-time contractors) and varying store formats. Identified **16 critical features** across 3 priority tiers.

**Key Findings:**
- **6 P0 (must-have) features** are deal-breakers: automated scheduling, task-based scheduling (format-dependent), demand forecasting, labor cost management, compliance engine, and mobile app
- **5 P1 (high-priority) features** provide competitive advantage and ROI
- **5 P2 (nice-to-have) features** add incremental value
- Task-based scheduling is critical for operational formats (grocery/supermarket) but optional for boutique/fashion stores
- Expected ROI: 70% reduction in scheduling time, ±3% budget variance, zero compliance violations, 98%+ task completion

---

## 📁 Documentation

| Document | Purpose | Audience |
|----------|---------|----------|
| **[ROSTERING_FEATURES_PRD.md](./ROSTERING_FEATURES_PRD.md)** | Comprehensive PRD with feature details, RICE scoring, risks, and implementation roadmap | Product, Engineering, Leadership |
| **[FEATURE_CHECKLIST.md](./FEATURE_CHECKLIST.md)** | Practical vendor evaluation checklist with demo scripts and red flags | Procurement, Evaluation Team |
| **This README** | Executive overview and navigation | All Stakeholders |

---

## 🎯 Top 6 Must-Have Features (P0)

### 1. **Automated Schedule Generation**
AI-driven scheduling that eliminates 80% of manual planning time by handling availability, skills, and business constraints automatically.

### 2. **Task & Activity-Based Scheduling**
Calculates manpower requirements based on operational tasks (deliveries, stocking, cleaning) alongside customer service needs. Critical for grocery/supermarket formats; configurable per store type. Prevents operational failures and optimizes labor allocation.

### 3. **Demand-Based Forecasting**  
Aligns staffing with predicted foot traffic using historical sales data—prevents both understaffing (lost revenue) and overstaffing (wasted cost). Integrates with task-based scheduling for unified manpower planning.

### 4. **Labor Cost Management**  
Real-time budget tracking with proactive overtime alerts; critical for P&L control and supporting different pay rates for FTE vs contractors.

### 5. **Compliance Engine**  
Automated enforcement of labor laws (max hours, rest periods, breaks) with audit trails—mitigates legal risk and potential $10k+ fines per violation.

### 6. **Mobile App**  
Native iOS/Android apps for schedule viewing, clock in/out (GPS verified), time-off requests, and shift swapping—meets modern workforce expectations.

---

## 💰 Expected Business Impact

| Metric | Target (6 months post-launch) |
|--------|-------------------------------|
| **Scheduling Efficiency** | 70% time reduction (7 hrs → 2 hrs/week) |
| **Labor Cost Accuracy** | Within ±3% of budget |
| **Compliance** | Zero violations |
| **Task Completion** | 98%+ P0 operational tasks completed on time |
| **Employee Adoption** | 90%+ using mobile app weekly |
| **Manager Satisfaction** | NPS >40 |
| **Operational Speed** | Fill open shifts in <24 hours |

---

## ⚠️ Critical Risks

1. **Integration Delays** (High Risk)  
   Payroll/HRIS APIs may have limitations or poor documentation  
   → *Mitigation:* Demand sandbox access during eval; POC integrations before contract

2. **Data Migration Quality** (High Risk)  
   Employee data from current system may be incomplete/inconsistent  
   → *Mitigation:* Data audit 8 weeks pre-launch; cleanup sprints

3. **User Adoption Resistance** (Medium Risk)  
   Older workforce may resist mobile-first tools  
   → *Mitigation:* Multi-modal access (web/kiosk); champions program; training

---

## 🗓️ Recommended Timeline

| Phase | Duration | Key Activities |
|-------|----------|----------------|
| **Vendor Shortlist** | Weeks 1-2 | Research 5-7 solutions matching P0 criteria |
| **RFP & Demos** | Weeks 3-6 | Detailed demos, scoring, reference calls |
| **Proof of Concept** | Weeks 7-10 | Test integrations + workflows with 1-2 pilot stores |
| **Contract Negotiation** | Weeks 11-12 | Finalize pricing, SLA, data terms |
| **Phased Rollout** | Months 4-6 | Pilot → region → full chain |

**Total Time to Full Deployment:** ~5-6 months

---

## 🏆 Vendor Evaluation Weights

Use these weights when scoring vendors:

- **Technical Fit (40%):** P0 completeness, integrations, mobile quality
- **Usability (25%):** Manager ease-of-use, employee mobile experience
- **Cost (20%):** Total 3-year cost including hidden fees
- **Vendor Viability (15%):** Market presence, support quality, roadmap

---

## 📞 Suggested Vendor Shortlist

Based on market research, consider demos from:

1. **Deputy** - Strong mobile-first, popular in retail
2. **When I Work** - Good for SMB retail chains
3. **Workforce.com** - Enterprise-grade, robust forecasting
4. **RosterElf** - Australia/region focus, compliance-strong
5. **Legion** - AI-driven demand forecasting
6. **7shifts** - Hospitality/retail hybrid

*(Full evaluation via checklist recommended)*

---

## 🔍 Key Questions for Vendors

During demos, validate these critical requirements:

1. **"Can your system handle task-based scheduling where we define operational activities like 'Delivery unloading Mon/Wed/Fri 7-9 AM needs 3 people with forklift cert'?"**  
   *(Tests task-based scheduling capability)*

2. **"How does your system integrate task requirements with demand-based customer service staffing? Can we see a unified schedule?"**  
   *(Tests integration between task-based and demand forecasting)*

3. **"Can we configure task-based mode for some stores (like supermarkets) but disable it for others (like boutiques)?"**  
   *(Tests store-specific configurability)*

4. **"Can you enforce part-time contractors can't exceed 20 hours/week while FTE can work 40?"**  
   *(Tests contract hour differentiation)*

5. **"Walk us through the payroll integration with [YOUR SYSTEM]—how do approved timesheets sync?"**  
   *(Tests integration maturity)*

6. **"Show us what happens when a manager tries to schedule someone 10 days straight."**  
   *(Tests compliance engine)*

7. **"Can employees view their schedule without internet on the mobile app?"**  
   *(Tests offline functionality)*

8. **"How do you handle different state labor laws across our locations?"**  
   *(Tests multi-jurisdiction compliance)*

---

## ✅ Next Steps

1. Review [ROSTERING_FEATURES_PRD.md](./ROSTERING_FEATURES_PRD.md) for detailed requirements
2. Use [FEATURE_CHECKLIST.md](./FEATURE_CHECKLIST.md) during vendor demos
3. Initiate vendor shortlist research (Week 1)
4. Secure stakeholder alignment on P0 non-negotiables
5. Confirm integration requirements with IT/payroll teams

---

## 📊 Assumptions

This analysis assumes:
- **Chain Size:** 10-50 stores
- **Workforce:** 200-1000 employees (FTE + contractors)
- **Current State:** Manual scheduling or basic spreadsheets
- **Tech Stack:** Existing payroll/HRIS with APIs
- **Geographic:** Moderate-to-strict labor law regions
- **Seasonality:** 20-30% demand fluctuation

*If your context differs significantly, priorities may need adjustment.*

---

## 📝 Document History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | Nov 20, 2025 | Initial research and PRD | Product Management |

---

For questions or feedback, contact the product team.
