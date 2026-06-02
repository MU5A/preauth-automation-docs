# PA-005: AMAN Field Clarification Request

**Status:** PENDING AMAN RESPONSE  
**Created:** 2026-06-02  
**Priority:** BLOCKER — Agent decisions paused until resolved

---

## Context

We are stabilizing the AMAN HMO pre-authorization webhook intake pipeline before resuming automated decisions. We need AMAN to confirm field mappings and semantics to ensure correct interpretation of payloads.

---

## Questions for AMAN

### 1. Status Fields

**Q1.1:** What are all possible values for `enrollee.status`?  
- We currently see `1` — does `1` always mean active/eligible?  
- What other values exist (e.g., `0`, `2`, strings like `"inactive"`)?

**Q1.2:** What are all possible values for `policy.policy_status`?  
- Same question — does `1` always mean active?  
- What indicates suspended, expired, cancelled, etc.?

---

### 2. Category & Type Mappings

**Q2.1:** What is the official mapping for `pa_items.category_id`?  
Our current assumption:
```
1 = Drugs and consumables
2 = Services and procedures
3 = Laboratory investigations
4 = Radiological investigations
5 = Dental care
6 = Optical care
7 = Immunization and vaccine
8 = Wellness
```
**Please confirm or correct.**

**Q2.2:** What is the official mapping for `encounter.care_type`?  
Our current assumption:
```
1 = Inpatient
2 = Outpatient
3 = Antenatal
4 = Dental Care
5 = Optical care
6 = Telemedicine
7 = Wellness
```
**Please confirm or correct.**

**Q2.3:** Should we rely on `checkin_type` text (e.g., `"Out-patient"`, `"Telemedicine"`) or `care_type` number for clinical workflow classification?

---

### 3. PA Identity & Event Updates

**Q3.1:** Should `encounter.checkin_id` be treated as the main PA/encounter identifier?

**Q3.2:** When the same `checkin_id` appears multiple times with different `event_id` values, what does this mean?
- Additional items being added to an existing PA?
- An edited/amended request?
- A full replacement snapshot that supersedes prior events?

**Q3.3:** Should the latest payload replace prior context entirely, or should we merge all payloads for the same `checkin_id`?

---

### 4. Item Identifiers & Decisioning

**Q4.1:** Which identifier should we use when referring to or deciding on a specific item?
- `claim_item_id`
- `facility_tariff_item_id`
- `submission.items_added.id`

**Q4.2:** Is `pa_items` the **full current PA item set** (all items in the request)?

**Q4.3:** Is `submission.items_added` **only the newly added items** for that specific event?

**Q4.4:** Which cost field is authoritative for decisioning?
- `pa_items.requested_cost`
- `pa_items.unit_cost * quantity`
- `submission.items_added.requested_cost`

---

### 5. Item Counts

**Q5.1:** What exactly does `encounter.item_counts.pending` represent?

**Q5.2:** Why does `item_counts.pending` sometimes differ from the number of items in `pa_items` with `status: "pending"`?

---

### 6. Consumption & Limits

**Q6.1:** What does it mean when `consumption.enrollee_limits` and `consumption.policy_limits` are empty arrays?
- No limits apply to this member?
- Limits data is unavailable/not loaded?
- Limits are not configured for this plan?

**Q6.2:** What is the official mapping for `limit_definition_id`?  
We see IDs like `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `10`, `16`, `17`, `18`, `19`, `20`.  
What does each ID represent (e.g., Outpatient, Inpatient, Dental, Optical)?

**Q6.3:** Are limits already enforced/checked before the payload is sent to us, or is enforcement our responsibility?

---

### 7. Proposed Impact

**Q7.1:** What does `proposed_impact.status = "allowed"` mean?
- Is it an advisory signal only?
- Should it influence our approve/reject decisions?

**Q7.2:** Under what cases will `proposed_impact.violations` be populated?

---

### 8. Comments & Operational Notes

**Q8.1:** Should `submission.comment` and `pa_comments` be treated as required review context?

**Q8.2:** Are PA comments usually provider notes, AMAN reviewer notes, or both?

**Q8.3:** Should any request with comments be automatically escalated for human review?

---

### 9. Manual Pricing & Tariff Issues

**Q9.1:** What does `pricing_source = "manual"` mean operationally?
- Item not in facility tariff?
- Manually entered price?
- Something else?

**Q9.2:** Should manual-priced items be treated normally, or should they be escalated?

**Q9.3:** How should we handle comments mentioning "tariff issue" or "portal issue"?

---

### 10. Decision Workflow & Rules

**Q10.1:** What are AMAN's exact approval, rejection, and escalation rules we should follow?

**Q10.2:** Which cases should **never** be auto-approved (always require human review)?

**Q10.3:** Which cases should **always** be escalated to AMAN staff?

---

## Response Instructions

Please provide responses in this format:

```
## Q1.1 Response
[Your answer]

## Q1.2 Response
[Your answer]

...
```

Or provide a comprehensive mapping document if available.

---

## Contact

**SaasPro Labs Team**  
Pre-Auth Automation Integration

---

*These answers will be documented in our integration specification (PA-005) and used to configure the automated decision pipeline.*
