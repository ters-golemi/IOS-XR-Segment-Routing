# Quick Start Guide - Segment Routing Configuration

This guide provides a fast-track approach to deploying the Segment Routing templates in your network.

## Prerequisites Checklist

- [ ] IOS XR 6.0+ routers (7.x recommended)
- [ ] Physical connectivity established
- [ ] IP addressing plan finalized
- [ ] SRGB range defined (default: 16000-23999)
- [ ] Node SID indices assigned

## 5-Step Deployment Process

### Step 1: Plan Your Network (15 minutes)

1. **Review the topology**: Read [`TOPOLOGY.md`](TOPOLOGY.md)
2. **Assign Node SIDs**:
   - Core routers: 101-199
   - Aggregation routers: 201-299
   - Edge routers: 301-399
3. **Document IP addresses**:
   - Loopback addresses for each router
   - Point-to-point link addresses
   - ISIS NET addresses

### Step 2: Prepare Configuration Files (30 minutes)

#### For Each Core Router:

1. Copy `configs/core-router-template.txt`
2. Replace placeholders:
   ```
   hostname CORE-X        → hostname CORE-1
   X.X.X.X               → 1.1.1.1 (loopback)
   prefix-sid index XXX  → prefix-sid index 101
   net 49.0001.XXXX...   → net 49.0001.0001.0001.0001.00
   ```
3. Update interface configurations with actual interface names and IPs

#### For Each Aggregation Router:

1. Copy `configs/aggregation-router-template.txt`
2. Replace placeholders:
   ```
   hostname AGG-X        → hostname AGG-1
   X.X.X.X              → 10.1.1.1 (loopback)
   prefix-sid index XXX → prefix-sid index 201
   net 49.0001.XXXX...  → net 49.0001.0010.0001.0001.00
   ```
3. Configure VRFs for customer services
4. Update BGP neighbor addresses

#### For Each Edge Router:

1. Copy `configs/edge-router-template.txt`
2. Replace placeholders (similar to aggregation)
3. Configure customer-facing interfaces
4. Set up route policies for customer BGP sessions

### Step 3: Deploy Core Routers (1 hour)

**Deploy in this order**: CORE-1 → CORE-2 → CORE-3

For each router:

```bash
# 1. Connect to router console
ssh admin@router-ip

# 2. Enter configuration mode
configure

# 3. Paste prepared configuration
# (copy-paste from your prepared file)

# 4. Commit configuration
commit

# 5. Verify basic connectivity
exit
show ipv4 interface brief
show isis adjacency
show segment-routing mpls connected-prefix-sid-map ipv4
```

**Expected results after each core router**:
- After CORE-1: No ISIS adjacencies (normal - only one router)
- After CORE-2: ISIS adjacency to CORE-1
- After CORE-3: ISIS adjacencies to CORE-1 and CORE-2

### Step 4: Deploy Aggregation Routers (1 hour)

**Deploy in this order**: AGG-1 → AGG-2 → AGG-3 → AGG-4

For each router:

```bash
# 1. Deploy configuration (same as Step 3)
configure
# [paste configuration]
commit

# 2. Verify IS-IS
show isis adjacency
# Should see adjacencies to core routers

# 3. Verify SR labels
show segment-routing mpls lb

# 4. Verify BGP (if route reflectors configured)
show bgp summary
```

**Expected results**:
- ISIS adjacencies to core routers
- SR labels learned from all routers in domain
- BGP sessions to route reflectors (CORE-1, CORE-2)

### Step 5: Deploy Edge Routers (1 hour)

**Deploy in this order**: EDGE-1 → EDGE-2 → EDGE-3 → EDGE-4 → EDGE-5

For each router:

```bash
# 1. Deploy configuration
configure
# [paste configuration]
commit

# 2. Verify uplinks
show isis adjacency
# Should see adjacencies to aggregation routers

# 3. Verify SR forwarding
show cef <destination-prefix> detail

# 4. Verify VRF configuration
show vrf all

# 5. Test customer connectivity
# (when customers connected)
```

## Verification Checklist

After deployment, verify these on ALL routers:

### IS-IS Verification
```bash
# Should see expected adjacencies
show isis adjacency

# Should see all Node SIDs in the network
show isis database verbose | include "Prefix-SID"

# Should show no errors
show isis neighbors detail
```

### Segment Routing Verification
```bash
# Should show all routers in the domain
show segment-routing mpls connected-prefix-sid-map ipv4

# Should show SR labels in forwarding table
show mpls forwarding

# Should show proper label programming
show cef <remote-loopback> detail
```

### BGP Verification (PE Routers)
```bash
# All sessions should be in "Established" state
show bgp summary

# Should see VPNv4 routes
show bgp vpnv4 unicast

# VRF routing tables should be populated
show route vrf CUSTOMER-A
```

