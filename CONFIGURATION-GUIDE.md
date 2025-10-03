# Segment Routing Configuration Guide for IOS XR

## Table of Contents
1. [Introduction](#introduction)
2. [Segment Routing Concepts](#segment-routing-concepts)
3. [Configuration Components](#configuration-components)
4. [Deployment Steps](#deployment-steps)
5. [Verification Commands](#verification-commands)
6. [Troubleshooting](#troubleshooting)
7. [Best Practices](#best-practices)

## Introduction

This guide provides detailed explanations for configuring Segment Routing (SR) on Cisco IOS XR routers in a Service Provider network. The configuration templates cover three router roles:
- **Core Routers (P Nodes)**: Backbone routers that forward traffic based on SR labels
- **Aggregation Routers (PE Nodes)**: Mid-layer routers providing service aggregation
- **Edge Routers (PE Nodes)**: Customer-facing routers providing service access

## Segment Routing Concepts

### What is Segment Routing?

Segment Routing (SR) is a source-based routing paradigm that simplifies traffic engineering and service delivery in modern networks. Key characteristics:

- **Source Routing**: The ingress router determines the path by encoding it in the packet header
- **Simplified Control Plane**: Eliminates the need for additional signaling protocols (like RSVP-TE)
- **Scalability**: Reduces state information in the network core
- **Flexibility**: Enables fine-grained traffic engineering and service chaining

### SR Components

#### 1. Segment Routing Global Block (SRGB)
- A range of MPLS labels reserved for SR
- Must be consistent across all routers in the domain
- Default range in our template: **16000-23999** (8000 labels)
- Provides the foundation for Node SIDs and other SR labels

#### 2. Segment ID (SID)
A SID is an identifier for a segment in the network:

**Node SID**: 
- Identifies a specific router's loopback interface
- Globally unique within the SR domain
- Calculated as: `Label = SRGB_START + Index`
- Example: CORE-1 with index 101 → Label 16101

**Adjacency SID**:
- Identifies a specific link between two routers
- Locally significant to the advertising router
- Automatically allocated by IS-IS/OSPF

**Prefix SID**:
- Associated with an IP prefix
- Can be used to reach any destination

#### 3. Segment Routing Path
A path through the network is encoded as a stack of SIDs:
- Top label is processed first
- Label is popped when the segment is completed
- Enables explicit path control

### IGP Extensions for SR

Our configuration uses **IS-IS** with SR extensions:

1. **TLV Extensions**: IS-IS carries SR information in new TLVs
2. **Prefix-SID Sub-TLV**: Advertises Node SIDs
3. **Adjacency-SID Sub-TLV**: Advertises link SIDs
4. **SRGB Sub-TLV**: Advertises the SRGB range

## Configuration Components

### 1. Global Segment Routing Configuration

```
segment-routing
 global-block 16000 23999
```

**Purpose**: Defines the label range for SR
**Considerations**:
- Must be identical on all routers
- Should not overlap with other MPLS label ranges
- Size depends on network scale (number of nodes)

### 2. IS-IS Configuration with SR

```
router isis CORE
 net 49.0001.XXXX.XXXX.XXXX.00
 address-family ipv4 unicast
  segment-routing mpls
  segment-routing prefix-sid-map advertise-local
```

**Key Elements**:
- `segment-routing mpls`: Enables SR with MPLS data plane
- `prefix-sid-map advertise-local`: Advertises local prefix-SID mappings
- `microloop avoidance segment-routing`: Prevents transient forwarding loops during convergence

### 3. Node SID Configuration

```
interface Loopback0
 address-family ipv4 unicast
  prefix-sid index 101
```

**Purpose**: Assigns a unique index to this router
**Result**: Creates a Node SID label = SRGB_START + Index (16000 + 101 = 16101)

### 4. TI-LFA (Topology Independent Loop-Free Alternate)

```
fast-reroute per-prefix
fast-reroute per-prefix ti-lfa
```

**Purpose**: Provides sub-50ms convergence during link/node failures
**How it works**:
- Pre-computes backup paths using SR
- Instantly switches to backup upon failure detection
- No need for manual configuration or additional protocols

### 5. BFD (Bidirectional Forwarding Detection)

```
bfd mode ietf
bfd address-family ipv4 timers start 180
bfd address-family ipv4 multiplier 3
bfd address-family ipv4 fast-detect
```

**Purpose**: Fast failure detection (typically < 1 second)
**Parameters**:
- `timers start 180`: Initial interval in milliseconds
- `multiplier 3`: Number of missed packets before declaring failure
- Detection time = 180ms × 3 = 540ms

### 6. VRF Configuration (PE Routers)

```
vrf CUSTOMER-A
 address-family ipv4 unicast
  import route-target 65000:100
  export route-target 65000:100
```

**Purpose**: Provides L3VPN service isolation
**Components**:
- **RD (Route Distinguisher)**: Makes customer routes unique
- **RT (Route Target)**: Controls route import/export between VRFs

### 7. BGP for VPNv4

```
router bgp 65000
 address-family vpnv4 unicast
 neighbor-group RR-CLIENTS
  address-family vpnv4 unicast
```

**Purpose**: Distributes VPN routes across the network
**Architecture**: Route Reflector design reduces iBGP full-mesh requirements

### 8. QoS Configuration

```
policy-map INGRESS-CLASSIFICATION
 class VOICE
  set mpls experimental imposition 5
  set traffic-class 5
  police rate 10 mbps
```

**Purpose**: Ensures quality of service for different traffic types
**Classes**:
- **VOICE** (EF): Real-time traffic, highest priority
- **BUSINESS-CRITICAL** (AF4x): Important data
- **BULK-DATA** (AF1x): Background traffic
- **BEST-EFFORT**: Default class

## Deployment Steps

### Phase 1: Core Router Configuration

1. **Configure SRGB and Loopback**
   ```
   segment-routing
    global-block 16000 23999
   interface Loopback0
    ipv4 address 1.1.1.1/32
   ```

2. **Configure IS-IS with SR**
   ```
   router isis CORE
    address-family ipv4 unicast
     segment-routing mpls
   ```

3. **Assign Node SID**
   ```
   interface Loopback0
    address-family ipv4 unicast
     prefix-sid index 101
   ```

4. **Configure Physical Interfaces**
   - Enable IS-IS
   - Configure TI-LFA
   - Enable BFD

### Phase 2: Aggregation Router Configuration

1. **Complete Phase 1 steps**
2. **Configure VRFs**
3. **Configure BGP**
   - Set up iBGP sessions to Route Reflectors
   - Enable VPNv4 address family
4. **Configure QoS policies**
5. **Configure customer-facing interfaces** (optional)

### Phase 3: Edge Router Configuration

1. **Complete Phase 1 and 2 steps**
2. **Configure customer interfaces**
   - Create sub-interfaces with VLANs if needed
   - Assign to appropriate VRFs
3. **Configure customer BGP sessions** (if applicable)
4. **Apply QoS and ACL policies**
5. **Configure L2VPN services** (if required)

### Phase 4: Verification and Testing

1. **Verify IS-IS adjacencies**
2. **Verify SR labels**
3. **Verify BGP sessions**
4. **Test connectivity**
5. **Verify TI-LFA backup paths**
6. **Test failover scenarios**

## Verification Commands

### IS-IS and Segment Routing

```bash
# Check IS-IS adjacencies
show isis adjacency

# Verify IS-IS database
show isis database verbose

# Check SR labels
show segment-routing mpls connected-prefix-sid-map ipv4

# Verify local SID
show isis segment-routing prefix-sid-map

# Check SR forwarding table
show mpls forwarding

# Verify SR label bindings
show segment-routing mpls lb
```

### Interface and Link Status

```bash
# Check interface status
show ipv4 interface brief

# Verify BFD sessions
show bfd session

# Check link metrics
show isis interface detail
```

### BGP and VPN

```bash
# Check BGP sessions
show bgp summary

# Verify BGP VPNv4 routes
show bgp vpnv4 unicast

# Check VRF status
show vrf all

# Verify VRF routing table
show route vrf CUSTOMER-A
```

### Traffic Engineering

```bash
# Check TI-LFA computation
show isis fast-reroute detail

# Verify backup paths
show cef X.X.X.X/32 detail

# Check SR-TE policies (if configured)
show segment-routing traffic-eng policy
```

### QoS Verification

```bash
# Check QoS policy statistics
show policy-map interface GigabitEthernet0/0/0/10

# Verify QoS configuration
show qos interface GigabitEthernet0/0/0/10 input
```

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: IS-IS Adjacency Not Coming Up

**Symptoms**: `show isis adjacency` shows no neighbors

**Possible Causes**:
- Interface not enabled for IS-IS
- NET (Network Entity Title) mismatch in area
- MTU mismatch
- Layer 2 connectivity issue

**Troubleshooting Steps**:
```bash
show isis interface detail
show isis adjacency detail
debug isis adj-packets
```

**Solution**:
- Verify interface is included in IS-IS process
- Check NET configuration matches area
- Verify MTU with: `show interface TenGigE0/0/0/0`

#### Issue 2: SR Labels Not Being Advertised

**Symptoms**: Other routers don't see your Node SID

**Possible Causes**:
- SR not enabled in IS-IS
- Prefix-SID not configured on Loopback
- SRGB mismatch

**Troubleshooting Steps**:
```bash
show isis segment-routing prefix-sid-map
show segment-routing mpls connected-prefix-sid-map ipv4
show isis database verbose | include "Prefix-SID"
```

**Solution**:
- Verify `segment-routing mpls` under IS-IS address-family
- Confirm prefix-sid index on Loopback0
- Ensure SRGB is consistent across domain

#### Issue 3: BGP VPNv4 Session Not Establishing

**Symptoms**: `show bgp vpnv4 unicast summary` shows neighbor in Idle/Active state

**Possible Causes**:
- Reachability to neighbor (via IGP)
- BGP configuration error
- Route reflector not configured correctly

**Troubleshooting Steps**:
```bash
ping X.X.X.X source Loopback0
show bgp vpnv4 unicast neighbors X.X.X.X
show route X.X.X.X
```

**Solution**:
- Verify IGP provides reachability to BGP neighbor
- Check BGP configuration (AS number, neighbor address)
- Verify update-source is set to Loopback0

#### Issue 4: TI-LFA Not Computing Backup Path

**Symptoms**: `show isis fast-reroute detail` shows no backup path

**Possible Causes**:
- Insufficient topology (no alternate path exists)
- TI-LFA not enabled on interface
- Link metrics causing suboptimal computation

**Troubleshooting Steps**:
```bash
show isis fast-reroute detail
show isis topology
show cef X.X.X.X/32 detail
```

**Solution**:
- Verify alternate physical path exists
- Confirm `fast-reroute per-prefix ti-lfa` is configured
- Review topology and adjust metrics if needed

#### Issue 5: BFD Session Not Coming Up

**Symptoms**: `show bfd session` shows no session or Down state

**Possible Causes**:
- BFD not enabled on both sides
- IP reachability issue
- Timer mismatch

**Troubleshooting Steps**:
```bash
show bfd session detail
show bfd interface
debug bfd packets
```

**Solution**:
- Enable BFD on both router interfaces
- Verify IP connectivity
- Ensure BFD parameters are compatible

## Best Practices

### 1. SRGB Planning
- **Standardize SRGB**: Use the same SRGB across all routers
- **Size appropriately**: Allocate enough labels for future growth
- **Document allocation**: Maintain a registry of SID assignments
- **Reserve ranges**: Separate Node SIDs, Adjacency SIDs, and service SIDs

### 2. Node SID Assignment
- **Use meaningful indices**: Group by router role (100s=core, 200s=agg, 300s=edge)
- **Leave gaps**: Allow for future insertions
- **Document clearly**: Maintain up-to-date network diagrams
- **Avoid reuse**: Don't reuse indices after decommissioning

### 3. IS-IS Optimization
- **Use level-2-only**: Simplifies design in flat networks
- **Enable NSR/NSF**: Provides graceful restart capabilities
- **Tune timers**: Balance convergence speed vs. stability
- **Use metric hierarchy**: Core=10, Agg=100, Edge=50

### 4. TI-LFA Configuration
- **Enable everywhere**: Maximize fast reroute coverage
- **Test failover**: Regularly verify backup paths work
- **Monitor convergence**: Track failover times
- **Document topology**: Understand backup path logic

### 5. BFD Deployment
- **Start conservative**: Use 300ms initially, tune down gradually
- **Match on both sides**: Ensure timer compatibility
- **Monitor CPU impact**: BFD has processing overhead
- **Use on critical links**: Prioritize core and uplinks

### 6. VRF and Service Design
- **Plan RT scheme**: Use hierarchical route targets
- **Limit VRF count**: Each VRF consumes resources
- **Use route policies**: Control route import/export precisely
- **Implement security**: Use ACLs and route filtering

### 7. QoS Strategy
- **Start simple**: Implement basic classification first
- **Align with SLA**: Match QoS classes to service agreements
- **Test thoroughly**: Verify marking and queuing behavior
- **Monitor utilization**: Track per-class statistics

### 8. Change Management
- **Test in lab**: Validate changes before production
- **Change during maintenance**: Schedule disruptive changes
- **Implement incrementally**: One router/link at a time
- **Have rollback plan**: Document revert procedures

### 9. Monitoring and Operations
- **Implement telemetry**: Use model-driven telemetry for SR metrics
- **Set up alerts**: Monitor IS-IS, BGP, BFD status
- **Track utilization**: Monitor link and label usage
- **Regular audits**: Verify configuration consistency

### 10. Security Considerations
- **Protect control plane**: Use control-plane policing
- **Secure management**: Use SSH, disable telnet
- **Implement ACLs**: Filter at service boundaries
- **Log events**: Enable comprehensive logging
- **Regular updates**: Keep IOS XR software current

## Advanced Topics

### SR Traffic Engineering (SR-TE)

SR-TE enables explicit path control without RSVP-TE:

```
segment-routing
 traffic-eng
  interface TenGigE0/0/0/0
  !
  segment-list SL-PATH-1
   index 1 address ipv4 2.2.2.2
   index 2 address ipv4 10.2.2.2
  !
  policy POL-TO-EDGE-2
   color 100 end-point ipv4 20.2.2.2
   candidate-paths
    preference 100
     explicit segment-list SL-PATH-1
    !
   !
  !
 !
!
```

### On-Demand Next-Hop (ODN)

ODN automates SR-TE policy creation based on BGP color community:

```
segment-routing
 traffic-eng
  on-demand color 100
   dynamic
    metric
     type te
    !
   !
  !
 !
!

route-policy ODN-COLOR-100
 set extcommunity color 100
end-policy

router bgp 65000
 vrf CUSTOMER-A
  neighbor 192.168.100.2
   address-family ipv4 unicast
    route-policy ODN-COLOR-100 out
```

### Anycast SID

Anycast SIDs enable service redundancy:

```
interface Loopback100
 ipv4 address 100.100.100.100/32

router isis CORE
 interface Loopback100
  passive
  address-family ipv4 unicast
   prefix-sid index 900
```

Configure the same index 900 on multiple routers for load balancing and redundancy.

### SR Performance Measurement

Monitor latency and performance:

```
performance-measurement
 delay-measurement
  advertisement
   accelerated
   periodic
    interval 30
    threshold 10
   !
  !
 !
!
```

## Conclusion

This configuration guide provides a comprehensive foundation for deploying Segment Routing in a Service Provider network using Cisco IOS XR. By following the templates, best practices, and troubleshooting guidance, you can build a scalable, resilient, and efficient SR-enabled network.

For additional information, consult:
- Cisco IOS XR Segment Routing Configuration Guide
- RFC 8402: Segment Routing Architecture
- RFC 8660: Segment Routing with MPLS Data Plane
- Cisco Live presentations on Segment Routing

## Appendix: Quick Reference

### Key Configuration Snippets

**Enable SR Globally**:
```
segment-routing
 global-block 16000 23999
```

**Enable SR in IS-IS**:
```
router isis CORE
 address-family ipv4 unicast
  segment-routing mpls
```

**Configure Node SID**:
```
interface Loopback0
 address-family ipv4 unicast
  prefix-sid index 101
```

**Enable TI-LFA**:
```
interface TenGigE0/0/0/0
 address-family ipv4 unicast
  fast-reroute per-prefix ti-lfa
```

**Essential Show Commands**:
```
show isis adjacency
show segment-routing mpls connected-prefix-sid-map ipv4
show mpls forwarding
show bgp vpnv4 unicast summary
show bfd session
show isis fast-reroute detail
```
