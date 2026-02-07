```mermaid
graph TD
    %% ===== USER / EXTERNAL =====
    U[👤 User / CI-CD System<br/>kubectl · API Client] -->|REST Request<br/>Create/Update/Delete| API[🧠 kube-apiserver<br/>Authentication · Authorization · Admission · Validate YAML]

    %% ===== PERSISTENT STATE =====
    API -->|Write / Read State<br/>Desired State| ETCD[(🗄 etcd<br/>Single Source of Truth<br/>Cluster State DB)]

    %% ===== CONTROL LOGIC =====
    API -->|Trigger Reconciliation<br/>Detect Drift| CTRL[⚙ kube-controller-manager<br/>Replica · Node · Endpoint Controllers<br/>Self-Healing · Auto Scaling]

    CTRL -->|State Mismatch Detected| API

    %% ===== SCHEDULING =====
    API -->|Schedule Pod<br/>Node Selection| SCH[📍 kube-scheduler<br/>CPU · Memory · Affinity · Constraints]
    SCH -->|Binding Decision| API

    %% ===== WORKER NODE INTERACTION =====
    API -->|Assign PodSpec| KUBELET[🖥 kubelet<br/>Node Agent<br/>Pod Lifecycle · Health Reporting]

    KUBELET -->|Node / Pod Status<br/>Heartbeat| API

    %% ================= STYLES =================
    classDef user fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#0D47A1;
    classDef api fill:#FFF3E0,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef state fill:#E8F5E9,stroke:#43A047,stroke-width:2px,color:#1B5E20;
    classDef control fill:#FFFDE7,stroke:#F9A825,stroke-width:2px,color:#F57F17;
    classDef scheduler fill:#FFE0B2,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef node fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#4A148C;

    %% ================= APPLY CLASSES =================
    class U user;
    class API api;
    class ETCD state;
    class CTRL control;
    class SCH scheduler;
    class KUBELET node;