### High Availability Verification
```bash
# All BFD sessions should be "Up"
show bfd session

# TI-LFA should compute backup paths
show isis fast-reroute detail

# Should show backup paths
show cef <prefix> detail
```

## Common First-Time Issues

### Issue 1: ISIS Adjacency Not Coming Up
```bash
# Check interface status
show ipv4 interface brief

# Verify ISIS configuration
show run router isis

# Check for MTU mismatch
show interfaces TenGigE0/0/0/0

# Solution: Ensure ISIS enabled on interface and NET configured
```

### Issue 2: No SR Labels Visible
```bash
# Verify SR is enabled
show run segment-routing

# Check ISIS SR configuration
show run router isis | include segment

# Solution: Ensure 'segment-routing mpls' under ISIS address-family
```

### Issue 3: BGP Sessions Not Establishing
```bash
# Check reachability via IGP
ping <neighbor-loopback> source Loopback0

# Verify BGP configuration
show run router bgp

# Check BGP neighbors
show bgp neighbors <ip> | include state

# Solution: Ensure IGP provides reachability to BGP neighbor
```

## Testing Failover

After successful deployment, test failover:

### Test 1: Link Failure
```bash
# On any router, shut down an interface
configure
interface TenGigE0/0/0/0
 shutdown
commit

# On other routers, verify fast convergence
show isis fast-reroute detail
show cef <affected-prefix> detail

# Re-enable interface
configure
interface TenGigE0/0/0/0
 no shutdown
commit
```

### Test 2: Node Failure
```bash
# Simulate by powering off a non-critical router
# Verify traffic flows via alternate paths
traceroute <destination> source <source>

# Verify TI-LFA activated backup paths
show cef <prefix> detail
```

## Useful Commands Reference

### Show Commands
```bash
# General Status
show version brief
show redundancy summary
show platform

# ISIS
show isis adjacency
show isis interface
show isis database verbose
show isis topology

# Segment Routing
show segment-routing mpls connected-prefix-sid-map ipv4
show segment-routing mpls lb
show mpls forwarding

# BGP
show bgp summary
show bgp vpnv4 unicast
show bgp neighbors
show bgp vpnv4 unicast vrf CUSTOMER-A

# Interfaces
show ipv4 interface brief
show interfaces TenGigE0/0/0/0
show controllers TenGigE0/0/0/0 phy

# High Availability
show bfd session
show isis fast-reroute detail
show cef <prefix> detail

# QoS
show policy-map interface <interface>
show qos interface <interface> input
```

### Debug Commands (Use with caution in production)
```bash
# ISIS Debugging
debug isis adj-packets
debug isis spf-events
debug isis segment-routing

# BGP Debugging  
debug bgp updates
debug bgp events

# BFD Debugging
debug bfd packets
```

### Configuration Rollback
```bash
# If something goes wrong, rollback
show configuration history

# Rollback to previous config
rollback configuration last 1

# Or load a backup
load configuration <backup-file>
commit
```

## Next Steps

1. **Configure customer services**: Add specific VRFs and BGP sessions
2. **Implement SR-TE policies**: For traffic engineering
3. **Enable telemetry**: For monitoring and analytics
4. **Set up automation**: Use NETCONF/RESTCONF for configuration management
5. **Review security**: Implement control-plane policing and ACLs

## Support Resources

- **Full Documentation**: [`CONFIGURATION-GUIDE.md`](CONFIGURATION-GUIDE.md)
- **Topology Details**: [`TOPOLOGY.md`](TOPOLOGY.md)
- **Cisco IOS XR Documentation**: [cisco.com/c/en/us/support/ios-nx-os-software/ios-xr-software](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-xr-software/)
- **RFC 8402**: Segment Routing Architecture
- **RFC 8660**: Segment Routing with MPLS Data Plane

## Time Estimates

| Task | Estimated Time |
|------|---------------|
| Planning and preparation | 1-2 hours |
| Core router deployment (3 routers) | 1 hour |
| Aggregation deployment (4 routers) | 1.5 hours |
| Edge deployment (5 routers) | 2 hours |
| Verification and testing | 2 hours |
| **Total** | **7.5-8.5 hours** |

*Times are approximate and assume familiarity with IOS XR*

## Success Criteria

Your deployment is successful when:

- All ISIS adjacencies are up
- All routers see all Node SIDs in SR database
- BGP sessions are established
- Traffic flows through the network
- TI-LFA provides backup paths
- BFD sessions are operational
- Failover tests complete successfully
- Customer services are working

---

**Ready to deploy?** Start with Step 1 above and work through each step systematically. Remember to verify at each stage before proceeding to the next!
