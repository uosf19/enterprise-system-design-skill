# Book 14: Enterprise Integration Patterns (EIP)
**Authors:** Gregor Hohpe & Bobby Woolf  
**Category:** Asynchronous Messaging & System Integration  
**Relevance to SaaS & ERP:** Adapters, Message Routing, Idempotent Receivers, Dead Letter Queues.

---

## 1. Executive Summary & Core Thesis
Enterprise software never lives in a vacuum. It must communicate with external third-party services, legacy hardware, and asynchronous micro-components without becoming tightly coupled. Hohpe and Woolf establish the standard pattern vocabulary for building resilient, event-driven integration pipelines using asynchronous messaging, message channels, routers, and translators.

---

## 2. Core Integration Patterns for ERP Architecture
*Source note: the EIP catalog contains 65 messaging patterns. The Outbox Table in the diagram is a derived bridge from Pillar 2, not an EIP pattern; the EIP patterns begin with the message channel and router.*

```text
[ Domain Event Producer ]
           |
     (Outbox Table)
     derived bridge, not EIP
           |
[ Message Channel / Router ]
           |
   +-------+-------+-------+
   |               |       |
[ ETA Adapter ] [ Payment Adapter ] [ WhatsApp Adapter ]
(Translator)    (Translator)        (Translator)
   |               |                  |
[ External ETA ] [ External Gateway ] [ Meta Cloud API ]
```

### A. Message Endpoints & Adapters
* **Message Translator (المترجم):** Converts data between the internal canonical data format of the ERP and the proprietary message formats of external APIs (e.g., translating a generic `SalesOrder` into an official `ETA Receipt v1.2 JSON` schema).
* **Idempotent Receiver (المستقبل ذو الأثر الحيادي):** A message consumer that safely handles duplicate messages without side effects:
  - If a payment webhook from Paymob or Fawry is delivered three times due to network retries, the consumer recognizes the existing transaction reference and acknowledges without re-crediting the account.

### B. Routing & Failure Handling Patterns
* **Dead Letter Channel (طابور الرسائل الميتة - DLQ):**
  - The book distinguishes undeliverable messages from invalid messages. In this skill's adaptation, isolate exhausted delivery failures and malformed or permanent rejections with distinct outcomes. An invalid tax document goes through approved correction or review without retrying unchanged content.
* **Content-Based Router (الموجّه المبني على المحتوى):**
  - Directs incoming orders or kitchen tickets to specific stations (Grill station, Barista station, Dessert prep) based on the metadata and category flags inside the order line items.

---

## 3. Architectural Rules for This Skill
1. **The Idempotent Consumer Invariant:** Every event handler and webhook endpoint must check and store the external message/event ID before executing any business logic.
2. **Failure classification (derived):** Retry transient failures within the delivery policy and retain an auditable outcome when exhausted. Route invalid input and permanent rejections directly to their review path. Reconcile an unknown payment or print result before repeating it.
3. **Canonical Event Decoupling:** Internal domain events must remain completely decoupled from external third-party schemas. Translators/Adapters must be isolated at the system edge.
