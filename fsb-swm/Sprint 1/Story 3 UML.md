# Story 3 UML

## User Story

[[Mai the Busy Office Worker]] => book appointment online with shortest expected wait time => plan work schedule better.

## Use Case Diagram

```mermaid
graph LR
    %% Actors
    Mai((Mai <br/> Office Worker))
    AI_Sys((AI Demand <br/> Forecaster))

    subgraph "Smart Appointment & Service Booking Sub-system"
        UC1([Browse Medical Services])
        UC2([View Real-time Wait Time Estimates])
        UC3([Forecast Patient Demand])
        UC4([Book Online Appointment])
        UC5([Sync with Personal Calendar])
        
        %% Relationships
        UC2 -.->|include| UC3
        UC1 --- UC2
        UC2 --- UC4
        UC4 -.->|extend| UC5
    end

    %% Actor Connections
    Mai --- UC1
    Mai --- UC2
    Mai --- UC4
    Mai --- UC5
    AI_Sys --- UC3

    %% Styling
    style Mai fill:#f9f,stroke:#333,stroke-width:2px
    style AI_Sys fill:#bbf,stroke:#333,stroke-width:2px
```

## Class Diagram

```mermaid
classDiagram
    class Patient {
        +int patientID
        +String name
        +String preExistingCondition
        +boolean techSavvy
        +searchServices()
        +viewWaitTimes()
    }

    class Appointment {
        +int appointmentID
        +DateTime scheduledTime
        +int estimatedDuration
        +String status
        +syncToCalendar()
    }

    class MedicalService {
        +int serviceID
        +String serviceName
        +float baseDuration
        +List requiredResources
    }

    class BookingController {
        +List~WaitTimeEstimate~ getAvailableSlots(serviceID)
        +confirmBooking(patientID, serviceID, time)
    }

    class DemandForecaster {
        <<AI Engine>>
        +float predictWaitTime(serviceID, DateTime)
        -analyzeStaffAvailability()
        -analyzeHistoricalTrends()
    }

    class WaitTimeEstimate {
        +int estimateID
        +int minutes
        +float confidenceScore
        +isShortestOption()
    }

    Patient "1" --> "0..*" Appointment : schedules
    Appointment "0..*" -- "1" MedicalService : for
    BookingController "1" ..> "1" DemandForecaster : requests forecast
    DemandForecaster "1" --> "0..*" WaitTimeEstimate : generates
    WaitTimeEstimate "1" --o "1" Appointment : informs
    BookingController "1" -- "1" Patient : interface for
```

## Sequence Diagram

```mermaid
%%{init: { 'sequence': { 'showSequenceNumbers': false, 'mirrorActors': false } }}%%
sequenceDiagram
    actor Mai as Mai (Office Worker)
    participant App as Mobile App (UI)
    participant BC as Booking Controller
    participant AI as Demand Forecaster (AI)
    participant DB as Resource Database

    Note over Mai, DB: Optimizing schedule with shortest wait time forecasts

    Mai->>App: Opens App & Searches for Medical Service
    App->>BC: requestAvailableSlots(serviceID)
    
    activate BC
    BC->>AI: getWaitTimePredictions(serviceID, timeRange)
    
    activate AI
    AI->>DB: queryStaffAvailability & HistoricalTrends
    DB-->>AI: return ResourceData
    AI->>AI: Analyze data to forecast shortest wait times
    AI-->>BC: return predictedWaitTimeEstimates
    deactivate AI
    
    BC-->>App: return slots with predicted wait times
    deactivate BC

    App->>Mai: Displays available slots (Shortest wait time highlighted)
    
    Note over Mai: Reviews options to plan her work schedule better
    
    Mai->>App: Selects optimal slot & confirms booking
    App->>BC: confirmAppointment(patientID, timeSlot)
    BC->>DB: Reserve resources and update schedule
    BC-->>App: bookingConfirmation
    
    App->>Mai: Shows Confirmation & Syncs with Personal Calendar
    
    Note over Mai: Successfully booked without a phone call
```