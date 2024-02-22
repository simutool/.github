# SIMUTOOL Data Lake

This article discusses the system design and motivation of a data lake / data management SaaS platform that was built for an EU _Horizon 2020_ project in the domain of computer-aided manufacturing in the aerospace and automotive industries ([SIMUTOOL](https://doi.org/10.3030/680569), 2015-2019). The goal of the project was to _increase the Technology Readiness Level (TRL) of the micorwave heating of composites (tooling and process optimization) to 6-7_. 

In this document (and github organization) we present an overview of the problem and the soltuon we developed, as well as an archive of the software we built.

## Introduction 

The project involved 8 companies with different specializations, terminologies, properietary systems, and scopes of data confidentiality, privacy, and legality.

The nature of the work and the structure of the project led to dense networks of closely coupled data-driven interdependencies including activities such as simulation, sensor measurements, process control, material development, tooling, and prototyping. 

Our role was support this project by devising a software solution to:

1. Speed up turn-around time and strengthening data-driven collaborabiltiy.
2. Support persistence and accumulation of data assets.
3. Enable discoverability and re-use of data assets.

Based on the insights we developed during our involvement in this domain and its end-users, we designed and developed a data lake / data management SaaS platform. 


## The Problem

![](profile/simutool-asynch-data-exchange.png)


The figure above depicts a partial view of some data sharing and exchange relationships between activities and groups in the project. The approach included the use of several kinds of simulations at different levels of granularity and using various methods such as electromagnetic field simulations, heat transfer simulations, oven simulations (etc.), along with other activities traditionally associated with production technology research lifecycle such as measurements, process control, material development and tooling, prototyping, etc. The project involved partners from eight geographically distributed locations with different areas of specialization and scopes of confidentiality, privacy, and legality, which did not know each other beforehand. It required the organization of various couplings of data-driven cooperations with dense networks of inter-dependencies to increase the turnover time of R&D activities and accumulate reusable data resources. 

What is challenging in this domain is not the variety of data assets, but the interrelationships and data-driven dependencies across activities, particularly the simulation activities which are more data-drive. Revisiting our Figure, we have a partial view of some data sharing and exchange couplings between activities and groups. The _process control_ activity produces production recipes (sometimes called 'curing cycles') on how to manufacture a part of family of similar parts. The process control is dependent on the _process simulation_ of the part to be manufactured in the oven, and vice versa. The process control is also influenced by the _oven simulation_ activity, which simulates the behavior of the microwave oven based on its power, size, and controls. The _part design_ activity produces CAD/CAM models and other artifacts that specify the properties of the part to be produced. In as sense, most of the activities are driven by the data produced the part design, but the closest coupling is with the _micro- and  macro-simulation_ activities which focus on the modeling the behavior of the part in the whole production process, going from the micro-level (electro-magnetic / Maxwell equations), to macro-level simulation (heat coupling). 

To complicate it even further, several of the previous activities are part of embedded cycles in which several activities closely cooperate on a problem until a stable result is reached (ex., a well-tuned, stable simulation of some part or process). One such cycle is depicted as a dashed square around the micro- and macro-simulation activities.

## Architecture

![](profile/simutool_system_design.drawio.svg)


The figure above presents the deployed SaaS architecture during the lifetime the system was used by the end users. The system was designed with future scalability in mind when traffic increases or higher performance and availability is required. See the section (Scaling the Architecture)[scaling-the-architecture] below for one possible scenario to scale this system. This was achieved in two steps:

- __Stateless nodes__: All the application servers (ex., platform layers) where designed to be stateless, hence horizontally scalable.
- __Storage isolated in independent nodes__: The architecture was built with storage components isolated in their own nodes, leaving room for horizontal and/or vertical DB scaling architectures  in order to scale both to be scalable in 
 

Below are the key elements of the system:

* Data Lake Server ([simutool/kgservice](https://github.com/simutool/kgservice), [simutool/model-builder ](https://github.com/simutool/model-builder), [simutool/dm-reader](https://github.com/simutool/dm-reader), and others): Manage data lake entries, in specific metadata, storage, and discovery of data lake contents. It builds a semantic data model layer on top of a property graph store. 
* Data Visualization Client ([simutool/om-tool](https://github.com/simutool/om-tool)): An application for visualizing manufacturing sensor data and comparing it with reference data, as well as uploading data assets and their metadata to the KGService.
* Native OS Client ([simutool/aku-client](https://github.com/simutool/aku-client)): End-user application to assist users to add and upload data assets and their metadata to the Data Lake Server.


![](profile/simutool-systems-interaction.png)

The figure above shows how different systems, activites and user personas 


## Scaling the Architecture

![](profile/simutool_system_design_scaled.drawio.svg)


System scaling varies greatly based on the context. It is at least a function of the properties and the usage patterns of the system, as well the availability rate desired and the resources available. 

During our experiences with the system above, we observed the following properties:


1. *Read-Heavy*: The system is read-heavy, particularly the **Catalog DB** (which contains the metadata of the data lake contents) and to a lesser degree the **Blob Store Server** (which stores heterogenous file types in their original format). However, there are *two key differences*:
   1. The **Catalog DB** has a large read traffic rate (almost every client operation results in multiple reads), but the size of the read data is small and constant.
   2. The usage of the **Blob Store Server** has an opposite nature: characterized by many traffic spikes and large and heterogenous data sizes.
2. *Data Lake Server: Heavy Load and single point-of-failure*: although we didn't face any performance problems at the time but it was clear that the first scalability action was planned on the Data Lake Server.

Due to the analysis above, there are several dimensions of scaling:

1. *Scale the Data Lake Server horizontally*:  Because the Data Lake Server was built stateless, that meant that it can be scaled horizontally without major modification. We already used Docker multi-container environments, so spinning up new instances was not a problem. In addition, one or more load balancers would be needed in front of the spun nodes.

2. *Support the Blob Store with a CDN*: Because data download (reads) typically spiked right after new data was added (this is because the system allowed users to subscribe to newly added data). Horizontally partitioning (i.e., sharding) the Blob Store is not a good idea because the spikes are typically around the same data, which will end up on the same shard. Adding a CDN (content delivery network) between the Blob Store and the clients is the better option here.

3. *Implement a Master-Slave (Write-Read) architecture on the Catalog DB*: Due to the read-heavy nature of the Catalog DB, the most straightforward and satisfactory solution is to implement a master-slave replication (write to master; read from the slave). Two slaves would be a good start, and more can be spun as needed.


