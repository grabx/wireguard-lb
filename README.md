# Wireguard Connections

## Single Instance

```mermaid
flowchart TD
    A[Extension Box Client] -->|Connect to Public IP| C{Wireguard Server}
    C -->|Connect to Private IP| D[Extension Box Backend]
```

Disadvantages:

- No HA

Advantages:

- Less complexity, simple setup
- Routing both ways is synchronous

## Active / Passive with AKS

```mermaid
flowchart TD
    A[Extension Box Client] -->|Connect to Public IP| B{Load Balancer}
    B --> C[AKS Cluster]
    C --> D{UDP Load Balancer}
    D --- X["Wireguard Server 1 (Active)"]
    D --- Y["Wireguard Server 2 (Passive)"]
    X -->|Connect to Private IP| E[Extension Box Backend]
    Y -->|Connect to Private IP| E[Extension Box Backend]
```

Disadvantages:

- Needs correctly configured NGINX reverse proxy

Advantages:

- Routing both ways is synchronous
- HA Setup possible
- Wireguard servers can be configured as active / passive
- Proxy runs as container

## Active / Active with Azure Loadbalancer

```mermaid
flowchart TD
    A[Extension Box Client] -->|Connect to Public IP| B{Load Balancer}
    B --- X["Wireguard Server 1"]
    B --- Y["Wireguard Server 2"]
    X -->|Connect to Private IP| E[Extension Box Backend]
    Y -->|Connect to Private IP| E[Extension Box Backend]
```

Disadvantages:

- No Active / Passive setup possible (Azure LoadBalancer limitation)
- Connections to Wireguard servers are random because of Active / Active
- Reverse connection from Backend to Client is random because of possible async routing (Active / Active issues)

Advantages:

- Native Azure components
- Automated with ARM deployments