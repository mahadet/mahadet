- 👋 Hi, I’m @mahadet
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
This is my first file in Git Repository

## 📊 Red Hat Ansible Automation Platform Deployments

### Red Hat Ansible Automation Platform 2.5

#### Reference Architecture
**[Infrastructure Topology 3 - Multi-Node Enterprise Deployment](./diagrams/ansible-aap-2.5/)**

Base reference architecture showing all AAP 2.5 components in a multi-node deployment.

Available in multiple editable formats:
- 🎨 [Mermaid Diagram](./diagrams/ansible-aap-2.5/infrastructure-topology-3.mmd) - Text-based, renders in GitHub
- 📐 [PlantUML Diagram](./diagrams/ansible-aap-2.5/infrastructure-topology-3.puml) - Industry standard UML
- ✏️ [Draw.io Diagram](./diagrams/ansible-aap-2.5/infrastructure-topology-3.drawio) - Visual editor compatible

#### High Availability Deployment Scenarios
**[AAP Deployment Scenarios - HA Architectures](./diagrams/aap-deployment-scenarios/)**

Complete deployment architectures with high availability configurations:

1. **[Scenario 1: Single Datacenter with HA](./diagrams/aap-deployment-scenarios/scenario1-single-dc/)**
   - Single IOC datacenter deployment
   - Component-level high availability
   - Common execution node pool for all targets
   - Zero-downtime maintenance capability
   - **Complexity**: Medium | **Nodes**: ~19

2. **[Scenario 2: Two Datacenters with HA](./diagrams/aap-deployment-scenarios/scenario2-dual-dc/)**
   - Dual datacenter (IOC + ADC) deployment
   - Complete datacenter failure tolerance
   - Separate execution nodes per datacenter
   - Cross-datacenter database replication
   - **Complexity**: High | **Nodes**: ~26

3. **[Scenario 3: Separate Prod/NonProd Environments](./diagrams/aap-deployment-scenarios/scenario3-nonprod-prod/)**
   - Complete Prod/NonProd isolation
   - Dual datacenter HA for each environment
   - Separate execution nodes per environment and DC
   - Compliance-ready (SOX, PCI-DSS, HIPAA)
   - **Complexity**: Very High | **Nodes**: ~39

Each scenario includes:
- 📊 Architecture diagrams (Mermaid, PlantUML)
- 📖 Comprehensive documentation
- 🔧 Capacity planning guides
- 🧪 Testing checklists
- 🔒 Security configurations

See the [deployment scenarios overview](./diagrams/aap-deployment-scenarios/README.md) for detailed comparison and selection guidance.

<!---
mahadet/mahadet is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
