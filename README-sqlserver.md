
# Debezium Microsoft SQL Server tutorial

These instructions assume you've already followed the instructions from the main README.</br>

Install SQL Server in the k8s cluster:
```
kubectl apply -f sqlserver-debezium.yaml
kubectl apply -f sqlserver-nodeport.yaml
```
Note that the SQL Server image used, does not contain tutorial data, like the MySQL example image.<br/>
To interact with the SQL Server from a Mac, it's recommended to install Azure Data Studio: https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio<br/>
Connect to minikube.local (replace with the host of your k8s cluster) port 1433, with user sa (='root') and the password from `sqlserver-debezium.yaml`.<br/>
Open and run `sqlserver-inventory.sql` with Azure Data Studio to put the tutorial data (same as used by the MySQL image) into the system, or execute the following statements on the commandline:
```
SQLSERVER_POD=`kubectl get pods --namespace demo |grep sqlserver |cut -c 1-38|xargs`
kubectl cp sqlserver-inventory.sql $SQLSERVER_POD:/tmp/ --namespace demo
kubectl exec --stdin --tty $SQLSERVER_POD --namespace demo -- /opt/mssql-tools/bin/sqlcmd -H localhost -U sa -P D3b3z17m -i /tmp/sqlserver-inventory.sql -o /tmp/result.txt
```
## CDC Prerequisites
- Enable agent on server (see yaml: MSSQL_AGENT_ENABLED=true)
- Use explicit hostname in K8s-cluster, otherwise uses only first 15 chars of automatically assigned hostname/podname (this is a bug that may be fixed in the future)
Note: the tutorial database script already contains the instructions below to enable CDC for the database and all tables:
- Enable snapshot isolation on DB:
```
ALTER DATABASE MyDatabase  
SET ALLOW_SNAPSHOT_ISOLATION ON
```
- Enable CDC on DB by executing stored procedure:
```
USE MyDatabase
EXEC sys.sp_cdc_enable_db;
```
- Enable CDC for each table by executing stored procedure:
```
EXEC sys.sp_cdc_enable_table @source_schema = 'dbo', @source_name = 'MyTable', @role_name = NULL, @supports_net_changes = 0;
```
