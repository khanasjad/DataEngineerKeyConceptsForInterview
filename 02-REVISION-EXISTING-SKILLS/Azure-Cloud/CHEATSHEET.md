# Azure Cloud Cheatsheet - Quick Reference

## Azure Data Factory (ADF)
- **Pipeline**: Workflow of activities (ETL/ELT orchestration)
- **Activity**: Unit of work (Copy, Data Flow, Stored Proc, Web)
- **Linked Service**: Connection to data source/destination
- **Dataset**: Data structure reference
- **Trigger**: Schedule or event to start pipeline
- **Integration Runtime**: Compute for data movement (Azure IR, Self-hosted IR, SSIS IR)
- **Mapping Data Flow**: Visual ETL designer

## Azure Synapse Analytics
- **Dedicated SQL Pool**: Massively parallel processing data warehouse (formerly SQL DW)
- **Serverless SQL**: On-demand SQL queries over data lake (pay-per-query)
- **Apache Spark Pool**: Managed Spark for big data processing
- **Synapse Pipelines**: Same as ADF pipelines
- **PolyBase**: Load data from external sources
- **Distribution**: Hash (even distribution by key), Round-robin (random), Replicated (copy to all nodes)

## Azure Data Lake Storage (ADLS) Gen2
- **Hierarchical namespace**: Folder structure (not just flat blobs)
- **Access control**: RBAC (role-based) + POSIX ACLs (file-level)
- **Storage tiers**: Hot (frequent access), Cool (30-day retention), Archive (long-term, rare access)
- **Lifecycle management**: Auto-move between tiers based on age
- **Blob types**: Block blob (files), Append blob (logs), Page blob (VMs)

## Azure Databricks
- **Workspace**: Databricks environment on Azure
- **Cluster**: Compute (Standard, High Concurrency, Single Node)
- **Unity Catalog**: Cross-workspace governance
- **Delta Lake**: ACID transactions, time travel
- **Notebooks**: Collaborative Python/SQL/Scala/R
- **Jobs**: Scheduled/triggered workflows
- **Repos**: Git integration

## Azure Security
- **Key Vault**: Secrets, keys, certificates management
- **Managed Identity**: Passwordless authentication for Azure services
- **Azure AD (Entra ID)**: Identity provider (users, groups, apps)
- **Service Principal**: Application identity for automation
- **RBAC**: Assign roles (Owner, Contributor, Reader, custom)
- **Private Endpoint**: Private IP access to PaaS services (no public internet)
- **Network Security Group (NSG)**: Firewall rules for subnets

## Azure Storage
- **Blob Storage**: Object storage (files, images, backups)
- **File Share**: SMB file system in cloud
- **Queue Storage**: Message queue for async processing
- **Table Storage**: NoSQL key-value store
- **Disk Storage**: Persistent disks for VMs
- **SAS Token**: Time-limited access without sharing keys

## Azure Databases
- **Azure SQL Database**: Managed PaaS relational DB (single DB, elastic pool)
- **SQL Managed Instance**: Lift-and-shift SQL Server (near 100% compatibility)
- **Cosmos DB**: Globally distributed NoSQL (multi-model: document, key-value, graph)
- **Synapse Analytics**: Data warehouse (covered above)
- **PostgreSQL / MySQL**: Managed open-source databases

## Azure Compute
- **Virtual Machines**: IaaS VMs (Windows/Linux)
- **VM Scale Sets**: Auto-scaling VM groups
- **AKS (Kubernetes Service)**: Managed Kubernetes
- **Container Instances**: Serverless containers
- **App Service**: Managed web app hosting (PaaS)
- **Azure Functions**: Serverless event-driven compute
- **Azure Batch**: Large-scale batch jobs

## Azure Networking
- **VNet**: Isolated network in Azure
- **Subnet**: Segment VNet (public/private)
- **VPN Gateway**: Site-to-site VPN
- **ExpressRoute**: Private dedicated connection to Azure
- **Load Balancer**: Layer 4 (TCP/UDP) load balancing
- **Application Gateway**: Layer 7 (HTTP/HTTPS) load balancer + WAF
- **Azure Front Door**: Global HTTP load balancer + CDN
- **Azure Bastion**: Secure RDP/SSH without public IP

## Azure Monitoring
- **Azure Monitor**: Centralized monitoring (metrics + logs)
- **Log Analytics**: Query logs with KQL (Kusto Query Language)
- **Application Insights**: APM for applications (traces, metrics, logs)
- **Alerts**: Metric/log-based notifications (email, SMS, webhook)
- **Diagnostic Settings**: Send logs to Log Analytics, Storage, Event Hub
- **Metrics Explorer**: Visualize and analyze metrics

## Azure Cost Management
- **Cost Analysis**: View and analyze spending
- **Budgets**: Set spending limits with alerts
- **Advisor**: Cost optimization recommendations
- **Reserved Instances**: 1-3 year commitment (30-70% savings)
- **Spot VMs**: Unused capacity (up to 90% off)
- **Tags**: Categorize resources for cost tracking

## Infrastructure as Code
- **ARM Templates**: JSON infrastructure definitions
- **Bicep**: Simplified ARM DSL (transpiles to ARM)
- **Terraform**: Multi-cloud IaC tool
- **Azure CLI**: Command-line interface `az <command>`
- **Azure PowerShell**: PowerShell module `Az.*` cmdlets

## Best Practices
✅ Use Managed Identities (no passwords) | ✅ Enable Private Endpoints | ✅ Tag all resources | ✅ Set up Cost Alerts | ✅ Implement RBAC (least privilege) | ✅ Use Key Vault for secrets | ✅ Monitor with Log Analytics | ✅ Backup critical data | ✅ Use lifecycle policies for storage | ✅ Automate with IaC
