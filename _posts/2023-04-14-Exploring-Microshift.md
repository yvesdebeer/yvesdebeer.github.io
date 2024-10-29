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

{% highlight shell %}
# sudo vgs
VG      #PV #LV #SN Attr   VSize  VFree
rhel      1   2   0 wz--n- 22.05g    0
{% endhighlight %}
    
- Enable MicroShift Repo's:

{% highlight shell %}
# sudo subscription-manager repos \
--enable rhocp-4.12-for-rhel-8-$(uname -i)-rpms \
--enable fast-datapath-for-rhel-8-$(uname -i)-rpms
{% endhighlight %}

- Install MicroShift:

{% highlight shell %}
# sudo dnf install -y microshift-4.12.1 openshift-clients
{% endhighlight %}
    
- Download your installation pull secret from the Red Hat Hybrid Cloud Console https://console.redhat.com/ -> OpenShift -> Downloads

This pull secret allows you to authenticate with the container registries that serve the container images used by MicroShift.

{% highlight shell %}
# sudo vi /etc/crio/openshift-pull-secret
{% endhighlight %}
    
- If your RHEL machine has a firewall enabled, you must configure a few mandatory firewall rules. For firewalld, run the following commands:

{% highlight shell %}
# sudo firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16
# sudo firewall-cmd --permanent --zone=trusted --add-source=169.254.169.1
# sudo firewall-cmd --reload
{% endhighlight %}

- Start the MicroShift service:

{% highlight shell %}
# sudo systemctl enable microshift.service
# sudo systemctl start microshift.service
{% endhighlight %}

- Install client to acces the cluster:

{% highlight shell %}
# curl -O https://mirror.openshift.com/pub/openshift-v4/$(uname -m)/clients/ocp/stable/openshift-client-linux.tar.gz
# sudo tar -xf openshift-client-linux.tar.gz -C /usr/local/bin oc kubectl
{% endhighlight %}
    
- Access the MicroShift cluster locally:

{% highlight shell %}
# mkdir ~/.kube
# sudo cat /var/lib/microshift/resources/kubeadmin/kubeconfig > ~/.kube/config
{% endhighlight %}
    
- Check that pods are starting:

{% highlight shell %}
# oc get pods -A
{% endhighlight %}

- Check MicroShift version:

{% highlight shell %}
#microshift version
MicroShift Version: 4.12.1
Base OCP Version: 4.12.1
    
### Deploy a simple application

{% highlight shell %}
	# mkdir myapp
    # cd myapp
    # sudo dnf install npm
    # npm init -y
    # npm install express --save
    # cat index.js
    const express = require('express')
	const app = express()
    app.get('/', (req, res) => {
      res.send('Hello, world!')
    })
    const port = process.env.PORT || 3000
    app.listen(port, () => {
      console.log(`Server running on port ${port}`)
    })
    # vi package.json *** Add a Script : "start": "node index.js"
    # npm start *** Verify the app is working
    # cat Dockerfile
    FROM node:14-alpine
    WORKDIR /app
    COPY package.json .
    RUN npm install
    COPY . .
    CMD ["npm", "start"]
    # podman build -t myapp .
    # podman images
    # podman login quay.io
    # podman tag localhost/myapp quay.io/yvesdebeer/myapp:v1
    # podman push quay.io/yvesdebeer/myapp:v1
    # kubectl create secret docker-registry myregistrykey --docker-server=quay.io --docker-username=<username> --docker-password=<password> --docker-email=<email-address>
    # kubectl create deployment myapp --image=quay.io/yvesdebeer/myapp:v1 --image-pull-secrets=myregistrykey
    # kubectl create deployment myapp --image=quay.io/yvesdebeer/myapp:v1
    # oc get pods
    NAME                    READY   STATUS         RESTARTS   AGE
	myapp-7bfc8fbcf-5vq6v   0/1     ErrImagePull   0          39s
    # kubectl get deployment myapp -o yaml > myapp.yaml
    # vi myapp.yaml -> add the imagePullSecrets to spec -> template -> spec -> containers 			spec:
      containers:
      - image: quay.io/yvesdebeer/myapp:v1
        imagePullPolicy: IfNotPresent
        name: myapp
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      imagePullSecrets:
        - name: myregistrykey
   	# kubectl apply -f myapp.yaml
    # oc get pods
    NAME                    READY   STATUS    RESTARTS   AGE
	myapp-85fc44587-26hjm   1/1     Running   0          12s
	# oc expose svc myapp
    # oc get svc
    NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
	kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP    8h
	myapp        ClusterIP   10.43.180.45   <none>        3000/TCP   9s
    # oc get route
    NAME    HOST                             ADMITTED   SERVICE   TLS
	myapp   myapp-default.apps.example.com   True       myapp
	# cat <<EOF | oc apply -f -
    apiVersion: v1
    kind: Service
    metadata:
      name: router
      namespace: openshift-ingress
    spec:
      type: NodePort
      selector:
        ingresscontroller.operator.openshift.io/deployment-ingresscontroller: default
      ports:
        - name: http
          port: 80
          targetPort: 80
          nodePort: 30080
        - name: https
          port: 443
          targetPort: 443
          nodePort: 30443
    EOF
    # oc get svc -n openshift-ingress
    # curl -H 'Host: myapp-default.apps.example.com' 192.168.0.130:30080
    Hello, world!
  {% endhighlight %}
