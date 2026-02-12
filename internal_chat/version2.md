```mermaid
graph TD
    %% ================= NODES & EXTERNAL VIP =================
    Node1[🏢 Node 1<br/>IP: 192.168.29.13]
    Node2[🏢 Node 2<br/>IP: 192.168.29.14]
    LoadBalancer[🌉 LoadBalancer VIP<br/>EXTERNAL IP: 20.30.40.50]

    %% ================= NODEPORT GUARDS =================
    NodePortA[👮 NodePort 32051]
    NodePortB[👮 NodePort 32052]

    %% ================= DEPLOYMENTS =================
    DeploymentA[🎩 Deployment: user-service]
    DeploymentB[🎩 Deployment: order-service]

    %% ================= DANCING PODS =================
    PodA1[🕺 Pod A1<br/>user-service<br/>IP: 192.168.199.74]
    PodA2[💃 Pod A2<br/>user-service<br/>IP: 192.168.199.75]
    PodB1[🕺 Pod B1<br/>order-service<br/>IP: 192.168.199.76]
    PodB2[💃 Pod B2<br/>order-service<br/>IP: 192.168.199.77]

    %% ================= INTERNAL SERVICES =================
    ServiceA[📜 ClusterIP Service: user-service]
    ServiceB[📜 ClusterIP Service: order-service]

    %% ================= EXTERNAL INTERNET USERS =================
    Internet[🌎 Internet Users]

    %% ================= CONNECTIONS =================
    LoadBalancer --> NodePortA
    LoadBalancer --> NodePortB

    NodePortA --> ServiceA
    NodePortB --> ServiceB
    Internet --> LoadBalancer

    DeploymentA --> PodA1
    DeploymentA --> PodA2
    DeploymentB --> PodB1
    DeploymentB --> PodB2

    ServiceA -->|Select Pods by label app=user-service| PodA1
    ServiceA -->|Select Pods by label app=user-service| PodA2
    ServiceB -->|Select Pods by label app=order-service| PodB1
    ServiceB -->|Select Pods by label app=order-service| PodB2

    %% ================= POD-TO-POD INTERNAL COMMUNICATION =================
    PodA1 -->|DNS gossip 🗣️ order-service.default.svc.cluster.local| ServiceB
    PodA2 -->|DNS gossip 🗣️ order-service.default.svc.cluster.local| ServiceB
    PodB1 -->|DNS gossip 🗣️ user-service.default.svc.cluster.local| ServiceA
    PodB2 -->|DNS gossip 🗣️ user-service.default.svc.cluster.local| ServiceA

    %% ================= NODES HOSTING PODS =================
    Node1 --> PodA1
    Node1 --> PodB1
    Node2 --> PodA2
    Node2 --> PodB2
