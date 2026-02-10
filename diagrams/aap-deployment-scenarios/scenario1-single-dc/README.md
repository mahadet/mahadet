# Scenario 1: Single Datacenter (IOC) with High Availability

## 📋 Overview

This deployment scenario implements Red Hat Ansible Automation Platform 2.5 in a **single datacenter (IOC)** with full high availability configuration. The architecture ensures the AAP environment remains available during scheduled maintenance and component failures.

## 🎯 Key Requirements

- ✅ **Single Datacenter**: All AAP components deployed in IOC datacenter
- ✅ **High Availability**: Clustered components with automatic failover
- ✅ **Maintenance Tolerance**: Zero downtime during maintenance windows
- ✅ **Common Execution Nodes**: Single execution node pool serving all target environments:
  - IOC Datacenter clients
  - ADC Datacenter clients
  - Azure Cloud clients

## 🏗️ Architecture Components

### High Availability Configuration

#### 1. **Load Balancer (HA Pair)**
- **Primary LB**: `lb1.ioc.example.com`
- **Secondary LB**: `lb2.ioc.example.com`
- **Configuration**: Active-Standby with VRRP/keepalived
- **Purpose**: Entry point for all external traffic with automatic failover

#### 2. **Automation Gateway Cluster (Active-Active)**
- **Node 1**: `gateway1.ioc.example.com` (Port 443)
- **Node 2**: `gateway2.ioc.example.com` (Port 443)
- **Configuration**: Both nodes actively serve traffic
- **Load Distribution**: Round-robin via load balancer
- **Failover**: Automatic, sub-second

#### 3. **Automation Controller Cluster (Active-Active)**
- **Node 1**: `controller1.ioc.example.com` (Ports 443, 80)
- **Node 2**: `controller2.ioc.example.com` (Ports 443, 80)
- **Node 3**: `controller3.ioc.example.com` (Ports 443, 80)
- **Configuration**: All nodes active, shared workload
- **Minimum**: 3 nodes for quorum-based operations
- **Session Persistence**: Stored in shared database

#### 4. **Automation Hub Cluster (Active-Active)**
- **Node 1**: `hub1.ioc.example.com` (Ports 443, 80)
- **Node 2**: `hub2.ioc.example.com` (Ports 443, 80)
- **Configuration**: Both nodes serve content
- **Content Sync**: Automatic via shared database

#### 5. **Event-Driven Ansible (Active-Passive)**
- **Primary**: `eda1.ioc.example.com` (Port 443)
- **Secondary**: `eda2.ioc.example.com` (Port 443)
- **Configuration**: Active-Passive with heartbeat
- **Failover**: Automatic promotion of secondary

#### 6. **Database Cluster (PostgreSQL HA)**
- **Primary**: `db1.ioc.example.com` (Port 5432)
- **Standby**: `db2.ioc.example.com` (Port 5432)
- **Configuration**: PostgreSQL streaming replication
- **Failover**: Automatic with pg_auto_failover or Patroni
- **Backup**: Continuous WAL archiving

#### 7. **Redis Cache (Master-Replica)**
- **Master**: `redis1.ioc.example.com` (Port 6379)
- **Replica**: `redis2.ioc.example.com` (Port 6379)
- **Configuration**: Master-replica replication
- **Failover**: Redis Sentinel for automatic promotion

#### 8. **Common Execution Node Pool**
- **Node 1**: `exec1.ioc.example.com` (Port 27199)
- **Node 2**: `exec2.ioc.example.com` (Port 27199)
- **Node 3**: `exec3.ioc.example.com` (Port 27199)
- **Node 4**: `exec4.ioc.example.com` (Port 27199)
- **Configuration**: Shared pool via Automation Mesh
- **Purpose**: Execute automation jobs for all target environments

## 🌐 Target Environment Connectivity

### Common Execution Node Pool Strategy

The execution nodes in this scenario form a **common pool** that serves all target environments:

1. **IOC Datacenter Clients**
   - Direct network connectivity
   - Low latency (< 5ms typical)
   - Same datacenter as AAP

2. **ADC Datacenter Clients**
   - WAN connectivity (IOC ↔ ADC)
   - Medium latency (5-50ms typical)
   - Requires firewall rules for ansible management

3. **Azure Cloud Clients**
   - Site-to-Site VPN or ExpressRoute
   - Variable latency (10-100ms typical)
   - Secure tunnel from IOC to Azure

