---
title: Shared Directory
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: shared_directory.html
folder: mydoc
---

The ENEXA platform is mainly based on the concept of a directory that is shared with all containers related to the execution of ENEXA. The sharing of data is handled by the ENEXA service. The service is configured as a web hook in the Kubernetes platform. For every container that is created within Kubernetes, the ENEXA service is contacted. If a container is marked as an ENEXA application or it belongs to an ENEXA experiment, the service adds the shared directory to this container.

However, to improve the safety of the data, the directory is shared with several restrictions. This ensures that a single experiment cannot manipulate the data of all other experiments. To this end, the directory is split into several subdirectories which are mounted with different rights into a container. Typically, all containers are allowed to read all data but can write only into a certain subdirectory.

- `/`: Only the ENEXA service has the right to write into the root directory.
- `/application-1`: Only the application with this identifier is allowed to write into this directory.
- `/application-1/experiment-1`: The experiment-1 started by application-1 is allowed to write into this directory.

Paths within the shared directory use the prefix `enexa-dir://`. This prefix has to be replaced with the local path of the shared directory to use these paths.
