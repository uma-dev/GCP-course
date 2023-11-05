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
|Cloud Run |Highly scalable containerized applications. **Does not need a cluster**| CaaS (Serverless)| 

----

## Compute Engine (Virtual Machines)

Google Compute Engine is a SERVICE to manage and create lifecycle of VM.
- You cannot change a instance template, you can copy it and modify the copy.
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

In GCP you are billed per second. For money saving, always choose the right machine type for your needs and image for your workload. Whenever you have a predictable resources need, you can commit the use of a VM (Kubernetes or Compute engine) for 1 or 3 years and get up to 70% of discount.
- **Preemptible VM** only last **24 hours**  or less due to system demands and they are 80% more cheaper but have a lot of disadvantages, e.g. not always available.
- Also, Preemptible VM doesn't work with free tier credits.
- The updated version of Preemptible VM is **Spot VM** and the key difference is that they **don't have a maximum runtime**.
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
| MIG managed application that survives ZONAL failures | Create multiple zones MIG. |
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
|High availability for instances| Use a MIG across different regions and configure a Load balancer.|
|Route request to multiple microservices with the same Load balancer| Configure the "The host and path rules" to redirect to specific microservice depending on the path|
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

- In **Flexible** environment applications runs in Docker containers which uses Docker Engine __virtual machines__ and support any runtime. Also provides access to background processes and local disks.

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
- Option 2: Deploy with no promote ```gcloud app deploy --no-promote``` and then shift traffic to second version with:
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

To delete microservices use: 
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

A deployment represents a service with all of its versions. It **ensures new releases with zero downtime**

```kubectl set image deployment DEPLOYMENT_NAME ACTUAL_IMAGE=NEW_IMAGE```

**Replica set** is a way to ensure that a specific number of pods are running specific microservice version. You can view the replicasets with:

```kubectl get replicasets```

### Type of Services 
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

### Delete the GKE service deployment and cluster step by step

- Delete the service with: ```kubectl delete service SERVICE_NAME```
- Delete the deployment with: ```kubectl delete deployment DEPLOYMENT_NAME```
- Delete the cluster with: ```gcloud container cluster delete CLUSTER_NAME```

## Cloud Functions 

Is useful when you want to execute some code when an event happens / is triggered from:
 
- Cloud Storage 
- Cloud Pub/Sub
- HTTP POST/GET/DELETE/PUT/OPTIONS
- Firebase
- Cloud Firestore (NoSQL)
- Stack driver logging

Another key features are:

- Don't have to worry about servers. 
- Pay for what you use (number of invocations, compute time, memory and CPU provisioned)
- Are time bound: timeout min 1 and MAX 60
- 2 Cloud Functions versions for the **environment**.

Another features are: 
- Autoscaling
- One function instance handles ONE request at a time
- Has the typical (serverless architectures) problem of __cold start__ which can be solved by a min number of instances

### Cloud functions V2

It is built on top of Cloud Run and Eventarc, it is also the recommended version to use of Cloud functions. 
- The timeout is up to 60 minutes for HTTP-triggered functions.
- Instance sizes up to 16GiB RAM with 4 vCPU (vs 8GB RAM with 2 vCPU in V1)
- Up to 100 concurrent request per function instance
- Supports multiple active versions/revisions 
- Supports traffic splitting between revisions/versions.
- Supports multiple request AT THE SAME TIME (concurrency) 
- Max 1000 concurrent invocations at the same time

### Commands

```gcloud functions deploy NAME```

Options:

  - ```--docker-registry```
  - ```--docker-repository```
  - ```--gen2```
  - ```--runtime```
  - ```--service-account```
  - ```--timeout```
  - ```--max-instances```
  - ```--min-instances```
  - ```--source``` (ZIP file or source repo URL or Local file)
  - trigger:
    - ```--trigger-bucket```
    - ```--trigger-http```
    - ```--trigger-topic```
    - ```--trigger-event-filters``` (gen2)

## Cloud run 

The idea with cloud run is "container to production in seconds". There is no infrastructure management and is **pay-per-use** and it is build on top of an open standard Knative. Some of other features are: 
- Portable (container based architecture).
- No limitations in languages.
- Integrates Cloud Code, Cloud Build, Cloud Monitoring and Cloud Logging Integrations.
- Supports YAML configuration.
- Supports revisions (redeploying a service) and traffic splitting.

There is another alternative which is **Anthos** to run clusters anywhere cloud or __multicloud__. 

### Commands

To deploy a new container use:

```gcloud run deploy SERVICE_NAME --image IMAGE_URL --revision-suffix VERSION```

You can also list available revisions with:

```gcloud run revisions list```

Adjust traffic assignments with:

```gcloud run services update-traffic SERVICE --to-revisions=V2=10,V1=90```

## Encryption in Google Cloud - Key Management System (KMS)

### Data States 
 
- Data in rest (stored hard disk, database, etc)
- Data in motion (being transferred across a network e.g. app talking to a database)
- Data in use (Data in RAM)

Data it is not stored in raw format so data in databases, hard disk, file servers, etc needs to be encrypted. 

### Types of encryption 

- Symmetric: same key for encryption and decryption
- Asymmetric: a public (encrypt) and a private (decrypt) key, 

### Cloud KMS

- Used to create and manage cryptographic keys (symmetric and asymmetric). 
- Control their use in applications and GCP services (integrates with almost all GCP services)
- Provides an API to encrypt, decrypt or sign data
- Can be used to store secrets on the cloud

## Block and File storage in GCP

There are two types of storage that can be attached to a virtual machine in GCP:

### File Storage

Allows multiple virtual servers attached to a file storage,
- __Filestore__  It allows high performance storage. (needs to be attached via SSH) it works with Google Cloud Engine and Kubernetes. Allows throughput up to 16 GB/s and storage up to 320 TB. 

