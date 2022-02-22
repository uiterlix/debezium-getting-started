
# Debezium tutorial

These instructions are a short version of the Debezium tutorial which can be found here: https://debezium.io/documentation/reference/tutorial.html<br/>
This tutorial provides the necessary Kubernetes configurations for the docker images used in the Debezium tutorial. To run this tutorial you'll need a k8s cluster
up and running, e.g. minikube or k3s.<br/>

Install the following tutorial containers

```
kubectl apply -f demo-ns.yaml
helm install -f kafka-values.yaml kafka bitnami/kafka --namespace demo
kubectl apply -f mysql-debezium.yaml
kubectl apply -f mysql-nodeport.yaml
kubectl apply -f kafka-connect.yaml
kubectl apply -f connect-nodeport.yaml
```
Verify the mysql database is up and running and contains demo content. Access the mysql pod shell and try the follwing:<br/>
```
kubectl exec --stdin --tty `kubectl get pods --namespace demo |grep mysql|cut -c1-36` /bin/bash --namespace demo
mysql -uroot -pdebezium
use inventory;
select * from customers;
```
Install the mysql inventory connector in debezium in the kafka connect container. Please note the "minikube.local" hostname in the fragments below, please replace that with the hostname or IP of your k8s cluster.

`curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" minikube.local:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'`

Verify the connector is installed.

`curl -H "Accept:application/json" minikube.local:8083/connectors/`

To delete the connector you can issue the following command:

`curl -i -X DELETE minikube.local:8083/connectors/inventory-connector/`

Check the created kafka topics. Either on the kafka pod 

```
kubectl exec --stdin --tty kafka-0 /bin/bash --namespace demo
cd /opt/bitnami/kafka/bin
./kafka-topics.sh --bootstrap-server localhost:9092 --list
```

or from your local shell.

`./kafka-topics.sh --bootstrap-server minikube.local:9092 --list`

Subscribe to the customers topic, on the kafka pod

```
kubectl exec --stdin --tty kafka-0 /bin/bash --namespace demo
cd /opt/bitnami/kafka/bin
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic dbserver1.inventory.customers --from-beginning
```

or on your local shell

`./kafka-console-consumer.sh --bootstrap-server minikube.local:9092 --topic dbserver1.inventory.customers --from-beginning`