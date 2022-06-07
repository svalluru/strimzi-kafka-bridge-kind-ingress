# Setup  Strimzi Kafka Bridge on Kind cluster and expose it externally using nginx Ingress.

# Pre-reqs
- Setup docker engine
- Setup kubectl
- Setup kind

# Step 1 -  Create kind cluster named 'kafka' as below
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

# Step 2 - Deploy nginx 
`kubectl apply -f deploy-nginx.yaml`
