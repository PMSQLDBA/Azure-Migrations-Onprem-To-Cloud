Consolidated cheatsheet based on the migration options, target-selection criteria, and scenarios in your document. 

# On-Prem SQL Server → Azure Migration CHEATSHEET

| S.No. | Requirement / Scenario                                    | Recommended Azure Target    | Migration Strategy     | Preferred Migration Method     | Mode             | Downtime     | Key Reason                          |
| ----: | --------------------------------------------------------- | --------------------------- | ---------------------- | ------------------------------ | ---------------- | ------------ | ----------------------------------- |
|     1 | New / cloud-native application                            | **Azure SQL Database**      | Modernize              | DMS / Schema + Data            | Offline          | Medium       | Maximum PaaS benefits               |
|     2 | Existing application can be refactored                    | **Azure SQL Database**      | Modernize              | DMS / Schema + Data            | Offline          | Medium       | Remove legacy SQL dependencies      |
|     3 | Simple standalone database                                | **Azure SQL Database**      | Modernize              | DMS                            | Offline          | Medium       | Fully managed PaaS                  |
|     4 | Small Dev/Test database                                   | **Azure SQL Database**      | Modernize              | BACPAC / Schema + Data         | Offline          | Medium       | Simple logical migration            |
|     5 | Minimum DBA/infrastructure management required            | **Azure SQL Database**      | Modernize              | DMS                            | Offline          | Medium       | Microsoft manages platform          |
|     6 | Serverless / elastic-pool requirement                     | **Azure SQL Database**      | Modernize              | DMS / Schema + Data            | Offline          | Medium       | SQL DB-specific PaaS capabilities   |
|     7 | Existing SQL application needs high SQL compatibility     | **Azure SQL MI**            | Managed Lift-and-Shift | Native Restore / DMS           | Offline          | Medium       | High SQL Server compatibility       |
|     8 | Existing `.bak` backup needs direct restore               | **Azure SQL MI**            | Managed Lift-and-Shift | `.bak` + `RESTORE FROM URL`    | Offline          | Medium–High  | Native backup restore supported     |
|     9 | SQL Agent heavily used                                    | **Azure SQL MI**            | Managed Lift-and-Shift | Native Restore / DMS           | Offline / Online | Medium       | SQL Agent supported                 |
|    10 | Cross-database queries required                           | **Azure SQL MI**            | Managed Lift-and-Shift | DMS / Native Restore           | Offline / Online | Medium       | Strong cross-DB compatibility       |
|    11 | Instance-level SQL dependencies but no OS access required | **Azure SQL MI**            | Managed Lift-and-Shift | DMS / LRS                      | Online / Offline | Low–Medium   | PaaS + high compatibility           |
|    12 | 500-GB DB + several-hour maintenance window               | **Azure SQL MI**            | Managed Lift-and-Shift | Native `.bak` Restore          | Offline          | Medium       | Simple migration approach           |
|    13 | Large database + minimal downtime                         | **Azure SQL MI**            | Managed Lift-and-Shift | **LRS**                        | Near-online      | **Low**      | Backup/log replay before cutover    |
|    14 | Near-zero downtime required                               | **Azure SQL MI**            | Managed Lift-and-Shift | **MI Link**                    | Online           | **Very Low** | Near-real-time synchronization      |
|    15 | Multi-TB mission-critical database                        | **Azure SQL MI**            | Managed Lift-and-Shift | MI Link / LRS                  | Online           | Low–Very Low | Pre-stage DB before cutover         |
|    16 | Azure Arc-enabled SQL estate                              | **Azure SQL MI**            | Managed Lift-and-Shift | Azure Arc Integrated Migration | Online           | Low          | Centralized migration experience    |
|    17 | Dev/Test database refresh using `.bak`                    | **Azure SQL MI**            | Managed Lift-and-Shift | Native Restore                 | Offline          | Medium       | Simple/repeatable restore           |
|    18 | Legacy application cannot be changed                      | **SQL Server on Azure VM**  | Rehost                 | Backup/Restore / DMS           | Offline / Online | Low–Medium   | Maximum compatibility               |
|    19 | Full Windows/OS access required                           | **SQL Server on Azure VM**  | Rehost                 | Azure Migrate                  | VM Replication   | Low–Medium   | Full OS control                     |
|    20 | Full SQL Server instance control required                 | **SQL Server on Azure VM**  | Rehost                 | DMS / Backup Restore           | Online / Offline | Low–Medium   | Full SQL administration             |
|    21 | Complex linked servers / CLR dependencies                 | **SQL Server on Azure VM**  | Rehost                 | DMS / Azure Migrate            | Online           | Low          | Maximum SQL compatibility           |
|    22 | Third-party agents installed on SQL Server                | **SQL Server on Azure VM**  | Rehost                 | Azure Migrate                  | VM Replication   | Low–Medium   | Preserves OS dependencies           |
|    23 | Entire SQL Server VM must move                            | **SQL Server on Azure VM**  | Rehost                 | **Azure Migrate**              | VM Replication   | Low–Medium   | Moves VM + OS + SQL                 |
|    24 | Production DB requires minimal downtime                   | **SQL Server on Azure VM**  | Rehost                 | **DMS Online**                 | Online           | Low          | Continuous migration before cutover |
|    25 | Mission-critical workload requiring very low downtime     | **SQL Server on Azure VM**  | Rehost                 | AG / Distributed AG            | Online           | **Very Low** | Continuous synchronization          |
|    26 | DBA-controlled traditional migration                      | **Azure SQL MI / Azure VM** | Lift-and-Shift         | Log Shipping                   | Near-online      | Low          | Familiar backup/log process         |
|    27 | DR-style migration                                        | **Azure SQL MI / Azure VM** | Lift-and-Shift         | LRS / AG / Backup Chain        | Near-online      | Low          | Controlled recovery/cutover         |
|    28 | Hundreds of SQL Servers need discovery first              | **DB / MI / VM**            | Assess First           | Azure assessment tooling       | Assessment       | N/A          | Select target after discovery       |

