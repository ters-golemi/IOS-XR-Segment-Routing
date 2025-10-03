# IOS XR Segment Routing Configuration Templates

Comprehensive configuration templates for deploying Segment Routing (SR) on Cisco IOS XR routers in a Service Provider network environment.

## Overview

This repository provides production-ready configuration templates for implementing Segment Routing across a three-tier Service Provider network architecture:

- **Core Routers (P Nodes)**: High-capacity backbone routers
- **Aggregation Routers (PE Nodes)**: Service aggregation layer
- **Edge Routers (PE Nodes)**: Customer-facing access routers

## Repository Structure

```
.
├── README.md                                  # This file
├── TOPOLOGY.md                                # Network topology and architecture
├── CONFIGURATION-GUIDE.md                     # Detailed configuration guide
└── configs/
    ├── core-router-template.txt              # Core router template
    ├── aggregation-router-template.txt       # Aggregation router template
    └── edge-router-template.txt              # Edge router template
```

## Key Features

### Segment Routing Implementation
- **SRGB (Segment Routing Global Block)**: 16000-23999 label range
- **Node SIDs**: Hierarchical assignment (100s=core, 200s=agg, 300s=edge)
- **IS-IS with SR extensions**: Level-2 only configuration
- **TI-LFA (Topology Independent Loop-Free Alternate)**: Sub-50ms convergence
- **MPLS data plane**: Full SR-MPLS implementation

### High Availability Features
- **BFD (Bidirectional Forwarding Detection)**: Fast failure detection (<1 second)
- **NSR/NSF (Non-Stop Routing/Forwarding)**: Graceful restart capabilities
- **Microloop avoidance**: Prevents transient forwarding loops
- **Redundant paths**: Multiple uplink connectivity

### Service Provider Services
- **L3VPN**: Full VRF and BGP VPNv4 configuration
- **L2VPN**: VPWS and EVPN examples (edge routers)
- **QoS**: Multi-class traffic management with DSCP marking
- **Route Policies**: Customer route filtering and control

### Management and Security
- **SSH and NETCONF**: Secure remote management
- **AAA**: Authentication, Authorization, and Accounting
- **ACLs**: Access control lists for security
- **Syslog and NTP**: Centralized logging and time synchronization

## Quick Start

### 1. Review Network Topology

Start by reading [`TOPOLOGY.md`](TOPOLOGY.md) to understand:
- Network architecture and design
- IP addressing scheme
- Node roles and connectivity
- Segment ID allocation

### 2. Read Configuration Guide

Review [`CONFIGURATION-GUIDE.md`](CONFIGURATION-GUIDE.md) for:
- Segment Routing concepts and theory
- Detailed configuration explanations
- Deployment procedures
- Verification commands
- Troubleshooting guidance
- Best practices

### 3. Customize Templates

The configuration templates in the `configs/` directory contain:
- **Placeholders**: Replace `X.X.X.X`, `XXX`, etc. with actual values
- **Comments**: Guidance for customization (lines starting with `!!`)
- **Examples**: Sample configurations for common scenarios

#### Core Router Template
```bash
# Edit: configs/core-router-template.txt
# Replace:
- hostname CORE-X → hostname CORE-1
- ipv4 address X.X.X.X → ipv4 address 1.1.1.1
- prefix-sid index XXX → prefix-sid index 101
- NET address → net 49.0001.0001.0001.0001.00
```

#### Aggregation Router Template
```bash
# Edit: configs/aggregation-router-template.txt
# Replace:
- hostname AGG-X → hostname AGG-1
- ipv4 address X.X.X.X → ipv4 address 10.1.1.1
- prefix-sid index XXX → prefix-sid index 201
- NET address → net 49.0001.0010.0001.0001.00
```

#### Edge Router Template
```bash
# Edit: configs/edge-router-template.txt
# Replace:
- hostname EDGE-X → hostname EDGE-1
- ipv4 address X.X.X.X → ipv4 address 20.1.1.1
- prefix-sid index XXX → prefix-sid index 301
- NET address → net 49.0001.0020.0001.0001.00
```

### 4. Deploy Configuration

#### Method 1: CLI Copy-Paste
```bash
# Enter configuration mode
configure

# Paste configuration sections
[paste configuration]

# Commit changes
commit

# Verify
show running-config
```

#### Method 2: NETCONF/RESTCONF
```bash
# Use NETCONF/RESTCONF APIs for programmatic deployment
# Refer to Cisco IOS XR programmability guides
```

#### Method 3: Configuration Files
```bash
# Copy configuration file to router
# Load configuration
load configuration <filename>
commit
```

## Network Topology

```
                        ┌──────────────────────────────┐
                        │    Service Provider Core     │
                        │                              │
              ┌─────────┼────────────────────┬─────────┼─────────┐
              │         │                    │         │         │
         ┌────▼────┐    │               ┌────▼────┐    │    ┌────▼────┐
         │  CORE-1 │────┼───────────────│  CORE-2 │────┼────│  CORE-3 │
         │SID: 101 │    │               │SID: 102 │    │    │SID: 103 │
         └────┬────┘    │               └────┬────┘    │    └────┬────┘
              │         │                    │         │         │
              │    Aggregation Layer         │         │         │
              │                              │         │         │
    ┌─────────┴─────────┐          ┌─────────┴─────────┐        │
    │                   │          │                   │        │
┌───▼─────┐      ┌──────▼────┐ ┌───▼─────┐      ┌──────▼────┐  │
│  AGG-1  │──────│   AGG-2   │ │  AGG-3  │──────│   AGG-4   │  │
│SID: 201 │      │ SID: 202  │ │SID: 203 │      │ SID: 204  │  │
└───┬─────┘      └──────┬────┘ └───┬─────┘      └──────┬────┘  │
    │                   │          │                   │        │
    │     Edge Layer    │          │     Edge Layer    │        │
    │                   │          │                   │        │
┌───▼─────┐      ┌──────▼────┐ ┌───▼─────┐      ┌──────▼────┐  │
│ EDGE-1  │      │  EDGE-2   │ │ EDGE-3  │      │  EDGE-4   │  │
│SID: 301 │      │ SID: 302  │ │SID: 303 │      │ SID: 304  │  │
└───┬─────┘      └──────┬────┘ └───┬─────┘      └──────┬────┘  │
    │                   │          │                   │        │
Customer           Customer    Customer           Customer     │
Sites              Sites       Sites              Sites        │
```

