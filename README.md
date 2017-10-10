# Create MongoDB on a kubernetes cluster using helm in Google Cloud Platform

## Create kubernetes cluster in the cloud provider
- In Google Cloud Platform:
	- Container Engine -> Container Cluster -> Create Kubernets cluster;
	- Using default config in this example: 3 nodes with 1 vCPU and 3.75 GB memory each;

### Install `gcloud`
https://cloud.google.com/sdk/downloads

## Install `helm`
https://github.com/kubernetes/helm/blob/master/docs/install.md

## Install `kubectl`
https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Gain access to the cluster
`gcloud container clusters get-credentials cluster-1 --zone <zone_name> --project <project_name>`

## Install MongoDB using helm chart
```
$ helm init;
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/;
$ helm install stable/mongodb-replicaset --version 1.4.1;
```
reference - https://kubeapps.com/charts/stable/mongodb-replicaset

## Find master node
`for ((i = 0; i < 3; ++i)); do kubectl exec --namespace default my-release-mongodb-replicaset-$i -- sh -c 'mongo --eval="printjson(rs.isMaster())"'; done`
```
{
	"hosts" : [
		"my-release-mongodb-replicaset-0.my-release-mongodb-replicaset.default.svc.cluster.local:27017",
		"my-release-mongodb-replicaset-1.my-release-mongodb-replicaset.default.svc.cluster.local:27017",
		"my-release-mongodb-replicaset-2.my-release-mongodb-replicaset.default.svc.cluster.local:27017"
	],
	"setName" : "rs0",
	"setVersion" : 3,
	"ismaster" : true,
	"secondary" : false,
	"primary" : "my-release-mongodb-replicaset-0.my-release-mongodb-replicaset.default.svc.cluster.local:27017",
	"me" : "my-release-mongodb-replicaset-0.my-release-mongodb-replicaset.default.svc.cluster.local:27017",
	"electionId" : ObjectId("7fffffff0000000000000002"),
	"lastWrite" : {
		"opTime" : {
			"ts" : Timestamp(1507246916, 1),
			"t" : NumberLong(2)
		},
		"lastWriteDate" : ISODate("2017-10-05T23:41:56Z")
	},
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2017-10-05T23:41:59.606Z"),
	"maxWireVersion" : 5,
	"minWireVersion" : 0,
	"readOnly" : false,
	"ok" : 1
}
```

# Test insert data in MongoDB
`kubectl exec --namespace default my-release-mongodb-replicaset-0 -- mongo --eval="printjson(db.test.insert({key1: 'value1'}))"`
```
MongoDB shell version v3.4.9
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.9
{ "nInserted" : 1 }
```

# Test read data from any node in Mongo
`kubectl exec --namespace default my-release-mongodb-replicaset-0 -- mongo --eval="rs.slaveOk(); db.test.find().forEach(printjson)"`
```
MongoDB shell version v3.4.9
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 3.4.9
{ "_id" : ObjectId("59d6c4adc37e28e1c76c5b2d"), "key1" : "value1" }
```