### Block Storage 

Allows only one virtual server attached to a (writing) block and multiple virtual servers attached to read-only blocks. 

- __Persistent disks__ (somewhere else in the network) 
Very flexible, the performance scales with size, the lifecycle is independent from VM. 
It allows snapshots (__incremental__ because they save changes between them) and configure a snapshot schedule no more often than 1 hour. It is recommended to delete unnecessary snapshots because you don't lose data, you can do it by the snapshot schedule rule. 
  - Zonal: data replicated in one zone
  - Regional: data replicated in multiple zones (high durability) but the cost can be the double of zonal.

- **Local SSD** (on the same host as the virtual machine) 
Provides high IOPS and low latency but data only persist until instance is running. Data is encrypted but only some machine types support this feature and it is physically attached to a VM instance.

### Comparison PD vs Local SSD

| Feature | PD | L SSDs |
| :------- |  :-------| :---- |
| Attachment to VM Instance | Network drive | Physically attached|
| Lifecycle| Independent to VM | Tied to VM|
|I/O speed| Lower due to network| 10-100x faster|
|Snapshots|Supported| Not supported|
|Use case| Permanent storage| Ephemeral storage|

### Comparison PD standard vs PD balanced

| Feature | PD Standard | PD Balanced |L SSDs |
| :------- |  :-------| :---- | :------ |
|Underlying storage| HDD| SSD| SSD|
|Referred to as| pd-standard | pd-balanced| pd-ssd|
|Performance for sequential IOPS| good| good| very good|
|Performance for random IOPS| bad| good| very good|
|Cost|--| -+| ++ |
|Use cases|big data | Balance between cost-performance| High performance|

### Machine Images 
 
Machine types != images

Remember that 
- Images contain is created from boot persistent disk (contains OS)
- Machine images is created from a Machine instances (configuration, metadata, permissions and data from one or more disk)

### Comparison of backups options

| Scenarios | Machine Image | Persistent Disk Snapshot | Custom Image | Instance template |
| :------- |  :-------| :---- | :------ | :------|
|Single disk backup|yes|yes|yes|no|
|Multiple disk backup|yes|no |no|no|
|Differential backup|yes|yes|no|no|
|Instance cloning and replication|yes|no|yes|yes|
|VM instance configuration|yes|no|no|yes|

#### Commands

**Disk**

- ```gcloud compute disks list```

- ```gcloud compute disks create NAME --zone=ZONE```

  - ```--size=SIZE```
  - ```--type=TYPE```
  - ```--image or --image-family or -source-disk or --source-snapshot```
  - ```--kms-key or --kms-project```

To increase disk size: 

- ```gcloud compute disks resize INSTANCE --size=SIZE_UNITS --zone=ZONE```

**Snapshot**

To take snapshot

- ```gcloud compute disks snapshot DISK_NAME --zone=ZONE --snapshot-name=SNAPSHOT_NAME```

- ```gcloud compute disks snapshot list/create/delete```

**Images**

- ```gcloud compute images list/update/import/export/describe/deprecate/create```

- ```gcloud compute images create NAME```
  - ```--source-disk=DISK --source-disk-zone=ZONE```
  - ```--source-snapshot=SOURCE```
  - ```--source-image=IMAGE --source-image-project=PROJECT```
  - ```-source-image-family=FAMILY --source-image-project=PROJECT```

- ```gcloud compute images deprecate IMAGE --state=DEPRECATED```
- ```gcloud compute images export --image=IMAGE --destination-uri=URI --export-format=vmdk --project=PROJECT```
- ```gcloud compute images delete IMAGE1 IMAGE2```

### Global, regional and zonal resources

| Global | Regional | Zonal |
| :------- |  :-------| :------|
| Images, Snapshots and Instance templates (not applicable for zonal resources) | Persistent disk, Regional managed instance groups| Persistent disk, Instances and Zonal managed instance groups|

------- 
## Cloud storage

Most popular, serverless and inexpensive multipurpose storage service. It provides very useful features like:

- Autoscaling and infinite scale
- Treat entire object as a unit
- Also called **Object Storage**
- Provides REST API to access and modify objects
- Provides CLI and Client Libraries for various languages
- Store all file types (text, binary, backup and archives)
- Backups of databases
-Staging data during on-premise to cloud database migration

### Buckets

Objects are stored as buckets. They need a globally unique name and it can't start with __goog__ prefix or contain __google__. 
It can be multi-region, dual-region or single region. Some other features are:

- Each bucket is associated with a project
- Each object has an unique ID
- Max object size in 5 TB
- There is no limit for the number of objects
- To __create a static website__ using buckets follow:
  - Create a bucket with the same name as a website name (you have to own the domain)
  - Copy files to the bucket (including index and error files)
  - Add member **allUsers** and grant the Storage Object Viewer option

### Storage Classes

Are made to optimize the cost based on your access needs. It is designed for durability of 11 9's (99.9999999%) and all of them provides low-latency. Here are the classes and their typical availability in multi-region / dual-region and single-region.

- Standard (99.95% / 99.99%): Frequently used data in a short period of time.
- Nearline Storage (99.9% / 99.0%): Minimum storage duration of 30 days and used for read or modify once a month on average.
- Coldline storage (99.9% / 99.0%): Minimum storage duration of 90 days and used for read or modify
at most once a quarter
- Archive storage (99.9% / 99.0%): Used less than once a year

### Types of uploading and downloading

**Download**

| Scenario | Option|
| :------- | :------|
| Small files that can be reuploaded in case of failures and **no** object metadata| Simple upload|
| Small files that can be reuploaded in case of failures and object metadata|Multipart Upload|
|Larger files and RECOMMENDED for most use cases | Resumable upload|
|Objects of unknown size| Stream transfers|
|Fast uploads when network and disk speed are not a limitation (file divided in 32 chunks uploaded in parallel) | Parallel composite uploads|


