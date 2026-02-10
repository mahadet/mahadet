# Scenario 3: Separate Non-Prod & Prod Environments in Two Datacenters

## 📋 Overview

This deployment scenario implements **completely separate Production and Non-Production environments** of Red Hat Ansible Automation Platform 2.5, each deployed across **two datacenters (IOC & ADC)** with full high availability and datacenter failure tolerance. This provides complete isolation between environments while maintaining geographic resilience.

## 🎯 Key Requirements

- ✅ **Environment Isolation**: Completely separate Prod and Non-Prod AAP environments
- ✅ **Two Datacenters Each**: Both environments span IOC and ADC datacenters
- ✅ **Datacenter Failover**: Each environment tolerates single datacenter failure
- ✅ **Separate Execution Nodes**: Dedicated execution nodes per environment, datacenter, and workload type:
  - Production IOC nodes → Prod IOC clients
  - Production ADC nodes → Prod ADC clients
  - Production Azure nodes → Prod Azure resources
  - Non-Prod IOC nodes → Non-Prod IOC clients
  - Non-Prod ADC nodes → Non-Prod ADC clients
  - Non-Prod Azure nodes → Non-Prod Azure resources

## 🏗️ Architecture Overview

### Environment Separation

This scenario provides **complete isolation** between Production and Non-Production:

| Aspect | Production | Non-Production |
|--------|-----------|----------------|
| **AAP Infrastructure** | Separate | Separate |
| **Databases** | Separate | Separate |
| **Networks** | Separate VLANs | Separate VLANs |
| **Execution Nodes** | Dedicated Prod nodes | Dedicated Non-Prod nodes |
| **Target Hosts** | Prod workloads only | Non-Prod/Dev/Test only |
| **Access Control** | Prod admins/users | Non-Prod users |
| **Change Control** | Strict CAB process | Flexible, rapid changes |

### Why Separate Environments?

1. **Risk Isolation**: Non-Prod changes cannot impact Production
2. **Testing**: Validate AAP upgrades in Non-Prod first
3. **Compliance**: Meet regulatory requirements for Prod separation
4. **Performance**: Dedicated resources prevent resource contention
5. **Security**: Different access controls and audit requirements

## 🏗️ Production Environment Architecture

### Production - Global Layer

#### **Production Global Load Balancer**
- **Endpoint**: `aap-prod.example.com`
- **Primary Route**: IOC Datacenter
- **Failover Route**: ADC Datacenter
- **Purpose**: Production traffic only

### Production - IOC Datacenter

#### Components
- **Load Balancer**: `prod-lb.ioc.example.com`
- **Gateway Cluster**: 
  - `prod-gw1.ioc.example.com`
  - `prod-gw2.ioc.example.com`
- **Controller Cluster**:
  - `prod-ctrl1.ioc.example.com`
  - `prod-ctrl2.ioc.example.com`
- **Hub**: `prod-hub1.ioc.example.com`
- **EDA**: `prod-eda.ioc.example.com`
- **Database**: 
  - Primary: `prod-db1.ioc.example.com`
  - Redis: `prod-redis1.ioc.example.com`
- **Execution Nodes** (IOC Workloads):
  - `prod-ioc-exec1.ioc.example.com`
  - `prod-ioc-exec2.ioc.example.com`

### Production - ADC Datacenter

#### Components
- **Load Balancer**: `prod-lb.adc.example.com`
- **Gateway Cluster**:
  - `prod-gw1.adc.example.com`
  - `prod-gw2.adc.example.com`
- **Controller Cluster**:
  - `prod-ctrl1.adc.example.com`
  - `prod-ctrl2.adc.example.com`
- **Hub**: `prod-hub1.adc.example.com`
- **EDA**: `prod-eda.adc.example.com`
- **Database**:
  - Standby: `prod-db1.adc.example.com`
  - Redis: `prod-redis1.adc.example.com`
- **Execution Nodes** (ADC Workloads):
  - `prod-adc-exec1.adc.example.com`
  - `prod-adc-exec2.adc.example.com`

### Production - Azure Execution Nodes

- **IOC-based**: `prod-azure-exec1.ioc.example.com`
- **ADC-based**: `prod-azure-exec2.adc.example.com`
- **Purpose**: Separate nodes for production Azure workloads

## 🧪 Non-Production Environment Architecture

