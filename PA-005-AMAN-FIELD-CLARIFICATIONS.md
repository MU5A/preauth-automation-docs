# PA-005: AMAN Field Clarification Request

**Status:** ✅ PARTIALLY RESOLVED  
**Created:** 2026-06-02  
**Updated:** 2026-06-06  
**Priority:** MEDIUM — Core mappings confirmed, some items still pending

---

## Responses from AMAN

### ✅ CONFIRMED (2026-06-01, via Sakeenah)

#### Status Fields
| Field | Value | Meaning |
|-------|-------|---------|
| `enrollee.status` | 1 | Active |
| `enrollee.status` | 0 | Inactive |
| `policy.policy_status` | 1 | Active |
| `policy.policy_status` | 0 | Inactive |

#### PA Item Status (`pa_items.status`)
| Value | Meaning |
|-------|---------|
| 0 | Pending |
| 1 | Approved |
| 2 | Queried |
| 3 | Rejected |

#### Request Categories (`pa_items.category_id`)
| ID | Category |
|----|----------|
| 1 | Drugs and consumables |
| 2 | Services and procedures |
| 3 | Laboratory investigations |
| 4 | Radiological investigations |
| 5 | Dental care |
| 6 | Optical care |
| 7 | Immunization and vaccine |
| 8 | Wellness |

#### Care Types (`encounter.care_type`)
| ID | Care Type |
|----|-----------|
| 1 | Inpatient |
| 2 | Outpatient |
| 3 | Antenatal |
| 4 | Dental Care |
| 5 | Optical care |
| 6 | Telemedicine |
| 7 | Wellness |

#### Empty Consumption Limits
When `consumption.enrollee_limits` and `consumption.policy_limits` are empty arrays:
- **Treat as:** Limits not configured (not "unlimited" or "unavailable")

---

### ⏳ PENDING (2026-06-03, via Sakeenah/Imran)

#### `limit_definition_id` Mapping
- Sakeenah confirmed a mapping exists and will provide it
- Still waiting for the mapping table
- Example IDs seen: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 16, 17, 18, 19, 20

**Sample limit object:**
```json
{
  "period": "annual",
  "target_id": null,
  "unit_label": "₦",
  "limit_value": 2100000,
  "metric_name": "Amount",
  "metric_type": "monetary",
  "target_type": "plan",
  "consumed_value": 0,
  "remaining_value": 2100000,
  "limit_definition_id": 11
}
```

#### Final Decision Webhook
- Mo requested a webhook/endpoint to see final AMAN decisions for QA comparison
- Imran: "This is noted and will be reviewed. You will be notified when/if it's available."

---

## Original Questions

### 1. Status Fields ✅ RESOLVED

**Q1.1:** What are all possible values for `enrollee.status`?  
**A:** `1` = active, `0` = inactive

**Q1.2:** What are all possible values for `policy.policy_status`?  
**A:** `1` = active, `0` = inactive

---

### 2. Category & Type Mappings ✅ RESOLVED

**Q2.1:** What is the official mapping for `pa_items.category_id`?  
**A:** See table above — confirmed correct.

**Q2.2:** What is the official mapping for `encounter.care_type`?  
**A:** See table above — confirmed correct.

**Q2.3:** Should we rely on `checkin_type` text or `care_type` number?  
**A:** Not explicitly answered. We use `care_type` number with text as fallback.

---

### 3. PA Identity & Event Updates ⚠️ NOT YET ASKED

Still need clarification on:
- `checkin_id` as main PA identifier
- Multiple events for same `checkin_id`
- Merge vs replace semantics

---

### 4. Item Identifiers & Decisioning ⚠️ NOT YET ASKED

Still need clarification on:
- Primary item identifier to use
- `pa_items` vs `submission.items_added` scope
- Authoritative cost field

---

### 5. Item Counts ⚠️ NOT YET ASKED

Still need clarification on:
- `item_counts.pending` semantics
- Why counts sometimes don't match item array

---

### 6. Consumption & Limits ⏳ PARTIALLY RESOLVED

**Q6.1:** Empty limits meaning?  
**A:** ✅ Treat as "limit not configured"

**Q6.2:** `limit_definition_id` mapping?  
**A:** ⏳ Sakeenah will provide — still waiting

**Q6.3:** Pre-enforcement by AMAN?  
**A:** Not answered

---

### 7-10. Other Questions ⚠️ NOT YET ASKED

These remain unanswered:
- Proposed impact semantics
- Comment handling rules
- Manual pricing treatment
- Decision workflow rules

---

## Implementation Status

### Applied to Code ✅
- `agent/agent.py` updated with confirmed mappings:
  - `CARE_TYPE_BUCKETS` — care_type → bucket mapping
  - `CATEGORY_BUCKET_OVERRIDES` — category_id → bucket mapping
  - `CATEGORY_LABELS` — category_id → display name
  - `CARE_TYPE_LABELS` — care_type → display name
  - `_item_status()` — already had correct 0/1/2/3 mapping

### Still Needed
- [ ] `limit_definition_id` mapping from AMAN
- [ ] Final decision webhook from AMAN (for QA comparison)

---

## Timeline

| Date | Event |
|------|-------|
| 2026-06-01 | Mo sent clarification questions to AMAN group |
| 2026-06-01 | Sakeenah responded with status, category, care_type mappings |
| 2026-06-03 | Mo asked about `limit_definition_id` mapping |
| 2026-06-03 | Sakeenah: "I'll get it for you and revert" |
| 2026-06-03 | Imran confirmed `limit_definition_id` is the limit definition object ID |
| 2026-06-04 | Sakeenah confirmed receiving AI recommendations |
| 2026-06-04 | Mo requested final decision data for QA |
| 2026-06-04 | Imran: "will be reviewed, you will be notified" |
| 2026-06-05 | Sakeenah left first comment on dashboard |
| 2026-06-05 | Mo replied to comment for PA CH/2026/06/04/0014602 |
| 2026-06-06 | Mappings applied to codebase |

---

## Contact

**SaasPro Labs Team**  
Pre-Auth Automation Integration

---

*Last updated: 2026-06-06*
