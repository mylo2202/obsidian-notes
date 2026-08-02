# Story 2 UML

## User Story

[[Dr. Hung the Specialist Physician]] => see treatment suggestions based on similar cases => make faster and more informed clinical decisions.

## Use Case Diagram

```mermaid
graph LR
    %% Actors
    Hung((Dr. Hung <br/> Specialist Physician))
    AI_Sys((AI Recommendation <br/> System))

    subgraph "AI-Powered Recommendation Sub-system"
        UC1([Request Treatment Suggestions])
        UC2([Match Current Case with Historical Data])
        UC3([Review AI-Generated Recommendations])
        UC4([Finalize Clinical Decision])
        
        %% Relationships
        UC1 -.->|include| UC2
        UC1 --- UC3
        UC3 -.->|extend| UC4
    end

    %% Actor Connections
    Hung --- UC1
    Hung --- UC3
    Hung --- UC4
    AI_Sys --- UC2
    AI_Sys --- UC3

    %% Styling
    style Hung fill:#f9f,stroke:#333,stroke-width:2px
    style AI_Sys fill:#bbf,stroke:#333,stroke-width:2px
```

## Class Diagram

```mermaid
classDiagram
    class Physician {
        +int physicianID
        +String specialty
        +requestSuggestions(PatientProfile)
        +reviewSuggestion(TreatmentSuggestion)
        +finalizeDecision(TreatmentSuggestion)
    }

    class PatientProfile {
        +int patientID
        +List healthConditions
        +String currentSymptoms
        +MedicalHistory history
    }

    class ClinicalCase {
        +int caseID
        +String diagnosis
        +String treatmentAdministered
        +String outcome
        +float similarityScore
    }

    class AIRecommendationEngine {
        +List~ClinicalCase~ findSimilarCases(PatientProfile)
        +TreatmentSuggestion generateSuggestion(List~ClinicalCase~)
    }

    class TreatmentSuggestion {
        +int suggestionID
        +String recommendedAction
        +float confidenceLevel
        +List~ClinicalCase~ referencedCases
    }

    Physician "1" --> "1" PatientProfile : evaluates
    Physician "1" --> "1" AIRecommendationEngine : queries
    AIRecommendationEngine "1" ..> "0..*" ClinicalCase : analyzes
    AIRecommendationEngine "1" --> "1" TreatmentSuggestion : produces
    TreatmentSuggestion "1" o-- "1..*" ClinicalCase : based on
    Physician "1" --> "0..*" TreatmentSuggestion : reviews

```

## Sequence Diagram

```mermaid
%%{init: { 'sequence': { 'showSequenceNumbers': false, 'mirrorActors': false } }}%%
sequenceDiagram
    actor Hung as Dr. Hung (Specialist Physician)
    participant UI as Clinical Dashboard (UI)
    participant AI as AI Recommendation System
    participant DB as Historical Medical DB

    Note over Hung, DB: Facilitates faster, informed clinical decisions

    Hung->>UI: Selects current Patient Profile
    Hung->>UI: Requests Treatment Suggestions
    UI->>AI: getRecommendations(PatientProfile)
    
    activate AI
    AI->>DB: querySimilarCases(healthConditions, symptoms)
    DB-->>AI: return historicalCaseData (List)
    
    Note right of AI: AI analyzes outcomes from similar historical cases
    
    AI->>AI: calculateConfidenceScores()
    AI-->>UI: return treatmentSuggestions
    deactivate AI
    
    UI->>Hung: Displays suggestions with confidence levels
    
    Note over Hung: Informed by AI data and historical outcomes
    
    Hung->>Hung: Finalizes Clinical Decision

```