# Scenario 2: Two Datacenters (IOC & ADC) with High Availability

## 📋 Overview

This deployment scenario implements Red Hat Ansible Automation Platform 2.5 across **two datacenters (IOC & ADC)** with full high availability and datacenter failure tolerance. The architecture ensures the AAP environment remains available even if an entire datacenter becomes unavailable.

## 🎯 Key Requirements

- ✅ **Two Datacenters**: AAP components deployed across IOC and ADC datacenters
- ✅ **Datacenter Failover**: Complete tolerance of single datacenter failure
- ✅ **Cross-DC High Availability**: Database replication and traffic failover
- ✅ **Separate Execution Nodes**: Dedicated execution nodes per target environment:
  - IOC Datacenter → IOC execution nodes → IOC clients
  - ADC Datacenter → ADC execution nodes → ADC clients
  - Azure Cloud → Separate Azure execution nodes (one per DC)

## 🏗️ Architecture Components

### Global Layer

#### **Global Load Balancer (GSLB)**
- **DNS-based** traffic management
- **Primary**: Routes to IOC datacenter
- **Secondary**: Automatically fails over to ADC datacenter
- **Health Checks**: Continuous monitoring of both datacenters
- **Failover Time**: < 30 seconds (DNS TTL dependent)

### IOC Datacenter (Primary)

#### 1. **Load Balancer**
- **LB**: `lb.ioc.example.com`
- **Purpose**: Distributes traffic to IOC Gateway nodes

#### 2. **Automation Gateway Cluster**
- **Gateway 1**: `gateway1.ioc.example.com` (Port 443)
- **Gateway 2**: `gateway2.ioc.example.com` (Port 443)
- **Configuration**: Active-Active within datacenter

#### 3. **Automation Controller Cluster**
- **Controller 1**: `controller1.ioc.example.com` (Ports 443, 80)
- **Controller 2**: `controller2.ioc.example.com` (Ports 443, 80)
- **Configuration**: Active-Active, handles IOC workloads primarily

#### 4. **Automation Hub Cluster**
- **Hub 1**: `hub1.ioc.example.com` (Ports 443, 80)
- **Hub 2**: `hub2.ioc.example.com` (Ports 443, 80)
- **Configuration**: Active-Active, syncs with ADC

#### 5. **Event-Driven Ansible**
- **EDA 1**: `eda1.ioc.example.com` (Port 443)
- **Configuration**: Active in IOC, standby in ADC

#### 6. **Database Cluster (Primary)**
- **Primary DB**: `db1.ioc.example.com` (Port 5432)
- **Redis**: `redis1.ioc.example.com` (Port 6379)
- **Replication**: Async replication to ADC

#### 7. **IOC Execution Nodes**
- **IOC Exec 1**: `ioc-exec1.ioc.example.com` (Port 27199)
- **IOC Exec 2**: `ioc-exec2.ioc.example.com` (Port 27199)
- **Purpose**: Dedicated to IOC datacenter clients
- **Advantage**: Low latency to IOC targets

### ADC Datacenter (Secondary)

#### 1. **Load Balancer**
- **LB**: `lb.adc.example.com`
- **Purpose**: Distributes traffic to ADC Gateway nodes

#### 2. **Automation Gateway Cluster**
- **Gateway 1**: `gateway1.adc.example.com` (Port 443)
- **Gateway 2**: `gateway2.adc.example.com` (Port 443)
- **Configuration**: Active-Active within datacenter

#### 3. **Automation Controller Cluster**
- **Controller 1**: `controller1.adc.example.com` (Ports 443, 80)
- **Controller 2**: `controller2.adc.example.com` (Ports 443, 80)
- **Configuration**: Active-Active, handles ADC workloads primarily

#### 4. **Automation Hub Cluster**
- **Hub 1**: `hub1.adc.example.com` (Ports 443, 80)
- **Hub 2**: `hub2.adc.example.com` (Ports 443, 80)
- **Configuration**: Active-Active, syncs with IOC

#### 5. **Event-Driven Ansible**
- **EDA 1**: `eda1.adc.example.com` (Port 443)
- **Configuration**: Standby, activates on IOC failure

#### 6. **Database Cluster (Standby)**
- **Standby DB**: `db1.adc.example.com` (Port 5432)
- **Redis**: `redis1.adc.example.com` (Port 6379)
- **Replication**: Receives from IOC, can be promoted

