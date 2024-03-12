# SIMUTOOL Data Lake

This overview delves into the design and purpose behind the creation of the SIMUTOOL Data Lake, a software platform developed as part of an EU Horizon 2020 project focused on computer-aided manufacturing within the aerospace and automotive sectors ([SIMUTOOL](https://doi.org/10.3030/680569), 2015-2019).

Hint: _Looking for the discussion on system design?_ Skip directly to [Architecture](https://github.com/simutool#architecture).

The project brought together 8 companies each with unique specializations, proprietary systems, and varying levels of concern regarding data confidentiality, privacy, and legal compliance. Within this collaborative framework, there existed intricate networks of interdependent data streams including simulation data, sensor data, and process control data. The objective of the software solution team was to devise a robust platform capable of supporting and enhancing the performance of such projects with intensive data sharing and exchange requirements.


### Software Solutions Team

- Professor / Supervisor: Prof. Daniela Nicklas (University of Bamberg)
- Project Lead / Architect / Developer / Research Associate: Nasr Kasrin
- Research Associates: Maliha Qureshi, Simon Steuer
- Developers: Adrian Lengenfelder, Valentina Veloso, Lukas Genssler, Katharina Broswik, Harshit Gupta

This document is the entry-point into the SIMUTOOL github organization which contains a public archive of the source code software solutions built in the project.


Project page at the University of Bamberg: [uni-bamberg.de/en/mobi/research/simutool](https://www.uni-bamberg.de/en/mobi/research/simutool/)

Project page at the European Commission's portal : [cordis.europa.eu/project/id/680569](https://cordis.europa.eu/project/id/680569)


## Achievements Overview

Our efforts yielded four key accomplishments:

1. **Accelerated R&D Processes**: By providing an integrated solution for sharing and exchanging data assets across participants and groups, we achieved a notable 30% reduction in the turnaround time of the R&D process.

2. **Enhanced Data Discovery**: We effectively slashed the time spent searching for and locating data assets by an impressive 50%.

3. **Streamlined Data Management**: Our system eliminated inefficiencies associated with requesting participants to re-upload or re-share data, thus saving valuable time.

4. **Promoted Data Asset Reusability**: Leveraging controlled vocabulary and semantic technologies, we significantly increased data asset reusability by an impressive 200%, thereby extending their lifespan beyond their original uploader's involvement.



## The Project Challenge

The project aimed to elevate the Technology Readiness Level (TRL) of microwave heating of composites for tooling and process optimization to levels 6-7. Illustrated below is a partial representation of the intricate web of data sharing and exchange relationships among various project activities and groups:

![Partial View of Data Sharing Relationships](profile/simutool-asynch-data-exchange.png)

With partners spanning eight geographically dispersed locations, each with distinct areas of expertise and considerations of confidentiality, privacy, and legality, the project faced the challenge of forging collaborations among previously unacquainted entities. The endeavor demanded to build an end-to-end software solution for streamlining the dense networks of data-driven cooperations to expedite R&D activities and amass reusable data resources.

The crux of the challenge lay not in the diversity of data assets but in the intricate interplay and data-driven dependencies across activities, particularly within simulation activities. For instance, the process control activity relied on production recipes, influenced by process simulations of the part, and vice versa. Moreover, the process control was further shaped by oven simulations, which predicted the behavior of microwave ovens based on their specifications.



## Architecture

Based on the analysis of the domain, we successfully conceptualized and constructed a comprehensive ecosystem of tools, centralized around a dynamic data lake, aimed at optimizing the seamless sharing and exchange of data assets among heterogenous systems/interfaces, participants, and groups.

The figure below showcases the deployed SaaS architecture utilized throughout the system's operational lifespan. From its inception, the system was designed with future scalability in mind, poised to accommodate increased traffic volumes and the need for enhanced performance and availability. For a detailed exploration of potential scaling scenarios for this system, refer to the section [Scaling the Architecture](https://github.com/simutool#scaling-the-architecture) below.

![](profile/simutool_system_design.drawio.svg)

### Future-Proof Design Principles

- **API-First Distributed System Design**: HTTP API interfaces took precedence in the project's design, anchored by meticulously crafted domain models.
  
- **Stateless Nodes**: Application servers, such as platform layers, were engineered to be stateless, enabling horizontal scalability effortlessly.

- **Isolated Storage Nodes**: Storage components were strategically isolated into independent nodes, laying the groundwork for both horizontal and vertical database scaling architectures, ensuring scalability.

### Key Components of the System

- **Data Lake Server ([simutool/kgservice](https://github.com/simutool/kgservice), [simutool/model-builder](https://github.com/simutool/model-builder), [simutool/dm-reader](https://github.com/simutool/dm-reader), and others)**: This component manages data lake entries, including metadata, storage, and data discovery. It establishes a semantic data model layer atop a property graph store.

- **Data Visualization Client ([simutool/om-tool](https://github.com/simutool/om-tool))**: An application designed for visualizing manufacturing sensor data, facilitating comparisons with reference data, and uploading data assets and their metadata to the KGService.

- **Native OS Client ([simutool/aku-client](https://github.com/simutool/aku-client))**: This end-user application aids users in adding and uploading data assets and their metadata to the Data Lake Server.



## External Ecosystem

The following depiction illustrates how diverse systems, activities, and user personas intertwine within the expansive ecosystem of the project. Circular identifiers denote the _external-facing_ systems of the various companies involved in the project. Additionally, a table below outlines select systems, with a comprehensive list available on page 66 of the [dissertation](https://fis.uni-bamberg.de/handle/uniba/91269) by Nasr Kasrin.

![](profile/simutool-systems-interaction.png)

| ID | Systems                          | Owner | License     | Platform       | Description                                                                                               |
|----|----------------------------------|-------|-------------|----------------|-----------------------------------------------------------------------------------------------------------|
| A  | PGD EM Solver                    | ECN   | n/a         | MATLAB         | Employs a 3D Maxwell solver for microscopic analysis of stratified media.                                  |
| B  | PGD Parameterization Tool        | ECN   | n/a         | MATLAB         | Facilitates the construction of parametric models based on simulation data.                                |
| C  | ESI CEM One                      | ESI   | Proprietary | Windows/Linux  | Offers a computational electromagnetic solution for virtual testing of large-scale industrial applications.|
| D  | ESI PAM-COMPOSITES               | ESI   | Proprietary | Windows/Linux  | Provides a simulator suite for modeling the manufacturing process of composite structural components.     |

This table showcases a glimpse of the external-facing systems, each contributing unique capabilities to the collaborative endeavor.


## Scaling the Architecture

System scalability hinges on several factors: usage patterns, desired performance and availability, and available project resources. The architecture depicted in the following figure is tailored to meet the specific requirements of our domain. 

![](profile/simutool_system_design_scaled.drawio.svg)

### Design Rationale

1. **Read-Heavy Workload**: The system primarily experiences a read-heavy workload, particularly in the **Catalog DB**, which houses metadata of data lake contents, and to a lesser extent in the **Blob Store Server**, storing heterogeneous file types in their original formats. However, two key distinctions are noted:
   - The **Catalog DB** witnesses a high rate of read traffic, albeit with small and constant data sizes per read operation.
   - Conversely, the **Blob Store Server** encounters frequent traffic spikes, handling large and diverse data sizes.

2. **Data Lake Server Load and Single Point of Failure**: Although no performance issues were encountered initially, it became evident that the first scalability measure should target the Data Lake Server.

Considering the above analysis, several dimensions of scaling are identified:

1. **Horizontal Scaling of Data Lake Server**: Leveraging the stateless architecture of the Data Lake Server, horizontal scaling can be seamlessly achieved without significant modifications. Utilizing Docker multi-container environments facilitates the deployment of new instances, supplemented by one or more load balancers to manage traffic distribution.

2. **Supporting Blob Store with CDN**: Given the spikes in data download following new data additions, horizontally partitioning (sharding) the Blob Store isn't optimal, as spikes typically involve the same data, potentially overloading specific shards. Implementing a CDN between the Blob Store and clients emerges as a superior solution to enhance data delivery efficiency.

3. **Implementing Master-Slave Architecture for Catalog DB**: Acknowledging the read-heavy nature of the Catalog DB, a master-slave replication model (write to master; read from the slave) is proposed as a straightforward and effective solution. Initial deployment could include two slave nodes, with the flexibility to scale further based on demand.

