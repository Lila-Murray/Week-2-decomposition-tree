# Digital Appointment and Referral Platform

## Revised decomposition tree

- **Digital Appointment and Referral Platform** — *Functional*
  - **Appointment Scheduling** — *Functional*
    - Maintain provider, clinic, service, and slot availability — *Process*
    - Create appointment — *Process*
    - Change or cancel appointment — *Process*
    - Manage waitlist and released slots — *Process*
    - Appointment — *Object*
    - Availability slot — *Object*
  - **Referral Intake and Clinical Routing** — *Functional*
    - Receive and validate GP referral — *Process*
    - Record referral and supporting documents — *Process*
    - Triage referral — *Process*
    - Route referral to specialty or service — *Process*
    - Referral — *Object*
    - Clinical attachment — *Object*
  - **Patient and Referrer Self-Service** — *Functional*
    - Patient booking, cancellation, and rescheduling — *Process*
    - GP referral submission and status tracking — *Process*
    - Patient and GP portal access — *Functional*
  - **Notifications** — *Functional*
    - Generate booking, change, cancellation, reminder, and referral-status messages — *Process*
    - Deliver message by SMS, email, or portal — *Process*
    - Notification — *Object*
    - Communication channel — *Object*
  - **Administration and Assurance** — *Functional*
    - Manage clinics, services, providers, schedules, and booking rules — *Process*
    - Manage user roles and access — *Process*
    - Capture consent and privacy preferences — *Process*
    - Maintain audit history — *Process*
    - Produce operational and compliance reports — *Process*
  - **External Interoperability** — *Functional*
    - Exchange patient and appointment data with the electronic medical record — *Process*
    - Exchange referrals and status updates with GP systems — *Process*
    - Integrate with messaging providers — *Process*

## Critical review

The original tree mixed capabilities, workflows, channels, and integrations at the same level. Patient communications overlapped with notification steps in booking and referral workflows, while platform services combined unrelated areas such as security, reporting, configuration, and integration. The revised tree separates these responsibilities and adds patient self-service, privacy preferences, operational configuration, and exception-related waitlist management.

## Least-certain branches

1. **Manage waitlist and released slots** — The scope varies depending on whether waitlists are centralised, specialty-specific, or not formally managed.
2. **GP referral submission and status tracking** — Some hospitals only accept referrals through accredited GP software rather than a portal.
3. **Electronic medical record exchange** — The integration boundary depends on which system is authoritative for patient identity, appointments, and clinical scheduling.
