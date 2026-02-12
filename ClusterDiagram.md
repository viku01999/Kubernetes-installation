# Kubernetes Networking Diagram – NodePort, Namespaces, Pods, ClusterIP

This document explains how Kubernetes networking works with **two nodes, two namespaces, multiple pods, NodePort, and ClusterIP**. It also shows where **duplicate IPs or ports** are allowed.

---

## Cluster Diagram (ASCII)

```yaml
                 EXTERNAL CLIENT
               IP: 203.0.113.10
                        │
                        │ HTTP Request to NodePort:32051 (dev-nginx)
                        ▼
      ┌─────────────────────────────────────┐
      │         Kubernetes Cluster          │
      │                                     │
      │              Node 1                 │
      │          (192.168.1.50)            │
      │                                     │
      │  ┌─────────────┐   ┌─────────────┐ │
      │  │ Namespace:  │   │ Namespace:  │ │
      │  │ dev         │   │ qa          │ │
      │  │ Pod A       │   │ Pod B       │ │
      │  │ IP:10.1.1.2 │   │ IP:10.1.1.3 │ │
      │  │ Port: 80    │   │ Port: 80    │ │
      │  └─────────────┘   └─────────────┘ │
      │                                     │
      │  ┌─────────────────────────────┐    │
      │  │ Service NodePort             │    │
      │  │ NodePort: 32051 → Pods A/C  │    │
      │  │ ClusterIP: 10.110.0.1       │    │
      │  │ Namespace: dev              │    │
      │  └─────────────────────────────┘    │
      └─────────────────────────────────────┘
      
      ┌─────────────────────────────────────┐
      │              Node 2                 │
      │          (192.168.1.51)            │
      │                                     │
      │  ┌─────────────┐   ┌─────────────┐ │
      │  │ Namespace:  │   │ Namespace:  │ │
      │  │ dev         │   │ qa          │ │
      │  │ Pod C       │   │ Pod D       │ │
      │  │ IP:10.1.1.4 │   │ IP:10.1.1.5 │ │
      │  │ Port: 80    │   │ Port: 80    │ │
      │  └─────────────┘   └─────────────┘ │
      │                                     │
      │  ┌─────────────────────────────┐    │
      │  │ Service NodePort             │    │
      │  │ NodePort: 32051 → Pods A/C  │    │
      │  │ ClusterIP: 10.110.0.1       │    │
      │  │ Namespace: dev              │    │
      │  └─────────────────────────────┘    │
      └─────────────────────────────────────┘
```

---

## Traffic Flow Examples

```yaml

1. **External Client → NodePort → Service → Pod**  
   - Example: `Browser → Node1:32051 → Service dev-nginx → Pod A → Response → Browser`
2. **NodePort Load Balancing**  
   - NodePort 32051 can forward requests to **Pod A (Node1) or Pod C (Node2)** automatically.
3. **Internal Pod-to-Pod Communication**  
   - Pod A (dev) → Pod B (qa) via Pod IP 10.1.1.3:80
   - Pod C (dev) → Pod D (qa) via Pod IP 10.1.1.5:80

---

## Rules for Duplicates

| Component                  | Duplicate Allowed? | Notes                                           |
|-----------------------------|------------------|------------------------------------------------|
| Pod IP                      | ❌ No             | Must be unique cluster-wide                    |
| Pod Port inside pod         | ✅ Yes            | Same port allowed in multiple pods/namespaces |
| ClusterIP (Service)         | ❌ No             | Must be unique                                 |
| NodePort                     | ❌ No             | Unique cluster-wide, but load-balances to pods|
| NodePort → multiple pods     | ✅ Yes            | Requests can go to any pod in the service     |
| Namespace pod name          | ✅ Yes            | Same pod name allowed across namespaces       |
| Client IP                   | -                | External client sending requests              |

---

## Summary (Memory Trick)

- **Namespace** = department / logical isolation  
- **NodePort** = main gate for external traffic  
- **Service** = receptionist / traffic director  
- **Pods** = rooms/employees handling requests  
- **ClusterIP** = internal IP for pod-to-pod communication

> NodePort lets external clients reach all pods in a service, across nodes and namespaces.  
> Pod ports can be duplicated inside pods, but NodePort and Pod IPs must be unique.


- **Pod IPs** are **unique cluster-wide** (10.1.1.2–10.1.1.5).  
- **Pod Ports** (80) can be duplicated across pods and namespaces.  
- **NodePort** (32051) is **unique cluster-wide**, but it **can forward requests to multiple pods**.  
- **ClusterIP** (10.110.0.1) is **internal service IP** for pod-to-pod communication.  
- **Namespaces** provide logical isolation; same pod names can exist in different namespaces.  
- External client always talks to the **NodePort**; the service load-balances traffic to pods across nodes.  

Node = 🏠 house

Pod = 🛏️ room inside the house

Service = 🛎 receptionist

NodePort = 🚪 main gate with the same door number for all houses

```

