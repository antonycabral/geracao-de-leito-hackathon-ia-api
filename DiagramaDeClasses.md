classDiagram
    class Location {
        +String id
        +String status
        +String alias
        +Object partOf
        +checkStatus()
    }
    class Patient {
        +String id
        +String name
        +String riskColor
    }
    class Encounter {
        +String id
        +DateTime start
        +DateTime estimatedEnd
        +String status
        +updateEDD(newDate, reason)
    }
    class Task {
        +String id
        +String type
        +String priority
        +String status
    }

    Location "1" -- "0..1" Encounter : aloja
    Patient "1" -- "0..*" Encounter : possui
    Encounter "1" -- "0..*" Task : gera
    Encounter "1" -- "1" Patient : pertence


diagrama baseado em FHIR