# Flight Propagation Engine Flowchart

This document contains the flowchart for the Flight Propagation Engine event handler process.

## Main Flowchart

```mermaid
flowchart TD
    A[Start: eventHandler] --> B{PropagationEngineEvent != null?}
    B -->|No| Z[End: No processing]
    B -->|Yes| C[startPropagationEvent]
    
    C --> D{FlightKey != null?}
    D -->|No| E[Throw IllegalArgumentException]
    D -->|Yes| F[Acquire synchronized lock]
    
    F --> G[Check crew exclusion flag]
    G --> H[populatePropagationPayload]
    H --> I[calculatePropagatedFlightTimes]
    
    I --> J{Event Type?}
    J -->|PE_CANCEL| K[Substitute with schedule times]
    J -->|PE_OUT| L[Process OUT event]
    J -->|PE_IN| M[Process IN event]
    J -->|PE_OFF| N[Process OFF event]
    J -->|PE_CONTINUED| O[Process CONTINUED event]
    
    K --> P[Calculate projected arrivals]
    L --> P
    M --> P
    N --> P
    O --> P
    
    P --> Q[Handle problem types]
    Q --> R[Publish flight times to cache]
    R --> S[checkPEContinuedforNextFlight]
    
    S --> T{Next leg propagation?}
    T -->|Yes| U[Create PE_CONTINUED events]
    T -->|No| V[Save propagation debug data]
    
    U --> W[Publish continuation events]
    W --> V
    V --> X[Release lock]
    X --> Y[Return PropagationEngineCalculationValues]
    
    %% Error handling
    F -->|Exception| AA[Handle lock exception]
    I -->|Exception| BB[Set status to DEBUG]
    BB --> CC[Handle LKA exception]
    CC --> X
    
    %% Key sub-processes
    H --> H1[Get crew details]
    H1 --> H2[Get equipment details]
    H2 --> H3[Get flight times]
    H3 --> H4[Build event payload]
    
    P --> P1[Calculate taxi times]
    P1 --> P2[Calculate air time]
    P2 --> P3[Update projected times]
    
    Q --> Q1[Check departure problem types]
    Q1 --> Q2[Check arrival problem types]
    
    R --> R1[Publish PTA events]
    R1 --> R2[Publish PTD events]
    R2 --> R3[Publish control events]
    
    style A fill:#e1f5fe
    style Y fill:#c8e6c9
    style E fill:#ffcdd2
    style AA fill:#ffcdd2
    style BB fill:#ffcdd2
```

## Flow Description

This flowchart illustrates the complete process flow for the Flight Propagation Engine event handler, including:

- **Event validation**: Checks for valid PropagationEngineEvent and FlightKey
- **Lock management**: Acquires and releases synchronized locks for thread safety
- **Event processing**: Handles different event types (PE_CANCEL, PE_OUT, PE_IN, PE_OFF, PE_CONTINUED)
- **Time calculations**: Calculates projected flight times and arrivals
- **Problem handling**: Manages various problem types and exceptions
- **Cache publishing**: Publishes flight times and events to cache
- **Continuation logic**: Handles next leg propagation when applicable

The flowchart uses color coding:
- **Blue**: Start node
- **Green**: Successful end node
- **Red**: Error/exception nodes
