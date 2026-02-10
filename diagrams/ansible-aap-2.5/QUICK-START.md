# Quick Start Guide

Welcome! This guide will help you quickly get started with the Infrastructure Topology 3 diagrams.

## 🚀 5-Minute Quick Start

### 1. View the Diagram (Easiest)
Just open the Mermaid file directly on GitHub:
- Click on `infrastructure-topology-3.mmd`
- GitHub will render it automatically
- No software needed!

### 2. Edit Online (No Installation Required)
**Mermaid (Recommended):**
1. Visit https://mermaid.live/
2. Copy content from `infrastructure-topology-3.mmd`
3. Paste and edit
4. Download PNG when done

**PlantUML:**
1. Visit http://www.plantuml.com/plantuml/
2. Copy content from `infrastructure-topology-3.puml`
3. Paste and click Submit
4. Edit and resubmit as needed

**Draw.io:**
1. Visit https://app.diagrams.net/
2. File → Open → Select `infrastructure-topology-3.drawio`
3. Edit visually
4. File → Save/Export

### 3. Interactive Preview
Open `index.html` in your browser for:
- Live diagram rendering
- Component details
- Network communication table
- Quick edit links

## 📋 What This Diagram Shows

This diagram represents **Red Hat Ansible Automation Platform 2.5** deployed in an enterprise configuration called **"Infrastructure Topology 3"** - a multi-node setup where each component runs on its own server.

### The 7 Main Components:

1. **🔐 Gateway** (gateway.example.com)
   - Entry point for users
   - Port 443 (HTTPS)

2. **🎮 Controller** (controller.example.com)
   - Main control panel
   - Manages automation
   - Ports 443, 80

3. **📦 Hub** (hub.example.com)
   - Stores automation content
   - Ports 443, 80

4. **⚡ Event-Driven Ansible** (eda.example.com)
   - Responds to events automatically
   - Port 443 (Optional component)

5. **💾 Database** (db.example.com)
   - PostgreSQL (Port 5432)
   - Redis cache (Port 6379)

6. **⚙️ Execution Nodes** (execnode1-3.example.com)
   - Workers that run automation
   - Port 27199

7. **🌐 External Systems**
   - Git for code storage
   - Target machines to manage

## 🎯 Common Use Cases

### For Architects:
- Use Draw.io for presentations
- Export to PowerPoint/PDF
- Customize colors and layout

### For Documentation:
- Use Mermaid in Markdown files
- Embeds directly in GitHub/GitLab
- Version control friendly

### For Teams:
- Edit PlantUML in text editors
- Review changes in pull requests
- Easy collaboration

## 🔧 Quick Customizations

### Add a Load Balancer (Mermaid):
```mermaid
subgraph LB["Load Balancer"]
    LoadBalancer[HAProxy]
end
Users --> LoadBalancer
LoadBalancer --> AG
```

### Change a Port:
Just edit the text:
- From: `Port: 443 HTTPS`
- To: `Port: 8443 HTTPS`

### Add a New Node:
Copy an existing execution node section and rename it.

## 📖 Learn More

- **Full Documentation:** See `README.md`
- **How to Edit:** See `EDITING-GUIDE.md`
- **Complete Overview:** See `PROJECT-SUMMARY.md`

## ❓ FAQ

**Q: Which format should I use?**
- For GitHub: Use Mermaid (.mmd)
- For presentations: Use Draw.io (.drawio)
- For enterprise docs: Use PlantUML (.puml)

**Q: Can I export to PNG/PDF?**
- Yes! All formats support export
- Mermaid: Via mermaid.live
- PlantUML: Via plantuml.com or IDE
- Draw.io: File → Export As

**Q: Can I modify the diagram?**
- Absolutely! That's the point
- All formats are fully editable
- See EDITING-GUIDE.md for details

**Q: Is this the official Red Hat diagram?**
- Based on official Red Hat documentation
- Represents the tested Topology 3
- Suitable for production planning

**Q: Can I use this for my company?**
- Yes, for planning deployments
- Always verify with official Red Hat docs
- Consult Red Hat support for production

## 🎨 Need Help?

1. Check the `EDITING-GUIDE.md` for detailed instructions
2. View `README.md` for architecture details
3. Open `index.html` for interactive preview
4. Refer to official Red Hat documentation

## ⚡ Pro Tips

- **Tip 1:** Mermaid renders in GitHub - use it for README files
- **Tip 2:** Draw.io is best for presentations and visual editing
- **Tip 3:** PlantUML is great for automated documentation generation
- **Tip 4:** Keep all three formats in sync when making changes
- **Tip 5:** Use version control to track diagram changes over time

---

**Ready to get started?** Pick your format and start editing! 🚀

For the full documentation, see [`README.md`](./README.md)
