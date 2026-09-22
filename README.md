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

## Abstractions and boundaries

| Component abstraction | Classification | Why | Operations exposed | Details hidden |
|---|---|---|---|---|
| `AppointmentSchedulingService` | Control | Coordinates booking and calendar changes without exposing how rules are applied. | `findAvailableSlots()`, `createAppointment()`, `changeAppointment()`, `cancelAppointment()` | Calendar queries, conflict detection, scheduling rules, and slot locking. |
| `ReferralWorkflowService` | Control | Controls referral progression through validation, triage, routing, and status changes. | `submitReferral()`, `validateReferral()`, `triageReferral()`, `routeReferral()`, `getReferralStatus()` | Clinical routing rules, queues, document checks, and lifecycle transitions. |
| `SelfServicePortal` | Interface | Provides patient and GP access without exposing internal hospital services. | `bookAppointment()`, `cancelAppointment()`, `rescheduleAppointment()`, `submitGPReferral()`, `viewStatus()` | User-interface behaviour, sessions, validation, and downstream service calls. |
| `NotificationService` | Control | Controls when and how a message is generated and delivered. | `dispatchNotification()`, `getDeliveryStatus()` | Templates, delivery channels, retry policies, and provider APIs. |
| `PlatformAdministrationService` | Conceptual | Represents operational rules and governance as manageable configuration. | `configureClinic()`, `setBookingRule()`, `assignUserRole()`, `recordConsent()`, `generateReport()` | Storage, audit-log structure, permission evaluation, and report queries. |
| `HealthcareIntegrationGateway` | Interface | Provides a stable boundary to external healthcare systems. | `syncPatient()`, `publishAppointmentUpdate()`, `receiveReferral()`, `sendReferralStatus()` | Vendor APIs, authentication, format conversion, retries, and protocols. |

## Design review and refactoring

The design has two connected problems: some components own more than one responsibility, and components can become coupled to each other's internal policies or data. A component should ask another component to perform a named business action, rather than deciding how that component performs the action.

| Problem | Why it matters | Refactoring action |
|---|---|---|
| `PlatformAdministrationService` combines configuration, access control, consent, auditing, and reporting. | It is a broad service with several independent reasons to change. | Split it into `SchedulingConfigurationService`, `AccessControlService`, `ConsentService`, `AuditTrail`, and `ReportingService`. |
| `ReferralWorkflowService` owns both referral lifecycle management and clinical triage/routing. | Intake records and clinical decisions have different rules, ownership, and release cycles. | Keep `ReferralService` for intake, documents, and status; introduce `ReferralTriageService` for prioritisation and routing. |
| `AppointmentSchedulingService` also manages waitlists. | Waitlist allocation has independent eligibility and prioritisation policy. | Introduce `WaitlistService`, which asks scheduling for eligible released slots without changing calendars directly. |
| `SelfServicePortal` groups patient and GP channels. | The two audiences have different permissions and journeys. | Provide reusable `AppointmentAPI` and `ReferralAPI`, with separate `PatientPortalAdapter` and `GPPortalAdapter`. |
| Notification methods are tied to individual business events. | Repeated send operations reduce reuse and expose template choices. | Use `dispatchNotification(notificationRequest)` and handle domain events such as `AppointmentBooked` internally. |
| `HealthcareIntegrationGateway` mixes different external business domains. | Clinical-record, GP-referral, and messaging integrations evolve independently. | Define `ClinicalRecordPort`, `GPReferralPort`, and `MessagingProviderPort`, each with vendor-specific adapters behind it. |

### Refined interaction model

```text
ReferralService
  -> publishes ReferralAccepted
  -> ReferralTriageService produces ReferralTriaged
  -> AppointmentSchedulingService creates an appointment
  -> publishes AppointmentBooked
  -> NotificationService dispatches the appropriate message
```

Each component exposes requests and outcomes, while hiding its rules, queues, data structures, vendor protocols, and implementation workflow. This applies the single-responsibility principle and reduces coupling between components.
