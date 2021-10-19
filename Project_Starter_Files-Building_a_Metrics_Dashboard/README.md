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
kubectl get pods --namespace=monitoring
NAME                                                     READY   STATUS    RESTARTS   AGE
prometheus-prometheus-node-exporter-wdst4                1/1     Running   0          2m45s
prometheus-kube-prometheus-operator-8554997fc-dvxnb      1/1     Running   0          2m45s
prometheus-kube-state-metrics-569d7854c4-2czlx           1/1     Running   0          2m45s
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          2m3s
prometheus-grafana-7f854c9f9-78cz5                       2/2     Running   0          2m45s
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0          118s
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
NAME                                   READY   STATUS    RESTARTS   AGE
pod/jaeger-operator-5977dbf59f-krb2r   1/1     Running   0          52s

NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
service/jaeger-operator-metrics   ClusterIP   10.43.201.73   <none>        8383/TCP,8686/TCP   5s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jaeger-operator   1/1     1            1           53s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/jaeger-operator-5977dbf59f   1         1         1       53s

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
NAME                                READY   STATUS    RESTARTS   AGE
pod/svclb-backend-service-f78xq     1/1     Running   0          3m13s
pod/svclb-frontend-service-rq9mk    1/1     Running   0          3m8s
pod/svclb-trial-service-bbx6b       1/1     Running   0          3m1s
pod/frontend-app-86c746c449-pgddj   1/1     Running   0          3m12s
pod/frontend-app-86c746c449-9kbc4   1/1     Running   0          3m10s
pod/frontend-app-86c746c449-b74fk   1/1     Running   0          3m10s
pod/backend-app-76fddbbb79-kvw9q    1/1     Running   0          3m14s
pod/trial-app-67788fc54c-lvtrh      1/1     Running   0          3m7s
pod/trial-app-67788fc54c-ksv9v      1/1     Running   0          3m7s
pod/trial-app-67788fc54c-v4sb9      1/1     Running   0          3m7s
pod/backend-app-76fddbbb79-2lnch    1/1     Running   0          3m14s
pod/backend-app-76fddbbb79-zfmw4    1/1     Running   0          3m14s

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes         ClusterIP      10.43.0.1       <none>        443/TCP          24m
service/backend-service    LoadBalancer   10.43.113.189   10.0.2.15     8081:32107/TCP   3m15s
service/frontend-service   LoadBalancer   10.43.68.45     10.0.2.15     8082:31037/TCP   3m11s
service/trial-service      LoadBalancer   10.43.119.175   10.0.2.15     8083:31413/TCP   3m7s