#### 7. **ADC Execution Nodes**
- **ADC Exec 1**: `adc-exec1.adc.example.com` (Port 27199)
- **ADC Exec 2**: `adc-exec2.adc.example.com` (Port 27199)
- **Purpose**: Dedicated to ADC datacenter clients
- **Advantage**: Low latency to ADC targets

### Azure Execution Nodes

#### **Distributed Azure Nodes**
- **IOC-based**: `azure-exec1.ioc.example.com`
  - Managed by IOC controllers
  - ExpressRoute/VPN to Azure from IOC
- **ADC-based**: `azure-exec2.adc.example.com`
  - Managed by ADC controllers
  - ExpressRoute/VPN to Azure from ADC
- **Purpose**: Workload-based routing to Azure
- **Configuration**: Both active, load distributed

## 🌐 Target Environment Connectivity

### Separate Execution Node Strategy

This scenario uses **dedicated execution nodes** per target environment for optimal performance:

#### 1. **IOC Datacenter Clients**
- **Execution Nodes**: IOC Exec 1-2
- **Connectivity**: Direct, same datacenter
- **Latency**: < 5ms
- **Advantage**: Optimal performance, no WAN traversal

#### 2. **ADC Datacenter Clients**
- **Execution Nodes**: ADC Exec 1-2
- **Connectivity**: Direct, same datacenter
- **Latency**: < 5ms
- **Advantage**: Optimal performance, no WAN traversal

#### 3. **Azure Cloud Clients**
- **Execution Nodes**: Azure Exec 1-2 (distributed)
- **Connectivity**: ExpressRoute or Site-to-Site VPN
- **Latency**: 10-100ms (variable)
- **Advantage**: Redundant paths from both datacenters

### Workload Type Separation

Execution nodes can be further segmented by workload type:

- **Database workloads** → Specific exec nodes with DB clients
- **Network equipment** → Exec nodes in management VLAN
- **Application servers** → General purpose exec nodes
- **Cloud resources** → Exec nodes with cloud SDK/CLI

## 🔄 Cross-Datacenter Replication

### Database Replication Architecture

#### **PostgreSQL Async Replication**
- **Method**: Streaming replication with WAL archiving
- **Direction**: IOC (Primary) → ADC (Standby)
- **RPO**: < 30 seconds (configurable)
- **RTO**: < 5 minutes for database promotion

#### **Promotion Process**
1. Detect IOC datacenter failure
2. Stop applications in ADC (if not already stopped)
3. Promote ADC database to primary
4. Update GSLB to route traffic to ADC
5. Start/resume ADC services
6. Point all controllers to ADC database

#### **Redis Replication**
- **Method**: Master-replica replication
- **Direction**: IOC → ADC
- **Purpose**: Session cache, task queue
- **Note**: Some data loss acceptable (caching layer)

### Content Synchronization

#### **Hub Content Sync**
- Both IOC and ADC hubs sync from same Git repositories
- Database stores collection metadata
- Automatic sync on database replication
- Manual re-sync available if needed

## 📊 Network Requirements

### Inter-Datacenter Connectivity

| Type | Bandwidth | Latency | Redundancy |
|------|-----------|---------|------------|
| Database Replication | 100 Mbps+ | < 50ms | Dual paths |
| Management Traffic | 50 Mbps+ | < 100ms | Dual paths |
| Monitoring | 10 Mbps+ | < 100ms | Single path OK |

### Firewall Rules

#### IOC → ADC
- PostgreSQL: IOC DB → ADC DB (Port 5432)
- Redis: IOC Redis → ADC Redis (Port 6379)
- Monitoring: Both directions (Ports 9090, 9100)

#### ADC → IOC
- Same as above (bidirectional for monitoring)

#### Execution Nodes → Targets
- SSH: Exec nodes → All Linux targets (Port 22)
- WinRM: Exec nodes → All Windows targets (Ports 5985, 5986)
- HTTPS: Exec nodes → Cloud APIs (Port 443)

## 🛡️ Datacenter Failure Scenarios

### Scenario 1: IOC Datacenter Failure

**Impact**: IOC unavailable, all IOC components down

**Automatic Actions**:
1. GSLB detects IOC failure (health checks fail)
2. GSLB routes all traffic to ADC
3. Users connect to ADC Gateway → ADC Controllers
4. ADC database promoted to primary (if auto-failover enabled)

