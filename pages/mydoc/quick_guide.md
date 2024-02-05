---
title: Quick Deployment Guide
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: quick_guide.html
folder: mydoc
---
## How run the service 
- In metadata directory put all needed metadata files
- Dedicate a directory as a shared directory  


## Option one use docker  
following you can see an example of docker for running the service
```
sudo docker run -d --name [set as you wish] --network [replace with network name] 
-v [replace with valid path on your machine]:/home/shared 
-v /var/run/docker.sock:/var/run/docker.sock 
-v [replace with valid path for metadata on your machine]:/home/metadata 
-e ENEXA_META_DATA_ENDPOINT=[triple store endpoint]  
-e ENEXA_META_DATA_GRAPH=[something like http://example.org/meta-data] 
-e ENEXA_MODULE_DIRECTORY=[set] 
-e ENEXA_RESOURCE_NAMESPACE=[set something like http://example.org/enexa/] 
-e ENEXA_SERVICE_URL=[on which URL this service would be availible somethink like http://enexaservice:8080/] 
-e ENEXA_SHARED_DIRECTORY=[shared directory path] 
-e DOCKER_NET_NAME=[network name] 
[image name something like hub.cs.upb.de/enexa/images/enexa-service-demo:1.2.0]
```

## Option two run from the code (not recommended) 
- clone the code from git repository here : https://github.com/EnexaProject/enexa-service
  - set all these variables
  ```
  export ENEXA_META_DATA_ENDPOINT=[triple store endpoint]
  export ENEXA_META_DATA_GRAPH=[something like http://example.org/meta-data]
  export ENEXA_MODULE_DIRECTORY=[set]
  export ENEXA_RESOURCE_NAMESPACE=[set something like http://example.org/enexa/]
  export ENEXA_SERVICE_URL=[on which URL this service would be availible somethink like http://enexaservice:8080/]
  export ENEXA_SHARED_DIRECTORY=[shared directory path] 
  export DOCKER_NET_NAME=[network name] 
  ```
  - use "mvn clean install"
  - run it with java -jar [path of jar file like target/enexa-service-0.0.1-SNAPSHOT.jar]

## Test service is up
- After running the service you can call the "[your api address like http://localhost:8080]/test" if you recieved OK then the service is up 
