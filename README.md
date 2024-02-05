# SIMUTOOL Data Lake SaaS (Source Code)

This repository contains the source code for a data lake / data management SaaS platform developed for an EU Horizon 2020 manufacturing project between the years of 2015 and 2019. The project involved a consortium of 8 organizations working cooperatively to further the know-how of microwave-assited manufacturing (as opposed to classical manufacturing in heat-ovens). The work was characterized by data-driven and close collaboration with requirements of quick turnaround time. We developed a software suite as a model for supporting the close collaboration and data sharing in industrial manufacturing projects. You can read more about the project [here](https://www.uni-bamberg.de/en/mobi/research/simutool/).

## Architecture

![](simutool_system_design.drawio.svg)


The figure above presents the deployed SaaS architecture during the lifetime the system was used by the end users. The system was designed to be scalable, however we did not need to scale to further during its lifetime. However, several nodes have been designed as *stateless/scalable nodes* that can be scaled whenever neeeded. See the next section figure below for how the deployment coult 

**Components**:

* Data Lake Server ([simutool/kgservice](https://github.com/simutool/kgservice), [simutool/model-builder ](https://github.com/simutool/model-builder), [simutool/dm-reader](https://github.com/simutool/dm-reader), and others): Manage data lake entries, in specific metadata, storage, and discovery of data lake contents. It builds a semnatic data model layer on top of a property graph store. 
* Data Visualization Client ([simutool/om-tool](https://github.com/simutool/om-tool)): An application for visualizing manufacturing sensor data and comparing it with reference data, as we well as uploading data assets and thier metadata to the KGService.
* Native OS Client ([simutool/aku-client](https://github.com/simutool/aku-client)): End-user application to assist users to add and upload data assets and thier metadata to the Data Lake Server.


## Architecture Scalability



![](simutool_system_design_scaled.drawio.svg)
