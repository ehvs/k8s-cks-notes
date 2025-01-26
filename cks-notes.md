# CKS 
## CIS (Center for Internet Security) Benchmark
Kube-Bench tool (from Aqua)
 ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml


## TLS
- What are TLS certificates?
- How does K8S works w certificates?

Certs enable encryption in the information user <-> server.
Asymmetric Encryption, pair of keys (Private and Public).

* Certificate Authority (CA) a.k.a ROOT certificates [Symantec, GlobalSign, Digicert]
* Server Certs
* Client Certs

### K8S + TLS

```
======== Server Certs
kube-api-server
etcd 
======== Server certs for Servers
kube-api-server
kubelet server
======== Client Certs for Clients
the user
kube-scheduler
kube-controller-manager
kube-proxy
========
```

## KubeConfig
Path: `~/.kube/config`
Contains: Clusters / Contexts USER@CLUSTER / Users

## KubeAPI + proxy


# COMMANDS
base64 -w 0
k config use-context $user@$cluster
k config --kubeconfig=/root/my-kube-config use-context research
openssl x509 -text -noout -in file.crt -dates
k auth can-i list pods --as=system:serviceaccount:default:my-service-account

APISERVER
curl --cacert ca.crt -H "Authorization: Bearer $TOKEN" "$APISERVER/api/v1/namespaces/default/pods"

JQ
- Check what items(keys) exists:
curl -sk https://localhost:10250/pods | jq '.items[].metadata  |keys'


# ABAC
Attribute Based - 
1. Setup in the kube-apiserver, KUBEAPI config file (/etc/kubernetes/abac/) must have `--authorization-policy-file` and `authorization-mode=ABAC`
2. Mount the volumePath and volumes.

.jsonl 
```json
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user":"scheduler", "namespace": "*",              "resource": "bindings"                                     }}
```

# Kubelet Security
- Authentication => Certificates x509 OR API Bearer Tokens
- Object `KubeletConfiguration`. 
- Check the service logs from `kubelet.service` to check the configuration paths.

# Accessing KubeAPI using kubectl
- Use `&` to run in the background.

kubectl uses the kubeconfig that authenticates to Kube APIServer
or curl kubeapi-ip:6443 -k --ca files.

kubectl proxy => localhost:8001 [Do not use certificate files]
eg. 
```bash
curl http://localhost:8001/api/v1/namespaces/default/services/nginx/proxy
```

kubectl port-forward service/<name> localport:svcport
```bash
curl http://localhost:<localport>
```


# Upgrading - per node/inside the host

- Verify the repository from the package manager, edit the file accordingly and fetch the new available version.
```
vim /etc/apt/sources.list.d/kubernetes.list
apt update
apt-cache madison kubeadm
```
- Running pre-flight checks: `kubeadm upgrade plan v.1.YY.Z`

## Control plane
1. Upgrade the `kubeadm` package
1. Upgrade "cluster components": `kubeadm upgrade apply v.1.YY.Z`
1. Upgrade the `kubelet` package
1. Restart the service `systemctl restart kubelet`

## Workers
- Before starting, **drain the node**.
1. Upgrade the `kubeadm` package
1. Upgrade the `kubelet` package
1. Upgrade "worker components": `kubeadm upgrade  node`
1. Restart the service `systemctl restart kubelet`
- Once finished, **uncordon**.

# Networking - Ingress/Egress and Network Policies

It is all about POV *Point of View*.  

Given a certain pod A, the traffic that ARRIVES to the pod A is the INGRESS. The traffic that leaves pod A is the EGRESS.

## Network Policies

- In the NetPol manifest, both `policyTypes` (Ingress and Egress) must be set to ensure isolation.

### Context

Pod "db" and "api" are both hosted in namespace "myapp"

#### Scenarios:
a. Isolating pod "db" to **only** receive ingress traffic from pod "api" that is hosted in namespace: "myapp"

**NOTE** the identation, where the `-` indicates that it is all part of one single rule.
```yaml
policyTypes:
  - Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api
    namespaceSelector:
      matchLabels:
        name: myapp
```
b. Isolating pod "db" to only receive ingress from pod "api" &&  from any pods in namespace "test-app"

*This means that the "db" pod will receive traffic from the "api" pod, that lives in the same namespace as "db", and also receive ingress traffic from any pod in ns/"test-app"*

**NOTE** the identation, where the `-` indicates that there are **two** rules.

```yaml
policyTypes:
- Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api
  - namespaceSelector:
      matchLabels:
        name: test-app
```

c. Isolating pod "db" to only receive ingress from pod "api" &&  from any pods in namespace "test-app" && from an external server IP.

*This means that the "db" pod will receive traffic from the "api" pod, that lives in the same namespace as "db", and also receive ingress traffic from any pod in ns/"test-app" AND also receives ingress trafic from an external server*

**NOTE** the identation, where the `-` indicates that there are **three** rules.

```yaml
policyTypes:
- Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api
  - namespaceSelector:
      matchLabels:
        name: test-app
  - ipBlock:
      cidr: 192.168.5.10/32
```

d. Sending traffic from "DB" pod to a backup server 
```yaml
policyTypes:
- Ingress
- Egress
egress:
- to:
  - ipBlock:
      cidr: 192.168.5.10/32
```

## Ingress

Ingress Controller => Nginx // Countour // HAProxy 
Deploying nginx-controller => deployment && configmap for configuration && service && auth: serviceaccount (roles,clusteroles,rolebindings)
Ingress Resource => CR of kind: Ingress (backend.serviceName/servicePort)