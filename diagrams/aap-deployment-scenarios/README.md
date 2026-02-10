# AAP Deployment Scenarios - High Availability Architectures

## 📋 Overview

This directory contains comprehensive deployment architectures for **Red Hat Ansible Automation Platform (AAP) 2.5** with various high availability configurations. Each scenario addresses specific business requirements for datacenter redundancy, maintenance tolerance, and environment isolation.

## 🎯 Available Scenarios

### [Scenario 1: Single Datacenter with HA](./scenario1-single-dc/)
**Single IOC datacenter deployment with full high availability**

- **Datacenters**: IOC only
- **HA Level**: Component-level within datacenter
- **Execution Nodes**: Common pool for all targets (IOC, ADC, Azure)
- **Maintenance**: Zero-downtime maintenance capability
- **Use Case**: Organizations with single datacenter, scheduled maintenance windows
- **Complexity**: Medium
- **Total Nodes**: ~19 nodes

**Key Features**:
- ✅ Load balancer HA pair
- ✅ Active-Active Gateway and Controller clusters
- ✅ PostgreSQL streaming replication
- ✅ Common execution node pool for all environments
- ✅ Maintenance without downtime

### [Scenario 2: Dual Datacenter with HA](./scenario2-dual-dc/)
**Two datacenter (IOC & ADC) deployment with datacenter failure tolerance**

- **Datacenters**: IOC (Primary) and ADC (Secondary)
- **HA Level**: Geographic redundancy with datacenter failover
- **Execution Nodes**: Separate per datacenter + Azure-specific
- **Maintenance**: Datacenter-level failure tolerance
- **Use Case**: Mission-critical environments requiring site resilience
- **Complexity**: High
- **Total Nodes**: ~26 nodes (13 per datacenter)

**Key Features**:
- ✅ Global load balancer (GSLB) for datacenter failover
- ✅ Cross-datacenter database replication
- ✅ Separate execution nodes per target environment
- ✅ Complete datacenter failure tolerance
- ✅ Workload-based execution node segregation

### [Scenario 3: Separate Prod/NonProd Environments](./scenario3-nonprod-prod/)
**Complete environment isolation with dual-datacenter HA for each**

- **Datacenters**: IOC and ADC for both Prod and NonProd
- **HA Level**: Geographic redundancy per environment
- **Execution Nodes**: Separate per environment, datacenter, and workload
- **Maintenance**: Independent failover per environment
- **Use Case**: Regulated industries, strict Prod/NonProd separation
- **Complexity**: Very High
- **Total Nodes**: ~39 nodes (24 Prod + 15 NonProd)

**Key Features**:
- ✅ Complete Prod/NonProd isolation
- ✅ Independent GSLB per environment
- ✅ Separate databases per environment
- ✅ Network segregation (VLANs, firewalls)
- ✅ Compliance-ready (SOX, PCI-DSS, HIPAA)
- ✅ Independent upgrade and testing paths

## 📊 Scenario Comparison Matrix

| Feature | Scenario 1 | Scenario 2 | Scenario 3 |
|---------|-----------|-----------|-----------|
| **Datacenters** | 1 (IOC) | 2 (IOC + ADC) | 2 per env (4 total) |
| **DC Failure Tolerance** | ❌ No | ✅ Yes | ✅ Yes (per env) |
| **Maintenance Tolerance** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Exec Node Strategy** | Common Pool | Separate per DC | Separate per env+DC |
| **Environments** | Mixed | Mixed | Isolated (Prod/NonProd) |
| **Complexity** | Medium | High | Very High |
| **Total Nodes** | ~19 | ~26 | ~39 |
| **Cost** | $ | $$ | $$$ |
| **RPO** | N/A | < 30s | < 30s (Prod), < 60s (NonProd) |
| **RTO** | < 2min | < 5min | < 5min (Prod), < 15min (NonProd) |

## 🏗️ Common Architecture Components

All scenarios include these AAP 2.5 components:

1. **Automation Gateway** - API gateway and entry point (New in 2.5)
2. **Automation Controller** - Central control plane (formerly Tower)
3. **Automation Hub** - Private content repository
4. **Event-Driven Ansible** - Event-based automation triggers
5. **PostgreSQL Database** - Persistent data storage
6. **Redis Cache** - Task queue and session cache
7. **Execution Nodes** - Scalable automation workers via Automation Mesh

## 🔧 Available Diagram Formats

Each scenario includes diagrams in **three editable formats**:

