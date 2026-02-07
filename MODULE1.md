```mermaid
graph TD
    %% ===== USER =====
    U[👤 User / DevOps Engineer<br/>kubectl · YAML · CI/CD] -->|Declare Desired State<br/>Create/Update/Delete| API[🧠 kube-apiserver<br/>REST API · Auth · Validation]

    %% ===== CONTROL PLANE =====
    API -->|Persist State / Desired State| ETCD[(🗄 etcd<br/>Cluster State DB<br/>Single Source of Truth)]

    API -->|Trigger Reconciliation / Healing| CTRL[⚙ kube-controller-manager<br/>Replica · Node · Endpoint Controllers<br/>Self-Healing · Auto-Scaling]

    CTRL -->|State Mismatch Detected| API

    API -->|Schedule Pod / Node Selection| SCH[📍 kube-scheduler<br/>CPU · Memory · Affinity · Constraints]
    SCH -->|Binding Decision| API

    %% ===== WORKER NODE =====
    API -->|Assign PodSpec| KUBELET[🖥 kubelet<br/>Node Agent<br/>Pod Lifecycle · Health Reporting]

    KUBELET -->|Create Pod| POD[📦 Pod<br/>Networking · Volumes · Namespace]
    POD -->|Run Container| CTR[🐳 Container<br/>Application Process]

    CTR -->|Health Status / Exit Code| KUBELET
    KUBELET -->|Node / Pod Status| API

    %% ================= STYLES =================
    classDef user fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#0D47A1;
    classDef api fill:#FFF3E0,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef state fill:#E8F5E9,stroke:#43A047,stroke-width:2px,color:#1B5E20;
    classDef control fill:#FFFDE7,stroke:#F9A825,stroke-width:2px,color:#F57F17;
    classDef scheduler fill:#FFE0B2,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef node fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#4A148C;
    classDef workload fill:#FCE4EC,stroke:#D81B60,stroke-width:2px,color:#880E4F;

    %% ================= APPLY CLASSES =================
    class U user;
    class API api;
    class ETCD state;
    class CTRL control;
    class SCH scheduler;
    class KUBELET node;
    class POD,CTR workload;
