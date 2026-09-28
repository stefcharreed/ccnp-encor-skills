---
name: ccnp-virtualization
description: >
  Use this skill when designing, troubleshooting, or reasoning about server virtualization
  and network functions virtualization in an enterprise network — VMs, containers, virtual
  switching, the ETSI NFV framework, VNF I/O performance, and Cisco's Enterprise NFV
  solution. Invoke when the user asks about: virtualization, server virtualization,
  containerization, bare-metal server, virtual machine, VM, VM migration, live migration,
  hypervisor, Type 1 hypervisor, Type 2 hypervisor, native hypervisor, VMware vSphere,
  VMware Fusion, Microsoft Hyper-V, Citrix XenServer, KVM, Kernel-based Virtual Machine,
  QEMU, Libvirt, guest OS, host OS, container, container image, container engine,
  container runtime, Docker, Docker engine, Docker0, rkt, rocket, Open Container
  Initiative, LXD, Linux-VServer, Windows Containers, Kubernetes, container orchestrator,
  lightweight VM, virtual switch, vSwitch, virtual bridge, Open vSwitch, OVS, vSphere
  Standard Switch, VSS, vSphere Distributed Switch, VDS, NSX vSwitch, Hyper-V Virtual
  Switch, Libvirt Virtual Network Switch, distributed virtual switching, distributed
  switch, pNIC, vNIC, veth, virtual Ethernet interface, eth0, 172.17.0.0/16, overlay
  network, NGFWv, NFV, network functions virtualization, ETSI, European
  Telecommunications Standards Institute, network function, NF, NFVI, NFV infrastructure,
  VNF, virtual network function, VNF Manager, NFV Orchestrator, MANO, management and
  orchestration, VIM, Virtualized Infrastructure Manager, element manager, EM, EMS, FCAPS,
  service chaining, OSS, BSS, Operations Support System, Business Support System, capex,
  opex, time to market, TTM, elasticity, Catalyst 8000V, Secure Firewall ASA Virtual,
  Secure Firewall Threat Defense Virtual, vWAAS, Catalyst 9800-CL, ThousandEyes, Meraki
  vMX, vEdge Cloud Router, VNF performance, north-south traffic, east-west traffic, I/O,
  I/O device, IRQ, interrupt request, interrupt handler, device driver, DMA, direct memory
  access, kernel space, user space, kernel, network stack, Rx queue, ring buffer, packet
  descriptor, socket receive buffer, OVS-DPDK, DPDK, Data Plane Development Kit, Poll Mode
  Driver, PMD, vHost-user, vHost-net, Virtio, PCI passthrough, PCIe, SR-IOV, single-root
  I/O virtualization, virtual function, VF, physical function, PF, VEB, Virtual Ethernet
  Bridge, VEPA, Virtual Ethernet Port Aggregator, ENFV, Cisco Enterprise NFV, Enterprise
  NFV solution, NFVIS, Network Functions Virtualization Infrastructure Software, ESC Lite,
  Elastic Services Controller, ENCS, Enterprise Network Compute System, Catalyst 8200
  Series Edge uCPE, uCPE, Cisco DNA Center, Catalyst Center, network profile, Plug and
  Play, PnP, zero-touch deployment, NETCONF/YANG, NSO, Network Service Orchestrator, MSX,
  Managed Services Accelerator.
---

## Purpose
Virtualization is what lets a network function stop being a box. This chapter covers two
layers of that: **server virtualization** (VMs, containers, and the vSwitches that connect
them) as the substrate, and **NFV** as the architectural framework — ETSI's standard for
decoupling network functions from proprietary appliances — plus the I/O technologies that
decide whether a virtualized firewall actually forwards at line rate, and Cisco's ENFV
packaging of all of it for the branch.

## Key Concepts

**Why server virtualization exists**
- The driver was **underutilization**: physical servers typically ran **a single operating
  system with a single application**, using only about **10% to 25% of CPU resources**. VMs
  and containers increase overall efficiency and cost-effectiveness by maximizing use of the
  available resources.
- A physical server running a single OS and dedicated to a single user is a **bare-metal
  server**.
- Virtualization using containers is also known as **containerization**.

**Virtual machines and hypervisors**
- A **virtual machine (VM)** is a **software emulation of a physical server with an operating
  system**. From an application's point of view the VM provides the look and feel of a real
  physical server, including CPU, memory, and NICs.
- A **hypervisor** is the virtualization software that creates VMs and performs the hardware
  abstraction that allows multiple VMs to run concurrently. The most popular in the server
  virtualization market: **VMware vSphere, Microsoft Hyper-V, Citrix XenServer, and Red Hat
  KVM (Kernel-based Virtual Machine)**.
- Two hypervisor types:
  - **Type 1** — runs **directly on the system hardware**. Commonly called **"bare metal"**
    or **"native."**
  - **Type 2** — **requires a host OS to run** (for example, VMware Fusion). This is the type
    **typically used by client devices**.
- **VM migration**: a VM can be moved from one server to another **while preserving
  transactional integrity during movement**. Two consequences worth remembering: a physical
  server can be upgraded (say, more memory) by migrating its VMs elsewhere **with no
  downtime**, and if a server fails the VMs can be **spun up on other servers** — high
  availability.

**Containers**
- A **container is an isolated environment where containerized applications run.** It contains
  the application along with the dependencies the application needs to run.
- **Containers are not VMs and should not be called "lightweight VMs."** The book says this
  explicitly, and the quiz tests it.
- The structural difference: **each VM requires its own guest OS; containers all share the
  same OS while remaining isolated from each other.**
  - A VM's guest OS comes with a large number of components — executables, libraries,
    dependencies — that are **not actually required for the application to run**, and it is up
    to the developer to strip the unwanted services out. A VM is basically a virtualized
    physical server, so it includes all the components of one.
  - Containers **share the underlying resources of the host OS and do not include a guest
    OS**, which is why they are lightweight. Only the application plus the specific binary
    files and libraries it needs are inside the container.
- **A container does not try to virtualize a physical server as a VM does — the abstraction is
  the application, or the components that make up the application.**
- **Container images**: a container image is a **file created by a container engine** that
  includes the application code along with its dependencies. **Container images become
  containers when they are run by the container engine.** Because the image contains
  everything the code needs, it is **extremely portable**, and it eliminates the classic
  failures — an application working on one machine but not another, or failing because the
  necessary libraries are not part of the OS and must be downloaded.
- **Startup time is the intuitive discriminator**: when a VM starts, the OS must load first,
  and only then can the application start — **usually minutes**. When a container starts, it
  **leverages the host OS kernel that is already running** — **typically a few seconds**.
- **Container engines**: **Docker** is the most popular. Others: **rkt** (pronounced
  "rocket"), **Open Container Initiative**, **LXD** (pronounced "lexdi," from Canonical Ltd.),
  **Linux-VServer**, and **Windows Containers**.

**Virtual switching**
- A **virtual switch (vSwitch)** is a **software-based Layer 2 switch that operates like a
  physical Ethernet switch**. It lets VMs communicate with each other inside a virtualized
  server and with external physical networks through the **physical NICs (pNICs)**.
