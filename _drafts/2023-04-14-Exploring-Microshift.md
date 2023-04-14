---
published: false
---
## What is Microshift ?

MicroShift is an experimental flavor of OpenShift/Kubernetes: https://next.redhat.com/project/microshift/

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

### Deploy VM on ESXI & Install MicroShift

Create a new VM on ESXi and make it boot with the RHEL 8.7 iso image.
I chose 2 CPU cores, 2GB Ram and 40 GB storage (just to be save).

Follow the procedure for installation from the link above. 
Specifically the storage configuration part.

- Verify the VG capacity: (make sure the VG is called rhel)
 	
    # sudo vgs
    VG                           #PV #LV #SN Attr   VSize  VFree
  	rhel                           1   2   0 wz--n- 22.05g    0
    
- Enable MicroShift Repo's:

	# sudo subscription-manager repos \
    --enable rhocp-4.12-for-rhel-8-$(uname -i)-rpms \
    --enable fast-datapath-for-rhel-8-$(uname -i)-rpms
      
- Install MicroShift:

	# sudo dnf install -y microshift-4.12.1 openshift-clients
    
- Download your installation pull secret from the Red Hat Hybrid Cloud Console https://console.redhat.com/ -> OpenShift -> Downloads

This pull secret allows you to authenticate with the container registries that serve the container images used by MicroShift.

	# sudo vi /etc/crio/openshift-pull-secret
    
- If your RHEL machine has a firewall enabled, you must configure a few mandatory firewall rules. For firewalld, run the following commands:

    # sudo firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
    # sudo firewall-cmd --permanent --zone=trusted --add-source=169.254.169.1
    # sudo firewall-cmd --reload

- Start the MicroShift service:

	# sudo systemctl enable microshift.service
	# sudo systemctl start microshift.service

- Install client to acces the cluster:

	# curl -O https://mirror.openshift.com/pub/openshift-v4/$(uname -m)/clients/ocp/stable/openshift-client-linux.tar.gz
	# sudo tar -xf openshift-client-linux.tar.gz -C /usr/local/bin oc kubectl
    
- Access the MicroShift cluster locally:

	# mkdir ~/.kube
	# sudo cat /var/lib/microshift/resources/kubeadmin/kubeconfig > ~/.kube/config
    
- Check that pods are starting:
	
    # oc get pods -A

- Check MicroShift version:

	#microshift version
	MicroShift Version: 4.12.1
	Base OCP Version: 4.12.1
    
### Deploy a simple application


