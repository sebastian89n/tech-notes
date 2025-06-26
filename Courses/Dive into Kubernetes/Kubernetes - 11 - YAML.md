>In Kubernetes, YAML configuration files are used to define the desired state of resources such as Pods, Deployments, Services, and more. These files describe the resource's metadata, specifications, and behaviors in a structured, human-readable format. By applying a YAML file with tools like `kubectl apply -f`, Kubernetes creates or updates the resources to match the defined configuration.

Modify the previous kubectl nginx example so that it includes **--dry-run=client** and **-o yaml**

`--dry-run=client` previews the command without execution, -o yaml to specify a yaml output declaration -

`$ kubectl run nginx --image=nginx --dry-run=client -o yaml`

Let's create a directory where we can save this -

`$ mkdir -p /etc/kubernetes/resources`

And let's output the yaml to a file -

`$ kubectl run nginx --image=nginx --dry-run=client -o yaml > /etc/kubernetes/resources/nginx_pod.yaml`

We can apply the yaml configuration as follows -

`$ kubectl apply -f /etc/kubernetes/resources/nginx_pod.yaml`

>**kubectl create:**  validates a resource before applying
>is used to create new resources based on the contents of a YAML or JSON file. It will not check if a resource with the same name and namespace already exists in the cluster. If it does, it will return an error and the resource will not be created. This command is useful if you want to ensure that a new resource is created, even if one with the same name and namespace already exists in the cluster.

>**kubectl apply:** does not validate a resource before applying
>is used to create or update resources based on the contents of a YAML or JSON file. It will first check if a resource with the same name and namespace already exists in the cluster. If it does, it will update the existing resource with the new configuration. If it does not, it will create a new resource with the specified configuration.

And now if we check, our pod is visible -

`$ kubectl get pods`

We can also remove from kubernetes using a yaml declaration -

`$ kubectl delete --grace-period=0 -f /etc/kubernetes/resources/nginx_pod.yaml`

And now if we check again, our pod has been deleted -

`$ kubectl get pods`