## Quick Target Selection

| Need                                 | Choose                            |
| ------------------------------------ | --------------------------------- |
| **Modernize**                        | 🟢 **Azure SQL Database**         |
| Maximum PaaS                         | 🟢 **Azure SQL Database**         |
| Lowest administration                | 🟢 **Azure SQL Database**         |
| Cloud-native application             | 🟢 **Azure SQL Database**         |
| Serverless / Elastic Pool            | 🟢 **Azure SQL Database**         |
| High SQL Server compatibility + PaaS | 🔵 **Azure SQL Managed Instance** |
| Native `.bak` restore                | 🔵 **Azure SQL MI**               |
| SQL Agent + cross-database workloads | 🔵 **Azure SQL MI**               |
| Minimal downtime to MI               | 🔵 **LRS**                        |
| Near-zero downtime to MI             | 🔵 **MI Link**                    |
| Full SQL Server compatibility        | 🟠 **SQL Server on Azure VM**     |
| Full OS access                       | 🟠 **SQL Server on Azure VM**     |
| Third-party/OS dependencies          | 🟠 **SQL Server on Azure VM**     |
| Move entire VM                       | 🟠 **Azure VM + Azure Migrate**   |

## Migration Method Cheat Code

| Downtime              | Azure SQL DB                | Azure SQL MI   | SQL Server Azure VM           |
| --------------------- | --------------------------- | -------------- | ----------------------------- |
| **Offline OK**        | DMS / Schema+Data           | `.bak` Restore | Backup/Restore                |
| **Low Downtime**      | Scenario-specific           | **LRS**        | **DMS Online / Log Shipping** |
| **Near-Zero**         | Requires specialized design | **MI Link**    | **AG / Distributed AG**       |
| **Whole Server Move** | N/A                         | N/A            | **Azure Migrate**             |

## Remember This

```text
                    ON-PREM SQL SERVER
                           │
                           ▼
                     ASSESS FIRST
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     MODERNIZE       MANAGED LIFT       REHOST
          │           & SHIFT              │
          ▼                ▼                ▼
   Azure SQL DB      Azure SQL MI       Azure VM
          │                │                │
          ▼                ▼                ▼
   DMS / Schema       Downtime?        Full DB only?
      + Data             │                 │
                    ┌────┴────┐       ┌────┴─────┐
                    │         │       │          │
                 Offline   Minimal   Yes       Whole VM
                    │         │       │          │
                    ▼         ▼       ▼          ▼
                  .BAK      LRS    DMS/BAK   Azure Migrate
                              │
                       Near-Zero?
                              │
                              ▼
                           MI LINK
```

**One-line architecture rule:** **Azure SQL Database = Modernize → Azure SQL MI = Managed Lift-and-Shift → SQL Server on Azure VM = Rehost.** This is also the classification used in your current material. 