- **Multiple vSwitches can be created under one virtualized server**, but two hard
  constraints apply:
  - **Network traffic cannot flow directly from one vSwitch to another vSwitch within the same
    host.**
  - **vSwitches cannot share the same pNIC.**
  - The practical consequence (Figure 27-5): if VM1 sits on vSwitch2, which has no pNIC, its
    traffic to the external network or to VM0 **must flow through a VNF in the path** — in the
    book's example, a virtual next-generation firewall (NGFWv). The "missing" adjacency is by
    design, not a misconfiguration.
- The most popular vSwitches: **Open vSwitch (OVS)**; VMware's **vSphere Standard Switch
  (VSS)**, **vSphere Distributed Switch (VDS)**, and the **NSX vSwitch**; **Microsoft Hyper-V
  Virtual Switch**; **Libvirt Virtual Network Switch**.
- **The downside of standard vSwitches**: every vSwitch in a cluster of virtualized servers
  must be **configured individually in every virtual host**. **Distributed virtual switching**
  solves this by aggregating vSwitches from a cluster of virtualized servers and treating them
  as a **single distributed virtual switch**. Benefits:
  - Centralized management of vSwitch configuration for multiple hosts in a cluster.
  - **Migration of networking statistics and policies with the VM during a live VM migration.**
  - Configuration consistency across all hosts that are part of the distributed switch.
- **Containers also rely on vSwitches** — in the container world usually called **virtual
  bridges** — for communication within a node (server) or with the outside world.
  - **Docker by default creates a virtual bridge called `Docker0`, assigned the default subnet
    block `172.17.0.0/16`.** This can be customized, and user-defined custom bridges can be
    used.
  - Every container Docker creates is assigned a **virtual Ethernet interface (`veth`)** on
    `Docker0`. **The veth appears to the container as `eth0`**, and gets an IP address from
    the bridge's subnet block.
  - **All containers can communicate with each other only if they are within the same node.**
    **Containers in other nodes are not reachable by default** — that is managed with routing
    at the OS level or with an **overlay network**.
  - **If Docker is installed on another node with the default configuration, it ends up with
    the same IP addressing as the first node**, and that has to be resolved node by node. The
    better answer is a **container orchestrator such as Kubernetes**.

**Network Functions Virtualization (NFV)**
- **NFV is an architectural framework created by ETSI** (the European Telecommunications
  Standards Institute) **that defines standards to decouple network functions from proprietary
  hardware-based appliances and have them run in software on standard x86 servers.** It also
  defines **how to manage and orchestrate** those network functions.
- A **network function (NF)** is the function performed by a physical appliance — a firewall
  function, a router function.
- Benefits (similar to server virtualization and cloud):
  - Reduced **capex and opex** through reduced equipment costs and efficiencies in space,
    power, and cooling.
  - Faster **time to market (TTM)**, because VMs and containers are easier to deploy than
    hardware.
  - Improved **ROI** from new services.
  - Ability to **scale up/out and down/in on demand (elasticity)**.
  - **Openness to the virtual appliance market** and pure software networking vendors.
  - Opportunities to **test and deploy new innovative services virtually and with lower risk**.

**The ETSI NFV architectural framework (Figure 27-7)**
- **NFVI (NFV infrastructure)** — **all the hardware and software components that comprise the
  platform environment in which VNFs are deployed**: virtual compute, virtual storage, virtual
  network, the virtualization layer, and the underlying hardware resources (compute, storage,
  network).
- **VNF (virtual network function)** — **the virtual or software version of an NF, typically
  running on a hypervisor as a VM**. Commonly used for **Layer 4 through Layer 7** functions
  such as load balancers (LBs), application delivery controllers (ADCs), firewalls, intrusion
  detection systems (IDSs), and WAN optimization appliances — **but not limited to them**; a
  VNF can also perform lower-level **Layer 2 and Layer 3** functions such as routers and
  switches. Cisco examples: **Catalyst 8000V**, **Secure Firewall ASA Virtual**, **Secure
  Firewall Threat Defense Virtual**.
- **VIM (Virtualized Infrastructure Manager)** — manages and controls the NFVI **hardware
  resources (compute, storage, network) and the virtualized resources**. Also responsible for
  the **collection of performance measurements and fault information**, **lifecycle management
  (setup, maintenance, teardown) of all NFVI resources**, and **VNF service chaining**.
- **Service chaining** — **connecting two or more VNFs in a chain to provide an NFV service or
  solution.** In Figure 27-8 the chain runs virtual load balancer → virtual firewall → virtual
  WAN optimization → virtual router, stitched together by a mix of external switches, a
  vSwitch, and a pNIC — which is why the "physical" and logical views look so different.
- **Element managers (EMs)**, also called **element management systems (EMSs)** — responsible
  for the **functional management of VNFs**; they perform **FCAPS** (fault, configuration,
  accounting, performance, and security) functions for VNFs. **A single EM can manage one or
  multiple VNFs, and an EM can itself be a VNF.**
- **MANO (management and orchestration)** — the **NFV orchestrator plus the VNF manager**,
  together.
  - The **NFV orchestrator** is responsible for **creating, maintaining, and tearing down VNF
    network services**. When multiple VNFs make up a network service, it enables the creation
    of an **end-to-end network service over multiple VNFs**.
  - The **VNF manager** manages **the lifecycle of one or multiple VNFs**, as well as FCAPS for
    the virtual components of a VNF.
- **OSS/BSS** — **OSS** is a platform typically operated by service providers and large
  enterprises to support all their network systems and services: maintaining network
  inventory, provisioning new services, configuring network devices, resolving network issues.
  For SPs, OSS typically operates in tandem with **BSS**, which is the combination of product
  management, customer management, **revenue management (billing)**, and order management
  systems used to run the SP's business operations.

**VNF performance — why the virtual layer costs you throughput**
- Two data traffic patterns in NFV solutions:
  - **North–south** — traffic comes into the hosting server through a **pNIC**, is sent to a
    VNF, then sent from the VNF **back out to the physical wire through the pNIC**.
  - **East–west** — traffic comes in through a pNIC to a VNF, and from there could be **sent to
    another VNF (service chained)**, possibly chained to more VNFs, and then back out to the
    wire through a pNIC.
  - **Combinations are normal** — a VNF may use north–south for user data and east–west to
    reach a VNF that is only collecting statistics, or being used for logs or storage.
  - These patterns and the purpose of the VNFs are what determine **which switching technology
    to use between VNFs and to the outside world** — pick wrong and the VNFs do not reach
    optimal throughput.
- The vocabulary the chapter sets up first:
  - **I/O** — communication between a computing system (such as a server) and the outside
    world. **Input** is data received by the system; **output** is data sent from it.
  - **I/O device** — a peripheral such as a mouse, keyboard, monitor, or NIC.
  - **IRQ (interrupt request)** — a **hardware signal sent to the CPU by an I/O device** to
    notify it that there is data to transfer. On receiving it the CPU **saves its current
    state, temporarily stops what it is doing, and runs an interrupt handler routine**
    associated with the device. The handler determines the cause, performs the processing,
    performs a CPU state restore, and issues a return-from-interrupt so the CPU resumes. **Each
    I/O device that generates IRQs has an interrupt handler that is part of the device's
    driver.**
  - **Device driver** — a program that controls an I/O device and allows the CPU to communicate
    with it.
  - **DMA (direct memory access)** — a memory access method allowing an I/O device to send or
    receive data **directly to or from main memory, bypassing the CPU**, to speed up overall
    operations.
  - **Kernel and user space** — the **kernel** ("core" in German) is the central part of an OS.
    It directly manages hardware such as RAM and CPU and provides system services to
    applications needing hardware access, including NICs and internal storage. Because it is
    the core, it is **executed in a protected area of main memory (kernel space)** so other
    processes cannot affect it. **Non-kernel processes execute in user space**, where
    applications and their associated libraries reside.
