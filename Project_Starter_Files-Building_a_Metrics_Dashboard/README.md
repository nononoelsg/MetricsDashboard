**Note:** For the screenshots, you can store all of your answer images in the `answer-img` directory.

## Run Kube-context from local machine
Create the file (or replace if it already exists) `~/.kube/config` and paste the content inside k3s.yaml file inside `~/.kube/config`

Afterwards, you can test that `kubectl` works by running a command like `kubectl describe services`. It should not return any errors.

Output
```bash
PS C:\Users\Noel> kubectl describe services
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Families:       <none>
IP:                10.43.0.1
IPs:               <none>
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.0.2.15:6443
Session Affinity:  None
Events:            <none>
```

## Install Helm

```bash
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard> vagrant ssh
Last login: Mon Oct 18 07:21:05 2021 from 10.0.2.2
Have a lot of fun...
Last login: Mon Oct 18 07:21:05 2021 from 10.0.2.2
Have a lot of fun...
vagrant@localhost:~> curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
vagrant@localhost:~> ls
bin  get_helm.sh
vagrant@localhost:~> chmod 700 get_helm.sh
vagrant@localhost:~> ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
vagrant@localhost:~> helm version
version.BuildInfo{Version:"v3.7.1", GitCommit:"1d11fcb5d3f3bf00dbe6fe31b8412839a96b3dc4", GitTreeState:"clean", GoVersion:"go1.16.9"}
vagrant@localhost:~>
```

## Install Grafana and Prometheus

```bash
vagrant@localhost:~> kubectl create namespace monitoring
namespace/monitoring created
vagrant@localhost:~> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories
vagrant@localhost:~> helm repo add stable https://charts.helm.sh/stable
"stable" has been added to your repositories
vagrant@localhost:~> helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
vagrant@localhost:~> helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --kubeconfig /etc/rancher/k3s/k3s.yaml
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /etc/rancher/k3s/k3s.yaml
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /etc/rancher/k3s/k3s.yaml
NAME: prometheus
LAST DEPLOYED: Mon Oct 18 07:37:18 2021
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitoring get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

##  Grafana and Prometheus Pods

```bash
vagrant@localhost:~> kubectl get pods --namespace=monitoring
NAME                                                     READY   STATUS    RESTARTS   AGE
prometheus-prometheus-node-exporter-77qjz                1/1     Running   0          5m13s
prometheus-kube-prometheus-operator-8554997fc-9k9nz      1/1     Running   0          5m12s
prometheus-kube-state-metrics-569d7854c4-x9f5b           1/1     Running   0          5m12s
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          4m29s
prometheus-grafana-7f854c9f9-s585f                       2/2     Running   0          5m12s
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0          4m25s
vagrant@localhost:~>
```

##  Install Jaeger

```bash
vagrant@localhost:~> kubectl create namespace observability
namespace/observability created
vagrant@localhost:~> kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
customresourcedefinition.apiextensions.k8s.io/jaegers.jaegertracing.io created
vagrant@localhost:~> kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
serviceaccount/jaeger-operator created
vagrant@localhost:~> kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
role.rbac.authorization.k8s.io/jaeger-operator created
vagrant@localhost:~> kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
rolebinding.rbac.authorization.k8s.io/jaeger-operator created
vagrant@localhost:~> kubectl create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
deployment.apps/jaeger-operator created
vagrant@localhost:~> kubectl get all -n observability
NAME                                   READY   STATUS              RESTARTS   AGE
pod/jaeger-operator-5977dbf59f-c2xmd   0/1     ContainerCreating   0          17s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jaeger-operator   0/1     1            0           30s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/jaeger-operator-5977dbf59f   1         1         0       30s
vagrant@localhost:~> kubectl get all -n observability
NAME                                   READY   STATUS    RESTARTS   AGE
pod/jaeger-operator-5977dbf59f-c2xmd   1/1     Running   0          88s

NAME                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/jaeger-operator-metrics   ClusterIP   10.43.197.144   <none>        8383/TCP,8686/TCP   16s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jaeger-operator   1/1     1            1           101s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/jaeger-operator-5977dbf59f   1         1         1       104s

