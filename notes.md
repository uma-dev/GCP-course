# GCP Key features

## Zones

GCP have 20+ zones around the world.
Each zone have al least 3 zones connected with low-latency links. Each zone has one or more discrete clusters.
- Region and zone utilization is based on latency, HW requirements, availability, regulations and cost.
- Instances runs in a Region, but images and templates (unless uses zonal machines) are global. 

----

## GCloud CLI 

- gcloud is part of Google Cloud SDK (requires Python) you can install it on your computer.
- Cloud Shell is a online shell, it has installed gcloud.
- Some GCP services have CLI tools such as BigQuery (bq), Cloud BigTable (cbt), Cloud Storage (gsutil) and Kubernetes (kubectl).

The general structure of a command is as follows:
  - ```gcloud GROUP SUBGROUP ACTION ```

### Configuration

  - ```gcloud --version ```
  - ```gcloud init ```
  - ```gcloud config list ```
  - ```gcloud config list account ```
  - ```gcloud config list compute/zone```
  - ```gcloud config configurations list ```
  - ```gcloud config set project NAME ```
  - ```gcloud config configurations active NAME ```
  - ```gcloud config configurations create NAME ```

----

## Managed Services in Google Cloud


| Service | Details | Category|
| :------- | :--------| :-------:|
| Compute Engine|High performance and general purpose VM that can scale globally| IaaS |
|Google Kubernetes Engine|Orchestrate containerized microservices on Kubernetes. Needs advanced **cluster configuration and monitoring**| CaaS|
|App Engine|Enables highly scalable applications on "fully managed platform" | PaaS (CaaS Serverless)|
|Cloud Functions |Build event driven applications using simple, single-purpose functions| FaaS Serverless|
|Cloud Run |Highly scalable containerized applications.  **Does not need a cluster**| CaaS (Serverless)| 

----

## Compute Engine (Virtual Machines)

Google Compute Engine is a SERVICE to manage and create lifecycle of VM.
- You cannot  change a instance template, you can copy it and modify the copy.
- Custom images can be used to create vm-instances with OS patches and/or software preinstalled.
- You can share an image across projects.
- **Hardening** an image is the process of creating an image from a vm-instance that meets the security requirements of an enterprise.
- It is preferred to use a custom image rather than startup scripts.
- For vCPUs/ memory changes, a VM instance has to be stopped.
- A good practice is use labels to organize VMs.
- Management:
  - Default metrics: CPU, Network and Disk Throughput.
  - Cloud agent: Memory Utilization and Disk space.

| Advantages | Disadvantages |
| :------- |  :-------|
|More flexibility | More responsibility ( Availability, installing software, choosing hardware and images)|

### Cost 

In GCP you are billed per second.  For money saving, always choose the right machine type for your needs and image for your workload. Whenever you have a predictable resources need, you can commit the use of a VM (Kubernetes or  Compute engine) for 1 or 3 years and get up to 70% of discount.
- **Preemptible VM** only last **24 hours**  or less due to system demands and they are 80% more cheaper but have a lot of disadvantages, e.g. not always available.
- Also, Preemptible VM doesn't work with free tier credits.
- The updated version of Preemptible VM is **Spot VM** and the key difference is that they **don't  have a maximum runtime**.
- Spot VM has a Dynamic pricing which represents 60-90% of discount to on-demand VM. 
- When you apart an static ip address you'll be billed, even if you aren't using it!
- The storage attached to a compute instance generate a bill. even if the compute instance is stopped. 
- Solutions to those problems are: Remove any static ip address that is not used and create budget alerts.
- A budget can send you email alerts about the limits of your budget.

### Live migration 

Whenever you need a system update (HW or SW) you can use Live migration. This means that your running instance is migrated to another host on the same zone. Also, this process doesn't change any attributes or properties of the VM.
- Preemptible instances doesn't support this feature.
- The way to configure the Live Migration is through Availability Policy section.

### GPU

- You can customize your VM with a GPU or use the predefined family of VM with GPU.
- You can add GPU to your machine but you have to use images with GPU libraries and supported machine types (shared core and memory-optimized machines does not support this feature).
- Adding GPU imply that VM won't support Live migration.
- GPU carry higher cost, but is useful for High performance, math and graphics intensive.

### Instances

  - ```gcloud compute instances create NAME ```
  - ```gcloud compute instances describe NAME ```
  - ```gcloud compute instances delete NAME ```
  - ```gcloud compute machine-types list ```
  - ```gcloud compute machine-types list --filter "zone:NAME"``` 
  - ```gcloud compute zones list --filter "region:(us-west2 us-west1)"```
  - ```gcloud compute zones list --filter region:us-west1 --sort-by ~name (~ means reverse order) ```
  - ```gcloud compute instances create my-test-vm --source-instance-template=NAME ```

