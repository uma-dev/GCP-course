# GCP Key features
## Zones
- GCP have 20+ zones around the world
- Each zone have al least 3 zones connected with low-l tency links.
- Each zone has one or more discrete clusters.

## Virtual Machines 
- Google Compute Engine is a SERVICE to manage and create lifecycle of VM.
- You cannot  change a instance template, you can copy it and modify the copy.
- Custom images can be used to create vm-instances with OS patches and/or software preinstalled.
- You can share an image across projects.
- **Hardening** an image is the proccess of creating an image from a vm-instance that meets the security requirements of an enterprise.
- It is preferred to use a custom image rather than startup scripts.

### Cost 
- Whenever you have a predictable resources need, you can commit the use of a VM (Kubernetes or  Compute engine) for 1 or 3 years and get up to 70% of discount.
- **Preemptible VM** only last 24 hours  or less due to system demands and they are 80% more cheaper but have a lot of disadvantages.
- Also, Preemptible VM doesn't work with free tier credits.
- The updated version of Preemptible VM is **Spot VM** and the key difference is that they don't  have a maximum runtime.