NAME                                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-backend-service    1         1         1       1            1           <none>          3m15s
daemonset.apps/svclb-frontend-service   1         1         1       1            1           <none>          3m11s
daemonset.apps/svclb-trial-service      1         1         1       1            1           <none>          3m7s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/frontend-app   3/3     3            3           3m15s
deployment.apps/trial-app      3/3     3            3           3m11s
deployment.apps/backend-app    3/3     3            3           3m15s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/frontend-app-86c746c449   3         3         3       3m14s
replicaset.apps/trial-app-67788fc54c      3         3         3       3m9s
replicaset.apps/backend-app-76fddbbb79    3         3         3       3m15s
```

##  Exposing Grafana

```bash
vagrant@localhost:~> kubectl get pod -n monitoring | grep grafana
prometheus-grafana-7f854c9f9-78cz5                       2/2     Running   0          17m
vagrant@localhost:~> kubectl port-forward -n monitoring prometheus-grafana-7f854c9f9-78cz5 --address 0.0.0.0 3000
Forwarding from 0.0.0.0:3000 -> 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
Handling connection for 3000
```
![Grafana login](https://github.com/nononoelsg/MetricsDashboard/blob/master/Project_Starter_Files-Building_a_Metrics_Dashboard/answer-img/Grafana%20login%20page.PNG)

##  Exposing the application

```bash
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard>  kubectl port-forward svc/frontend-service 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080
```
![Frontend-service](https://github.com/nononoelsg/MetricsDashboard/blob/master/Project_Starter_Files-Building_a_Metrics_Dashboard/answer-img/frontend.PNG)

## Verify the monitoring installation
run `kubectl` command to show the running pods and services for all components. Take a screenshot of the output and include it here to verify the installation

```bash
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard> kubectl get pods --all-namespaces
NAMESPACE       NAME                                                     READY   STATUS      RESTARTS   AGE
kube-system     local-path-provisioner-7ff9579c6-d6d74                   1/1     Running     0          31m
kube-system     metrics-server-7b4f8b595-2jcjz                           1/1     Running     0          31m
kube-system     helm-install-traefik-2ntd5                               0/1     Completed   0          31m
kube-system     svclb-traefik-hlgbl                                      2/2     Running     0          30m
monitoring      prometheus-prometheus-node-exporter-wdst4                1/1     Running     0          22m
monitoring      prometheus-kube-prometheus-operator-8554997fc-dvxnb      1/1     Running     0          22m
monitoring      prometheus-kube-state-metrics-569d7854c4-2czlx           1/1     Running     0          22m
monitoring      alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running     0          22m
monitoring      prometheus-grafana-7f854c9f9-78cz5                       2/2     Running     0          22m
monitoring      prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running     0          22m
observability   jaeger-operator-5977dbf59f-krb2r                         1/1     Running     0          17m
default         svclb-backend-service-f78xq                              1/1     Running     0          10m
default         svclb-frontend-service-rq9mk                             1/1     Running     0          10m
default         svclb-trial-service-bbx6b                                1/1     Running     0          10m
default         frontend-app-86c746c449-pgddj                            1/1     Running     0          10m
kube-system     coredns-88dbd9b97-26fmc                                  1/1     Running     0          31m
default         frontend-app-86c746c449-9kbc4                            1/1     Running     0          10m
kube-system     traefik-5dd496474-zlmh5                                  1/1     Running     0          30m
default         frontend-app-86c746c449-b74fk                            1/1     Running     0          10m
default         backend-app-76fddbbb79-kvw9q                             1/1     Running     0          10m
default         trial-app-67788fc54c-lvtrh                               1/1     Running     0          10m
default         trial-app-67788fc54c-ksv9v                               1/1     Running     0          10m
default         trial-app-67788fc54c-v4sb9                               1/1     Running     0          10m
default         backend-app-76fddbbb79-2lnch                             1/1     Running     0          10m
default         backend-app-76fddbbb79-zfmw4                             1/1     Running     0          10m
```

```bash
PS C:\Users\Noel\Desktop\udacity\MetricsDashboard\Project_Starter_Files-Building_a_Metrics_Dashboard> kubectl get services --all-namespaces
NAMESPACE       NAME                                                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                        AGE
default         kubernetes                                           ClusterIP      10.43.0.1       <none>        443/TCP                        32m
kube-system     kube-dns                                             ClusterIP      10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP         32m
kube-system     metrics-server                                       ClusterIP      10.43.85.78     <none>        443/TCP                        32m
kube-system     traefik-prometheus                                   ClusterIP      10.43.192.44    <none>        9100/TCP                       31m
kube-system     traefik                                              LoadBalancer   10.43.91.46     10.0.2.15     80:31185/TCP,443:32531/TCP     31m
monitoring      prometheus-prometheus-node-exporter                  ClusterIP      10.43.178.21    <none>        9100/TCP                       23m
kube-system     prometheus-kube-prometheus-coredns                   ClusterIP      None            <none>        9153/TCP                       23m
kube-system     prometheus-kube-prometheus-kube-scheduler            ClusterIP      None            <none>        10251/TCP                      23m
kube-system     prometheus-kube-prometheus-kube-proxy                ClusterIP      None            <none>        10249/TCP                      23m
kube-system     prometheus-kube-prometheus-kube-etcd                 ClusterIP      None            <none>        2379/TCP                       23m
kube-system     prometheus-kube-prometheus-kube-controller-manager   ClusterIP      None            <none>        10252/TCP                      23m
monitoring      prometheus-kube-state-metrics                        ClusterIP      10.43.21.143    <none>        8080/TCP                       23m
monitoring      prometheus-grafana                                   ClusterIP      10.43.33.105    <none>        80/TCP                         23m
monitoring      prometheus-kube-prometheus-prometheus                ClusterIP      10.43.55.187    <none>        9090/TCP                       23m
monitoring      prometheus-kube-prometheus-alertmanager              ClusterIP      10.43.130.31    <none>        9093/TCP                       23m
monitoring      prometheus-kube-prometheus-operator                  ClusterIP      10.43.88.13     <none>        443/TCP                        23m
kube-system     prometheus-kube-prometheus-kubelet                   ClusterIP      None            <none>        10250/TCP,10255/TCP,4194/TCP   23m
monitoring      alertmanager-operated                                ClusterIP      None            <none>        9093/TCP,9094/TCP,9094/UDP     23m
monitoring      prometheus-operated                                  ClusterIP      None            <none>        9090/TCP                       23m
observability   jaeger-operator-metrics                              ClusterIP      10.43.201.73    <none>        8383/TCP,8686/TCP              17m
default         backend-service                                      LoadBalancer   10.43.113.189   10.0.2.15     8081:32107/TCP                 11m
default         frontend-service                                     LoadBalancer   10.43.68.45     10.0.2.15     8082:31037/TCP                 11m
default         trial-service                                        LoadBalancer   10.43.119.175   10.0.2.15     8083:31413/TCP                 11m
```
## Setup the Jaeger and Prometheus source
Expose Grafana to the internet and then setup Prometheus as a data source. Provide a screenshot of the home page after logging into Grafana.

![Grafana dashboard](https://github.com/nononoelsg/MetricsDashboard/blob/master/Project_Starter_Files-Building_a_Metrics_Dashboard/answer-img/Grafana_dashboard.PNG)

## Create a Basic Dashboard
Create a dashboard in Grafana that shows Prometheus as a source. Take a screenshot and include it here.

![Grafana Basic dashboard](https://github.com/nononoelsg/MetricsDashboard/blob/master/Project_Starter_Files-Building_a_Metrics_Dashboard/answer-img/basicDashboard.PNG)

## Describe SLO/SLI
Describe, in your own words, what the SLIs are, based on an SLO of *monthly uptime* and *request response time*.

A Service-Level Indicator (SLI) is a specific metric used to measure the performance of a service.

SLO (service Level Object) is something like your company require the application to have an monthly uptime of 99.9% and the average request response time of a web request would take < 1 seconds to complete.

## Creating SLI metrics.
It is important to know why we want to measure certain metrics for our customer. Describe in detail 5 metrics to measure these SLIs. 

1. disk, cpu and memory usage.
2. Services uptime
3. http error log
4. Traffic status and logs
5. API call montoring

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
