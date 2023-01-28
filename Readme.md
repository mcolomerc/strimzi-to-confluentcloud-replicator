# Strimzi to Confluent Cloud with Replicator Example

**Source**: Strimzi Kafka

**Destination**: Confluent Cloud Standard cluster

## Create Strimzi Kafka Cluster

1. Install Strimzi Operator

`helm repo add strimzi https://strimzi.io/charts/`
 
`helm install strimzi-kafka strimzi/strimzi-kafka-operator`

`kubectl get pods -l=name=strimzi-cluster-operator`

`kubectl get crd | grep strimzi`

2. Kafka Deployment 
   
`kubectl apply -f ./strimzi-kafka-deployment/kafka.yaml`

3. Create Topics

`kubectl apply -f ./strimzi-kafka-deployment/topics.yaml`

4. Produce and Consume 

```sh
kubectl run kafka-producer -ti \
  --image=quay.io/strimzi/kafka:0.32.0-kafka-3.2.0 \
  --rm=true \
  --restart=Never \
  -- bin/kafka-console-producer.sh \
  --broker-list strimzi-cluster-kafka-bootstrap:9092 \
  --topic topic1
```

```sh
kubectl run kafka-consumer -ti \
  --image=quay.io/strimzi/strimzi/kafka:0.32.0-kafka-3.2.0 \
  --rm=true \
  --restart=Never \
  -- bin/kafka-console-consumer.sh \
  --bootstrap-server strimzi-cluster-kafka-bootstrap:9092 \
  --topic topic1 \
  --from-beginning
```

Optional: Kafka-UI web tool

```sh
helm repo add kafka-ui https://provectus.github.io/kafka-ui 
helm install kafka-ui kafka-ui/kafka-ui --set envs.config.KAFKA_CLUSTERS_0_NAME=strimzi-cluster  --set envs.config.KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=strimzi-cluster-kafka-bootstrap:9092 --set service.type=LoadBalancer 
```

## Confluent Replicator 

1. Connfluent For Kubernetes deployment

Create the namespace for the confluent workloads
 
`kubectl create namespace confluent`

Setup the CFK Operator

```sh
helm repo add confluentinc https://packages.confluent.io/helm
helm repo update
helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes --namespace confluent  
```

2. Connect deployment

Add Confluent Cloud credentials `ccloud-credentials.txt`.

Create secret with Confluent Cloud credentials: `kubectl create secret generic ccloud-credentials --from-file=plain.txt=ccloud-credentials.txt`

Configure Confluent Cloud bootstrap server at `./confluent/connect.yaml`

Deploy Connect: `kubectl apply -f ./confluent/connect.yaml`

3. Replicator deployment

Configure Confluent Cloud bootstrap server and credentials at `./confluent/replicator.yaml`

`kubectl apply -f ./confluent/replicator.yaml`

```sh
kubectl get connector -n confluent
NAME         STATUS    CONNECTORSTATUS   TASKS-READY   AGE
replicator   CREATED   RUNNING           2/2           80s
```

[Replicator Configuration](https://docs.confluent.io/platform/current/multi-dc-deployments/replicator/configuration_options.html)

[Replicator Configuration Example](https://github.com/confluentinc/confluent-kubernetes-examples/blob/master/hybrid/replicator/connector.yaml)