### Non-Production - Global Layer

#### **Non-Production Global Load Balancer**
- **Endpoint**: `aap-nonprod.example.com`
- **Primary Route**: IOC Datacenter
- **Failover Route**: ADC Datacenter
- **Purpose**: Non-Prod/Dev/Test traffic only

### Non-Production - IOC Datacenter

#### Components (Right-Sized for Non-Prod)
- **Load Balancer**: `nonprod-lb.ioc.example.com`
- **Gateway**: `nonprod-gw.ioc.example.com` (1 node sufficient)
- **Controller**: `nonprod-ctrl.ioc.example.com` (1 node sufficient)
- **Hub**: `nonprod-hub.ioc.example.com`
- **Database**:
  - Primary: `nonprod-db1.ioc.example.com`
  - Redis: `nonprod-redis1.ioc.example.com`
- **Execution Node**: `nonprod-ioc-exec.ioc.example.com`

### Non-Production - ADC Datacenter

#### Components (Right-Sized for Non-Prod)
- **Load Balancer**: `nonprod-lb.adc.example.com`
- **Gateway**: `nonprod-gw.adc.example.com` (1 node)
- **Controller**: `nonprod-ctrl.adc.example.com` (1 node)
- **Hub**: `nonprod-hub.adc.example.com`
- **Database**:
  - Standby: `nonprod-db1.adc.example.com`
  - Redis: `nonprod-redis1.adc.example.com`
- **Execution Node**: `nonprod-adc-exec.adc.example.com`

### Non-Production - Azure Execution Node

- **IOC-based**: `nonprod-azure-exec.ioc.example.com`
- **Purpose**: Non-production Azure workloads

## 🌐 Target Environment Strategy

### Production Target Mapping

| Target Environment | Execution Nodes | Workload Examples |
|-------------------|-----------------|-------------------|
| **Prod IOC Datacenter** | prod-ioc-exec1, prod-ioc-exec2 | Production servers in IOC |
| **Prod ADC Datacenter** | prod-adc-exec1, prod-adc-exec2 | Production servers in ADC |
| **Prod Azure Cloud** | prod-azure-exec1, prod-azure-exec2 | Production Azure VMs/services |

### Non-Production Target Mapping

| Target Environment | Execution Nodes | Workload Examples |
|-------------------|-----------------|-------------------|
| **NonProd IOC Datacenter** | nonprod-ioc-exec | Dev/Test servers in IOC |
| **NonProd ADC Datacenter** | nonprod-adc-exec | Dev/Test servers in ADC |
| **NonProd Azure Cloud** | nonprod-azure-exec | Dev/Test Azure VMs/services |

### Workload Type Segregation

Within each environment, execution nodes can be further segmented:

#### Production Workload Types
1. **Critical Databases**: Dedicated high-spec exec nodes
2. **Application Servers**: General purpose exec nodes
3. **Network Devices**: Nodes with network management tools
4. **Security Appliances**: Nodes in security management zone

#### Non-Production Workload Types
1. **Development**: Rapid change, flexible scheduling
2. **Testing**: Performance testing, load testing
3. **UAT**: User acceptance testing environments
4. **Demo/Training**: Sandbox environments

## 🔄 Cross-Datacenter Replication

### Production Environment Replication

#### Database Replication (Prod)
- **Primary**: IOC `prod-db1.ioc.example.com`
- **Standby**: ADC `prod-db1.adc.example.com`
- **Method**: PostgreSQL async streaming replication
- **RPO**: < 30 seconds
- **RTO**: < 5 minutes

#### Redis Replication (Prod)
- **Master**: IOC `prod-redis1.ioc.example.com`
- **Replica**: ADC `prod-redis1.adc.example.com`

### Non-Production Environment Replication

#### Database Replication (Non-Prod)
- **Primary**: IOC `nonprod-db1.ioc.example.com`
- **Standby**: ADC `nonprod-db1.adc.example.com`
- **Method**: PostgreSQL async streaming replication
- **RPO**: < 60 seconds (less stringent than Prod)
- **RTO**: < 15 minutes (less stringent than Prod)

#### Redis Replication (Non-Prod)
- **Master**: IOC `nonprod-redis1.ioc.example.com`
- **Replica**: ADC `nonprod-redis1.adc.example.com`

### Independent Replication

