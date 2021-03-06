# POC - Cloudera API 
## Objectives
* How to retrieve Impala's full text queries?
* How to retrieve Hive's full text queries?
* How to retrieve text queries made by program applications? (ex. Spark _jobs_)
  ...
  
## Cloudera Manager 
* API Rest Doc: `https://cloudera.github.io/cm_api/apidocs/v11/index.html`
* Testing at Quickstart VM:
 ```
 curl -u cloudera:cloudera -X GET http://quickstart.cloudera:7180/api/v11/clusters
 curl -u cloudera:cloudera -X GET http://quickstart.cloudera:7180/api/v11/clusters/Cloudera%20QuickStart/services/
 curl -u cloudera:cloudera -X GET "http://quickstart.cloudera:7180/api/v11/cm/service/"
 curl -u cloudera:cloudera -X GET "http://quickstart.cloudera:7180/api/v11/users/"
 curl -u cloudera:cloudera -X GET "http://quickstart.cloudera:7180/api/v11/clusters/Cloudera%20QuickStart/services/"
```
* **Retrieving content queries via Impala**:
Query:
```
 curl -u cloudera:cloudera -X GET  http://quickstart.cloudera:7180/api/v11/clusters/Cloudera%20QuickStart/services/impala/impalaQueries
```
Output (query content in `statement` field) : 
```json
{
  "queries" : [ {
    "queryId" : "b748fb153f6d24d2:898315600000000",
    "statement" : "SELECT * FROM db.table1",
    "queryType" : "N/A",
    "queryState" : "EXCEPTION",
    "startTime" : "2018-04-23T21:01:28.939Z",
    "endTime" : "2018-04-23T21:01:29.582Z",
    "rowsProduced" : null,
    "attributes" : {
      "admission_result" : "Unknown",
      "session_id" : "9f4b93d417729234:5796113b035c7e97",
      "oom" : "false",
      "stats_missing" : "false",
      "connected_user" : "cloudera",
      "session_type" : "HIVESERVER2",
      "network_address" : "10.0.2.15:42032",
      "impala_version" : "impalad version 2.9.0-cdh5.12.0 RELEASE (build 03c6ddbdcec39238be4f5b14a300d5c4f576097e)",
      "query_status" : "AnalysisException: Could not resolve table reference: 'db1.tb1'\n",
      "file_formats" : ""
    },
    "user" : "cloudera",
    "coordinator" : {
      "hostId" : "quickstart.cloudera"
    },
    "detailsAvailable" : true,
    "database" : "default",
    "durationMillis" : 643
  }
```
* Viewing hive queries via YARN Applications (Not Rest API):
  * Access at `Clusters -> Yarn Applications` and filter by user(hive), dates, etc.
  * Access at `Diagnostics -> Log` and filter by source(hive), dates, etc.
  * Via HiveServer2 web UI: `http://<host>:<port>/hiveserver2.jsp`
    *  It can be accessed via CM: `https://www.cloudera.com/documentation/enterprise/5-11-x/topics/cm_mc_hive_webui.html`
  * Hue History.
  
* **Retrieving hive queries via CM REST API (YARN Applications)**:
 ```
 curl -u semantix:1\&PnQBKE1YMg -v -X GET http://ec2-18-204-108-136.compute-1.amazonaws.com:7180/api/v16/clusters/Cluster-Poc001/services/CD-YARN-CTJqbtLF/yarnApplications?from=17-04-18&to=now&filter='user=hive'
 ```
   Output about a hive yarn application:
   ```json
    {
    "applicationId" : "job_1524687286344_0002",
    "name" : "INSERT INTO TABLE ale...137','destination2')(Stage-1)",
    "startTime" : "2018-04-25T21:39:04.002Z",
    "endTime" : "2018-04-25T21:39:09.774Z",
    "user" : "admin",
    "pool" : "root.users.admin",
    "state" : "SUCCEEDED",
    "attributes" : {
      "rack_local_maps_percentage" : "100",
      "map_counter:org.apache.hadoop.mapreduce.filesystemcounter.hdfs_write_ops" : "3",
      "map_counter:org.apache.hadoop.mapreduce.taskcounter.split_raw_bytes" : "294",
      "reduce_counter:org.apache.hadoop.mapreduce.filesystemcounter.file_bytes_written" : "0",
      "map_counter:hive.records_out_1_alertas.empregado" : "1",
      "reduce_counter:hive.deserialize_errors" : "0",
      "reduce_counter:org.apache.hadoop.mapreduce.taskcounter.spilled_records" : "0",
      "counter:org.apache.hadoop.mapreduce.jobcounter.rack_local_maps" : "1",
      "reduce_counter:org.apache.hadoop.mapreduce.jobcounter.slots_millis_maps" : "0",
      "total_launched_maps" : "1",
      "reduce_counter:org.apache.hadoop.mapreduce.taskcounter.cpu_milliseconds" : "0",
      "cm_cpu_milliseconds" : "1600",
      "counter:org.apache.hadoop.mapreduce.jobcounter.millis_maps" : "3541",
      "counter:hive.deserialize_errors" : "0",
      "map_counter:org.apache.hadoop.mapreduce.jobcounter.vcores_millis_maps" : "0",
      "counter:org.apache.hadoop.mapreduce.filesystemcounter.file_bytes_written" : "291355",
      "hive_query_string" : "INSERT INTO TABLE alertas.empregado(id, name, salary, destination) \nVALUES(13,'Luara','137','destination2')",
      "physical_memory_bytes" : "274825216",
      "reduce_counter:org.apache.hadoop.mapreduce.jobcounter.vcores_millis_maps" : "0",
      ...
      }
   ```
   * *Remarks*: 
     *  `Cluster-Poc001` is the cluster name and `CD-YARN-CTJqbtLF`is the yarn service name.
     *  The complete query is located into *`hive_query_string`* attribute.
     *  To get the applications about Hive, it's important use the dates parameters (`from` and `to`)  and the `hive` user in the filter parameter.
     *  These queries obtained correspond to those that have become MR jobs.
      
    
