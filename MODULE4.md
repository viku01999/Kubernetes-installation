```mermaid
graph TD
    %% ===== USER / EXTERNAL =====
    U[👤 User / DevOps Engineer<br/>kubectl · YAML · CI/CD] -->|Apply Manifest<br/>Create/Update/Delete| API[🧠 kube-apiserver<br/>Validation · Auth · Admission]

    %% ===== PERSISTENT STATE =====
    API -->|Store Desired State| ETCD[(🗄 etcd<br/>Cluster State)]

    %% ===== CONTROL LOGIC =====
    API -->|Watch & Reconcile| CTRL[⚙ kube-controller-manager<br/>ReplicaSet · Deployment · Self-Healing]

    %% ===== SCHEDULER =====
    API -->|Schedule Pod| SCH[📍 kube-scheduler<br/>CPU · Memory · Node Constraints]
    SCH -->|Binding Decision| API

    %% ===== WORKER NODE =====
    API -->|Assign PodSpec| KUBELET[🖥 kubelet<br/>Pod Lifecycle · Health Checks]

    KUBELET -->|Create & Manage| POD[📦 Pod<br/>Containers · Volumes · IP · Labels]

    %% ===== CONTAINER =====
    POD -->|Run Application| CTR[🐳 Container<br/>App Process]

    %% ===== NETWORKING =====
    POD -->|Service Traffic| PROXY[🌐 kube-proxy<br/>Service Networking · Load Balancing]
    PROXY -->|Forward Requests| POD

    %% ===== CONFIG / SECRET =====
    POD -->|Consume Config| CONFIG[🗃 ConfigMap / Secret<br/>Env / Volumes]

    %% ===== HEALTH & STATUS =====
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
    classDef network fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#0D47A1;
    classDef config fill:#FFFDE7,stroke:#8E24AA,stroke-width:2px,color:#4A148C;

    %% ================= APPLY CLASSES =================
    class U user;
    class API api;
    class ETCD state;
    class CTRL control;
    class SCH scheduler;
    class KUBELET node;
    class POD,CTR workload;
    class PROXY network;
    class CONFIG config;