**Upload**

| Scenario | Option|
| :------- | :------|
|Downloading objects to a destination| Simple download|
|Downloading to a process|Streaming Download| 
|Slice and download large objects| Sliced object download|

### Object versioning

It is used to prevent accidental deletion and provides history.

- Enabled at bucket level
- Can be turned off/on anytime
- Live version is the latest version
- Older versions are identified by object key (live version) + generation number (older version)
- To keep cost low you can delete older versions of objects, for example older than 15 days

### Object Lifecycle Management

When you identify the conditions for using objects based on:
- Age 
- IsLive 
- CreatedBefore
- NumberOfNewerVersions
- Combinations of conditions

There are two kinds of actions
- SetStorageClass (change from one class to another)
- Deletion (delete objects)

____

Restrictions (more usage to less):
- [Standard, Multi-regional or Regional] to [Nearline or Coldline or Archive]
- [Nearline] to [Coldline or Archive]
- [Coldline] to [Archive]

______

### Cloud Storage - Encryption

- Data is encrypted on the server side. 
- You can configure to use the default (Google Managed encryption key) or change it to use (Customer Managed encryption key using KMS)
- Optionally you can encrypt on the client side 

__Tip and trick__. To change storage class of and existing bucket you have to:
- Change the default class of bucket
- Update the class of the objects in the bucket

### Commands (gsutil)

- Make bucket (mb) ```gsutil mb gs://BKT_NAME```
- List everything inside a bucket ```gsutil ls -a gs://BKT_NAME```
- Copy buckets ```gsutil cp gs://SRC_BKT/SRC_OBJ gs://DESTN_BKT/NAME_COPY```
  - Upload  ```gsutil cp LOCAL_LOCATION gs://DESTN_BKT_NAME```
  - Download ```gsutil cp  gs://BKT_NAME/OBJ_PATH LOCAL_LOCATION```
- Move or rename objects of bucket ```gsutil mv gs://BKT_NAME/OLD_OBJ_NAME gs://BKT_NAME/NEW_OBJ_NAME```
- Rewrite / change storage class for objects ```gsutil rewrite -s STORAGE_CLASS gs://BKT_NAME/OBJ_PATH gs://BKT_NAME/```
- Versioning  ```gsutil versioning set on/off gs:BKT_NAME```
- Set access permissions ```gsutil acl ch```
- Set IAM access / role  ```gsutil iam ch MBR_TYPE:MBR_NAME:IAM_ROLE gs://BKT_NAME```
- Signed URL for temporary access   to somebody (10 minutes) ```gsutil signurl -d 10m YOUR_KEY gs:/BUCKET_NAME/OBJECT_PATH```


## Google IAM 

Provides the service for **authentication** (is the right user?) and **authorization** (is the right access?) for the identities to access the resources. 

An Identity can be: 
- A GCP user 
- A group of GCP users
- An application running in GCP
- An application running in your data center
- Unauthenticated users (can be external users)

Some concepts:
- Member: The one who needs the permissions, is also called principal or identity. It can be a:
  - User (represents a person with an email address)
  - Service Account (represents an application account)
  - Group (has a unique email address)
  - Domain
    - Google workspace domain
    - Cloud identity domain 
- Roles: set of permissions to perform a specific actions on specific resources. __The roles don't know about members like they do in AWS__. Some types of roles are: 
  - **Basic** roles (viewer, editor[viewer + edit actions], owner [editor + manage roles and permissions + billing]) **not recommended for production** (See principle of least privilege)
  - **Predefined** roles (storage admin, storage object admin, storage object viewer, storage object creator)
  - **Custom** roles: are used when predefined roles are not sufficient
- Policy: Allows to bind a or more roles to members

** In Google IAM there is a section called "Policy Troubleshooter" to check/troubleshoot the resources and permissions assigned.

### Commands

- Describe the current project ```gcloud compute project-info describe```
- Access the Cloud Platform with Google user credentials ```gcloud auth login```
- Revoke access credentials for an account ```gcloud auth revoke```
- List active accounts ```gcloud auth list```
- To get the current project's policy ```gcloud projects get-iam-policy```
- Add binding  ```gcloud projects add-iam-policy-binding PROJECT --member=user:USER --role=roles/ROLE```
- Remove binding  ```gcloud projects remove-iam-policy-binding PROJECT --member=user:USER --role=roles/ROLE```
- Set binding  ```gcloud projects set-iam-policy-binding PROJECT```
- Delete a project with  ```gcloud projects delete PROJECT```
- Describe an IAM role ```gcloud iam roles describe roles/ROLE```
- Create your own roles ```gcloud iam roles create --project PROJECT --permissions PERMISSIONS --stage STAGE```
- Copy roles ```gcloud iam roles copy --source=roles/ROLE --destination=my.custom.role --dest-project=PROJECT```
- Create a Service Account User Managed Key ```gcloud iam service-accounts keys create```

### Service Account

Is an identified email address. It is both an Identity and a Resource. Its also recommended to have one service account per project. They are very useful when you don't want to use personal credentials to allow access. They can be either: 
- Default service accounts (Automatically created when some services are used) NOT RECOMMENDED
- User managed (fine grained access control and conditions) **RECOMMENDED**
- Google-managed service accounts (Created and managed by Google) They are used by GCP to perform operations on user's behalf, we don't generally worry about them.

Beware that the process of providing access is different if the application is on premise or if is hosted in GCP.

### Access Control List (ACL)

Defines who can access to a bucket and objects and the level of the access. It is different from IAM because ACL is more granular and instead of granting access to all objects in a bucket, it can customize the specific access to different objects. It can be: 
- Uniform (using IAM) RECOMMENDED
- Fine grained (using IAM + ACL)

### Signed URL 

