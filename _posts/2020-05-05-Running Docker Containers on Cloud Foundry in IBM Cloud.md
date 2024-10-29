---
published: true
---
If you are an experienced developer already familiar with Docker, here's a quick way to just deploy your containers into the cloud without having to worry about setting up and managing a Kubernetes cluster and important also ... it comes for free using Cloud Foundry !

Let's start by creating a simple NodeJS application locally using 'npm init', give your app a name e.g. 'tinyapp' and use 'server.js' as the entry point. 

This will generate a 'package.json' file:

    {
	  "name": "tinyapp",
	  "version": "1.0.0",
	  "description": "",
	  "main": "server.js",
	  "scripts": {
	    "test": "echo \"Error: no test specified\" && exit 1"
	  },
	  "author": "",
	  "license": "ISC"
    }

### Next install the NodeJS 'Express-framework' using the command: 
    
	# npm install express --save

### Create a NodeJS application file 'server.js'

	const express = require('express');
	const app = express();
	const port = process.env.PORT || 3000;
	
	app.get('/', (req,res) => res.send('Hello there !!!'));
	app.listen(port, () => console.log(`Example app listening on port ${port}!`));

#### Test the NodeJS application locally using the command 
	
	# node server.js
	
This will launch the app locally to listen on port 3000. You can verify the working of the app with a  browser.

Now we can already deploy this app onto Cloud Foundry with the standard NodeJS runtime buildpack using the command:
	
	# cf push tinynodejs
	
Once the deployment is done we can test the app using the given URL with **Curl** or with a **Web Browser**.

Of course you will need an **IBM CLoud account** for this. 
If you don't have an account you can [register](https://ibm.biz/freeaccount) for a free Lite account - no credit card needed ! <https://ibm.biz/freeaccount>

More details on using the IBM Cloud CLI can be found here: <https://cloud.ibm.com/docs?tab=develop>

You might get an error during deployment as Cloud Foundry will try to deploy your app using 1G of memory by default and with a lite account you're limitted to 256M. In order to resolve this you can create a **manifest.yaml** file from your failed deployed application using the command : 

	# cf create-app-manifest tinynodejs -p manifest.yaml

Next you can edit the local manifest.yaml file and change the memory to e.g. 128M and do a 'cf push' again.

This is the most common way to deploy an applcation using a Cloud Foundry runtime.

### Let's now **'Dockerize'** our application.

First we need to create a **'Dockerfile'** with following content:

	FROM node
	WORKDIR /app
	COPY package.json /app
	RUN npm install
	COPY . /app
	CMD node server.js
	EXPOSE 8080

Next perform a docker build:

	# docker build . -t tinyapp

Now you can verify the docker image was stored successfully:

	# docker images
	REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
	tinyapp                   latest              e30aeb035b14        59 seconds ago      948MB

Run the container locally: 

	# docker run -p 8080:3000 -d tinyapp
	
Verify the container is running:

	# docker ps
	
And access the application using http://localhost:8080 as the initial port 3000 is now mapped to port 8080 on Docker.

	CONTAINER ID  IMAGE    COMMAND                 CREATED        STATUS        PORTS                              NAMES
	1e8571529040  tinyapp  "docker-entrypoint.s…"  7 minutes ago  Up 7 minutes  8080/tcp, 0.0.0.0:8080->3000/tcp   hungry_montalcini

You can also execute a shell command into the container using: 

	# docker exec -it 1e8571529040 /bin/bash

Next we can deploy this docker image to the cloud using Cloud Foundry **'cf push'** command but
before we do that we will first clean up the existing app:

	# cf delete tinynodejs
	
**Caution** here because deleting the app does not delete the application route ! 
You can check this with:

	# cf routes
	
So let's also delete the route: 

	# cf delete-route eu-de.mybluemix.net --hostname tinynodejs

### And now deploy the Docker container:

First we need to upload our Docker image to a central container repository e.g. <https://hub.docker.com>

- login to Dockerhub:

		# docker login --username=ydebeer
	
- tag local image using the 'IMAGE ID'

		# docker tag e30aeb035b14 ydebeer/tinynodejs:latest

- push the image to your Dockerhub repository:

		# docker push ydebeer/tinynodejs:latest

And now we can push the image to run on the IBM Cloud using CLoud Foundry:

	# cf push tinynodejs -o ydebeer/tinynodejs:latest

And there you have it: a NodeJS app running as a Docker container in the IBM Cloud.
