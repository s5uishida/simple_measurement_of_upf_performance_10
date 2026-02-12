# Simple Measurement of UPF Performance 10
This describes simple performance measurements of several open source UPFs by using the traffic generator [TRex](https://github.com/cisco-system-traffic-generator/trex-core) as the performance measurement tool and [Simple PFCP Client](https://github.com/s5uishida/simple_pfcp_client) as the PFCP simulator.
This was measured on the VMs on Proxmox VE.
For other measurement results, please see [Performance Measurement](https://github.com/s5uishida/sample_config_misc_for_mobile_network#performance_measurement).

**Note. Performance measurement results are highly dependent on the measurement conditions. These results are only examples of results under certain measurement conditions.
And this is a very simple measurement, and according to [this comment](https://github.com/open5gs/open5gs/discussions/1780#discussioncomment-10853290), it doesn't seem to make much sense to measure between VMs. I hope it will serve as a reference for a simple configuration when measuring on real devices.**

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Simple Overview of UPF Performance Measurements](#overview)
- [Changes in configuration files of Simple PFCP Client, TRex and UPFs](#changes)
  - [Changes in configuration files of Simple PFCP Client](#changes_pfcp)
  - [Changes in configuration files of TRex](#changes_trex)
  - [Changes in configuration files of UPFs](#changes_up)
    - [a-1. Changes in configuration files of Open5GS 5GC UPF (TUN)](#changes_up_a1)
    - [a-2. Changes in configuration files of Open5GS 5GC UPF (TAP)](#changes_up_a2)
    - [b. Changes in configuration files of free5GC 5GC UPF](#changes_up_b)
    - [c. Changes in configuration files of UPG-VPP](#changes_up_c)
    - [d. Changes in configuration files of eUPF](#changes_up_d)
    - [e-1. Changes in configuration files of OAI-CN5G-UPF (AF_PACKET)](#changes_up_e1)
    - [e-2. Changes in configuration files of OAI-CN5G-UPF (eBPF/XDP)](#changes_up_e2)
- [Network settings of TRex and UPFs](#network_settings)
  - [Network settings of TRex](#network_settings_trex)
  - [a-1. Network settings of Open5GS 5GC UPF (TUN)](#network_settings_up_a1)
  - [a-2. Network settings of Open5GS 5GC UPF (TAP)](#network_settings_up_a2)
  - [b. Network settings of free5GC 5GC UPF](#network_settings_up_b)
  - [c. Network settings of UPG-VPP](#network_settings_up_c)
  - [d. Network settings of eUPF](#network_settings_up_d)
  - [e-1. Network settings of OAI-CN5G-UPF (AF_PACKET)](#network_settings_up_e1)
  - [e-2. Network settings of OAI-CN5G-UPF (eBPF/XDP)](#network_settings_up_e2)
- [Build Simple PFCP Client, TRex and UPFs](#build)
- [Run Simple PFCP Client, TRex and UPFs](#run)
  - [Run UPFs](#run_up)
    - [a-1. Run Open5GS 5GC UPF (TUN)](#run_up_a1)
    - [a-2. Run Open5GS 5GC UPF (TAP)](#run_up_a2)
    - [b. Run free5GC 5GC UPF](#run_up_b)
    - [c. Run UPG-VPP](#run_up_c)
    - [d. Run eUPF](#run_up_d)
    - [e-1. Run OAI-CN5G-UPF (AF_PACKET)](#run_up_e1)
    - [e-2. Run OAI-CN5G-UPF (eBPF/XDP)](#run_up_e2)
  - [Run Simple PFCP Client](#run_pfcp)
  - [Run TRex](#run_trex)
- [Measure UPF Performance](#measure)
- [Results](#results)
  - [Load measurement](#load_measurement)
  - [Latency measurement](#latency_measurement)
  - [Summary](#summary)
  - [Performance of N6 interface only](#n6_performance)
- [Changelog (summary)](#changelog)

---

<a id="overview"></a>

## Simple Overview of UPF Performance Measurements

I will easily measure the performance of several open source UPFs by using TRex as the traffic generator and Simple PFCP Client as the PFCP simulator.
**Note that this configuration is implemented with Proxmox VE VMs.**

The following minimum configuration was set as a condition.
- One PFCP client, TRex and DUT (UPF)

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=700px></img>

Simple PFCP Client, TRex used are as follows.
- Simple PFCP Client (2026.01.14) - https://github.com/s5uishida/simple_pfcp_client
- TRex v3.08 (2025.11.05) - https://github.com/cisco-system-traffic-generator/trex-core
- Scapy v2.6.1 (2024.11.05) - https://github.com/secdev/scapy

The UPFs used are as follows.
- Open5GS v2.7.6 (2026.01.17) - https://github.com/open5gs/open5gs
- free5GC UPF (go-upf) v1.2.8 (2026.01.05) - https://github.com/free5gc/go-upf  
  gtp5g v0.9.16 (2025.12.02) - https://github.com/free5gc/gtp5g
- UPG-VPP v1.13.0 (2024.03.25) - https://github.com/travelping/upg-vpp
- eUPF v0.7.1 (2025.06.16) - https://github.com/edgecomllc/eupf
- OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-upf

Each VMs are as follows.  
| VM | SW & Role | IP address | OS | CPU | Mem | HDD |
| --- | --- | --- | --- | --- | --- | --- |
| VM1 | Simple PFCP Client | 192.168.0.111/24 | Ubuntu 24.04 | 1 | 1GB | 10GB |
| VM-TG | TRex<br>Traffic Generator | 192.168.0.131/24 | Ubuntu 24.04 | 3 | 8GB | 10GB |
| **VM-DUT** | **each UPF DUT<br>(Device Under Test)** | **192.168.0.151/24** | **Ubuntu 24.04<br>or 22.04** | **2** | **8GB** | **20GB** |

**Each VM-DUT(UPFs) are as follows.**
| # | SW / *packet processing* | Date | Commit | OS |
| --- | --- | --- | --- | --- |
| a | Open5GS UPF v2.7.6<br>***user space*** | 2026.01.17 | `926256b78de9409387ebbb3e05904784dd65e83a` | Ubuntu 24.04 |
| b | free5GC UPF<br>(go-upf) v1.2.8<br>***kernel module*** | 2026.01.05 | `b798fe5ee6a984be492fa53958dd5f1305469f85` | Ubuntu 24.04 |
| c | UPG-VPP v1.13.0<br>***DPDK/VPP*** | 2024.03.25 | `dfdf64000566d35955d7c180720ff66086bd3572` | Ubuntu 22.04 |
| d | eUPF v0.7.1<br>***eBPF/XDP*** | 2025.06.16 | `a8d774a0533ad71ddd59899be26f4aee8a31b5d2` | Ubuntu 24.04 |
| e | OAI-CN5G-UPF v2.2.0<br>***AF_PACKET, eBPF/XDP*** | 2025.12.13 | `e025cdfb3a9c18a228f2efe36bd06b9de998554c` | Ubuntu 24.04 |

The network interfaces of each VM except VM-DUT are as follows.
| VM | Device | Model | Linux Bridge | IP address | Interface |
| --- | --- | --- | --- | --- | --- |
| VM1 | ens18 | VirtIO | vmbr1 | 10.0.0.111/24 | (NAPT NW) |
| | ens19 | VirtIO | mgbr0 | 192.168.0.111/24 | (Mgmt NW) |
| | ens20 | VirtIO | vmbr4 | 192.168.14.111/24 | N4 |
| VM-TG | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~  | ~~10.0.0.131/24~~ | ~~(NAPT NW)~~ ***down*** |
| | ens19 | VirtIO | mgbr0 | 192.168.0.131/24 | (Mgmt NW) |
| | ens20 | VirtIO | vmbr3 | 192.168.13.131/24 | N3 ***(Under DPDK by uio_pci_generic)*** |
| | ens21 | VirtIO | vmbr6 | 192.168.16.152/24 | N6 ***(Under DPDK by uio_pci_generic)*** |

**The network interfaces of each VM-DUT(UPFs) are as follows.**
| # | SW | Device | Model | Linux Bridge | IP address | Interface |
| --- | --- | --- | --- | --- | --- | --- |
| a | Open5GS UPF | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** |
| | | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) |
| | | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 |
| | | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 |
| | | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 |
| b | free5GC UPF<br>(go-upf) | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** |
| | | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) |
| | | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 |
| | | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 |
| | | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 |
| c | UPG-VPP | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** |
| | | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) |
| | | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 ***(Under DPDK by vfio-pci)*** |
| | | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 ***(Under DPDK by vfio-pci)*** |
| | | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 ***(Under DPDK by vfio-pci)*** |
| d | eUPF | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** |
| | | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) |
| | | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 ***(XDP)*** |
| | | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 |
| | | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 ***(XDP)*** |
| e | OAI-CN5G-UPF | ~~ens18~~ | ~~VirtIO~~ | ~~vmbr1~~ | ~~10.0.0.151/24~~ | ~~(NAPT NW)~~ ***down*** |
| | | ens19 | VirtIO | mgbr0 | 192.168.0.151/24 | (Mgmt NW) |
| | | ens20 | VirtIO | vmbr3 | 192.168.13.151/24 | N3 ***(XDP)*** |
| | | ens21 | VirtIO | vmbr4 | 192.168.14.151/24 | N4 |
| | | ens22 | VirtIO | vmbr6 | 192.168.16.151/24 | N6 ***(XDP)*** |

Linux Bridges of Proxmox VE are as follows.
| Linux Bridge | Network CIDR | Interface |
| --- | --- | --- |
| vmbr1 | 10.0.0.0/24 | NAPT NW |
| mgbr0 | 192.168.0.0/24 | Mgmt NW |
| vmbr3 | 192.168.13.0/24 | N3 |
| vmbr4 | 192.168.14.0/24 | N4 |
| vmbr6 | 192.168.16.0/24 | N6 |

UE IP address and TEID are as follows.
| UE IP address | UpLink TEID | DownLink TEID |
| --- | --- | --- |
| 10.45.0.2/24 | 0x00000001 | 0x00000002 |

<a id="changes"></a>

## Changes in configuration files of Simple PFCP Client, TRex and UPFs

Please refer to the following for building Simple PFCP Client, TRex and UPFs respectively.
- Simple PFCP Client (2026.01.14) - https://github.com/s5uishida/simple_pfcp_client
- TRex v3.08 (2026.01.14) - https://github.com/s5uishida/install_trex
- Open5GS v2.7.6 (2026.01.17) - https://open5gs.org/open5gs/docs/guide/02-building-open5gs-from-sources/
- free5GC UPF (go-upf) v1.2.8 (2026.01.05) - https://free5gc.org/guide/
- UPG-VPP v1.13.0 (2024.03.25) - https://github.com/s5uishida/install_vpp_upf_dpdk
- eUPF v0.7.1 (2025.06.16) - https://github.com/s5uishida/install_eupf
- OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://github.com/s5uishida/install_oai_upf

<a id="changes_pfcp"></a>

### Changes in configuration files of Simple PFCP Client

See [here](https://github.com/s5uishida/simple_pfcp_client#set_param) for the original file.

- `/root/pfcp_request.py`  
There is no change.

<a id="changes_trex"></a>

### Changes in configuration files of TRex

See [here](https://github.com/s5uishida/install_trex#config) for the original file.

- `/etc/trex_cfg.yaml`  
There is no change.

See [here](https://github.com/s5uishida/install_trex#load_profile) for the original load profiles.

- `/opt/trex/stl/gtp_1pkt_simple.py` for UpLink  
There is no change.

- `/opt/trex/stl/udp_1pkt_simple.py` for DownLink  
There is no change.

See [here](https://github.com/s5uishida/install_trex#latency_profile) for the original latency profiles.

- `/opt/trex/stl/gtp_latency_1pkt_simple.py` for UpLink  
There is no change.

- `/opt/trex/stl/udp_latency_1pkt_simple.py` for DownLink  
There is no change.

<a id="changes_up"></a>

### Changes in configuration files of UPFs

<a id="changes_up_a1"></a>

#### a-1. Changes in configuration files of Open5GS 5GC UPF (TUN)

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-05-02 19:52:00.000000000 +0900
+++ upf.yaml        2024-05-19 12:38:00.000000000 +0900
@@ -11,18 +11,18 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.14.151
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.13.151
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
+      dev: ogstun
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_up_a2"></a>

#### a-2. Changes in configuration files of Open5GS 5GC UPF (TAP)

- `open5gs/install/etc/open5gs/upf.yaml`
```diff
--- upf.yaml.orig       2024-05-02 19:52:00.000000000 +0900
+++ upf.yaml        2024-09-23 14:00:20.724467385 +0900
@@ -11,18 +11,18 @@
 upf:
   pfcp:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.14.151
     client:
 #      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
 #        - address: 127.0.0.4
   gtpu:
     server:
-      - address: 127.0.0.7
+      - address: 192.168.13.151
   session:
     - subnet: 10.45.0.0/16
       gateway: 10.45.0.1
-    - subnet: 2001:db8:cafe::/48
-      gateway: 2001:db8:cafe::1
+      dnn: internet
+      dev: ogstap
   metrics:
     server:
       - address: 127.0.0.7
```

<a id="changes_up_b"></a>

#### b. Changes in configuration files of free5GC 5GC UPF

- `go-upf/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-10-14 04:53:12.341028732 +0900
+++ upfcfg.yaml 2024-10-14 06:11:36.636303534 +0900
@@ -4,8 +4,8 @@
 # PFCP Configuration
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.14.151   # IP addr for listening
+  nodeID: 192.168.14.151 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -18,7 +18,7 @@
   # If you bind to a specific IP, ensure SMF uses the same IP in its N3 configuration.
   # If you bind to all (0.0.0.0), SMF can use any of the available UPF IPs, but do not use 0.0.0.0 in SMF.
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.13.151
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -28,9 +28,7 @@
 # List of Data Network Names (DNN) supported by this UPF.
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
-  - dnn: internet # Data Network Name
-    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.45.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 # Logging Configuration
```

<a id="changes_up_c"></a>

#### c. Changes in configuration files of UPG-VPP

See [here](https://github.com/s5uishida/install_vpp_upf_dpdk#conf) for the original files.

- `upg-vpp/startup.conf`  
There is no change.

- `upg-vpp/init.conf`  
There is no change.

<a id="changes_up_d"></a>

#### d. Changes in configuration files of eUPF

See [here](https://github.com/s5uishida/install_eupf#create-configuration-file) for the original file.

- `eupf/config.yml`  
There is no change.

<a id="changes_up_e1"></a>

#### e-1. Changes in configuration files of OAI-CN5G-UPF (AF_PACKET)

See [here](https://github.com/s5uishida/install_oai_upf#conf) for the original file.
And change this `config.yaml` to apply [AF_PACKET mode](https://github.com/s5uishida/install_oai_upf#af_conf).
Additionally, to prevent performance degradation, change the log level as follows.
```yaml
log_level:
  general: warning
```
- `openair-upf/config.yaml`

<a id="changes_up_e2"></a>

#### e-2. Changes in configuration files of OAI-CN5G-UPF (eBPF/XDP)

See [here](https://github.com/s5uishida/install_oai_upf#conf) for the original file.

- `openair-upf/config.yaml`  
There is no change.

<a id="network_settings"></a>

## Network settings of TRex and UPFs

<a id="network_settings_trex"></a>

### Network settings of TRex

Set the OS kernel parameter according to [this](https://github.com/s5uishida/install_trex#set_param).

<a id="network_settings_up_a1"></a>

### a-1. Network settings of Open5GS 5GC UPF (TUN)

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```
Then, configure the TUNnel interface.
```
# ip tuntap add name ogstun mode tun
# ip addr add 10.45.0.1/16 dev ogstun
# ip link set ogstun up
```

<a id="network_settings_up_a2"></a>

### a-2. Network settings of Open5GS 5GC UPF (TAP)

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```
Then, configure the TAP interface.
```
# ip tuntap add name ogstap mode tap
# ip addr add 10.45.0.1/16 dev ogstap
# ip link set ogstap up
```

<a id="network_settings_up_b"></a>

### b. Network settings of free5GC 5GC UPF

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```

<a id="network_settings_up_c"></a>

### c. Network settings of UPG-VPP

See [this](https://github.com/s5uishida/install_vpp_upf_dpdk#setup_up).

<a id="network_settings_up_d"></a>

### d. Network settings of eUPF

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```

<a id="network_settings_up_e1"></a>

### e-1. Network settings of OAI-CN5G-UPF (AF_PACKET)

First, see [this](https://github.com/s5uishida/install_oai_upf#network_settings).
Then, down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```

<a id="network_settings_up_e2"></a>

### e-2. Network settings of OAI-CN5G-UPF (eBPF/XDP)

Down the interface `ens18` of the VM-DUT to delete default GW.
```
# ip link set dev ens18 down
```

<a id="build"></a>

## Build Simple PFCP Client, TRex and UPFs

Please refer to the following for building Simple PFCP Client, TRex and UPFs respectively.
- Simple PFCP Client (2026.01.14) - https://github.com/s5uishida/simple_pfcp_client
- TRex v3.08 (2026.01.14) - https://github.com/s5uishida/install_trex
- Open5GS v2.7.6 (2026.01.17) - https://github.com/s5uishida/install_open5gs_upf
- free5GC UPF (go-upf) v1.2.8 (2026.01.05) - https://github.com/s5uishida/install_goupf
- UPG-VPP v1.13.0 (2024.03.25) - https://github.com/s5uishida/install_vpp_upf_dpdk
- eUPF v0.7.1 (2025.06.16) - https://github.com/s5uishida/install_eupf
- OAI-CN5G-UPF v2.2.0 (2025.12.13) - https://github.com/s5uishida/install_oai_upf

<a id="run"></a>

## Run Simple PFCP Client, TRex and UPFs

First run each UPF, then Simple PFCP Client and TRex last.
Each UPF uses the same IP address, so start only the UPF you want to measure.

<a id="run_up"></a>

### Run UPFs

<a id="run_up_a1"></a>

#### a-1. Run Open5GS 5GC UPF (TUN)

Please use the configuration files changed for TUN interface.
```
# cd open5gs
# ./install/bin/open5gs-upfd
```

<a id="run_up_a2"></a>

#### a-2. Run Open5GS 5GC UPF (TAP)

Please use the configuration files changed for TAP interface.
```
# cd open5gs
# ./install/bin/open5gs-upfd
```

<a id="run_up_b"></a>

#### b. Run free5GC 5GC UPF

```
# cd go-upf
# ./upf -c upfcfg.yaml
```

<a id="run_up_c"></a>

#### c. Run UPG-VPP

See [this](https://github.com/s5uishida/install_vpp_upf_dpdk#run).

<a id="run_up_d"></a>

#### d. Run eUPF

See [this](https://github.com/s5uishida/install_eupf#run).

<a id="run_up_e1"></a>

#### e-1. Run OAI-CN5G-UPF (AF_PACKET)

See [this](https://github.com/s5uishida/install_oai_upf#run).

<a id="run_up_e2"></a>

#### e-2. Run OAI-CN5G-UPF (eBPF/XDP)

See [this](https://github.com/s5uishida/install_oai_upf#run).

<a id="run_pfcp"></a>

### Run Simple PFCP Client

See [this](https://github.com/s5uishida/simple_pfcp_client#run).

<a id="run_trex"></a>

### Run TRex

Please refer to [this](https://github.com/s5uishida/install_trex#run) to run TRex.

<a id="measure"></a>

## Measure UPF Performance

Please see below for methods for measuring UpLink and DownLink performances.
- [UpLink measurement](https://github.com/s5uishida/install_trex#ul_measurement)
- [DownLink measurement](https://github.com/s5uishida/install_trex#dl_measurement)

<a id="results"></a>

## Results

In this measurement, the UDP payload size is set to 1400 bytes.

<a id="load_measurement"></a>

### Load measurement

| # | UPF / Date | UpLink<br>Gbps | <br>Kpps | (VM-TG)<br>CPU%[1] | DownLink<br>Gbps | <br>Kpps | (VM-TG)<br>CPU%[1] |
| --- | --- | --- | --- | --- | --- | --- | --- |
| a-1 | Open5GS UPF v2.7.6 (TUN)<br>2026.01.17 | Tx:1.79<br>Rx:1.11 | Tx:150.05<br>Rx:95.96 | 1.82 | Tx:1.74<br>Rx:1.21 | Tx:150.13<br>Rx:101.75 | 1.4 |
| a-2 | Open5GS UPF v2.7.6 (TAP)<br>2026.01.17 | Tx:1.79<br>Rx:1.13 | Tx:149.95<br>Rx:97.65 | 1.35 | Tx:1.73<br>Rx:1.2 | Tx:149.53<br>Rx:100.92 | 1.57 |
| b | free5GC UPF v1.2.8<br>2026.01.05 | Tx:5.92<br>Rx:4.81 | Tx:496.44<br>Rx:416.04 | 6.51 | Tx:5.77<br>Rx:3.92 | Tx:498.45<br>Rx:330.15 | 6.04 |
| c | UPG-VPP v1.13.0<br>2024.03.25 | Tx:10.49<br>Rx:7.77 | Tx:880.07<br>Rx:671.52 | 11.15 | Tx:10.07<br>Rx:7.73 | Tx:870.89<br>Rx:651.87 | 11.37 |
| d | eUPF v0.7.1 (native mode)<br>2025.06.16 | Tx:11.48<br>Rx:9.58 | Tx:963.22<br>Rx:828.42 | 67.53 | Tx:11.08<br>Rx:9.72 | Tx:957.63<br>Rx:815.05 | 65.42 |
| e-1 | OAI-CN5G-UPF v2.2.0<br>(AF_PACKET)<br>2025.12.13 | Tx:2.39<br>Rx:1.32 | Tx:200.46<br>Rx:114.25 | 1.96 | Tx:2.29<br>Rx:1.88 | Tx:197.79<br>Rx:157.95 | 3.6 |
| e-2 | OAI-CN5G-UPF v2.2.0<br>(eBPF/XDP)<br>2025.12.13 | Tx:11.39<br>Rx:9.58 | Tx:955.15<br>Rx:827.88 | 66.71 | Tx:11.25<br>Rx:9.82 | Tx:972.35<br>Rx:823.87 | 65.25 |

1. CPU load - per core of TRex VM (VM-TG). In this case only one core is used.

<details><summary>a-1. logs for Open5GS UPF v2.7.6 (TUN)</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 150kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 1.79 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 1.81 Gbps                      
cpu_util.    : 1.82% @ 1 cores (1 per dual port)          total_rx     : 1.11 Gbps                      
rx_cpu_util. : 0.15% / 95.96 Kpps                         total_pps    : 150.05 Kpps                    
async_util.  : 0% / 17.83 bps                             drop_rate    : 678.53 Mbps                    
total_cps.   : 0 cps                                      queue_full   : 20,153 pkts                    

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |             1.82% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |         1.79 Gbps |          0.02 bps |         1.79 Gbps 
Tx bps L1  |         1.81 Gbps |          0.03 bps |         1.81 Gbps 
Tx pps     |       150.05 Kpps |             0 pps |       150.05 Kpps 
Line Util. |            0.91 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         1.11 Gbps |         1.11 Gbps 
Rx pps     |             0 pps |        95.96 Kpps |        95.96 Kpps 
----       |                   |                   |                   
opackets   |           5732680 |                 2 |           5732682 
ipackets   |                 0 |           3637927 |           3637927 
obytes     |        8541693200 |                92 |        8541693292 
ibytes     |                 0 |        5260438270 |        5260438270 
tx-pkts    |        5.73 Mpkts |            2 pkts |        5.73 Mpkts 
rx-pkts    |            0 pkts |        3.64 Mpkts |        3.64 Mpkts 
tx-bytes   |           8.54 GB |              92 B |           8.54 GB 
rx-bytes   |               0 B |           5.26 GB |           5.26 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 150kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 1.74 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 1.76 Gbps                      
cpu_util.    : 1.4% @ 1 cores (1 per dual port)           total_rx     : 1.21 Gbps                      
rx_cpu_util. : 0.25% / 101.76 Kpps                        total_pps    : 150.13 Kpps                    
async_util.  : 0% / 15.53 bps                             drop_rate    : 530.42 Mbps                    
total_cps.   : 0 cps                                      queue_full   : 27,375 pkts                    

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |              1.4% |                   
--         |                   |                   |                   
Tx bps L2  |          1.41 bps |         1.74 Gbps |         1.74 Gbps 
Tx bps L1  |          2.02 bps |         1.76 Gbps |         1.76 Gbps 
Tx pps     |             0 pps |       150.13 Kpps |       150.13 Kpps 
Line Util. |               0 % |            0.88 % |                   
---        |                   |                   |                   
Rx bps     |         1.21 Gbps |             0 bps |         1.21 Gbps 
Rx pps     |       101.75 Kpps |             0 pps |       101.75 Kpps 
----       |                   |                   |                   
opackets   |           6000003 |           1870234 |           7870237 
ipackets   |           1260211 |           3813208 |           5073419 
obytes     |        8940003026 |        2704355564 |       11644358590 
ibytes     |        1867629858 |        5513894596 |        7381524454 
tx-pkts    |           6 Mpkts |        1.87 Mpkts |        7.87 Mpkts 
rx-pkts    |        1.26 Mpkts |        3.81 Mpkts |        5.07 Mpkts 
tx-bytes   |           8.94 GB |            2.7 GB |          11.64 GB 
rx-bytes   |           1.87 GB |           5.51 GB |           7.38 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  \

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>a-2. logs for Open5GS UPF v2.7.6 (TAP)</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 150kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 1.79 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 1.81 Gbps                      
cpu_util.    : 1.35% @ 1 cores (1 per dual port)          total_rx     : 1.13 Gbps                      
rx_cpu_util. : 0.15% / 97.65 Kpps                         total_pps    : 149.95 Kpps                    
async_util.  : 0% / 4.65 bps                              drop_rate    : 657.79 Mbps                    
total_cps.   : 0 cps                                      queue_full   : 10,721 pkts                    

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |             1.35% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |         1.79 Gbps |           0.7 bps |         1.79 Gbps 
Tx bps L1  |         1.81 Gbps |          1.01 bps |         1.81 Gbps 
Tx pps     |       149.95 Kpps |             0 pps |       149.95 Kpps 
Line Util. |            0.91 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         1.13 Gbps |         1.13 Gbps 
Rx pps     |             0 pps |        97.65 Kpps |        97.65 Kpps 
----       |                   |                   |                   
opackets   |           2142215 |                 1 |           2142216 
ipackets   |                 0 |           1331921 |           1331921 
obytes     |        3191900350 |                46 |        3191900396 
ibytes     |                 0 |        1925956366 |        1925956366 
tx-pkts    |        2.14 Mpkts |            1 pkts |        2.14 Mpkts 
rx-pkts    |            0 pkts |        1.33 Mpkts |        1.33 Mpkts 
tx-bytes   |           3.19 GB |              46 B |           3.19 GB 
rx-bytes   |               0 B |           1.93 GB |           1.93 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 150kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 1.73 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 1.75 Gbps                      
cpu_util.    : 1.57% @ 1 cores (1 per dual port)          total_rx     : 1.2 Gbps                       
rx_cpu_util. : 0.26% / 100.92 Kpps                        total_pps    : 149.53 Kpps                    
async_util.  : 0% / 8.55 bps                              drop_rate    : 533.27 Mbps                    
total_cps.   : 0 cps                                      queue_full   : 39,521 pkts                    

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |             1.57% |                   
--         |                   |                   |                   
Tx bps L2  |          0.04 bps |         1.73 Gbps |         1.73 Gbps 
Tx bps L1  |          0.06 bps |         1.75 Gbps |         1.75 Gbps 
Tx pps     |             0 pps |       149.53 Kpps |       149.53 Kpps 
Line Util. |               0 % |            0.88 % |                   
---        |                   |                   |                   
Rx bps     |          1.2 Gbps |             0 bps |          1.2 Gbps 
Rx pps     |       100.92 Kpps |             0 pps |       100.92 Kpps 
----       |                   |                   |                   
opackets   |           4500002 |           2700880 |           7200882 
ipackets   |           1782930 |           2865306 |           4648236 
obytes     |        6705001536 |        3905471080 |       10610472616 
ibytes     |        2642300824 |        4143231076 |        6785531900 
tx-pkts    |         4.5 Mpkts |         2.7 Mpkts |         7.2 Mpkts 
rx-pkts    |        1.78 Mpkts |        2.87 Mpkts |        4.65 Mpkts 
tx-bytes   |           6.71 GB |           3.91 GB |          10.61 GB 
rx-bytes   |           2.64 GB |           4.14 GB |           6.79 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  -

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>b. logs for free5GC UPF v1.2.8</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 500kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 5.92 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 6 Gbps                         
cpu_util.    : 6.51% @ 1 cores (1 per dual port)          total_rx     : 4.81 Gbps                      
rx_cpu_util. : 0.75% / 416.04 Kpps                        total_pps    : 496.44 Kpps                    
async_util.  : 0% / 7.22 bps                              drop_rate    : 1.1 Gbps                       
total_cps.   : 0 cps                                      queue_full   : 99,175 pkts                    

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |             6.51% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |         5.92 Gbps |             0 bps |         5.92 Gbps 
Tx bps L1  |            6 Gbps |             0 bps |            6 Gbps 
Tx pps     |       496.44 Kpps |             0 pps |       496.44 Kpps 
Line Util. |               3 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         4.81 Gbps |         4.81 Gbps 
Rx pps     |             0 pps |       416.04 Kpps |       416.04 Kpps 
----       |                   |                   |                   
opackets   |          12624256 |                 1 |          12624257 
ipackets   |                 0 |          10450514 |          10450514 
obytes     |       18810141440 |                46 |       18810141486 
ibytes     |                 0 |       15111441844 |       15111441844 
tx-pkts    |       12.62 Mpkts |            1 pkts |       12.62 Mpkts 
rx-pkts    |            0 pkts |       10.45 Mpkts |       10.45 Mpkts 
tx-bytes   |          18.81 GB |              46 B |          18.81 GB 
rx-bytes   |               0 B |          15.11 GB |          15.11 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 500kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 5.77 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 5.85 Gbps                      
cpu_util.    : 6.04% @ 1 cores (1 per dual port)          total_rx     : 3.92 Gbps                      
rx_cpu_util. : 0.41% / 330.15 Kpps                        total_pps    : 498.45 Kpps                    
async_util.  : 0% / 18.46 bps                             drop_rate    : 1.84 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 315,658 pkts                   

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |             6.04% |                   
--         |                   |                   |                   
Tx bps L2  |          1.42 bps |         5.77 Gbps |         5.77 Gbps 
Tx bps L1  |          2.04 bps |         5.85 Gbps |         5.85 Gbps 
Tx pps     |             0 pps |       498.45 Kpps |       498.45 Kpps 
Line Util. |               0 % |            2.92 % |                   
---        |                   |                   |                   
Rx bps     |         3.92 Gbps |             0 bps |         3.92 Gbps 
Rx pps     |       330.15 Kpps |             0 pps |       330.15 Kpps 
----       |                   |                   |                   
opackets   |                 2 |          21720258 |          21720260 
ipackets   |          12731457 |                 0 |          12731457 
obytes     |                92 |       31407493068 |       31407493160 
ibytes     |       18918942222 |                 0 |       18918942222 
tx-pkts    |            2 pkts |       21.72 Mpkts |       21.72 Mpkts 
rx-pkts    |       12.73 Mpkts |            0 pkts |       12.73 Mpkts 
tx-bytes   |              92 B |          31.41 GB |          31.41 GB 
rx-bytes   |          18.92 GB |               0 B |          18.92 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  /

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>c. logs for UPG-VPP v1.13.0</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 900kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 10.49 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 10.63 Gbps                     
cpu_util.    : 11.15% @ 1 cores (1 per dual port)         total_rx     : 7.77 Gbps                      
rx_cpu_util. : 1.14% / 671.52 Kpps                        total_pps    : 880.07 Kpps                    
async_util.  : 0% / 3.54 bps                              drop_rate    : 2.72 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 884,970 pkts                   

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |            11.15% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |        10.49 Gbps |             0 bps |        10.49 Gbps 
Tx bps L1  |        10.63 Gbps |             0 bps |        10.63 Gbps 
Tx pps     |       880.07 Kpps |             0 pps |       880.07 Kpps 
Line Util. |            5.32 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         7.77 Gbps |         7.77 Gbps 
Rx pps     |             0 pps |       671.52 Kpps |       671.52 Kpps 
----       |                   |                   |                   
opackets   |          24225908 |                 0 |          24225908 
ipackets   |                 0 |          16448470 |          16448470 
obytes     |       36096602920 |                 0 |       36096602920 
ibytes     |                 0 |       23784487620 |       23784487620 
tx-pkts    |       24.23 Mpkts |            0 pkts |       24.23 Mpkts 
rx-pkts    |            0 pkts |       16.45 Mpkts |       16.45 Mpkts 
tx-bytes   |           36.1 GB |               0 B |           36.1 GB 
rx-bytes   |               0 B |          23.78 GB |          23.78 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 900kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 10.07 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 10.21 Gbps                     
cpu_util.    : 11.37% @ 1 cores (1 per dual port)         total_rx     : 7.73 Gbps                      
rx_cpu_util. : 1.53% / 651.87 Kpps                        total_pps    : 870.89 Kpps                    
async_util.  : 0% / 7.61 bps                              drop_rate    : 2.35 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 1,434,135 pkts                 

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |            11.37% |                   
--         |                   |                   |                   
Tx bps L2  |             0 bps |        10.07 Gbps |        10.07 Gbps 
Tx bps L1  |             0 bps |        10.21 Gbps |        10.21 Gbps 
Tx pps     |             0 pps |       870.89 Kpps |       870.89 Kpps 
Line Util. |               0 % |            5.11 % |                   
---        |                   |                   |                   
Rx bps     |         7.73 Gbps |             0 bps |         7.73 Gbps 
Rx pps     |       651.87 Kpps |             0 pps |       651.87 Kpps 
----       |                   |                   |                   
opackets   |          27000000 |           9979552 |          36979552 
ipackets   |           7037128 |          18494362 |          25531490 
obytes     |       40230000000 |       14430432192 |       54660432192 
ibytes     |       10429023696 |       26742847452 |       37171871148 
tx-pkts    |          27 Mpkts |        9.98 Mpkts |       36.98 Mpkts 
rx-pkts    |        7.04 Mpkts |       18.49 Mpkts |       25.53 Mpkts 
tx-bytes   |          40.23 GB |          14.43 GB |          54.66 GB 
rx-bytes   |          10.43 GB |          26.74 GB |          37.17 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>d. logs for eUPF v0.7.1</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 1000kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 11.48 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 11.64 Gbps                     
cpu_util.    : 67.53% @ 1 cores (1 per dual port)         total_rx     : 9.58 Gbps                      
rx_cpu_util. : 2.92% / 828.42 Kpps                        total_pps    : 963.22 Kpps                    
async_util.  : 0% / 7.76 bps                              drop_rate    : 1.9 Gbps                       
total_cps.   : 0 cps                                      queue_full   : 10,737,930 pkts                

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |            67.53% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |        11.48 Gbps |             0 bps |        11.48 Gbps 
Tx bps L1  |        11.64 Gbps |             0 bps |        11.64 Gbps 
Tx pps     |       963.22 Kpps |             0 pps |       963.22 Kpps 
Line Util. |            5.82 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         9.58 Gbps |         9.58 Gbps 
Rx pps     |             0 pps |       828.42 Kpps |       828.42 Kpps 
----       |                   |                   |                   
opackets   |          19068097 |                 0 |          19068097 
ipackets   |                 0 |          16052039 |          16052039 
obytes     |       28411464530 |                 0 |       28411464530 
ibytes     |                 0 |       23211248394 |       23211248394 
tx-pkts    |       19.07 Mpkts |            0 pkts |       19.07 Mpkts 
rx-pkts    |            0 pkts |       16.05 Mpkts |       16.05 Mpkts 
tx-bytes   |          28.41 GB |               0 B |          28.41 GB 
rx-bytes   |               0 B |          23.21 GB |          23.21 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  /

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 1000kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 11.08 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 11.23 Gbps                     
cpu_util.    : 65.42% @ 1 cores (1 per dual port)         total_rx     : 9.72 Gbps                      
rx_cpu_util. : 1.92% / 815.05 Kpps                        total_pps    : 957.63 Kpps                    
async_util.  : 0% / 10.39 bps                             drop_rate    : 1.36 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 25,780,840 pkts                

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |            65.42% |                   
--         |                   |                   |                   
Tx bps L2  |          0.01 bps |        11.08 Gbps |        11.08 Gbps 
Tx bps L1  |          0.01 bps |        11.23 Gbps |        11.23 Gbps 
Tx pps     |             0 pps |       957.63 Kpps |       957.63 Kpps 
Line Util. |               0 % |            5.62 % |                   
---        |                   |                   |                   
Rx bps     |         9.72 Gbps |          0.01 bps |         9.72 Gbps 
Rx pps     |       815.05 Kpps |             0 pps |       815.05 Kpps 
----       |                   |                   |                   
opackets   |          30000001 |          15746289 |          45746290 
ipackets   |          13195910 |          25139581 |          38335491 
obytes     |       44700001490 |       22769133894 |       67469135384 
ibytes     |       19661904484 |       36351832754 |       56013737238 
tx-pkts    |          30 Mpkts |       15.75 Mpkts |       45.75 Mpkts 
rx-pkts    |        13.2 Mpkts |       25.14 Mpkts |       38.34 Mpkts 
tx-bytes   |           44.7 GB |          22.77 GB |          67.47 GB 
rx-bytes   |          19.66 GB |          36.35 GB |          56.01 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  |

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>e-1. logs for OAI-CN5G-UPF v2.2.0 (AF_PACKET)</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 200kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 2.39 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 2.42 Gbps                      
cpu_util.    : 1.96% @ 1 cores (1 per dual port)          total_rx     : 1.32 Gbps                      
rx_cpu_util. : 0.1% / 114.25 Kpps                         total_pps    : 200.46 Kpps                    
async_util.  : 0% / 23.52 bps                             drop_rate    : 1.07 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 589,263 pkts                   

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |             1.96% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |         2.39 Gbps |             0 bps |         2.39 Gbps 
Tx bps L1  |         2.42 Gbps |             0 bps |         2.42 Gbps 
Tx pps     |       200.46 Kpps |             0 pps |       200.46 Kpps 
Line Util. |            1.21 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |             0 bps |         1.32 Gbps |         1.32 Gbps 
Rx pps     |             0 pps |       114.25 Kpps |       114.25 Kpps 
----       |                   |                   |                   
opackets   |          34072065 |          24000020 |          58072085 
ipackets   |          18873660 |          24090715 |          42964375 
obytes     |       50767369630 |       34704009320 |       85471378950 
ibytes     |       28121743348 |       34835151546 |       62956894894 
tx-pkts    |       34.07 Mpkts |          24 Mpkts |       58.07 Mpkts 
rx-pkts    |       18.87 Mpkts |       24.09 Mpkts |       42.96 Mpkts 
tx-bytes   |          50.77 GB |           34.7 GB |          85.47 GB 
rx-bytes   |          28.12 GB |          34.84 GB |          62.96 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  \

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 200kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 2.29 Gbps                      
version      : STL @ v3.08                                total_tx_L1  : 2.32 Gbps                      
cpu_util.    : 3.6% @ 1 cores (1 per dual port)           total_rx     : 1.88 Gbps                      
rx_cpu_util. : 0.12% / 157.95 Kpps                        total_pps    : 197.79 Kpps                    
async_util.  : 0% / 26.38 bps                             drop_rate    : 405.24 Mbps                    
total_cps.   : 0 cps                                      queue_full   : 539,413 pkts                   

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |              3.6% |                   
--         |                   |                   |                   
Tx bps L2  |             0 bps |         2.29 Gbps |         2.29 Gbps 
Tx bps L1  |             0 bps |         2.32 Gbps |         2.32 Gbps 
Tx pps     |             0 pps |       197.79 Kpps |       197.79 Kpps 
Line Util. |               0 % |            1.16 % |                   
---        |                   |                   |                   
Rx bps     |         1.88 Gbps |             0 bps |         1.88 Gbps 
Rx pps     |       157.95 Kpps |             0 pps |       157.95 Kpps 
----       |                   |                   |                   
opackets   |          27000030 |          21014475 |          48014505 
ipackets   |          16760891 |          20245339 |          37006230 
obytes     |       40230038924 |       30386912650 |       70616951574 
ibytes     |       24973718982 |       29274739250 |       54248458232 
tx-pkts    |          27 Mpkts |       21.01 Mpkts |       48.01 Mpkts 
rx-pkts    |       16.76 Mpkts |       20.25 Mpkts |       37.01 Mpkts 
tx-bytes   |          40.23 GB |          30.39 GB |          70.62 GB 
rx-bytes   |          24.97 GB |          29.27 GB |          54.25 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  /

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<details><summary>e-2. logs for OAI-CN5G-UPF v2.2.0 (eBPF/XDP)</summary>

**UpLink measurement**
```
start -f stl/gtp_1pkt_simple.py -p 0 -m 1000kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 11.39 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 11.54 Gbps                     
cpu_util.    : 66.71% @ 1 cores (1 per dual port)         total_rx     : 9.58 Gbps                      
rx_cpu_util. : 1.99% / 827.88 Kpps                        total_pps    : 955.15 Kpps                    
async_util.  : 0% / 15.61 bps                             drop_rate    : 1.81 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 5,170,749 pkts                 

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |      TRANSMITTING |              IDLE |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |            66.71% |              0.0% |                   
--         |                   |                   |                   
Tx bps L2  |        11.39 Gbps |             0 bps |        11.39 Gbps 
Tx bps L1  |        11.54 Gbps |             0 bps |        11.54 Gbps 
Tx pps     |       955.15 Kpps |             0 pps |       955.15 Kpps 
Line Util. |            5.77 % |               0 % |                   
---        |                   |                   |                   
Rx bps     |          0.57 bps |         9.58 Gbps |         9.58 Gbps 
Rx pps     |             0 pps |       827.88 Kpps |       827.88 Kpps 
----       |                   |                   |                   
opackets   |           8215450 |                 0 |           8215450 
ipackets   |                 1 |           7102558 |           7102559 
obytes     |       12241020500 |                 0 |       12241020500 
ibytes     |                74 |       10270297496 |       10270297570 
tx-pkts    |        8.22 Mpkts |            0 pkts |        8.22 Mpkts 
rx-pkts    |            1 pkts |         7.1 Mpkts |         7.1 Mpkts 
tx-bytes   |          12.24 GB |               0 B |          12.24 GB 
rx-bytes   |              74 B |          10.27 GB |          10.27 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  /

Press 'ESC' for navigation panel...
status: 

tui>
```

**DownLink measurement**
```
start -f stl/udp_1pkt_simple.py -p 1 -m 1000kpps -d 60
```
```
Global Statistics

connection   : localhost, Port 4501                       total_tx_L2  : 11.25 Gbps                     
version      : STL @ v3.08                                total_tx_L1  : 11.4 Gbps                      
cpu_util.    : 65.25% @ 1 cores (1 per dual port)         total_rx     : 9.82 Gbps                      
rx_cpu_util. : 2.17% / 823.87 Kpps                        total_pps    : 972.35 Kpps                    
async_util.  : 0% / 10.44 bps                             drop_rate    : 1.43 Gbps                      
total_cps.   : 0 cps                                      queue_full   : 60,397,608 pkts                

Port Statistics

   port    |         0         |         1         |       total       
-----------+-------------------+-------------------+------------------
owner      |              root |              root |                   
link       |                UP |                UP |                   
state      |              IDLE |      TRANSMITTING |                   
speed      |          200 Gb/s |          200 Gb/s |                   
CPU util.  |              0.0% |            65.25% |                   
--         |                   |                   |                   
Tx bps L2  |             0 bps |        11.25 Gbps |        11.25 Gbps 
Tx bps L1  |             0 bps |         11.4 Gbps |         11.4 Gbps 
Tx pps     |             0 pps |       972.35 Kpps |       972.35 Kpps 
Line Util. |               0 % |             5.7 % |                   
---        |                   |                   |                   
Rx bps     |         9.82 Gbps |             0 bps |         9.82 Gbps 
Rx pps     |       823.87 Kpps |             0 pps |       823.87 Kpps 
----       |                   |                   |                   
opackets   |          30000001 |          85612755 |         115612756 
ipackets   |          71071564 |          24342334 |          95413898 
obytes     |       44700001490 |      123796043730 |      168496045220 
ibytes     |      105896630360 |       35199014964 |      141095645324 
tx-pkts    |          30 Mpkts |       85.61 Mpkts |      115.61 Mpkts 
rx-pkts    |       71.07 Mpkts |       24.34 Mpkts |       95.41 Mpkts 
tx-bytes   |           44.7 GB |          123.8 GB |          168.5 GB 
rx-bytes   |          105.9 GB |           35.2 GB |          141.1 GB 
-----      |                   |                   |                   
oerrors    |                 0 |                 0 |                 0 
ierrors    |                 0 |                 0 |                 0 

status:  \

Press 'ESC' for navigation panel...
status: 

tui>
```

</details>

<a id="latency_measurement"></a>

### Latency measurement

| # | UPF / Date | UpLink<br>Avg<br>(msec) | <br>Max<br>(msec) | <br>Min<br>(msec) | DownLink<br>Avg<br>(msec) | <br>Max<br>(msec) | <br>Min<br>(msec) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| a-1 | Open5GS UPF v2.7.6 (TUN)<br>2026.01.17 | 0.302 | 0.327 | 0.255 | 0.246 | 0.256 | 0.206 |
| a-2 | Open5GS UPF v2.7.6 (TAP)<br>2026.01.17 | 0.303 | 0.337 | 0.280 | 0.271 | 0.306 | 0.253 |
| b | free5GC UPF v1.2.8<br>2026.01.05 | 0.211 | 0.220 | 0.195 | 0.222 | 0.234 | 0.201 |
| c | UPG-VPP v1.13.0<br>2024.03.25 | 0.152 | 0.185 | 0.091 | 0.153 | 0.177 | 0.141 |
| d | eUPF v0.7.1 (native mode)<br>22025.06.16 | 0.227 | 0.241 | 0.193 | 0.211 | 0.246 | 0.194 |
| e-1 | OAI-CN5G-UPF v2.2.0<br>(AF_PACKET)<br>2025.12.13 | 0.287 | 0.297 | 0.276 | 0.283 | 0.299 | 0.252 |
| e-2 | OAI-CN5G-UPF v2.2.0<br>(eBPF/XDP)<br>2025.12.13 | 0.207 | 0.217 | 0.190 | 0.200 | 0.205 | 0.190 |

<details><summary>a-1. logs for Open5GS UPF v2.7.6 (TUN)</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            327 
Min latency  |            255 
Avg latency  |            302 
-- Window -- |                
Last max     |            290 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             19 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            256 
Min latency  |            206 
Avg latency  |            246 
-- Window -- |                
Last max     |            241 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             20 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>a-2. logs for Open5GS UPF v2.7.6 (TAP)</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            337 
Min latency  |            280 
Avg latency  |            303 
-- Window -- |                
Last max     |            310 
Last-1       |              0 
Last-2       |              0 
Last-3       |              0 
Last-4       |              0 
Last-5       |              0 
Last-6       |              0 
Last-7       |              0 
Last-8       |              0 
Last-9       |              0 
Last-10      |              0 
Last-11      |              0 
Last-12      |              0 
Last-13      |              0 
---          |                
Jitter       |             25 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            306 
Min latency  |            253 
Avg latency  |            271 
-- Window -- |                
Last max     |            256 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             33 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>b. logs for free5GC UPF v1.2.8</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            220 
Min latency  |            195 
Avg latency  |            211 
-- Window -- |                
Last max     |            207 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             35 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            234 
Min latency  |            201 
Avg latency  |            222 
-- Window -- |                
Last max     |            219 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             14 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>c. logs for UPG-VPP v1.13.0</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            185 
Min latency  |             91 
Avg latency  |            152 
-- Window -- |                
Last max     |            170 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             22 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            177 
Min latency  |            141 
Avg latency  |            153 
-- Window -- |                
Last max     |            141 
Last-1       |              0 
Last-2       |              0 
Last-3       |              0 
Last-4       |              0 
Last-5       |              0 
Last-6       |              0 
Last-7       |              0 
Last-8       |              0 
Last-9       |              0 
Last-10      |              0 
Last-11      |              0 
Last-12      |              0 
Last-13      |              0 
---          |                
Jitter       |             13 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>d. logs for eUPF v0.7.1</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            241 
Min latency  |            193 
Avg latency  |            227 
-- Window -- |                
Last max     |            230 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             16 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            246 
Min latency  |            194 
Avg latency  |            211 
-- Window -- |                
Last max     |            201 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             16 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>e-1. logs for OAI-CN5G-UPF v2.2.0 (AF_PACKET)</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            297 
Min latency  |            276 
Avg latency  |            287 
-- Window -- |                
Last max     |            289 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             22 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            299 
Min latency  |            252 
Avg latency  |            283 
-- Window -- |                
Last max     |            292 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             20 
----         |                
Errors       |              0 

trex>
```

</details>

<details><summary>e-2. logs for OAI-CN5G-UPF v2.2.0 (eBPF/XDP)</summary>

**UpLink measurement**
```
start -f stl/gtp_latency_1pkt_simple.py -p 0 -d 10
```
```
Latency Statistics

   PG ID     |       0        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            217 
Min latency  |            190 
Avg latency  |            207 
-- Window -- |                
Last max     |            206 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             11 
----         |                
Errors       |              0 

trex>
```

**DownLink measurement**
```
start -f stl/udp_latency_1pkt_simple.py -p 1 -d 10
```
```
Latency Statistics

   PG ID     |       1        
-------------+---------------
TX pkts      |             11 
RX pkts      |             11 
Max latency  |            205 
Min latency  |            190 
Avg latency  |            200 
-- Window -- |                
Last max     |            199 
Last-1       |                
Last-2       |                
Last-3       |                
Last-4       |                
Last-5       |                
Last-6       |                
Last-7       |                
Last-8       |                
Last-9       |                
Last-10      |                
Last-11      |                
Last-12      |                
Last-13      |                
---          |                
Jitter       |             11 
----         |                
Errors       |              0 

trex>
```

</details>

<a id="summary"></a>

### Summary

These measurement results show that eBPF/XDP-based eUPF and OAI-CN5G-UPF have relatively outstanding performance even on Proxmox VE VM.
Although I was unable to confirm any performance differences between eUPF and OAI-CN5G-UPF through this simple measurement, I speculate that performance differences can be confirmed through other measurement methods and analyses.
In general, the measurement environment, conditions and tools have a significant impact on the measurement results.

If measuring using virtual machines, it would be better to measure on VMs on a hypervisor such as Proxmox VE.
Also, it is good to select VirtIO as the network interface to ensure that the network does not become a bottleneck in the measurement.

It is very simple mesurement and may not be very meaningful when measuring between virtual machines, but it may be a little helpful when comparing the relative performance of UPF. I would appreciate it if you could use this as a reference as a configuration example when measuring with real devices.

<a id="n6_performance"></a>

### Performance of N6 interface only

I simply measured the raw communication performance between VM-TG and VM-DUT.
This is a measurement of the N6 interface and therefore does not include communication over GTP-U.

| A--B | TCP[1]<br>throughput | UDP[2]<br>throughput | UDP[2]<br>packet loss | RTT[3]<br>(msec) |
| --- | --- | --- | --- | --- |
| VM-TG --(N6)-- VM-DUT | S:26.8 Gbps<br>R:26.8 Gbps | S:2.04 Gbps<br>R:2.03 Gbps | 0.4 % | 0.262 |

<details><summary>1. iperf3 -c 192.168.16.151</summary>

```
# iperf3 -c 192.168.16.151
Connecting to host 192.168.16.151, port 5201
[  5] local 192.168.16.152 port 58568 connected to 192.168.16.151 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  3.00 GBytes  25.7 Gbits/sec    0   1.87 MBytes       
[  5]   1.00-2.00   sec  3.08 GBytes  26.4 Gbits/sec    0   2.30 MBytes       
[  5]   2.00-3.00   sec  3.10 GBytes  26.7 Gbits/sec    0   2.30 MBytes       
[  5]   3.00-4.00   sec  3.11 GBytes  26.7 Gbits/sec    0   2.43 MBytes       
[  5]   4.00-5.00   sec  3.18 GBytes  27.3 Gbits/sec    0   2.84 MBytes       
[  5]   5.00-6.00   sec  3.21 GBytes  27.5 Gbits/sec    0   2.84 MBytes       
[  5]   6.00-7.00   sec  3.20 GBytes  27.5 Gbits/sec    0   2.84 MBytes       
[  5]   7.00-8.00   sec  3.14 GBytes  27.0 Gbits/sec    0   3.62 MBytes       
[  5]   8.00-9.00   sec  3.14 GBytes  27.0 Gbits/sec    0   3.62 MBytes       
[  5]   9.00-10.00  sec  3.07 GBytes  26.3 Gbits/sec    0   3.62 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  31.2 GBytes  26.8 Gbits/sec    0             sender
[  5]   0.00-9.99   sec  31.2 GBytes  26.8 Gbits/sec                  receiver

iperf Done.
```

</details>

<details><summary>2. iperf3 -c 192.168.16.151 -u -b 5G</summary>

```
# iperf3 -c 192.168.16.151 -u -b 5G
Connecting to host 192.168.16.151, port 5201
[  5] local 192.168.16.152 port 52639 connected to 192.168.16.151 port 5201
[ ID] Interval           Transfer     Bitrate         Total Datagrams
[  5]   0.00-1.00   sec   242 MBytes  2.03 Gbits/sec  175079  
[  5]   1.00-2.00   sec   240 MBytes  2.01 Gbits/sec  173824  
[  5]   2.00-3.00   sec   230 MBytes  1.93 Gbits/sec  166912  
[  5]   3.00-4.00   sec   246 MBytes  2.07 Gbits/sec  178413  
[  5]   4.00-5.00   sec   241 MBytes  2.02 Gbits/sec  174618  
[  5]   5.00-6.00   sec   246 MBytes  2.06 Gbits/sec  177980  
[  5]   6.00-7.00   sec   247 MBytes  2.07 Gbits/sec  178878  
[  5]   7.00-8.00   sec   246 MBytes  2.07 Gbits/sec  178375  
[  5]   8.00-9.00   sec   247 MBytes  2.07 Gbits/sec  178931  
[  5]   9.00-10.00  sec   248 MBytes  2.08 Gbits/sec  179748  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-10.00  sec  2.38 GBytes  2.04 Gbits/sec  0.000 ms  0/1762758 (0%)  sender
[  5]   0.00-10.00  sec  2.37 GBytes  2.03 Gbits/sec  0.005 ms  7076/1762758 (0.4%)  receiver

iperf Done.
```

</details>

<details><summary>3. ping 192.168.16.151 -c 10</summary>

```
# ping 192.168.16.151 -c 10
PING 192.168.16.151 (192.168.16.151) 56(84) bytes of data.
64 bytes from 192.168.16.151: icmp_seq=1 ttl=64 time=0.285 ms
64 bytes from 192.168.16.151: icmp_seq=2 ttl=64 time=0.292 ms
64 bytes from 192.168.16.151: icmp_seq=3 ttl=64 time=0.282 ms
64 bytes from 192.168.16.151: icmp_seq=4 ttl=64 time=0.221 ms
64 bytes from 192.168.16.151: icmp_seq=5 ttl=64 time=0.319 ms
64 bytes from 192.168.16.151: icmp_seq=6 ttl=64 time=0.219 ms
64 bytes from 192.168.16.151: icmp_seq=7 ttl=64 time=0.260 ms
64 bytes from 192.168.16.151: icmp_seq=8 ttl=64 time=0.232 ms
64 bytes from 192.168.16.151: icmp_seq=9 ttl=64 time=0.261 ms
64 bytes from 192.168.16.151: icmp_seq=10 ttl=64 time=0.258 ms

--- 192.168.16.151 ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9215ms
rtt min/avg/max/mdev = 0.219/0.262/0.319/0.030 ms
```

</details>

---

I would like to thank all the excellent developers and contributors who developed these great systems and tools.

<a id="changelog"></a>

## Changelog (summary)

- [2026.02.11] Added measurement of OAI-CN5G-UPF(AF_PACKET).
- [2026.01.23] Measured again using OAI-CN5G-UPF(eBPF/XDP) built on Ubuntu 24.04.
- [2026.01.18] Initial release. This measurement is an update of the following measurement, adding the measurement of OAI-CN5G-UPF(eBPF/XDP).  
  [Simple Measurement of UPF Performance 9](https://github.com/s5uishida/simple_measurement_of_upf_performance_9)