### Instances group

Instances groups can be: 
- **Managed** (MIG) Identical instances autoscaling, auto-healing, etc compatible. Also you can use:  
  - Rolling updates (step by step updates)
  - Canary deployment (test newer versions on a group of instances)

- **Unmanaged** (UIG) different type instances not compatible with autoscaling, auto-healing, etc.

### Some scenarios and solutions

| Scenario | Solution |
| :------- | :--------|
| MIG managed  application that survives ZONAL failures | Create multiple zones MIG. |
| Group of multiple VMs with different configurations | Create an UIG. |
|Preserve state in a MIG (like databases)| Create stateful MIG. |
|High availability in MIG when there are hw or sw updates| Create a template with Availability police for automatic restart = on and on-host maintenance = migrate VM instance. |
| Remove unhealthy instances automatically | Create a MIG with auto-healing properly configured|
|Avoid scale up and downs|Cool down period (initial delay)| 

### Cloud Load balancing 

Enables: High availability, Auto Scaling and Resiliency.
Most common protocols such as:
- Network layer: Internet Protocol (IP)
- Transport layer: TCP, TLS (Secure version of TCP) , UDP 
- Application layer: HTTP, SMTP (email), FTP etc

### Terminology in google cloud balancing

- **Backend** refers to a group of endpoints that receives traffic from the load balancer

- **Frontend** specify an IP and protocol

- **Host and path rules** defines the rules for redirecting traffic to different backends based on:

  - Path
  - Host 
  - HTTP headers

### Scenarios

| Scenario | Solution |
| :------- | :--------|
| Only healthy instances can receive traffic| Use health check |
|High availability for instances| Use a MIG  across different regions and configure a Load balancer.|
|Route request to multiple microservices with the same Load balancer| Configure the "The host and path rules" to redirect to specific  microservice depending on the path|
|Load balance Global external HTPPS traffic across backend instances, and multiple regions| Global external HTTP(S) Load balancer|
|SSL termination for Global non-HTTPS traffic with Load balancing| SSL Proxy Load Balancer |

### Instance-groups

 - ```gcloud compute instance-groups list```
 - ```gcloud compute instance-groups managed list```
 - ```gcloud compute instance-groups managed create NAME --zone ZONE --template TEMPLATE --size SIZE_NUMBER```
 - ```gcloud compute instance-groups managed set-autoscaling NAME --zone=INSTANCE_ZONE --max-num-replicas=NUM_REPLICAS```
 - ```gcloud compute instance-groups managed delete-autoscaling NAME --zone=INSTANCE_ZONE```
 - ```gcloud compute instance-groups managed recreate-instances GROUP_NAME --instances=INSTANCE_NAME```
 - ```gcloud compute instance-groups managed set-instance-template GROUP_NAME --template=NEW_TEMPLATE_NAME```
 - ```gcloud compute instance-groups managed resize GROUP_NAME --size=SIZE```

 After some update you can manage the new release without downtime:
 - ```gcloud compute instance-groups managed rolling action restart GROUP_NAME ```
 - ```gcloud compute instance-groups managed rolling action replace GROUP_NAME ```
-----
## App Engine 
Is a service in Google cloud since 2008 which provides end to end application management, is a good option for microservices. Also, App Engine in regional, so you cannot change the region of an application. You pay for resources provisioned.
Some of the features that supports:
- Java, Go, .NET, Node.js, PHP, Python, Ruby preconfigured runtimes. 
- Use custom runtime and write in any language
- Connect to variety of Google Cloud storage products like Cloud SQL
- Automatic load balancing and auto-scaling
- Managed platform updates and application health monitoring
- Supports traffic splitting 

| Advantages | Disadvantages |
| :------- |  :-------|
| Lesser responsibility | Lower flexibility ( Cannot add GPU) and lack of containers orchestration|

### Environments

App engine provides two different environments: **Standard** and **Flexible**.

- In **Standard** environment applications runs in __language specific sandboxes__ and two versions. It does not support some runtimes like C++ (.NET).
  - In the V1 (old versions) Python and PHP runtimes have some additional restrictions with network and libraries.
  - In the V2 (new versions) there are not restrictions.

- In **Flexible** environment  applications runs in Docker containers which uses Docker Engine __virtual machines__ and support any runtime. Also provides access to background processes and local disks.

Here it is a simple comparison:

