# Enable Metrics Server on Docker Desktop


The steps below in this blog will help you setup [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) on Docker Desktop which provides a standalone instance of Kubernetes running as a Docker container.

[Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server) is a scalable, efficient source of container resource metrics for Kubernetes built-in autoscaling pipelines. Metrics Server collects resource metrics from Kubelets and exposes them in Kubernetes apiserver through Metrics API for use by [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler). 

Metrics Server offers:

* A single deployment that works on most clusters
* Scalable support up to 5,000 node clusters
* Resource efficiency: Metrics Server uses 1m core of CPU and 3 MB of memory per node


You can use Metrics Server for:

* CPU/Memory based horizontal autoscaling
* Automatically adjusting/suggesting resources needed by containers

### Prerequisites
* [Install Docker Desktop](https://docs.docker.com/docker-for-mac/install/)
* Enable Kubernetes on Docker Desktop

Once you have enabled the Kubernetes on Docker Desktop, and if you run the below commands you should see messages like:

{{< highlight console >}}
kubectl top node 
error: Metrics API not available
{{< /highlight >}}


{{< highlight console >}}
kubectl top pod -A
error: Metrics API not available
{{< /highlight >}}


Metrics server isn't included with Docker Desktop's installation of Kubernetes and to install it we will have to download the latest `components.yaml` file from [Metrics-Server](https://github.com/kubernetes-sigs/metrics-server/releases)releases page and open it in your text editor.

If you try to execute the command `kubectl apply -f components.yaml` you will see the pods get created but with some errors as highlighted below:

![MetricsServer_Setup](/images/ms_01.png)

Add the line --kubelet-insecure-tls under the args section as shown below :

![MetricsServer_Setup](/images/ms_02.png)

Execute the command `kubectl apply -f components.yaml` to apply the changes:
![MetricsServer_Setup](/images/ms_03.png)

Now if you execute the `kubectl top node` & `kubectl top pod -A` commands you should see the output:
{{< highlight console >}}
$ kubectl top node 
NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
docker-desktop   1310m        32%    1351Mi          71%       
{{< /highlight >}}


{{< highlight console >}}
$ kubectl top pod -A
NAMESPACE     NAME                                     CPU(cores)   MEMORY(bytes)   
cpu-example   cpu-demo                                 1003m        1Mi             
kube-system   coredns-f9fd979d6-g2rfx                  9m           9Mi             
kube-system   coredns-f9fd979d6-wndgm                  6m           9Mi             
kube-system   etcd-docker-desktop                      35m          36Mi            
kube-system   kube-apiserver-docker-desktop            55m          325Mi           
kube-system   kube-controller-manager-docker-desktop   41m          47Mi            
kube-system   kube-proxy-s72fj                         1m           25Mi            
kube-system   kube-scheduler-docker-desktop            9m           17Mi            
kube-system   metrics-server-56c59cf9ff-jndxd          10m          14Mi            
kube-system   storage-provisioner                      4m           5Mi             
kube-system   vpnkit-controller                        1m           15Mi 
{{< /highlight >}}

You can also use [Kubernetes Dashboard](https://github.com/kubernetes/dashboard) to view the above data (and more information) in a web UI. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.

To deploy Dashboard, execute following command:
{{< highlight console >}}
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml
{{< /highlight >}}

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:
{{< highlight console >}}
$ kubectl proxy
{{< /highlight >}}

Get the token for login to dashboard using the below command:
{{< highlight console >}}
$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret |grep default-token | awk '{print $1}')
{{< /highlight >}}

To access the HTTPS endpoint of dashboard go to:
{{< highlight console >}}
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
{{< /highlight >}}

Login to the dashboard using the token from above step and you should see a dashboard as below:
![MetricsServer_Setup](/images/ms_04.png)

{{% admonition info %}}
***This setting should only be used for the local Docker Desktop Kubernetes cluster, and not recommended for any hosted or production clusters.***
{{% /admonition %}}