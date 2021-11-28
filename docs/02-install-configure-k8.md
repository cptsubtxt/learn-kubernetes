# Install K8s

In this section we are going to install the software needed for a K8s Cluster.

The following steps have to be made on all machines that where created in the previous chapter.

Afterwards there are steps to fullfill on controle plane (master node), or worker nodes.

## Install iptables legacy

In the first step we have to downgrade iptables and afterwards install docker, kubectl and kubeadm.

We gonna wanna use Legacy packages of iptable, ip6tables, arptables and etables. If you are curious why check out [Why use package iptables-legacy], otherwise just issue the commands:


```console
apt-get install -y iptables arptables ebtables

update-alternatives --set iptables /usr/sbin/iptables-legacy
update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
update-alternatives --set arptables /usr/sbin/arptables-legacy
update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

## Install Docker

Now let's install Docker.


```console
apt-get install -y apt-transport-https ca-certificates curl software-properties-common nfs-common
curl -s https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get install -y docker-ce docker-ce-cli
```

We gonna wanna check if docker is installed the right way with this command:

`docker info`

We might get an error that we can fix with adding this to daemon.json config of docker.

```console
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "insecure-registries" : ["10.96.0.0/12", "127.0.0.0/8", "23.88.110.237/32"],
  "registry-mirrors": ["http://23.88.110.237:5000"]
}
EOF
```

We restart Docker Daemon after all operations are done.

`systemctl restart docker`


## Installing Kubernetes Packages

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

With apt-mark we gonna wanna make sure that apt-get update does not install a newer version if available. For update process we want to make sure that we controll it.

## Pinning versions

If we want to pin versions e.g. we want to make sure a specific version of docker, kubelet, kubeadm or kubectl is installed, we can pin versions.

For example if we want to pin kubelet version to 1.22.1, this is achieved like this:

```console
echo "
Package: kubelet
Pin: version 1.22.1
Pin-Priority: 1000
" > /etc/apt/preferences.d/kubelet
```

For the other packages this can be done accordingly.


# Configure K8s

After installing all required software packages we are going to configre K8s Cluster and Join worker nodes.

## Configure iptables

We want ipfilter to also see bridged traffic, so we have to set net.bridge.bridge-nf-call for IPv4 and IPv6 to 1. 

See [Letting iptables see bridged traffic] for more details

```console
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p
```

## Install Kubernetes Master

Log in to Master Node with SSH. With kubeadm we will install our control plane on master machine that we created in the previous chapter.

```console
master$ kubeadm init \
  --pod-network-cidr=10.0.0.0/16 \
  --kubernetes-version=1.22.1 \
  --ignore-preflight-errors=NumCPU \
  --upload-certs \
  --apiserver-cert-extra-sans 10.0.0.1
```

We use the command with following options: 

**pod-network-cidr** - we want to use the network that all provisioned hosts belong to
**kubernetes-version** - our version we intend to install
**ignore-preflight-errors** - We don't want that installation stops, if Number of CPU is not enough.
**upload-certs** - Upload generated certs to kubernetes secret store
**apiserver-cert-extra-sans** - We are specifying the Subject Alternative Names (SANs) for our certs that will be generated.

After issuing the command you should get a join command that you can use on node machines to join this cluster afterwards. Alternatively, if something goes wrong or it takes you longer to set up nodes you can create a new join token

`kubeadm token create --print-join-command`


## Install Kubernetes Node(s)

On our Nodes all packages should be good to go, so we only have to issue kubeadm join command that we copied from above master installation.

node-x$ kubeadm join <.....>

# Test your installation

When we return to master machines we can do some tests if everything went fine.

e.g.

```console
kubectl get nodes
kubectl cluster-info
```

[Why use package iptables-legacy]: https://github.com/kubernetes/kubernetes/issues/71305#issuecomment-448351413
[Letting iptables see bridged traffic]: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#letting-iptables-see-bridged-traffic