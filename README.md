# Event Ticketing & Booking System

> **Course:** EF234402 – Konstruksi Perangkat Lunak / Software Construction
> **Institution:** Institut Teknologi Sepuluh Nopember
> **Architecture:** Clean Architecture + Domain-Driven Design (DDD)
> **Language:** TypeScript
> **Framework:** NestJS
> **Database:** PostgreSQL

---

## Table of Contents

1. [Clean Architecture Folder Structure](#clean-architecture-folder-structure)
2. [Business Rules](#business-rules)
3. [Initial Domain Model Draft](#initial-domain-model-draft)
4. [Ubiquitous Language Glossary](#ubiquitous-language-glossary)

---

## Clean Architecture Folder Structure

```
event-ticketing/
├── src/
│   ├── domain/
│   │   ├── aggregates/
│   │   │   ├── event/
│   │   │   │   ├── event.aggregate.ts
│   │   │   │   ├── ticket-category.entity.ts
│   │   │   │   └── event-status.enum.ts
│   │   │   ├── booking/
│   │   │   │   ├── booking.aggregate.ts
│   │   │   │   ├── ticket.entity.ts
│   │   │   │   ├── booking-status.enum.ts
│   │   │   │   └── ticket-status.enum.ts
│   │   │   └── refund/
│   │   │       ├── refund.aggregate.ts
│   │   │       └── refund-status.enum.ts
│   │   ├── value-objects/
│   │   │   ├── money.vo.ts
│   │   │   ├── event-schedule.vo.ts
│   │   │   ├── sales-period.vo.ts
│   │   │   ├── ticket-code.vo.ts
│   │   │   └── payment-deadline.vo.ts
│   │   ├── events/
│   │   │   ├── event-created.event.ts
│   │   │   ├── event-published.event.ts
│   │   │   ├── event-cancelled.event.ts
│   │   │   ├── ticket-category-created.event.ts
│   │   │   ├── ticket-category-disabled.event.ts
│   │   │   ├── ticket-reserved.event.ts
│   │   │   ├── booking-paid.event.ts
│   │   │   ├── booking-expired.event.ts
│   │   │   ├── ticket-checked-in.event.ts
│   │   │   ├── refund-requested.event.ts
│   │   │   ├── refund-approved.event.ts
│   │   │   ├── refund-rejected.event.ts
│   │   │   └── refund-paid-out.event.ts
│   │   ├── repositories/
│   │   │   ├── event.repository.interface.ts
│   │   │   ├── booking.repository.interface.ts
│   │   │   └── refund.repository.interface.ts
│   │   ├── services/
│   │   │   ├── ticket-quota.domain-service.ts
│   │   │   └── refund-eligibility.domain-service.ts
│   │   └── exceptions/
│   │       └── domain.exception.ts
│   │
│   ├── application/
│   │   ├── event/
│   │   │   ├── commands/
│   │   │   │   ├── create-event/
│   │   │   │   │   ├── create-event.command.ts
│   │   │   │   │   └── create-event.handler.ts
│   │   │   │   ├── publish-event/
│   │   │   │   │   ├── publish-event.command.ts
│   │   │   │   │   └── publish-event.handler.ts
│   │   │   │   └── cancel-event/
│   │   │   │       ├── cancel-event.command.ts
│   │   │   │       └── cancel-event.handler.ts
│   │   │   └── queries/
│   │   │       ├── get-available-events/
│   │   │       └── get-event-details/
│   │   ├── booking/
│   │   │   ├── commands/
│   │   │   │   ├── create-booking/
│   │   │   │   ├── pay-booking/
│   │   │   │   └── expire-booking/
│   │   │   └── queries/
│   │   │       └── get-purchased-tickets/
│   │   ├── refund/
│   │   │   ├── commands/
│   │   │   │   ├── request-refund/
│   │   │   │   ├── approve-refund/
│   │   │   │   ├── reject-refund/
│   │   │   │   └── mark-refund-as-paid-out/
│   │   │   └── queries/
│   │   ├── dtos/
│   │   │   ├── event.dto.ts
│   │   │   ├── booking.dto.ts
│   │   │   └── refund.dto.ts
│   │   └── service-interfaces/
│   │       ├── payment-gateway.interface.ts
│   │       ├── refund-payment.interface.ts
│   │       └── notification.interface.ts
│   │
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── typeorm/
│   │   │   │   ├── entities/
│   │   │   │   │   ├── event.orm-entity.ts
│   │   │   │   │   ├── booking.orm-entity.ts
│   │   │   │   │   └── refund.orm-entity.ts
│   │   │   │   └── migrations/
│   │   │   └── repositories/
│   │   │       ├── event.repository.ts
│   │   │       ├── booking.repository.ts
│   │   │       └── refund.repository.ts
│   │   └── external-services/
│   │       ├── payment-gateway.service.ts
│   │       ├── refund-payment.service.ts
│   │       └── notification.service.ts
│   │
│   ├── presentation/
│   │   └── controllers/
│   │       ├── events.controller.ts
│   │       ├── bookings.controller.ts
│   │       ├── tickets.controller.ts
│   │       └── refunds.controller.ts
│   │
│   └── app.module.ts
│
├── test/
│   └── domain/
│       ├── event.spec.ts
│       ├── ticket-category.spec.ts
│       ├── booking.spec.ts
│       ├── ticket.spec.ts
│       └── refund.spec.ts
│
├── nest-cli.json
├── tsconfig.json
├── package.json
└── .env.example
```

---

## Business Rules

Business rules are derived from the user stories and acceptance criteria and enforced strictly in the **Domain Layer**.

### Event Management

| # | Business Rule |
|---|---|
| BR-01 | An event cannot be created if the end date is earlier than the start date. |
| BR-02 | An event cannot be created if the maximum capacity is less than or equal to zero. |
| BR-03 | A newly created event must have the status `Draft`. |
| BR-04 | An event can only be published if it has at least one active ticket category. |
| BR-05 | An event can only be published if the total ticket quota does not exceed the maximum event capacity. |
| BR-06 | An event with status `Cancelled` cannot be published. |
| BR-07 | An event with status `Published` can be cancelled. |
| BR-08 | An event with status `Completed` cannot be cancelled. |
| BR-09 | When an event is cancelled, all paid bookings must be marked as requiring a refund. |

### Ticket Category Management

| # | Business Rule |
|---|---|
| BR-10 | A ticket price cannot be less than zero. |
| BR-11 | A ticket quota must be greater than zero. |
| BR-12 | The ticket sales period must end before or at the event start date. |
| BR-13 | The total quota of all ticket categories must not exceed the maximum event capacity. |
| BR-14 | A ticket category can only be disabled if the event has not been completed. |
| BR-15 | Customers cannot purchase tickets from an inactive ticket category. |

### Booking

| # | Business Rule |
|---|---|
| BR-16 | A booking can only be created for an event with status `Published`. |
| BR-17 | A booking can only be created for an active ticket category within its sales period. |
| BR-18 | The ticket quantity must be greater than zero. |
| BR-19 | The ticket quantity must not exceed the remaining ticket quota. |
| BR-20 | A customer cannot have more than one active booking for the same event. |
| BR-21 | A newly created booking must have the status `PendingPayment`. |
| BR-22 | A booking must have a payment deadline (15 minutes after creation). |

### Booking Payment

| # | Business Rule |
|---|---|
| BR-23 | A booking can only be paid if its status is `PendingPayment`. |
| BR-24 | A booking cannot be paid if the payment deadline has passed. |
| BR-25 | The payment amount must equal the total booking price. |
| BR-26 | After payment, the system issues tickets with unique ticket codes. |

### Booking Expiry

| # | Business Rule |
|---|---|
| BR-27 | A booking with status `PendingPayment` changes to `Expired` after its payment deadline has passed. |
| BR-28 | A booking with status `Paid` cannot be marked as expired. |
| BR-29 | When a booking expires, the previously reserved ticket quota is released. |

### Ticket & Check-in

| # | Business Rule |
|---|---|
| BR-30 | Check-in can only be performed for tickets matching the correct event. |
| BR-31 | Only tickets with status `Active` can be checked in. |
| BR-32 | A ticket that has already been checked in cannot be used again. |
| BR-33 | Check-in can only be performed on the event day or within the allowed check-in time window. |

### Refund Management

| # | Business Rule |
|---|---|
| BR-34 | A refund can only be requested for a booking with status `Paid`. |
| BR-35 | A refund cannot be requested if any ticket from the booking has already been checked in. |
| BR-36 | A refund can only be requested before the refund deadline, unless the event is cancelled. |
| BR-37 | A refund can only be approved or rejected if its status is `Requested`. |
| BR-38 | A rejection reason must be provided when rejecting a refund. |
| BR-39 | When a refund is approved, related tickets change to `Cancelled` and the booking changes to `Refunded`. |
| BR-40 | A refund can only be marked as `PaidOut` if its status is `Approved`. |
| BR-41 | A paid-out refund cannot be approved, rejected, or cancelled again. |

### Pricing

| # | Business Rule |
|---|---|
| BR-42 | The total price is calculated from ticket unit price multiplied by ticket quantity, plus any service fee. |
| BR-43 | The total price cannot be negative. |
| BR-44 | The total price is represented using the `Money` value object. |

---

## Initial Domain Model Draft

### Aggregates and Entities

```
Event (Aggregate Root)
├── id: EventId
├── organizerId: OrganizerId
├── name: string
├── description: string
├── schedule: EventSchedule             ← Value Object (startDate, endDate)
├── location: string
├── maxCapacity: number
├── status: EventStatus                 ← Enum (Draft, Published, Cancelled, Completed)
└── ticketCategories: TicketCategory[]

    TicketCategory (Entity)
    ├── id: TicketCategoryId
    ├── name: string
    ├── price: Money                    ← Value Object
    ├── quota: number
    ├── remainingQuota: number
    ├── salesPeriod: SalesPeriod        ← Value Object (startDate, endDate)
    └── isActive: boolean

Booking (Aggregate Root)
├── id: BookingId
├── customerId: CustomerId
├── eventId: EventId
├── ticketCategoryId: TicketCategoryId
├── quantity: number
├── totalPrice: Money                   ← Value Object
├── status: BookingStatus               ← Enum (PendingPayment, Paid, Expired, Refunded)
├── paymentDeadline: PaymentDeadline    ← Value Object
└── tickets: Ticket[]

    Ticket (Entity)
    ├── id: TicketId
    ├── bookingId: BookingId
    ├── ticketCode: TicketCode          ← Value Object
    └── status: TicketStatus            ← Enum (Active, CheckedIn, Cancelled)

Refund (Aggregate Root)
├── id: RefundId
├── bookingId: BookingId
├── requestedAt: Date
├── status: RefundStatus                ← Enum (Requested, Approved, Rejected, PaidOut)
├── rejectionReason?: string
└── paymentReference?: string
```

### Value Objects

| Value Object | Attributes | Validation Rules |
|---|---|---|
| `Money` | amount (number), currency (string) | amount >= 0 |
| `EventSchedule` | startDate (Date), endDate (Date) | endDate >= startDate |
| `SalesPeriod` | startDate (Date), endDate (Date) | endDate <= event startDate |
| `TicketCode` | code (string) | unique, non-empty |
| `PaymentDeadline` | deadline (Date) | deadline > booking createdAt |

### Repository Interfaces (declared in Domain Layer)

- `IEventRepository` — `findById`, `findPublished`, `save`, `update`
- `IBookingRepository` — `findById`, `findByCustomerAndEvent`, `save`, `update`
- `IRefundRepository` — `findById`, `findByBookingId`, `save`, `update`

---

## Ubiquitous Language Glossary

| Term | Meaning |
|---|---|
| **Event** | An activity organized by an Event Organizer and attended by customers. |
| **Event Organizer** | A user who creates and manages events. |
| **Customer** | A user who books and purchases tickets. |
| **Gate Officer** | A user who validates tickets during event check-in. |
| **System Admin** | A user who manages refund payouts and monitors operations. |
| **Ticket Category** | A type of ticket offered for an event, such as Regular, VIP, or Early Bird. |
| **Quota** | The maximum number of tickets available in a ticket category. |
| **Remaining Quota** | The number of tickets still available for purchase in a ticket category. |
| **Booking** | A temporary reservation created before payment is completed. |
| **PendingPayment** | A booking status indicating that payment has not yet been completed. |
| **Paid** | A booking status indicating that payment has been successfully completed. |
| **Expired** | A booking status indicating that the payment deadline has passed without payment. |
| **Refunded** | A booking status indicating that a refund has been approved for the booking. |
| **Ticket** | Proof of attendance generated after a booking is paid. |
| **Ticket Code** | A unique code used to identify and validate a ticket during check-in. |
| **Check-in** | The process of validating a ticket when a participant enters the event venue. |
| **Refund** | The process of returning money to a customer. |
| **Money** | A value object representing a monetary amount and its currency. |
| **Sales Period** | The time window during which a ticket category can be purchased. |
| **Payment Deadline** | The deadline by which a booking must be paid before it expires. |
| **Draft** | An event status indicating the event has been created but not yet published. |
| **Published** | An event status indicating the event is visible and open for ticket purchases. |
| **Cancelled** | An event status indicating the event has been cancelled. |
| **Completed** | An event status indicating the event has concluded. |
| **Active** | A ticket status indicating the ticket is valid and has not yet been used. |
| **CheckedIn** | A ticket status indicating the ticket holder has successfully entered the event. |
| **Requested** | A refund status indicating a refund request has been submitted and is awaiting review. |
| **Approved** | A refund status indicating the refund request has been accepted. |
| **Rejected** | A refund status indicating the refund request has been denied. |
| **PaidOut** | A refund status indicating the refund amount has been disbursed to the customer. |