See [`TOPOLOGY.md`](TOPOLOGY.md) for complete topology details.

## Configuration Highlights

### Segment Routing Global Block (SRGB)
```
segment-routing
 global-block 16000 23999
```
All routers use the same SRGB range, providing 8000 labels for SR operations.

### Node SID Assignment
Each router is assigned a unique Segment ID:
```
interface Loopback0
 address-family ipv4 unicast
  prefix-sid index 101
```
This creates label 16101 (16000 + 101) for CORE-1.

### TI-LFA for Fast Convergence
```
fast-reroute per-prefix
fast-reroute per-prefix ti-lfa
```
Provides sub-50ms failover without additional protocols.

### BFD for Fast Failure Detection
```
bfd mode ietf
bfd address-family ipv4 fast-detect
```
Detects link failures in milliseconds.

## Verification Commands

### Check Segment Routing Status
```bash
# Verify SR labels
show segment-routing mpls connected-prefix-sid-map ipv4

# Check SR forwarding
show mpls forwarding

# Verify IS-IS SR database
show isis database verbose
```

### Check Network Connectivity
```bash
# Verify IS-IS adjacencies
show isis adjacency

# Check BGP sessions
show bgp summary

# Verify BFD sessions
show bfd session
```

### Verify TI-LFA
```bash
# Check backup paths
show isis fast-reroute detail

# Verify CEF backup
show cef <prefix> detail
```

## Troubleshooting

Common issues and solutions are documented in [`CONFIGURATION-GUIDE.md`](CONFIGURATION-GUIDE.md#troubleshooting).

Quick troubleshooting commands:
```bash
# IS-IS issues
show isis adjacency detail
show isis interface detail
show isis database verbose

# SR issues
show segment-routing mpls lb
show mpls forwarding

# BGP issues
show bgp vpnv4 unicast summary
show bgp neighbors

# BFD issues
show bfd session detail
```

## Best Practices

1. **SRGB Consistency**: Ensure all routers use identical SRGB ranges
2. **Node SID Planning**: Use hierarchical index assignment (100s, 200s, 300s)
3. **Incremental Deployment**: Deploy one router at a time
4. **Test in Lab**: Validate configurations before production deployment
5. **Monitor Convergence**: Track failover times and optimize if needed
6. **Document Changes**: Keep network diagrams and SID allocation tables current

See [`CONFIGURATION-GUIDE.md`](CONFIGURATION-GUIDE.md#best-practices) for comprehensive best practices.

## Prerequisites

### Software Requirements
- Cisco IOS XR 6.0 or later (7.x recommended for latest SR features)
- Segment Routing feature set enabled

### Hardware Requirements
- Cisco ASR 9000 series
- Cisco NCS 5500 series
- Cisco 8000 series
- Other IOS XR-capable platforms

### Knowledge Requirements
- Basic understanding of MPLS and Label Switching
- Familiarity with IS-IS or OSPF routing protocols
- Understanding of BGP and VPN technologies
- IOS XR CLI experience

## Advanced Topics

The configuration templates support advanced features:

- **SR Traffic Engineering (SR-TE)**: Explicit path control
- **On-Demand Next-Hop (ODN)**: Automated TE policy creation
- **Anycast SIDs**: Service redundancy
- **SR Performance Measurement**: Latency monitoring
- **Flexible Algorithm**: Multiple path computation algorithms

See [`CONFIGURATION-GUIDE.md`](CONFIGURATION-GUIDE.md#advanced-topics) for detailed information.

## References

### Cisco Documentation
- [IOS XR Segment Routing Configuration Guide](https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r7-3/segment-routing/configuration/guide/b-segment-routing-cg-asr9000-73x.html)
- [IOS XR MPLS Configuration Guide](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-xr-software/products-installation-and-configuration-guides-list.html)

### IETF Standards
- [RFC 8402: Segment Routing Architecture](https://datatracker.ietf.org/doc/html/rfc8402)
- [RFC 8660: Segment Routing with MPLS Data Plane](https://datatracker.ietf.org/doc/html/rfc8660)
- [RFC 8667: IS-IS Extensions for Segment Routing](https://datatracker.ietf.org/doc/html/rfc8667)

### Learning Resources
- Cisco Live presentations on Segment Routing
- Cisco DevNet Segment Routing learning labs
- Segment Routing community forums

## Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly in a lab environment
5. Submit a pull request with detailed description

## License

This project is provided as-is for educational and deployment purposes.

## Support

For issues or questions:
- Review the [Configuration Guide](CONFIGURATION-GUIDE.md)
- Check the [Troubleshooting section](CONFIGURATION-GUIDE.md#troubleshooting)
- Consult Cisco Technical Support for production deployments

## Changelog

### Version 1.0 (Initial Release)
- Core router configuration template
- Aggregation router configuration template
- Edge router configuration template
- Network topology documentation
- Comprehensive configuration guide
- Complete documentation set

## Acknowledgments

Based on Cisco IOS XR best practices and real-world Service Provider deployments.
