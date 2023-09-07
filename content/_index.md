
+++

title = "Infrastructures for the Edge-Cloud Continuum on a Small Scale: a Practical Case Study"
description = "Infrastructures for the Edge-Cloud Continuum on a Small Scale: a Practical Case Study"
outputs = ["Reveal"]
aliases = [
    "/guide/"
]

+++

# Infrastructures for the
# *Edge-Cloud Continuum*
# on a **Small Scale**:
# a Practical Case Study

### [Martina Baiardi](https://www.unibo.it/sitoweb/m.baiardi/en), [Giovanni Ciatto](https://www.unibo.it/sitoweb/giovanni.ciatto/en), [*Danilo Pianini*](https://www.unibo.it/sitoweb/danilo.pianini/en)

---

{{< slide background-image="https://github.com/DanySK/slides-2023-asmecc/assets/1991673/d73158e1-faf0-4681-94a0-115590bce133" >}}

---

* 8th gen Intel Core i5, 8/16GB RAM, 1TB storage

![](https://github.com/DanySK/slides-2023-asmecc/assets/1991673/d73158e1-faf0-4681-94a0-115590bce133)

---

## Multiple sources of inefficiency

* Hardware cannot be used for **scopes** other than those for which it was *funded*
  * Sounds reasonable, until you discover hundreds of unused systems left in a warehouse
    * Bought for *teaching*, already *replaced* with more capable machinery
* Groups **owning** hardware resources *hardly share*
  * Several *compute servers* dramatically *underutilized*
  * Not for lack of will!
    * How do I *control access* to my hardware?
    * How *expensive (in time and money)* is it to share?
    * No *standardized procedure* to do so nimbly!
* Hundreds of *decent lab PCs* dramatically *underutilized*
  * Most get turned on in the morning and never shut down
  * Most are *idle* most of the time
  * They get turned off when the campus closes, but they could potentially still be useful...

---

The real issue with systems like this one (arguably, small scale) is

## *RESOURCE COMPARTMENTALIZATION*

* Systems are compartmentalized by *ownership*
  * Different research groups
  * Campus vs. department
* Systems are compartmentalized by *scope*
  * Teaching vs. research
* Systems are compartmentalized by *type*
  * Rack vs. desktop
  * Operating system (<i class="fa-brands fa-linux"></i> vs. <i class="fa-brands fa-windows"></i> vs. <i class="fa-brands fa-apple"></i>)


---

![image](https://github.com/DanySK/slides-2023-asmecc/assets/1991673/906e14ef-e213-4dee-bc56-6d399a993567)

(AI-generated)

---

## Tear down the wall:
## small-scale edge-cloud continuum

### **Desiderata**

* *Runtime addition and removal* of hardware resources
* *Heterogeneous* hardware resources
* (Relative) ease of use: people with *no system-level knowledge* should be able to use it
* Capability to host *long-lived* services and execute *short-lived* jobs
* Integration with the *existing authentication and authorization* infrastructure

---

## Problem model

#### Roles

*student*, *researcher*, *IT Personnel*
* They differ by the kinds of resources that should be able to access
* Different contexts may require different roles, but the right management should be similar<br>
(unless the scale grows signficantly)

#### Workloads

*jobs* (short-lived), *services* (long-lived)

#### Capabilities and constraints

workloads must be able to *constrain* their execution to nodes with specific *capabilities*, e.g.:<br>
GPU or FPGA (hardware), specific OS, specific runtime (e.g., the JVM, or the CLR)

#### Isolation

Different workloads $\to$ different constraints $\to$ isolation at multiple levels

#### access control

RBAC backed by a pre-existing central authority

---

## Architecture

![architecture](architecture.png)

---

## Process

1. **IaaS** infrastructure selection
    1. Consider ease of *configuration*
    1. Compute the *cost*
    1. If needed, verify that the hardware *passthrough* is actually viable
    1. Pick an *administration UI*
1. Container **orchestrator** selection
    1. Verify the installation and *configuration effort*
    1. Evaluate the handyness of *task description languages*
    1. If needed, verify that node-*tagging* mechanisms are in place to support peculiar hardware
    1. Find a *UI* for the orchestrator
1. Shared **Storage** configuration
1. **Access control**

---

## Application
### The [Alma Mater Research Institute for Human Centered Artificial Intelligence](https://centri.unibo.it/alma-ai/it)
### Just **AlmaAI** for friends

#### Context

* Multiple projects and research groups can
* Hardware can be joined in or reserved for other uses at any time

#### Hardware

* Several compute servers (heavy-load CPU and a lot of memory)
* Several GPU-equipped servers (NVIDIA Tesla A100)
* 2 "inference servers" with custom NPU (Atlas 800-model 3010)

#### Constraints

* The authentication infrastructure must use the current UniBo Active Directory
* Re-training of the current IT personnel to be avoided
    * Replacement of personnel that would need re-train is not an option either
* One-command join and leave of hardware resources

---

## 1. **IaaS** infrastructure selection

| | Bare Metal | VMWare VSphere | OpenStack |
| --- | --- | --- | --- |
| *configuration* | {{< tilde >}} low flexibility | {{< tick >}} | {{< cross >}} Non portable across hardware/OSs |
| *license cost* | {{< tick >}} | {{< tilde >}} we had licenses | {{< tick >}} |
| *training cost* | {{< tick >}} | {{< tilde >}} IT personnel pre-trained | {{< cross >}} |
| *passthrough* | {{< tick >}} | {{< tilde >}} NPU not working | {{< tilde >}} NPU not working |
| *admin UI* | {{< cross >}} | {{< tick >}} | {{< cross >}}  |

**Decision**: Bare Metal for servers with NPU, VMWare Sphere otherwise

#### *Driving factors*
* Licenses already available due to a previous agreement with VMWare, pre-trained IT personnel
* Need for a standard installation procedure
* NPU support (favoring bare metal for inference servers)

#### *Risk analysis*
* Vendor lock-in with VMWare.
* Changes in the licensing model of VMWare Sphere may make the VMM not viable.

---

## 2. Container **orchestrator** selection

| | Kubernetes | <i class="fa-brands fa-docker"></i> Docker Swarm |
| --- | --- | --- |
| *setup* | {{< tilde >}} `kubeadm`, `kubelet`, `kubectl`, `kube-proxy` | {{< tick >}} |
| *language* | {{< tick >}} YAML | {{< tick >}} YAML |
| *tagging* | {{< tick >}} | {{< tick >}} |
| *admin UI* | {{< tilde >}} Third party | {{< tilde >}} Third party |

**Decision**: <i class="fa-brands fa-docker"></i> Docker Swarm

#### *Driving factors*
* Easier to setup
* One-command join to the swarm

#### *Risk analysis*
* Less flexible than Kubernetes, could be an issue if the scale grows

---

## 2 bis. Container **orchestrator** admin UI

| | Swarmpit | Kubernetes Dashboard | Portainer |
| --- | --- | --- | --- |
| *license cost* | {{< tick >}} | {{< tick >}} | {{< tilde >}} Freemium |
| *support for Kubernetes* | {{< cross >}} | {{< tick >}} | {{< tick >}} |
| *support for Docker Swarm* | {{< tick >}} | {{< cross >}} | {{< tick >}} |
| *LDAP authentication* | {{< cross >}} [Feature requested](https://github.com/swarmpit/swarmpit/issues/329) | {{< tilde >}} Needs to bind Kubernetes to LDAP | {{< tick >}} |

**Decision**: Portainer

#### *Driving factors*
* LDAP working out of the box with the free version
* Supports both Kubernetes and Docker Swarm, in case a switch will be needed in the future

#### *Risk analysis*
* Changes in the free feature set may make the free version not viable

---

## 3. Shared **storage**

{{% multicol %}}

{{% col %}}
#### **Federation**

Unification of multiple storage resources
* *distributed* file system
  * GlusterFS, HadoopFS, CEPH
{{% /col %}}

{{% col %}}
#### **Distribution**

Access remote file systems as if they were local
* *remote* file system
  * NFS, Samba, CIFS, iSCSI
{{% /col %}}

{{% /multicol %}}
**Decision**: NFS + GlusterFS

#### *Driving factors*
* Storage servers' resources must be distributed (NFS)
* Smaller storage resources must get unified (GlusterFS)

#### *Risk analysis*
* Disruption in a single node may make data (temporarily) disappear
  * *Mitigation*: sharding or replication, at the price of worse latency
* Container volumes may expose the underlying file system, breaking isolation (Docker Swarm only)
  * *Mitigation*: a dedicated plugin for Docker is being built
* Poor backup support (both Docker Swarm and Kubernetes)
  * *Mitigation*: a dedicated plugin for Docker is being built

---

## 4. **Access Control**

Authentication mechanisms already in place
* Common situation

* Available protocols: LDAP, Active Directory, Shibboleth, OAuth

**Decision**: Small-scale ECC hid behind a VPN + LDAP

#### *Driving factors*
* It is the only protocol supported by Portainer in the free version

#### *Risk analysis*
* VPN-ification may not be viable in some contexts

---

## Conclusion

* The ECC paradigm is *viable even at a small scale*
* ECC in the small can help with *underutilized hardware* resources
* Technical challenges are mixed with *organizational* ones
* Factors to take into account include *hidden costs*
  * Adopting a FOSS solution may be ideal, but the IT personnel must be *trained*
    * Also people may be reluctant to adopt something they don't know
  * Proprietary solutions are at a risk of *vendor lock-in*

### In this work

* A possibile architecture for a small-scale ECC
* An actionable decision taking process to build it
* An example of the infrastructure being built

### Future work

* Docker plugins for backup and file system isolation
* Incorporation of wake-on-lan for on-demand usage of unused lab desktops

---

**and, by the way, the ACSOS 2023 Telegram bot is actually running on the infrastructure we just described**
