---
layout: docs
title: "Join Windows node"
permalink: /docs/join-windows-node
---

# Joining a Windows node to MicroK8s

MicroK8s supports running Windows workloads.  This guide will go through the process of joining a Windows Server 2019 node to an existing MicroK8s cluster running Calico.

## Prerequisites

* MicroK8s cluster with Calico CNI (default in 1.19 and onwards).
* `calicoctl` installed on the same host as MicroK8s (see [official documentation](https://docs.projectcalico.org/getting-started/clis/calicoctl/install#installing-calicoctl-as-a-container-on-a-single-host)).
* Windows Server 2019 with [Docker running on the system](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/deploy-containers-on-server).

## Prepare MicroK8s

In order for Windows pods to schedule, strict affinity must be set to `true`. This is required to prevent Linux nodes from borrowing IP addresses from Windows nodes.  To set this, we will use the `calicoctl` binary.

### Export kubeconfig

So that `calicoctl` can access the cluster, we need to export the kubeconfig from MicroK8s.  This can be done to any location, but we'll use the default path for this example.

```bash
mkdir -p ~/.kube
sudo microk8s config > ~/.kube/config
```

### Set affinity

Now we can set affinity to true.  We must be sure to point `calicoctl` to the Kubernetes API, rather than directly to Etcd.

```bash
DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl ipam configure --strictaffinity=true
```

### Gather Kubernetes version

At a later stage, we'll need to know the exact version of Kubernetes installed.  This can be grabbed from MicroK8s.

```bash
sudo microk8s kubectl get node $(hostname)
```

Note the output in the format `X.X.X`, e.g. `1.19.3`.

## Install components on the Windows node

We're now ready to install Calico onto the Windows node.  This will also install Kubernetes components required for a working node.

All code snippets here should be run in PowerShell running as Administrator.

### Create directory for Kubernetes

All Kubernetes components will be installed into this directory.

```powershell
mkdir C:\k\
```

In here, we place the kubeconfig file that we already exported from MicroK8s.  Be careful as some editors may try to append file extensions.  You can test that it's named correctly by trying to print the contents, after you've saved it.

```powershell
cat C:\k\config
```

### Install

The Calico Project provides a helper script to fetch all the required binaries and services.  [Official documentation](https://docs.projectcalico.org/getting-started/windows-calico/standard) is available for this script, but we'll cover the required steps.

First, download the script.

```powershell
Invoke-WebRequest https://docs.projectcalico.org/scripts/install-calico-windows.ps1 -OutFile C:\install-calico-windows.ps1
```

We can then run the script with the required parameters.  Change the `-KubeVersion` argument to the version noted earlier.

```powershell
C:\install-calico-windows.ps1 -DownloadOnly yes -KubeVersion 1.19.3
```

Register the Calico services.

```powershell
C:\CalicoWindows\install-calico.ps1
```

Then register the Kubernetes services so they come up with the node.

```powershell
C:\CalicoWindows\kubernetes\install-kube-services.ps1
```

This script won't start the Kubernetes services.  Let's do that now.

```powershell
Start-Service kubelet
Start-Service kube-proxy
```

## Cleanup

Let's remove the temporary files created.

```powershell
rm C:\install-calico-windows.ps1
rm C:\calico-windows.zip
```

## Finish

We can see the node come up on the cluster by switching back to the MicroK8s node.

```bash
sudo microk8s kubectl get no -o wide
```
