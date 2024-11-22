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
