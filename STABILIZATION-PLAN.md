# AMAN Pre-Auth Intake Stabilization Plan

**Status:** IN PROGRESS  
**Created:** 2026-06-02  
**Updated:** 2026-06-10  
**Owner:** SaasPro Labs Engineering

---

## Current Situation

| Component | Status |
|-----------|--------|
| AMAN live PA webhook delivery | ✅ Confirmed working (2026-06-10) |
| Raw payload storage | ✅ Working |
| Webhook delivery logging | ✅ Implemented |
| Payload field extraction | ✅ Core mappings confirmed |
| PA event model | ✅ Clarified (no `pa.rejected`, see PA-005) |
| Agent auto-decisions | 🛑 PAUSED |

---

## Primary Goals

1. ✅ Add reliable webhook delivery logging
2. ⬜ Track received vs persisted requests (dashboard metrics)
3. ⬜ Normalize AMAN payload fields (pending AMAN confirmation)
4. ⬜ Clean bad historical rows with unknown patient/request IDs
5. ⬜ Understand duplicate/additional-item submissions
6. ⬜ Define criteria for safely re-enabling the agent
7. ✅ Prepare clarification list for AMAN (PA-005-AMAN-FIELD-CLARIFICATIONS.md)
8. ⬜ AMAN confirms mappings and documents responses

---

## Action Items

### Phase 1: Immediate (Agent Paused)

- [x] **P1-01:** Add `webhook_delivery_logs` table and logging
- [x] **P1-02:** Add `preauth_events` table for event-level tracking
- [x] **P1-03:** Draft AMAN clarification questions (PA-005)
- [ ] **P1-04:** Add feature flag to pause agent execution
- [ ] **P1-05:** Send PA-005 clarifications to AMAN contact
- [ ] **P1-06:** Create intake dashboard view for received vs persisted

### Phase 2: Data Cleanup

- [ ] **P2-01:** Query and identify rows with `patient_id = 'unknown'`
- [ ] **P2-02:** Query and identify rows with generated `request_id` (prefix `intake-`)
- [ ] **P2-03:** Analyze duplicate `event_id` patterns
- [ ] **P2-04:** Analyze same `checkin_id` with multiple `event_id` values
- [ ] **P2-05:** Document findings in PA-005 addendum

### Phase 3: Schema Normalization (Post-AMAN Response)

- [ ] **P3-01:** Update `category_id` mapping based on AMAN confirmation
- [ ] **P3-02:** Update `care_type` mapping based on AMAN confirmation
- [ ] **P3-03:** Update `limit_definition_id` mapping
- [ ] **P3-04:** Clarify item identifier strategy (`claim_item_id` vs `facility_tariff_item_id`)
- [ ] **P3-05:** Clarify cost field authority (`requested_cost` vs calculated)
- [ ] **P3-06:** Update event merge/replace logic based on AMAN guidance

### Phase 4: Agent Re-enablement

- [ ] **P4-01:** Define acceptance criteria checklist
- [ ] **P4-02:** Run agent on historical data in dry-run mode
- [ ] **P4-03:** Compare agent decisions vs AMAN manual decisions
- [ ] **P4-04:** Achieve >95% alignment on test set
- [ ] **P4-05:** Re-enable agent in advisory mode only
- [ ] **P4-06:** Graduate to enforcement mode after monitoring period

---

## Acceptance Criteria for Re-enablement

| Criterion | Target |
|-----------|--------|
| AMAN confirms all PA-005 field mappings | ✅ Required |
| Responses documented in PA-005 | ✅ Required |
| No `patient_id = 'unknown'` in last 7 days | ✅ Required |
| No `request_id` prefix `intake-` in last 7 days | ✅ Required |
| Agent dry-run alignment with AMAN decisions | ≥95% |
| Webhook delivery success rate | ≥99.5% |
| Mean processing latency | <500ms |

---

## Key Files

| File | Purpose |
|------|---------|
| `webhook/router.py` | Intake endpoint, field extraction, logging |
| `services/webhook_delivery.py` | Delivery log CRUD |
| `services/preauth_events.py` | Event persistence, dedup |
| `agent/agent.py` | Multi-agent decision pipeline |
| `PA-005-AMAN-FIELD-CLARIFICATIONS.md` | Questions for AMAN |

---

## SQL Queries for Investigation

### Find unknown patient_id rows
```sql
SELECT COUNT(*), DATE(received_at) as day
FROM preauth_logs
WHERE patient_id = 'unknown'
GROUP BY day
ORDER BY day DESC;
```

### Find generated request_id rows
```sql
SELECT COUNT(*), DATE(received_at) as day
FROM preauth_logs
WHERE request_id LIKE 'intake-%'
GROUP BY day
ORDER BY day DESC;
```

### Analyze checkin_id with multiple events
```sql
SELECT checkin_id, COUNT(DISTINCT event_id) as event_count
FROM preauth_events
GROUP BY checkin_id
HAVING COUNT(DISTINCT event_id) > 1
ORDER BY event_count DESC
LIMIT 20;
```

### Duplicate event_id occurrences
```sql
SELECT event_id, duplicate_count, first_seen_at, last_seen_at
FROM preauth_events
WHERE duplicate_count > 0
ORDER BY duplicate_count DESC
LIMIT 20;
```

---

## Next Steps

1. Send PA-005 clarifications to AMAN (email/Slack)
2. Implement agent pause flag in `agent/agent.py`
3. Run investigation queries and document findings
4. Await AMAN response and update mappings

---

*Last updated: 2026-06-10*
