# Sequence Diagram – Booking Flow

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant API as Booking API
    participant Cache as Redis (Lock Manager)
    participant DB as SQL Database
    participant Queue as Message Broker (RabbitMQ)
    participant Worker as Notification Worker

    Client->>API: Request to book Slot (ID: 101)
    
    Note over API,Cache: Step 1: Concurrency Control
    API->>Cache: SETNX lock:slot:101 (TTL 5m)
    
    alt Slot is already locked by another user
        Cache-->>API: Lock Failed
        API-->>Client: 409 Conflict: Slot is temporarily held
    else Lock Acquired
        Cache-->>API: Lock Success
        
        Note over API,DB: Step 2: Database Transaction
        API->>DB: Start Transaction
        API->>DB: Create Appointment (State: LOCKED)
        API->>DB: Commit Transaction
        
        API-->>Client: 201 Created (Proceed to Payment/Confirm)
        
        Client->>API: Confirm Booking
        API->>DB: Update Appointment (State: CONFIRMED)
        API->>Cache: DEL lock:slot:101
        
        Note over API,Queue: Step 3: Event-Driven Notifications
        API->>Queue: Publish Appointment_Confirmed Event
        API-->>Client: Booking Successful!

        Queue->>Worker: Consume Event
        Worker->>Worker: Generate Email/SMS
        Worker-->>Client: Receive Confirmation Notification
    end
