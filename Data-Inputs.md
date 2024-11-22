<h2><u>Data Inputs</u></h2>
Workstations
  - What data SPECIFICALLY are we interested in?

- **Exclude**: 
  - perfmon

- **Include**:
  - **Windows Event Logs**:
    - Security logs
    - Application logs
    - System logs
    - Setup logs
    - Directory Service logs
    - Windows PowerShell logs
  - DHCP Events

**Volume: Estimate daily ingestion volume**

<h1><u>Routing Architectures</u></h2>
<h2>Direct Forwarding</h1>
### Direct Data Forwarding to Splunk

In the most direct scenario, each **Universal Forwarder (UF)** sends data directly to Splunk.

---

### **Benefits**
- Minimal complexity.
- Indexing is optimized to handle many connections, with forwarders balancing over time.
- Fastest time to index with the lowest number of hops.
- Proxy-compatible using `httpout` (S2S[Splunk 2 Splunk] encapsulated in HTTP).

---

### **Limitations**
- Direct network access to indexers may not be possible.
- Network bandwidth or capability may be limited at egress.
- Splunk Cloud compatibility is limited to specific forwarder versions.
- Event-level processing is generally executed on the indexing tier.
- For Splunk Cloud forwarder certificates and applications:
  - There is an inability to revoke keys for a business unit experiencing an increase in ingest, affecting the indexers.
- Enabling event processing can impact client performance.
- Sensitive environments (e.g., PCI, PII, FedRAMP) often require IP allow lists, which may limit the number of UFs that can feasibly send data directly.

---

### **When to Use Direct Forwarding**
- When the UF is the correct choice for data collection, the most straightforward approach is to send data directly to Splunk from the forwarder.
- For **customer-managed indexing tiers**, network routing is often straightforward, and the S2S protocol is optimal.
- When sending data across the internet:
  - Use an **HTTP proxy**.
  - Configure the forwarder to send data in **HEC format** if deep packet inspection or hardware load balancing is required.

---

### **Guaranteed Delivery**
If guaranteed delivery is a requirement, see the Splunk documentation on [HTTP Event Collector Indexer Acknowledgement](https://docs.splunk.com/Documentation/Splunk/latest/Data/AboutHECIDXAck).
