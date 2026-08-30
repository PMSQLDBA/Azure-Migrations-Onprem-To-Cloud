### 1. Migration Strategy Overview

Start by defining the three architecture choices:

| S.No. | Migration Strategy         | Azure Target               | Primary Goal                                        | Typical Scenario                        |
| ----: | -------------------------- | -------------------------- | --------------------------------------------------- | --------------------------------------- |
|     1 | **Rehost**                 | SQL Server on Azure VM     | Move with minimum change                            | Legacy / tightly coupled SQL workloads  |
|     2 | **Managed Lift-and-Shift** | Azure SQL Managed Instance | Move to PaaS while retaining high SQL compatibility | Existing enterprise SQL applications    |
|     3 | **Modernize / Refactor**   | Azure SQL Database         | Maximum PaaS benefits                               | Cloud-native or modernized applications |

This matches the high-level classification already captured in your document. 

---

### 2. Target Selection

Next answer **“Where should the database move?”**

| S.No. | Requirement                                               | Recommended Target             |
| ----: | --------------------------------------------------------- | ------------------------------ |
|     1 | Cloud-native/new application                              | **Azure SQL Database**         |
|     2 | Maximum PaaS benefits                                     | **Azure SQL Database**         |
|     3 | Minimum database administration                           | **Azure SQL Database**         |
|     4 | Existing SQL Server workload requiring high compatibility | **Azure SQL Managed Instance** |
|     5 | Native `.bak` restore required without managing SQL VMs   | **Azure SQL Managed Instance** |
|     6 | SQL Agent and cross-database functionality required       | **Azure SQL Managed Instance** |
|     7 | SQL instance features needed but OS access isn't required | **Azure SQL Managed Instance** |
|     8 | Application cannot easily be changed                      | **SQL Server on Azure VM**     |
|     9 | Windows/OS access required                                | **SQL Server on Azure VM**     |
|    10 | Full SQL Server instance control required                 | **SQL Server on Azure VM**     |
|    11 | Third-party SQL/Windows agents required                   | **SQL Server on Azure VM**     |
|    12 | Entire existing SQL Server VM must move                   | **SQL Server on Azure VM**     |

Your existing decision table already identifies these distinctions, including native backup restore, SQL Agent, cross-database queries, OS access and instance control. 

---

### 3. Target Capability Comparison

After identifying the targets, provide the detailed comparison.

| S.No. | Capability                | Azure SQL Database                | Azure SQL Managed Instance | SQL Server on Azure VM    |
| ----: | ------------------------- | --------------------------------- | -------------------------- | ------------------------- |
|     1 | Service Model             | PaaS                              | PaaS                       | IaaS                      |
|     2 | SQL compatibility         | Moderate                          | **High**                   | **Highest**               |
|     3 | Native `.bak` restore     | No                                | **Yes**                    | **Yes**                   |
|     4 | SQL Server Agent          | Alternative required              | **Yes**                    | **Yes**                   |
|     5 | Cross-database workloads  | Limited / redesign                | **Strong**                 | **Full**                  |
|     6 | Linked servers            | Limited                           | Supported with limitations | **Full**                  |
|     7 | OS access                 | No                                | No                         | **Yes**                   |
|     8 | Full instance control     | No                                | Managed/Partial            | **Yes**                   |
|     9 | Infrastructure management | Microsoft                         | Microsoft                  | Customer                  |
|    10 | Database engine patching  | Microsoft                         | Microsoft                  | Customer / Azure-assisted |
|    11 | Built-in platform HA      | **Yes**                           | **Yes**                    | Must be architected       |
|    12 | Serverless option         | **Yes, supported configurations** | No                         | No                        |
|    13 | Elastic Pool              | **Yes**                           | No                         | No                        |
|    14 | Application changes       | Potentially significant           | Usually lower              | **Lowest**                |
|    15 | Modernization potential   | **Highest**                       | High                       | Low                       |
|    16 | Management overhead       | **Lowest**                        | Low                        | **Highest**               |
|    17 | Customization             | Low                               | Medium                     | **Highest**               |
|    18 | Best migration strategy   | Modernize                         | Managed Lift-and-Shift     | Rehost                    |

---

# 4. Migration Methods — Segregate by Target

This is where your existing 15 migration methods should be reorganized. Currently they appear in one table. 

## 4.1 On-Prem SQL Server → Azure SQL Database

