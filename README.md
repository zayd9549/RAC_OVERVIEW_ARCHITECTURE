# ORACLE RAC: OVERVIEW AND ARCHITECTURE NOTES

## PART 1: ORACLE RAC OVERVIEW

---

### 1. What is Oracle RAC?

Oracle Real Application Clusters (RAC) is a database architecture. Instead of a standard database that runs on a single server (a "single-instance" database), a RAC database runs on a **cluster of multiple servers** that operate as a single database.

All servers (called "nodes") in the cluster work together. Clients and applications can connect to any node in the cluster and access the same data. This design is the foundation for its key advantages.


---

### 2. Oracle RAC History

Oracle's clustering technology evolved from **Oracle Parallel Server (OPS)**, which was available as early as Oracle 6.2 on VAX/VMS systems. The architecture was redesigned for modern hardware, and the "Real Application Clusters" (RAC) name was introduced with **Oracle 9i**.

---

### 3. Key Advantages of Oracle RAC

The primary reasons for using Oracle RAC are High Availability and Scalability.

* **High Availability (HA):** This is the main benefit. A RAC database can survive server failures without downtime.
    * **Handles Unplanned Downtime:** It protects against downtime due to hardware and OS failures. If one server (node) in the cluster fails, the database instances on the *surviving* nodes remain active and continue to operate.
    * **Handles Planned Downtime:** You can perform maintenance (like hardware upgrades/fixing, OS patching, or database patching/upgrades) on one node at a time while the other nodes keep the database online. This is called a "rolling" patch or upgrade.
    * **SLA Implementation:** This high availability allows companies to meet very strict Service Level Agreements (SLAs) that require very high availability, such as 99.99%.

* **Database Scalability:** RAC provides "horizontal scalability".
    * Instead of "scaling up" (buying a single, massive, and very expensive server), you can "scale out" by adding more, relatively inexpensive servers (nodes) to the cluster as your workload grows.

* **Load Balancing:** The cluster can automatically distribute client connections and database workload across all the available nodes.

* **Resource Utilization:** In a RAC environment, all nodes are in an "active-active" configuration, meaning all servers are actively processing work.

---

### 4. Disadvantages (Drawbacks) of Oracle RAC

While powerful, RAC introduces costs and complexity.

* **Additional License Cost:**
    * **Oracle Database Enterprise Edition:** The Oracle RAC license is separate and must be purchased in addition.
    * **Database Standard Edition:** The Oracle RAC license is included, but *only if* the total number of CPU sockets in all the servers in the cluster is less than or equal to 4.

* **Complicated Hardware Architecture:** RAC requires a specialized and complex hardware setup and requirements, including shared storage and a private network interconnect.

* **NOT a Disaster Recovery (DR) Solution:** This is a common misconception. RAC is not designed for Disaster Recovery. It protects against **server failure** within a *single* data center, not a total site failure (like a fire or flood). For DR, you must use a separate technology like Oracle Data Guard.

* **More Complex Administration:** Managing a RAC database requires more complex DBA administration skills compared to a single-instance database.

---

### 5. Business Considerations: Should a Firm Use RAC?

Before deciding on RAC, a firm should evaluate the business case by asking:

* If the server goes down, **how long does it take to recover?**
* **How much business does the company lose** when the application is not available during that recovery time?

This cost of downtime must be weighed against the total cost of implementing RAC, which includes:

* **Hardware infrastructure cost**
* **Oracle Database and RAC licenses**
* **Management cost** (due to increased complexity)

For businesses that need high availability but not the full scalability of a multi-node cluster, Oracle also offers an alternative: **Consider Oracle RAC One Node**.

---

### 6. Why Should You Learn Oracle RAC?

* **High Enterprise Adoption:** Oracle RAC is highly used by enterprises for their critical systems.
* **High Market Demand:** Because it's complex and widely used, DBAs with Oracle RAC skills are highly demanded in the market.

---

## PART 2: ORACLE RAC ARCHITECTURE

---

### 1. Core Architectural Components

A RAC architecture is built on several key components that work together:

* **Nodes:** These are the individual servers that make up the cluster. Every server in the RAC is called a node.
* **Shared Storage:** This is the core of RAC. It is a set of disks that can be accessed, read, and written to by *all nodes* in the cluster at the same time. All the database datafiles are stored here.
* **Public Network:** This is the standard network that clients (application servers and end-users) use to connect to the database.
* **Private Network (Interconnect):** This is a dedicated, high-speed, private network used *only* for communication *between* the nodes. It is critical for:
    * **Cache Fusion:** The high-speed transfer of data blocks between the memory (SGA) of different instances.
    * **Heartbeat:** The Clusterware uses this network to send a "heartbeat" signal to check that all nodes are alive and responsive.
* **Database Instances:** A RAC database has multiple instances, with each instance running on a single node. An instance consists of memory (SGA) and background processes. For the database to be available, at least one instance must be up and running.
* **Virtual IP (VIP):** Each node has a VIP in addition to its static public IP. If a node fails, its VIP is failed over to a surviving node. This allows for fast client notification, rather than waiting for a long TCP/IP timeout.


