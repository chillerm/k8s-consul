# k8s-consul

Repository to work through Hashicorp examples and play with Consul running on GKE.

**Note:** These instructions are shorthand for myself to keep track of the steps I followed.  For the Full Tutorial, check out the [hashicorp official tutorial](https://developer.hashicorp.com/consul/tutorials/kubernetes/kubernetes-gke-google).

# Setup

## Install dependencies
I will assume since you are working with GKE you already have the GCloud CLI installed.  You can find brew directions [here](https://formulae.brew.sh/cask/google-cloud-sdk).


```bash
brew install helm
brew install consul
```

## Clean Up
Since running a gke cluster can run up costs, we'll mention what you should do even if you do not finish the tutorial.  Running the below will remove your GCP infrastructure.

```bash
./script/delete_cluster
```

## Configure cluster

```bash
# Create a cluster
./script/create_cluster

# Configure Cluster
./script/configure_consul

# Validate Cluster is up and consul is running
kubectl get pods --namespace consul
```

This should give you an output similar to below

```bash
➜  k8s-consul git:(main) ✗ kubectl get pods --namespace consul
NAME                                           READY   STATUS    RESTARTS   AGE
consul-connect-injector-6d57b47476-kqnnx       1/1     Running   0          70s
consul-server-0                                1/1     Running   0          69s
consul-server-1                                1/1     Running   0          69s
consul-server-2                                1/1     Running   0          68s
consul-webhook-cert-manager-6856d5f69c-9cczk   1/1     Running   0          70s
```

## Configure Consul Access

```bash
# Retrieve the ACL bootstrap token from the respective Kubernetes secret and set it as an environment variable.
export CONSUL_HTTP_TOKEN=$(kubectl get --namespace consul secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)

# Set the Consul destination address.
 export CONSUL_HTTP_ADDR=https://$(kubectl get services/consul-ui --namespace consul -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Remove SSL verification checks to simplify communication to your Consul cluster.
 export CONSUL_HTTP_SSL_VERIFY=false
```

## Validte Consul Configuration

```bash
# List Consul Services in the current namespace.
consul catalog services

# List Consul Servers Consul is currently aware of
consul members
```

The output should look similar to the below.
```bash
➜  k8s-consul git:(main) ✗ consul catalog services
consul
➜  k8s-consul git:(main) ✗ consul members
Node             Address         Status  Type    Build   Protocol  DC   Partition  Segment
consul-server-0  10.52.0.9:8301  alive   server  1.14.0  2         dc1  default    <all>
consul-server-1  10.52.1.4:8301  alive   server  1.14.0  2         dc1  default    <all>
consul-server-2  10.52.2.9:8301  alive   server  1.14.0  2         dc1  default    <all>
➜  k8s-consul git:(main) ✗ 
```