| S.No. | Migration Method                         | Mode       | Downtime           | Best Use Case                      |
| ----: | ---------------------------------------- | ---------- | ------------------ | ---------------------------------- |
|     1 | Azure Database Migration Service         | Offline    | Medium–High        | Enterprise database modernization  |
|     2 | Schema + Data Migration                  | Offline    | Medium             | Small/medium controlled migrations |
|     3 | BACPAC Export/Import                     | Offline    | High for large DBs | Small DB / Dev/Test                |
|     4 | Transactional Replication / Sync pattern | Online-ish | Low                | Specialized phased migration       |

**Recommended when:** application/database can be modernized and SQL Server instance dependencies can be removed.

---

## 4.2 On-Prem SQL Server → Azure SQL Managed Instance

| S.No. | Migration Method                                          | Mode                                             | Downtime     | Best Use Case                            |
| ----: | --------------------------------------------------------- | ------------------------------------------------ | ------------ | ---------------------------------------- |
|     1 | Native `.bak` + Restore from URL                          | Offline                                          | Medium–High  | Simple migration with maintenance window |
|     2 | Azure DMS                                                 | Online / Offline depending on supported scenario | Low–Medium   | Managed enterprise migration             |
|     3 | Log Replay Service (LRS)                                  | Near-online                                      | **Low**      | Large DB / minimal downtime              |
|     4 | Managed Instance Link                                     | Online                                           | **Very Low** | Mission-critical / near-zero downtime    |
|     5 | Azure Arc Integrated Migration                            | Online / Minimal downtime                        | Low          | Large Azure Arc-enabled SQL estates      |
|     6 | Log Shipping / backup-log based approach where applicable | Near-online                                      | Low          | DBA-controlled migration                 |

Your source correctly highlights native restore as the simpler option when downtime is acceptable and MI Link/LRS for lower-downtime migrations. 

---

## 4.3 On-Prem SQL Server → SQL Server on Azure VM

| S.No. | Migration Method           | Mode           | Downtime     | Best Use Case                         |
| ----: | -------------------------- | -------------- | ------------ | ------------------------------------- |
|     1 | Native Backup & Restore    | Offline        | Medium–High  | Simple database lift-and-shift        |
|     2 | Azure DMS Online           | Online         | **Low**      | Production minimal-downtime migration |
|     3 | Azure DMS Offline          | Offline        | Medium       | Managed migration with downtime       |
|     4 | Always On / Distributed AG | Online         | **Very Low** | Mission-critical workloads            |
|     5 | Azure Migrate              | VM Replication | Low–Medium   | Move entire SQL Server VM             |
|     6 | Log Shipping               | Near-online    | Low          | Traditional DBA-controlled migration  |

---

# 5. Select Migration Method Based on Downtime

Once the target is selected, **downtime should drive the migration method**.

| S.No. | Downtime Requirement | SQL Database                      | SQL Managed Instance  | SQL Server Azure VM           |
| ----: | -------------------- | --------------------------------- | --------------------- | ----------------------------- |
|     1 | Downtime acceptable  | DMS Offline / Schema+Data         | Native Backup/Restore | Backup/Restore                |
|     2 | Moderate downtime    | DMS / controlled migration        | DMS / LRS             | DMS Offline                   |
|     3 | Minimal downtime     | Specialized approach              | **LRS**               | **DMS Online / Log Shipping** |
|     4 | Near-zero downtime   | Requires scenario-specific design | **MI Link**           | **AG / Distributed AG**       |

This is important because **target selection and migration-method selection are two separate decisions**.

---

# 6. Select Based on Database Size

Then consider database size.

| S.No. | Database Size | Typical Approach                                          |
| ----: | ------------- | --------------------------------------------------------- |
|     1 | < 100 GB      | Most migration methods are practical                      |
|     2 | 100–500 GB    | DMS / Native Restore / controlled migration               |
|     3 | 500 GB–1 TB   | Native Restore, DMS, LRS depending on downtime            |
|     4 | 1–5 TB        | LRS / MI Link / DMS / optimized backup strategy           |
|     5 | > 5 TB        | Detailed architecture and performance assessment required |

These should be treated as **planning guidance rather than Azure service limits**. Actual migration time depends on network bandwidth, backup compression, storage throughput, workload, service tier, and migration technology.

---

# 7. Real-World Migration Scenarios

Now put your use cases **after** the architecture and method selection.

