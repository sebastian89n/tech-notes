>**kubeadm** is a tool provided by Kubernetes to simplify the process of setting up a Kubernetes cluster. It handles the installation and configuration of core components like the API server, scheduler, controller manager, and kubelet. kubeadm is commonly used to bootstrap production-grade clusters quickly and reliably.

Official kubernetes documentations at: https://kubernetes.io 

Install Kubernetes with kubeadm, we're going to specify the pod-network-cidr to use the subnet range of 10.10.0.1/16 and we're going to be implicit about the kubernetes version installed -

`$ kubeadm init --pod-network-cidr=10.10.0.0/16 --kubernetes-version=1.31.0`

As per the output, we will configure our .kube/config file for access to the Kubernetes cluster -

`$ mkdir -p $HOME/.kube; sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config; sudo chown $(id -u):$(id -g) $HOME/.kube/config`

We can now query our Kubernetes nodes as follows -

`$ kubectl get nodes`

At the moment, our node is showing as Not-Ready as there is no CNI configured. We have a simple CNI configuration file example that will use the bridge plugin to serve pods on the subnet range 10.10.0.0/16. We can view this file here -

`$ cat /resources/cni/10-bridge.conf`

Copy this configuration file to /etc/cni/net.d, the standard location expected by Kubernetes. After this we'll run a watch command to see any changes to the node status -

`$ sleep 3 && cp /resources/cni/10-bridge.conf /etc/cni/net.d & watch kubectl get nodes -o wide`

This will also have created the mynet0 bridge that we saw in the configuration file, verify this in the ip addr output -

`$ ip addr`

By default, a cluster installed with kubeadm may have taints, using kubectl, describe the node and check the taints section -

`$ kubectl describe node control-plane | more`

We can remove the taint by issuing a taint command, with the taint and a minus symbol appended -

`$ kubectl taint node control-plane node-role.kubernetes.io/control-plane:NoSchedule-`

If we check again, the taint will be removed -

`$ kubectl describe node control-plane | more`

Our single kubeadm based node is now ready for workloads, in the next video we'll take a look at Kubernetes Pods

![[Pasted image 20250615180343.png]]