Useful to allow a user **limited time** access, users don't require Google accounts. The steps to create a signed URL are: 
- Create a key for the service account/ user with the desired permissions. 
- Create a signed URL with the key and gsutil ```gsutil signurl -d 10m YOUR_KEY gs://BUCKET_NAME/OBJECT_PATH```

## Databases 

### Technical measures 

**Availability** [downtime of an app]: commonly 9 nines (typically online apps aim this availability). 
- A common way to improve availability in databases is by using standby databases (synchronous replication) in multiple zones or multiple regions. 
 
**Durability** [percentage of data losed]: commonly 11 nines, it is really high because once data is lost, there is no way to get it back. It is commonly more important that availability. 

- A common way to improve durability is storing multiple copies of data, but this bring some challenges like:

  - Consistency 
    - Strong consistency -> slow
    - Eventual consistency -> lag of seconds between replicas
    - Read-after-write consistency -> Inserts are immediate but updates have eventual consistency

**Recovery Point Objective (RPO)**
Maximum acceptable period of data loss

**Recovery Time Objective (RTO)**
Maximum acceptable downtime

Some ****scenarios**** related to data loss and downtime are: 

| Scenario | Solution |
| :------- | :------|
|Very small data loss ( RPO -1 min) and Very small downtime (RTO -5 minutes)| Hot standby| 
|Very small data loss ( RPO -1 min) and admissible to downtime (RTO -15 minutes) | Warm standby|
| Data is critical ( RPO -1 min) and downtime admissible in range of hours (RTO - few hours)| Snapshots and transaction logs|
|Data can be lost| Failover a new server| 

Increase database performance

| Scenario | Solution |
| :------- | :------|
|Database performance is impacted by two applications reading data from one database| Vertically scale the database (increase CPU and memory)|
|Database performance is impacted by two applications reading data from one database|Create a database cluster (Distribute the database) -> expensive setup|
|Database performance is impacted by two applications reading data from one database|Create read replicas (run read only applications against read replicas)|

### Choosing a database 
Some of the factors to choose a database are: 
- Fixed schema or schemaless?
- Transaction properties that you need (atomicity and consistency)
- Latency required (seconds, milliseconds or microseconds)
- Number of transactions per second expected (hundreds of thousands or millions)
- Data that will be stored (Mbs or GBs or TBs or PBs)
- etc 

### Databases categories

- **Relational** (most popular) It has tables and relations between them. Used for: 
  - __OLTP__ (row storage): applications where large number of users make large number of small transactions. MySQL, Oracle, SQL server, etc. In __Google Managed services__ Cloud SQL supports PostgreSQL, MySQL and SQL server for regional relational databases. Cloud Spanner is intended for global applications with horizontal scaling.
    - Cloud SQL
    - Cloud Spanner
  - __OLAP__ (columnar storage - high compression): applications that allows users to analyze petabytes of data like Reporting applications, data ware houses, business intelligence applications and Analytics systems. The __GCP managed service__ recommended is BigQuery. 
    - BigQuery
- **NoSQL** (scalability and high performance) Flexible schema, structure data the way your application needs it. The schema can evolve with time. The __GCP managed service__ is Cloud Firestore and Cloud BigTable. 
  - Cloud Firestore/Datastore: **Serverless**. Is the next version of Datastore, __designed for transactional__ mobile and web applications, and it has strong consistency and mobile and Web client libraries, recommended for a maximum of few Terabytes 
  - Cloud BigTable: **Not serverless** needs instances, wide column database, recommended for streaming (IOT) data size > 10 Terabytes to several Petabytes. __Not recommended for transactional__ workloads.

- **In Memory databases** Faster databases than disk databases, microseconds response. The __GCP Managed Service__ recommended is Memory Store. 
  - Memory Store

- Document
- Key value
- Graph

### Some scenarios 

| Scenario | Solution |
| :------- | :------|
|A startup with quick evolving schema (table structure)| NoSQL-Firestore/Datastore |
|Non relational db with less storage(10 GB)| NoSQL-Firestore/Datastore|
|Transactional global database with predefined schema needing to process million of transactions per second| Global/Relational/OLTP-CloudSpanner|
|Transactional local database processing thousands of transactions per second|Local/OLTP-CloudSQL|
|Cache data (from database) for a web application| Cache-Memory Store|
|Database for analytics processing of petabytes of data| Analytics/OLAP-BigQuery|
|Database for storing huge volumes stream data from IOT devices | Stream/NoSQL-Cloud Big Table|
|Database for storing huge streams of time series (time stamp) data | Stream-Cloud Big Table|

### Cloud SQL 

You can import/export data from:
- Console/gcloud/REST API
- Large databases use serverless mode

Configure your needs and do not worry about managing the database. Used for simple relational use cases. It it cheaper than Cloud Spanner. It supports: 
- MySQL, PostgreSQL and SQL Server
- Regional (multiple zones) service with high availability (99.95%)
- HDDs and SSDs
- Automatic encryption (tables and backups), maintenance and updates
- Automated backups 
- High availability and failover 
- Up to 416 GB RAM and 30 TB data storage
- Allows you to create read replicas in the same region or different region or external (non Cloud SQL db)
- Automatic storage increase without downtime
- Schedule automatic backups
- Migration and export of data
- You first need to create an instance in order to use the service

#### Cloud SQL high availability

You have to choose a primary and secondary zones, then, you need to create two instances, the changes from the primary will be replicated on the secondary instance. One restriction is that you can not connect simultaneously to both instances, secondary instances is intended to work when primary fails. (failover)

### Cloud Spanner
You can import and export data from:
- Console

It is a high cost (pay per nodes and storage) alternative to Cloud SQL that allows:
- Fully managed mission critical
- Strong transactional consistency at global scale
- Automatically scale to PBs of data
- Scale horizontally (in reads and writes)
- High availability (99.999%)
- Working with huge volumes of relational data 
- Infinite scaling for a growing application 
- Global database
- Data export through Cloud Console and Data flow (no gcloud option)
- You first need to create an instance in order to use the service

