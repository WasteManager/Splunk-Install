<h2><u>Topography</u></h2>

- For this particular deployment I recommend a Distributed Clustered Deployment with SHC(Search head cluster) - Single Site (C3 / C13)

# Architecture Overview

The **Single Site Distributed Clustered Deployment with a Search Head Cluster (SHC)** topology uses clustering to add horizontal scalability and removes the single point of failure from the search tier.

There are no high availability (HA) requirements, meaning no runtime role, for the search head cluster deployer.

### Key Points:
- To implement an SHC, you need at least **three search heads**.
- To deploy configuration files in the cluster, use a **separate search head cluster deployer** for each SHC.
- There are no high availability (HA) requirements, meaning no runtime role, for the search head cluster deployer.
- To ensure users remain on a single search head throughout their session, use a **third-party network load-balancer** that supports sticky sessions in front of the SHC members. 
  - For more details, see [Use a load balancer with search head clustering](https://docs.splunk.com/Documentation/Splunk/latest/DistSearch/SHCwithLoadBalancer) in the Splunk Enterprise Distributed Search manual.

---

# Benefits
The benefits of this topology include:
- **Increased search capacity** beyond what a single search head can provide.
- **Distribution of scheduled search workload** across the cluster.
- **Optimal user failover** if a search head fails.

---
# Splunk Components and Their Roles

## Indexers
- **Purpose:** Store and process incoming data to make it searchable.
- **Role:**
  - Parse, index, and store the data received from forwarders or other data sources.
  - Create and manage indexes (logical data repositories) and ensure efficient storage.
  - Serve search requests from search heads by retrieving relevant indexed data.
  - Can be organized into clusters for data replication and high availability.

## Search Heads
- **Purpose:** Provide an interface for users to search and visualize data.
- **Role:**
  - Accept search queries from users and distribute them to indexers.
  - Aggregate and display search results in dashboards, reports, and alerts.
  - Enable collaboration and sharing of knowledge objects (e.g., saved searches, dashboards).
  - Can be organized into search head clusters for load balancing and redundancy.

## Cluster Managers
- **Purpose:** Manage indexer clusters for replication and high availability.
- **Role:**
  - Enforce replication and search factor settings in the indexer cluster.
  - Ensure data integrity and redundancy by replicating data across indexers.
  - Monitor the health of the indexer cluster and facilitate failover in case of node failures.
  - Act as the central authority for indexer cluster configuration.

## License Manager
- **Purpose:** Enforce compliance with Splunk's licensing model.
- **Role:**
  - Track daily indexing volume across the deployment to ensure it stays within the licensed capacity.
  - Provide reports and alerts if the license volume limit is exceeded.
  - Serve as the central repository for managing Splunk license keys.
  - Issue warnings and prevent searches if licensing violations are not addressed.

## Deployer
- **Purpose:** Manage and distribute configuration updates to search head clusters.
- **Role:**
  - Push configuration bundles (apps, settings, configurations) to search head cluster members.
  - Simplify the process of maintaining consistency across search head nodes.
  - Ensure that all cluster members are in sync with the latest configurations.

## Deployment Server
- **Purpose:** Centralize configuration management for forwarders and standalone Splunk components.
- **Role:**
  - Push configuration updates (inputs, outputs, apps) to universal and heavy forwarders.
  - Enable simplified management of large-scale forwarder deployments.
  - Ensure consistent data collection settings across the deployment.

## Monitoring Console (MC)
- **Purpose:** Provide insights into the health and performance of the Splunk deployment.
- **Role:**
  - Monitor the performance of all Splunk components (e.g., indexers, search heads, forwarders).
  - Generate dashboards and alerts for resource utilization, search performance, and indexing rates.
  - Help diagnose issues such as resource bottlenecks, dropped data, or search delays.
  - Can be run on a standalone node or within an existing Splunk component (e.g., a search head).

# Limitations
The limitation of this topology is the **lack of disaster recovery (DR) capability** if a site outage occurs.

To ensure the best experience, see [Splunk Enterprise service limits and constraints](https://docs.splunk.com/Documentation/Splunk/latest/Capacity/ReferenceHardware) in the Splunk Enterprise Capacity Planning manual.

# Backend Configurations
- Linux backend
- RHel 9
- STIGs?
- Hosted on-prem?
- Hardware requirements?
    - Search Heads
        - vCPU's - 8
        - RAM -32 GiB
        - OS disk -32GiB
        - Data Disks: 2- 512GiB
    - Indexers
        - vCPU's - 16
        - RAM - 64GiB
        - OS disk -32GiB
        - Data Disks: 3 -30000 GiB
    - Cluster Manager
        - vCPU's - 4
        - OS disk -32GiB
        - RAM - 16 GiB
        - Data Disks: 2 -4096 GiB

# 