### Network Requirements

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Users | Load Balancer | 443 | HTTPS | Web UI/API access |
| Load Balancer | Gateway Nodes | 443 | HTTPS | API routing |
| Gateway | Controller | 443 | HTTPS | Control plane |
| Gateway | Hub | 443 | HTTPS | Content access |
| Gateway | EDA | 443 | HTTPS | Event processing |
| Controller | Database | 5432 | PostgreSQL | Data persistence |
| Controller | Redis | 6379 | Redis | Task queue/cache |
| Controller | Exec Nodes | 27199 | Automation Mesh | Job execution |
| Hub | Database | 5432 | PostgreSQL | Content metadata |
| EDA | Database | 5432 | PostgreSQL | Event data |
| Exec Nodes | IOC Targets | 22/5986 | SSH/WinRM | Automation |
| Exec Nodes | ADC Targets | 22/5986 | SSH/WinRM | Automation |
| Exec Nodes | Azure Targets | 22/5986 | SSH/WinRM | Automation |

## 🔧 Maintenance Strategy

### Zero-Downtime Maintenance

#### Gateway Maintenance
1. Remove one Gateway node from load balancer pool
2. Perform maintenance on offline node
3. Return node to pool
4. Repeat for other Gateway nodes

#### Controller Maintenance
1. Mark Controller node for maintenance (no new jobs)
2. Wait for running jobs to complete
3. Perform maintenance
4. Return node to active state
5. Repeat for other Controller nodes (maintain quorum)

#### Hub Maintenance
1. Remove Hub node from load balancer
2. Perform maintenance
3. Re-sync content if needed
4. Return to pool

#### Database Maintenance
1. Perform maintenance on standby first
2. Test standby integrity
3. Failover to standby (now primary)
4. Perform maintenance on old primary
5. Re-establish replication

#### Execution Node Maintenance
1. Mark node as unavailable in Controller
2. Wait for running jobs to complete
3. Perform maintenance
4. Mark node as available
5. Repeat for other nodes

## 📊 Capacity Planning

### Minimum Resources (Per Component)

| Component | Nodes | CPU/Node | RAM/Node | Disk/Node |
|-----------|-------|----------|----------|-----------|
| Load Balancer | 2 | 2 cores | 4 GB | 20 GB |
| Gateway | 2 | 4 cores | 8 GB | 40 GB |
| Controller | 3 | 8 cores | 16 GB | 60 GB |
| Hub | 2 | 4 cores | 8 GB | 100 GB |
| EDA | 2 | 4 cores | 8 GB | 40 GB |
| Database | 2 | 8 cores | 32 GB | 200 GB+ |
| Redis | 2 | 2 cores | 8 GB | 20 GB |
| Execution Nodes | 4 | 4 cores | 8 GB | 40 GB |

**Total Minimum**: 19 nodes, 110 cores, 288 GB RAM

### Scaling Recommendations

- **More Execution Nodes**: Add for higher job concurrency
- **More Controller Nodes**: Add for higher API load
- **Larger Database**: Scale based on inventory size and job history retention

## ✅ High Availability Validation

### Failover Testing Checklist

- [ ] Load Balancer failover (primary to secondary)
- [ ] Gateway node failure (automatic removal from pool)
- [ ] Controller node failure (jobs continue on remaining nodes)
- [ ] Hub node failure (content remains accessible)
- [ ] EDA failover (secondary takes over)
- [ ] Database failover (standby promotion)
- [ ] Redis failover (replica promotion)
- [ ] Execution node failure (jobs reschedule to other nodes)

### Expected Recovery Times

| Component | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|-----------|-------------------------------|--------------------------------|
| Load Balancer | < 5 seconds | 0 (no data loss) |
| Gateway | < 10 seconds | 0 (stateless) |
| Controller | < 30 seconds | 0 (shared DB) |
| Hub | < 30 seconds | 0 (shared DB) |
| EDA | < 60 seconds | < 1 minute |
| Database | < 2 minutes | < 30 seconds |
| Redis | < 30 seconds | < 5 seconds |
| Execution Node | < 1 minute | Current job only |

## 🔒 Security Considerations

### Network Segmentation
- **DMZ**: Load Balancers
- **Application Tier**: Gateway, Controller, Hub, EDA
- **Data Tier**: Database, Redis (isolated, no direct external access)
- **Execution Tier**: Execution nodes (controlled outbound only)

### Firewall Rules
- Restrict database access to AAP components only
- Limit execution node access to managed targets only
- Use bastion hosts for administrative access

## 📈 Monitoring and Alerting

### Key Metrics to Monitor
- Load Balancer health and failover status
- Gateway/Controller/Hub node availability
- Database replication lag
- Redis replication status
- Execution node capacity and job queue depth
- Job success/failure rates
- Network latency to remote targets

### Recommended Tools
- Prometheus + Grafana for metrics
- AAP built-in analytics
- Database monitoring (pg_stat_activity)
- Network monitoring for remote site connectivity

## 📚 Additional Resources

- [Red Hat AAP 2.5 Documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [PostgreSQL High Availability](https://www.postgresql.org/docs/current/high-availability.html)
- [Automation Mesh Architecture](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/automation_mesh)

---

**Deployment Complexity**: Medium  
**Recommended For**: Organizations with maintenance windows, single datacenter constraint  
**HA Level**: Component-level high availability within single datacenter