| Feature | Standard | Flexible |
| :------- |  :-------| :---------|
| Pricing | Instance hours| vCPU, Memory and persistent disk |
|Scaling| Manual, **Basic** (Instances are created when request are received) and Automatic (continuously running workloads)| Manual and Automatic|
| Scaling to zero|**Yes** |No (Minimum one container running)|
| Instance startup time| **Seconds**| Minutes|
|Rapid scaling (up and down) | **Yes**| No|
|Max. request timeout | 1-10 minutes | **60 minutes** |
|Local disk (storage) | Mostly except for Python and PHP. Can write to /tmp | Yes, ephemeral. New disk every startup |
|SSH for debugging| No |**Yes**|

### Request routing

Routing URLs have the following structure:
 - __default__ https://PROJECT_ID.REGION.appspot.com 
 - __specific service__ https://SERVICE-dot-PROJECT_ID.appspot.com 
 - __specific version of service__ https://VERSION-dot-SERVICE-dot-PROJECT_ID.REGION.appspot.com
 

### Commands

```gcloud app services list```
```gcloud app versions list```
```gcloud app instances list```

To create type 

```gcloud app create --region=REGION```

After a change in the app you must run

```gcloud app deploy (--version-NUMBER)```

To use other filename of yaml use: 

```gcloud app deploy app.yaml```

After the new deploy the old version still running, you can check the URL in: 

```gcloud app browse --version="NUMBER"```'
```gcloud app browse --service=NAME --version="NUMBER"```

After a change you can test your new version without launching it in the main URL with:

```gcloud app deploy --version-NUMBER --no-promote ```

And find the new version using the browse option described earlier. Also you can open the Console dashboard with the following command:

```gcloud app open-console --version=VERSION```

### Traffic splitting 

```gcloud app services set-traffic --split-by=OPTION ```
- __Based on IP__: Accuracy issues because IP change, also the request from a single VPN IP cause to request go to the same version.
- __Based on Cookies__: Cookies can be controlled from application.
- __Random__: Splitting randomly.

To split traffic **by IP** between versions use:

```gcloud app services --set-traffic --splits=vNUMBER=.5,vNUMBER=.5```

To split it by **random** use:

```gcloud app services set-traffic --splits=v3=.5,v2=.5 --split-by=random```

### Deploy an application without downtime 
- Option 1: Deploy and shift at once with ```gcloud app deploy```
- Option 2: Deploy with no promote  ```gcloud app deploy --no-promote``` and then shift traffic to second version with:
  - All at once ```gcloud app services set-traffic SERVICE --splits V2=1```
  - Gradually with migrate (not supported in flexible) ```--migrate```
  - Manually with ```gcloud app services set-traffic SERVICE --splits=v2=.5,v1=.5```

### Yaml files

 ```gcloud app deploy FILE.yaml```

#### Cron

- Allows you to run scheduled jobs for example:
  - Send report by email every day 
  - Refresh cache data every 30 minutes
- Configured using cron.yaml and the command 

#### Override routing rules
- dispatch.yaml

#### Manage task queues
- queue.yaml

### Scenarios 

| Scenario | Solution |
| :------- |  :-------| 
|Two Google App Engine Apps in the same project | It is impossible to deploy two apps in the same project in App Engine, instead you can deploy multiple services and versions of each service |
|Two Google App services in the same app | It is possible as described earlier |
|Want to move Google App Engine App to different region | Cannot move apps from App Engine to another region, new project will be needed |
|Perform Canary deployments| You can deploy a new version without shifting traffic and then split traffic/migrate to the new version |

## Google Kubernetes Engine

Open source container orchestration solution. Kubernetes also supports **Stateful** deployments. It needs a Cluster configuration (Remember that autopilot simplifies cluster management) and upgrades. It uses a **Container-Optimized OS** (hardened by Google), supports persistent disks, local SSD and it is also possible to add GPU.
The GKE provides features such as:
- Zero downtime deployments
- Service discovery
- Self healing
- Load balancer
- Auto scaling

### Cluster modes
- Autopilot: The goal of autopilot is to reduce the cost of operating a container. Google manages the cluster. 
- Standard: Pay and configure each node. 
 
### Commands 
Create a cluster

```gcloud container clusters create```

Connect to cluster with:

```gcloud container clusters get-credentials CLUSTER_NAME --zone=ZONE --project=PROJECT_NAME```

Then create a deployment:

```kubectl create deployment NAME --image=in28min/hello-world-rest-api:0.0.1.RELEASE```

Can check the deployment with 

```kubectl get deployment```

Then expose the deployment (to the external world) to create a service

