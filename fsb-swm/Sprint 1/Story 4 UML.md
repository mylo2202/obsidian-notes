# Story 4 UML

## User Story

[[Lan the Hospital Operations Manager]] => view statistics on the most frequently asked chatbot questions => improve service processes

## Use Case Diagram

```mermaid
graph LR
    %% Actors
    Lan((Lan <br/> Operations Manager))
    Analytics((Analytics <br/> Engine))

    subgraph "Operational Analytics Sub-system"
        UC1([View Chatbot Question Statistics])
        UC2([Analyze Frequent Health Inquiries])
        UC3([Identify Service Process Gaps])
        UC4([Generate Operational Reports])
        UC5([Access Historical Chatbot Data])
        
        %% Relationships
        UC1 -.->|include| UC2
        UC1 -.->|extend| UC3
        UC1 --- UC4
        UC2 -.->|include| UC5
    end

    %% Actor Connections
    Lan --- UC1
    Lan --- UC4
    Analytics --- UC2
    Analytics --- UC5

    %% Styling
    style Lan fill:#f9f,stroke:#333,stroke-width:2px
    style Analytics fill:#bbf,stroke:#333,stroke-width:2px
```

## Class Diagram

```mermaid
classDiagram
    class HospitalOperationsManager {
        +int managerID
        +String name
        +viewQuestionStatistics()
        +analyzeTrends()
        +optimizeResources(ServiceProcess)
    }

    class AnalyticsEngine {
        +List~QuestionStatistic~ aggregateInquiryData(TimePeriod)
        +TrendReport identifyServiceGaps()
        -filterByKeywords(String)
    }

    class ChatbotInteraction {
        +int interactionID
        +String patientQuestion
        +DateTime timestamp
        +String topicCategory
        +int sentimentScore
    }

    class QuestionStatistic {
        +String categoryName
        +int hitCount
        +float percentageOfTotal
        +isEmergingTrend: boolean
    }

    class ServiceProcess {
        +int processID
        +String processName
        +float currentEfficiency
        +List allocatedStaff
        +updateWorkflow(String)
    }

    class TimePeriod {
        <<enumeration>>
        DAILY
        WEEKLY
        MONTHLY
        QUARTERLY
    }

    HospitalOperationsManager "1" --> "1" AnalyticsEngine : queries
    AnalyticsEngine "1" ..> "0..*" ChatbotInteraction : analyzes
    AnalyticsEngine "1" --> "0..*" QuestionStatistic : produces
    QuestionStatistic "0..*" --o "1" HospitalOperationsManager : viewed by
    HospitalOperationsManager "1" --> "0..*" ServiceProcess : manages
```

## Sequence Diagram

```mermaid
%%{init: { 'sequence': { 'showSequenceNumbers': false, 'mirrorActors': false } }}%%
sequenceDiagram
    actor Lan as Lan (Operations Manager)
    participant UI as Operations Dashboard (UI)
    participant Engine as Analytics Engine
    participant DB as Chatbot Interaction DB

    Note over Lan, DB: Analyzing chatbot trends to improve service processes

    Lan->>UI: Selects time period (e.g., Weekly)
    Lan->>UI: Requests "Frequently Asked Questions" report
    UI->>Engine: getAggregatedStatistics(timePeriod)
    
    activate Engine
    Engine->>DB: fetchInteractions(timePeriod)
    DB-->>Engine: return rawInteractionData (List)
    
    Note right of Engine: Engine categorizes questions and <br/>calculates frequency (hitCount)
    
    Engine->>Engine: aggregateByTopicCategory()
    Engine-->>UI: return list of QuestionStatistics
    deactivate Engine
    
    UI->>Lan: Displays statistical charts and trend reports
    
    Note over Lan: Analyzes data to identify service gaps 
    Note over Lan: Updates staff/resource allocation accordingly
```
