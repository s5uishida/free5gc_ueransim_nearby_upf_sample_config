# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - Select nearby UPF according to the connected gNodeB
This describes a very simple configuration that uses free5GC and UERANSIM to select the nearby UPF according to the connected gNodeB.

---

<h2 id="toc">Table of Contents</h2>

- [Overview of free5GC 5GC Simulation Mobile Network](#overview)
- [Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN](#changes)
  - [Changes in configuration files of free5GC 5GC C-Plane](#changes_cp)
  - [Changes in configuration files of free5GC 5GC U-Plane1](#changes_up1)
  - [Changes in configuration files of free5GC 5GC U-Plane2](#changes_up2)
  - [Changes in configuration files of UERANSIM UE / RAN](#changes_ueransim)
    - [Changes in configuration files of RAN (gNodeB1)](#changes_ran1)
    - [Changes in configuration files of RAN (gNodeB2)](#changes_ran2)
    - [Changes in configuration files of UE for Loc1 (IMSI-001010000000000)](#changes_ue_loc1)
    - [Changes in configuration files of UE for Loc2 (IMSI-001010000000000)](#changes_ue_loc2)
- [Network settings of free5GC 5GC and UERANSIM UE / RAN](#network_settings)
  - [Network settings of free5GC 5GC C-Plane](#network_settings_cp)
  - [Network settings of free5GC 5GC U-Plane1](#network_settings_up1)
  - [Network settings of free5GC 5GC U-Plane2](#network_settings_up2)
- [Build free5GC and UERANSIM](#build)
- [Run free5GC 5GC and UERANSIM UE / RAN](#run)
  - [Run free5GC 5GC U-Plane1 & U-Plane2](#run_up)
  - [Run free5GC 5GC C-Plane](#run_cp)
  - [Run UERANSIM (gNodeBs)](#run_ran)
    - [Start gNodeB1 with TAC=1 in Loc1](#run_ran1)
    - [Start gNodeB2 with TAC=2 in Loc2](#run_ran2)
  - [Run UERANSIM (UE in Loc1)](#run_ue1)
    - [Start UE connected to gNodeB1 in Loc1](#con_ue1)
    - [Ping google.com going through DN=60.60.0.0/16 on Loc1](#ping_ue1)
  - [Run UERANSIM (UE in Loc2)](#run_ue2)
    - [Start UE connected to gNodeB2 in Loc2](#con_ue2)
    - [Ping google.com going through DN=60.61.0.0/16 on Loc2](#ping_ue2)
- [Changelog (summary)](#changelog)

---
<h2 id="overview">Overview of free5GC 5GC Simulation Mobile Network</h2>

The following minimum configuration was set as a condition.
- The pair of gNodeB and UPF exists in the same location.
- The UE connected to gNodeB connects to DN managed by UPF in the same location.

**In this example, TAC is matched to connect gNodeB and AMF, and AMF searches for SMF using `preferred target NF location` as one of the parameters.**

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v3.0.5 + updated version at 2021.08.29 - https://github.com/free5gc/free5gc
- UE / RAN - UERANSIM v3.2.3 - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | free5GC  5GC C-Plane | 192.168.0.141/24 <br> 192.168.0.142/24 <br> 192.168.0.143/24 | Ubuntu 20.04 | 2GB | 20GB |
| VM2 | free5GC  5GC U-Plane1  | 192.168.0.144/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM3 | free5GC  5GC U-Plane2  | 192.168.0.145/24 | Ubuntu 20.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB1) | 192.168.0.131/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM5 | UERANSIM RAN (gNodeB2) | 192.168.0.132/24 | Ubuntu 20.04 | 1GB | 10GB |
| VM6 | UERANSIM UE | 192.168.0.133/24 | Ubuntu 20.04 | 1GB | 10GB |

AMF & SMF addresses are as follows.  
| NF | IP address | IP address on SBI | Supported TACs | preferred target NF location |
| --- | --- | --- | --- | --- |
| AMF1 | 192.168.0.142 | 127.0.0.18 | 1 | loc1 |
| SMF1 | 192.168.0.142 | 127.0.0.2 | -- | loc1 |
| AMF2 | 192.168.0.143 | 127.0.0.28 | 2 | loc2 |
| SMF2 | 192.168.0.143 | 127.0.0.12 | -- | loc2 |

gNodeB Information (other information is default) is as follows.  
| gNodeB # | Location # | TAC # | IP address |
| --- | --- | --- | --- |
| gNodeB1 | Loc1 | 1 | 192.168.0.131 |
| gNodeB2 | Loc2 | 2 | 192.168.0.132 |

Subscriber Information (other information is default) is as follows.  
**Note. Please select OP or OPc according to the setting of UERANSIM UE configuration files.**
| UE | IMSI | DNN | OP/OPc | gNodeB # |
| --- | --- | --- | --- | --- |
| UE | 001010000000000 | internet | OPc | gNodeB1 in Loc1 <br> gNodeB2 in Loc2|

I registered these information with the free5GC WebUI.

**Note. Please follow the procedure below to install WebConsole. There is a bug in WebConsole v1.0.0, so please checkout to v1.0.1 (or later) before building.**

https://github.com/free5gc/free5gc/wiki/Installation#d-install-webconsole

**In addition, according to [here](https://github.com/free5gc/free5gc/wiki/New-Subscriber-via-webconsole#7-modify-the-existed-subscriber), there is a bug that the function to change the subscriber information registered in WebConsole does not work properly. Therefore, when you change the subscriber information, please delete it and then register again.**

**--> Then, according to [this PR](https://github.com/free5gc/webconsole/pull/9), the WebConsole bug was fixed at 2021.08.03.**

In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | Location # |  TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- | --- |
| 60.60.0.0/16 | Loc1 | upfgtp | internet | uesimtun0 | U-Plane1 |
| 60.61.0.0/16 | Loc2 | upfgtp | internet | uesimtun0 | U-Plane2 |

<h2 id="changes">Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN</h2>

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.0.5 + updated version at 2021.08.29 - https://github.com/free5gc/free5gc/wiki/Installation
- UERANSIM v3.2.3 - https://github.com/aligungr/UERANSIM/wiki/Installation

**Note. This article is written in updated version of 2021.08.29, so please follow [this procedure](https://github.com/s5uishida/free5gc_ueransim_nearby_upf_sample_config#build-free5gc-and-ueransim) to change to the latest `main` branch before building free5GC.**

<h3 id="changes_cp">Changes in configuration files of free5GC 5GC C-Plane</h3>

- `free5gc/config/amfcfg1.yaml`
```diff
--- amfcfg.yaml.orig    2021-08-09 13:22:28.111343037 +0000
+++ amfcfg1.yaml        2021-08-09 15:49:40.520730141 +0000
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.1
+    - 192.168.0.142
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
     registerIPv4: 127.0.0.18 # IP used to register to NRF
@@ -20,23 +20,21 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
   supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       tac: 1 # Tracking Area Code (uinteger, range: 0~16777215)
   plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
   supportDnnList:  # the DNN (Data Network Name) list supported by this AMF
     - internet
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
@@ -50,7 +48,7 @@
   networkName:  # the name of this core network
     full: free5GC
     short: free
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF and UPFs are located
   networkFeatureSupport5GS: # 5gs Network Feature Support IE, refer to TS 24.501
     enable: true # append this IE in Registration accept or not
     imsVoPS: 0 # IMS voice over PS session indicator (uinteger, range: 0~1)
```
- `free5gc/config/amfcfg2.yaml`
```diff
--- amfcfg.yaml.orig    2021-08-09 13:22:28.111343037 +0000
+++ amfcfg2.yaml        2021-08-09 15:49:54.320756213 +0000
@@ -5,11 +5,11 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.1
+    - 192.168.0.143
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.18 # IP used to register to NRF
-    bindingIPv4: 127.0.0.18  # IP used to bind the service
+    registerIPv4: 127.0.0.28 # IP used to register to NRF
+    bindingIPv4: 127.0.0.28  # IP used to bind the service
     port: 8000 # port used to bind the service
   serviceNameList: # the SBI services provided by this AMF, refer to TS 29.518
     - namf-comm # Namf_Communication service
@@ -20,23 +20,21 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
   supportTaiList:  # the TAI (Tracking Area Identifier) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-      tac: 1 # Tracking Area Code (uinteger, range: 0~16777215)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+      tac: 2 # Tracking Area Code (uinteger, range: 0~16777215)
   plmnSupportList: # the PLMNs (Public land mobile network) list supported by this AMF
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       snssaiList: # the S-NSSAI (Single Network Slice Selection Assistance Information) list supported by this AMF
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-        - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-          sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
   supportDnnList:  # the DNN (Data Network Name) list supported by this AMF
     - internet
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
@@ -50,7 +48,7 @@
   networkName:  # the name of this core network
     full: free5GC
     short: free
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF and UPFs are located
   networkFeatureSupport5GS: # 5gs Network Feature Support IE, refer to TS 24.501
     enable: true # append this IE in Registration accept or not
     imsVoPS: 0 # IMS voice over PS session indicator (uinteger, range: 0~1)
```
- `free5gc/config/smfcfg1.yaml`
```diff
--- smfcfg.yaml.orig    2021-08-09 13:22:28.113342966 +0000
+++ smfcfg1.yaml        2021-08-09 15:50:16.258809418 +0000
@@ -32,17 +32,17 @@
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.142
   userplane_information: # list of userplane information
     up_nodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        node_id: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        node_id: 192.168.0.144 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -51,23 +51,17 @@
               - dnn: internet
                 pools:
                   - cidr: 60.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 60.61.0.0/16
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.144
             networkInstance: internet # Data Network Name (DNN)
     links: # the topology graph of userplane, A and B represent the two nodes of each link
       - A: gNB1
         B: UPF
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF and UPFs are located
+  ulcl: false
 
 # the kind of log output
   # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/smfcfg2.yaml`
```diff
--- smfcfg.yaml.orig    2021-08-09 13:22:28.113342966 +0000
+++ smfcfg2.yaml        2021-08-09 15:50:37.608873139 +0000
@@ -6,8 +6,8 @@
   smfName: SMF # the name of this SMF
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.2 # IP used to register to NRF
-    bindingIPv4: 127.0.0.2  # IP used to bind the service
+    registerIPv4: 127.0.0.12 # IP used to register to NRF
+    bindingIPv4: 127.0.0.12  # IP used to bind the service
     port: 8000 # Port used to bind the service
     tls: # the local path of TLS key
       key: free5gc/support/TLS/smf.key # SMF TLS Certificate
@@ -32,17 +32,17 @@
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.143
   userplane_information: # list of userplane information
     up_nodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        node_id: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        node_id: 192.168.0.145 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -50,24 +50,18 @@
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
-                  - cidr: 60.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
                   - cidr: 60.61.0.0/16
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.145
             networkInstance: internet # Data Network Name (DNN)
     links: # the topology graph of userplane, A and B represent the two nodes of each link
       - A: gNB1
         B: UPF
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF and UPFs are located
+  ulcl: false
 
 # the kind of log output
   # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2021-08-09 13:22:28.111343037 +0000
+++ ausfcfg.yaml        2021-08-09 14:35:05.044653272 +0000
@@ -12,8 +12,8 @@
     - nausf-auth # Nausf_UEAuthentication service
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
     - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
       mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2021-08-09 13:22:28.113342966 +0000
+++ nrfcfg.yaml 2021-08-09 14:35:45.724132931 +0000
@@ -11,8 +11,8 @@
     bindingIPv4: 127.0.0.10  # IP used to bind the service
     port: 8000 # port used to bind the service
   DefaultPlmnId:
-    mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-    mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+    mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   serviceNameList: # the SBI services provided by this NRF, refer to TS 29.510
     - nnrf-nfm # Nnrf_NFManagement service
     - nnrf-disc # Nnrf_NFDiscovery service
```
- `free5gc/config/nssfcfg.yaml`
```diff
--- nssfcfg.yaml.orig   2021-08-09 13:22:28.113342966 +0000
+++ nssfcfg.yaml        2021-08-09 14:36:28.022586165 +0000
@@ -14,12 +14,12 @@
     - nnssf-nssaiavailability # Nnssf_NSSAIAvailability service
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   supportedPlmnList: # the PLMNs (Public land mobile network) list supported by this NSSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   supportedNssaiInPlmnList: # Supported S-NSSAI List for each PLMN
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
       supportedSnssaiList: # Supported S-NSSAIs of the PLMN
         - sst: 1 # Slice/Service Type (uinteger, range: 0~255)
           sd: 010203 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
```

<h3 id="changes_up1">Changes in configuration files of free5GC 5GC U-Plane1</h3>

- `free5gc/NFs/upf/build/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2021-08-09 13:46:27.893044456 +0000
+++ upfcfg.yaml 2021-08-09 14:58:26.429886652 +0000
@@ -11,12 +11,12 @@
 
   # The IP list of the N4 interface on this UPF (Can't set to 0.0.0.0)
   pfcp:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.144
 
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   gtpu:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.144
     # [optional] gtpu.name
     # - name: upf.5gc.nctu.me
     # [optional] gtpu.ifname
@@ -25,6 +25,6 @@
   # The DNN list supported by UPF
   dnn_list:
     - dnn: internet # Data Network Name
-      cidr: 60.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+      cidr: 60.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
       # [optional] dnn_list[*].natifname
       # natifname: eth0
```

<h3 id="changes_up2">Changes in configuration files of free5GC 5GC U-Plane2</h3>

- `free5gc/NFs/upf/build/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2021-08-09 13:49:26.627940862 +0000
+++ upfcfg.yaml 2021-08-09 15:00:07.959774257 +0000
@@ -11,12 +11,12 @@
 
   # The IP list of the N4 interface on this UPF (Can't set to 0.0.0.0)
   pfcp:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.145
 
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   gtpu:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.145
     # [optional] gtpu.name
     # - name: upf.5gc.nctu.me
     # [optional] gtpu.ifname
@@ -25,6 +25,6 @@
   # The DNN list supported by UPF
   dnn_list:
     - dnn: internet # Data Network Name
-      cidr: 60.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+      cidr: 60.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
       # [optional] dnn_list[*].natifname
       # natifname: eth0
```

<h3 id="changes_ueransim">Changes in configuration files of UERANSIM UE / RAN</h3>

<h4 id="changes_ran1">Changes in configuration files of RAN (gNodeB1)</h4>

- `UERANSIM/config/free5gc-gnb.yaml`
```diff
--- free5gc-gnb.yaml.orig       2021-02-11 11:03:28.000000000 +0000
+++ free5gc-gnb.yaml    2021-08-10 00:28:30.000000000 +0000
@@ -1,17 +1,17 @@
-mcc: '208'          # Mobile Country Code value
-mnc: '93'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
 tac: 1              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.131   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.131   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.131    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.1
+  - address: 192.168.0.142
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<h4 id="changes_ran2">Changes in configuration files of RAN (gNodeB2)</h4>

- `UERANSIM/config/free5gc-gnb.yaml`
```diff
--- free5gc-gnb.yaml.orig       2021-02-11 11:03:28.000000000 +0000
+++ free5gc-gnb.yaml    2021-08-10 00:30:54.000000000 +0000
@@ -1,17 +1,17 @@
-mcc: '208'          # Mobile Country Code value
-mnc: '93'           # Mobile Network Code value (2 or 3 digits)
+mcc: '001'          # Mobile Country Code value
+mnc: '01'           # Mobile Network Code value (2 or 3 digits)
 
 nci: '0x000000010'  # NR Cell Identity (36-bit)
 idLength: 32        # NR gNB ID length in bits [22...32]
-tac: 1              # Tracking Area Code
+tac: 2              # Tracking Area Code
 
-linkIp: 127.0.0.1   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
-ngapIp: 127.0.0.1   # gNB's local IP address for N2 Interface (Usually same with local IP)
-gtpIp: 127.0.0.1    # gNB's local IP address for N3 Interface (Usually same with local IP)
+linkIp: 192.168.0.132   # gNB's local IP address for Radio Link Simulation (Usually same with local IP)
+ngapIp: 192.168.0.132   # gNB's local IP address for N2 Interface (Usually same with local IP)
+gtpIp: 192.168.0.132    # gNB's local IP address for N3 Interface (Usually same with local IP)
 
 # List of AMF address information
 amfConfigs:
-  - address: 127.0.0.1
+  - address: 192.168.0.143
     port: 38412
 
 # List of supported S-NSSAIs by this gNB
```

<h4 id="changes_ue_loc1">Changes in configuration files of UE for Loc1 (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-loc1.yaml`
```diff
--- free5gc-ue.yaml.orig        2021-08-15 14:16:46.000000000 +0000
+++ free5gc-ue-loc1.yaml        2021-08-17 06:42:53.673883361 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000003'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 
 # Permanent subscription key
 key: '8baf473f2f8fd09487cccbd7097c6862'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.131
 
 # UAC Access Identities Configuration
 uacAic:
```

<h4 id="changes_ue_loc2">Changes in configuration files of UE for Loc2 (IMSI-001010000000000)</h4>

- `UERANSIM/config/free5gc-ue-loc2.yaml`
```diff
--- free5gc-ue.yaml.orig        2021-08-15 14:16:46.000000000 +0000
+++ free5gc-ue-loc2.yaml        2021-08-17 06:43:10.441964717 +0000
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000003'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 
 # Permanent subscription key
 key: '8baf473f2f8fd09487cccbd7097c6862'
@@ -20,7 +20,7 @@
 
 # List of gNB IP addresses for Radio Link Simulation
 gnbSearchList:
-  - 127.0.0.1
+  - 192.168.0.132
 
 # UAC Access Identities Configuration
 uacAic:
```

<h2 id="network_settings">Network settings of free5GC 5GC and UERANSIM UE / RAN</h2>

<h3 id="network_settings_cp">Network settings of free5GC 5GC C-Plane</h3>

Add IP addresses for (AMF1 & SMF1) and (AMF2 & SMF2).
```
ip addr add 192.168.0.142/24 dev enp0s8
ip addr add 192.168.0.143/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<h3 id="network_settings_up1">Network settings of free5GC 5GC U-Plane1</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 60.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<h3 id="network_settings_up2">Network settings of free5GC 5GC U-Plane2</h3>

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 60.61.0.0/16 ! -o upfgtp -j MASQUERADE
```

<h2 id="build">Build free5GC and UERANSIM</h2>

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.0.5 + updated version at 2021.08.29 - https://github.com/free5gc/free5gc/wiki/Installation
- UERANSIM v3.2.3 - https://github.com/aligungr/UERANSIM/wiki/Installation

Note. Install MongoDB with package manager on free5GC 5GC C-Plane machine.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.
```
# apt update
# apt install mongodb
# systemctl start mongodb
# systemctl enable mongodb
```
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.

**Note. Please follow the procedure below to install WebConsole. There is a bug in WebConsole v1.0.0, so please checkout to v1.0.1 (or later) before building.**

https://github.com/free5gc/free5gc/wiki/Installation#d-install-webconsole

**Note. This article is written in updated version of 2021.08.29, so please run the following script to checkout all NFs and WebConsole of free5GC to the latest `main` branch before building.**
```
#!/usr/bin/env bash

NF_LIST="nrf amf smf udr pcf udm nssf ausf upf n3iwf"

for NF in ${NF_LIST}; do
    cd NFs/${NF}
    git checkout main
    cd ../..
done

cd webconsole
git checkout main
```

<h2 id="run">Run free5GC 5GC and UERANSIM UE / RAN</h2>

First run the 5GC, then UERANSIM (UE & RAN implementation).

<h3 id="run_up">Run free5GC 5GC U-Plane1 & U-Plane2</h3>

First, run free5GC 5GC U-Planes. Please see [here](https://github.com/free5gc/free5gc/issues/170#issuecomment-773214169) for the reason.

- free5GC 5GC U-Plane1
```
# cd free5gc/NFs/upf/build
# bin/free5gc-upfd
```
- free5GC 5GC U-Plane2
```
# cd free5gc/NFs/upf/build
# bin/free5gc-upfd
```
Then run `tcpdump` on one more terminal for each U-Plane.
- Run `tcpdump` on VM2 (U-Plane1)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), capture size 262144 bytes
```
- Run `tcpdump` on VM3 (U-Plane2)
```
# tcpdump -i upfgtp -n
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on upfgtp, link-type RAW (Raw IP), capture size 262144 bytes
```

<h3 id="run_cp">Run free5GC 5GC C-Plane</h3>

Next, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```
#!/usr/bin/env bash

PID_LIST=()

NF_LIST1="amf smf"
NF_LIST2="udr pcf udm nssf ausf"

export GIN_MODE=release

./bin/nrf &
PID_LIST+=($!)
sleep 1

for NF in ${NF_LIST1}; do
    ./bin/${NF} -${NF}cfg config/${NF}cfg1.yaml &
    PID_LIST+=($!)
    sleep 1
    ./bin/${NF} -${NF}cfg config/${NF}cfg2.yaml &
    PID_LIST+=($!)
    sleep 1
done

for NF in ${NF_LIST2}; do
    ./bin/${NF} &
    PID_LIST+=($!)
    sleep 1
done

function terminate()
{
    sudo kill -SIGTERM ${PID_LIST[${#PID_LIST[@]}-2]} ${PID_LIST[${#PID_LIST[@]}-1]}
    sleep 2
}

trap terminate SIGINT
wait ${PID_LIST}
```

<h3 id="run_ran">Run UERANSIM (gNodeBs)</h3>

Run each gNodeB with TAC=1 and TAC=2 in two locations.  
Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<h4 id="run_ran1">Start gNodeB1 with TAC=1 in Loc1</h4>

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.3
[2021-08-30 13:13:06.348] [sctp] [info] Trying to establish SCTP connection... (192.168.0.142:38412)
[2021-08-30 13:13:06.400] [sctp] [info] SCTP connection established (192.168.0.142:38412)
[2021-08-30 13:13:06.400] [sctp] [debug] SCTP association setup ascId[3]
[2021-08-30 13:13:06.400] [ngap] [debug] Sending NG Setup Request
[2021-08-30 13:13:06.522] [ngap] [debug] NG Setup Response received
[2021-08-30 13:13:06.522] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2021-08-30T13:13:06Z [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.131:53364
2021-08-30T13:13:06Z [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.131:53364
2021-08-30T13:13:06Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle NG Setup request
2021-08-30T13:13:06Z [INFO][AMF][NGAP][192.168.0.131:53364] Send NG-Setup response
```

<h4 id="run_ran2">Start gNodeB2 with TAC=2 in Loc2</h4>

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.3
[2021-08-30 13:14:17.774] [sctp] [info] Trying to establish SCTP connection... (192.168.0.143:38412)
[2021-08-30 13:14:17.822] [sctp] [info] SCTP connection established (192.168.0.143:38412)
[2021-08-30 13:14:17.822] [sctp] [debug] SCTP association setup ascId[3]
[2021-08-30 13:14:17.822] [ngap] [debug] Sending NG Setup Request
[2021-08-30 13:14:17.875] [ngap] [debug] NG Setup Response received
[2021-08-30 13:14:17.875] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2021-08-30T13:14:17Z [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.132:60322
2021-08-30T13:14:17Z [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.132:60322
2021-08-30T13:14:17Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle NG Setup request
2021-08-30T13:14:17Z [INFO][AMF][NGAP][192.168.0.132:60322] Send NG-Setup response
```

<h3 id="run_ue1">Run UERANSIM (UE in Loc1)</h3>

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<h4 id="con_ue1">Start UE connected to gNodeB1 in Loc1</h4>

```
# ./nr-ue -c ../config/free5gc-ue-loc1.yaml 
UERANSIM v3.2.3
[2021-08-30 13:15:29.654] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2021-08-30 13:15:29.654] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2021-08-30 13:15:29.655] [nas] [info] Selected plmn[001/01]
[2021-08-30 13:15:29.655] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2021-08-30 13:15:29.656] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2021-08-30 13:15:29.656] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2021-08-30 13:15:29.656] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2021-08-30 13:15:29.657] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-08-30 13:15:29.657] [nas] [debug] Sending Initial Registration
[2021-08-30 13:15:29.657] [rrc] [debug] Sending RRC Setup Request
[2021-08-30 13:15:29.658] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2021-08-30 13:15:29.658] [rrc] [info] RRC connection established
[2021-08-30 13:15:29.659] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2021-08-30 13:15:29.659] [nas] [info] UE switches to state [CM-CONNECTED]
[2021-08-30 13:15:29.986] [nas] [debug] Authentication Request received
[2021-08-30 13:15:30.079] [nas] [debug] Security Mode Command received
[2021-08-30 13:15:30.079] [nas] [debug] Selected integrity[2] ciphering[0]
[2021-08-30 13:15:30.279] [nas] [debug] Registration accept received
[2021-08-30 13:15:30.280] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2021-08-30 13:15:30.280] [nas] [debug] Sending Registration Complete
[2021-08-30 13:15:30.280] [nas] [info] Initial Registration is successful
[2021-08-30 13:15:30.281] [nas] [debug] Sending PDU Session Establishment Request
[2021-08-30 13:15:30.281] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-08-30 13:15:30.755] [nas] [debug] PDU Session Establishment Accept received
[2021-08-30 13:15:30.761] [nas] [info] PDU Session establishment is successful PSI[1]
[2021-08-30 13:15:31.319] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 60.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2021-08-30T13:15:29Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Initial UE Message
2021-08-30T13:15:29Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2021-08-30T13:15:29Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2021-08-30T13:15:29Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:29Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2021-08-30T13:15:29Z [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2021-08-30T13:15:29Z [INFO][AUSF][UeAuthPost] Serving network authorized
2021-08-30T13:15:29Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:29Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2021-08-30T13:15:29Z [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2021-08-30T13:15:29Z [INFO][LIB][3GPP] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2021-08-30T13:15:29Z [INFO][LIB][3GPP] scheme 0
2021-08-30T13:15:29Z [INFO][LIB][3GPP] SUPI type is IMSI
http://127.0.0.10:8000
2021-08-30T13:15:29Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:29Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2021-08-30T13:15:29Z [INFO][UDR][DRepo] Handle QueryAuthSubsData
2021-08-30T13:15:29Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2021-08-30T13:15:29Z [ERRO][UDM][UEAU] opStr length is  0
2021/08/30 13:15:29 http2: server connection error from 127.0.0.1:39112: connection error: PROTOCOL_ERROR
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle ModifyAuthentication
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
AUTN = ac146f61609080009954ba6598a78924
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2021-08-30T13:15:30Z [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2021-08-30T13:15:30Z [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2021-08-30T13:15:30Z [INFO][AUSF][5gAkaAuth] XresStar = 6130366465336337633838363065303937616662666364643232386633633564
2021-08-30T13:15:30Z [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Uplink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2021/08/30 13:15:30 http2: server connection error from 127.0.0.1:49974: connection error: PROTOCOL_ERROR
2021-08-30T13:15:30Z [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2021-08-30T13:15:30Z [INFO][AUSF][5gAkaAuth] res*: 6130366465336337633838363065303937616662666364643232386633633564
Xres*: 6130366465336337633838363065303937616662666364643232386633633564
2021-08-30T13:15:30Z [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2021/08/30 13:15:30 http2: server connection error from 127.0.0.1:42640: connection error: PROTOCOL_ERROR
2021-08-30T13:15:30Z [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2021/08/30 13:15:30 http2: server connection error from 127.0.0.1:39124: connection error: PROTOCOL_ERROR
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2021-08-30T13:15:30Z [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Uplink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle GetNssai
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle QueryAmData
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=00101 |
2021-08-30T13:15:30Z [ERRO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai: [{ServingSnssai:0xc0002b3600 HomeSnssai:<nil>}]
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2021-08-30T13:15:30Z [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2021-08-30T13:15:30Z [INFO][UDM][UECM] UEID: imsi-001010000000000
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle GetAmData
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle QueryAmData
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=00101 |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=00101 |
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle GetSmfSelectData
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle QuerySmfSelectData
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=00101 |
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle GetUeContextInSmfData
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle QuerySmfRegList
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle Subscribe
2021/08/30 13:15:30 http2: server connection error from 127.0.0.1:39148: connection error: PROTOCOL_ERROR
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2021-08-30T13:15:30Z [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2021-08-30T13:15:30Z [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2021-08-30T13:15:30Z [INFO][AMF][Comm] new AMF Status Subscription[1]
2021-08-30T13:15:30Z [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2021-08-30T13:15:30Z [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Initial Context Setup Response
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Uplink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle Uplink Nas Transport
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2021-08-30T13:15:30Z [INFO][NSSF][NsSelect] Handle NSSelectionGet
2021-08-30T13:15:30Z [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=4e55600f-472a-4b9e-8db1-7738f16db023&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2021-08-30T13:15:30Z [INFO][SMF][PduSess] Recieve Create SM Context Request
2021-08-30T13:15:30Z [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2021-08-30T13:15:30Z [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2021-08-30T13:15:30Z [INFO][SMF][CTX] Allocated UE IP address: 60.60.0.1
2021-08-30T13:15:30Z [INFO][SMF][CTX] Selected UPF: UPF
2021-08-30T13:15:30Z [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[60.60.0.1]
2021-08-30T13:15:30Z [INFO][UDM][SDM] Handle GetSmData
2021-08-30T13:15:30Z [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle QuerySmData
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2021-08-30T13:15:30Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=00101&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2021-08-30T13:15:30Z [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2021-08-30T13:15:30Z [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc000397040 0xc000397080]
2021-08-30T13:15:30Z [INFO][SMF][GSM] Protocol Configuration Options
2021-08-30T13:15:30Z [INFO][SMF][GSM] &{[0xc000397040 0xc000397080]}
2021-08-30T13:15:30Z [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2021-08-30T13:15:30Z [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=PCF |
2021-08-30T13:15:30Z [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2021-08-30T13:15:30Z [INFO][UDR][DRepo] Handle PolicyDataUesUeIdSmDataGet
2021-08-30T13:15:30Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2021-08-30T13:15:30Z [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2021-08-30T13:15:30Z [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2021-08-30T13:15:30Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:15:30Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=4e55600f-472a-4b9e-8db1-7738f16db023&target-nf-type=AMF |
2021-08-30T13:15:30Z [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2021-08-30T13:15:30Z [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2021-08-30T13:15:30Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2021-08-30T13:15:30Z [INFO][SMF][PFCP] In HandlePfcpSessionEstablishmentResponse
&{200 Mbps 100 Mbps}
2021-08-30T13:15:30Z [INFO][LIB][PFCP] Remove Request Transaction [2]
2021-08-30T13:15:30Z [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2021-08-30T13:15:30Z [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2021-08-30T13:15:30Z [INFO][AMF][NGAP][192.168.0.131:53364] Handle PDU Session Resource Setup Response
2021-08-30T13:15:30Z [INFO][SMF][PduSess] Recieve Update SM Context Request
2021-08-30T13:15:30Z [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2021-08-30T13:15:30Z [INFO][SMF][PFCP] In HandlePfcpSessionModificationResponse
2021-08-30T13:15:30Z [INFO][SMF][PduSess] [SMF] PFCP Modification Resonse Accept
2021-08-30T13:15:30Z [INFO][SMF][PFCP] PFCP Session Modification Success[1]
2021-08-30T13:15:30Z [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:91d16146-30ee-4278-83cb-376bb02b4681/modify |
2021-08-30T13:15:30Z [INFO][LIB][PFCP] Remove Request Transaction [3]
```
The free5GC U-Plane1 log when executed is as follows.
```
2021-08-30T13:15:30Z [INFO][UPF][Util] [PFCP] Handle PFCP session establishment request
2021-08-30T13:15:30Z [INFO][UPF][Util] [PFCP] Session Establishment Response
2021-08-30T13:15:30Z [INFO][UPF][Util] [PFCP] Handle PFCP session modification request
2021-08-30T13:15:30Z [INFO][UPF][Util] [PFCP] Session Modification Response
```
The TUNnel interface `uesimtun0` is created as follows.
```
...
4: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 60.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::8ff:3f63:3203:cfe4/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue1">Ping google.com going through DN=60.60.0.0/16 on Loc1</h4>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.161.206) from 60.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.161.206: icmp_seq=1 ttl=61 time=45.7 ms
64 bytes from 172.217.161.206: icmp_seq=2 ttl=61 time=44.5 ms
64 bytes from 172.217.161.206: icmp_seq=3 ttl=61 time=35.8 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
13:19:44.809572 IP 60.60.0.1 > 172.217.161.206: ICMP echo request, id 1, seq 1, length 64
13:19:44.852556 IP 172.217.161.206 > 60.60.0.1: ICMP echo reply, id 1, seq 1, length 64
13:19:45.810390 IP 60.60.0.1 > 172.217.161.206: ICMP echo request, id 1, seq 2, length 64
13:19:45.852472 IP 172.217.161.206 > 60.60.0.1: ICMP echo reply, id 1, seq 2, length 64
13:19:46.811776 IP 60.60.0.1 > 172.217.161.206: ICMP echo request, id 1, seq 3, length 64
13:19:46.845468 IP 172.217.161.206 > 60.60.0.1: ICMP echo reply, id 1, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 in the same Loc1 according to the connected gNodeB1 in Loc1.**

<h3 id="run_ue2">Run UERANSIM (UE in Loc2)</h3>

Then the UE disconnects from gNodeB1 and connects to gNodeB2 in Loc2.

<h4 id="con_ue2">Start UE connected to gNodeB2 in Loc2</h4>

```
# ./nr-ue -c ../config/free5gc-ue-loc2.yaml 
UERANSIM v3.2.3
[2021-08-30 13:21:04.729] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2021-08-30 13:21:04.730] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2021-08-30 13:21:04.730] [nas] [info] Selected plmn[001/01]
[2021-08-30 13:21:04.730] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2021-08-30 13:21:04.731] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2021-08-30 13:21:04.731] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2021-08-30 13:21:04.731] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2021-08-30 13:21:04.732] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-08-30 13:21:04.732] [nas] [debug] Sending Initial Registration
[2021-08-30 13:21:04.733] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2021-08-30 13:21:04.733] [rrc] [debug] Sending RRC Setup Request
[2021-08-30 13:21:04.734] [rrc] [info] RRC connection established
[2021-08-30 13:21:04.734] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2021-08-30 13:21:04.735] [nas] [info] UE switches to state [CM-CONNECTED]
[2021-08-30 13:21:04.798] [nas] [debug] Authentication Request received
[2021-08-30 13:21:04.843] [nas] [debug] Security Mode Command received
[2021-08-30 13:21:04.843] [nas] [debug] Selected integrity[2] ciphering[0]
[2021-08-30 13:21:04.907] [nas] [debug] Registration accept received
[2021-08-30 13:21:04.907] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2021-08-30 13:21:04.907] [nas] [debug] Sending Registration Complete
[2021-08-30 13:21:04.908] [nas] [info] Initial Registration is successful
[2021-08-30 13:21:04.908] [nas] [debug] Sending PDU Session Establishment Request
[2021-08-30 13:21:04.908] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-08-30 13:21:05.177] [nas] [debug] PDU Session Establishment Accept received
[2021-08-30 13:21:05.181] [nas] [info] PDU Session establishment is successful PSI[1]
[2021-08-30 13:21:05.198] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 60.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Initial UE Message
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2021-08-30T13:21:04Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:04Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2021-08-30T13:21:04Z [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2021-08-30T13:21:04Z [INFO][AUSF][UeAuthPost] Serving network authorized
2021-08-30T13:21:04Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:04Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2021-08-30T13:21:04Z [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2021-08-30T13:21:04Z [INFO][LIB][3GPP] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2021-08-30T13:21:04Z [INFO][LIB][3GPP] scheme 0
2021-08-30T13:21:04Z [INFO][LIB][3GPP] SUPI type is IMSI
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle QueryAuthSubsData
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2021-08-30T13:21:04Z [ERRO][UDM][UEAU] opStr length is  0
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle ModifyAuthentication
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
AUTN = c69a85006e62800026798e02af935380
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2021-08-30T13:21:04Z [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2021-08-30T13:21:04Z [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2021-08-30T13:21:04Z [INFO][AUSF][5gAkaAuth] XresStar = 6162393335353338326666346139326461306137393534393935316263663066
2021-08-30T13:21:04Z [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Uplink Nas Transport
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2021-08-30T13:21:04Z [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2021-08-30T13:21:04Z [INFO][AUSF][5gAkaAuth] res*: 6162393335353338326666346139326461306137393534393935316263663066
Xres*: 6162393335353338326666346139326461306137393534393935316263663066
2021-08-30T13:21:04Z [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2021/08/30 13:21:04 http2: server connection error from 127.0.0.1:42724: connection error: PROTOCOL_ERROR
2021-08-30T13:21:04Z [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2021-08-30T13:21:04Z [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Uplink Nas Transport
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2021-08-30T13:21:04Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:04Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2021-08-30T13:21:04Z [INFO][UDM][SDM] Handle GetNssai
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle QueryAmData
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=00101 |
2021-08-30T13:21:04Z [ERRO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai: [{ServingSnssai:0xc00035c2e0 HomeSnssai:<nil>}]
2021-08-30T13:21:04Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:04Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2021-08-30T13:21:04Z [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2021-08-30T13:21:04Z [INFO][UDM][UECM] UEID: imsi-001010000000000
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2021-08-30T13:21:04Z [ERRO][UDM][HTTP] unsupported scheme[]
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 204 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2021-08-30T13:21:04Z [INFO][UDM][SDM] Handle GetAmData
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle QueryAmData
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=00101 |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=00101 |
2021-08-30T13:21:04Z [INFO][UDM][SDM] Handle GetSmfSelectData
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle QuerySmfSelectData
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=00101 |
2021-08-30T13:21:04Z [INFO][UDM][SDM] Handle GetUeContextInSmfData
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle QuerySmfRegList
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2021-08-30T13:21:04Z [INFO][UDM][SDM] Handle Subscribe
2021/08/30 13:21:04 http2: server connection error from 127.0.0.1:39226: connection error: PROTOCOL_ERROR
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2021-08-30T13:21:04Z [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2021-08-30T13:21:04Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:04Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2021-08-30T13:21:04Z [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2021-08-30T13:21:04Z [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2021-08-30T13:21:04Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2021-08-30T13:21:04Z [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2021-08-30T13:21:04Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2021-08-30T13:21:04Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Initial Context Setup Response
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Uplink Nas Transport
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:21:05Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle Uplink Nas Transport
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2021-08-30T13:21:05Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2021-08-30T13:21:05Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2021-08-30T13:21:05Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2021-08-30T13:21:05Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:05Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2021-08-30T13:21:05Z [INFO][NSSF][NsSelect] Handle NSSelectionGet
2021-08-30T13:21:05Z [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=52bfa26e-30a4-42ff-9a2a-be7707cf393c&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2021-08-30T13:21:05Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:05Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc2&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2021-08-30T13:21:05Z [INFO][SMF][PduSess] Recieve Create SM Context Request
2021-08-30T13:21:05Z [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2021-08-30T13:21:05Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:05Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2021-08-30T13:21:05Z [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2021-08-30T13:21:05Z [INFO][SMF][CTX] Allocated UE IP address: 60.61.0.1
2021-08-30T13:21:05Z [INFO][SMF][CTX] Selected UPF: UPF
2021-08-30T13:21:05Z [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[60.61.0.1]
2021-08-30T13:21:05Z [INFO][UDM][SDM] Handle GetSmData
2021-08-30T13:21:05Z [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2021-08-30T13:21:05Z [INFO][UDR][DRepo] Handle QuerySmData
2021-08-30T13:21:05Z [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2021-08-30T13:21:05Z [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=00101&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2021-08-30T13:21:05Z [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2021-08-30T13:21:05Z [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0003a7080 0xc0003a70c0]
2021-08-30T13:21:05Z [INFO][SMF][GSM] Protocol Configuration Options
2021-08-30T13:21:05Z [INFO][SMF][GSM] &{[0xc0003a7080 0xc0003a70c0]}
2021-08-30T13:21:05Z [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2021-08-30T13:21:05Z [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2021-08-30T13:21:05Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:05Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=PCF |
2021-08-30T13:21:05Z [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2021-08-30T13:21:05Z [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2021-08-30T13:21:05Z [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2021-08-30T13:21:05Z [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2021-08-30T13:21:05Z [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=52bfa26e-30a4-42ff-9a2a-be7707cf393c&target-nf-type=AMF |
2021-08-30T13:21:05Z [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2021-08-30T13:21:05Z [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2021-08-30T13:21:05Z [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2021-08-30T13:21:05Z [INFO][SMF][PFCP] In HandlePfcpSessionEstablishmentResponse
&{200 Mbps 100 Mbps}
2021-08-30T13:21:05Z [INFO][LIB][PFCP] Remove Request Transaction [2]
2021-08-30T13:21:05Z [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2021-08-30T13:21:05Z [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2021-08-30T13:21:05Z [INFO][AMF][NGAP][192.168.0.132:60322] Handle PDU Session Resource Setup Response
2021-08-30T13:21:05Z [INFO][SMF][PduSess] Recieve Update SM Context Request
2021-08-30T13:21:05Z [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2021-08-30T13:21:05Z [INFO][SMF][PFCP] In HandlePfcpSessionModificationResponse
2021-08-30T13:21:05Z [INFO][SMF][PduSess] [SMF] PFCP Modification Resonse Accept
2021-08-30T13:21:05Z [INFO][SMF][PFCP] PFCP Session Modification Success[1]
2021-08-30T13:21:05Z [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:8304f7e8-1b66-4922-b720-67ffa6216304/modify |
2021-08-30T13:21:05Z [INFO][LIB][PFCP] Remove Request Transaction [3]
```
The free5GC U-Plane2 log when executed is as follows.
```
2021-08-30T13:21:05Z [INFO][UPF][Util] [PFCP] Handle PFCP session establishment request
2021-08-30T13:21:05Z [INFO][UPF][Util] [PFCP] Session Establishment Response
2021-08-30T13:21:05Z [INFO][UPF][Util] [PFCP] Handle PFCP session modification request
2021-08-30T13:21:05Z [INFO][UPF][Util] [PFCP] Session Modification Response
```
The TUNnel interface `uesimtun0` is created as follows.
```
...
5: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 60.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::5f64:aaa7:57b2:8920/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<h4 id="ping_ue2">Ping google.com going through DN=60.61.0.0/16 on Loc2</h4>

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.250.76.142) from 60.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.250.76.142: icmp_seq=1 ttl=61 time=11.9 ms
64 bytes from 142.250.76.142: icmp_seq=2 ttl=61 time=31.4 ms
64 bytes from 142.250.76.142: icmp_seq=3 ttl=61 time=35.6 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
13:23:24.980218 IP 60.61.0.1 > 142.250.76.142: ICMP echo request, id 2, seq 1, length 64
13:23:24.989465 IP 142.250.76.142 > 60.61.0.1: ICMP echo reply, id 2, seq 1, length 64
13:23:25.980338 IP 60.61.0.1 > 142.250.76.142: ICMP echo request, id 2, seq 2, length 64
13:23:26.009444 IP 142.250.76.142 > 60.61.0.1: ICMP echo reply, id 2, seq 2, length 64
13:23:26.981956 IP 60.61.0.1 > 142.250.76.142: ICMP echo request, id 2, seq 3, length 64
13:23:27.015232 IP 142.250.76.142 > 60.61.0.1: ICMP echo reply, id 2, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<h2 id="changelog">Changelog (summary)</h2>

- [2021.08.17] Initial release.