```

##  Cluster wide Jaeger

```bash
vagrant@localhost:~> kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role.yaml
clusterrole.rbac.authorization.k8s.io/jaeger-operator created
vagrant@localhost:~> kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/cluster_role_binding.yaml
clusterrolebinding.rbac.authorization.k8s.io/jaeger-operator created
```

##  Deploying the Application

```bash
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard\manifests> kubectl apply -f app/
deployment.apps/backend-app created
service/backend-service created
deployment.apps/frontend-app created
service/frontend-service created
deployment.apps/trial-app created
service/trial-service created
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard\manifests> kubectl get all
NAME                               READY   STATUS              RESTARTS   AGE
pod/backend-app-5f749755f4-fbbkx   0/1     ContainerCreating   0          28s
pod/svclb-backend-service-qpr6x    0/1     Pending             0          27s
pod/frontend-app-75cd57cfd-lzkp4   0/1     ContainerCreating   0          27s
pod/backend-app-5f749755f4-dhxrg   0/1     ContainerCreating   0          27s
pod/backend-app-5f749755f4-l4664   0/1     ContainerCreating   0          27s
pod/frontend-app-75cd57cfd-qzgbc   0/1     ContainerCreating   0          27s
pod/svclb-trial-service-crlbn      0/1     Pending             0          19s
pod/frontend-app-75cd57cfd-cvnmv   0/1     ContainerCreating   0          27s
pod/svclb-frontend-service-6rt6l   0/1     ContainerCreating   0          25s
pod/trial-app-6cd98d67f4-496bz     0/1     ContainerCreating   0          25s
pod/trial-app-6cd98d67f4-b7rmx     0/1     ContainerCreating   0          24s
pod/trial-app-6cd98d67f4-2w42c     0/1     ContainerCreating   0          24s

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes         ClusterIP      10.43.0.1       <none>        443/TCP          50m
service/backend-service    LoadBalancer   10.43.19.211    <pending>     80:30447/TCP     29s
service/frontend-service   LoadBalancer   10.43.225.146   <pending>     8080:32576/TCP   27s
service/trial-service      LoadBalancer   10.43.60.43     <pending>     8080:32537/TCP   22s

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-backend-service    1         1         0       1            0           <none>          28s
daemonset.apps/svclb-frontend-service   1         1         0       1            0           <none>          27s
daemonset.apps/svclb-trial-service      1         1         0       1            0           <none>          22s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend-app   0/3     3            0           28s
deployment.apps/backend-app    0/3     3            0           29s
deployment.apps/trial-app      0/3     3            0           27s

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend-app-75cd57cfd   3         3         0       28s
replicaset.apps/backend-app-5f749755f4   3         3         0       29s
replicaset.apps/trial-app-6cd98d67f4     3         3         0       25s
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard\manifests>
```

##  Exposing Grafana

```bash
vagrant@localhost:~> kubectl get pod -n monitoring | grep grafana
prometheus-grafana-7f854c9f9-s585f                       2/2     Running      0          64m
vagrant@localhost:~> kubectl port-forward -n monitoring prometheus-grafana-7f854c9f9-s585f --address 0.0.0.0 3000
Forwarding from 0.0.0.0:3000 -> 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
```

##  Exposing the application

```bash