- **The overhead**: in a non-virtualized environment, traffic received by a pNIC is sent
  through kernel space to an application in user space. **In a virtual environment there are
  pNICs and vNICs with a hypervisor and a virtual switch in between them**, and the hypervisor
  and vSwitch must move data from the pNIC to the VM/VNF's vNIC and finally to the
  application. **That added virtual layer introduces additional packet processing and
  virtualization overhead, which creates bottlenecks and reduces I/O packet throughput.**
- **Every packet goes through the same process, so the CPU is continuously interrupted.** The
  number of interrupts **rises with high-speed NICs (for example 40 Gbps) and with small
  packet sizes**, because more packets must be processed per second. Interrupts are expensive:
  any activity the CPU is doing must be stopped, the state saved, the interrupt processed, and
  the original process restored.
- Three I/O technologies exist to avoid that overhead: **OVS-DPDK**, **PCI passthrough**, and
  **SR-IOV**. **Physical NICs that support them are required** to implement any of them.

**OVS-DPDK**
- OVS enhanced with the **Data Plane Development Kit (DPDK)** libraries. **OVS with DPDK
  operates entirely in user space.**
- The **DPDK Poll Mode Driver (PMD)** in OVS **polls** for data coming into the pNIC and
  processes it — **bypassing the network stack and the need to send an interrupt to the CPU**,
  in other words **bypassing the kernel entirely**.
- **DPDK PMD requires one or more CPU cores dedicated to polling** and handling the incoming
  data. That is the cost of admission.
- Once the packet is in OVS it is already in user space and **can be switched directly to the
  appropriate VNF**, which is where the large performance benefit comes from.

**PCI passthrough**
- Allows VNFs to have **direct access to physical PCI devices**, which appear and behave as if
  they were **physically attached to the VNF**. It can map a **pNIC to a single VNF**, and from
  the VNF's perspective it looks directly connected to that pNIC.
- Advantages: **exclusive one-to-one mapping**, **bypassed hypervisor**, **direct access to I/O
  resources**, **reduced CPU utilization**, **reduced system latency**, **increased I/O
  throughput**.
- **The downside: the entire pNIC is dedicated to a single VNF and cannot be used by other
  VNFs**, so **the number of VNFs that can use this technology is limited by the number of
  pNICs in the system.**

**SR-IOV**
- **An enhancement to PCI passthrough that allows multiple VNFs to share the same pNIC.**
- SR-IOV **emulates multiple PCIe devices on a single PCIe device** (such as a pNIC). The
  **emulated PCIe devices are virtual functions (VFs)**; the **physical PCIe devices are
  physical functions (PFs)**. **VNFs have direct access to the VFs using PCI passthrough
  technology.**
- An SR-IOV-enabled pNIC supports two modes for switching traffic between VNFs:
  - **VEB (Virtual Ethernet Bridge)** — traffic between VNFs attached to the same pNIC is
    **hardware switched directly by the pNIC**.
  - **VEPA (Virtual Ethernet Port Aggregator)** — traffic between VNFs attached to the same
    pNIC is **switched by an external switch**.

**Cisco Enterprise NFV (ENFV)**
- The problem: enterprise branches often need multiple physical devices — WAN acceleration,
  firewall, WLC, intrusion prevention, collaboration services, routing and switching —
  sometimes deployed **with redundancy**, multiplied across **many branches**.
- **Cisco ENFV is a Cisco solution based on the ETSI NFV architectural framework.** It reduces
  the operational complexity of branch environments by running the required networking
  functions as **VNFs on standard x86-based hosts** — replacing physical firewalls, routers,
  WLCs, load balancers and so on with **virtual devices running in a single x86 platform**.
- Benefits: fewer physical devices at the branch (space, power, maintenance, cooling); **fewer
  truck rolls and technician site visits**; roll out new services, critical updates, VNFs, and
  **branch locations in minutes**; **centralized management through Cisco DNA Center**;
  flexibility from VM moves, snapshots, and upgrades; **supports Cisco SD-WAN cEdge and vEdge
  virtual router onboarding**; **supports third-party VNFs**.
- **Four main components**, mapped to the ETSI framework:
  - **MANO** — **Cisco DNA Center** provides the VNF management and NFV orchestration
    capabilities, allowing easy automation of the deployment of virtualized network services
    made of multiple VNFs.
  - **VNFs** — provide the desired virtual networking functions.
  - **NFVIS** — an operating system providing virtualization capabilities and facilitating the
    deployment and operation of VNFs and hardware components.
  - **Hardware resources** — **x86-based compute** providing the CPU, memory, and storage
    required to deploy and operate VNFs and run applications.
  - **Managed service providers (MSPs) have the option of adding an OSS/BSS component** using
    **Cisco Network Service Orchestrator (NSO)** or **Cisco Managed Services Accelerator
    (MSX)**.
- **MANO detail** — DNA Center provides a centralized dashboard and tools to design, provision,
  manage, and monitor all branch sites. **Its two main functions are rolling out new branch
  locations and deploying new VNFs and virtualized services.**
  - **Centralized policies are created by building network profiles.** Multiple network
    profiles can exist, each with specific design requirements and virtual services; branch
    sites are then **assigned to the profile that matches their requirements**. A network
    profile includes: configuration for **LAN and WAN virtual interfaces**; the **services or
    VNFs** to be used and their requirements such as **service chaining parameters, CPU, and
    memory**; and the **device configuration required for the VNFs**, customizable through
    **custom configuration templates** created with a template editor tool.
  - **Plug and Play provisioning** automatically and remotely provisions and onboards new
    devices. When a new ENFV platform is brought up for the first time it uses **PnP to
    register with DNA Center**; DNA Center then **matches the site to the network profile
    assigned for that site** and provisions and onboards the device automatically.
- **VNFs and applications** — ENFV virtualizes both **network functions and applications** in
  the branch. **Both Cisco and third-party VNFs can be onboarded**, and applications running
  in a **Linux or Windows server environment can also be instantiated on top of NFVIS** and
  supported by DNA Center.
  - **Cisco-supported VNFs**: Catalyst 8000V Edge (Viptela SD-WAN and virtual routing), vEdge
    SD-WAN Cloud Router, Secure Firewall ASA Virtual, Secure Firewall Threat Defense Virtual,
    vWAAS (virtualized WAN optimization), Catalyst 9800-CL Cloud Wireless Controller,
    ThousandEyes, Meraki vMX.
  - **Third-party vendors supported**: Microsoft Windows Server, Linux Server, Accedian, AVI
    Networks, Check Point, Citrix, CTERA, F5, Fortinet, InfoVista, NETSCOUT, Palo Alto
    Networks, Riverbed Technology.

