# SAA-63: Applied Pilot Rollout Checklist

**Status:** DRAFT — REQUIRES AMAN SIGN-OFF  
**Created:** 2026-06-10  
**Author:** SaasPro Labs Engineering  
**Ticket:** [SAA-63](https://linear.app/saasprolabs/issue/SAA-63)

---

## Purpose

This checklist defines everything that must be true before enabling **applied mode** (automated PA decisions that write back to AMAN) for a limited pilot scope.

Applied mode means the AI's decisions are **enforced**, not just advisory. This is a significant step that requires explicit AMAN approval.

---

## Pre-Pilot Checklist

### 1. AMAN Written Approval ⬜

| Requirement | Status | Owner | Notes |
|-------------|--------|-------|-------|
| AMAN leadership approves pilot scope | ⬜ Pending | AMAN CEO / Medical Director | |
| Scope document signed (SAA-60) | ⬜ Pending | AMAN + SaasPro | |
| Pilot duration agreed (recommend 2 weeks) | ⬜ Pending | Both parties | |
| Success criteria defined | ⬜ Pending | Both parties | |
| Rollback criteria defined | ⬜ Pending | Both parties | |

**Sign-off required from:**
- [ ] AMAN CEO or delegate
- [ ] AMAN Medical Services lead (Dr Sam or delegate)
- [ ] SaasPro CTO (Mo)

---

### 2. Operational Owner Designated ⬜

| Requirement | Status | Owner | Notes |
|-------------|--------|-------|-------|
| AMAN escalation contact named | ⬜ Pending | AMAN | Who receives escalated PAs? |
| AMAN monitoring contact named | ⬜ Pending | AMAN | Who reviews daily reports? |
| SaasPro on-call contact named | ⬜ Pending | SaasPro | Who handles technical issues? |
| Escalation response time agreed | ⬜ Pending | Both | e.g., 30 min during business hours |

---

### 3. Allowlist & Thresholds Confirmed ⬜

Reference: [SAA-60 - Applied Mode Categories](./SAA-60-APPLIED-MODE-CATEGORIES.md)

| Requirement | Status | Threshold | Notes |
|-------------|--------|-----------|-------|
| Category allowlist approved | ⬜ Pending | Labs (3), Immunizations (7), Wellness (8) | |
| Category denylist approved | ⬜ Pending | Inpatient (1), Antenatal (3), Radiology (4) | |
| Per-item amount cap | ⬜ Pending | ≤ ₦25,000 | |
| Per-PA amount cap | ⬜ Pending | ≤ ₦100,000 | |
| Confidence threshold | ⬜ Pending | HIGH only | |
| Plan allowlist | ⬜ Pending | Bronze, Silver, Gold (exclude Platinum+) | |

---

### 4. Escalation Policy Confirmed ⬜

| Scenario | Action | Response Time |
|----------|--------|---------------|
| Agent confidence < HIGH | Advisory only, flag for review | Next business day |
| Amount > threshold | Advisory only, escalate to AMAN | 4 hours |
| Denylist category | Advisory only, manual review | Next business day |
| Agent DENY on active enrollee | Flag + notification to AMAN | 2 hours |
| AMAN override of agent decision | Log + include in weekly review | Weekly |
| System error / timeout | Pause applied mode, alert SaasPro | 15 min |

---

### 5. Audit Trail & Dashboard Visibility ⬜

| Requirement | Status | Notes |
|-------------|--------|-------|
| Every applied decision logged with timestamp | ⬜ Pending | `callback_mode = 'applied'` |
| Agent reasoning captured | ⬜ Pending | `agent_result.reasoning` |
| AMAN can view all applied decisions in dashboard | ⬜ Pending | Filter by `callback_mode` |
| Decision export available (CSV/PDF) | ⬜ Pending | Audit compliance |
| Comparison job shows applied vs advisory split | ⬜ Pending | SAA-53 |

---

### 6. Incident Rollback Process ⬜

| Trigger | Action | Owner |
|---------|--------|-------|
| 3+ mismatch escalations in 1 hour | Pause applied mode automatically | System |
| AMAN requests pause | Disable within 5 minutes | SaasPro |
| System error rate > 5% | Pause + alert | System |
| SaasPro identifies critical bug | Pause + hotfix | SaasPro |

**Pause Switch Location:**
- Environment variable: `AGENT_ENABLED=false`
- Dashboard toggle: Settings → Applied Mode → OFF
- Emergency: Contact SaasPro on-call

**Rollback does NOT affect:**
- Historical decisions (already applied)
- Advisory recommendations (continue as normal)
- Dashboard visibility (all data retained)

---

### 7. Weekly QA Review Rhythm ⬜

| Day | Activity | Owner |
|-----|----------|-------|
| Monday | Generate weekly comparison report | SaasPro |
| Monday | Share report with AMAN medical team | SaasPro |
| Tuesday-Wednesday | AMAN reviews mismatches | AMAN |
| Thursday | Joint call to discuss findings | Both |
| Friday | Implement agreed rule updates | SaasPro |

**Metrics reviewed weekly:**
- Total PAs processed (applied vs advisory)
- Accuracy rate (agent vs AMAN alignment)
- Mismatch breakdown by category
- Amount delta distribution
- Processing latency (agent vs AMAN turnaround)

---

### 8. Advisory-Only Categories ⬜

The following categories remain **advisory only** (no auto-apply) even after pilot launch:

| Category | Reason |
|----------|--------|
| Inpatient (care_type 1) | Admissions require clinical judgment |
| Antenatal (care_type 3) | Maternity complexity |
| Radiology (category 4) | High-value diagnostics |
| Surgeries (keyword filter) | Procedural risk |
| Amount > ₦100,000 | High-value review |
| Platinum+ plans | VIP oversight |
| Missing eligibility data | Cannot verify |
| Near-limit enrollees (>80% used) | Limit management |

---

## Go-Live Checklist (Day of Launch)

| Step | Status | Owner |
|------|--------|-------|
| 1. All pre-pilot items checked ✓ | ⬜ | Both |
| 2. Production deploy completed | ⬜ | SaasPro |
| 3. `AGENT_ENABLED=true` with applied scope | ⬜ | SaasPro |
| 4. Test PA submitted and verified | ⬜ | SaasPro |
| 5. AMAN confirms receipt of test decision | ⬜ | AMAN |
| 6. Monitoring dashboard open | ⬜ | SaasPro |
| 7. On-call contacts confirmed available | ⬜ | Both |
| 8. Go/No-Go call completed | ⬜ | Both |

---

## Success Criteria (End of Pilot)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Accuracy rate | ≥ 95% | Agent matches AMAN on scored PAs |
| Amount accuracy | ≤ 5% delta | Within tolerance on approved amounts |
| Processing time | < 10 seconds | Agent decision latency |
| Escalation rate | < 5% | PAs requiring manual intervention |
| Zero critical misses | 0 | No wrong denials on valid enrollees |
| AMAN satisfaction | Positive | Qualitative feedback |

---

## Signatures

| Role | Name | Date | Signature |
|------|------|------|-----------|
| AMAN Leadership | | | |
| AMAN Medical Services | | | |
| SaasPro CTO | Mo (Mousa Mansa) | | |
| SaasPro Engineering | | | |

---

## Appendix: Quick Reference

**Pause Applied Mode:**
```bash
# Via environment
AGENT_ENABLED=false

# Via API (if implemented)
POST /admin/applied-mode/pause
```

**Check Applied Mode Status:**
```bash
GET /health
# Returns: { "applied_mode": true/false, ... }
```

**Emergency Contacts:**
- SaasPro: [TBD]
- AMAN: [TBD]

---

*Last updated: 2026-06-10*
