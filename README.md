# free5GC 5GC & UERANSIM UE / RAN Sample Configuration - Select nearby UPF according to the connected gNodeB
This describes a very simple configuration that uses free5GC and UERANSIM to select the nearby UPF according to the connected gNodeB.

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

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
    - [Ping google.com going through DN=10.60.0.0/16 on Loc1](#ping_ue1)
  - [Run UERANSIM (UE in Loc2)](#run_ue2)
    - [Start UE connected to gNodeB2 in Loc2](#con_ue2)
    - [Ping google.com going through DN=10.61.0.0/16 on Loc2](#ping_ue2)
- [Changelog (summary)](#changelog)

---
<a id="overview"></a>

## Overview of free5GC 5GC Simulation Mobile Network

The following minimum configuration was set as a condition.
- The pair of gNodeB and UPF exists in the same location.
- The UE connected to gNodeB connects to DN managed by UPF in the same location.

**In this example, TAC is matched to connect gNodeB and AMF, and AMF searches for SMF using `preferred target NF location` as one of the parameters.
Also, AMF and SMF search for PCF using `preferred target NF location` as one of the parameters.**

The built simulation environment is as follows.

<img src="./images/network-overview.png" title="./images/network-overview.png" width=1000px></img>

The 5GC / UE / RAN used are as follows.
- 5GC - free5GC v3.2.1 - https://github.com/free5gc/free5gc
- UE / RAN - UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM

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
| PCF1 | -- | 127.0.0.7 | -- | loc1 |
| AMF2 | 192.168.0.143 | 127.0.0.28 | 2 | loc2 |
| SMF2 | 192.168.0.143 | 127.0.0.12 | -- | loc2 |
| PCF2 | -- | 127.0.0.17 | -- | loc2 |

GUAMIs are as follows.  
| AMF | MCC | MNC | AMF ID |
| --- | --- | --- | --- |
| AMF1 | 001 | 01 | cafe00 |
| AMF2 | 001 | 01 | cafe01 |

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
In addition, [3GPP TS 35.208](https://www.3gpp.org/DynaReport/35208.htm) "4.3 Test Sets" is published by 3GPP as test data for the 3GPP authentication and key generation functions (MILENAGE).

Each DNs are as follows.
| DN | Location # |  TUNnel interface of DN | DNN | TUNnel interface of UE | U-Plane # |
| --- | --- | --- | --- | --- | --- |
| 10.60.0.0/16 | Loc1 | upfgtp | internet | uesimtun0 | U-Plane1 |
| 10.61.0.0/16 | Loc2 | upfgtp | internet | uesimtun0 | U-Plane2 |

<a id="changes"></a>

## Changes in configuration files of free5GC 5GC and UERANSIM UE / RAN

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.2.1 - https://free5gc.org/guide/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

- `free5gc/config/amfcfg1.yaml`
```diff
--- amfcfg.yaml.orig    2022-04-01 20:25:53.176313323 +0900
+++ amfcfg1.yaml        2022-04-03 22:17:29.059691038 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.142
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
     registerIPv4: 127.0.0.18 # IP used to register to NRF
@@ -23,23 +23,21 @@
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
@@ -53,7 +51,7 @@
   networkName:  # the name of this core network
     full: free5GC
     short: free
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF and UPFs are located
   networkFeatureSupport5GS: # 5gs Network Feature Support IE, refer to TS 24.501
     enable: true # append this IE in Registration accept or not
     length: 1 # IE content length (uinteger, range: 1~3)
```
- `free5gc/config/amfcfg2.yaml`
```diff
--- amfcfg.yaml.orig    2022-04-01 20:25:54.000000000 +0900
+++ amfcfg2.yaml        2022-08-11 16:45:08.075088545 +0900
@@ -5,11 +5,11 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.143
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.18 # IP used to register to NRF
-    bindingIPv4: 127.0.0.18  # IP used to bind the service
+    registerIPv4: 127.0.0.28 # IP used to register to NRF
+    bindingIPv4: 127.0.0.28  # IP used to bind the service
     port: 8000 # port used to bind the service
     tls: # the local path of TLS key
       pem: config/TLS/amf.pem # AMF TLS Certificate
@@ -23,23 +23,21 @@
   servedGuamiList: # Guami (Globally Unique AMF ID) list supported by this AMF
     # <GUAMI> = <MCC><MNC><AMF ID>
     - plmnId: # Public Land Mobile Network ID, <PLMN ID> = <MCC><MNC>
-        mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-        mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-      amfId: cafe00 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+      amfId: cafe01 # AMF identifier (3 bytes hex string, range: 000000~FFFFFF)
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
@@ -53,7 +51,7 @@
   networkName:  # the name of this core network
     full: free5GC
     short: free
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF and UPFs are located
   networkFeatureSupport5GS: # 5gs Network Feature Support IE, refer to TS 24.501
     enable: true # append this IE in Registration accept or not
     length: 1 # IE content length (uinteger, range: 1~3)
```
- `free5gc/config/smfcfg1.yaml`
```diff
--- smfcfg.yaml.orig    2022-04-01 20:25:53.177313344 +0900
+++ smfcfg1.yaml        2022-04-03 22:13:41.582022190 +0900
@@ -32,18 +32,18 @@
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+  locality: loc1 # Name of the location where a set of AMF, SMF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.142
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.144 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -52,22 +52,16 @@
               - dnn: internet
                 pools:
                   - cidr: 10.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.61.0.0/16
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
+  ulcl: false
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/smfcfg2.yaml`
```diff
--- smfcfg.yaml.orig    2022-04-01 20:25:53.177313344 +0900
+++ smfcfg2.yaml        2022-04-03 22:16:16.540556548 +0900
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
       key: config/TLS/smf.key # SMF TLS Certificate
@@ -32,18 +32,18 @@
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: "208" # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: "93" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-  locality: area1 # Name of the location where a set of AMF, SMF and UPFs are located
+    - mcc: "001" # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: "01" # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+  locality: loc2 # Name of the location where a set of AMF, SMF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
-    addr: 127.0.0.1
+    addr: 192.168.0.143
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF:  # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.145 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -51,23 +51,17 @@
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
-                  - cidr: 10.60.0.0/16
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
                   - cidr: 10.61.0.0/16
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
+  ulcl: false
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/pcfcfg1.yaml`
```diff
--- pcfcfg.yaml.orig    2022-04-03 19:31:43.524558018 +0900
+++ pcfcfg1.yaml        2022-04-03 22:20:00.252348433 +0900
@@ -27,7 +27,7 @@
   mongodb:       # the mongodb connected by this PCF
     name: free5gc                  # name of the mongodb
     url: mongodb://localhost:27017 # a valid URL of the mongodb
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/pcfcfg2.yaml`
```diff
--- pcfcfg.yaml.orig    2022-04-03 19:31:43.524558018 +0900
+++ pcfcfg2.yaml        2022-04-03 22:20:33.389124657 +0900
@@ -6,8 +6,8 @@
   pcfName: PCF # the name of this PCF
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.7 # IP used to register to NRF
-    bindingIPv4: 127.0.0.7  # IP used to bind the service
+    registerIPv4: 127.0.0.17 # IP used to register to NRF
+    bindingIPv4: 127.0.0.17  # IP used to bind the service
     port: 8000              # port used to bind the service
     tls: # the local path of TLS key
       pem: config/TLS/pcf.pem # PCF TLS Certificate
@@ -27,7 +27,7 @@
   mongodb:       # the mongodb connected by this PCF
     name: free5gc                  # name of the mongodb
     url: mongodb://localhost:27017 # a valid URL of the mongodb
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
 
 # the kind of log output
 # debugLevel: how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2022-04-01 20:25:53.176313323 +0900
+++ ausfcfg.yaml        2022-04-03 21:56:52.574055040 +0900
@@ -15,8 +15,8 @@
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
--- nrfcfg.yaml.orig    2022-04-01 20:25:53.177313344 +0900
+++ nrfcfg.yaml 2022-04-03 21:57:18.376344873 +0900
@@ -14,8 +14,8 @@
       pem: config/TLS/nrf.pem # NRF TLS Certificate
       key: config/TLS/nrf.key # NRF TLS Private key
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
--- nssfcfg.yaml.orig   2022-04-01 20:25:53.177313344 +0900
+++ nssfcfg.yaml        2022-04-03 21:58:01.240821301 +0900
@@ -17,12 +17,12 @@
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

<a id="changes_up1"></a>

### Changes in configuration files of free5GC 5GC U-Plane1

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2022-08-11 14:42:32.545805826 +0900
+++ upfcfg.yaml 2022-08-11 16:52:48.563998874 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.144   # IP addr for listening
+  nodeID: 192.168.0.144 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,7 +13,7 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.144
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -21,7 +21,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.60.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_up2"></a>

### Changes in configuration files of free5GC 5GC U-Plane2

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2022-08-11 14:44:10.418739050 +0900
+++ upfcfg.yaml 2022-08-11 16:56:14.233871256 +0900
@@ -3,8 +3,8 @@
 
 # The listen IP and nodeID of the N4 interface on this UPF (Can't set to 0.0.0.0)
 pfcp:
-  addr: 127.0.0.8   # IP addr for listening
-  nodeID: 127.0.0.8 # External IP or FQDN can be reached
+  addr: 192.168.0.145   # IP addr for listening
+  nodeID: 192.168.0.145 # External IP or FQDN can be reached
   retransTimeout: 1s # retransmission timeout
   maxRetrans: 3 # the max number of retransmission
 
@@ -13,7 +13,7 @@
   # The IP list of the N3/N9 interfaces on this UPF
   # If there are multiple connection, set addr to 0.0.0.0 or list all the addresses
   ifList:
-    - addr: 127.0.0.8
+    - addr: 192.168.0.145
       type: N3
       # name: upf.5gc.nctu.me
       # ifname: gtpif
@@ -21,7 +21,7 @@
 # The DNN list supported by UPF
 dnnList:
   - dnn: internet # Data Network Name
-    cidr: 10.60.0.0/24 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
+    cidr: 10.61.0.0/16 # Classless Inter-Domain Routing for assigned IPv4 pool of UE
     # natifname: eth0
 
 logger: # log output setting
```

<a id="changes_ueransim"></a>

### Changes in configuration files of UERANSIM UE / RAN

<a id="changes_ran1"></a>

#### Changes in configuration files of RAN (gNodeB1)

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

<a id="changes_ran2"></a>

#### Changes in configuration files of RAN (gNodeB2)

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

<a id="changes_ue_loc1"></a>

#### Changes in configuration files of UE for Loc1 (IMSI-001010000000000)

- `UERANSIM/config/free5gc-ue-loc1.yaml`
```diff
--- free5gc-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ free5gc-ue-loc1.yaml        2022-04-04 19:33:36.993974680 +0900
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

<a id="changes_ue_loc2"></a>

#### Changes in configuration files of UE for Loc2 (IMSI-001010000000000)

- `UERANSIM/config/free5gc-ue-loc2.yaml`
```diff
--- free5gc-ue.yaml.orig        2021-09-18 21:11:52.000000000 +0900
+++ free5gc-ue-loc2.yaml        2022-04-04 19:34:35.676208193 +0900
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

<a id="network_settings"></a>

## Network settings of free5GC 5GC and UERANSIM UE / RAN

<a id="network_settings_cp"></a>

### Network settings of free5GC 5GC C-Plane

Add IP addresses for (AMF1 & SMF1 & PCF1) and (AMF2 & SMF2 & PCF2).
```
ip addr add 192.168.0.142/24 dev enp0s8
ip addr add 192.168.0.143/24 dev enp0s8
```
**Note. `enp0s8` is the network interface of `192.168.0.0/24` in my VirtualBox environment.
Please change it according to your environment.**

<a id="network_settings_up1"></a>

### Network settings of free5GC 5GC U-Plane1

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.60.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="network_settings_up2"></a>

### Network settings of free5GC 5GC U-Plane2

First, uncomment the next line in the `/etc/sysctl.conf` file and reflect it in the OS.
```
net.ipv4.ip_forward=1
```
```
# sysctl -p
```
Next, configure NAPT.
```
# iptables -t nat -A POSTROUTING -s 10.61.0.0/16 ! -o upfgtp -j MASQUERADE
```

<a id="build"></a>

## Build free5GC and UERANSIM

**Note. It is recommended to use go1.18.x according to the commit to free5gc/openapi on 2022.10.26.**

Please refer to the following for building free5GC and UERANSIM respectively.
- free5GC v3.2.1 - https://free5gc.org/guide/
- UERANSIM v3.2.6 - https://github.com/aligungr/UERANSIM/wiki/Installation

Install MongoDB on free5GC 5GC C-Plane machine.
It is not necessary to install MongoDB on free5GC 5GC U-Plane machines.
[MongoDB Compass](https://www.mongodb.com/products/compass) is a convenient tool to look at the MongoDB database.

**Note. The installation guide also includes instructions on building the latest committed version.**

<a id="run"></a>

## Run free5GC 5GC and UERANSIM UE / RAN

First run the 5GC, then UERANSIM (UE & RAN implementation).

<a id="run_up"></a>

### Run free5GC 5GC U-Plane1 & U-Plane2

First, run free5GC 5GC U-Planes. Please see [here](https://github.com/free5gc/free5gc/issues/170#issuecomment-773214169) for the reason.  
**Note. It was improved on 2022.11.08, and you don't have to worry about the startup order of C-Plane and U-Plane.**

- free5GC 5GC U-Plane1
```
# cd free5gc
# bin/upf
```
- free5GC 5GC U-Plane2
```
# cd free5gc
# bin/upf
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

<a id="run_cp"></a>

### Run free5GC 5GC C-Plane

Next, run free5GC 5GC C-Plane.

- free5GC 5GC C-Plane

Create the following shell script and run it.
```bash
#!/usr/bin/env bash

PID_LIST=()

NF_LIST1="amf smf pcf"
NF_LIST2="udr udm nssf ausf"

export GIN_MODE=release

./bin/nrf &
PID_LIST+=($!)
sleep 1

for NF in ${NF_LIST1}; do
    ./bin/${NF} -c config/${NF}cfg1.yaml &
    PID_LIST+=($!)
    sleep 1
    ./bin/${NF} -c config/${NF}cfg2.yaml &
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

<a id="run_ran"></a>

### Run UERANSIM (gNodeBs)

Run each gNodeB with TAC=1 and TAC=2 in two locations.  
Please refer to the following for usage of UERANSIM.

https://github.com/aligungr/UERANSIM/wiki/Usage

<a id="run_ran1"></a>

#### Start gNodeB1 with TAC=1 in Loc1

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2022-08-11 17:15:23.053] [sctp] [info] Trying to establish SCTP connection... (192.168.0.142:38412)
[2022-08-11 17:15:23.055] [sctp] [info] SCTP connection established (192.168.0.142:38412)
[2022-08-11 17:15:23.056] [sctp] [debug] SCTP association setup ascId[6]
[2022-08-11 17:15:23.056] [ngap] [debug] Sending NG Setup Request
[2022-08-11 17:15:23.058] [ngap] [debug] NG Setup Response received
[2022-08-11 17:15:23.058] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2022-08-11T17:15:23+09:00 [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.131:45635
2022-08-11T17:15:23+09:00 [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.131:45635
2022-08-11T17:15:23+09:00 [INFO][AMF][NGAP][192.168.0.131:45635] Handle NG Setup request
2022-08-11T17:15:23+09:00 [INFO][AMF][NGAP][192.168.0.131:45635] Send NG-Setup response
```

<a id="run_ran2"></a>

#### Start gNodeB2 with TAC=2 in Loc2

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2022-08-11 17:16:15.782] [sctp] [info] Trying to establish SCTP connection... (192.168.0.143:38412)
[2022-08-11 17:16:15.785] [sctp] [info] SCTP connection established (192.168.0.143:38412)
[2022-08-11 17:16:15.785] [sctp] [debug] SCTP association setup ascId[4]
[2022-08-11 17:16:15.786] [ngap] [debug] Sending NG Setup Request
[2022-08-11 17:16:15.788] [ngap] [debug] NG Setup Response received
[2022-08-11 17:16:15.788] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2022-08-11T17:16:15+09:00 [INFO][AMF][NGAP] [AMF] SCTP Accept from: 192.168.0.132:45375
2022-08-11T17:16:15+09:00 [INFO][AMF][NGAP] Create a new NG connection for: 192.168.0.132:45375
2022-08-11T17:16:15+09:00 [INFO][AMF][NGAP][192.168.0.132:45375] Handle NG Setup request
2022-08-11T17:16:15+09:00 [INFO][AMF][NGAP][192.168.0.132:45375] Send NG-Setup response
```

<a id="run_ue1"></a>

### Run UERANSIM (UE in Loc1)

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<a id="con_ue1"></a>

#### Start UE connected to gNodeB1 in Loc1

```
# ./nr-ue -c ../config/free5gc-ue-loc1.yaml 
UERANSIM v3.2.6
[2022-08-11 17:16:56.819] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-08-11 17:16:56.820] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-08-11 17:16:56.821] [nas] [info] Selected plmn[001/01]
[2022-08-11 17:16:56.822] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2022-08-11 17:16:56.822] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-08-11 17:16:56.822] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-08-11 17:16:56.822] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-08-11 17:16:56.823] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-11 17:16:56.823] [nas] [debug] Sending Initial Registration
[2022-08-11 17:16:56.823] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-08-11 17:16:56.823] [rrc] [debug] Sending RRC Setup Request
[2022-08-11 17:16:56.824] [rrc] [info] RRC connection established
[2022-08-11 17:16:56.825] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-08-11 17:16:56.825] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-08-11 17:16:56.848] [nas] [debug] Authentication Request received
[2022-08-11 17:16:56.860] [nas] [debug] Security Mode Command received
[2022-08-11 17:16:56.860] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-08-11 17:16:56.903] [nas] [debug] Registration accept received
[2022-08-11 17:16:56.904] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-08-11 17:16:56.904] [nas] [debug] Sending Registration Complete
[2022-08-11 17:16:56.904] [nas] [info] Initial Registration is successful
[2022-08-11 17:16:56.904] [nas] [debug] Sending PDU Session Establishment Request
[2022-08-11 17:16:56.905] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-11 17:16:57.165] [nas] [debug] PDU Session Establishment Accept received
[2022-08-11 17:16:57.168] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-08-11 17:16:57.190] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635] Handle Initial UE Message
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2022-08-11T17:16:56+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2022-08-11T17:16:56+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2022-08-11T17:16:56+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2022-08-11T17:16:56+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2022-08-11T17:16:56+09:00 [INFO][UDM][Suci] scheme 0
2022-08-11T17:16:56+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2022-08-11T17:16:56+09:00 [ERRO][UDM][UEAU] opStr length is  0
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2022-08-11T17:16:56+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2022-08-11T17:16:56+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2022-08-11T17:16:56+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 6265363635386332376533306163656162636339613936353264656663303635
2022-08-11T17:16:56+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2022-08-11T17:16:56+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2022-08-11T17:16:56+09:00 [INFO][AUSF][5gAkaAuth] res*: 6265363635386332376533306163656162636339613936353264656663303635
Xres*: 6265363635386332376533306163656162636339613936353264656663303635
2022-08-11T17:16:56+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2022-08-11T17:16:56+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2022-08-11T17:16:56+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2022-08-11T17:16:56+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2022-08-11T17:16:56+09:00 [INFO][UDM][SDM] Handle GetNssai
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2022-08-11T17:16:56+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2022-08-11T17:16:56+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2022-08-11T17:16:56+09:00 [INFO][UDM][SDM] Handle GetAmData
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:16:56+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:16:56+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2022-08-11T17:16:56+09:00 [INFO][UDM][SDM] Handle Subscribe
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2022-08-11T17:16:56+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2022-08-11T17:16:56+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2022-08-11T17:16:56+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2022-08-11T17:16:56+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2022-08-11T17:16:56+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:56+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2022-08-11T17:16:56+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2022-08-11T17:16:56+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2022-08-11T17:16:56+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2022-08-11T17:16:56+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2022-08-11T17:16:56+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2022-08-11T17:16:56+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Handle Initial Context Setup Response
2022-08-11T17:16:57+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:16:57+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2022-08-11T17:16:57+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2022-08-11T17:16:57+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2022-08-11T17:16:57+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:16:57+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2022-08-11T17:16:57+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2022-08-11T17:16:57+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2022-08-11T17:16:57+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2022-08-11T17:16:57+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:57+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2022-08-11T17:16:57+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2022-08-11T17:16:57+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=cc20b04f-f17c-4cda-9f48-733299e2fdcd&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2022-08-11T17:16:57+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:57+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2022-08-11T17:16:57+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:57+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2022-08-11T17:16:57+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2022-08-11T17:16:57+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.60.0.1]
2022-08-11T17:16:57+09:00 [INFO][UDM][SDM] Handle GetSmData
2022-08-11T17:16:57+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2022-08-11T17:16:57+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2022-08-11T17:16:57+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:16:57+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:16:57+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2022-08-11T17:16:57+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc00000de20 0xc00000de60]
2022-08-11T17:16:57+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2022-08-11T17:16:57+09:00 [INFO][SMF][GSM] &{[0xc00000de20 0xc00000de60]}
2022-08-11T17:16:57+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2022-08-11T17:16:57+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:57+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=SMF&target-nf-type=PCF |
2022-08-11T17:16:57+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2022-08-11T17:16:57+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdSmDataGet
2022-08-11T17:16:57+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:16:57+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2022-08-11T17:16:57+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:16:57+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=cc20b04f-f17c-4cda-9f48-733299e2fdcd&target-nf-type=AMF |
2022-08-11T17:16:57+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2022-08-11T17:16:57+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2022-08-11T17:16:57+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2022-08-11T17:16:57+09:00 [INFO][LIB][PFCP] Remove Request Transaction [2]
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2022-08-11T17:16:57+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2022-08-11T17:16:57+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2022-08-11T17:16:57+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2022-08-11T17:16:57+09:00 [INFO][AMF][NGAP][192.168.0.131:45635][AMF_UE_NGAP_ID:1] Handle PDU Session Resource Setup Response
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2022-08-11T17:16:57+09:00 [INFO][LIB][PFCP] Remove Request Transaction [3]
2022-08-11T17:16:57+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2022-08-11T17:16:57+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:ed9f05c9-09e0-4628-b04f-2dbb1e88ecd1/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2022-08-11T17:16:57+09:00 [INFO][UPF][Pfcp][192.168.0.144:8805] handleSessionEstablishmentRequest
2022-08-11T17:16:57+09:00 [INFO][UPF][Pfcp][192.168.0.144:8805][rNodeID:192.168.0.142][SEID:L(0x1),R(0x1)] New session
2022-08-11T17:16:57+09:00 [INFO][UPF][Pfcp][192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
6: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::6ccf:a815:3ea3:5278/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue1"></a>

#### Ping google.com going through DN=10.60.0.0/16 on Loc1

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.31.142) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.31.142: icmp_seq=1 ttl=61 time=19.1 ms
64 bytes from 172.217.31.142: icmp_seq=2 ttl=61 time=19.8 ms
64 bytes from 172.217.31.142: icmp_seq=3 ttl=61 time=19.6 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
17:19:31.293704 IP 10.60.0.1 > 172.217.31.142: ICMP echo request, id 3, seq 1, length 64
17:19:31.311046 IP 172.217.31.142 > 10.60.0.1: ICMP echo reply, id 3, seq 1, length 64
17:19:32.295029 IP 10.60.0.1 > 172.217.31.142: ICMP echo request, id 3, seq 2, length 64
17:19:32.312711 IP 172.217.31.142 > 10.60.0.1: ICMP echo reply, id 3, seq 2, length 64
17:19:33.296769 IP 10.60.0.1 > 172.217.31.142: ICMP echo request, id 3, seq 3, length 64
17:19:33.314139 IP 172.217.31.142 > 10.60.0.1: ICMP echo reply, id 3, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane2. The UE connects to the DN of U-Plane1 in the same Loc1 according to the connected gNodeB1 in Loc1.**

<a id="run_ue2"></a>

### Run UERANSIM (UE in Loc2)

Then the UE disconnects from gNodeB1 and connects to gNodeB2 in Loc2.

<a id="con_ue2"></a>

#### Start UE connected to gNodeB2 in Loc2

```
# ./nr-ue -c ../config/free5gc-ue-loc2.yaml 
UERANSIM v3.2.6
[2022-08-11 17:21:47.921] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2022-08-11 17:21:47.922] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2022-08-11 17:21:47.922] [nas] [info] Selected plmn[001/01]
[2022-08-11 17:21:47.922] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2022-08-11 17:21:47.923] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2022-08-11 17:21:47.923] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2022-08-11 17:21:47.923] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2022-08-11 17:21:47.923] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-11 17:21:47.923] [nas] [debug] Sending Initial Registration
[2022-08-11 17:21:47.924] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2022-08-11 17:21:47.924] [rrc] [debug] Sending RRC Setup Request
[2022-08-11 17:21:47.925] [rrc] [info] RRC connection established
[2022-08-11 17:21:47.925] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2022-08-11 17:21:47.925] [nas] [info] UE switches to state [CM-CONNECTED]
[2022-08-11 17:21:47.941] [nas] [debug] Authentication Request received
[2022-08-11 17:21:47.949] [nas] [debug] Security Mode Command received
[2022-08-11 17:21:47.950] [nas] [debug] Selected integrity[2] ciphering[0]
[2022-08-11 17:21:47.984] [nas] [debug] Registration accept received
[2022-08-11 17:21:47.985] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2022-08-11 17:21:47.985] [nas] [debug] Sending Registration Complete
[2022-08-11 17:21:47.985] [nas] [info] Initial Registration is successful
[2022-08-11 17:21:47.985] [nas] [debug] Sending PDU Session Establishment Request
[2022-08-11 17:21:47.986] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2022-08-11 17:21:48.243] [nas] [debug] PDU Session Establishment Accept received
[2022-08-11 17:21:48.248] [nas] [info] PDU Session establishment is successful PSI[1]
[2022-08-11 17:21:48.272] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375] Handle Initial UE Message
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Registration Request
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Authentication procedure
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2022-08-11T17:21:47+09:00 [INFO][AUSF][UeAuthPost] HandleUeAuthPostRequest
2022-08-11T17:21:47+09:00 [INFO][AUSF][UeAuthPost] Serving network authorized
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2022-08-11T17:21:47+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2022-08-11T17:21:47+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2022-08-11T17:21:47+09:00 [INFO][UDM][Suci] scheme 0
2022-08-11T17:21:47+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle QueryAuthSubsData
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2022-08-11T17:21:47+09:00 [ERRO][UDM][UEAU] opStr length is  0
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle ModifyAuthentication
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2022-08-11T17:21:47+09:00 [INFO][AUSF][UeAuthPost] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2022-08-11T17:21:47+09:00 [INFO][AUSF][UeAuthPost] Use 5G AKA auth method
2022-08-11T17:21:47+09:00 [INFO][AUSF][5gAkaAuth] XresStar = 6462626561356534343436636634303638323165353265393834636263343238
2022-08-11T17:21:47+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Send Authentication Request
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1] Handle Authentication Response
2022-08-11T17:21:47+09:00 [INFO][AUSF][5gAkaAuth] Auth5gAkaComfirmRequest
2022-08-11T17:21:47+09:00 [INFO][AUSF][5gAkaAuth] res*: 6462626561356534343436636634303638323165353265393834636263343238
Xres*: 6462626561356534343436636634303638323165353265393834636263343238
2022-08-11T17:21:47+09:00 [INFO][AUSF][5gAkaAuth] 5G AKA confirmation succeeded
2022-08-11T17:21:47+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle CreateAuthenticationStatus
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2022-08-11T17:21:47+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Security Mode Command
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Send Downlink Nas Transport
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Security Mode Complete
2022-08-11T17:21:47+09:00 [INFO][LIB][FSM] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle InitialRegistration
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2022-08-11T17:21:47+09:00 [INFO][UDM][SDM] Handle GetNssai
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features= |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2022-08-11T17:21:47+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2022-08-11T17:21:47+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle CreateAmfContext3gpp
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2022-08-11T17:21:47+09:00 [ERRO][UDM][HTTP] unsupported scheme[]
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 204 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2022-08-11T17:21:47+09:00 [INFO][UDM][SDM] Handle GetAmData
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle QueryAmData
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:21:47+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle QuerySmfSelectData
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data?supported-features= |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:21:47+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle QuerySmfRegList
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations?supported-features= |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2022-08-11T17:21:47+09:00 [INFO][UDM][SDM] Handle Subscribe
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle CreateSdmSubscriptions
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2022-08-11T17:21:47+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2022-08-11T17:21:47+09:00 [INFO][PCF][Ampolicy] Handle AM Policy Create Request
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2022-08-11T17:21:47+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdAmDataGet
2022-08-11T17:21:47+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2022-08-11T17:21:47+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:47+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe01%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2022-08-11T17:21:47+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2022-08-11T17:21:47+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Send Registration Accept
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Send Initial Context Setup Request
2022-08-11T17:21:47+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Handle Initial Context Setup Response
2022-08-11T17:21:48+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:21:48+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2022-08-11T17:21:48+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle Registration Complete
2022-08-11T17:21:48+09:00 [INFO][LIB][FSM] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2022-08-11T17:21:48+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Uplink NAS Transport (RAN UE NGAP ID: 1)
2022-08-11T17:21:48+09:00 [INFO][LIB][FSM] Handle event[Gmm Message], transition from [Registered] to [Registered]
2022-08-11T17:21:48+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Handle UL NAS Transport
2022-08-11T17:21:48+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2022-08-11T17:21:48+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2022-08-11T17:21:48+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:48+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2022-08-11T17:21:48+09:00 [INFO][NSSF][NsSelect] Handle NSSelectionGet
2022-08-11T17:21:48+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=ccc15824-b48b-4977-9afd-3c15484993d0&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2022-08-11T17:21:48+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:48+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc2&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2022-08-11T17:21:48+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:48+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Send NF Discovery Serving UDM Successfully
2022-08-11T17:21:48+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2022-08-11T17:21:48+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] IP[10.61.0.1]
2022-08-11T17:21:48+09:00 [INFO][UDM][SDM] Handle GetSmData
2022-08-11T17:21:48+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2022-08-11T17:21:48+09:00 [INFO][UDR][DRepo] Handle QuerySmData
2022-08-11T17:21:48+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:21:48+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:21:48+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2022-08-11T17:21:48+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc00000de20 0xc00000de60]
2022-08-11T17:21:48+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2022-08-11T17:21:48+09:00 [INFO][SMF][GSM] &{[0xc00000de20 0xc00000de60]}
2022-08-11T17:21:48+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] PCF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2022-08-11T17:21:48+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:48+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=SMF&target-nf-type=PCF |
2022-08-11T17:21:48+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2022-08-11T17:21:48+09:00 [INFO][UDR][DRepo] Handle PolicyDataUesUeIdSmDataGet
2022-08-11T17:21:48+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2022-08-11T17:21:48+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] SUPI[imsi-001010000000000] has no pre-config route
2022-08-11T17:21:48+09:00 [INFO][NRF][DSCV] Handle NFDiscoveryRequest
2022-08-11T17:21:48+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=ccc15824-b48b-4977-9afd-3c15484993d0&target-nf-type=AMF |
2022-08-11T17:21:48+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2022-08-11T17:21:48+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2022-08-11T17:21:48+09:00 [INFO][AMF][GMM][AMF_UE_NGAP_ID:1][SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2022-08-11T17:21:48+09:00 [INFO][LIB][PFCP] Remove Request Transaction [2]
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2022-08-11T17:21:48+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2022-08-11T17:21:48+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Send PDU Session Resource Setup Request
2022-08-11T17:21:48+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2022-08-11T17:21:48+09:00 [INFO][AMF][NGAP][192.168.0.132:45375][AMF_UE_NGAP_ID:1] Handle PDU Session Resource Setup Response
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextUpdate
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Sending PFCP Session Modification Request to AN UPF
2022-08-11T17:21:48+09:00 [INFO][LIB][PFCP] Remove Request Transaction [3]
2022-08-11T17:21:48+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2022-08-11T17:21:48+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:0626b41f-0c48-4c81-97d7-b21ca25fdb75/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2022-08-11T17:21:48+09:00 [INFO][UPF][Pfcp][192.168.0.145:8805] handleSessionEstablishmentRequest
2022-08-11T17:21:48+09:00 [INFO][UPF][Pfcp][192.168.0.145:8805][rNodeID:192.168.0.143][SEID:L(0x1),R(0x1)] New session
2022-08-11T17:21:48+09:00 [INFO][UPF][Pfcp][192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
7: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::4d16:9ca0:7a90:38ea/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue2"></a>

#### Ping google.com going through DN=10.61.0.0/16 on Loc2

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.31.142) from 10.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.31.142: icmp_seq=1 ttl=61 time=19.9 ms
64 bytes from 172.217.31.142: icmp_seq=2 ttl=61 time=20.1 ms
64 bytes from 172.217.31.142: icmp_seq=3 ttl=61 time=25.8 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
17:23:58.380217 IP 10.61.0.1 > 172.217.31.142: ICMP echo request, id 4, seq 1, length 64
17:23:58.398164 IP 172.217.31.142 > 10.61.0.1: ICMP echo reply, id 4, seq 1, length 64
17:23:59.382056 IP 10.61.0.1 > 172.217.31.142: ICMP echo request, id 4, seq 2, length 64
17:23:59.400024 IP 172.217.31.142 > 10.61.0.1: ICMP echo reply, id 4, seq 2, length 64
17:24:00.383294 IP 10.61.0.1 > 172.217.31.142: ICMP echo request, id 4, seq 3, length 64
17:24:00.406703 IP 172.217.31.142 > 10.61.0.1: ICMP echo reply, id 4, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2022.08.11] Updated to free5GC v3.2.1.
- [2022.04.04] Updated to free5GC v3.1.0 and UERANSIM v3.2.6.
- [2021.09.25] Updated to free5GC v3.0.6.
- [2021.08.17] Initial release.