---

### 2. Oracle Grid Infrastructure (GI)

Before you can install the Oracle Database software for RAC, you must install **Oracle Grid Infrastructure (GI)**. This software provides the foundation for the cluster.

* **Components:** GI provides both **Clusterware** (for cluster management) and **ASM** (for storage).
* **Installation:** It must be installed in its **own, local home** (separate from the Database Home).
* **Ownership:** It is usually owned by a dedicated OS user, such as `"grid"`.
* **CRS Resources:** The Clusterware (also called Cluster Ready Service or CRS) manages a list of resources, including:
    * ASM instances
    * ASM Diskgroups
    * Virtual IP (VIP) services
    * Single Client Access Name (SCAN)
    * SCAN Listeners
    * Database services
    * Oracle Net Listeners (Local Listeners)
    * Oracle Notification Service (ONS)

#### 2a. Oracle Clusterware

This is the software that manages the cluster itself. Its jobs include:

* Managing the whole cluster.
* Monitoring the availability of all resources.
* Controlling the startup and shutdown order of resources.
* Can be programmed to manage user applications as well.

Clusterware stores its configuration data in two locations:

1.  **Shared Storage:**
    * **Voting Files:** Used to determine node membership (quorum) and manage the "heartbeat".
    * **Oracle Cluster Registry (OCR):** Stores the configuration information for the entire cluster and all its resources.
    * *Note: Both of these must be saved in high-redundant storage*.

2.  **Local Node Storage:**
    * **Oracle Local Registry (OLR):** Stores metadata needed for the local node to start up *before* it can access the shared storage.
    * **Grid Plug and Play (GPNP) Profile:** A local file with the network profile and other cluster-wide settings.


#### 2b. Automatic Storage Management (ASM)

This is Oracle's recommended volume manager and cluster file system. It is specifically designed for Oracle database files, is more efficient, and is the *only* supported option for RAC on Standard Edition.

---

### 3. Hardware and Memory Requirements

#### Hardware
* **Servers:** Two or more servers.
* **OS:** Similar hardware architecture and the same operating system on all nodes.
* **Network Cards:** At least two network cards installed into every node:
    * **Public network:** Connected to the clients.
    * **Private network:** Used for the interconnect network among the nodes. A redundant private interconnect is highly recommended.
* **Shared Storage:**
    * A high-speed storage network is required, such as 16GbE Fibre Channel (FC).
    * Redundant connections to the storage with a multipathing configuration are required.

#### Memory
* Migrating from a single instance to RAC requires **more memory** for the same workload. This "RAC tax" is for:
    * **~10% more buffer cache**
    * **~15% more shared pool**
    * Memory used by the **ASM and Clusterware processes**
* Memory consumed by client sessions will be **distributed among the instances**.

---

### 4. Shared Storage Options

You must use a cluster-aware file system or volume manager so all nodes can access the storage.

* **ASM (Automatic Storage Management):** This is the recommended option. It's a volume manager and file system built by Oracle for database files.
* **Oracle ACFS (ASM Cluster File System):** A general-purpose cluster file system built on top of ASM. It extends ASM to support all file types and provides features like dynamic resizing and I/O parallelism.
* **OCFS2 (Oracle Cluster File System 2):** An open-source, general-purpose cluster file system developed by Oracle.
* **Third-party Certified cluster file systems:** Examples include Veritas Storage Foundation Cluster File System (SFCFS).
* **NFS Server:** Using the Network File System (NFSv3) protocol is also supported.
* **Raw Disks:** (Legacy/Deprecated) Using raw disk partitions without a file system.

---

### 5. Oracle Clusterware Network Configuration

This defines how names and addresses are resolved in the cluster.

* **1. Oracle Grid Naming Service (GNS):**
    * This is a dynamic option where a subdomain is delegated to the cluster in the main DNS server.
    * You only define the GNS server's VIP in the main DNS.
    * GNS then manages all other cluster addresses (VIPs, SCAN IPs, etc.) automatically.
    * This configuration supports DHCP (on Linux only).

* **2. Static Configuration:**
    * This is the more common method where no subdomain is delegated.
    * All names and addresses must be manually defined in the DNS server *before* installation. This includes:
        * A single **SCAN name** that resolves to **three static IP addresses**.
        * A public IP name and address for each node.
        * A virtual IP (VIP) name and address for each node.

---

### 6. Client Connectivity Cycle (SCAN)

In a RAC environment, clients connect using the **Single Client Access Name (SCAN)**, not a specific node's IP. This provides location independence.

Here is the 4-step connection process:

1.  **Client Request:** The client requests a connection using the **SCAN name** and a **Service Name**. The DNS or GNS server returns one of the three SCAN IP addresses to the client.
2.  **Connect to SCAN Listener:** Using the SCAN IP, the client connects to a **SCAN Listener**. A SCAN Listener is a special listener that can run on *any* node in the cluster.
3.  **Redirection to Local Listener:** The SCAN Listener checks the workload and determines which database instance hosts the requested service. It then redirects the client to the **Local Listener** on the *least-loaded node* running that service.
4.  **Final Connection:** The client disconnects from the SCAN listener and connects to the specified Local Listener, which connects the client to the local database instance.


