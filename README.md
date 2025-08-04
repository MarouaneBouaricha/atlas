# atlas
GitOps configuration with Cluster-API to provision k8s clusters on Proxmox

## Requirements
- KinD as management cluster
- Proxmox Server 
- Cluster-API
- ArgoCD for GitOps workflow

## Roadmap
- Create a management cluster for Cluster-API
- Create a Kubernetes VM Template
- Initialize Cluster-API and configure GitOps like workflow
- Create first workload cluster

### Create Management cluster
```shell
cd kind
kind create cluster --name management-cluster --config kind-config.yaml
```

### Create kubernetes VM template
- Create API access to Proxmox, on the proxmox node run:
```shell
pveum user add caprox@pve
pveum aclmod / -user caprox@pve -role PVEAdmin
pveum user token add caprox@pve capi -privsep 0
```
```shell
root@server1:~# pveum user token add caprox@pve capi -privsep 0
┌──────────────┬──────────────────────────────────────┐
│ key          │ value                                │
╞══════════════╪══════════════════════════════════════╡
│ full-tokenid │ caprox@pve!capi                      │
├──────────────┼──────────────────────────────────────┤
│ info         │ {"privsep":"0"}                      │
├──────────────┼──────────────────────────────────────┤
│ value        │ 6e59df15-a2c9-4dc5-b293-367772950c68 │
└──────────────┴──────────────────────────────────────┘
```
- Start Build-Job for VM template
```shell
# create a namespace for the build
kubectl create namespace proxmox-build-infrastructure-system
# apply secret & job
kubectl apply -f image-builder/secret.yaml
kubectl apply -f image-builder/job.yaml
# start the build
kubectl create job build-image --from cj/proxmox-template-builder -n proxmox-build-infrastructure-system 
```
- Monitor current state of the build-pod
```shell
kubectl get pods -n proxmox-build-infrastructure-system
NAME                READY   STATUS    RESTARTS   AGE
build-image-tdv78   1/1     Running   0          97s
kubectl logs -f -n proxmox-build-infrastructure-system build-image-tdv78
```
Wait until the job is done and a VM template will show up on the Proxmox UI. Kubernetes will automatically restart the job if the build fails.
