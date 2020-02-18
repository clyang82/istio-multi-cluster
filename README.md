This repo is designed to install shared control plane (multi-network) in 2 eks clusters. The document is https://istio.io/docs/setup/install/multicluster/shared-gateways/
After installed and configurated successfully, deploy the https://github.com/GoogleCloudPlatform/microservices-demo for demo purpose.

## Notes for shared control plane (multi-network)
1. the `context` and `cluster` in .kube/config has some special words so that cannot be used directly. You have modify
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ...
    server: ...
  name: cyang-test2
- cluster:
    certificate-authority-data: ...
    server: ...
  name: cyang-test1
contexts:
- context:
    cluster: cyang-test2
    user: arn:aws:eks:us-east-2:675801125365:cluster/cyang-test2
  name: eks-us-east-2-cyang-test2
- context:
    cluster: cyang-test1
    user: arn:aws:eks:us-east-2:675801125365:cluster/cyang-test1
  name: eks-us-east-2-cyang-test1
current-context: eks-us-east-2-cyang-test1
kind: Config
preferences: {}
users:
- name: arn:aws:eks:us-east-2:675801125365:cluster/cyang-test2
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - us-east-2
      - eks
      - get-token
      - --cluster-name
      - cyang-test2
      command: aws
- name: arn:aws:eks:us-east-2:675801125365:cluster/cyang-test1
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      args:
      - --region
      - us-east-2
      - eks
      - get-token
      - --cluster-name
      - cyang-test1
      command: aws
```
2. enable telemetry with the following command
```
istioctl manifest generate -f install/kubernetes/operator/examples/multicluster/values-istio-multicluster-primary.yaml --set values.grafana.enabled=true --set values.kiali.enabled=true --set values.tracing.enabled=true --set values.kiali.createDemoSecret=true
```
3. Due to the endpoint cannot support FQDN(1.17 has endpointslice), so you have to input the ip address for mesh network configuration

4. There is not gateway for telemetry components by default, you have to enable to external access. Here is original task. I just put the manifest files are in istio/gateways


## Deploy microservices-demo

We use https://github.com/GoogleCloudPlatform/microservices-demo as an exmaple to demonstrate the multi-cluster capability.

1. install the applications in cluster1 and cluster2.

cluster1:
```
kubectl apply -f ./microservices-demo-release/kubernetes-manifests-cluster1.yaml --context=$CTX_CLUSTER1
kubectl apply -f ./microservices-demo-release/all-svcs.yaml --context=$CTX_CLUSTER1
kubectl apply -f ./microservices-demo-release/istio-manifests.yaml --context=$CTX_CLUSTER2

```
cluster2:
```
kubectl apply -f ./microservices-demo-release/kubernetes-manifests-cluster2.yaml --context=$CTX_CLUSTER2
kubectl apply -f ./microservices-demo-release/all-svcs.yaml --context=$CTX_CLUSTER2
kubectl apply -f ./microservices-demo-release/istio-manifests-cluster2.yaml --context=$CTX_CLUSTER2
```