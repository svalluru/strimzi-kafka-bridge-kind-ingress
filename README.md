# Setup  Strimzi Kafka Bridge on Kind cluster and expose it externally using nginx Ingress.

## Pre-reqs
- Setup docker engine
- Setup kubectl
- Setup kind

## Step 1 -  Create kind cluster named 'kafka' as below
```
cat <<EOF | ./kind create cluster --name kafka --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
EOF
```

## Step 2 - Deploy nginx 
`kubectl apply -f deploy-nginx.yaml`

## Step 3 - Before deploying Strimzi Kafka operator, let’s first create our kafka namespace:
	`kubectl create namespace kafka`

## Step 4 - Deploy Strimzi operator
`kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka	`

## Step 5 - Follow the deployment of the Strimzi Kafka operator:
`kubectl get pod -n kafka --watch`

## Step 6 - Provision the Apache Kafka cluster
```
   kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka 
   
   We now need to wait while Kubernetes starts the required pods, services and so on:
   
   kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka 
   ```

## Step 7 Send and receive messages
```
Once the cluster is running, you can run a simple producer to send messages to a Kafka topic (the topic will be automatically created):

kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.29.0-kafka-3.2.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic

And to receive them in a different terminal you can run:

kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.29.0-kafka-3.2.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
```
## Step 8 - Install Strimzi Kafka Bridge

` kubectl -n kafka apply -f https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/0.29.0/examples/bridge/kafka-bridge.yaml `

## Step 9 - Create an Ingress object for Kafka Bridge	
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kafka-bridge
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: my-bridge-bridge-service
            port:
              number: 8080
```
## Step 10 - Test if you can hit Kafka Bridge

```
curl localhost

Output : 
{"bridge_version":"0.21.5"}
```

### References : 
```
https://strimzi.io/quickstarts/
https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx
```