**NFVIS**
- **NFVIS is based on standard Linux packaged with additional functions for virtualization,
  VNF lifecycle management, monitoring, device programmability, and hardware acceleration.**
- Components:
  - **Linux** — drives the underlying hardware platforms (ENCS, Cisco UCS servers, x86 enhanced
    network devices) and hosts the virtualization layer for VNFs, virtual switching API
    interfaces, interface drivers, platform drivers, and management.
  - **Hypervisor** — based on **KVM**, including **QEMU**, **Libvirt**, and other associated
    processes.
  - **vSwitch** — **Open vSwitch (OVS)**, enabling communication **between different VNFs
    (service chaining)** and to the outside world.
  - **VM lifecycle management** — NFVIS provides the **VIM functionality** specified in the NFV
    architectural framework, through the embedded **Elastic Services Controller (ESC) Lite**.
    ESC-Lite supports **dynamic bringup of VNFs** — creating and deleting VNFs and adding CPU
    cores, memory, and storage — plus **built-in VNF monitoring that auto-restarts VNFs when
    they are down** and sends alarms (SNMP or syslog).
  - **Plug and Play client** — automates bringing up any NFVIS-based host, communicating with a
    **PnP server running in Cisco DNA Center** and being provisioned with the right host
    configuration. Enables **a true zero-touch deployment model** (no human intervention).
  - **Orchestration** — **REST, CLI, HTTPS, and NETCONF/YANG** communication models are
    supported.
  - **HTTPS web server** — connectivity into NFVIS through HTTPS to a local device web portal,
    from which you can upload VNF packages, do full lifecycle management, turn services up and
    down, connect to VNF consoles, and monitor critical parameters **without complex commands**.
  - **Device management** — includes a **resource manager** to report the number of CPU cores
    allocated to VMs and the cores already used by them.
  - **RBAC** — users accessing the platform are authenticated using role-based access control.
- **x86 hosting platforms** for Cisco Enterprise NFVIS: **Cisco Enterprise Network Compute
  System (ENCS)** and **Cisco Catalyst 8200 Series Edge uCPE**. Which one to choose depends on
  **VoIP requirements, non-Ethernet interfaces (T1 or DSL), 4G-LTE, the I/O technologies
  supported (for example SR-IOV), and the number of CPU cores needed for existing and future
  service requirements.**

## Procedure

**OVS packet flow — pNIC to the application inside a VM (Figure 27-10), 11 steps:**
1. Data traffic is received by the **pNIC** and placed into an **Rx queue (ring buffers)**
   within the pNIC.
2. The pNIC sends the packet and a **packet descriptor** to the **main memory buffer through
   DMA**. The packet descriptor includes **only the memory location and size** of the packet.
3. The pNIC sends an **IRQ to the CPU**.
4. The CPU transfers control to the **pNIC driver**, which services the IRQ, receives the
   packet, and moves it into the **network stack**, where it eventually arrives in a socket and
   is placed into a **socket receive buffer**.
5. The packet data is **copied from the socket receive buffer to the OVS virtual switch**.
6. OVS processes the packet and forwards it to the VM. **This entails switching the packet
   between the kernel and user space, which is expensive in terms of CPU cycles.**
7. The packet arrives at the **vNIC** of the VM and is placed into an **Rx queue**.
8. The vNIC sends the packet and a packet descriptor to the **virtual memory buffer through
   DMA**.
9. The vNIC sends an **IRQ to the vCPU**.
10. The vCPU transfers control to the **vNIC driver**, which services the IRQ, receives the
    packet, and moves it into the network stack, where it arrives in a socket and is placed
    into a socket receive buffer.
11. The packet data is **copied and sent to the application in the VM**.

> Steps 3–4 and 9–10 are the interrupt pairs, and step 6 is the kernel/user-space crossing.
> Those are exactly the costs OVS-DPDK (poll instead of interrupt, all user space), PCI
> passthrough (bypass the hypervisor entirely), and SR-IOV (same, but shared) each remove.

**Choosing an I/O acceleration technology:**
1. **Confirm the pNIC supports the technology** — none of the three work without hardware
   support.
2. Characterize the traffic: is it **north–south** (in a pNIC, to a VNF, back out) or
   **east–west** (service chained between VNFs before leaving)? East–west chains benefit most
   from a fast software path; pure north–south per-VNF flows benefit most from bypassing the
   software path altogether.
3. **Many VNFs, few pNICs, heavy VNF-to-VNF switching** → **OVS-DPDK**. You keep a flexible
   software switch and pay for it with **one or more dedicated CPU cores** for the poll mode
   driver.
4. **One VNF that needs maximum throughput and lowest latency from a dedicated pNIC** →
   **PCI passthrough**. Accept that the pNIC is consumed entirely by that VNF.
5. **Multiple VNFs that each want passthrough-class performance on a shared pNIC** →
   **SR-IOV**. Then choose the switching mode: **VEB** (pNIC hardware switches VNF-to-VNF
   traffic) or **VEPA** (an external switch does, which is what you want when that traffic must
   be seen by physical-network policy or monitoring).

**Deploying a branch with Cisco ENFV:**
1. **Build a network profile** in Cisco DNA Center with the LAN/WAN virtual interface
   configuration, the VNFs and services required, their service chaining parameters, CPU and
   memory requirements, and any custom configuration templates for the VNFs.
2. **Assign the branch site to the network profile** that matches its requirements.
3. **Bring up the ENFV platform** (ENCS or Catalyst 8200 Series Edge uCPE) for the first time;
   the **NFVIS PnP client registers with the PnP server in DNA Center**.
4. **DNA Center matches the site to its assigned network profile** and **provisions and
   onboards the device automatically** — zero-touch, no human intervention.
5. **Add or chain services** as needed from the DNA Center Add Services window, connecting VNFs
   across LAN, management, and services interfaces.

## Reference Tables

**VMs vs containers**

| | Virtual machine | Container |
|---|---|---|
| What is abstracted | **A physical server** | **The application**, or the components that make it up |
| Guest OS | **Each VM has its own** | **None — all share the host OS**, while staying isolated |
| Size | Heavy — includes components the app does not need; the developer must strip them | **Lightweight** — only the app plus its binaries and libraries |
| Startup | OS loads first, then the app — **usually minutes** | Leverages the already-running host kernel — **typically seconds** |
| Runs on | A **hypervisor** | A **container engine**, from a **container image** |
| Portability | Migratable between servers with transactional integrity preserved | **Extremely portable** — the image carries everything the code needs |
| Do not call it | — | **"a lightweight VM"** |

**Hypervisor types**

| Type | Runs on | Also called | Typical use |
|---|---|---|---|
| **Type 1** | **Directly on the system hardware** | **"bare metal"**, **"native"** | Server virtualization |
| **Type 2** | **On top of a host OS** (e.g., VMware Fusion) | — | **Client devices** |

**Container engines**

| Engine | Note |
|---|---|
| **Docker** | **The most popular** |
| **rkt** | Pronounced "rocket" |
| **Open Container Initiative** | |
| **LXD** | Pronounced "lexdi", from Canonical Ltd. |
| **Linux-VServer** | |
| **Windows Containers** | |

