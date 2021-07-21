This step is required before run spaceone-helm chart.

# Update Values

* mongo-shard.pem
* mongos-conf.yaml

# Mongo Shard Cluster ONLY
If you use mongodb shard cluster, run

~~~
kubectl create -f mongos-conf.yaml
~~~

# Shared configuration (ALL)

~~~
kubectl create -f shared.yaml
~~~