## Kubernetes NodePort & Namespaces Diagram (Mermaid)

```mermaid
graph TD
    %% ================= EXTERNAL CLIENT =================
    Client[💻 External Client\nIP: 203.0.113.10]

    %% ================= NODE 1 =================
    subgraph Node1["🖥 Node 1\n192.168.1.50"]
        style Node1 fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px

        subgraph DevNS1["📦 Namespace: dev"]
            PodA[🐳 Pod A\nIP:10.1.1.2\nPort:80]
            style PodA fill:#C6F1D6,stroke:#2CA02C,stroke-width:2px
        end

        subgraph QaNS1["📦 Namespace: qa"]
            PodB[🐳 Pod B\nIP:10.1.1.3\nPort:80]
            style PodB fill:#FFE0B2,stroke:#FF7F0E,stroke-width:2px
        end

        Service1[🛡 NodePort Service\nNodePort:32051\nClusterIP:10.110.0.1\nNamespace: dev]
        style Service1 fill:#D0E1FF,stroke:#1F77B4,stroke-width:2px,stroke-dasharray:5 5
    end

    %% ================= NODE 2 =================
    subgraph Node2["🖥 Node 2\n192.168.1.51"]
        style Node2 fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px

        subgraph DevNS2["📦 Namespace: dev"]
            PodC[🐳 Pod C\nIP:10.1.1.4\nPort:80]
            style PodC fill:#C6F1D6,stroke:#2CA02C,stroke-width:2px
        end

        subgraph QaNS2["📦 Namespace: qa"]
            PodD[🐳 Pod D\nIP:10.1.1.5\nPort:80]
            style PodD fill:#FFE0B2,stroke:#FF7F0E,stroke-width:2px
        end

        Service2[🛡 NodePort Service\nNodePort:32051\nClusterIP:10.110.0.1\nNamespace: dev]
        style Service2 fill:#D0E1FF,stroke:#1F77B4,stroke-width:2px,stroke-dasharray:5 5
    end

    %% ================= CONNECTIONS =================
    Client -->|HTTP Request NodePort:32051| Service1
    Client -->|HTTP Request NodePort:32051| Service2

    Service1 --> PodA
    Service1 --> PodC
    Service2 --> PodA
    Service2 --> PodC

    %% Internal pod-to-pod traffic
    PodA ---|Internal traffic| PodB
    PodC ---|Internal traffic| PodD

    %% ================= STYLES =================
    classDef client fill:#FFF3E0,stroke:#FB8C00,stroke-width:2px,color:#E65100;
    classDef service fill:#D0E1FF,stroke:#1F77B4,stroke-width:2px,color:#1F77B4,stroke-dasharray:5 5;
    classDef node fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#0D47A1;
    classDef pod_dev fill:#C6F1D6,stroke:#2CA02C,stroke-width:2px,color:#2CA02C;
    classDef pod_qa fill:#FFE0B2,stroke:#FF7F0E,stroke-width:2px,color:#FF7F0E;

    %% APPLY CLASSES
    class Client client;
    class Service1,Service2 service;
    class Node1,Node2 node;
    class PodA,PodC pod_dev;
    class PodB,PodD pod_qa;
