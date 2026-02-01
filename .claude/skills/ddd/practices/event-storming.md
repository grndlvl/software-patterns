# Event Storming: Collaborative Domain Discovery

## Definition

Event Storming is a collaborative, workshop-based technique for rapidly discovering and understanding a domain model. Participants use colored sticky notes on a timeline to model domain events, commands, aggregates, policies, and external systems. It bridges the gap between business stakeholders and technical teams through a visual, tactile process that reveals implicit business rules and bounded context boundaries.

---

## Sticky Note Colors and Meanings

| Color | Represents | Example |
|-------|-----------|---------|
| **Orange** | Domain Events (past tense) | OrderPlaced, PaymentProcessed, InventoryReserved |
| **Blue** | Commands (imperative) | PlaceOrder, ProcessPayment, ReserveInventory |
| **Yellow** | Aggregates (nouns) | Order, Payment, Inventory |
| **Purple** | Policies (if-then rules) | "If payment fails 3 times, mark order as fraud risk" |
| **Pink** | External Systems | PaymentGateway, EmailService, ShippingProvider |

---

## Workshop Flow

### Phase 1: Chaotic Exploration
- Participants shout out events as they occur to them
- No order or structure—just rapid capture
- Everyone adds orange stickies simultaneously
- Creates initial event vocabulary

**Goal:** Surface the full breadth of domain events without worrying about sequence.

### Phase 2: Timeline Construction
- Place events in chronological order on a long surface (whiteboard, wall, table)
- Build a narrative of the process from start to finish
- Remove duplicates; merge similar events
- Identify gaps and missing events

**Goal:** Establish the logical sequence and flow of the domain.

### Phase 3: Commands Discovery
- For each event, identify the command that caused it
- Add blue stickies upstream of orange events
- Question: "What action triggered this event?"

**Goal:** Connect user/system actions to business outcomes.

### Phase 4: Aggregates and Policies
- Identify aggregates (yellow) that participate in each command/event
- Add policies (purple) that govern behavior: "If X happens, then Y must follow"
- Mark external systems (pink) that integrate with the domain

**Goal:** Reveal entity boundaries, consistency rules, and system dependencies.

---

## Pseudocode: Discovered Model Example

```pseudocode
// Domain Events (Orange stickies)
Event OrderPlaced {
  orderId: string
  customerId: string
  items: Item[]
  totalAmount: decimal
  timestamp: datetime
}

Event PaymentProcessed {
  orderId: string
  paymentId: string
  amount: decimal
  status: "success" | "failed"
  timestamp: datetime
}

Event InventoryReserved {
  orderId: string
  reservationId: string
  items: Item[]
  timestamp: datetime
}

// Commands (Blue stickies)
Command PlaceOrder {
  customerId: string
  items: Item[]
  shippingAddress: Address
  
  Execute() -> OrderPlaced {
    // Validate customer
    // Validate items in inventory
    // Create aggregate
    return OrderPlaced(...)
  }
}

Command ProcessPayment {
  orderId: string
  amount: decimal
  paymentMethod: PaymentMethod
  
  Execute() -> PaymentProcessed {
    // Call external payment gateway
    // Emit event based on result
    return PaymentProcessed(...)
  }
}

// Aggregates (Yellow stickies)
Aggregate Order {
  Id: OrderId
  CustomerId: CustomerId
  Items: Item[]
  Status: "pending" | "confirmed" | "shipped" | "cancelled"
  CreatedAt: datetime
  
  PlaceOrder(customerId, items) {
    this.Status = "pending"
    Emit(OrderPlaced(...))
  }
  
  ConfirmPayment(paymentId) {
    if this.Status == "pending" {
      this.Status = "confirmed"
      Emit(OrderConfirmed(...))
    }
  }
  
  CancelOrder(reason) {
    if this.Status != "shipped" {
      this.Status = "cancelled"
      Emit(OrderCancelled(...))
    }
  }
}

Aggregate Payment {
  Id: PaymentId
  OrderId: OrderId
  Amount: decimal
  Status: "pending" | "success" | "failed"
  Attempts: int
  
  ProcessPayment(method) {
    Call ExternalPaymentGateway.Charge(this.Amount, method)
    if result.success {
      this.Status = "success"
      Emit(PaymentProcessed(...))
    } else {
      this.Attempts++
      if this.Attempts >= 3 {
        Emit(FraudRiskDetected(...))
      }
      Emit(PaymentFailed(...))
    }
  }
}

Aggregate Inventory {
  Id: InventoryId
  SKU: string
  AvailableQty: int
  ReservedQty: int
  
  Reserve(orderId, quantity) {
    if this.AvailableQty >= quantity {
      this.ReservedQty += quantity
      this.AvailableQty -= quantity
      Emit(InventoryReserved(...))
      return Success
    }
    return Failure("Insufficient stock")
  }
  
  Release(orderId) {
    // Restore to available when order cancelled
    this.ReservedQty -= quantity
    this.AvailableQty += quantity
    Emit(InventoryReleased(...))
  }
}

// Policies (Purple stickies)
Policy PaymentRetryPolicy {
  When(PaymentFailed event) {
    if event.Payment.Attempts < 3 {
      Schedule(RetryPayment, delay: 5 minutes)
    } else {
      NotifyFraudDetection(event.OrderId)
      FlagOrderForReview(event.OrderId)
    }
  }
}

Policy InventoryReservationPolicy {
  When(OrderPlaced event) {
    for each item in event.Items {
      Command(ReserveInventory, 
        itemId: item.Id, 
        quantity: item.Qty)
    }
  }
}

// External Systems (Pink stickies)
ExternalSystem PaymentGateway {
  Charge(amount, method) -> (success: bool, transactionId: string)
}

ExternalSystem EmailService {
  SendOrderConfirmation(orderId)
  SendPaymentFailureNotice(orderId)
}

ExternalSystem ShippingProvider {
  CreateShipment(orderId, items, address) -> ShipmentId
}
