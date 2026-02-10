# Quick Start Guide - AAP Deployment Scenarios

## 🚀 5-Minute Quick Start

This guide helps you quickly understand and choose the right AAP deployment scenario for your needs.

## 📋 Quick Decision Tree

Answer these questions to find your scenario:

### Question 1: How many datacenters do you have?
- **One datacenter** → Go to Question 2
- **Two or more datacenters** → Go to Question 3

### Question 2: Single Datacenter Path
**Do you need maintenance without downtime?**
- **Yes** → ✅ **[Scenario 1](./scenario1-single-dc/)** - Single DC with HA
- **No** → Consider basic AAP deployment (not covered here)

### Question 3: Multi-Datacenter Path
**Do you need complete Prod/NonProd separation?**
- **Yes** → ✅ **[Scenario 3](./scenario3-nonprod-prod/)** - Separate environments with HA
- **No** → ✅ **[Scenario 2](./scenario2-dual-dc/)** - Dual DC with HA

## 🎯 Scenario Summaries

### Scenario 1: Single Datacenter with HA
**"I have one datacenter and need to do maintenance without downtime"**

```
Users → Load Balancer → AAP Cluster → Common Exec Nodes → All Targets
                         (IOC DC)
```

**Best For**:
- Organizations with single datacenter
- Need maintenance windows
- Acceptable downtime if datacenter fails
- Want to minimize complexity

**Cost**: $ (Baseline)  
**Complexity**: Medium  
**Setup Time**: 2-3 days

**Quick Facts**:
- 19 nodes total
- All components in one location
- Common execution nodes serve all targets
- Database HA within datacenter
- RTO: < 2 minutes (component failure)

### Scenario 2: Two Datacenters with HA
**"I need to survive if one datacenter goes down"**

```
                    ┌→ IOC AAP Cluster → IOC Exec → IOC Targets
Users → GSLB ──────┤
                    └→ ADC AAP Cluster → ADC Exec → ADC Targets
                                      └→ Azure Exec → Azure
```

**Best For**:
- Mission-critical applications
- Two datacenter availability
- Need geographic redundancy
- Datacenter-level disaster recovery

**Cost**: $$ (+40% vs Scenario 1)  
**Complexity**: High  
**Setup Time**: 4-5 days

**Quick Facts**:
- 26 nodes total (13 per datacenter)
- Cross-datacenter database replication
- Separate exec nodes per datacenter
- Datacenter failover capability
- RTO: < 5 minutes (datacenter failure)
- RPO: < 30 seconds

### Scenario 3: Separate Prod/NonProd
**"I need complete isolation between Production and Non-Production"**

```
PRODUCTION:
Users → Prod GSLB → Prod IOC + Prod ADC → Prod Exec → Prod Targets

NON-PRODUCTION:
Users → NonProd GSLB → NonProd IOC + NonProd ADC → NonProd Exec → NonProd Targets
```

**Best For**:
- Regulated industries (SOX, PCI, HIPAA)
- Strict compliance requirements
- Complete environment isolation
- Independent upgrade paths

**Cost**: $$$ (+90% vs Scenario 1)  
**Complexity**: Very High  
**Setup Time**: 7-10 days

**Quick Facts**:
- 39 nodes total (24 Prod + 15 NonProd)
- Complete network isolation
- Separate GSLB per environment
- Independent failover per environment
- Compliance-ready architecture

## 📊 Quick Comparison Table

| Feature | Scenario 1 | Scenario 2 | Scenario 3 |
|---------|:----------:|:----------:|:----------:|
| **Datacenters** | 1 | 2 | 2 per env |
| **DC Failure Tolerance** | ❌ | ✅ | ✅ |
| **Maintenance w/o Downtime** | ✅ | ✅ | ✅ |
| **Prod/NonProd Isolation** | ❌ | ❌ | ✅ |
| **Nodes** | ~19 | ~26 | ~39 |
| **Cost** | $ | $$ | $$$ |
| **Complexity** | Medium | High | Very High |
| **Setup Time** | 2-3 days | 4-5 days | 7-10 days |
| **Best For** | Single DC HA | Geographic redundancy | Compliance |

## 🎨 View the Diagrams

