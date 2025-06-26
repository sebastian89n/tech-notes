>**CoreDNS** is the default DNS server for Kubernetes, responsible for resolving service names to IP addresses within the cluster. It is highly modular and configurable, allowing for DNS-based service discovery and support for custom DNS plugins.
>It is a Go(lang) based DNS server that stores DNS records and answers domain name queries

>**kube-dns** is the name of the service that serves on 53 for TCP/UDP(Standard DNS), this points at the codedns pods. Historically, kube-dns was a DNS server in Kubernetes <= v1.11. The service name of kube-dns was by design, kept for compatibility purposes.

Let's restart our curl pod, we will still have our aliases from earlier -

`$ kubectl run curl --image=curlimages/curl --restart=Never -- sh -c "sleep `infinity"

Let's recreate our friendly shell function for curl as kcurl -

`$ kcurl() { kubectl exec -it curl -- sh -c "curl $1"; }`

And let's check if we can curl via our pod to nginx.default.svc.cluster.local -

`$ kcurl nginx.default.svc.cluster.local`

Let's re-apply our coredns yaml file -

`$ kubectl apply -f /etc/kubernetes/resources/coredns.yaml`

And our kube-dns yaml file -

`$ kubectl apply -f /etc/kubernetes/resources/kube-dns.yaml`

Confirm they are visible from kubernetes -

`$ kubectl get all -A`

And again, attempt to query dns via our pod -

`$ kcurl nginx.default.svc.cluster.local`

Success! if you wish, you can now clean up the running curl pod -

`$ kubectl delete pod/curl`

![[Pasted image 20250625083921.png]]
![[Pasted image 20250625084356.png]]