| S.No. | Scenario                               | Target       | Method                                       |
| ----: | -------------------------------------- | ------------ | -------------------------------------------- |
|     1 | Small standalone application           | Azure SQL DB | DMS / Schema+Data                            |
|     2 | Cloud-native application               | Azure SQL DB | Schema+Data                                  |
|     3 | Legacy app using SQL Agent             | Azure SQL MI | Native Restore / LRS                         |
|     4 | Cross-database enterprise application  | Azure SQL MI | DMS / LRS                                    |
|     5 | 500-GB DB + 6-hour window              | Azure SQL MI | Native `.bak` Restore                        |
|     6 | 5-TB DB + 15-minute window             | Azure SQL MI | MI Link / LRS                                |
|     7 | Near-zero downtime                     | Azure SQL MI | MI Link                                      |
|     8 | Very large DB + minimal downtime       | Azure SQL MI | LRS / MI Link                                |
|     9 | Application cannot be changed          | Azure SQL VM | Backup/Restore / DMS                         |
|    10 | Full OS access required                | Azure SQL VM | DMS / Azure Migrate                          |
|    11 | Entire existing VM must move           | Azure SQL VM | Azure Migrate                                |
|    12 | Complex CLR / third-party dependencies | Azure SQL VM | DMS / Azure Migrate                          |
|    13 | Dev/Test DB refresh                    | Azure SQL MI | Native `.bak` Restore                        |
|    14 | Large SQL estate                       | DB / MI / VM | Azure assessment → target-specific migration |
|    15 | DR-style migration                     | MI / VM      | LRS / AG / Backup Chain                      |

This reorganizes the scenarios you already have rather than mixing them into target selection. 

---

# 8. Final Sequential Decision Flow

The document should finish with one clear decision process:

```text
1. DISCOVER
   │
   ├── SQL Servers
   ├── Databases
   ├── Versions / Editions
   ├── Database Sizes
   └── Application Dependencies
   │
   ▼
2. ASSESS
   │
   ├── SQL Agent
   ├── Linked Servers
   ├── Cross-Database Dependencies
   ├── CLR / SSIS / OS Dependencies
   └── Compatibility
   │
   ▼
3. SELECT MIGRATION STRATEGY
   │
   ├── Rehost
   ├── Managed Lift-and-Shift
   └── Modernize
   │
   ▼
4. SELECT AZURE TARGET
   │
   ├── Azure SQL Database
   │      └── Modernize
   │
   ├── Azure SQL Managed Instance
   │      └── Managed Lift-and-Shift
   │
   └── SQL Server on Azure VM
          └── Rehost
   │
   ▼
5. DETERMINE DOWNTIME REQUIREMENT
   │
   ├── Offline
   ├── Minimal Downtime
   └── Near-Zero Downtime
   │
   ▼
6. SELECT MIGRATION METHOD
   │
   ├── DMS
   ├── Native Backup / Restore
   ├── LRS
   ├── MI Link
   ├── Azure Migrate
   ├── Log Shipping
   └── AG / Distributed AG
   │
   ▼
7. DESIGN TARGET
   │
   ├── Compute / SKU
   ├── Storage
   ├── Networking
   ├── Private Connectivity
   ├── Authentication
   ├── HA / DR
   └── Security
   │
   ▼
8. TEST MIGRATION
   │
   ├── Measure Migration Time
   ├── Validate Data
   ├── Validate Application
   └── Test Rollback
   │
   ▼
9. PRODUCTION MIGRATION
   │
   ├── Stop / Control Writes
   ├── Final Synchronization
   ├── Cutover
   └── Connection Change
   │
   ▼
10. VALIDATE
    │
    ├── Database
    ├── Application
    ├── Performance
    ├── Security
    └── Business Validation
    │
    ▼
11. STABILIZE
    │
    ├── Monitor
    ├── Performance Tune
    └── Resolve Issues
    │
    ▼
12. DECOMMISSION SOURCE
```

### Recommended document order

**1. Overview → 2. Migration Strategies → 3. Target Selection → 4. Target Capability Comparison → 5. Migration Methods by Target → 6. Downtime-Based Selection → 7. Database Size Considerations → 8. Real-World Scenarios → 9. Migration Decision Flow → 10. Assessment → 11. Target Design → 12. Test Migration → 13. Production Cutover → 14. Validation → 15. Rollback → 16. Stabilization → 17. Source Decommissioning.**

Migration decision flow

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