**Important**: Production and Non-Production replications are completely independent:
- Separate replication streams
- No shared infrastructure
- Different monitoring and alerting
- Can fail independently

## 🛡️ Failover Scenarios

### Production Datacenter Failover

#### IOC Failure (Production)
1. Production GSLB routes traffic to ADC
2. Prod ADC database promoted
3. Production workloads continue via ADC exec nodes
4. Prod IOC workloads fail (IOC exec nodes down)

#### ADC Failure (Production)
1. Production continues on IOC (already primary)
2. Prod ADC workloads fail (ADC exec nodes down)
3. Database replication stops (no standby)

### Non-Production Datacenter Failover

#### IOC Failure (Non-Production)
1. Non-Prod GSLB routes traffic to ADC
2. NonProd ADC database promoted
3. Non-Prod workloads continue via ADC exec nodes

#### ADC Failure (Non-Production)
1. Non-Production continues on IOC
2. Non-Prod ADC workloads fail

### Independent Failure Handling

Production and Non-Production failures are handled independently:
- Prod IOC fail + NonProd ADC fail = Both route to alternate DC
- Prod success + NonProd fail = Only NonProd affected
- Network issue affecting Prod = NonProd unaffected (separate VLANs)

## 📊 Capacity Planning

### Production Environment (Full Scale)

| Component | IOC Nodes | ADC Nodes | CPU/Node | RAM/Node |
|-----------|-----------|-----------|----------|----------|
| Load Balancer | 1 | 1 | 2 cores | 4 GB |
| Gateway | 2 | 2 | 4 cores | 8 GB |
| Controller | 2 | 2 | 8 cores | 16 GB |
| Hub | 1 | 1 | 4 cores | 8 GB |
| EDA | 1 | 1 | 4 cores | 8 GB |
| Database | 1 | 1 | 8 cores | 32 GB |
| Redis | 1 | 1 | 2 cores | 8 GB |
| Exec Nodes (DC) | 2 | 2 | 4 cores | 8 GB |
| Exec Nodes (Azure) | 1 | 1 | 4 cores | 8 GB |

**Prod Total**: 24 nodes across both DCs

### Non-Production Environment (Right-Sized)

| Component | IOC Nodes | ADC Nodes | CPU/Node | RAM/Node |
|-----------|-----------|-----------|----------|----------|
| Load Balancer | 1 | 1 | 2 cores | 4 GB |
| Gateway | 1 | 1 | 2 cores | 4 GB |
| Controller | 1 | 1 | 4 cores | 8 GB |
| Hub | 1 | 1 | 2 cores | 4 GB |
| Database | 1 | 1 | 4 cores | 16 GB |
| Redis | 1 | 1 | 2 cores | 4 GB |
| Exec Nodes (DC) | 1 | 1 | 2 cores | 4 GB |
| Exec Nodes (Azure) | 1 | 0 | 2 cores | 4 GB |

**NonProd Total**: 15 nodes across both DCs (40% of Prod size)

### Total Deployment

**Combined Total**: 39 nodes across both environments and DCs

### Cost Optimization for Non-Prod

1. **Smaller Instances**: Half the resources of Production
2. **Single Nodes**: Where HA is less critical
3. **Scheduled Shutdown**: Power off Non-Prod nights/weekends
4. **Fewer Exec Nodes**: Lower concurrency needs
5. **Shared Azure Node**: One Azure exec node sufficient

## 🔒 Security and Isolation

### Network Isolation

#### VLAN Segmentation
- **Prod IOC**: VLAN 100-109
- **Prod ADC**: VLAN 200-209
- **NonProd IOC**: VLAN 110-119
- **NonProd ADC**: VLAN 210-219

#### Firewall Rules
- **Prod ↔ NonProd**: **NO** direct connectivity
- **Prod IOC ↔ Prod ADC**: Allowed for replication
- **NonProd IOC ↔ NonProd ADC**: Allowed for replication
- **Prod Exec ↔ Prod Targets**: Allowed
- **NonProd Exec ↔ NonProd Targets**: Allowed
- **Prod Exec ↔ NonProd Targets**: **BLOCKED**
- **NonProd Exec ↔ Prod Targets**: **BLOCKED**

### Access Control Separation

