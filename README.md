# rpi-k8s
Kubernetes installation on RPI


Building a RPi Kubernetes Cluster
A quick overview of how to build a Raspberry Pi Cluster using Linux.
Bill of materials

- [x] Raspberry Pi (Minimum x 2)
- [x] MicroSDHC Card (Minimum 8GB)
- [x] USB Charging Cable (Same amount as RPi)
- [x] Machine to creating a RPi Disk Image

## Create the Disk Image

- [x] Download a Raspbian Buster Image 
- [x] Use the raspbian debian (buster) desktop/lite version - Lite if you dont want the dashboard
- [x] Take the Memory Card and install an operating system.

For our kubernetes cluster:

1. Download either the zip file to your machine
__Note:__ the location where the file is being downloaded

2. Insert the memory card into your host machine and confirm it is recognised. 

3. Take note of how the memory card is labelled as we will use this information later 

4. Locate the downloaded file on your host.  
5. Extract the file to an image file
6. Right click on the downloaded image to show the 
7. Open with Disk Image Writer option.

8. In the image writer, select the memory card inserted to be used to write the image. Once the image has been written to the memory card there will be two volumes available:

* boot
* rootfs 

9. Select the BOOT from the file manager:

Click on the boot folder to mount the image. Then right click in the folder window and select the “open in terminal” option.

In the Terminal window for the boot folder add a SSH file to the root of the boot folder

```
touch ssh
```

This ensure the SSH service is started

Close the open terminal window

10. Select the ROOTFS from the file manager:

Click on the rootfs folder to mount the image. Then right click on the folder window and select the “open in terminal” option

In the Terminal window for the rootfs folder. Enter the config directory
```
cd etc
```

Edit the hostname file (use whatever editor you like)
```
sudo vi hostname
```
Change the hostname from the default raspberrypi

e.g. 
k8s-master-01
k8s-worker-01

Edit the hosts file (use whatever editor you like)
sudo vi hosts

Amend the host file to use the updated hostname used above e.g.

Master
| Before  |After |
|---------|------|
|raspberrypi | k8s-master-01 |
|raspberrypi | k8s-worker-01 |

You can now eject the rpi filesystem (i.e. boot & rootfs) mounts using the file manager 

## Configuring the RPi

Connect the devices via ethernet cables and power them on
They should now be accessible via DHCP - check router 

Note: Optional - reserve an IP based on the mac address of the RPi

| Name | Mac Address | Device IP |
|------|-------------|-----------|
|k8s-master-01 | xx:xx:xx:xx:xx:xx | 192.168.86.180 |
|k8s-worker-01 | xx:xx:xx:xx:xx:xx | 192.168.86:181 |

To connect to the devices, use an SSH connection from a device on the same network

```
ssh pi@[device ip]
```

__Note:__ The default password for RPI is "raspberry"

Update the device ( do this for each device)
```
sudo apt update && sudo apt upgrade
```
Edit the `/boot/cmdline.txt` and __append__ the following to the end of the line
```
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

Amend the swap setting to off
```
sudo dphys-swapfile swapoff
```
Uninstall the swapfile command
```
sudo dphys-swapfile uninstall -y && sudo apt purge -y dphys-swapfile
```

Reboot the device
```
sudo reboot
```

## Installing Docker on the RPi

1. Install Docker
```
curl -sSL get.docker.com | sh
```
2. Add the pi user to the Docker group

```
sudo usermod -aG docker pi
```

Press `ctrl-d` to disconnect from the RPi

3. Update the docker daemon - edit `/etc/docker/daemon.json`, add the following:
```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
      "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
```
4. Reboot the device
```
sudo reboot
```

## Configure Kubernetes on the RPi
1. Add the required packages
```
sudo apt update && sudo apt install -y apt-transport-https curl
```

2. Add the GPG key
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

3. Create the Kubernetes package list - /etc/apt/sources.list.d/kubernetes.list
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
4. Update the device packages
```
sudo apt update
```

5. Install the Kubernetes specific packages
```
sudo apt-get install -y kubelet kubeadm kubectl
```

6. __Optional:__ Pin the packages 
```
sudo apt-mark hold kubelet kubeadm kubectl
```

Configure a kubernetes image

7. On the master node - Pull the images 
```
kubeadm config images pull
```

The following images will be pulled to the device:

[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.18.3
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.18.3
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.18.3
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.18.3
[config/images] Pulled k8s.gcr.io/pause:3.2
[config/images] Pulled k8s.gcr.io/etcd:3.4.3-0
[config/images] Pulled k8s.gcr.io/coredns:1.6.

## Initialise Kubernetes Master

1. On the master node i.e. k8s-master-01
```
sudo kubeadm init --pod-network-cidr=10.17.0.0/16 --service-cidr=10.18.0.0/24 --service-dns-domain=mydomain.com
```

Capture the Example join command from the above command:
e.g. 
kubeadm join 192.168.86.183:6443 --token 9hj1ar.zko79d0pldz3x1x6 \
    --discovery-token-ca-cert-hash sha256:47cb858687610945e8b781eaf2daa361d464eb1f857ea9ca0cab927770f433e4

2. On the worker node use the join command shown on the master node
sudo kubeadm join 192.168.86.180:6443 --token 14cdv5.5m2npm147k0kqtuz \
    --discovery-token-ca-cert-hash sha256:c1640d15af659b9ed4d658b927f7358dd8e6e78a9bd0e3bc44c5caf95e159791


3. On the master node kubectl will not be working at this stage and return an error as we have a bit more configuration to do. 
Try running kubectl get nodes will result in the following error:

__The connection to the server localhost:8080 was refused - did you specify the right host or port?__


4. Make a directory for kubernetes configurations
```
mkdir -p $HOME/.kube
```

5. Copy across a configuration 
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

6. Amend the owner permissions for the directory
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

7. At this point the nodes will be available but in a NotReady state.  They lack a network overlay.
```
NAME            STATUS     ROLES    AGE     VERSION
k8s-master-01   NotReady   master   10m     v1.19.0
k8s-worker-01   NotReady   <none>   6m59s   v1.19.0
```

## Configure a network overlay on RPI
1. On the Master Node apply a Flannel Pod network
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

2. Confirm the Pod network is running
```
kubectl get pods --all-namespaces
```

3. Check the available nodes
```
kubectl get nodes
```

4. At this point the Cluster should be up and running as per below:
```
NAME            STATUS   ROLES    AGE   VERSION
k8s-master-01   Ready    master   15m   v1.19.0
k8s-worker-01   Ready    <none>   12m   v1.19.0
```
