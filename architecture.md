``` mermaid
flowchart TB
    %% Styles
    classDef actor fill:#f9f,stroke:#333,stroke-width:2px;
    classDef frontend fill:#d4e1f5,stroke:#333,stroke-width:2px;
    classDef backend fill:#e1f5fe,stroke:#333,stroke-width:2px;
    classDef storage fill:#fff9c4,stroke:#333,stroke-width:2px;
    classDef external fill:#eee,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef security fill:#ffccbc,stroke:#333,stroke-width:2px;

    %% External Actors
    User_Compliance[Compliance Officer]:::actor
    User_Admin[System Admin]:::actor
    Vendor_Site[Third Party Websites]:::external

    %% Bank Infrastructure Boundary
    subgraph Bank_Intranet [Bank Intranet / Private Cloud]
        direction TB

        %% Authentication
        subgraph IAM_Layer [Identity & Access Management]
            LDAP[LDAP / AD / IAM]:::security
        end

        %% Frontend
        subgraph Presentation_Layer [Presentation Layer]
            Web_UI[Web Dashboard & Review UI]:::frontend
        end

        %% Backend Services (Microservices or Modular Monolith)
        subgraph Application_Layer [Application Layer]
            API_Gateway[API Gateway]:::backend
            
            subgraph Core_Services
                Scheduler[Job Scheduler]:::backend
                Workflow_Engine[Core API & Workflow Engine]:::backend
                Audit_Service[Audit Logger]:::backend
            end

            subgraph Worker_Services
                Crawler[Crawler Service]:::backend
                Diff_Engine[Diff & Analysis Service]:::backend
                Notifier[Notification Service]:::backend
            end
        end

        %% Data Storage
        subgraph Data_Layer [Data Persistence]
            Relational_DB[(Relational DB\nUsers, Vendors, Workflow State)]:::storage
            Object_Store[(Object Storage\nHTML Snapshots)]:::storage
            WORM_Archive[(Immutable Archive\nRegulatory Retention)]:::storage
        end
        
        %% Outbound Proxy (Crucial for Banks)
        Proxy[Secure Forward Proxy]:::security
    end

    %% External Integrations
    subgraph Enterprise_Tools [Enterprise Integrations]
        Mail_Server[SMTP Server]:::external
        IM_Server[WeChat/DingTalk/Teams]:::external
    end

    %% Connections
    User_Compliance -->|HTTPS| Web_UI
    User_Admin -->|HTTPS| Web_UI
    Web_UI -->|Auth Request| LDAP
    Web_UI -->|REST/GraphQL| API_Gateway
    
    API_Gateway --> Workflow_Engine
    
    %% Workflow & Logic
    Workflow_Engine -->|Read/Write| Relational_DB
    Workflow_Engine -->|Log Actions| Audit_Service
    Audit_Service -->|Write Logs| Relational_DB

    %% Scheduling Flow
    Scheduler -->|Trigger| Crawler
    
    %% Crawling Flow
    Crawler -->|Fetch Request| Proxy
    Proxy -->|HTTPS| Vendor_Site
    Crawler -->|Raw HTML| Object_Store
    Crawler -->|Notify Completion| Diff_Engine

    %% Diff Flow
    Diff_Engine -->|Fetch Previous & Current| Object_Store
    Diff_Engine -->|Clean & Compare| Diff_Engine
    Diff_Engine -->|Save Diff Metadata| Relational_DB
    Diff_Engine -->|Send Alert| Notifier
    
    %% Archiving
    Object_Store -.->|Sync/Backup| WORM_Archive

    %% Notification Flow
    Notifier -->|Send Email| Mail_Server
    Notifier -->|Send IM| IM_Server

    %% Review Flow
    User_Compliance -->|View Diff| Web_UI
    Workflow_Engine -->|Update Status: Reviewed| Relational_DB
```