**ETSI NFV framework components**

| Component | Responsibility |
|---|---|
| **NFVI** | All hardware and software comprising the platform environment where VNFs are deployed — virtual compute/storage/network, virtualization layer, hardware resources |
| **VNF** | The virtual/software version of an NF, **typically running on a hypervisor as a VM**. Usually L4–L7 (LB, ADC, firewall, IDS, WAN opt), but also L2–L3 (routers, switches) |
| **EM / EMS** | **Functional** management of VNFs — **FCAPS**. One EM can manage several VNFs, **and an EM can itself be a VNF** |
| **VIM** | Manages/controls NFVI hardware and virtualized resources; performance measurement and fault collection; NFVI resource lifecycle; **VNF service chaining** |
| **VNF Manager** | Lifecycle of one or multiple **VNFs**, plus FCAPS for a VNF's virtual components |
| **NFV Orchestrator** | **Creating, maintaining, tearing down VNF network services**; end-to-end service across multiple VNFs |
| **MANO** | **NFV Orchestrator + VNF Manager together** |
| **OSS / BSS** | OSS: network inventory, service provisioning, device configuration, issue resolution. BSS: product, customer, **revenue (billing)**, and order management |

**I/O acceleration technologies compared**

| | Standard OVS | OVS-DPDK | PCI passthrough | SR-IOV |
|---|---|---|---|---|
| Where it runs | Kernel space (with user-space crossing) | **Entirely user space** | Bypasses the hypervisor | Bypasses the hypervisor |
| How data is taken in | **Interrupts (IRQ)** | **Polling — DPDK Poll Mode Driver**, kernel bypassed entirely | Direct access to the physical PCI device | Direct access to a **VF** via PCI passthrough |
| pNIC sharing | Shared | Shared | **Exclusive one-to-one — one VNF owns the pNIC** | **Shared — multiple VNFs per pNIC** |
| Cost / limit | Packet processing and virtualization overhead; CPU continuously interrupted | **Requires one or more dedicated CPU cores** for the PMD | **VNF count limited by the number of pNICs** | Requires SR-IOV-capable pNIC |
| VNF-to-VNF switching | In OVS | In OVS (user space) | n/a | **VEB** (pNIC hardware switches) or **VEPA** (external switch switches) |
| Requires supporting pNIC | No | **Yes** | **Yes** | **Yes** |

**Cisco ENFV solution components**

| Component | What fills the role |
|---|---|
| **MANO** | **Cisco DNA Center** — VNF management and NFV orchestration |
| **VNFs** | Cisco and third-party virtual networking functions |
| **NFVIS** | The OS providing virtualization and VNF/hardware deployment and operation |
| **Hardware resources** | **x86-based compute** — CPU, memory, storage |
| *(optional, MSPs)* | **OSS/BSS** via **Cisco NSO** or **Cisco MSX** |
| **x86 hosting platforms** | **Cisco ENCS**, **Cisco Catalyst 8200 Series Edge uCPE** |

**NFVIS components**

| Component | What it provides |
|---|---|
| **Linux** | Drives the hardware (ENCS, UCS, x86 enhanced network devices); hosts the virtualization layer, virtual switching APIs, interface and platform drivers, management |
| **Hypervisor** | **KVM**, including **QEMU**, **Libvirt** |
| **vSwitch** | **OVS** — VNF-to-VNF communication (service chaining) and to the outside world |
| **VM lifecycle management** | **VIM functionality via embedded ESC Lite** — dynamic VNF bringup, add CPU/memory/storage, VNF monitoring with **auto restart** and SNMP/syslog alarms |
| **PnP client** | Talks to the **PnP server in Cisco DNA Center**; **true zero-touch deployment** |
| **Orchestration** | **REST, CLI, HTTPS, NETCONF/YANG** |
| **HTTPS web server** | Local device web portal — upload VNF packages, full lifecycle management, service up/down, VNF consoles, monitoring |
| **Device management** | Resource manager — CPU cores allocated to VMs and cores already used |
| **RBAC** | Authenticates users accessing the platform |

## Config Patterns

> **Provenance:** Chapter 27 contains **no device configuration at all** — it is a concepts and
> architecture chapter, and its only interface artifacts are ETSI framework diagrams and a
> Cisco DNA Center GUI screenshot (Figure 27-15, the Add Services window). Rather than invent
> IOS-XE that the chapter never shows, the commands below are the small set of **host-side
> commands that make the chapter's concepts visible on a real box**. They are **added by this
> skill, not transcribed from the chapter, and are not gear-validated.** Verify NFVIS syntax
> against the NFVIS command reference for your release before using it.

**Seeing the Docker bridge the chapter describes (`Docker0`, `172.17.0.0/16`, veth → eth0)**
```bash
docker network ls                      # the default 'bridge' network is Docker0
docker network inspect bridge          # shows the subnet, gateway, and attached containers
ip -br addr show docker0               # the bridge IP, 172.17.0.1/16 by default
ip -br link show type veth             # the host-side veth pairs, one per container
docker exec <container> ip -br addr    # inside the container the veth appears as eth0
```
> This is the concrete version of Figure 27-6. Run it on two separate Docker hosts with
> default configuration and you will reproduce the chapter's warning directly: **both nodes
> come up with the same `172.17.0.0/16`**, and containers on one node cannot reach the other.

**Open vSwitch — confirming bridges and ports**
```bash
ovs-vsctl show                         # bridges, ports, and their attached interfaces
ovs-vsctl list-br                      # bridge list
ovs-ofctl dump-flows <bridge>          # the flows OVS is actually applying
ovs-vsctl get Open_vSwitch . dpdk_initialized   # whether DPDK is active on this OVS
```

**SR-IOV — confirming VFs exist on a pNIC**
```bash
lspci | grep -i "virtual function"     # the VFs SR-IOV has emulated
cat /sys/class/net/<pNIC>/device/sriov_totalvfs   # VFs the PF can support
cat /sys/class/net/<pNIC>/device/sriov_numvfs     # VFs currently instantiated
ip link show <pNIC>                    # lists the VFs and their MAC/VLAN assignment
```

**NFVIS — VNF and resource state** *(verify against the NFVIS command reference)*
```
show system deployments all
show vm_lifecycle deployments all
show system-monitoring host cpu-table
show bridges
show pnic
```

## Design Baseline