| Aspect | Production | Non-Production |
|--------|-----------|----------------|
| **Authentication** | Enterprise AD/LDAP | Enterprise AD/LDAP |
| **Authorization** | Prod RBAC groups | NonProd RBAC groups |
| **MFA** | Required | Optional |
| **Audit Logging** | Full, retained 7 years | Standard, retained 1 year |
| **Change Control** | CAB approval required | Self-service |
| **Maintenance Windows** | Restricted, planned | Flexible, on-demand |

### Data Classification

- **Production Data**: Confidential, PII, customer data
- **Non-Production Data**: Sanitized, synthetic, test data
- **Prohibition**: No production data in non-prod environment

## 🧪 Testing and Promotion Strategy

### Environment Promotion Flow

```
Development (NonProd IOC) 
    → Testing (NonProd ADC)
    → UAT (NonProd)
    → Production (Prod IOC/ADC)
```

### AAP Upgrade Strategy

1. **Test in Non-Prod**: Upgrade NonProd IOC first
2. **Validate**: Run automation tests, verify functionality
3. **NonProd ADC**: Upgrade NonProd ADC
4. **Validate Cross-DC**: Test NonProd failover
5. **Production**: Schedule Prod maintenance window
6. **Prod Upgrade**: Upgrade Prod IOC, then Prod ADC
7. **Validate Prod**: Smoke tests, monitor for issues

### Content Promotion

1. **Develop**: Create playbooks/roles in NonProd
2. **Test**: Execute in NonProd environments
3. **Review**: Code review, security scan
4. **Promote**: Git tag/branch promotion
5. **Deploy**: Prod Hub syncs approved content

## ✅ High Availability Testing

### Production Environment Testing
- [ ] Prod IOC datacenter failure
- [ ] Prod ADC datacenter failure
- [ ] Prod IOC-ADC WAN failure
- [ ] Prod database failover
- [ ] Prod Gateway/Controller node failure
- [ ] Prod execution node failure

### Non-Production Environment Testing
- [ ] NonProd IOC datacenter failure
- [ ] NonProd ADC datacenter failure
- [ ] NonProd database failover
- [ ] NonProd component failures

### Isolation Testing
- [ ] Verify no Prod → NonProd network connectivity
- [ ] Verify no NonProd → Prod network connectivity
- [ ] Verify separate RBAC (users can't cross environments)
- [ ] Verify separate job logs and audit trails
- [ ] Test Prod failure doesn't impact NonProd
- [ ] Test NonProd failure doesn't impact Prod

## 📈 Monitoring Strategy

### Separate Monitoring Stacks

#### Production Monitoring
- **Tool**: Dedicated Prometheus/Grafana for Prod
- **Retention**: 90 days
- **Alerting**: PagerDuty for 24/7 on-call
- **Dashboards**: Real-time production metrics
- **SLAs**: 99.9% uptime target

#### Non-Production Monitoring
- **Tool**: Shared Prometheus/Grafana for NonProd
- **Retention**: 30 days
- **Alerting**: Email/Slack during business hours
- **Dashboards**: Development metrics
- **SLAs**: Best effort

### Key Metrics Per Environment

- GSLB health and failover status
- Database replication lag (per environment)
- Job success rates (per environment)
- Execution node capacity (per environment)
- Component availability (per environment)

## 📚 Compliance and Audit

### Separation Requirements

Many compliance frameworks require Prod/NonProd separation:

- **SOX**: Separation of duties, production isolation
- **PCI-DSS**: Isolated production card data environment
- **HIPAA**: PHI only in production, test data in non-prod
- **SOC 2**: Logical separation, access controls
- **ISO 27001**: Environment segregation

### Audit Trails

- **Production**: Full audit logging, immutable logs
- **Non-Production**: Standard logging
- **Separation**: Separate log aggregation systems
- **Retention**: Prod 7 years, NonProd 1 year

## 📚 Additional Resources

- [Red Hat AAP 2.5 Documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [Environment Separation Best Practices](https://www.redhat.com/en/topics/devops/what-is-separation-of-duties)
- [PostgreSQL Multi-Site Replication](https://www.postgresql.org/docs/current/warm-standby.html)

---

**Deployment Complexity**: Very High  
**Recommended For**: Regulated industries, large enterprises with strict Prod/NonProd separation  
**HA Level**: Full geographic redundancy per environment with complete isolation  
**Compliance**: Suitable for SOX, PCI-DSS, HIPAA, SOC 2, ISO 27001