```kubectl expose deployment NAME --type=LoadBalancer --port=8080```

and then get the services with:

```kubectl get services```

you can also scale the number of instances (services) of deployment with:

```kubectl scale deployment NAME --replicas=3```

and you can verify that there are 3 instances now with:

```kubectl get deployment```

You can also manually scale the number of nodes in the cluster with:

```gcloud container clusters resize CLUSTER_NAME --node-pool NODE_POOL_NAME --num-nodes=2 --zone=ZONE```

Also you can auto scale the deployment (instances) with:

```kubectl autoscale deployment DEPLOYMENT_NAME --max=NODES_NUMBER --cpu-percent=PERCENTAGE```

Similarly you can set autoscale the Kubernetes cluster with: 

```gcloud container clusters update CLUSTER_NAME --enable-autoscaling --min-nodes=MIN --max-nodes=MAX```

to see the pods you can use:

```kubectl get pods```

to see the hpa:

```kubectl get hpa```

the, you can create a config map with:

```kubectl create configmap CONFIGMAP_NAME --from-literal=RDS_DB_NAME=todos```

the you can see it with:

```kubectl get configmap```

or describe it with:

```kubectl describe configmap CONFIGMAP_NAME```

You can also store values as a secret (also called a secret map) with:

```kubectl describe secret SECRET_NAME```

It is important to remember that there are **two ways to deploy things in Kubernetes**, first one is with commands as described earlier or you can use a declarative option using YAML files.

Even from terminal you can deploy with ```kubectl apply -f deployment.yaml```

Also, you can list your node-pools with

```gcloud container node-pools list --zone=us-central1-c```

You can deploy a new service which need nodes with GPU:

- Attach a new node pool with: 
```gcloud container node-pools create POOL_NAME --cluster CLUSTER_NAME```

then, you can view it with:
 
```gcloud container node-pools list --cluster CLUSTER_NAME```

Finally you can deploy the new service to the new pool by setting the node selector (The node selector is where you can choose the pool used in the service).

nodeSelector: cloud.google.com/gke-nodepool:POOL_NAME

To delete  microservices use: 
```kubectl delete service```
```kubectl delete deployment```

To delete cluster use: 

```gcloud container clusters delete```

### Clusters

- Cluster is a group of Compute Engine instances, it has:
   - Master node (which manages the cluster).
      - API server which handles communications with nodes and outside.
      - Scheduler which decides placement of pods.
      - Control manager which manages deployments and replicasets.
      - etcd which is a distributed database storing the cluster state.
   - Worker nodes (who run the workloads [pods]).
      - Kubelet which manages the communication with master node(s).

- The above statements means that some CPU on the nodes is reserved by control plane.

- There are lots of types of clusters: 
    - Zonal cluster 
      - Single zone (Single control plane, nodes in same zone)
      - Multi zonal (Single control plane, nodes in different zones)
    - Regional (Multiple control plane, nodes in different regions)
    - Private (only internal IP address in VPC) 
    - Alpha (early features)

- Node pool is a set of nodes with the same configuration within a cluster.

### Pods 

They are the smallest deployable unit in Kubernetes and they can contain one or more containers. Some of their features are:

- Each pod is assigned an ephemeral IP address
- The containers inside a pod shares:
  - Network
  - Storage
  - IP Address
  - Ports 
  - Volumes

### Deployments

A deployment represents a service with all of its versions.  It **ensures new releases with zero downtime**

```kubectl set image deployment DEPLOYMENT_NAME ACTUAL_IMAGE=NEW_IMAGE```

**Replica set** is a way to ensure that a specific number of pods are running specific microservice version. You can view the replicasets with:

```kubectl get replicasets```

### Services 
- Cluster IP (communication inside the cluster)
- Load balancer (exposes service using cloud provider's load balancer)
- Node Port (exposes service on each node's IP at a static port)

### Image Repository

Container Registry is a private location to store Images repositories provided by GCP. 
- It is similar to Docker Hub, which is a public container registry.
- Can secure images and analyze for vulnerabilities. 
- Naming: **HostName/ProjectID/Image:Tag-gcr.io/ProjectName/Microservice:version**

You can create your own images following the next steps:

- Build an image with ```docker build -t```
- Run it with ```docker run -d -p```
- Push it with ```docker push```

## Delete the GKE service deployment and cluster step by step

- Delete the service with: ```kubectl delete service SERVICE_NAME```
- Delete the deployment with: ```kubectl delete deployment DEPLOYMENT_NAME```
- Delete the cluster with: ```gcloud container cluster delete CLUSTER_NAME```