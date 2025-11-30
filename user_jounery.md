``` mermaid
flowchart TD
    %% Roles
    classDef admin fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef officer fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef system fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    classDef decision fill:#ffe0b2,stroke:#e65100,stroke-width:2px,shape:diamond;
    classDef endState fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,shape:pill;

    %% Swimlanes via Subgraphs
    subgraph Admin_Actions [Admin / Setup Phase]
        Start((Start)) --> Admin_Login[Log in to Dashboard]:::admin
        Admin_Login --> Add_Vendor[Add Vendor URL]:::admin
        Add_Vendor --> Config_Freq[Configure Frequency]:::admin
        Config_Freq --> Assign_Reviewer[Assign Compliance Officer]:::admin
        Assign_Reviewer --> Activate[Activate Monitoring]:::admin
    end

    subgraph System_Process [System Automation]
        Activate --> Scheduler{Scheduled Time?}:::system
        Scheduler -- No --> Scheduler
        Scheduler -- Yes --> Crawl[Crawl URL via Proxy]:::system
        
        Crawl --> Compare{Change Detected?}:::decision
        Compare -- No --> Log_NoChange[Log: No Change]:::system
        Log_NoChange --> Scheduler
        
        Compare -- Yes --> Gen_Snapshot[Generate Snapshot & Text Diff]:::system
        Gen_Snapshot --> Send_Alert[Send Email/IM Notification]:::system
    end

    subgraph User_Actions [Compliance Officer / Review Phase]
        Send_Alert --> Receive_Notif[Receive Alert Link]:::officer
        Receive_Notif --> Click_Link[Click Link to Dashboard]:::officer
        Click_Link --> Auth[SSO Login]:::officer
        Auth --> View_Dash[View Vendor List]:::officer
        
        View_Dash --> View_Diff[Open Change Detail Page]:::officer
        View_Diff --> Analyze{Is Change Material?}:::decision
        
        Analyze -- Yes (Risk) --> Escalate[Flag as 'Risk' & Add Comment]:::officer
        Escalate --> Create_Ticket[Create Internal Compliance Ticket]:::officer
        
        Analyze -- No (Safe) --> Mark_Reviewed[Click 'Mark as Reviewed']:::officer
        
        Create_Ticket --> Audit_Log
        Mark_Reviewed --> Audit_Log
    end

    subgraph Final_State [Audit & Retention]
        Audit_Log[Log Action in Audit Trail]:::system
        Audit_Log --> Archive[Archive Snapshot to WORM Storage]:::system
        Archive --> Complete((Process Complete)):::endState
    end
```