All scenarios include visual diagrams in multiple formats:

### Mermaid (Renders on GitHub)
Just click on the `.mmd` files:
- [Scenario 1 Diagram](./scenario1-single-dc/scenario1-single-dc-ha.mmd)
- [Scenario 2 Diagram](./scenario2-dual-dc/scenario2-dual-dc-ha.mmd)
- [Scenario 3 Diagram](./scenario3-nonprod-prod/scenario3-nonprod-prod-ha.mmd)

### PlantUML (Copy to plantuml.com)
Copy content from `.puml` files and paste at [plantuml.com](http://www.plantuml.com/plantuml/)

## 💰 Cost Optimization Tips

### Scenario 1 Optimization
- Start with minimum nodes (2 controllers, 2 gateways)
- Add execution nodes as needed
- Use smaller VM sizes for non-critical components

### Scenario 2 Optimization
- Start with asymmetric deployment (more resources in primary DC)
- Use cloud resources for secondary datacenter
- Right-size based on actual failover needs

### Scenario 3 Optimization
- NonProd can be 40-50% size of Production
- Power off NonProd environments nights/weekends (save 60%)
- Use smaller instances for NonProd
- Share some NonProd resources across environments

**Example Scenario 3 Savings**:
```
Prod running 24/7:     24 nodes × $100 × 730 hrs = $175,200/mo
NonProd M-F 8am-6pm:   15 nodes × $50 × 220 hrs = $16,500/mo
Total:                                             $191,700/mo
vs. Full 24/7:                                     $280,000/mo
Savings:                                           $88,300/mo (31%)
```

## 🔧 Minimum Hardware Requirements

### Per Controller Node (All Scenarios)
- **CPU**: 8 cores
- **RAM**: 16 GB
- **Disk**: 60 GB
- **Network**: 1 Gbps

### Per Database Node (All Scenarios)
- **CPU**: 8 cores
- **RAM**: 32 GB
- **Disk**: 200 GB+ (SSD recommended)
- **Network**: 1 Gbps

### Per Execution Node (All Scenarios)
- **CPU**: 4 cores
- **RAM**: 8 GB
- **Disk**: 40 GB
- **Network**: 1 Gbps

## 🌐 Network Requirements

### Scenario 1 (Single DC)
- **Internal Network**: 1 Gbps between all components
- **External Access**: 1 Gbps for load balancer
- **Firewall Rules**: ~15 rules

### Scenario 2 (Dual DC)
- **Internal Network**: 1 Gbps within each DC
- **Inter-DC Link**: 100 Mbps minimum, < 50ms latency
- **External Access**: 1 Gbps per DC
- **Firewall Rules**: ~30 rules

### Scenario 3 (Prod/NonProd)
- **Internal Network**: 1 Gbps within each DC
- **Inter-DC Link**: 100 Mbps minimum, < 50ms latency
- **Network Isolation**: Separate VLANs for Prod/NonProd
- **External Access**: 1 Gbps per DC per environment
- **Firewall Rules**: ~60 rules (with isolation)

## ⚡ Quick Implementation Steps

### 1. Choose Your Scenario
- Review comparison table above
- Consider your requirements
- Check budget and timeline

### 2. Review Detailed Documentation
- Read the full README for your chosen scenario
- Review the architecture diagrams
- Understand all components

### 3. Prepare Infrastructure
- **Compute**: Provision VMs or physical servers
- **Network**: Configure VLANs, subnets, routing
- **Storage**: Set up shared storage if needed
- **Load Balancers**: Deploy HA pair or GSLB

### 4. Deploy AAP Components
- **Phase 1**: Database cluster (primary + standby)
- **Phase 2**: Controller and Gateway nodes
- **Phase 3**: Hub and EDA nodes
- **Phase 4**: Execution nodes
- **Phase 5**: Configure replication (Scenarios 2 & 3)

### 5. Test and Validate
- Verify component connectivity
- Test failover scenarios
- Validate execution node connectivity
- Run sample automation jobs

### 6. Production Cutover
- Configure GSLB/DNS
- Migrate automation content
- Train operations team
- Go live!

## 🧪 Testing Checklist

### Basic Testing (All Scenarios)
- [ ] Login to web UI
- [ ] Create test inventory
- [ ] Create test job template
- [ ] Run test playbook
- [ ] Verify job output
- [ ] Check logs

### HA Testing (All Scenarios)
- [ ] Stop one controller → verify service continues
- [ ] Stop one gateway → verify service continues
- [ ] Stop database → verify automatic failover
- [ ] Restart node under maintenance

### DC Failover Testing (Scenarios 2 & 3)
- [ ] Simulate IOC datacenter failure
- [ ] Verify automatic failover to ADC
- [ ] Test job execution from ADC
- [ ] Validate data replication
- [ ] Test failback procedure

### Isolation Testing (Scenario 3 Only)
- [ ] Verify no Prod→NonProd network connectivity
- [ ] Test separate user access
- [ ] Validate independent failover
- [ ] Check audit log separation

## 📚 Next Steps

### For Scenario 1 Users
1. Read [Scenario 1 Detailed README](./scenario1-single-dc/README.md)
2. Review [Mermaid Diagram](./scenario1-single-dc/scenario1-single-dc-ha.mmd)
3. Check [Capacity Planning](./scenario1-single-dc/README.md#capacity-planning)
4. Follow [Maintenance Strategy](./scenario1-single-dc/README.md#maintenance-strategy)

### For Scenario 2 Users
1. Read [Scenario 2 Detailed README](./scenario2-dual-dc/README.md)
2. Review [Mermaid Diagram](./scenario2-dual-dc/scenario2-dual-dc-ha.mmd)
3. Understand [Cross-DC Replication](./scenario2-dual-dc/README.md#cross-datacenter-replication)
4. Review [Failover Scenarios](./scenario2-dual-dc/README.md#datacenter-failure-scenarios)

### For Scenario 3 Users
1. Read [Scenario 3 Detailed README](./scenario3-nonprod-prod/README.md)
2. Review [Mermaid Diagram](./scenario3-nonprod-prod/scenario3-nonprod-prod-ha.mmd)
3. Understand [Environment Isolation](./scenario3-nonprod-prod/README.md#security-and-isolation)
4. Review [Compliance Requirements](./scenario3-nonprod-prod/README.md#compliance-and-audit)

## ❓ FAQ

**Q: Can I start with Scenario 1 and upgrade to Scenario 2 later?**  
A: Yes! You can add a second datacenter and migrate to Scenario 2. Plan for the migration carefully.

**Q: Do I need three instances of everything for HA?**  
A: No. Most components work with 2 instances. Controllers benefit from 3+ for quorum-based operations.

**Q: What if I have more than two datacenters?**  
A: Scenarios can be extended. Consider one primary + multiple secondary DCs or active-active across all sites.

**Q: Can I mix scenarios (e.g., Scenario 2 for Prod, Scenario 1 for NonProd)?**  
A: Yes! This is common. Use higher HA for Production, simpler for Non-Production.

**Q: How long does failover take?**  
A: Component failure: seconds. Datacenter failure: 2-5 minutes. Depends on detection and automation.

**Q: Do I need Red Hat support for these scenarios?**  
A: Recommended. These are complex deployments. Red Hat support helps with planning and troubleshooting.

**Q: Can I use cloud services (AWS, Azure, GCP) instead of datacenters?**  
A: Absolutely! Replace "IOC datacenter" with "AWS us-east-1" and "ADC datacenter" with "AWS us-west-2".

**Q: What about disaster recovery (DR)?**  
A: Scenario 2 and 3 provide DR. Scenario 1 needs additional DR planning (backups, DR site).

## 🎓 Training and Resources

### Official Documentation
- [Red Hat AAP 2.5 Docs](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [AAP Installation Guide](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/rpm_installation/)

### Community Resources
- [Ansible Community Forum](https://forum.ansible.com/)
- [Red Hat Customer Portal](https://access.redhat.com/)

### Training
- Red Hat Training: Ansible Automation Platform Administration
- Red Hat Training: Advanced Automation: Ansible Best Practices

---

**Ready to Deploy?** Choose your scenario above and dive into the detailed documentation!

**Need Help?** Contact Red Hat support or your Red Hat account team for deployment assistance.
