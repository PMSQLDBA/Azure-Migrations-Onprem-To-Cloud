# Azure-Migrations-Onprem-To-Cloud

Below is a practical architecture-level view of the main **on-premises SQL Server → Azure migration options** and when each one fits. 

Microsoft currently supports migration paths to **Azure SQL Database, Azure SQL Managed Instance, and SQL Server on Azure VM**, with different tooling and downtime profiles. ([Microsoft Learn][1])

| Source                               | Azure Target                   | Migration Method                                     | Online / Offline                       | Typical Downtime   | Best Fit / Use Case                                        | Key Notes                                                                                                                                                             |
| ------------------------------------ | ------------------------------ | ---------------------------------------------------- | -------------------------------------- | ------------------ | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| On-prem SQL Server                   | **Azure SQL Database**         | Azure Database Migration Service (DMS)               | Offline                                | Medium to High     | Modernizing a standalone application database to PaaS      | Current DMS support for SQL Database is offline; application downtime begins when migration starts. Instance-level features may need redesign. ([Microsoft Learn][2]) |
| On-prem SQL Server                   | **Azure SQL Database**         | Schema + data migration using SSMS / migration tools | Offline                                | Medium             | Small/medium databases, dev/test, controlled migrations    | Good when you can remediate unsupported SQL Server features before migration.                                                                                         |
| On-prem SQL Server                   | **Azure SQL Database**         | BACPAC export/import                                 | Offline                                | High for large DBs | Small databases, dev/test, logical migration               | Not preferred for large production databases or minimal-downtime migrations.                                                                                          |
| On-prem SQL Server                   | **Azure SQL Database**         | Transactional replication / custom sync pattern      | Online-ish                             | Low                | Specialized cases requiring phased migration               | More complex operationally; requires careful application and consistency design.                                                                                      |
| On-prem SQL Server                   | **Azure SQL Managed Instance** | Native `.bak` backup + `RESTORE FROM URL`            | Offline                                | Medium to High     | Straightforward lift-and-shift when downtime is acceptable | One of the simplest MI migration methods. Backup is uploaded to Azure Blob Storage and restored directly to MI. ([Microsoft Learn][1])                                |
| On-prem SQL Server                   | **Azure SQL Managed Instance** | Azure Database Migration Service                     | Online / Offline depending on scenario | Low to Medium      | Enterprise-managed migration                               | Good for guided migration and centralized monitoring.                                                                                                                 |
| On-prem SQL Server                   | **Azure SQL Managed Instance** | Log Replay Service (LRS)                             | Online / Near-online                   | Low                | Large databases and minimal-downtime migrations            | Uses full/differential/log backups and continuously replays them to MI until cutover. ([Microsoft Learn][1])                                                          |
| On-prem SQL Server                   | **Azure SQL Managed Instance** | Managed Instance Link                                | Online                                 | Very Low           | Mission-critical systems needing near-zero downtime        | Uses near-real-time replication and allows controlled cutover. Strong choice when supported by source/version requirements. ([Microsoft Learn][1])                    |
| Azure Arc-enabled on-prem SQL Server | **Azure SQL Managed Instance** | Azure Arc integrated migration                       | Online / Minimal downtime              | Low                | Large estates already using Azure Arc                      | Azure portal can assess readiness and orchestrate MI Link or LRS-based migration. ([Microsoft Learn][1])                                                              |
| On-prem SQL Server                   | **SQL Server on Azure VM**     | Backup and Restore                                   | Offline                                | Medium to High     | Pure lift-and-shift with minimal application changes       | Highest compatibility because destination is still full SQL Server. ([Microsoft Learn][3])                                                                            |
| On-prem SQL Server                   | **SQL Server on Azure VM**     | Azure DMS Online Migration                           | Online                                 | Low                | Production workloads requiring minimal downtime            | DMS restores backup chain and completes cutover after final log synchronization. ([Microsoft Learn][4])                                                               |
| On-prem SQL Server                   | **SQL Server on Azure VM**     | Azure DMS Offline Migration                          | Offline                                | Medium             | Simpler managed migration with downtime allowed            | Useful when you want DMS orchestration without online synchronization. ([Microsoft Learn][5])                                                                         |
| On-prem SQL Server                   | **SQL Server on Azure VM**     | Always On / Distributed AG                           | Online                                 | Very Low           | Large, highly critical SQL Server workloads                | More complex but useful when HA and very low downtime are required.                                                                                                   |
| On-prem SQL Server VM                | **Azure VM**                   | Azure Migrate server migration                       | Offline / Replicated VM migration      | Low to Medium      | Moving the entire SQL Server VM, not only databases        | Appropriate when OS configuration, SQL instance, local files, agents, and applications need to move together.                                                         |
| On-prem SQL Server                   | **Azure SQL MI / VM**          | Log Shipping                                         | Online-ish                             | Low                | Traditional DBA-controlled migration                       | Good when existing operational knowledge and backup/log shipping processes are already mature.                                                                        |

### Which target should you choose?

