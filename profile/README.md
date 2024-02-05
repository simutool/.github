# SIMUTOOL Data Lake SaaS (Architecture / Source Code)

This is the source code for a data lake / data management SaaS platform developed for an EU Horizon 2020 project. The information found in this repository includes a key deliverable, developed by [a team at the University of Bamberg, Germany](https://www.uni-bamberg.de/en/mobi/research/simutool/) between the years of 2015 and 2019.

The project involved a consortium of 8 organizations working cooperatively on an R\&D project to further the state-of-the-art of *microwave-assisted manufacturing* (as opposed to classical heat oven manufacturing). The work was heavily data-driven (simulation, CAD modeling, sensor data, production data, etc) and required closely coupled collaboration and quick turn-around time between various groups working in different companies across 6 geographical locations.

Based on the insights developed based on our involvement in this domain and its end-users, we designed and developed a software suite for supporting data-driven collaboration and data sharing for manufacturing projects. In this repository (and github organization) we present the technical architecture as well as the source code of some form the components we developed.

## Architecture

![](profile/simutool_system_design.drawio.svg)


The figure above presents the deployed SaaS architecture during the lifetime the system was used by the end users. The system was designed to be scalable, however we did not need to scale it further than that during its lifetime. However, key nodes have been designed as *stateless/scalable* that can be scaled when needed. See the section below for a discussion of one possible scenario to scale the system design. 

Below are the key elements of the system:

* Data Lake Server ([simutool/kgservice](https://github.com/simutool/kgservice), [simutool/model-builder ](https://github.com/simutool/model-builder), [simutool/dm-reader](https://github.com/simutool/dm-reader), and others): Manage data lake entries, in specific metadata, storage, and discovery of data lake contents. It builds a semantic data model layer on top of a property graph store. 
* Data Visualization Client ([simutool/om-tool](https://github.com/simutool/om-tool)): An application for visualizing manufacturing sensor data and comparing it with reference data, as well as uploading data assets and their metadata to the KGService.
* Native OS Client ([simutool/aku-client](https://github.com/simutool/aku-client)): End-user application to assist users to add and upload data assets and their metadata to the Data Lake Server.


## Scaling the Architecture

![](profile/simutool_system_design_scaled.drawio.svg)


System scaling varies greatly based on the context. It is at least a function of the properties and the usage patterns of the system, as well the availability rate desired and the resources available. 

During our experiences with the system above, we observed the following properties:


1. *Read-Heavy*: The system is read-heavy, particularly the **Catalog DB** (which contains the metadata of the data lake contents) and to a lesser degree the **Blob Store Server** (which stores heterogenous file types in their original format). However, there are *two key differences*:
  - The **Catalog DB** has a large read traffic rate (almost every client operation results in multiple reads), but the size of the read data is small and constant.
  - The usage of the **Blob Store Server** has an opposite nature: characterized by many traffic spikes and large and heterogenous data sizes.
2. *Data Lake Server: Heavy Load and single point-of-failure*: although we didn't face any performance problems at the time but it was clear that the first scalability action was planned on the Data Lake Server.

Due to the analysis above, there are several dimensions of scaling:

1. *Scale the Data Lake Server horizontally*:  Because the Data Lake Server was built stateless, that meant that it can be scaled horizontally without major modification. We already used Docker multi-container environments, so spinning up new instances was not a problem. In addition, one or more load balancers would be needed in front of the spun nodes.

2. *Support the Blob Store with a CDN*: Because data download (reads) typically spiked right after new data was added (this is because users could subscribe to newly added data and hence be informed as soon as such data is upload). *Horizontally partitioning* (i.e., sharding) the Blob Store is not a good idea because the spikes are typically around related data, which will end up on the same shard, so will not distribute the load. Adding a CDN (content delivery network) between the Blob Store and the clients is the better option to implement.

3. *Implement a Master-Slave (Write-Read) architecture on the Catalog DB*: Due to the read-heavy nature of the Catalog DB, the most straightforward and satisfactory solution is to implement a master-slave replication (write to master; read from the slave). Two slaves would be a good start, and more can be spun as needed.


