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
    B ---|Health probe successful| X["Wireguard Server 1 (Active)"]
    B ---|Health Probe failed| Y["Wireguard Server 2 (Pseudo Passive)"]
    X -->|Connect to Private IP| E[Extension Box Backend]
    Y -->|Connect to Private IP| E[Extension Box Backend]
```

Disadvantages:

- No Active / Passive setup possible (Azure LoadBalancer limitation)
- Connections to Wireguard servers are random because of Active / Active
- Reverse connection from Backend to Client is random because of possible async routing (Active / Active issues)
- Needs to manually set health probe of one Wireguard server as failed and needs to set the probe as healthy in case of incident so that synchronous routing works

Advantages:

- Native Azure components
- Automated with ARM deployments

## HA Solution with Headscale

Headscale is an open source version of Tailscale which makes it easy to connect multiple systems in a mesh network. The solution is deployed inside an AKS Cluster (HA by default) which acts as the server. Clients connect to the server. If direct connectivity cannot be ensured the connection will be made over a relay server in the internet. All communication is End-To-End encrypted with modern TLS encryption.

```mermaid
flowchart TD
    A[Extension Box Client] -->|Connect to Public IP| B{Load Balancer}
    B --> C[AKS Cluster]
    C --> D{Headscale Server}
    D --> X[Tailscale Relay]
    X --> F["Extension Box Backend (Via Relay)"]
    D --> E["Extension Box Backend (Direct)"]
```

Advantages:

- Communication is possible both ways
- Uses clever technologies to do NAT traversal
- Uses relays (hosted by Tailscale) to make communication possible everywhere

Disadvantages:

- Setup on client on server is more involved
- Communication via relay causes slightly more latency

## Tailscale (Hosted in the Cloud by Tailscale)

Tailscale is the commercial version of Headscale that enables users to create their own private mesh networks.

Advantages:

- Hosted externally. No patching, backup, etc. of servers neccessary
- Simplified configuration: Clients just connect by opening a link and authenticating themselves.
- Uses Wireguard as underlying meshing solution.
- Is HA already since it is managed by Tailscale.

Disadvantages:

- Cost: Free version sets a maximum of 3 Users (for Web interface) and 100 devices. For more users and some more special features there are other price plans.
- Since it is a service it has to be trusted not to do nefarious things with your data.