# Sprint 1 Story 1 Diagrams

## User Story

[[Mr. Tam the Elderly Patient]] => ask chatbot about symptoms via voice or text => get basic guidance without phone call

## Use Case Diagram

```mermaid
graph LR
    %% Actors
    Tam((Mr. Tam <br/> Elderly Patient))
    AI((AI Chatbot <br/> System))

    subgraph "Automated Customer Support Sub-system"
        UC1([Consult Chatbot for Symptoms])
        UC2([Provide Voice Input])
        UC3([Provide Text Input])
        UC4([Receive Health Guidance])
        UC5([Analyze Patient Health Profile])
        
        %% Relationships
        UC2 -.->|extend| UC1
        UC3 -.->|extend| UC1
        UC1 --- UC4
        UC4 -.->|include| UC5
    end

    %% Actor Connections
    Tam --- UC1
    AI --- UC4
    AI --- UC5

    %% Styling
    style Tam fill:#f9f,stroke:#333,stroke-width:2px
    style AI fill:#bbf,stroke:#333,stroke-width:2px
```

## Class Diagram

```mermaid
classDiagram
    class Patient {
        +int patientID
        +String name
        +int age
        +List chronicConditions
        +requestConsultation()
    }

    class ChatbotInterface {
        +listenVoice()
        +receiveText()
        +displayGuidance(Guidance)
        +speakGuidance(Guidance)
    }

    class Message {
        +int messageID
        +DateTime timestamp
        +String content
        +InputType type
    }

    class InputType {
        <<enumeration>>
        VOICE
        TEXT
    }

    class AIEngine {
        +Guidance analyzeSymptoms(Message)
        -processNaturalLanguage(String)
    }

    class Guidance {
        +int guidanceID
        +String adviceText
        +String audioFilePath
        +UrgencyLevel level
    }

    class UrgencyLevel {
        <<enumeration>>
        REST_AND_MONITOR
        SCHEDULE_APPOINTMENT
        EMERGENCY
    }

    Patient "1" --> "0..*" Message : provides
    ChatbotInterface "1" ..> "1" Message : captures
    ChatbotInterface "1" --> "1" AIEngine : sends data to
    AIEngine "1" ..> "1" Guidance : generates
    Guidance "1" --o "1" ChatbotInterface : returned to
```

## Sequence Diagram

```mermaid
%%{init: { 'sequence': { 'showSequenceNumbers': false, 'mirrorActors': false } }}%%
sequenceDiagram
    actor Tam as Mr. Tam (Elderly Patient)
    participant UI as Frontend Interface (Mobile/Web)
    participant AI as AI Chatbot System (Backend)

    Note over Tam, UI: Simplified UI for low tech literacy [1]

    Tam->>UI: Opens App & Selects "Ask for Help"
    UI->>Tam: Prompts "How can I help you today?" (Audio & Text)
    
    alt Voice Interaction
        Tam->>UI: Speaks symptoms (e.g., "My chest feels tight") [2]
        UI->>UI: Processes Audio to Text
    else Text Interaction
        Tam->>UI: Types symptoms into simple text box [2]
    end

    UI->>AI: Sends symptom data for analysis [3]
    AI-->>UI: Returns basic guidance (e.g., "Rest and monitor") [2]
    
    UI->>Tam: Displays guidance in large font [1]
    UI->>Tam: Reads guidance aloud via Text-to-Speech [1]
    
    Note over Tam: Receives consultation without a phone call [2]
```