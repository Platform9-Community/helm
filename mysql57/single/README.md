
# Bootstrap simple stateful Mysql DB on Platform9 Managed Kubernetes 4.5 with helm v3


MySQL is one of the most popular database servers. Here we are going to bootstrap a Mysql 5.7 database server on top of Platform9 Managed kubernetes 4.5.2 with helm. 

## Prerequisites
1. Dynamic persistent volume provisioning with CSI driver via [Rook](https://github.com/KoolKubernetes/csi/tree/master/rook/) or a suitable CSI volume provisioner.
2. [Platform9 Managed Kubernetes 4.5](https://docs.platform9.com/release-notes/)
3. [Helm version 3](https://helm.sh/docs/intro/install/)
4. [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) v1.17.0

## Bootstrapping MySQL
Verify Helm v3 is present and kubeconfig context is set to the right cluster.

```bash
$ helm version
version.BuildInfo{Version:"v3.3.4", GitCommit:"a61ce5633af99708171414353ed49547cf05013d", GitTreeState:"clean", GoVersion:"go1.14.9"}

$ kubectl config get-contexts
CURRENT   NAME      CLUSTER               AUTHINFO            NAMESPACE
*         default   cl6.platform9.horse   user@platform9.com   default

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.9", GitCommit:"4fb7ed12476d57b8437ada90b4f93b17ffaeed99", GitTreeState:"clean", BuildDate:"2020-07-15T16:18:16Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.6", GitCommit:"d32e40e20d167e103faf894261614c5b45c44198", GitTreeState:"clean", BuildDate:"2020-05-20T13:08:34Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

Add MySQL stable helm chart repository into helm v3.
```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Verify the repo has been added.
```bash
$ helm repo list
NAME         	URL
stable       	https://kubernetes-charts.storage.googleapis.com/
```

Update the helm repo
```bash
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

List the MySQL versions available on the repo. To list down all available MySQL chart versions add '--versions' flag at the end of the command.
```bash
$ helm search repo stable/mysql --max-col-width=0
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
stable/mysql    	1.6.7        	5.7.30     	Fast, reliable, scalable, and easy to use open-source relational database system.
stable/mysqldump	2.6.1        	2.4.1      	A Helm chart to help backup MySQL databases using mysqldump
```

Bootstrap MySQL helm chart. Add '--debug' flag at the end of the command for verbose logging during the command execution.
```bash
$ helm install mysql stable/mysql -n mysql --create-namespace
```
Verify the release is in deployed state
```bash
$ helm ls -n mysql
NAME 	NAMESPACE	REVISION	UPDATED                            	STATUS  	CHART      	APP VERSION
mysql	mysql    	1       	2020-10-28 14:36:13.70676 +0530 IST	deployed	mysql-1.6.7	5.7.30
```

Fetch the MySQL root password from kubernetes secret:
```bash
kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo
```

Test the connection to the MySQL DB within kubernetes cluster by running a MySQL client pod.
```bash
$ kubectl run -it --rm --image=mysql:5.7 --restart=Never mysql-client -n mysql -- mysql -h mysql.mysql.svc.cluster.local -p<paste-MySQL-root-password>
If you don't see a command prompt, try pressing enter.

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
31 rows in set (0.00 sec)

mysql> \q
Bye
pod "mysql-client" deleted
```

## References
https://github.com/helm/charts/tree/master/stable/mysql