```

## Verify the monitoring installation
*TODO:* run `kubectl` command to show the running pods and services for all components. Take a screenshot of the output and include it here to verify the installation

PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard> kubectl get pods --all-namespaces
NAMESPACE       NAME                                                     READY   STATUS      RESTARTS   AGE
kube-system     helm-install-traefik-nxrhd                               0/1     Completed   0          106m
default         svclb-backend-service-qpr6x                              0/1     Pending     0          56m
default         svclb-trial-service-crlbn                                0/1     Pending     0          56m
default         backend-app-5f749755f4-dhxrg                             1/1     Running     3          56m
default         trial-app-6cd98d67f4-2w42c                               1/1     Running     2          56m
default         svclb-frontend-service-6rt6l                             1/1     Running     2          56m
monitoring      prometheus-kube-prometheus-operator-8554997fc-9k9nz      1/1     Running     2          76m
default         backend-app-5f749755f4-fbbkx                             1/1     Running     2          56m
default         frontend-app-75cd57cfd-qzgbc                             1/1     Running     2          56m
monitoring      prometheus-prometheus-node-exporter-77qjz                1/1     Running     2          76m
default         frontend-app-75cd57cfd-lzkp4                             1/1     Running     2          56m
kube-system     local-path-provisioner-7ff9579c6-nbl49                   1/1     Running     3          106m
monitoring      alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running     6          76m
default         frontend-app-75cd57cfd-cvnmv                             1/1     Running     1          56m
kube-system     coredns-88dbd9b97-nv6l9                                  1/1     Running     3          106m
monitoring      prometheus-kube-state-metrics-569d7854c4-x9f5b           1/1     Running     4          76m
kube-system     svclb-traefik-dvs6x                                      2/2     Running     4          105m
default         trial-app-6cd98d67f4-496bz                               1/1     Running     2          56m
monitoring      prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running     2          76m
observability   jaeger-operator-5977dbf59f-c2xmd                         1/1     Running     2          66m
default         trial-app-6cd98d67f4-b7rmx                               1/1     Running     2          56m
kube-system     metrics-server-7b4f8b595-lvbnq                           1/1     Running     2          106m
default         backend-app-5f749755f4-l4664                             1/1     Running     2          56m
monitoring      prometheus-grafana-7f854c9f9-s585f                       2/2     Running     2          76m
kube-system     traefik-5dd496474-ndn6f                                  1/1     Running     2          105m


PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard> kubectl get services --all-namespaces
NAMESPACE       NAME                                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
default         kubernetes                                           ClusterIP      10.43.0.1       <none>        443/TCP                        109m
kube-system     kube-dns                                             ClusterIP      10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP         109m
kube-system     metrics-server                                       ClusterIP      10.43.127.82    <none>        443/TCP                        109m
kube-system     traefik-prometheus                                   ClusterIP      10.43.82.175    <none>        9100/TCP                       108m
kube-system     prometheus-kube-prometheus-coredns                   ClusterIP      None            <none>        9153/TCP                       79m
monitoring      prometheus-kube-prometheus-prometheus                ClusterIP      10.43.209.235   <none>        9090/TCP                       79m
kube-system     prometheus-kube-prometheus-kube-scheduler            ClusterIP      None            <none>        10251/TCP                      79m
kube-system     prometheus-kube-prometheus-kube-proxy                ClusterIP      None            <none>        10249/TCP                      79m
kube-system     prometheus-kube-prometheus-kube-etcd                 ClusterIP      None            <none>        2379/TCP                       79m
kube-system     prometheus-kube-prometheus-kube-controller-manager   ClusterIP      None            <none>        10252/TCP                      79m
monitoring      prometheus-kube-state-metrics                        ClusterIP      10.43.50.121    <none>        8080/TCP                       79m
monitoring      prometheus-grafana                                   ClusterIP      10.43.232.244   <none>        80/TCP                         79m
monitoring      prometheus-kube-prometheus-alertmanager              ClusterIP      10.43.29.151    <none>        9093/TCP                       79m
monitoring      prometheus-kube-prometheus-operator                  ClusterIP      10.43.118.144   <none>        443/TCP                        79m
monitoring      prometheus-prometheus-node-exporter                  ClusterIP      10.43.250.150   <none>        9100/TCP                       79m
monitoring      alertmanager-operated                                ClusterIP      None            <none>        9093/TCP,9094/TCP,9094/UDP     78m
kube-system     prometheus-kube-prometheus-kubelet                   ClusterIP      None            <none>        10250/TCP,10255/TCP,4194/TCP   78m
monitoring      prometheus-operated                                  ClusterIP      None            <none>        9090/TCP                       78m
observability   jaeger-operator-metrics                              ClusterIP      10.43.197.144   <none>        8383/TCP,8686/TCP              67m
default         backend-service                                      LoadBalancer   10.43.19.211    <pending>     80:30447/TCP                   58m
default         trial-service                                        LoadBalancer   10.43.60.43     <pending>     8080:32537/TCP                 58m
default         frontend-service                                     LoadBalancer   10.43.225.146   10.0.2.15     8080:32576/TCP                 58m
kube-system     traefik                                              LoadBalancer   10.43.248.101   10.0.2.15     80:30603/TCP,443:31841/TCP     108m

## Setup the Jaeger and Prometheus source
*TODO:* Expose Grafana to the internet and then setup Prometheus as a data source. Provide a screenshot of the home page after logging into Grafana.



## Create a Basic Dashboard
*TODO:* Create a dashboard in Grafana that shows Prometheus as a source. Take a screenshot and include it here.

## Describe SLO/SLI
*TODO:* Describe, in your own words, what the SLIs are, based on an SLO of *monthly uptime* and *request response time*.

## Creating SLI metrics.
*TODO:* It is important to know why we want to measure certain metrics for our customer. Describe in detail 5 metrics to measure these SLIs. 

## Create a Dashboard to measure our SLIs
*TODO:* Create a dashboard to measure the uptime of the frontend and backend services We will also want to measure to measure 40x and 50x errors. Create a dashboard that show these values over a 24 hour period and take a screenshot.

## Tracing our Flask App
*TODO:*  We will create a Jaeger span to measure the processes on the backend. Once you fill in the span, provide a screenshot of it here.

## Jaeger in Dashboards
*TODO:* Now that the trace is running, let's add the metric to our current Grafana dashboard. Once this is completed, provide a screenshot of it here.

## Report Error
*TODO:* Using the template below, write a trouble ticket for the developers, to explain the errors that you are seeing (400, 500, latency) and to let them know the file that is causing the issue.

TROUBLE TICKET

Name:

Date:

Subject:

Affected Area:

Severity:

Description:


## Creating SLIs and SLOs
*TODO:* We want to create an SLO guaranteeing that our application has a 99.95% uptime per month. Name three SLIs that you would use to measure the success of this SLO.

## Building KPIs for our plan
*TODO*: Now that we have our SLIs and SLOs, create KPIs to accurately measure these metrics. We will make a dashboard for this, but first write them down here.

## Final Dashboard
*TODO*: Create a Dashboard containing graphs that capture all the metrics of your KPIs and adequately representing your SLIs and SLOs. Include a screenshot of the dashboard here, and write a text description of what graphs are represented in the dashboard.  
