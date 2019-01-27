# Rasp Cluster
Info and files for setting on raspberry pi cluster with kubernetes

# Infrastructure

- 6 Raspberry Pi 3B+
- 6 MicroSDHC cards
- 1 Anker PowerPort 6-port 60W
- 1 Ubiquiti UniFi Switch 8
- 6 USB cables
- 6 Ethernet cables
- 1 GeauxRobot Raspberry Pi 3 Model B 7-layer

# Software

- Raspbian Stretch Lite

# Steps

## Flash

First step is finding your favorite flashing tool for flashing the sd cards,
this guide uses Raspbian Lite as the OS for the Raspberry Pi's and Etcher is
used for flashing the sd cards.

When the flashing is complete for unmount and reinsert the card

## SSH

For making it possible to SSH into a Raspberry Pi, create an empty file on the
sd card called ssh, on a mac below command can be used

```bash
touch /Volumes/boot/ssh
```

The sd card can now be unmounted and inserted into the Raspberry Pi. After the
Raspberry Pi is booted, ssh should be enabled

```bash
ssh pi@ip-for-raspberry
```

Where the default password id _raspberry_, remember to change the default
password by running

```bash
passwd
```

## Device setup

Below steps should be done for all the nodes in the cluster

### Update

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

Reboot device after update

### Hostname

Update hostname by using the *raspi-config*

```bash
sudo raspi-config
```

### Setup files

Copy the folder setup to the Raspberry Pi by running

```bash
scp -r setup pi@ip-for-raspberry:/home/pi/
```

### Locale setup

Then setup correct locale by editing /etc/locale.gen and uncomment
*en_US.UTF-8*. Update */etc/default/locale* with

```bash
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
LANGUAGE=en_US.UTF-8
```

And then run

```bash
sudo locale-gen en_US.UTF-8
```

```bash
sudo update-locale en_US.UTF-8
```

### Network setup

Use *raspi-config* to set the hostname and reboot, for setting the ip, use
either dhcpcd or set a fixed ip through your network.

It should now be possible to ssh by doing

```bash
ssh pi@pi-hostname.local
```

Which verifies that hostname and static ip has been set

### Installs

In *setup/scripts* is the file *Installs.sh* which installs docker, disables
swap and installs kubeadm, so run the file

```bash
./setup/scripts/installs.sh
```

## Kubernetes setup

*kubeadm* should be used to setup the kubernetes master and worker nodes, in the
*/setup/kubernetes_config/* folder is the python file *init.py*, which wraps the
*kubeadm init* command, with tracking of the manifest file, so that the init
will not end up timing out and not enable the cluster.

### Master node

On the master node run

```bash
python3 init.py
```
It will take a couple of minutes to let kubeadm spin up the master node, when
finished kubeadm asks for some things to be executed

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

And also prints the line which should be uses for the workers to join the
network

```bash
sudo kubeadm join --token TOKEN 192.168.1.100:6443 --discovery-token-ca-cert-hash HASH
```

Check whether the node is up by running

```bash
kubectl get nodes
```

### Worker node

After the full setup run

```bash
sudo kubeadm join --token TOKEN 192.168.1.100:6443 --discovery-token-ca-cert-hash HASH
```

Then check that the node has joined the network

```bash
kubectl get nodes
```