### Cloud Datastore and Firestore
You can import/export data from:
- Console
- Rest API
- gcloud

#### Datastore

Highly scalable NoSQL Document database that automatically scales and partitions data as it grows, its intended to work for use cases needing a __flexible schema with transactions__. It is recommended for up to a few TBs of data, supports: 
- Transactions 
- Indexes
- SQL like queries (GQL language)

But it doesn't support joins and you can only export data only from gcloud (not from cloud console ).

### Firestore

It is an enhanced version that will replace Datastore is **Firestore**. It is optimized for __multiple devices access__ (mobile, IOT, etc). Also it offers client side libraries Web, iOS, Android and more. It stores data in hierarchical structure. It offers two modes: 
- Datastore compatible
- Native modes

It can export data to buckets and also import data from buckets

### Cloud BigTable

Import and export information from:
- Create Dataflows

Petabyte scale with millisecond response, wide column noSQL database. It is HBase API compatible (open source alternative). Designed for huge volumes of analytical and operational data and real-time analytics such as time-series data, financial data, transaction histories or stock prices. Some of the important features that provides: 
- Millions of read/writes TPS at very low-latency
- Single row transactions (does not support multi row transactions)
- Needs server instance with SSD or HHD (not serverless)
- Scale horizontally with multiple nodes and zero downtime
- Export data using Java application or HBase commands (cannot use gcloud or console)
- The CLI for using BigTable is **cbt** (not gcloud)
- Compatible with Dataflow for exporting data

### Memory store 

In-memory datastore service to reduce access times. It is fully managed with hight availability or 99.9%. Supports two options
- Redis: low latency with persistence
- Memcached: pure caching

### BigQuery

In the context of tons of data, import and export becomes very important, so you can use:
- Console / bq
- Cloud Storage
- Batch loading with BigQuery Data Transfer Service
- Dataflow (streaming pipeline)

Exabyte scale modern Datawarehousing solution from GCP. It provides Relational database (SQL-case sensitive, schema, consistency). Data in organized in **datasets**, inside datasets there are multiple tables. It offers: 
- Two approaches: __traditional__ (storage + compute) and __modern__ (realtime + serverless) 
- Importing (lost of sources: CSV, JSON, ARO, PAKE, ORC, 
Datastore backups, etc) and exporting (Cloud Storage and Data Studio, formats: CSV/JSON with Gzip compression, Avro with deflate or snappy compression)
- Can configure Table Expiration 
- Can query info from Cloud Storage, Cloud SQL, BigTable, Google Drive without storing it in BigQuery
- Access to BigQuery from: 
  - Cloud console
  - bq command-line tool (no gcloud)
  - BigQuery REST API
  - HBase API based libraries (Java, .NET and Python)
- A good practice is to estimate queries before running, either console or ```bq --dry-run``` (you pay for the amount of data scanned by the query)

### Commands

- Cloud SQL
  - ```gcloud sql instances create/clone/delete/describe/patch```
  - ```gcloud sql databases create/clone/delete/describe/patch```
  - ```gcloud sql connect INSTANCE --database=DATABASE --user=root```
  - ```gcloud sql backups create/describe/list --async --instance INSTANCE```
  - ```gcloud sql import```

