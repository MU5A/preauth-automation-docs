# SAA-60: Safe Applied-Mode Item Categories

**Status:** DRAFT  
**Created:** 2026-06-10  
**Author:** SaasPro Labs Engineering  
**Ticket:** [SAA-60](https://linear.app/saasprolabs/issue/SAA-60)

---

## Purpose

Define the narrow first set of PA line items that can be considered for **applied mode** (auto-decisioning) after advisory QA validation. Everything outside this list remains in **advisory mode** (recommendation only, human review required).

---

## Guiding Principles

1. **Start narrow, expand cautiously** — Better to manually review safe items than auto-decide risky ones
2. **Deterministic over probabilistic** — Only auto-decide when rules are clear-cut
3. **Low-value first** — Cap auto-decisions by amount until trust is established
4. **Reversible preferred** — Favor auto-approving small items over auto-denying (denials have higher impact)
5. **Full audit trail required** — Every applied-mode action must be logged with reasoning

---

## Category Risk Classification

### AMAN Request Categories (`pa_items.category_id`)

| ID | Category | Risk Level | Applied Mode? | Notes |
|----|----------|------------|---------------|-------|
| 1 | Drugs and consumables | 🟡 MEDIUM | ✅ WITH LIMITS | Safe for tariff drugs ≤₦10,000 per item |
| 2 | Services and procedures | 🔴 HIGH | ⚠️ PARTIAL | Consultations only; exclude surgeries, procedures |
| 3 | Laboratory investigations | 🟢 LOW | ✅ YES | Routine labs are deterministic |
| 4 | Radiological investigations | 🔴 HIGH | ❌ NO | MRI, CT, X-ray — high-value, needs review |
| 5 | Dental care | 🟡 MEDIUM | ⚠️ PARTIAL | Checkups yes; extractions, surgeries no |
| 6 | Optical care | 🟡 MEDIUM | ⚠️ PARTIAL | Eye tests yes; surgeries, lenses no |
| 7 | Immunization and vaccine | 🟢 LOW | ✅ YES | Standard vaccines are deterministic |
| 8 | Wellness | 🟢 LOW | ✅ YES | Preventive care, low risk |

---

### Care Type Risk Classification (`encounter.care_type`)

| ID | Care Type | Risk Level | Applied Mode? | Notes |
|----|-----------|------------|---------------|-------|
| 1 | Inpatient | 🔴 HIGH | ❌ NO | Admissions, ward stays — always manual |
| 2 | Outpatient | 🟢 LOW | ✅ YES | Standard outpatient eligible for applied |
| 3 | Antenatal | 🔴 HIGH | ❌ NO | Maternity — complex, always manual |
| 4 | Dental Care | 🟡 MEDIUM | ⚠️ PARTIAL | Depends on procedure |
| 5 | Optical care | 🟡 MEDIUM | ⚠️ PARTIAL | Depends on procedure |
| 6 | Telemedicine | 🟢 LOW | ✅ YES | Virtual consultations, low risk |
| 7 | Wellness | 🟢 LOW | ✅ YES | Preventive, low risk |

---

## Applied Mode Allowlist (v1)

### ✅ AUTO-APPROVE Eligible

Items meeting ALL of the following criteria:

| Criterion | Threshold |
|-----------|-----------|
| Care type | Outpatient (2), Telemedicine (6), or Wellness (7) |
| Category | Labs (3), Immunization (7), Wellness (8), OR Drugs (1) ≤₦10k, OR Consultation (2) ≤₦15k |
| Item amount | ≤ ₦25,000 per line item |
| Total PA amount | ≤ ₦100,000 per PA request |
| Confidence | Agent confidence = HIGH |
| Eligibility | Enrollee active, policy active, not expired |
| Limits | Sufficient remaining balance (≥ requested amount) |
| Data completeness | All required fields present (no nulls in critical paths) |

### ❌ ALWAYS MANUAL (Denylist)

Force advisory mode if ANY of these conditions:

| Condition | Reason |
|-----------|--------|
| Care type = Inpatient (1) | Admissions require clinical judgment |
| Care type = Antenatal (3) | Maternity is complex, high-stakes |
| Category = Radiology (4) | High-value diagnostics (MRI, CT, etc.) |
| Any surgery keyword in item_name | Surgical procedures need review |
| Item amount > ₦50,000 | High-value items need human eyes |
| Total PA > ₦200,000 | Large requests need oversight |
| Agent confidence ≠ HIGH | Uncertainty = escalate |
| Missing eligibility data | Can't verify = don't auto-decide |
| Missing utilization/limits | Can't check limits = don't auto-decide |
| Enrollee near annual cap (≥80%) | Needs careful limit management |
| Diagnosis includes ICD codes for chronic/complex conditions | Clinical complexity |

---

## Confidence Thresholds

| Agent Confidence | Applied Mode Action |
|------------------|---------------------|
| HIGH | Eligible for auto-decision (if other criteria met) |
| MEDIUM | Advisory only — flag for review |
| LOW | Advisory only — escalate to medical team |

---

## Amount Thresholds

| Level | Per-Item Limit | Per-PA Limit | Action |
|-------|----------------|--------------|--------|
| Tier 1 (Auto) | ≤ ₦25,000 | ≤ ₦100,000 | Applied mode eligible |
| Tier 2 (Review) | ₦25,001 - ₦100,000 | ₦100,001 - ₦500,000 | Advisory + flag |
| Tier 3 (Escalate) | > ₦100,000 | > ₦500,000 | Advisory + escalate |

---

## Item Name Keyword Filters

### 🚫 Exclude from Applied Mode (surgery/procedure keywords)

```
surgery, surgical, operation, incision, excision, 
amputation, transplant, resection, biopsy, 
catheter, dialysis, chemotherapy, radiotherapy,
admission, ward, bed, icu, intensive care,
c-section, caesarean, delivery, labor, labour
```

### ✅ Safe for Applied Mode (routine keywords)

```
consultation, checkup, check-up, screening,
blood test, urine test, stool test, 
malaria test, typhoid test, fbc, rbs,
paracetamol, ibuprofen, antimalarial, antibiotic,
vaccination, immunization, vitamin
```

---

## Plan-Based Overrides

| Plan Tier | Applied Mode | Notes |
|-----------|--------------|-------|
| Bronze / Bronze Customize | ✅ Standard limits | Lower caps, simpler coverage |
| Silver | ✅ Standard limits | Standard coverage |
| Gold | ✅ Standard limits | Higher coverage, same rules |
| Platinum / Platinum Plus | ⚠️ Elevated review | VIP plans — tighter human oversight |

---

## Rollout Phases

### Phase 1: Labs & Immunizations Only (Week 1-2)
- Category 3 (Labs) + Category 7 (Immunizations)
- Outpatient only
- Amount ≤ ₦25,000
- Monitor 100% of decisions

### Phase 2: Add Basic Drugs & Consultations (Week 3-4)
- Add Category 1 (Drugs ≤ ₦10k) + Category 2 (Consultations ≤ ₦15k)
- Expand amount to ≤ ₦50,000 per PA
- Sample 50% for manual verification

### Phase 3: Expand Thresholds (Week 5+)
- Increase per-item to ₦50,000
- Increase per-PA to ₦150,000
- Add Wellness (8) and Telemedicine (6)
- Sample 25% for verification

---

## AMAN Approval Requirements

Before enabling applied mode:

- [ ] AMAN medical team reviews this document
- [ ] AMAN confirms allowlist categories
- [ ] AMAN confirms amount thresholds
- [ ] AMAN designates operational owner for escalations
- [ ] Written sign-off from AMAN leadership

---

## Success Criteria

1. Applied mode has a written allowlist and denylist ✅
2. Anything outside the allowlist remains advisory/manual review ✅
3. AMAN can confidently approve the scope before go-live

---

## Appendix: AMAN Category Reference

```
Category 1: Drugs and consumables
Category 2: Services and procedures  
Category 3: Laboratory investigations
Category 4: Radiological investigations
Category 5: Dental care
Category 6: Optical care
Category 7: Immunization and vaccine
Category 8: Wellness

Care Type 1: Inpatient
Care Type 2: Outpatient
Care Type 3: Antenatal
Care Type 4: Dental Care
Care Type 5: Optical care
Care Type 6: Telemedicine
Care Type 7: Wellness
```

---

*Last updated: 2026-06-10*
