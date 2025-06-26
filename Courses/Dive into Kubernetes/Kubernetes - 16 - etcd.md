>**etcd** is a distributed key-value store used by Kubernetes to store all cluster data, including the configuration, state, and metadata. It acts as the **source of truth** for the cluster, ensuring consistency and coordination across components. Because of its critical role, etcd must be highly available and backed up regularly.

![[Pasted image 20250624082737.png]]

Using our static pods manifests directory, let's move the etcd yaml file back to /etc/kubernetes/manifests, as we do let's watch this start using nerdctl, press CTRL-C to exit -

`$ mv /etc/kubernetes/resources/etcd.yaml /etc/kubernetes/manifests; watch nerdctl -n k8s.io ps -a`

Let's explore etcd, firstly we'll install a convenient tool for interacting with etcd -

`$ apt update && apt install -y etcd-client`

If we check our processes, etcd is running and there is useful information in the command line output such as the endpoint and certificate locations -

`$ ps -ef | grep etcd`

Capture the ENDPOINT as a variable -

`$ ENDPOINT=$(ps -ef | grep etcd | awk -F ' ' '{for(i=1;i<=NF;i++){if ($i ~ /--advertise-client-urls=/) {print $i}}}' | awk -F= '{print $2}');echo $ENDPOINT`

Using the ENDPOINT variable, query etcd using the etcd-client cli, prefix the command with ETCDCTL_API=3 to use v3 of the API -

`$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --write-out=table endpoint status`

Let's use etcd to query the available keys that are being used by Kubernetes -

`$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only | grep -v ^$ | head -10`

ETCD is a distributed key/value store, let's set a key of foo -

`$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key put foo bar`

And let's query that value, it should be bar -

`$ ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key get foo`