### 1. **Mermaid (.mmd)**
- **Best for**: GitHub documentation, Markdown files
- **Editing**: Text-based, version control friendly
- **Rendering**: Automatic in GitHub, GitLab, many wikis
- **Online Editor**: [mermaid.live](https://mermaid.live/)

### 2. **PlantUML (.puml)**
- **Best for**: Enterprise architecture documentation
- **Editing**: Text-based UML format
- **Rendering**: VS Code, IntelliJ, Eclipse plugins
- **Online Editor**: [plantuml.com](http://www.plantuml.com/plantuml/)

### 3. **Draw.io (.drawio)** *(Coming Soon)*
- **Best for**: Visual editing, presentations
- **Editing**: Drag-and-drop visual editor
- **Rendering**: [diagrams.net](https://app.diagrams.net/), VS Code extension
- **Export**: PNG, PDF, SVG, XML

## 📖 Documentation Structure

Each scenario includes:

- **README.md** - Complete architecture documentation:
  - Component descriptions
  - Network requirements
  - Failover procedures
  - Capacity planning
  - Testing checklists
  - Security considerations

- **Diagrams** - Visual architecture representations:
  - Mermaid (.mmd)
  - PlantUML (.puml)
  - Draw.io (.drawio) *(coming soon)*

## 🎯 Choosing the Right Scenario

### Choose Scenario 1 if:
- ✅ You have a single datacenter
- ✅ You need maintenance windows without downtime
- ✅ Datacenter-level failure is acceptable (backup/DR separate)
- ✅ You want to minimize infrastructure complexity
- ✅ Common execution nodes are acceptable for all targets

### Choose Scenario 2 if:
- ✅ You have two datacenters
- ✅ You need to survive complete datacenter failure
- ✅ You want workload-specific execution nodes
- ✅ You need geographic redundancy
- ✅ Mission-critical availability is required

### Choose Scenario 3 if:
- ✅ You need complete Prod/NonProd separation
- ✅ You have compliance requirements (SOX, PCI, HIPAA)
- ✅ You want independent upgrade and testing paths
- ✅ You need separate execution nodes per environment
- ✅ Budget allows for maximum isolation and redundancy

## 🌐 Target Environment Strategy

### Scenario 1: Common Execution Pool
```
                 ┌─ IOC Targets
Common Exec Pool ├─ ADC Targets
                 └─ Azure Targets
```

### Scenario 2: Datacenter-Specific Nodes
```
IOC Exec Nodes ──────→ IOC Targets
ADC Exec Nodes ──────→ ADC Targets
Azure Exec Nodes ────→ Azure Targets (from both DCs)
```

### Scenario 3: Environment + DC-Specific Nodes
```
Prod IOC Exec ──────→ Prod IOC Targets
Prod ADC Exec ──────→ Prod ADC Targets
Prod Azure Exec ────→ Prod Azure Targets
NonProd IOC Exec ───→ NonProd IOC Targets
NonProd ADC Exec ───→ NonProd ADC Targets
NonProd Azure Exec ─→ NonProd Azure Targets
```

## 📈 Capacity Planning Summary

### Resource Requirements by Scenario

| Scenario | Total Nodes | Total CPUs | Total RAM | Relative Cost |
|----------|-------------|------------|-----------|---------------|
| Scenario 1 | ~19 | ~110 cores | ~288 GB | Baseline |
| Scenario 2 | ~26 | ~124 cores | ~360 GB | +40% |
| Scenario 3 | ~39 | ~200 cores | ~550 GB | +90% |

*Note: Scenario 3 includes right-sized NonProd (40% of Prod). Costs can be optimized with scheduled shutdowns.*

## 🔒 Security Considerations

### Network Segmentation (All Scenarios)
- **DMZ**: Load Balancers
- **Application Tier**: Gateway, Controller, Hub, EDA
- **Data Tier**: Database, Redis (no direct external access)
- **Execution Tier**: Execution nodes (controlled outbound)

### Scenario 3 Additional Security
- **VLAN Isolation**: Separate VLANs for Prod and NonProd
- **Firewall Rules**: No Prod ↔ NonProd connectivity
- **Access Control**: Separate RBAC per environment
- **Audit Logging**: Separate, longer retention for Prod

## 🧪 Testing and Validation

### High Availability Testing (All Scenarios)
- [ ] Component failure testing (Gateway, Controller, etc.)
- [ ] Database failover testing
- [ ] Load balancer failover testing
- [ ] Execution node failure testing
- [ ] Network failure testing

### Datacenter Failover Testing (Scenarios 2 & 3)
- [ ] Primary datacenter failure
- [ ] Secondary datacenter failure
- [ ] WAN link failure (split-brain prevention)
- [ ] Database promotion and failback
- [ ] GSLB failover validation

### Isolation Testing (Scenario 3 Only)
- [ ] Network isolation validation
- [ ] Access control separation
- [ ] Independent failover per environment
- [ ] Data segregation compliance

## 📚 Additional Resources

### Official Documentation
- [Red Hat AAP 2.5 Documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [Tested Deployment Models](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/tested_deployment_models/)
- [Automation Mesh Architecture](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/automation_mesh/)

### High Availability References
- [PostgreSQL High Availability](https://www.postgresql.org/docs/current/high-availability.html)
- [Redis Replication](https://redis.io/docs/management/replication/)
- [HAProxy Configuration](http://www.haproxy.org/#docs)

### Compliance Frameworks
- [SOX IT Controls](https://www.sec.gov/spotlight/sarbanes-oxley.htm)
- [PCI DSS Requirements](https://www.pcisecuritystandards.org/)
- [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html)

## 🤝 Contributing

To enhance these scenarios:

1. Review the existing diagrams and documentation
2. Make improvements to Mermaid/PlantUML/Draw.io files
3. Update README documentation
4. Test diagram rendering
5. Submit pull request with description

## 📝 Version Information

- **AAP Version**: 2.5
- **Deployment Method**: RPM-based installation
- **Architecture Type**: Multi-node enterprise deployments
- **Last Updated**: 2026-02-10

---

**Need Help Choosing?** Review each scenario's README for detailed use cases, requirements, and decision criteria.
