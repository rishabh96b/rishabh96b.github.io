---
layout: single
title:  ""
date:   2021-06-27 22:52:41 +0530
tags: kubernetes kind gcp container
---

# Running KIND cluster in a container optimised OS on GCP

In this blog, we will be creating a K8s cluster using `KIND` in a container optimised OS on GCP and then accessing this cluster locally. But first things first, the question arises here is

### Why prefer KIND cluster on a GCP VM instance vs runnning it locally?

- Not everyone have machines powerful enough to run KIND locally along with the other applications feeding heavily on the system resources.
- This kind of environment is well suited for running tests for your applications especially resource intensive ones which might trouble your development system.
- With Google Cloud's [free trail scheme](https://cloud.google.com/free/docs/gcp-free-tier/#free-trial), we can leverage GCP's special images optimised for running containers which is ideal for running containers and Kubernetes.

### Part 1: Creating a VM instance with container optimised OS image

Create a VPC with custom subnet

```bash
gcloud compute networks create kind-vpc --project=ancient-house-315814 \
--description=VPC\ for\ the\ instance\ hosting\ the\ kind\ cluster. \
--subnet-mode=custom \
--mtu=1500 \
--bgp-routing-mode=regional

gcloud compute networks subnets create kind-subnet --project=ancient-house-315814 \
--description=Subnet\ for\ VPC\ dedicated\ to\ the\ instance\ for\ hosting\ the\ kind\ cluster. \
--range=10.2.0.0/16 \
--network=kind-vpc \
--region=asia-south1
```

Next, we need to create the firewall rules for the `kind-vpc` network we just created

```bash
gcloud compute --project=$PROJECT firewall-rules create kind-firewall-rules \
--description="Firewall rules for kind-vpc" \
--direction=INGRESS \
--priority=1000 \
--network=kind-vpc \
--action=ALLOW \
--rules=all \
--source-ranges=0.0.0.0/0
```

> Note that we are allowing all IPs to be able to access this VPC by specifying that in firewall rules. This is only suitable for individual testing and is not recommended for use restricted environments eg. running tests for private code of your company.

Let's now create a new VM instance with the above specified VPC and firewall rules. We'll also allow `http` and `https` ingress for this instance.

```bash
gcloud beta compute --project=$PROJECT instances create kind-instance \
--zone=asia-south1-c \
--machine-type=e2-standard-2 \
--subnet=kind-subnet \
--network-tier=PREMIUM \
--maintenance-policy=MIGRATE \
--service-account=$SVC_ACCOUNT_ID-compute@developer.gserviceaccount.com \
--scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
--tags=http-server,https-server \
--image=cos-89-16108-470-1 \
--image-project=cos-cloud \
--boot-disk-size=20GB \
--boot-disk-type=pd-balanced \
--boot-disk-device-name=kind-instance \
--no-shielded-secure-boot \
--shielded-vtpm \
--shielded-integrity-monitoring \
--reservation-affinity=any

gcloud compute --project=$PROJECT firewall-rules create kind-vpc-allow-http \
--direction=INGRESS \
--priority=1000 \
--network=kind-vpc \
--action=ALLOW \
--rules=tcp:80 \
--source-ranges=0.0.0.0/0 \
--target-tags=http-server

gcloud compute --project=$PROJECT firewall-rules create kind-vpc-allow-https \
--direction=INGRESS \
--priority=1000 \
--network=kind-vpc \
--action=ALLOW \
--rules=tcp:443 \
--source-ranges=0.0.0.0/0 \
--target-tags=https-server
```

Now, we'll need to ssh into the freshly created vm instance and install `kind`. If you have not generated the ssh keys and attached to the project, generate the keys using `ssh-keygen` and add it to the project's metadata which can be located at [https://console.cloud.google.com/compute/metadata/sshKeys?project=](https://console.cloud.google.com/compute/metadata/sshKeys?project=ancient-house-315814)<your-project-name>

We can now ssh into the instance by running

```bash
ssh -i path/to/private-key $(whoami)@<external-ip-of-instance>
```

As per the container optimised OS [docs](https://cloud.google.com/container-optimized-os/docs/concepts/disks-and-filesystem), we need to download the kind binary and keep it in stateful partitions as it is **persistent** and **writable**. The root partition `/` is read only.

Let's download and keep the kind binary at `/var/lib/google` directory as it is both stateful and executable.

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
chmod +x ./kind
sudo mv kind /var/lib/google
```

Also add `/var/lib/google` to the $PATH so that `kind` can be run from any directory.

```bash
echo "export PATH=\$PATH:/var/lib/google" >> $HOME/.bashrc
source $HOME/.bashrc
```

Create a kind cluster by running

```bash
kind create cluster --name kind --wait 120s
```

And we'll have a fully functional kubernetes cluster in seconds.

### Part 2: Accessing the cluster locally

Once the cluster is created, we will need to start the proxy to access the cluster at instance's `localhost:port`. We can do this by running

```bash
kubectl proxy --port 8080 &
```
>**Tip**: We can run the above command locally by
```bash
ssh -i /path/to/private-key -o StrictHostKeyChecking=accept-new -A $(whoami)@<external-ip-of-instance> "kubectl proxy --port 8080"
```

Verify if the cluster is accessible at `127.0.0.1:8080` by making an API call to the kube-apiserver

```bash
curl -XGET http://127.0.0.1:8080/api/v1/namespaces/kube-system/pods
```

This will return all the pods running in the `kube-system` namespace. 

The next challenge is to access this cluster from local machine. This can be achieved by `Local port forwarding`. We need to map the local port(here we will be using 8080) to the api-server which is serving on `127.0.0.1:8080` in our VM instance running on GCP. We'll achieve this by running

```bash
ssh -L 8080:127.0.0.1:8080 $(whoami)@<external-ip-of-instance>
```

>**Note**: If you see an error like `bind: Cannot assign requested address`, it is most probably because it is trying to bind on an IPv6 address, not IPv4. In that case, run the command with `-4` flag
```bash
ssh -4 -L 8080:127.0.0.1:8080 $(whoami)@<external-ip-of-instance>
```

Next, we need to copy the `kubeconfig` file present at `$HOME/.kube/config` in the local system. This could be done either by 
- Running `cat $HOME/.kube/config` and then copy-pasting it localy.
- Using `scp` to transfer the file.

Once you have the kubeconfig locally, open the file and change the `server` key from ~~`server: https://127.0.0.1:40261`~~ to `server: http://127.0.0.1:8080`

> Don'e forget to make https -> http in the local kubeconfig file. ;)

We are finally ready to access our kind kubernetes cluster running inside a container optimised os on GCP locally. I have copied my custom kubeconfig file named `config` at $HOME

```bash
kubectl --kubeconfig $HOME/config get po -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-558bd4d5db-7p59g                     1/1     Running   0          24h
kube-system          coredns-558bd4d5db-fwg5j                     1/1     Running   0          24h
kube-system          etcd-kind-control-plane                      1/1     Running   0          24h
kube-system          kindnet-hf284                                1/1     Running   0          24h
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          24h
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          24h
kube-system          kube-proxy-wzdh9                             1/1     Running   0          24h
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          24h
local-path-storage   local-path-provisioner-547f784dff-ftqq7      1/1     Running   0          24h
```

If everything remains okay, we'll be able to get the list of all the pods running in our cluster.