---

### 7. RAC Database File Locations

Understanding what's shared and what's local is key.


* **Located on Shared Storage (e.g., ASM):**
    * System and user datafiles
    * Control Files
    * SPFILE (Server Parameter File)
    * Password File
    * **Per-Instance Undo Tablespace:** Each instance gets its *own* Undo tablespace.
    * **Per-Instance Redo Log Groups:** Each instance gets its *own* set of Redo Log files (called a "redo thread").

* **Located on Local Storage (on each node):**
    * **DB Home:** The Oracle Database software binaries are typically installed on the local disks of each node.
    * **Grid Home:** The Grid Infrastructure software is also on local storage.
    * *Note: The DB Home can also be placed on shared storage, but this is "not recommended"*.

---

### 8. Oracle RAC Daemons and Background Processes

A RAC environment runs two sets of processes: the **Grid Infrastructure (Clusterware)** processes that manage the cluster, and the **Database Instance** processes (both standard and RAC-specific) that manage the database.

#### 8.1. Grid Infrastructure (Clusterware) Stack Processes

These daemons (services) manage the cluster, node membership, and high availability. They run out of the Grid Home.

* **`ohasd.bin` (Oracle High Availability Services Daemon):** This is the very first process that starts on a node. It is responsible for starting and monitoring all other Clusterware daemons.
* **`orarootagent` (Oracle Root Agent):** A process run by `ohasd` as the `root` user. It manages resources owned by root, such as the network and virtual IPs (VIPs).
* **`oraagent` (Oracle Agent):** A process run by `ohasd` as the `oracle` (or grid) user. It manages resources owned by the Oracle user, like the ASM instance and listeners.
* **`ocssd.bin` (Cluster Synchronization Service):** This is one of the most critical daemons. It manages node membership, provides the cluster "heartbeat," and updates the node status in the Voting Disk.
* **`cssdagent` (CSS Agent):** An agent that monitors, starts, and stops the `ocssd` service.
* **`cssdmonitor` (CSS Monitor):** Works with the `cssdagent` to provide data integrity and monitor for node hangs or CPU starvation. It may reboot the node to protect the cluster.
* **`crsd.bin` (Cluster Ready Services Daemon):** The "high availability" daemon. It is responsible for starting, stopping, monitoring, and failing over all cluster resources (like database instances, services, listeners, and VIPs) based on the configuration in the OCR.
* **`octssd.bin` (Cluster Time Synchronization Service):** Maintains time synchronization among all nodes in the cluster if an external time source like NTP is not found.
* **`evmd.bin` (Event Manager Daemon):** A daemon that publishes events to all cluster nodes, which are used by other processes (like ONS).
* **`ons` (Oracle Notification Service):** Manages Fast Application Notification (FAN) events, which quickly notify applications about cluster changes (like a node failure or service relocation).

#### 8.2. RAC-Specific Database Instance Processes (GES/GCS)

These processes run as part of the database instance (out of the Database Home) and exist *only* in a RAC environment. They manage the "Cache Fusion" technology.

* **`LMSn` (Global Cache Service Process):** This is the main "Cache Fusion" process. It is responsible for managing and transferring data blocks between the buffer caches of different instances.
* **`LMON` (Global Enqueue Service Monitor):** Monitors the entire cluster to manage global resources and enqueues. It handles recovery and reconfiguration (e.g., when a node joins or leaves the cluster).
* **`LMDn` (Global Enqueue Service Daemon):** Manages requests for global enqueues (locks) and is responsible for detecting and resolving deadlocks *between* instances.
* **`LCK0` (Instance Enqueue Process):** Manages requests for non-cache fusion resources, such as library cache and row cache locks.
* **`DIAG` (Diagnosability Daemon):** Monitors the health of the instance and captures diagnostic data (traces) for instance or process failures.
* **`RMSn` (RAC Management Processes):** Performs various manageability tasks for the RAC database, such as creating resources when a new instance is added.

#### 8.3. Standard Database Processes (with RAC Context)

These are the standard, mandatory Oracle background processes, but some have special roles in a RAC environment.

* **`SMON` (System Monitor):** Performs instance recovery at startup. In RAC, one instance's `SMON` is responsible for performing instance recovery for *another failed instance*.
* **`PMON` (Process Monitor):** Cleans up failed user processes.
* **`LGWR` (Log Writer):** Writes the redo log buffer to the online redo log files. In RAC, each instance has its own `LGWR` writing to its own set of redo log files (known as a "redo thread").
* **`DBWn` (Database Writer):** Writes modified ("dirty") data blocks from the buffer cache to the datafiles.
* **`CKPT` (Checkpoint):** Updates the headers of all datafiles and control files to record the checkpoint position.
