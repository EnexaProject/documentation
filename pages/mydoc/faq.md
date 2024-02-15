---
title: Frequently Asked Questions
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: faq.html
folder: mydoc
---

## Offline Platform

**Question**: Can I deploy the ENEXA platform completely offline without any internet access?

**Answer**: Yes, this is possible. However, it needs some additional preparation and a good knowledge about the images that will be used throughout experiments, since the platform won't be able to download Docker images.

The platform needs the following Docker images (They can also be gathered from the docker-compose file that is used to run the platform):
* `stain/jena-fuseki` (Any other triple store with a SPARQL 1.1 interface should be fine as well)
* `hub.cs.upb.de/enexa/images/enexa-service:latest`

The Docker images of the modules that should be used can be gathered from the module meta data files.

Images can be moved to an offline machine using the [`docker image save`](https://docs.docker.com/engine/reference/commandline/image_save/) and [`docker image load`](https://docs.docker.com/engine/reference/commandline/image_load/) commands.