**Manual Actions**:
1. Verify database promotion completed
2. Update DNS if needed (GSLB should handle)
3. Monitor ADC capacity (handling all traffic)
4. Consider scaling ADC temporarily

**Jobs Impact**:
- **IOC target jobs**: Will fail (IOC exec nodes down)
- **ADC target jobs**: Continue normally (ADC exec nodes active)
- **Azure target jobs**: Continue via ADC-based Azure exec node

**Recovery**:
- When IOC returns, database replication reverses (ADC → IOC)
- Or rebuild IOC as new standby
- GSLB can be set to failback or stay on ADC

### Scenario 2: ADC Datacenter Failure

**Impact**: ADC unavailable, all ADC components down

**Automatic Actions**:
1. IOC continues serving traffic (already primary)
2. Database replication stops (no standby)
3. No user-facing impact if GSLB already points to IOC

**Jobs Impact**:
- **IOC target jobs**: Continue normally
- **ADC target jobs**: Will fail (ADC exec nodes down)
- **Azure target jobs**: Continue via IOC-based Azure exec node

**Recovery**:
- Restore ADC services
- Re-establish database replication
- Resume ADC execution nodes

### Scenario 3: WAN Link Failure (Split Brain Risk)

**Impact**: Both datacenters operating independently

**Risk**: Split brain - both DBs accepting writes

**Prevention**:
1. **Witness/Quorum Server**: Third site or cloud witness
2. **Fencing**: Automatic shutdown of standby on split
3. **STONITH**: Shoot The Other Node In The Head
4. **Manual Verification**: Operator confirms primary

**Recommended**: Use Patroni or pg_auto_failover with proper fencing

## 📈 Capacity Planning

### Per Datacenter Resources

| Component | Nodes/DC | CPU/Node | RAM/Node | Disk/Node |
|-----------|----------|----------|----------|-----------|
| Load Balancer | 1 | 2 cores | 4 GB | 20 GB |
| Gateway | 2 | 4 cores | 8 GB | 40 GB |
| Controller | 2 | 8 cores | 16 GB | 60 GB |
| Hub | 2 | 4 cores | 8 GB | 100 GB |
| EDA | 1 | 4 cores | 8 GB | 40 GB |
| Database | 1 | 8 cores | 32 GB | 200 GB+ |
| Redis | 1 | 2 cores | 8 GB | 20 GB |
| Exec Nodes (DC) | 2 | 4 cores | 8 GB | 40 GB |
| Exec Nodes (Azure) | 1 | 4 cores | 8 GB | 40 GB |

**Total Per DC**: 13 nodes, 62 cores, 180 GB RAM  
**Total Both DCs**: 26 nodes, 124 cores, 360 GB RAM

### Scaling Considerations
- Each datacenter should handle 100% of load during single DC failure
- Plan for 50% normal load per DC, 100% failover capacity
- Scale execution nodes based on job concurrency needs

## ✅ Testing Checklist

### High Availability Testing
- [ ] IOC datacenter failure (planned)
- [ ] ADC datacenter failure (planned)
- [ ] WAN link failure (test split brain prevention)
- [ ] Database failover (IOC to ADC)
- [ ] Database failback (ADC to IOC)
- [ ] GSLB failover (DNS propagation time)
- [ ] Individual component failures in each DC

### Performance Testing
- [ ] Network latency between DCs
- [ ] Database replication lag under load
- [ ] Job execution from IOC exec nodes to IOC targets
- [ ] Job execution from ADC exec nodes to ADC targets
- [ ] Job execution from Azure exec nodes to Azure targets

## 🔒 Security Considerations

### Network Segmentation
- Separate VLANs per datacenter
- Firewall rules between datacenters
- Encrypted replication (TLS for PostgreSQL)
- VPN tunnels for exec nodes to remote targets

### Access Control
- Separate admin credentials per environment
- MFA required for production access
- Audit logging enabled on all components

## 📚 Additional Resources

- [Red Hat AAP Geographic Resilience](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [PostgreSQL Replication](https://www.postgresql.org/docs/current/runtime-config-replication.html)
- [Patroni for PostgreSQL HA](https://patroni.readthedocs.io/)

---

**Deployment Complexity**: High  
**Recommended For**: Mission-critical environments requiring datacenter-level resilience  
**HA Level**: Geographic redundancy with datacenter failure tolerance
