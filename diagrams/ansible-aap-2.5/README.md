# Red Hat Ansible Automation Platform 2.5
## Infrastructure Topology 3 - Multi-Node Enterprise Deployment (RPM)

This directory contains editable diagrams for the Infrastructure Topology 3 of Red Hat Ansible Automation Platform (AAP) 2.5, deployed using RPM packages.

## 📋 Overview

**Infrastructure Topology 3** is an enterprise-grade, multi-node deployment architecture where each major AAP component runs on a dedicated host (VM or physical server). This topology is designed for:

- **High Availability**: Separation of services allows for independent scaling and failover
- **Enterprise Scale**: Supports large-scale automation across multiple teams and environments
- **Production Workloads**: Red Hat tested and supported configuration for mission-critical automation
- **Security**: Component isolation reduces attack surface and allows for network segmentation

## 🏗️ Architecture Components

### 1. **Automation Gateway** (`gateway.example.com`)
- **Role**: Entry point for all external API access and user interactions
- **Ports**: 443 (HTTPS)
- **Purpose**: Provides unified API gateway and routing to backend services
- **New in AAP 2.5**: Required component for all deployments

### 2. **Automation Controller** (`controller.example.com`)
- **Role**: Central control plane for automation (formerly Ansible Tower)
- **Ports**: 443, 80 (HTTPS/HTTP)
- **Purpose**: 
  - Manages job scheduling and execution
  - Provides Web UI and REST API
  - Handles inventory, credentials, and projects
  - Coordinates with execution nodes
- **Connections**:
  - Database (PostgreSQL) on port 5432
  - Redis cache on port 6379
  - Execution nodes via mesh (port 27199)

### 3. **Automation Hub** (`hub.example.com`)
- **Role**: Private content repository for Ansible collections
- **Ports**: 443, 80 (HTTPS/HTTP)
- **Purpose**:
  - Hosts curated Ansible content collections
  - Syncs with Red Hat certified content and community Galaxy
  - Provides collection dependencies for playbooks
- **Connections**:
  - Database (PostgreSQL) on port 5432
  - External Git repositories for sync

### 4. **Event-Driven Ansible** (`eda.example.com`) *[Optional]*
- **Role**: Real-time, event-based automation trigger system
- **Port**: 443 (HTTPS)
- **Purpose**:
  - Listens to event sources (webhooks, monitoring, etc.)
  - Triggers automation based on predefined rules
  - Enables reactive automation workflows
- **Connections**:
  - Database (PostgreSQL) on port 5432

### 5. **Database Server** (`db.example.com`)
- **Components**:
  - **PostgreSQL Database** (Port 5432)
    - Shared database for all AAP components
    - Stores configuration, job history, inventory data
  - **Redis Cache** (Port 6379)
    - Task queuing and caching
    - Used by Automation Controller for job management

### 6. **Execution Nodes** (`execnode1-3.example.com`)
- **Role**: Scalable workers that execute automation tasks
- **Port**: 27199 (Automation Mesh communication)
- **Purpose**:
  - Run Ansible playbooks and ad-hoc commands
  - Connect to target infrastructure
  - Can be distributed across networks/regions
- **Scalability**: Add more nodes as needed for capacity

### 7. **External Systems**
- **Git Repositories**: Source control for playbooks and roles
- **Target Infrastructure**: Hosts and systems managed by automation

## 🔗 Network Requirements

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Users/APIs | Automation Gateway | 443 | HTTPS | External access |
| Gateway | Controller | 443 | HTTPS | API routing |
| Gateway | Hub | 443 | HTTPS | API routing |
| Gateway | EDA | 443 | HTTPS | API routing |
| Controller | Database | 5432 | PostgreSQL | Data persistence |
| Controller | Redis | 6379 | Redis | Task queue |
| Controller | Execution Nodes | 27199 | Automation Mesh | Job execution |
| Hub | Database | 5432 | PostgreSQL | Data persistence |
| EDA | Database | 5432 | PostgreSQL | Data persistence |
| Hub/Controller | Git | 443/22 | HTTPS/SSH | Content sync |
| Execution Nodes | Targets | Various | Various | Automation tasks |

## 📁 Available Diagram Formats

This repository provides the topology in three editable formats:

### 1. **Mermaid Diagram** (`infrastructure-topology-3.mmd`)
- **Best for**: GitHub documentation, Markdown files, GitLab wikis
- **How to edit**: 
  - Edit the text file directly
  - View/edit online: [Mermaid Live Editor](https://mermaid.live/)
  - Renders automatically in GitHub Markdown
- **Advantages**: Version control friendly, lightweight, text-based

### 2. **PlantUML Diagram** (`infrastructure-topology-3.puml`)
- **Best for**: Software documentation, enterprise architecture
- **How to edit**:
  - Edit the text file directly
  - View/edit online: [PlantUML Web Server](http://www.plantuml.com/plantuml/)
  - Use IDE plugins (VS Code, IntelliJ, Eclipse)
- **Advantages**: Industry standard, extensive styling options, text-based

### 3. **Draw.io/Diagrams.net** (`infrastructure-topology-3.drawio`)
- **Best for**: Presentations, detailed visual editing
- **How to edit**:
  - Open in [Diagrams.net](https://app.diagrams.net/) (formerly Draw.io)
  - Use VS Code with Draw.io Integration extension
  - Desktop app available for offline editing
- **Advantages**: Visual editor, drag-and-drop, export to PNG/PDF/SVG

## 🚀 Deployment Considerations

### Minimum Host Requirements (per component):
- **CPU**: 4 cores (8+ recommended for Controller)
- **RAM**: 8 GB minimum (16+ GB recommended for Controller/Database)
- **Disk**: 40 GB minimum (adjust for database based on scale)
- **OS**: RHEL 9.x or RHEL 8.x

### High Availability Options:
- **Controller**: Deploy multiple instances behind load balancer
- **Hub**: Deploy multiple instances behind load balancer
- **Database**: PostgreSQL clustering or managed database service
- **Execution Nodes**: Add nodes as needed (horizontal scaling)

### Network Segmentation:
- Place Gateway in DMZ for external access
- Controller, Hub, EDA in application tier
- Database in data tier with restricted access
- Execution nodes in appropriate network zones based on targets

## 📚 References

- [Official Red Hat AAP 2.5 Documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/)
- [RPM Topologies - Tested Deployment Models](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/tested_deployment_models/rpm-topologies)
- [AAP Installation Guide](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/rpm_installation/)

## 📝 License

These diagrams are provided as reference implementations based on Red Hat's publicly documented architecture. Please refer to Red Hat's official documentation for production deployments.

## 🤝 Contributing

To modify or enhance these diagrams:

1. Choose your preferred format (Mermaid, PlantUML, or Draw.io)
2. Edit the diagram using appropriate tools
3. Test rendering to ensure correctness
4. Update this README if adding new components or changing architecture

## ✅ Validation

The diagrams in this repository represent:
- ✅ Red Hat AAP 2.5 official architecture
- ✅ Infrastructure Topology 3 (multi-node enterprise deployment)
- ✅ RPM-based installation method
- ✅ All required components for AAP 2.5
- ✅ Correct network ports and protocols
- ✅ Editable in multiple formats

---

**Last Updated**: 2026-02-10  
**AAP Version**: 2.5  
**Topology**: Infrastructure Topology 3 (RPM)
