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

### Event

1. An event cannot be created if the end date is earlier than the start date.
2. An event cannot be created if the maximum capacity is less than or equal to zero.
3. A newly created event must have the status `Draft`.
4. An event can only be published if it has at least one active ticket category.
5. An event can only be published if the total ticket quota does not exceed the maximum event capacity.
6. An event with status `Cancelled` cannot be published.
7. An event with status `Completed` cannot be cancelled.
8. When an event is cancelled, all paid bookings must be marked as requiring a refund.
9. A ticket price cannot be less than zero.
10. A ticket quota must be greater than zero.
11. The ticket sales period must end before or at the event start date.
12. The total quota of all ticket categories must not exceed the maximum event capacity.

### Booking

1. A booking can only be created for an event with status `Published`.
2. A booking can only be created for an active ticket category within its sales period.
3. The ticket quantity must be greater than zero and must not exceed the remaining quota.
4. A customer cannot have more than one active booking for the same event.
5. A newly created booking must have the status `PendingPayment` with a payment deadline of 15 minutes after creation.
6. A booking can only be paid if its status is `PendingPayment` and the payment deadline has not passed.
7. The payment amount must equal the total booking price.
8. A booking with status `PendingPayment` changes to `Expired` after its payment deadline has passed.
9. A booking with status `Paid` cannot be marked as expired.
10. When a booking expires, the previously reserved ticket quota is released.
11. After payment, the system issues tickets with unique ticket codes.
12. The total price is calculated from ticket unit price multiplied by ticket quantity and is represented using the `Money` value object.

### Refund

1. A refund can only be requested for a booking with status `Paid`.
2. A refund cannot be requested if any ticket from the booking has already been checked in.
3. A refund can only be requested before the refund deadline, unless the event is cancelled.
4. A refund can only be approved or rejected if its status is `Requested`.
5. A rejection reason must be provided when rejecting a refund.
6. When a refund is approved, related tickets change to `Cancelled` and the booking changes to `Refunded`.
7. A refund can only be marked as `PaidOut` if its status is `Approved`.
8. A paid-out refund cannot be approved, rejected, or cancelled again.

---

## Initial Domain Model Draft

### Classes Involved

* Aggregates: `Event`, `Booking`, `Refund`
* Entities: `TicketCategory`, `Ticket`
* Value Objects: `Money`, `EventSchedule`, `SalesPeriod`, `TicketCode`, `PaymentDeadline`

### Implementation Details

1. **Money** (Value Object)
   * Stores the monetary amount of a ticket price or booking total.
   * Amount must not be negative.
   * Must use `number` with fixed decimal precision to avoid floating point issues.
   * Example: `new Money(200000)`

2. **EventSchedule** (Value Object)
   * Stores the start and end date of an event.
   * End date must not be earlier than start date.
   * Example: `new EventSchedule(new Date('2025-08-01'), new Date('2025-08-02'))`

3. **SalesPeriod** (Value Object)
   * Stores the ticket sales start and end date for a ticket category.
   * Sales end date must not be later than the event start date.
   * Example: `new SalesPeriod(new Date('2025-07-01'), new Date('2025-07-31'))`

4. **TicketCode** (Value Object)
   * Stores the unique code assigned to a ticket after booking is paid.
   * Must not be empty and must be unique across the system.
   * Example: `new TicketCode('TIX-ABC123')`

5. **PaymentDeadline** (Value Object)
   * Stores the deadline by which a booking must be paid.
   * Deadline must be later than the booking creation time.
   * Example: `new PaymentDeadline(new Date(Date.now() + 15 * 60 * 1000))`

6. **TicketCategory** (Entity)
   * Belongs to an `Event` aggregate.
   * Has a name, `Money` price, quota, remaining quota, `SalesPeriod`, and active status.
   * Example:
     ```typescript
     new TicketCategory('VIP', new Money(500000), 100, new SalesPeriod(...))
     ```

7. **Ticket** (Entity)
   * Generated after a booking is paid.
   * Has a `TicketCode` and a status of `Active`, `CheckedIn`, or `Cancelled`.
   * Example:
     ```typescript
     new Ticket(new TicketCode('TIX-ABC123'), TicketStatus.Active)
     ```

8. **Event** (Aggregate Root)
   * Manages ticket categories and enforces event lifecycle rules.
   * Example:
     ```typescript
     const event = new Event('Java Jazz Festival', schedule, 'Jakarta', 5000);
     event.addTicketCategory(new TicketCategory('Regular', new Money(200000), 4000, salesPeriod));
     event.publish();
     ```

9. **Booking** (Aggregate Root)
   * Created when a customer reserves tickets for an event.
   * Manages payment and ticket issuance.
   * Example:
     ```typescript
     const booking = new Booking(customerId, eventId, ticketCategoryId, 2);
     booking.pay(new Money(400000));
     ```

10. **Refund** (Aggregate Root)
    * Created when a customer requests a refund for a paid booking.
    * Manages refund approval, rejection, and payout.
    * Example:
      ```typescript
      const refund = new Refund(bookingId);
      refund.approve();
      refund.markAsPaidOut('REF-PAY-001');
      ```

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
| **Money** | A value object representing a monetary amount. |
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
