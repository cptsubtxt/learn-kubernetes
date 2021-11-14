# Learn Kubernetes

This repo contains all ressource for learning kubernetes. The idea is to set up a cluster in an automated fashion, explore the components of kubernetes with some examples. A special sections deals with certification as cka.

At the end a kubernetes cluster should be running compliant with the cka requirements.

The cluster will be run on hetzner cloud, but ressources can be altered to run in any other cloud environment such as digital ocean, google cloud engine just to name some of them.

## Required Tools

A List of all required tools

* [git](tools/git.md)
* [docker](tools/docker.md)
* [kubectl](tools/kubectl.md)
* [kubeadm](tools/kubeadm.md)
* [helm](tools/helm.md)

## Cluster Details

We will set up a cluster compliant with CKA Requirements as outlined in [CKA & CKAD Environment]

* [kubernetes](https://github.com/kubernetes/kubernetes) v1.22
* [containerd](https://github.com/containerd/containerd) v1.4.4
* [coredns](https://github.com/coredns/coredns) v1.8.4
* [cni](https://github.com/containernetworking/cni) v0.9.1
* [etcd](https://github.com/etcd-io/etcd) v3.5.0


## Labs

* [01 | Provision the hosts](docs/01-provision.md)

## Further Reading


[CKA & CKAD Environment]: https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad#cka-and-ckad-environment