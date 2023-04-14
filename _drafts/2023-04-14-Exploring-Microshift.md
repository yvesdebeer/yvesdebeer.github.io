---
published: false
---
## What is Microshift ?

- A small form factor OpenShift (K8s) optimized for edge devices
- Provide a minimal OpenShift experience
- Developed for resource constrained environments
- Can be managed by an orchestrator such as Open Cluster Management

## Installing Microshift on a VM

I'm using a VMWare ESXi in my lab environment so I will be using an RHEL 8.7 VM as a base.
The system requirements are very reasonable : 2 CPU cores, 2GB Ram and 10 GB of storage !

You will also need to register the machine with RedHat. 
You can get a free developer account for that at https://developers.redhat.com/

Reference documentation for installation : https://access.redhat.com/documentation/en-us/red_hat_build_of_microshift/4.12/html/installing/microshift-install-rpm

### Deploy VM on ESXI

Create a new VM on ESXi and make it boot with the RHEL 8.7 iso image.
I chose 2 CPU cores, 2GB Ram and 40 GB storage (just to be save).

Follow the procedure for installation from the link above. 
Specifically the storage configuration part.

- Verify the VG capacity








