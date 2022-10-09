# Prometheus, kube Static Metrics and Integrate Grafana with Kubernetes
### Overview
- This set up with show how to use Prometheus tools to monitor Kubernetes cluster.
- You wil learn how to set up Prometheus server, metric exporters, and kube-State-Metrics as well as how to pull, scrape and collect metrics, configure Alermanager alerts, and Grafana dashboards.
- Kube State Metric is a service that communicates with the kubernetes API server to obtain information about all API objects such as deployments, pods, and DaemonSets. 
- Few key objects you can monitor with Kube State Metrics:
 * Monitor node status, node capacity (CPU and memory)
 * Monitor replica-set compliance (desire/available/unavailable/updated status of replicas per deployment)
 * Monitor pod status (waiting, running, ready, etc)
 * Monitor the resources requests and limits

#### 📚 Resources
* [Website project (fork this)](https://github.com/mnforba/Prometheus-kubeStaticMetrics-Grafana-Kubernetes.git)
### Prerequisites
Before you begin with this guide, below are the prerequisites:
- A Kubernetes cluster (You can refer to kubernetes docs or any other kubernetes engine)
- a fully configured kubectl (https://kubernetes.io/docs/tasks/tools/) command-line interface on your local machine
### Create Namespace
- Build a namespace for all our monitoring components
- A namespace is where kubernetes starts all its resources. 
- For easy reference, our namespace is named `monitoring`
- Apply changes  by runing the command: `kubectl create -f monitoring-namespace.yml`

### Create Roles & Service Account
- we create a ClusterRole to allow Prometheus access to nodes and pods across the cluster to obtain all of the metrics
- We are going to make a default service accoun for the Monitoring namespace. This means Prometheus will use this service account by default
- Apply the changes by running the command: `kubectl create -f clusterRole.yml`

### Create a ConfigMap
- First we create a ConfigMap with the name = `prometheus-server-conf` in `monitoring` the namespace where the Prometheus Deployment should be running.
- `prometheus.rules` will contain all the alert rules for sending alerts to the alert manager
- The `prometheus.yml` contains all the configurations to dynamically discover pods and services running in the kubernetes cluster.
- We have the following `scrape jobs` in our Prometheus scrape configuration
   * `kubernetes-apiservers`: Collects all the metrics from the API servers
   * `kubernetes-nodes`: Collect all kubernetes node metrics
   * `kubernetes-pods`: All the pod metrics will be discovered if the pod metadata is annotated with `prometheus.io/scrapre` and `prometheus.io/port` annotations.
   * `kubernetes-cadvisor`: Collect all cAdvisor metrics.
   * `kubernetes-service-endpoints`: All the Service endpoints will be scrapped if the service metadata is annotated with `prometheus.io/scrape` and `prometheus.io/port`. 
- Apply changes by running the command: `kubectl create -f config-map.yml`

### Create a Prometheus Deployment
- The Prometheus config map is mounted as a file in /etc/prometheus. 
- We are using the official Prometheus image from docker
- the `prometheus.yml` key containing Prometheus's configuration is mounted to /etc/prometheus/prometheus.yml
- Apply changes by running the command: `kubectl apply -f prometheus-deployment.yml`

## Connection to Prometheus Dashboard
- There are two ways to access the Prometheus dashboard that has been deployed.
    * Using kubectl Port Forwarding with command: `kubectl poirt-forward <your-prometheus-deployment-podName> 8080:9090 -n monitoring`.
    * Using NodePort/Load Balancer by exposing the Prometheus Deployment. For this we create `prometheus-service.yml`, to access the Prometheus dashboard over an IP or a DNS name.
    * We will expose Prometheus on all kubernetes node IP on port 30000
    * `prometheus.io/scrape` will scrape all pods and, if set to `false`, this annotation will exclude the pod from the scraping process.
    * `prometheus.io/port` scrape the pod on the indicated port instead of the pod's declared ports.
- Apply changes by running the command: `kubectl create -f prometheus-service.yml --namespace=monitoring`
- View the dashboard on `localhost:9090`

## Setting up Kube State Metric in our cluster to monitor all the API objects

### Create a Service Account
- Apply the changes by running the command: `kubectl apply -f service-account.yml`.
- `aws s3 cp` allows us to copy a file to and from AWS S3 

### Create Cluster Role
- ClusterRole will allow Kube-State-Metrics to access all the Kubernetes API objects.
- Apply the changes by running the command: `kubectl apply -f cluster-role.yml`.

### Create Cluster Role Binding
- To bind the ClusterRole with the Service Account, we will need to create ClusterRoleBinding
- Apply changes by running the command: `kubectl apply -f cluster-role-binding.yml`.
### Create a Deployment 
- using the Kube-State-Metrics docker image, we will create a Deployment
- Apply the changes by running the command: `kubectl apply -f deployment.yml`.
- To view the metric locally we can use port-forwarding. For that we get the name of the deployment pod with `kubectl get pods -n kube-system`.
- Use the pod name for port forwarding: `kubectl port-forward <pod-name> 8080:8080 -n kube-system`
-
## Setup Grafana on Kubernetes
- Through Prometheus we can construct a dashboard on Grafana for all kubernetes metrics.

### Create Config Map 
- Create Config Map containing the data source configuration for Prometheus.
- You can add more data from Grafana if necessary
- Apply the changes by running the command: `kubectl apply -f grafana-datasource-config.yml`.

## Create a Grafana Deployment

- Create a Deployment using the Grafana docker image
- Apply the changes by running the command: `kubectl apply -f grafana-deployment.yml`.
- Access Grafana on localhost using port-forwarding. First get the name of the pod with `kubectl get pods -n monitoring`.
- Use the Grafana pod name for port forwarding using `kubectl port-forward <grafana-pod-name> 3000:3000 -n monitoring`.
- The default credentials are listed below:
  * user: admin
  * Password: admin