Every row traces to the ENCOR 350-401 OCG Chapter 27 with a page number — the chapter states
these constraints and recommendations directly, so nothing here was sourced from memory.
**A deviation is a question for the network's operator, not automatically a finding.**

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| **Plan a VNF in the path when traffic must cross between vSwitches on the same host** | **Traffic cannot flow directly from one vSwitch to another within the same host, and vSwitches cannot share a pNIC.** The book's own Figure 27-5 routes VM1 through an NGFWv for exactly this reason | None — this is a hard platform constraint, not a preference. The choice is *which* VNF sits in the path | Ch. 27, p. 831 |
| **Use distributed virtual switching for a cluster of virtualized servers** rather than standard vSwitches | A standard vSwitch **must be configured individually in every host in the cluster**; distributed switching gives centralized config, **policy and statistics migration during live VM migration**, and consistency across hosts | A single host, or a cluster whose hypervisor edition does not license distributed switching | Ch. 27, p. 832 |
| **Use a container orchestrator such as Kubernetes** instead of default per-node Docker bridging once containers span more than one node | **Docker's default config gives every node the same `172.17.0.0/16`**, and containers in other nodes are **unreachable by default** — otherwise it is routing at the OS level or an overlay, resolved node by node | A single-node deployment, or a deliberately customized bridge subnet per node with routing already solved | Ch. 27, pp. 832–833 |
| **Confirm the pNIC supports the I/O technology before designing around it** | **OVS-DPDK, PCI passthrough, and SR-IOV all require physical NICs that support them** | None — this is a hardware prerequisite | Ch. 27, p. 839 (NOTE) |
| **Budget one or more dedicated CPU cores for the DPDK Poll Mode Driver** when using OVS-DPDK | The PMD **polls** rather than waiting on interrupts, and **requires dedicated cores** to do so. Cores not accounted for at design time come out of the VNFs' budget | None — the requirement is architectural | Ch. 27, p. 839 |
| **Do not plan more PCI-passthrough VNFs than you have pNICs** | **The entire pNIC is dedicated to a single VNF and cannot be used by others**, so pNIC count is a hard ceiling on VNF count | Use **SR-IOV** instead when VNFs must share a pNIC — that is precisely what it was built for | Ch. 27, pp. 840–841 |
| **Choose the SR-IOV switching mode deliberately: VEB vs VEPA** | **VEB** switches VNF-to-VNF traffic **in the pNIC hardware**, so it never reaches the physical network; **VEPA** sends it to an **external switch**. If that traffic must be subject to physical-network policy or monitoring, VEB hides it | VEB when raw east–west performance matters more than external visibility | Ch. 27, p. 841 |
| **Choose the ENFV x86 platform on service requirements, not just price** — VoIP, non-Ethernet interfaces (T1/DSL), 4G-LTE, I/O technologies such as SR-IOV, and CPU cores | The chapter names these explicitly, and the CPU core count must cover **future** service requirements, not only current VNFs | None stated — this is the selection method | Ch. 27, p. 847 |
| **Build ENFV branches from network profiles and let PnP onboard them** | Profiles give **consistent policy across branches**, and PnP gives **true zero-touch deployment** — the operational payoff the whole solution is sold on | A one-off branch with genuinely unique requirements still gets a profile; the profile is the unit of consistency | Ch. 27, pp. 843–844 |

## Verification Commands

> Chapter 27 shows no CLI, so this table is the host- and platform-side equivalent — where to
> look to confirm each concept is real on a running system. **Added by this skill; verify
> platform-specific syntax against the relevant command reference.**

| Command | What to look for |
|---------|-----------------|
| `docker network inspect bridge` | The Docker0 subnet (default **172.17.0.0/16**), gateway, and which containers are attached — proves or disproves the "same subnet on every node" problem |
| `ip -br link show type veth` | One host-side **veth** per running container; the container sees its end as **eth0** |
| `docker exec <c> ip -br addr` | The container's `eth0` address, from the bridge's subnet block |
| `ovs-vsctl show` | OVS bridges, ports, and attached interfaces — the vSwitch topology as OVS actually holds it |
| `ovs-ofctl dump-flows <bridge>` | The flows being applied, with packet counters — proves traffic is taking the path you think |
| `ovs-vsctl get Open_vSwitch . dpdk_initialized` | Whether **DPDK** is actually active, rather than assumed |
| `lspci \| grep -i "virtual function"` | The **VFs** SR-IOV has emulated on the pNIC |
| `cat /sys/class/net/<pNIC>/device/sriov_numvfs` | How many VFs are instantiated vs `sriov_totalvfs` supported — the ceiling on SR-IOV VNFs for that pNIC |
| `ip link show <pNIC>` | The PF plus its VFs, with per-VF MAC and VLAN — confirms which VNF owns which VF |
| `virsh list --all` / `virsh domiflist <vm>` | KVM/Libvirt VM inventory and each VM's vNIC-to-bridge mapping — the NFVIS hypervisor layer |
| `show system deployments all` *(NFVIS)* | Deployed VNFs and their state |
| `show vm_lifecycle deployments all` *(NFVIS)* | VNF lifecycle state — the ESC-Lite view, including VNFs that auto-restarted |
| `show system-monitoring host cpu-table` *(NFVIS)* | CPU cores allocated vs used — the resource manager view that tells you whether another VNF fits |
| `show bridges` / `show pnic` *(NFVIS)* | NFVIS OVS bridges and physical NIC state |
| Cisco DNA Center → site / network profile view | Which profile a branch is assigned to, and whether PnP onboarding completed |

## Intent Questions

- **Is this workload supposed to be a VM or a container, and why?** They are not
  interchangeable: a VM virtualizes a server and takes minutes to start; a container
  virtualizes an application and takes seconds. Choosing wrong shows up as either wasted
  resources or an app that needed an OS it does not have.
- **Which vSwitch is each VNF supposed to be on, and does any required flow cross vSwitches?**
  If it does, there must be a VNF in the path by design — because the platform will never
  bridge two vSwitches in the same host for you.
- **What is the expected traffic pattern — north–south, east–west, or both?** This is the
  question that picks the I/O technology, and it has to be answered before the pNICs are
  chosen, not after.
- **How many pNICs does this host have, and how many VNFs need direct hardware access?** That
  ratio decides PCI passthrough vs SR-IOV, and it is a hard ceiling, not a tuning knob.
- **For ENFV: which network profile is this branch assigned to, and does the profile actually
  describe what the branch needs?** In an ENFV deployment the profile *is* the intent — a
  branch behaving unexpectedly is usually assigned to the wrong profile rather than
  misconfigured locally.

## Troubleshooting Checklist

