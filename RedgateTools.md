# Welcome to my Redgate Tools Notes

We have a lot of tools at Redgate so it might be overwhelming at first. I recommend to work with a single tool to get used to the type of tickets then move on to the next once you've hit a good comfort level.

I recommend reading through the documentage page to get an understanding of the different tools. 

I highly recommend to download and try interfacing the tool (this goes a long way when trying to understand the tool). 
- [Documentation Page](https://documentation.red-gate.com/home)
- [Downloads Directory](https://download.red-gate.com/)

:rewind:[GO BACK TO ROOT DIRECTORY](https://github.com/daviddang-redgate/my-notes/):rewind:

---

## SQL Monitor
- Enable enhanved logging for 10 minutes. Run the following in the URL:
  http://127.0.0.1:8080/configuration/Logging/EnablePerformanceLogging?enabledSeconds=600
  *Note: replace SQL Monitor URL with theirs if they are using a domain (i.e., company.com:8080)*

- Query to Data Repository to see version
  ```
  SELECT  Id ,
          Date ,
          CodeVersion ,
          SchemaVersion
  FROM versioning.SchemaVersionHistory
  ```

- Query to check connection for Base Monitor and Data Repository
  ```
  USE RedGateMonitor
  GO
  SELECT * FROM settings.LicenceActivations
  CREATE TABLE #temp (
        SPID INT
      , Status VARCHAR(1000) NULL
      , Login SYSNAME NULL
      , HostName SYSNAME NULL
      , BlkBy SYSNAME NULL
      , DBName SYSNAME NULL
      , Command VARCHAR(1000) NULL
      , CPUTime INT NULL
      , DiskIO INT NULL
      , LastBatch VARCHAR(1000) NULL
      , ProgramName VARCHAR(1000) NULL
      , SPID2 INT
      , REQUESTID INT NULL
      )
  INSERT  INTO #temp
          EXECUTE sys.sp_who2
  SELECT  *
  FROM    #temp
  WHERE   DBName = N'RedGateMonitor'
  DROP TABLE #temp
  ```

- Remove SQL Monitor Active Directory Admin and Password entirely (use with caution)
  ```
  DELETE FROM settings.UserAccount WHERE UserName = 'admin'
  UPDATE settings.KeyValuePairs SET KeyValue = 0 WHERE KeyName = 'ActiveDirectory-Enabled'
  DELETE FROM settings.ActiveDirectoryDomains
  DELETE FROM settings.ActiveDirectoryPrincipal
  DELETE FROM settings.ActiveDirectoryPrincipalAuthorisation
  ```
  Then restart the SQL Monitor Web Service or IIS that is hosting the site.

- Remove License Entries from SQL Monitor.
  1. Query below to see and store the results.
  ```
  SELECT * FROM settings.LicenceActivations
  ```
  2. Remove any entries from the settings.LicenceActivations table and restart the base monitor

- SQL Monitor Sample PowerShell Script to Connect SQL Monitor
  ```
  using module .\RedgateSQM
  $ErrorActionPreference = 'Stop'
  Connect-SqlMonitor -ServerUrl 'http://127.0.0.1:8080' -AuthToken 'MToyYTQ0ZGM0My0wNjRlLTQ4ZWMtYWVhNi05YTMwYTQ3NWViYjI='

  $baseMonitorName = 'localhost'
  $baseMonitor = Get-SqlMonitorBaseMonitor -Name $baseMonitorName
  <# $monitoredObjects = Get-SqlMonitorMachine -BaseMonitor $baseMonitor #>
  $monitoredObjects = Get-SqlMonitorCluster -BaseMonitor $baseMonitor

  $monitoredObjects

  $cluster = 'abcluster.testnet.red-gate.com'
  $cluster2 = 'sqlmonitor-web.testnet.red-gate.com'
  $addObjects = Add-SqlMonitorMonitoredObject -MonitoredObject $cluster -BaseMonitor $BaseMonitor -AutoDetectClusterName 0
  ```

- Troubleshoot No Names for SQL Monitor > Estate > Disk Usage
  1. Log in to the cluster, run command below in PowerShell and send a screenshot of the results back to us
  ```
  Get-WmiObject -Namespace root\MSCluster -ClassName MSCluster_DiskPartition
  ```
  3. Run the query below in the SQM repository and send the results back to us in csv format
  ```
  SELECT ck._Name, ccs._MountPoints, ccs._TotalBytes, ccs._VolumeLabel, ccs.CollectionDate  FROM [SQM].[data].[Cluster_ClusterSharedVolumes_StableSamples] AS ccs
    JOIN [SQM].[data].[Cluster_ClusterSharedVolumes_Keys] AS cck
    ON ccs.Id = cck.Id
    JOIN [SQM].[data].[Cluster_Keys] AS ck
    ON cck.ParentId = ck.Id
  ```
  3. Save the result of the request to the http://{*your_domain*}/api/estate/disk-usage/disks as HAR file and send it to us. They can find it on the Monitor > Estate > Disk Usage page under the Network tab of the DevTools window (which can be opened by pressing F12 in Chrome browser):

- Correlated Deadlocks Explanation (Alex)
  For correlated deadlocks, there can be deadlocks raised when threads are merged back into the main thread of the sql process that is doing something in parallel and that can raise deadlocks at times that may seem related. Otherwise it's usually a blocking chain that is happening (x blocks y, y blocks z, z blocks a) and the lead blocker needs to unblock to free up the entire chain.

---

## SQL Change Automation
- SQL Change Automation is a beast if you have not worked with any CI/CD.
- I recommend following the documentation on creating your on Build and Release pipeline.
- 
---

## SQL Source Control
- **Slowness while navigating in Object Explorer:** SQL Source Control Options - SQL Source Control tab > Options > Uncheck the options (particularly try the "Indicate changed objects in the Object Explorer...." option)

---

## SQL Prompt
- **Issues on startup:** SQL Prompt's Tab History - SQL Prompt > Options > Tabs > History > Disable Tab history
- **Slowness while working in the query editor:** SQL Prompt's Suggestions and/or Code Analysis - SQL Prompt > Uncheck "Enable Suggestions"/"Enable Code Analysis"

---

## General Notes
- Always try to have customer provide "Verbose" or "Debug" logging/log files (this provides more details/logging of events to help with troubleshoot)
