# GCP Key features

## Zones

GCP have 20+ zones around the world.
Each zone have al least 3 zones connected with low-l tency links. Each zone has one or more discrete clusters.
- Region and zone utilization is based on latency, HW requirements, availability, regulations and cost.
- Instances runs in a Region, but images and templates (unless uses zonal machines) are global. 

## Virtual Machines 

Google Compute Engine is a SERVICE to manage and create lifecycle of VM.
- You cannot  change a instance template, you can copy it and modify the copy.
- Custom images can be used to create vm-instances with OS patches and/or software preinstalled.
- You can share an image across projects.
- **Hardening** an image is the proccess of creating an image from a vm-instance that meets the security requirements of an enterprise.
- It is preferred to use a custom image rather than startup scripts.
- For vCPUs/ memory changes, a VM instance has to be stopped.
- A good practice is use labels to organize VMs.
- Managment:
  - Default metrics: CPU, Network and Disk Throughput.
  - Cloud agent: Memory Utilization and Disk space.


## Cost 

In GCP you are billed per second.  For money saving, always choose the right machine type for your needs and image for your workload. Whenever you have a predictable resources need, you can commit the use of a VM (Kubernetes or  Compute engine) for 1 or 3 years and get up to 70% of discount.
- **Preemptible VM** only last **24 hours**  or less due to system demands and they are 80% more cheaper but have a lot of disadvantages, e.g. not always available.
- Also, Preemptible VM doesn't work with free tier credits.
- The updated version of Preemptible VM is **Spot VM** and the key difference is that they **don't  have a maximum runtime**.
- Spot VM has a Dynamic pricing wich represents 60-90% of discount to on-demand VM. 
- When you apart an static ip address you'll be billed, even if you aren't using it!
- The storage attached to a compute instance generate a bill. even if the compute instance is stopped. 
- Solutions to those problems are: Remove any static ip address that is not used and create budget alerts.
- A budget can send you email alerts about the limits of your budget.

## Live migration 

Whenever you need a system update (HW or SW) you can use Live migration. This means that your running instance is migrated to another host on the same zone. Also, this proccess doesn't change any attributes or properties of the VM.
- Preemptible instances doesn't support this feature.
- The way to configure the Live Ligration is through Availability Policy section.

## GPU

- You can customize your VM with a GPU or use the predifined family of VM with GPU.
- You can add GPU to your machine but you have to use images with GPU libraries and supported machine types (shared core and memory-optimized machines does not support this feature).
- Adding GPU imply that VM won't support Live migration.
- GPU carry higher cost, but is useful for High performance, math and graphics intensive.

##  GCloud CLI 

- Gcloud is part of Google Cloud SDK (requires Python) you can install it on your computer.
- Cloud Shell is a online shell, it has installed Gcloud.
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

### Instances

  - ```glcoud compute instances create NAME ```
  - ```gcloud compute instances describe NAME ```
  - ```gcloud compute instances delete NAME ```
  - ```gcloud compute machine-types list ```
  - ```gcloud compute machine-types list --filter "zone:NAME"``` 
  - ```gcloud compute zones list --filter "region:(us-west2 us-west1)"```
  - ```gcloud compute zones list --filter region:us-west1 --sort-by ~name (~ means reverse order) ```
  - ```gcloud compute instances create my-test-vm --source-instance-template=NAME ```

### Instance-groups

 - ```gcloud compute instance-groups list```
 - ```gcloud compute instance-groups managed list```
 - ```gcloud compute instance-groups managed create NAME --zone ZONE --template TEMPLATE --size SIZE_NUMBER```
 - ```gcloud compute instance-groups managed set-autoscaling NAME --zone=INSTANCE_ZONE --max-num-replicas=NUM_REPLICAS```
 - ```gcloud compute instance-groups managed delete-autoscaling NAME --zone=INSTANCE_ZONE```
 - ```gcloud compute instance-groups managed recreate-instances GROUP_NAME --instances=INSTANCE_NAME```
 - ```gcloud compute instance-groups managed set-instance-template GROUP_NAME --template=NEW_TEMPLATE_NAME```
 - ```gcloud compute instance-groups managed resize GROUP_NAME --size=SIZE```

 After some update you can manage the new realese withou downtime:
 - ```gcloud compute instance-groups managed rolling action restart GROUP_NAME ```
 - ```gcloud compute instance-groups managed rolling action replace GROUP_NAME ```

## Instances group

Instances groups can be: 
- **Managed** (MIG) Identical instances autoscaling, autohealing, etc compatible. Also you can use:  
  - Rolling updates (step by step updates)
  - Canary deployment (test newer versions on a group of instances)

- **Unmanaged** (UIG) different type instances not compatible with autoscaling, autohealing, etc.

## Some escenarios and solutions

| Scenario | Solution |
| :------- | :--------|
| MIG managed  application that survives ZONAL failures | Create multiple zones MIG. |
| Group of multiple VMs with different configurations | Create an UIG. |
|Preserve state in a MIG (like databases)| Create stateful MIG. |
|High availability in MIG when there are hw or sw updates| Create a template with Availability police for automatic restart = on and on-host maintenance = migrate VM instance. |
| Remove unhealty instances automatically | Create a MIG with autohealing properly configured|
|Avoid scale up and downs|Cool down period (inital delay)| 

## Cloud Load balancing 

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
| Only healty instances can receive traffic| Use health check |
|High availability for instances| Use a MIG  accross different regions and configure a Load balancer.|
|Route request to multiple microservices with the same Load balancer| Configure the "The host and path rules" to redirect to specific  microservice depending on the path|
|Load balance Global external HTPPS traffic across backend instances, and multiple regions| Global external HTTP(S) Loas balancer|
|SSL termination for Global non-HTTPS traffic with Load balancing| SSL Proxy Load Balancer |

## Managed Services in Google Cloud

Quick overview

| Service | Details | Category|
| :------- | :--------| :-------:|
| Compute Engine|High performance and general purpose VM that can scale globally| IaaS |
|Google Kubernetes Engine|Orchestrate containerized microservices on Kubernetes. Needs advanced **cluster configuration and monitoring**| CaaS|
|App Engine|Enables highly scalable applications on "fully managed platform" | PaaS (CaaS Serverless)|
|Cloud Functions |Build event driven applications using simple, single-purpose functions| FaaS Serverless|
|Cloud Run |Highly scalable containerized applications.  **Does not need a cluster**| CaaS (Serverless)| 

### App Engine 
Is a service in Google cloud since 2008 wich provides end to end application managment. You pay for resources provisioned.
Some of the features that supports:
- Java, Go, .NET, Node.js, PHP, Python, Ruby preconfigured runtimes. 
- Use custom runtime and write in any language
- Connect to variety of Google Cloud storage products like Cloud SQL
- Automatic load balancing and auto-scaling
- Managed platform updates and application health monitoring
- Supports traffic spliting 

| Advatages | Disadvantages |
| :------- |  :-------|
| Lesser responsability | Lower flexibility ( Cannot add GPU)|

App engine provides two different enviroments: **Standar** and **Flexible**

### Compute Engine


| Advatages | Disadvantages |
| :------- |  :-------|
|More flexibility | More responsability ( Availability, installing software, choosing hardware and images)|