## Cloudera Manager API Client
* Cloudera Manager API Client (cm_api) Repository: `https://github.com/cloudera/cm_api`
* Available languages: Java and Python.

### CM_API in Python 
* Documentation: `https://cloudera.github.io/cm_api/docs/python-client/`
* Install `cm_api`: `sudo pip install cm-api`
* Testing the CM API in `/scripts` of this repository.  

## Cloudera Navigator
* We can get the *Hive* or *Impala* queries on the search panel, and filtering by the SERVICE_NAME

* Testing REST API:

* **Retrieving content queries via Impala**:
Query: 
SERVICE_NAME==*IMPALA*
startTime = Sat Jun 29 18:41:39 -03 2002  #should be in miliseconds (seconds / 1000)
endTime = Thu May  3 19:34:59 -03 2018    #should be in miliseconds (seconds / 1000)
limit=10
```
curl -X GET -u semantix:1\&PnQBKE1YMg "http://ec2-18-204-108-136.compute-1.amazonaws.com:7187/api/v13/audits/?query=SERVICE_NAME%3D%3D*IMPALA*&startTime=1025386899000&endTime=1525386899000&limit=10&offset=0&format=JSON&attachment=false"
```
Output (query content in `operation_text` field) : 
```
[ {
  "timestamp" : "2018-04-26T16:50:56.000Z",
  "service" : "CD-IMPALA-JvZLYwfC",
  "username" : "admin",
  "ipAddress" : "172.31.90.138",
  "command" : "USE",
  "resource" : "null:null",
  "operationText" : "USE `default`",
  "allowed" : true,
  "serviceValues" : {
    "object_type" : null,
    "database_name" : null,
    "session_id" : "984984942cccb8f6:b7cbc9a311fe17a6",
    "operation_text" : "USE `default`",
    "status" : "",
    "privilege" : null,
    "table_name" : null,
    "query_id" : "d34440aa3837e19d:6980fecc00000000"
  }
}, {
  "timestamp" : "2018-04-26T16:50:56.000Z",
  "service" : "CD-IMPALA-JvZLYwfC",
  "username" : "admin",
  "ipAddress" : "172.31.90.138",
  "command" : "QUERY",
  "resource" : "alertas:empregado",
  "operationText" : "select * from alertas.empregado limit 7",
  "allowed" : true,
  "serviceValues" : {
    "object_type" : "TABLE",
    "database_name" : "alertas",
    "session_id" : "984984942cccb8f6:b7cbc9a311fe17a6",
    "operation_text" : "select * from alertas.empregado limit 7",
    "status" : "",
    "privilege" : "SELECT",
    "table_name" : "empregado",
    "query_id" : "d04a29f028188699:b618b5e100000000"
  }
}, 
```
* **Retrieving content queries via Hive**:
Query: 
SERVICE_NAME==*HIVE*
startTime = Sat Jun 29 18:41:39 -03 2002     #should be in miliseconds (seconds / 1000)
endTime = Thu May  3 19:34:59 -03 2018       #should be in miliseconds (seconds / 1000)
limit=10
```
curl -X GET -u semantix:1\&PnQBKE1YMg "http://ec2-18-204-108-136.compute-1.amazonaws.com:7187/api/v13/audits/?query=SERVICE_NAME%3D%3D*HIVE*&startTime=1025386899000&endTime=1525386899000&limit=10&offset=0&format=JSON&attachment=false"
```
Output (query content in `operation_text` field) : 
```
[ {
  "timestamp" : "2018-04-26T13:47:12.810Z",
  "service" : "CD-HIVE-gybetoEX",
  "username" : "admin",
  "ipAddress" : "172.31.90.138",
  "command" : "QUERY",
  "resource" : "alertas:empregado",
  "operationText" : "select * from alertas.empregado order by salary",
  "allowed" : true,
  "serviceValues" : {
    "object_type" : "TABLE",
    "database_name" : "alertas",
    "operation_text" : "select * from alertas.empregado order by salary",
    "resource_path" : "/user/hive/warehouse/alertas.db/empregado",
    "table_name" : "empregado"
  }
}, {
  "timestamp" : "2018-04-26T13:47:12.810Z",
  "service" : "CD-HIVE-gybetoEX",
  "username" : "admin",
  "ipAddress" : "172.31.90.138",
  "command" : "QUERY",
  "resource" : ":",
  "operationText" : "select * from alertas.empregado order by salary",
  "allowed" : true,
  "serviceValues" : {
    "object_type" : "DFS_DIR",
    "database_name" : "",
    "operation_text" : "select * from alertas.empregado order by salary",
    "resource_path" : "/tmp/hive/admin/a309768c-639e-47a9-833c-aea698e382f4/hive_2018-04-26_13-47-12_843_5439617981827652357-1/-mr-10000",
    "table_name" : ""
  }
}, 
```



## Spark's logging queries
* The YARN applications (via API REST or web app) about Spark's jobs don't show the used Hive's queries.

## MISC
* Convert date to timestamp in bash
```
date "+%s" -d "02/20/2013 08:41:15"
date "+%s" -d now
```

* Convert timestamp (in seconds) to date in bash
```
date -d @1025386899
```
