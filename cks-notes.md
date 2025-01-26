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
```{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user":"scheduler", "namespace": "*",              "resource": "bindings"                                     }}```

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


