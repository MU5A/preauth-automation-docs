# AMAN Webhook Event Model

**Created:** 2026-06-10  
**Source:** Clarification from AMAN team (Imran)

---

## Overview

AMAN sends webhook events when pre-authorization requests are processed. Understanding the event model is critical for proper integration.

---

## Event Types

### `pa.approved`

**Meaning:** The PA request was received and processed by AMAN.

> ⚠️ **Important:** This does NOT mean "all items were approved." It means "we finished processing your request."

**When it fires:**
- After AMAN completes processing a PA request
- This is the ONLY PA-level event type

**What to do:**
1. Parse the payload for line-item decisions
2. Check `line_outcomes[].aman_decisions` for individual verdicts
3. Update your records accordingly

### `pa.rejected` — Does NOT Exist

There is no `pa.rejected` event. The PA itself is always "approved" in the sense that it was received and processed. Individual items within the PA can be rejected.

---

## Decision Levels

| Level | Where to Look | Possible Values |
|-------|---------------|-----------------|
| PA level | Event type | Always `pa.approved` |
| Line item | `line_outcomes[].aman_decisions` | Approve / Reject per item |
| Item status | `pa_items[].status` | 0=Pending, 1=Approved, 2=Queried, 3=Rejected |

---

## Payload Structure

```json
{
  "event_type": "pa.approved",
  "checkin_id": "CH/2026/06/10/0001234",
  "pa_items": [
    {
      "id": 12345,
      "status": 1,
      "item_name": "Blood Test",
      "category_id": 3
    },
    {
      "id": 12346,
      "status": 3,
      "item_name": "MRI Scan",
      "category_id": 4
    }
  ],
  "line_outcomes": [
    {
      "item_id": 12345,
      "aman_decisions": {
        "approved": true,
        "reason": "Within policy limits"
      }
    },
    {
      "item_id": 12346,
      "aman_decisions": {
        "approved": false,
        "reason": "Exceeds annual limit"
      }
    }
  ]
}
```

---

## Implementation Checklist

- [ ] Listen for `pa.approved` events only
- [ ] Never wait for `pa.rejected` (it won't come)
- [ ] Parse `line_outcomes[].aman_decisions` for verdicts
- [ ] Use `pa_items[].status` as authoritative item status
- [ ] Handle mixed outcomes (some items approved, some rejected)

---

## Related Documents

- [PA-005 Field Clarifications](./PA-005-AMAN-FIELD-CLARIFICATIONS.md) — Full field mapping reference
- [Stabilization Plan](./STABILIZATION-PLAN.md) — Integration status and roadmap

---

*Last updated: 2026-06-10*
