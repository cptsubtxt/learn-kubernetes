# Provision the nodes

For this tutorial we are using [Hetzner Cloud], you can [Sign up] here.

As we use three nodes, cx11 for master and two worker nodes cx21, the expected cost per hour is 0,027 € and 15,81 € per month.

## Hetzner Cloud

### Get the tools for Hetzner Cloud

Download [hcloud] Cli that works for your OS.

### Hetzner Cloud Console | Project and API Token

After [Sign up] go to [Hetzner Cloud Console] and create a project with "+ new project"

Generate an API Token with clicking on the project just created and choose security and API-Tokens.

The above mentioned tasks also work with hcloud Commandline tool. You have to confirm with your API Token just created.

`hcloud create context k8-learn`

### Create SSH Key

Next we have to create a SSH Key for adding to our Server that we will provision in the next place.

For Windows we follow the infos presented under [Generate SSH Keys in Windows]

For *ix Systems we make use of the steps presented under [Generate SSH Keys with ssh-gen] with a Name for example an Emailaddress or a Label for later steps

e.g.

`ssh-keygen -f ~/ssh-key-rsa-learnk8 -t rsa -b 4096 -C "nomail@somedomain.com"`

After you have created SSH Keys go back to [Hetzner Cloud Console] and paste content of your ssh-key-rsa-learnk8.pub file under Project k8-learn, Security, SSH Keys.

## Let's create resources

Now we can create a subnet that will be used afterwards from the Master and Nodes

### Create VPC netowrk and Subnet

```console
hcloud network create --name learn-k8 --ip-range 10.0.0.0/16
hcloud network add-subnet --ip-range 10.0.0.0/24 --network-zone eu-central --type cloud learn-k8
```
### Remember network_id

We should remember the network id as we will use it in later steps.

`network_id=$(hcloud network list -o noheader -o columns=id)`

### Create Master and Worker Nodes

The ssh-key entry will be the entry former created in Step [Create SSH Key]

```console
# Master
for i in 0; 
do
  ${HCLOUD} server create \
    --datacenter nbg1-dc3 \
    --name master-${i} \
    --image  ubuntu-20.04 \
    --type cx21 \
    --ssh-key nomail@somedomain.com \
    --network ${network_id}
done

# Nodes
for i in 0 1; do
  hcloud server create  \
    --name node-${i} \
    --datacenter nbg1-dc3 \
    --image  ubuntu-20.04 \
    --type cx21 \
    --ssh-key nomail@somedomain.com \
    --network ${network_id}
done
```

Now we can follow the steps presented in LFS 258 Chapter 3 Installation and Configuration.

See [Install K8 on Commandline]

## Let's use the process and provision with terraform

As you could see it might be cumbersome to install all the stuff on the machines. We will use the steps presented here and will automate the stuff with [Terraform].



[Hetzner Cloud Console]: https://console.hetzner.cloud/
[hcloud]: https://github.com/hetznercloud/cli
[Kubernetes Tutorial]: https://community.hetzner.com/tutorials/install-kubernetes-cluster
[Terraform to provision hosts]: https://community.hetzner.com/tutorials/howto-hcloud-terraform
[Sign up]: https://www.hetzner.com/cloud?country=de
[Generate SSH Keys in Windows]: https://www.ssh.com/academy/ssh/putty/windows/puttygen
[Generate SSH Keys with ssh-gen]: https://www.ssh.com/academy/ssh/keygen
[Terraform]: https://www.terraform.io
[Install K8 on Commandline]: 02-install-configure-k8.md