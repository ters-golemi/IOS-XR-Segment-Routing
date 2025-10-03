# Network Topology

## Service Provider Network Architecture

This template demonstrates a typical Service Provider network with three layers implementing Segment Routing (SR):

```
                        ┌──────────────────────────────┐
                        │    Service Provider Core     │
                        │                              │
              ┌─────────┼────────────────────┬─────────┼─────────┐
              │         │                    │         │         │
         ┌────▼────┐    │               ┌────▼────┐    │    ┌────▼────┐
         │  CORE-1 │────┼───────────────│  CORE-2 │────┼────│  CORE-3 │
         │ (P Node)│    │               │ (P Node)│    │    │ (P Node)│
         │ SID:101 │────┴───────────────│ SID:102 │────┴────│ SID:103 │
         └────┬────┘                    └────┬────┘         └────┬────┘
              │                              │                   │
              │      Aggregation Layer       │                   │
              │                              │                   │
    ┌─────────┴─────────┐          ┌─────────┴─────────┐        │
    │                   │          │                   │        │
┌───▼─────┐      ┌──────▼────┐ ┌───▼─────┐      ┌──────▼────┐  │
│  AGG-1  │──────│   AGG-2   │ │  AGG-3  │──────│   AGG-4   │  │
│(PE Node)│      │ (PE Node) │ │(PE Node)│      │ (PE Node) │  │
│SID: 201 │      │ SID: 202  │ │SID: 203 │      │ SID: 204  │  │
└───┬─────┘      └──────┬────┘ └───┬─────┘      └──────┬────┘  │
    │                   │          │                   │        │
    │     Edge Layer    │          │     Edge Layer    │        │
    │                   │          │                   │        │
┌───▼─────┐      ┌──────▼────┐ ┌───▼─────┐      ┌──────▼────┐  │
│ EDGE-1  │      │  EDGE-2   │ │ EDGE-3  │      │  EDGE-4   │  │
│(PE Node)│      │ (PE Node) │ │(PE Node)│      │ (PE Node) │  │
│SID: 301 │      │ SID: 302  │ │SID: 303 │      │ SID: 304  │  │
└───┬─────┘      └──────┬────┘ └───┬─────┘      └──────┬────┘  │
    │                   │          │                   │        │
    │                   │          │                   │        │
 Customer           Customer    Customer           Customer     │
 Sites              Sites       Sites              Sites        │
                                                                 │
                                ┌────────────────────────────────┘
                                │
                          ┌─────▼──────┐
                          │   EDGE-5   │
                          │ (PE Node)  │
                          │  SID: 305  │
                          └─────┬──────┘
                                │
                            Customer
                            Sites
```

## Node Roles and Descriptions

### Core Routers (P Nodes - Provider Routers)
- **CORE-1, CORE-2, CORE-3**: High-capacity routers forming the backbone
- Segment ID Range: 101-199
- Primary function: Packet forwarding using SR labels
- No customer-facing interfaces
- Full mesh connectivity for redundancy

### Aggregation Routers (PE Nodes - Provider Edge)
- **AGG-1, AGG-2, AGG-3, AGG-4**: Mid-layer routers
- Segment ID Range: 201-299
- Connect core to edge routers
- Handle service aggregation and policy enforcement
- Can terminate customer services (L3VPN, L2VPN)

### Edge Routers (PE Nodes - Provider Edge)
- **EDGE-1 through EDGE-5**: Customer-facing routers
- Segment ID Range: 301-399
- Direct customer connectivity
- Service ingress/egress points
- Traffic classification and QoS marking

## IP Addressing Scheme

### Loopback Interfaces (Node SIDs)
- CORE-1: 1.1.1.1/32
- CORE-2: 2.2.2.2/32
- CORE-3: 3.3.3.3/32
- AGG-1: 10.1.1.1/32
- AGG-2: 10.2.2.2/32
- AGG-3: 10.3.3.3/32
- AGG-4: 10.4.4.4/32
- EDGE-1: 20.1.1.1/32
- EDGE-2: 20.2.2.2/32
- EDGE-3: 20.3.3.3/32
- EDGE-4: 20.4.4.4/32
- EDGE-5: 20.5.5.5/32

### Point-to-Point Links
Links use /30 subnets from 172.16.0.0/16 range:
- Core-to-Core: 172.16.0.0/24 subnet
- Core-to-Aggregation: 172.16.1.0/24 to 172.16.4.0/24 subnets
- Aggregation-to-Edge: 172.16.10.0/24 to 172.16.20.0/24 subnets

## Segment Routing Design

### SRGB (Segment Routing Global Block)
- Range: 16000-23999 (8000 labels)
- Applied uniformly across all nodes
- Node SID = SRGB_BASE + Index
  - Example: CORE-1 with index 101 → Label 16101

### IGP Configuration
- Protocol: IS-IS (Intermediate System to Intermediate System)
- Area: 49.0001
- SR Extension: Enabled
- TI-LFA (Topology Independent Loop-Free Alternate): Enabled for fast convergence

### Traffic Engineering
- SR-TE (Segment Routing Traffic Engineering) policies
- ODN (On-Demand Next-hop) for automated TE
- Anycast SIDs for redundancy scenarios (optional)

## Redundancy and High Availability

1. **Core Layer**: Full mesh provides multiple paths
2. **Aggregation Layer**: Dual-homed to core routers
3. **Edge Layer**: Dual-homed to aggregation where possible
4. **TI-LFA**: Provides sub-50ms convergence
5. **BFD**: Bidirectional Forwarding Detection for fast failure detection
