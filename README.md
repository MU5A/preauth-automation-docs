# Pre-Authorization Automation Documentation

Visual diagrams explaining the AI-assisted pre-authorization workflow.

## Files

- **`preauth-automation-flow.drawio`** - Main workflow diagram showing the automation integration
- **`WEBHOOK-EVENT-MODEL.md`** - AMAN webhook event types and payload structure
- **`PA-005-AMAN-FIELD-CLARIFICATIONS.md`** - Field mapping reference (status codes, categories, etc.)
- **`STABILIZATION-PLAN.md`** - Integration status and roadmap
- **`ROADMAP-Q3-Q4-2026.md`** - Future feature roadmap

## How to View/Edit

1. Go to [app.diagrams.net](https://app.diagrams.net/)
2. File → Open from → GitHub
3. Select this repository and the `.drawio` file

Or directly import:
- File → Import from → Device → select the downloaded `.drawio` file

## Workflow Overview

The automation sits on top of your existing system:

1. **Provider** submits pre-auth request through existing platform
2. **Your System** sends request to automation via webhook/API
3. **Automation Service** processes:
   - Extracts request details
   - Checks plan & eligibility
   - Validates against policy/rules
   - Reviews utilization history
   - Generates recommendation (Approve/Decline/Escalate)
4. **Result returned** via database update, API response, or workflow status
5. **Your Team** handles escalations, exceptions, and high-risk cases

### Key Benefits
- ✅ Minimal disruption to infrastructure
- ✅ No changes to existing workflows  
- ✅ Faster turnaround time
- ✅ Human team stays in control
- ✅ Handles repetitive validation automatically