0. **State intent vs. observed.** Answer the Intent Questions above, then write the one-line
   symptom ("the branch firewall VNF should be service chained between the router and the LAN,
   traffic is bypassing it") — before opening a single console.
1. **Is it a VM or a container problem?** A container that will not start is an image or
   dependency problem; a VM that will not start is a hypervisor, resource, or image problem.
   They have almost no overlapping failure modes.
2. **Containers: same node or different nodes?** **Containers reach each other by default only
   within the same node.** Cross-node failure is expected behavior without routing at the OS
   level or an overlay — not a bug. Check whether an orchestrator is supposed to be handling
   this.
3. **Containers: duplicate subnets across nodes?** Two Docker hosts installed with defaults
   both use **172.17.0.0/16**. Symptoms look like bizarre routing; the cause is address
   collision.
4. **VM-to-VM traffic failing within one host: are they on the same vSwitch?** If not,
   **traffic cannot flow directly between vSwitches on the same host** and there must be a VNF
   bridging them. `ovs-vsctl show` (or the hypervisor equivalent) to see the real topology.
5. **Is the vSwitch attached to a pNIC at all?** A vSwitch with no pNIC has no path to the
   external network — Figure 27-5's vSwitch2. Also confirm **no two vSwitches are expected to
   share one pNIC**, which is not supported.
6. **Cluster-wide inconsistency after a VM migration?** With **standard** vSwitches, each host
   is configured separately and **networking statistics and policies do not follow the VM**.
   With a **distributed** switch they do. A VM that works on host A and breaks on host B is
   this, most of the time.
7. **Throughput far below the pNIC's rate?** Walk the 11-step OVS path. The expensive points
   are the **interrupts (steps 3–4 and 9–10)** and the **kernel/user-space crossing (step 6)**.
   Confirm whether any acceleration is actually enabled — `ovs-vsctl get Open_vSwitch .
   dpdk_initialized`, `lspci` for VFs — rather than assumed from the design document.
8. **Throughput worse with small packets or a fast NIC?** Expected: **more packets per second
   means more interrupts**, and interrupt overhead is per packet, not per byte. That is the
   signature that says "you need DPDK, passthrough, or SR-IOV," not "the NIC is faulty."
9. **Acceleration configured but not working?** **The pNIC must support the technology.** Check
   hardware support before configuration. For SR-IOV also check `sriov_numvfs` is non-zero.
10. **OVS-DPDK configured but performance unchanged?** The **PMD needs one or more dedicated
    CPU cores**. Without them the poll mode driver has nothing to poll with.
11. **Cannot add another passthrough VNF?** With PCI passthrough **the whole pNIC belongs to
    one VNF** — you are out of pNICs, not out of capacity. SR-IOV is the answer if the VNFs can
    share.
12. **SR-IOV VNF-to-VNF traffic invisible to the physical network's monitoring or policy?**
    That is **VEB** doing what it does — hardware switching inside the pNIC. **VEPA** is the
    mode that sends it to an external switch.
13. **ENFV branch onboarded but wrong services?** Check which **network profile** the site is
    assigned to before touching the device. DNA Center matches the site to its profile and
    provisions from that — the device is downstream of the profile.
14. **ENFV platform never came up?** Trace the **PnP client → PnP server in DNA Center**
    registration. Zero-touch fails silently from the branch's point of view.
15. **VNF keeps restarting?** **ESC-Lite auto-restarts VNFs when they are down and sends SNMP
    or syslog alarms.** The restarts are the symptom being reported to you, not the fault —
    find the alarm and the underlying resource or image problem.
16. **No room for another VNF?** `show system-monitoring host cpu-table` (or the DNA Center
    view) for cores allocated vs used. Remember the platform choice itself is driven by core
    count for **future** requirements, so this may be a sizing decision, not a fault.

## Common Pitfalls

- **A container is not a lightweight VM.** The book states this outright and the quiz punishes
  it. A VM abstracts a **physical server**; a container abstracts the **application**.
- **A VM is an emulation of a *physical* server, not a virtual one.** The quiz's wrong answers
  all say "virtual server" — a VM emulates physical hardware, which is why the guest OS is
  fooled.
- **Containers do need vSwitches.** They rely on vSwitches — called **virtual bridges** in the
  container world — to talk to each other and the outside world. Docker's is `Docker0`.
- **Traffic cannot flow between two vSwitches in the same host, and vSwitches cannot share a
  pNIC.** This is the single most surprising constraint in the chapter and the reason a VNF
  ends up in a path that "should" be a simple Layer 2 hop.
- **More than one vSwitch per virtualized server is supported** — the constraint is on traffic
  between them, not on their existence. (Quiz Q5's trap.)
- **Containers in different nodes are unreachable by default.** No routing, no overlay, no
  orchestrator means no cross-node container traffic — by design.
- **Every default Docker install uses `172.17.0.0/16`.** Two nodes with defaults collide, and
  it must be resolved node by node unless an orchestrator is doing it for you.
- **Type 2 hypervisors need a host OS; Type 1 does not.** "Bare metal" and "native" both mean
  **Type 1**, which collides confusingly with **bare-metal server** — a physical server running
  one OS for one user. Same two words, two unrelated meanings, both in this chapter.
- **VNFs are not limited to Layer 4 through Layer 7.** They are *commonly* LBs, ADCs,
  firewalls, IDSs, and WAN optimizers, but they can also be routers and switches.
- **An EM can itself be a VNF**, and one EM can manage several VNFs — the framework boxes in
  Figure 27-7 are roles, not necessarily separate appliances.
- **MANO is two things, not one**: the **NFV orchestrator** (creates, maintains, tears down
  network *services*) plus the **VNF manager** (lifecycle of individual *VNFs*). The split
  matters because they fail independently.
- **VIM manages NFVI resources; the VNF manager manages VNFs.** Both do "lifecycle
  management," of different objects.
- **Service chaining is the term** for connecting VNFs to build a service. "Daisy chaining,"
  "bridging," "linking" are not. (Quiz Q9's trap.)
- **OVS-DPDK's cost is dedicated CPU cores.** It is not free performance — the poll mode driver
  burns cores continuously by design, which is the whole point of not being interrupt-driven.
- **PCI passthrough consumes a whole pNIC per VNF.** Elegant for one VNF, a scaling dead end
  for several.
- **VFs are the emulated ones, PFs are the physical ones.** Easy to reverse under exam
  pressure; "V for virtual" is the whole mnemonic. (Quiz Q10.)
- **SR-IOV is an enhancement to PCI passthrough, not an alternative to it** — VNFs still reach
  their VFs *using* PCI passthrough technology.
- **All three I/O technologies need pNIC hardware support.** Designing around SR-IOV on a NIC
  that does not do SR-IOV is a procurement problem discovered at deployment time.
- **Cisco DNA Center is the ENFV orchestrator**, not APIC-EM and not the APIC controller (which
  is ACI). (Quiz Q11's trap — all the distractors are real Cisco platforms.)
- **NFVIS is standard Linux with additions**, and it supplies the **VIM** role via **ESC-Lite**.
  Do not look for a separate VIM appliance in the ENFV architecture — there are four
  components, and VIM is inside NFVIS.
- **ENFV has four components, and OSS/BSS is not one of them by default** — it is an **optional
  addition for MSPs** via NSO or MSX.

## Exam Preparation Tasks

### Key topics coverage map

Table 27-2, mapped to where each element lives in this skill. The mapping is not one-to-one —
the three I/O technologies share a single Reference Table here, and the ENFV rows are split
across Key Concepts, Procedure, and Reference Tables.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Section | Server Virtualization | 828 | Key Concepts → "Why server virtualization exists" (10–25% CPU utilization driver, bare-metal servers, containerization NOTE) |
| Paragraph | Virtual machine definition | 828 | Key Concepts → "Virtual machines and hypervisors"; Reference Tables → "VMs vs containers"; Common Pitfalls (physical, not virtual, server) |
| List | Hypervisor types | 829 | Key Concepts → "Virtual machines and hypervisors" (Type 1 / Type 2); Reference Tables → "Hypervisor types"; Common Pitfalls (bare metal vs bare-metal server) |
| Paragraph | Container definition | 830 | Key Concepts → "Containers"; Reference Tables → "VMs vs containers" and "Container engines"; Common Pitfalls (not a lightweight VM) |
| Paragraph | Virtual switch definition | 831 | Key Concepts → "Virtual switching"; Design Baseline rows 1–2; three Common Pitfalls bullets; Troubleshooting steps 4–6 |
| Paragraph | NFV definition | 833 | Key Concepts → "Network Functions Virtualization (NFV)" (ETSI, decoupling, NF definition, the six benefits) |
| Paragraph | OVS-DPDK definition | 839 | Key Concepts → "OVS-DPDK"; Reference Tables → "I/O acceleration technologies compared"; Procedure → "Choosing an I/O acceleration technology" step 3; Design Baseline row 5 |
| Paragraph | PCI passthrough definition | 840 | Key Concepts → "PCI passthrough"; Reference Tables → "I/O acceleration technologies compared"; Design Baseline row 6; Common Pitfalls |
| Paragraph | SR-IOV definition | 841 | Key Concepts → "SR-IOV" (VFs, PFs, VEB, VEPA); Reference Tables → "I/O acceleration technologies compared"; Design Baseline row 7; two Common Pitfalls bullets |
| Paragraph | Enterprise NFV definition | 842 | Key Concepts → "Cisco Enterprise NFV (ENFV)" (the branch problem and the eight benefits) |
| List | Enterprise NFV architecture | 843 | Key Concepts → "Cisco ENFV" → four main components incl. the MSP OSS/BSS NOTE; Reference Tables → "Cisco ENFV solution components"; Common Pitfalls (four components, OSS/BSS optional) |
| Paragraph | Enterprise NFV MANO definition | 843 | Key Concepts → "MANO detail" (DNA Center, network profiles, PnP); Procedure → "Deploying a branch with Cisco ENFV"; Design Baseline row 9; Common Pitfalls (DNA Center, not APIC-EM) |
| Section | Virtual Network Functions and Applications | 845 | Key Concepts → "VNFs and applications" (the eight Cisco VNFs and the thirteen third-party vendors) |
| Section | Network Function Virtualization Infrastructure Software (NFVIS) | 846 | Key Concepts → "NFVIS" (all nine components); Reference Tables → "NFVIS components"; Verification Commands (NFVIS show commands); Common Pitfalls (VIM lives inside NFVIS via ESC-Lite) |

**Coverage note.** **All 14 rows of Table 27-2 map to content in this skill — no gaps, no
partials.** The whole chapter (pp. 826–848) was transcribed, including the quiz and its answer
key. The one honest caveat is **Config Patterns and Verification Commands**: the chapter
contains no CLI whatsoever, so those two sections were written by this skill as the host- and
platform-side equivalents and are **not gear-validated**. The NFVIS commands in particular
should be checked against the NFVIS command reference for your release.

### "Do I Know This Already?" question analysis

Answer key as printed on p. 830: **1** B · **2** D · **3** A, B, D · **4** B · **5** B ·
**6** B · **7** A · **8** B · **9** D · **10** C · **11** B · **12** A.

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| 1 | Whether you know a VM emulates a **physical** server — the guest OS is fooled into thinking it has real hardware | **B** — a software emulation of a **physical** server **with** an OS | Three of the four options say "**virtual** server," which is circular and meaningless, and (c)/(d) toy with "without an operating system." The single discriminating word is **physical** |
| 2 | The actual definition of a container, against the most common industry misconception | **D** — an isolated environment where containerized applications run | **(a) "a lightweight virtual machine" is the trap**, and the book pre-empts it by name on p. 830. (c) "tarball" is close enough to sound right — an image *is* a packaged app with dependencies — but a *container* is the running environment, not the package. Promoted to Common Pitfalls |
| 3 | Whether you can separate **container engines** from **hypervisors** | **A, B, D** — rkt, Docker, LXD | **(c) vSphere hypervisor is a real product in the wrong category** — the highest-value distractor type. Everything on the list virtualizes something; only three of them virtualize *applications* |
| 4 | The precise layer a vSwitch operates at | **B** — a software version of a physical **Layer 2** switch | (a) "multilayer" and (c) "advanced routing capabilities" both over-promote it; **(d) VSS is a real Cisco technology (a switch cluster) that shares an acronym with VMware's vSphere Standard Switch** — and both appear in this chapter. Genuinely nasty |
| 5 | Whether you know multiple vSwitches are allowed per host | **B** — False | Easy to answer True by conflating "**only one vSwitch is supported**" with the real constraint, which is that **traffic cannot flow between vSwitches and they cannot share a pNIC**. The restriction is on the traffic, not the count. Promoted to Common Pitfalls |
| 6 | Whether containers use the same networking substrate as VMs | **B** — False | Sounds plausible because containers are "lighter" and share the host kernel, so you might assume they skip the switch. They do not — they use vSwitches, **called virtual bridges**, and Docker's is `Docker0`. The terminology change is what makes the trap work. Promoted to Common Pitfalls |
| 7 | VNF vs the framework vs the infrastructure vs the OS | **A** — VNF | A four-way acronym-discrimination test with no filler: **NFV** is the framework, **NFVI** the infrastructure, **NFVIS** Cisco's OS. Only **VNF** is the function itself. Q7, Q8, and Q12 test the same four acronyms from three different angles |
| 8 | Same acronym set, asked from the framework side | **B** — NFV | Mirror image of Q7. The discriminator in the stem is "**architectural framework created by ETSI**" — that phrase belongs to NFV and nothing else |
| 9 | The correct term for connecting VNFs into a service | **D** — service chaining | "Daisy chaining" (a) is the trap: it describes the right *shape* with the wrong *name*, and it sounds more natural in English than the term of art. "Bridging," "switching," and "linking" are all real networking words in the wrong slot. Promoted to Common Pitfalls |
| 10 | Whether you can attach VFs and PFs to the right technology | **C** — SR-IOV | **(d) PCI passthrough is the highest-value distractor**, because SR-IOV *is* an enhancement to PCI passthrough and VNFs reach VFs *using* PCI passthrough technology. The line is: VFs and PFs exist **only** in SR-IOV. Promoted to Common Pitfalls |
| 11 | Which Cisco platform is the ENFV orchestrator | **B** — Cisco DNA Center | **All four options are real Cisco platforms**, which is what makes this hard: APIC-EM is the predecessor controller, **APIC Controller is ACI's**, and "Cisco Enterprise Service Automation (ESA)" is plausible-sounding next to real Cisco automation product names. Promoted to Common Pitfalls |
| 12 | The NFVIS definition, essentially verbatim | **A** — True | **Straight recall, no trap** — the stem is a near word-for-word restatement of p. 846. Worth noting only because it confirms Cisco considers the NFVIS definition itself testable |

**Pattern worth noting.** Ten of the twelve questions are **definition-discrimination** rather
than mechanism, and the chapter's whole difficulty is a dense acronym field — **NFV / NFVI /
NFVIS / VNF / VIM / MANO / VF / PF / VEB / VEPA** — where several distractors are **real
technologies from an adjacent slot** (vSphere hypervisor as a container engine, VSS the switch
cluster vs VSS the vSphere Standard Switch, PCI passthrough vs SR-IOV, APIC-EM vs DNA Center).
The defence is a clean mental table of *which acronym names which layer*, not re-reading the
prose. Only Q5 and Q6 reward reasoning from the underlying mechanism, and Q12 is pure recall.
