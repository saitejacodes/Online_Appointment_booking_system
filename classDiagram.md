# Class Diagram – SmartSlot Architecture

```mermaid
classDiagram
    class User {
        <<Abstract>>
        +UUID id
        +String name
        +String email
        +login()
    }
    class Client {
        +joinWaitlist()
        +cancelAppointment()
    }
    class Provider {
        +defineWorkingHours()
        +markComplete()
    }
    User <|-- Client
    User <|-- Provider

    class Appointment {
        -AppointmentState state
        +DateTime startTime
        +setStrategy(CancellationStrategy s)
        +changeState(newState)
    }
    
    class AppointmentState {
        <<Interface>>
        +handle(context)
    }
    class DraftState
    class ConfirmedState
    class CancelledState
    
    Appointment *-- AppointmentState
    AppointmentState <|-- DraftState
    AppointmentState <|-- ConfirmedState
    AppointmentState <|-- CancelledState

    class CancellationStrategy {
        <<Interface>>
        +calculatePenalty(amount) float
    }
    class StrictStrategy
    class FlexibleStrategy

    Appointment o-- CancellationStrategy
    CancellationStrategy <|-- StrictStrategy
    CancellationStrategy <|-- FlexibleStrategy

    class WaitlistManager {
        +List~Client~ queue
        +onAppointmentCancelled()
    }

    Client "1" -- "*" Appointment : "books"
    Provider "1" -- "*" Appointment : "hosts"
    WaitlistManager ..> Appointment : "observes"