- BigQuery
  -  ```bq show biggquery-public-data:sample.shakespeare```
  - ```bq query 'QUERY-STRING'```
  - ```bq extract```
  - ```bq load``` to import data
  - ````gcloud config set project PROJECT``` for queries inside an specific project

- Cloud BigTable
  - ```gcloud components install cbt```
  - ```cbt listinstances```
  - Create .cbtrc file with the configuration 
    - ```echo instance=quickstart-instance >> ~/.cbtrc``` 
    - ```echo project=PROJECT >> ~/.cbtrc```
  - ```cbt createinstance```
  - ```cbt createcluster```
  - ```cbt createtable```
  - ```cbt deleteinstance```
  - ```cbt ls (list tables and column families)```

### Important detail about import and export of databases

Whenever you import or export databases, and you are using Cloud Storage Buckets, you need to give the access to the bucket:
- Using ACL with the ```gsutil``` command
- Giving the role 

## Asynchronous Communication

It is a form of communication that implies putting a Topic between web server (publisher) and Logging service (subscriber). It is necessary because provides the following improvements over the traditional Logging service model such as: 
- Availability -> Even if Logging service is down, the publisher remains up
- Scalability -> You can process more logs scaling the topic
- Decoupling -> Apps (publishers) doesn't care who listen their logs.
- Durability -> messages are not lost even if the Logging service (subscriber) is down 

### Pub/Sub

The __GCP managed service__ for asynchronous messaging service, is **Pub/Sub**, the main features are:
- Auto scale
- Pay for use (low cost)

The main use cases are: 
- Event ingestion and delivery for streaming analytics pipelines.

Supports push and pull message deliveries. The main parts of Pub/Sub are:

- Publisher: Who sends a message by HTTP requests
- Subscriber: Who receives a message (and send an acknowledge). Messages are sent to subscribers, if there are multiple subscribers, each one process messages __individually__. The message can be received through: 
  - Pulls a message whenever it is ready by a HTTP request
  - Push is the process when messages are sent to subscribers

The relationship between pb and sb can be:
- one to many 
- many to one
- many to many

The process of sending or receiving a message is: 
- Create a topic
- Create subscriptions 

Remember that subscriptions supports snapshots.

### Pub/Sub Lite

It is a zonal messaging service optimized for low cost. It uses zonal storage. 

### Commands

- Create a topic ```gcloud pubsub topics create TOPIC_NAME```
- Create a subscription ```gcloud pubsub subscriptions create SUBSCRIPTION --topic=TOPIC```
  - ```--enable-message-ordering```
  - ```--ack-deadline```
  - ```--message-filter```
- Pull messages ```gcloud pubsub subscriptions pull SUBSCRIPTION```
  - To acknowledge the message you can ```--auto-ack```
  - Specify the number of messages with ```--limit```
- Publish a message  ```gcloud pubsub topics publish topic-from-gcloud --message="MESSAGE"```
- List/delete topics with ```gcloud pubsub topics list/delete```
- Delete topics with ```gcloud pubsub topics delete TOPIC```
- List subscriptions with  ```gcloud pubsub topics list-subscriptions TOPIC```

## Google Cloud VPC

VPC is a service to help separate public resources (accessible from internet) from private resources (a database for example) inside GCP.

Since your data and applications are outside a corporate network, surges the need for a private network/cloud isolated from the outside, so your network traffic is not visible from all other Cloud VPC. Unlike AWS (regional), Google VPC is a **global** resource. 

Besides, you can create subnets, which are a **regional** resource. In general you want to create different VPC subnets for different access types resources, also you can connect subnets with VPC network peering.

Until now, the VPC used in the above sections was the default, but you can create your own VPC with the following options: 

- Auto mode (also used in default VPC): subnets automatically created in each region
- Custom mode: Complete control over subnet and IP ranges. Its RECOMMENDED for production. The following options have to be enabled:
  - Private Google Access -> Communication through private IP
  - FlowLogs -> To troubleshoot network issues

### CIDR (Classless )

The continuous resources in a network makes routing easy. CIDR blocks express a range of addresses, consist of a starting IP address and a range which indicates the number of fixed bits in that address. Each address consist in (IPv4) 4 groups of 8 bits (32 bits), so 69.208.0.0/28 means that only 32 - 28 = 4 bits are free which means 2^4=16 possible combinations or addresses and 69.208.0.0 means the initial IP address, so the range expressed is from 69.208.0.0 to 69.208.0.15 
Remember: 
- 0.0.0.0/0 Represents all the range of IPv4 addresses
- 0.0.0.0/32 Represents one address. 

### Firewall rules

Useful to control the traffic going in or out of the network. Eac of the rules have a priority, represented with a number between 0-65535, where 0 is the highest priority.

The default rule with the lowest priority (65535) allows all egress and denies all ingress of traffic, this rule cannot be deleted but you can override it with the remaining rules. 

The default VPC has 4 rules with priority of 65534: 
- Allow incoming traffic from VM instances in same network
- Allow incoming TCP traffic on port 22 (SSH)
- Allow incoming TCP traffic on port 3389 (RDP-remote desktop protocol [Windows])
- Allow incoming ICMP traffic (error and control messages for IP) from any source on the network

Two types of rules: 
- Ingress: __Target__ (Instances with TAG or specific Service account) and __Source__ (CIDR)
- Egress: __Target__ (Instances with TAG or specific Service account) and __Destination__ (CIDR Block)

### Shared VPC 

When you want to communicate resources in **different projects**, Shared VPC is a solution. There is a host project and service projects.

### VPC Peering 

Useful to connect VPC networks of different organizations, different projects in same organization and same project in same organization.
The communication happens inside Google network (internal IP address) and is not accessible from internet. 


## Hybrid Cloud

### Cloud VPN

Is a service **to connect on-premise network** to the GCP network (hybrid cloud). The traffic is encrypted using Internet Key Exchange protocol, the VPN is implemented using IPSec VPN Tunnel. The two different types of Cloud VPN solutions are:
- HA VPN (SLA of 99.99% service availability with two external IP addresses) support only dynamic routing
- Classic VPN (SLA of 99.99% with a single external IP address) Support static and dynamic routing.

### Cloud Interconnect

Is another alternative to connect on-premise network with GCP network (hybrid cloud), it provides a high speed physical connection between on-premise and VPC networks, the data exchange happens through a private network. The main features are: 
- Highly available and high throughput
- Connections:
  - Dedicated Interconnect: 10 or 100 Gbps 
  - Partner Interconnect: 50 Mbps to 10 Gbps.
- Recommended only for high bandwidth needs.

### Direct peering

Is another alternative to connect on-premise network with GCP network (hybrid cloud) 

Is not a GCP service, is lower lever connection who establish a direct path from on-premises network to Google services (network peering). 
It is not typically recommended. 

## Cloud Operations

### Cloud monitoring

Its a set of tools that works for GCP projects and AWS projects, in order to group all the information from multiple GCP projects you need to create a workspace on the host project and then add other GCP or AWS projects to the workspace.

What you should know: 
- Are users having any issue?
- Does db has enough space?
- Are servers running in optimal capacity?

Tools that provide: 
- Measures key aspects of services (metrics). The default metrics are:
  - CPU utilization
  - Disk traffic metrics
  - Network traffic
  - Uptime information
  - For more metrics you can install Cloud monitoring agent on a VM
- Create visualizations (Graphs and Dashboard)
- Configure alerts when metrics aren't healthy

### Cloud Logging

 Log management and analysis tool. Allows to store search and analyze ans alert on massive volume of data. Some of the key features are: 
 - Logs Explorer 
 - Logs Dashboard 
 - Logs Metrics 
 - Logs Router

Is worth mentioning that most of GCP managed services automatically send logs to Cloud Logging. 

In order to get logs from Google Compute Engine VMs, you have to install **Logging agent** (based on fluentd).

If you want logs from on-premises, you can use BindPlane tool from Blue Medora or the cloud Logging API.

#### Audit and Security Logs

The related logs are: 
- **Access Transparency Logs** (needs gold support or above): Captures Actions performed by GCP team on your cloud resources. 
- **Clod Audit Logs**: Represents who did a change when and where. (which service, which operation, what resource and who is making the call). <br>
 The different Audit logs (default enabled?):

  - __Admin activity logs__ (yes) 
    - -> API calls or other actions that modify configuration of resources. 
    - -> VM Creation, Patching resources, Change IAM permissions, etc. 
  - __Data access logs__ (no) 
    - -> Reading configuration of resources. 
    - -> Listing resources. 
  - __System Event Audit Logs__ (yes) 
    - -> Google Cloud administrative actions. 
    - -> On host maintenance, Instance preemption, Automatic restart. 
  - __Policy Denied Audit Logs__ (yes) 
    - -> When user or service account access is denied.
    - -> Security policy violation logs. 
 
 #### Log Router 

 Is a tool that manages the logs from Cloud Logging API and decides what to ingest, what to discard and where to route. It sends logs to two types of logs buckets: 
 - **_Required**: Holds Admin activity, system events and access transparency logs. Is retained for __400 days and cannot delete logs before it or modify retention period__
 - **_Default**: All other logs. Logs are retained for __30 days, logs can be disabled and retention period can be edited up to 10 years__.

 Old logs can be exported to:
 - Clod storage bucket
 - Big Query dataset 
 - Cloud Pub/Sub topic

 Some export use cases are: 

 - Troubleshooting using VM Logs: Install cloud logging agent in all VM
 - Export Logs to BigQuery for querying with SQL queries: Install cloud logging agent in VMs, create a BigQuery dataset and export **sink** in Cloud logging with BQ as destination.
 - Retain logs for external auditors: Create a export **sink** in Cloud logging with a bucket as a destination, provide auditors with Storage Object Viewer role and you can see it with Data Studio.

### Cloud Trace

In microservices architecture tracing is a method of __monitoring__ and observing service requests in applications that are based on microservice architecture. 

You can collect latency data from:
- Supported Google cloud services
- Instrumented applications using Cloud Trace API. 

Its supported by: 
- Compute Engine
- GKE
- App Engine (flexible and standard)

Libraries available for: 
- C#
- Go 
- Java
- NOde.js 
- PHP
- Python 
- Ruby

### Cloud debugger

Used to debug in  production or test environments. It captures **state of a running application**. This tool no need to redeploy or add logging statements. Also it is very lightweight, so it doesn't impact users experience.

### Cloud Profiler

Used to identify performance bottlenecks in production. Some of its main features: 
- Statistical 
- Low overhead profiler
- Allows connect profiling data with application source code. 

It consist of two major components:
- Profiling agent 
- Profiler interface

### Error Reporting

Supported to various languages and helps to **identify production problems in real time** via a centralized error management console. In the case of Android and iOS client you can use Firebase Crash Reporting. 

### Some scenarios

| Scenario | Solution |
| :------- | :------|
|Record all operations/requests on all objects in a bucket| Turn on data access audit logging for the bucket |
|Trace a request across multiple microservices| Cloud trace|
|Identify prominent exceptions or  errors for a specific microservice| Error reporting|
|Debug a problem in production step by step| Cloud debugger|
|Look logs for a specific request | Cloud logging|

## Organizing GCP resources

### Hierarchy

The general  hierarchy is: 
Organization > Folder** > Project > Resources

** Folder is not available in trial accounts. 

Some recommendations for enterprises:
- Create different folders for each department. (Isolation between apps of different departments)
- Create separate projects for different environments. (Isolation between DEV and PROD)
- One project per application per environment 

### Billing account

Projects with active resources should be associated with a Billing Account. 
__One Billing Account can be associated with one or more projects__. A startup can have only one billing account, but an enterprise can have multiple billing accounts per department. 
There are two types of Billing accounts: 
- Self serve (automatic payment with credit card or bank account)
- Invoiced (Generates invoices for larger enterprises)

You can set a **budget** to avoid surprises and configure alerts with notifications to:
- Pub/Sub
- Email  

Also, you can **export billing data** to: 
- Big Query (to query information)
- Cloud storage (history)

## IAM 

Focuses on who and gives their accesses. 
Some principles/recommendations: 
- Give the least possible privilege needed for a role (**Least privilege**)
- At least 2 people in sensitive tasks. (**Separation of duties**)
- Review Cloud Audit Logs to audit changes to IAM policies and access to Service accounts Keys (**Constant Monitoring**)
- Simplify roles management (**Use groups**)

### User Identity Management

For enterprises is not recommended to add user  with individual Gmail accounts. There are two solutions that can work:
- Use Google workspace
- Use and bind Google Cloud with the identity provider of the enterprise.

### Corporate Directory  Federation

Federate Cloud Identity or Google workspace with your external identity provider (IdP)  such as Active Directory or Azure Active Directory.

** __Federation__ is a process where one system is responsible for the authentication of a user. That system then sends a message to a second system, announcing who the user is, and verifying that they were properly authenticated. 

### Organization Policy

Focuses on what can be done on specific resources. 
Centralized constrains on all resources created in an Organization, it needs a role of **Organization Policy Administrator**. For example:
- Disable creation of Service Accounts
- Allow/Deny creation of resources in specific regions. 

### IAM policy 

Can be set at any level of the hierarchy. The resources inherit the policies of All parents. The policy in one specific level is the union of the policy of that resource and its parents. 
 
### Roles 

#### Organization, Billing and Project

- Organization
  - Organization Administrator
- Billing 
  - Billing account creator (finance team)
  - Billing account administrator (finance team)
  - Billing account user (project owner)
  - Billing account viewer (auditor)
- Project
  - Project Creator 

#### Compute Engine

  - Compute Engine Admin (Everything in compute)
  - Compute Instance Admin (Everything in instances)
  - Compute Engine Network Admin
  - Compute Engine Security Admin (firewall rules and SSL certificates)
  - Compute Storage Admin
  - Compute Engine Viewer (read only in everything in Compute Engine)
  - Compute OS Admin Login 
  - Compute OS Login 

#### App Engine

CRUD (Create, Read, Update, Delete)
- App Engine Creator (CD)
- App Engine Admin (application -> RU, services/instances/versions -> CRUD )
- App Engine Viewer (apps/services/instances/versions -> read)
- App Engine Code Viewer (unique role that can view code)
- App Engine Deployer (versions -> CRD, app/s/v -> R)
- App Engine Service Admin ( versions -> RUD, a -> R, s/i -> CRUD)

Remember that none App Engine Role allow to:
- View and download applications logs
- View monitoring charts in the cloud console
- Enable/disable billing
- Access configuration or data stored in other services

#### Kubernetes Engine

- Kubernetes Engine Admin: Access to Clusters and Kubernetes API objects
- Kubernetes Engine Cluster Admin
- Kubernetes Engine Developer
- Kubernetes Engine Viewer read (get/list) clusters and kubernetes API objects

#### Cloud Storage 

- Storage Admin (buckets and objects)
- Storage Object Admin (objects only)
- Storage Object Creator
- Storage Object Viewer

#### Big Query 

- BigQuery Admin
- BigQuery Data Owner (doesn't have access to Jobs)
- BigQuery Data Editor (edit data, also doesn't has access to jobs)
- BigQuery Data Viewer (view data, also doesn't has access to jobs)
- BigQuery Job User (Create jobs, run queries)
- BigQuery User (Data Viewer + read Jobs )

#### Logging
- Logs viewer (read all logs except access transparency and data access audit logs)
- Private Logs Viewer (logs viewer + transparency and data access audit logs)
- Logging Admin (all permissions related to logging)

#### Service accounts

- serviceAccountAdmin (Create and manage service accounts)
- serviceAccountUser (Run operations)
- serviceAccountTokenCreator (Impersonate service accounts)
- serviceAccountKeyAdmin (Create and manage and rotate service account keys)

#### IAM Roles 

- Security 
  - securityAdmin
  - securityReviewer
- Organization
  - organizationRoleAdmin
  - organizationRoleViewer
- Project
  - roleAdmin
  - roleViewer
  - browser (cannot see resources in the project)

  ### Authentication in Compute Engine VM

  - Linux (SSH)
    - Metadata managed -> manually create
    - OS login -> Link Linux user account to Google identity. Has advantages importing Linux accounts 
  - Windows 
    - passwords

  There are different ways to login via SSH: 
  - Console SSH Button: temporary keys
  - Gcloud command ```gcloud compute ssh``` (persistent keys, reused in the future)
  - Customized keys 
    - Meta managed (centralized, all keys in one place)
    - OS login (each one maintain its key, so easy)

### Some scenarios

| Scenario | Solution |
| :------- | :------|
|Permanent access to sub set of objects in a bucket| ACL|
|Permanent access to a bucket| IAM|
|Limited access to specific object in a bucket| signed URL|
|Access to a set of resources in a team | Create a group and add users|
|Role to upload objects to cloud storage|Storage Object creator|
|Role to manage kubernetes API objects|Kubernetes Engine Developer|
|Role to manage service accounts| Service 
Account Admin|
|Role to view data in BigQuery| BigQuery Data Viewer|
  
## Pricing Calculator

Used to estimate the cost of a GCP solution. Works in more than 40 services including Compute Engine, Kubernetes Engine, Cloud Run, App Engine, Cloud Storage, etc.

## Cloud deployment manager

Tool created to automate the process of infrastructure deployment and modification of resources in a controlled and predictable way with a script (YAML file). The advantages of using it are: 
- Avoid configuration drift: Updating one environment without updating the others. 
- Avoid manual mistakes.
- Free
- Allows creating templates with Python or JinJa2
- Automatic rollback on errors
- Track changes with a single script.

The steps to deploy infrastructure with Cloud deployment manager are:
- Create VM
- Create subnets
- Create database

** The changes in resources have to be done using  environment manager. 

## Cloud Marketplace

Its a useful tool to install software/stack that involves multiple resources of GCP, it is similar to the App Store/Play Store for mobile applications. Some examples are: 
- WordPress 
- Elasticsearch
- Jenkins
- Apache Kafka
- LAMP

## Cloud DNS

It is a Global Domain Name System, it helps you setting up a DNS routing for your website (setting up a website with a domain name). So you can manage the mapping from a name to an IP address. The process to follow is: 

- Create a zone (public or private)
- Create a  record set

Equivalent commands are: 

- Create a zone ```gcloud dns manages-zones create ZONE_NAME``` with the following options: 
  - ```--description``` REQUIRED
  - ```--dns-name``` REQUIRED
  - ```--visibility``` (public/private)
  - ```--networks```

- Create a  record set (3 steps involved)
  - Start transaction zone ```gcloud dns record-sets transaction start --zone```
  - Make changes ```gcloud dns record-sets transaction add --name=RECORD_NAME --ttl --type --A/CNAME --zone```
  - End transaction of zone

## Cloud Dataflow 

It a serverless and autoscaling tool, based on Apache Beam  used for Streaming and batch use cases like:
- Streaming
  - Pub/sub > Dataflow > BigQuery 
  - Pub/sub > Dataflow > Cloud Storage
- Batch
  - Cloud Storage > Dataflow > Big Table/Cloud Spanner/Datastore/BigQuery 
  - Convert file formats between Avro, Parquet and CSV

Offers pre-built templates and support for Java, Python, Go, etc. 

## Cloud Dataproc 

Is a data analysis platform, so you can export cluster configuration but not data. An alternative could be BigQuery (run SQL queries), but Cloud Dataproc is needed when you need complex batch processing for ML or AI workloads. It is useful to **move Hadoop and Spark clusters** to the cloud.

Supports the following cluster modes: 
- Single node
- Standard
- High availability
