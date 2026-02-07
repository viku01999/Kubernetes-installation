```mermaid
graph TD
    %% ===== CONTROL PLANE INTERACTION =====
    API[🧠 kube-apiserver<br/>Pod Assignment · Status API] -->|Assign PodSpec| KUBELET[🖥 kubelet<br/>Node Agent<br/>• Pod Lifecycle<br/>• Health Checks<br/>• Talks to API]

    %% ===== KUBELET → RUNTIME =====
    KUBELET -->|CRI Calls| CRI[🔌 CRI<br/>Container Runtime Interface]
    CRI -->|Create / Start / Stop| RUNTIME[⚙ Container Runtime<br/>containerd]

    %% ===== IMAGE MANAGEMENT =====
    RUNTIME -->|Pull Image| REG[📥 Image Registry<br/>Docker Hub / Private]
    RUNTIME -->|Run Process| CTR[🐳 Container<br/>Application Process]

    %% ===== POD MANAGEMENT =====
    KUBELET -->|Create & Manage| POD[📦 Pod<br/>• Smallest Unit<br/>• Pod IP<br/>• Volumes]

    CTR -->|Runs Inside| POD

    %% ===== NETWORKING =====
    POD -->|Traffic via Service IP| PROXY[🌐 kube-proxy<br/>• Service Networking<br/>• Load Balancing<br/>• iptables / IPVS]
    PROXY -->|Forward Traffic| POD

    %% ===== HEALTH & STATUS =====
    CTR -->|Exit Code · Health| KUBELET
    KUBELET -->|NodeStatus · PodStatus| API

    %% ===== STYLES =====
    classDef api fill:#FFF3E0,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef node fill:#F3E5F5,stroke:#8E24AA,stroke-width:2px,color:#4A148C;
    classDef runtime fill:#E8F5E9,stroke:#43A047,stroke-width:2px,color:#1B5E20;
    classDef workload fill:#FCE4EC,stroke:#D81B60,stroke-width:2px,color:#880E4F;
    classDef network fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#0D47A1;
    classDef external fill:#ECEFF1,stroke:#546E7A,stroke-width:2px,color:#263238;

    %% ===== APPLY STYLES =====
    class API api;
    class KUBELET node;
    class CRI,RUNTIME,REG runtime;
    class POD,CTR workload;
    class PROXY network;
