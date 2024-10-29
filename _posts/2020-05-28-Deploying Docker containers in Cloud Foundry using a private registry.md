---
published: true
---
In my [previous](https://yvesdebeer.github.io/Running-Docker-Containers-on-Cloud-Foundry-in-IBM-Cloud/) blog, I explained how to create a simple NodeJS application, Dockerize it and deploy it on IBM Cloud using the public DockerHub as a registry.

This works fine if security is not an issue but there are some additional advantages of using a private registry such as the one in IBM Cloud. Not only does it protect your images, it also scans the images for security vulnerabilities.

The IBM registry is available for free to experiment within the IBM Cloud. Even for a Lite Account but beware of limited storage.

If you don’t have an account yet, you can [register](https://ibm.biz/freeaccount) for a free Lite account - no credit card needed ! <https://ibm.biz/freeaccount>

You will also need to install the IBM Cloud Cli to interact with the Container Registry.
More details on using the IBM Cloud CLI can be found here: <https://cloud.ibm.com/docs?tab=develop>

To get started with the Container Regsitry select 'Kubernetes -> Registry' from the naviation bar or goto : <https://cloud.ibm.com/kubernetes/registry/main/start>

Just follow the instructions for a "Quick Start" to connect to a registry in your region and to do some first testing:

	# docker pull hello-world
	Using default tag: latest
	latest: Pulling from library/hello-world
	0e03bdcc26d7: Pull complete 
	Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1
	Status: Downloaded newer image for hello-world:latest
	docker.io/library/hello-world:latest
	
	# docker tag hello-world de.icr.io/ydbns/hello-world:1.0
	
	# docker push de.icr.io/ydbns/hello-world:1.0
	The push refers to repository [de.icr.io/ydbns/hello-world]
	9c27e219663c: Pushed 
	1.0: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525

Let's take our simple NodeJS application and name it 'server.js' :

	const express = require('express');
	const app = express();
	const port = process.env.PORT || 3000;
	
	app.get('/', (req,res) => res.send('Hello Meetup !!!'));
	app.listen(port, () => console.log(`Example app listening on port ${port}!`))

Create a 'Dockerfile':

	FROM node
	WORKDIR /app
	COPY package.json /app
	RUN npm install
	COPY . /app
	CMD node server.js
	EXPOSE 8080

Now we can build a local Docker image:

	# docker build  . -t tinyapp
	# docker images
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	tinyapp             latest              e0e844dd9544        7 minutes ago       948MB
	node                latest              91a3cf793116        7 days ago          942MB
	
Next, tag the image with a reference to our Container registry and push it to the IBM Cloud:
	
	# docker tag tinyapp de.icr.io/ydbns/tinyapp:1.0
	# docker push de.icr.io/ydbns/tinyapp:1.0
	# ic cr images
	Listing images...
	Repository                Tag   Digest         Namespace   Created         Size     Security status   
	de.icr.io/ydbns/tinyapp   1.0   ba2f1c028c72   ydbns       4 minutes ago   365 MB   3 Issues   
	
We can already see that the vulnerability scanner has scanned the image and reports 3 issues.
You can review the issues by requesting a vulnerability assessment for the image:

	# ic cr va de.icr.io/ydbns/tinyapp:1.0
	Checking security issues for 'de.icr.io/ydbns/tinyapp:1.0'...
	
	Image 'de.icr.io/ydbns/tinyapp:1.0' was last scanned on Thu May 28 14:02:59 UTC 2020
	The scan results show that 3 ISSUES were found for the image.
	
	Configuration Issues Found
	==========================
	
	Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve   
	application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.   
	                                                           Certificate Authority (CA) certificate.                 
	application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf   
	                                                           server public key certificate. This certificate      file.   
	                                                           can be sent to the client and authenticated             
	                                                           against its CA certificate.                             
	application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.   
	                                                           server private key.      
	                                                           
	                                                           
It's up to you to decide whether you want to take actions to resolve those issues but for now we'll just go ahead and deploy the image as a Docker container in Cloud Foundry.

Before we can do that we need to create an API key to give Cloud Foundry deployment access to our registry:

	# ic iam api-keys
	# ic iam api-key-create MyKey -d "This is my API key" --file key-file
	Creating API key MyKey under f1e90dde43954fc68994fe16fb8be633 as ...
	OK
	API key MyKey was created
	Successfully save API key information to key-file
	
	# cat key-file 
	{
		"id": "ApiKey-4678aeca-4291-41f2-8901-9b31a4b560f4",
		"crn": "crn:v1:bluemix:public:iam-identity::a/f1e90dde43954fc68994fe16fb8be633F::apikey:ApiKey-4678aeca-4291-41f2-8901-9b31a4b560f4",
		"iam_id": "IBMid-550002G2B4",
		"account_id": "f1e90dde43954fc68994fe16fb8be633",
		"name": "MyKey",
		"description": "This is my API key",
		"apikey": "Au1--YZ7a3pt2EblHAb28P_55ZDGsLuDug..........",
		"locked": false,
		"entity_tag": "1-d97ba8e50dd7f9ee0c87b17b295c631d",
		"created_at": "2020-05-28T14:14+0000",
		"created_by": "IBMid-550002G2B4",
		"modified_at": "2020-05-28T14:14+0000"
	}
	
Next we'll set 'CF_DOCKER_PASSWORD' to our 'apikey' and then we are ready to deploy using 'iamapikey'.

	# export CF_DOCKER_PASSWORD=Au1--YZ7a3pt2EblHAb28P_55ZDGsLuDug..........
	# ic cf push ydbtinyapp -o de.icr.io/ydbns/tinyapp:1.0 --docker-username iamapikey
	
When the application is sucessfully deployed you can verify the correct working :

	# cf open ydbtinyapp	
	
If you want to know more about Automating access to the IBM Cloud Container Registry goto :
<https://cloud.ibm.com/docs/Registry?topic=Registry-registry_access>	

		
	
	