| Requirement                          |          Azure SQL Database | Azure SQL Managed Instance |         SQL Server on Azure VM |
| ------------------------------------ | --------------------------: | -------------------------: | -----------------------------: |
| Maximum PaaS benefits                |                    **Best** |                  Very Good |                            Low |
| Native `.bak` restore                |                          No |                    **Yes** |                        **Yes** |
| SQL Server Agent                     |   Limited / redesign needed |                    **Yes** |                        **Yes** |
| Cross-database queries               |                     Limited |         **Strong support** |               **Full support** |
| Linked servers                       |                     Limited | Supported with limitations | **Full SQL Server capability** |
| OS access                            |                          No |                         No |                        **Yes** |
| SQL instance-level control           |                         Low |                       High |                       **Full** |
| Minimal application changes          |         Sometimes difficult |           **Usually good** |                       **Best** |
| Platform modernization               |                    **Best** |                       Good |                            Low |
| Lowest administration effort         |                    **Best** |                  Very Good |                        Highest |
| Near-zero downtime migration options | Limited depending on method |          **MI Link / LRS** |                   **DMS / AG** |
| Existing `.bak` migration            |                  Not direct |          **Excellent fit** |              **Excellent fit** |

### Common real-world cases

| Case                                                                                               | Recommended Target                   | Preferred Migration Method          | Why                                                     |
| -------------------------------------------------------------------------------------------------- | ------------------------------------ | ----------------------------------- | ------------------------------------------------------- |
| Small standalone application DB with no SQL Agent or instance dependencies                         | Azure SQL Database                   | DMS offline                         | Maximum PaaS benefit                                    |
| Old application heavily using SQL Agent, cross-database queries and instance features              | Azure SQL MI                         | Native restore / LRS                | High SQL Server compatibility without VM management     |
| Legacy application that cannot be modified                                                         | Azure VM                             | Backup/restore or DMS               | Full SQL Server compatibility                           |
| 5-TB production DB with only 10–15 minutes downtime available                                      | Azure SQL MI                         | **Managed Instance Link or LRS**    | Keeps target synchronized before cutover                |
| 500-GB database with a 6-hour maintenance window                                                   | Azure SQL MI                         | Native backup + restore             | Simple and cost-effective                               |
| Hundreds of SQL Servers requiring inventory and target recommendations                             | Azure Migrate + Azure Arc assessment | Target-specific migration afterward | Centralized discovery, sizing and migration planning    |
| Entire SQL Server VM including SQL Agent, SSIS packages, file shares and OS dependencies must move | Azure VM                             | Azure Migrate / VM migration        | Moves the complete server environment                   |
| Application being redesigned to cloud-native architecture                                          | Azure SQL Database                   | Schema/data migration               | Removes instance-level SQL dependencies                 |
| Production system using many linked servers and CLR components                                     | Azure VM or MI after assessment      | DMS / native migration              | SQL Database would usually require significant redesign |
| Need near-zero downtime and source supports MI Link                                                | Azure SQL MI                         | **Managed Instance Link**           | Near-real-time replication and controlled cutover       |
| Need near-zero downtime but prefer backup/log-based process                                        | Azure SQL MI                         | **LRS**                             | Continuous backup replay until final cutover            |
| Development/Test database refresh                                                                  | Azure SQL MI                         | Native `.bak` restore               | Simple operational process                              |
| Disaster-recovery style migration                                                                  | Azure SQL MI / Azure VM              | Backup chain, LRS or AG             | Better control of recovery and cutover                  |

### Migration decision flow

```text
                        On-Prem SQL Server
                                │
                                ▼
                    Assess Application / DB
                                │
               ┌────────────────┼─────────────────┐
               │                │                 │
               ▼                ▼                 ▼
        Can modernize?    Need SQL instance   Need full OS /
               │           compatibility?     SQL control?
               │                │                 │
              YES              YES               YES
               │                │                 │
               ▼                ▼                 ▼
        Azure SQL DB      Azure SQL MI       Azure SQL VM
               │                │                 │
               ▼                ▼                 ▼
           DMS /          Choose downtime      DMS / Backup
       Schema + Data          requirement       / AG / VM move
                                │
                       ┌────────┴─────────┐
                       │                  │
                Downtime OK?      Minimal downtime?
                       │                  │
                      YES                YES
                       │                  │
                       ▼                  ▼
               Native Backup       MI Link / LRS
                  Restore
```

For **Azure SQL Managed Instance specifically**, Microsoft currently lists native backup/restore, Log Replay Service, Managed Instance Link, Azure DMS, and Azure Arc-integrated migration among the main migration approaches. Native `.bak` restore is the simplest when downtime is acceptable; MI Link and LRS are stronger options when downtime must be minimized. ([Microsoft Learn][1])

One current limitation worth noting: 

for **SQL Server → Azure SQL Database**, Microsoft’s DMS documentation currently describes the migration as **offline**, while DMS supports online/minimal-downtime scenarios for targets such as SQL Server on Azure VM. ([Microsoft Learn][2])

For an enterprise design document, I would normally classify the options into **Rehost → Refactor → Modernize**:

**Rehost:** SQL Server on Azure VM
**Managed lift-and-shift:** Azure SQL Managed Instance
**Modernize:** Azure SQL Database

That classification usually makes the architecture decision much easier for application teams.

[1]: https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/guide "SQL Server to Azure SQL Managed Instance: Migration Guide | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/dms/tutorial-sql-server-azure-sql-database-offline-ads "Tutorial: Migrate SQL Server to Azure SQL Database (Offline) - Azure Database Migration Service | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/data-migration/sql-server/virtual-machines/guide "SQL Server to SQL Server on Azure Virtual Machines: Migration Guide - SQL Server on Azure VMs | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/data-migration/sql-server/virtual-machines/database-migration-service-online "Tutorial: Migrate SQL Server to SQL Server on an Azure Virtual Machine with Azure DMS (Online) - Azure Database Migration Service | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/data-migration/sql-server/virtual-machines/database-migration-service-offline "Tutorial: Migrate SQL Server to SQL Server on an Azure Virtual Machine with Azure DMS (Offline) - Azure Database Migration Service | Microsoft Learn"
