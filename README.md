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
- 5GC - free5GC v3.4.1 (2024.03.28) - https://github.com/free5gc/free5gc
- UE / RAN - UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM

Each VMs are as follows.  
| VM # | SW & Role | IP address | OS | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- | --- | --- |
| VM1 | free5GC  5GC C-Plane | 192.168.0.141/24 <br> 192.168.0.142/24 <br> 192.168.0.143/24 | Ubuntu 22.04 | 2GB | 20GB |
| VM2 | free5GC  5GC U-Plane1  | 192.168.0.144/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM3 | free5GC  5GC U-Plane2  | 192.168.0.145/24 | Ubuntu 22.04 | 1GB | 20GB |
| VM4 | UERANSIM RAN (gNodeB1) | 192.168.0.131/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM5 | UERANSIM RAN (gNodeB2) | 192.168.0.132/24 | Ubuntu 22.04 | 1GB | 10GB |
| VM6 | UERANSIM UE | 192.168.0.133/24 | Ubuntu 22.04 | 1GB | 10GB |

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
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

<a id="changes_cp"></a>

### Changes in configuration files of free5GC 5GC C-Plane

- `free5gc/config/amfcfg1.yaml`
```diff
--- amfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ amfcfg1.yaml        2024-03-31 15:06:30.789884969 +0900
@@ -5,7 +5,7 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.142
   ngapPort: 38412 # the SCTP port listened by NGAP
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
@@ -24,23 +24,21 @@
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
       tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
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
@@ -111,7 +109,7 @@
     enable: true     # true or false
     expireTime: 6s   # default is 6 seconds
     maxRetryTimes: 4 # the max number of retransmission
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   sctp: # set the sctp server setting <optinal>, once this field is set, please also add maxInputStream, maxOsStream, maxAttempts, maxInitTimeOut
     numOstreams: 3 # the maximum out streams of each sctp connection
     maxInstreams: 5 # the maximum in streams of each sctp connection
```
- `free5gc/config/amfcfg2.yaml`
```diff
--- amfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ amfcfg2.yaml        2024-03-31 17:06:03.278028423 +0900
@@ -5,12 +5,12 @@
 configuration:
   amfName: AMF # the name of this AMF
   ngapIpList:  # the IP list of N2 interfaces on this AMF
-    - 127.0.0.18
+    - 192.168.0.143
   ngapPort: 38412 # the SCTP port listened by NGAP
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.18 # IP used to register to NRF
-    bindingIPv4: 127.0.0.18  # IP used to bind the service
+    registerIPv4: 127.0.0.28 # IP used to register to NRF
+    bindingIPv4: 127.0.0.28  # IP used to bind the service
     port: 8000 # port used to bind the service
     tls: # the local path of TLS key
       pem: cert/amf.pem # AMF TLS Certificate
@@ -24,23 +24,21 @@
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
-      tac: 000001 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
+        mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+        mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+      tac: 000002 # Tracking Area Code (3 bytes hex string, range: 000000~FFFFFF)
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
@@ -111,7 +109,7 @@
     enable: true     # true or false
     expireTime: 6s   # default is 6 seconds
     maxRetryTimes: 4 # the max number of retransmission
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   sctp: # set the sctp server setting <optinal>, once this field is set, please also add maxInputStream, maxOsStream, maxAttempts, maxInitTimeOut
     numOstreams: 3 # the maximum out streams of each sctp connection
     maxInstreams: 5 # the maximum in streams of each sctp connection
```
- `free5gc/config/ausfcfg.yaml`
```diff
--- ausfcfg.yaml.orig   2024-03-30 10:35:10.534612278 +0900
+++ ausfcfg.yaml        2024-03-30 10:48:53.470630936 +0900
@@ -16,10 +16,8 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
   plmnSupportList: # the PLMNs (Public Land Mobile Network) list supported by this AUSF
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-    - mcc: 123 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 45  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01  # Mobile Network Code (2 or 3 digits string, digit: 0~9)
   groupId: ausfGroup001 # ID for the group of the AUSF
   eapAkaSupiImsiPrefix: false # including "imsi-" prefix or not when using the SUPI to do EAP-AKA' authentication
 
```
- `free5gc/config/nrfcfg.yaml`
```diff
--- nrfcfg.yaml.orig    2024-03-30 10:35:10.534612278 +0900
+++ nrfcfg.yaml 2024-03-31 12:34:36.380715909 +0900
@@ -15,8 +15,8 @@
       key: cert/nrf.key # NRF TLS Private key
     oauth: true
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
--- nssfcfg.yaml.orig   2024-03-30 10:35:10.535609025 +0900
+++ nssfcfg.yaml        2024-03-31 15:19:58.888726629 +0900
@@ -18,12 +18,12 @@
   nrfUri: http://127.0.0.10:8000 # a valid URI of NRF
   nrfCertPem: cert/nrf.pem # NRF Certificate
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
- `free5gc/config/pcfcfg1.yaml`
```diff
--- pcfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ pcfcfg1.yaml        2024-03-31 15:17:42.281994415 +0900
@@ -28,7 +28,7 @@
   mongodb:       # the mongodb connected by this PCF
     name: free5gc                  # name of the mongodb
     url: mongodb://localhost:27017 # a valid URL of the mongodb
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
 
 logger: # log output setting
   enable: true # true or false
```
- `free5gc/config/pcfcfg2.yaml`
```diff
--- pcfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ pcfcfg2.yaml        2024-03-31 15:18:07.590831559 +0900
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
       pem: cert/pcf.pem # PCF TLS Certificate
@@ -28,7 +28,7 @@
   mongodb:       # the mongodb connected by this PCF
     name: free5gc                  # name of the mongodb
     url: mongodb://localhost:27017 # a valid URL of the mongodb
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+  locality: loc2 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
 
 logger: # log output setting
   enable: true # true or false
```
- `free5gc/config/smfcfg1.yaml`
```diff
--- smfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ smfcfg1.yaml        2024-03-31 15:10:15.020009181 +0900
@@ -24,32 +24,23 @@
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+  locality: loc1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.142 # the Node ID of this SMF
+    listenAddr: 192.168.0.142 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.142 # the IP/FQDN of N4 interface on this SMF (PFCP)
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.144 # the Node ID of this UPF
+        addr: 192.168.0.144 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -60,19 +51,10 @@
                   - cidr: 10.60.0.0/16
                 staticPools:
                   - cidr: 10.60.100.0/24
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
-                  - cidr: 10.61.0.0/16
-                staticPools:
-                  - cidr: 10.61.100.0/24
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.144
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -93,6 +75,7 @@
   urrPeriod: 10 # default usage report period in seconds
   urrThreshold: 1000 # default usage report threshold in bytes
   requestedUnit: 1000
+  ulcl: false
 logger: # log output setting
   enable: true # true or false
   level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```
- `free5gc/config/smfcfg2.yaml`
```diff
--- smfcfg.yaml.orig    2024-03-30 10:35:10.535609025 +0900
+++ smfcfg2.yaml        2024-03-31 15:16:28.761472010 +0900
@@ -6,8 +6,8 @@
   smfName: SMF # the name of this SMF
   sbi: # Service-based interface information
     scheme: http # the protocol for sbi (http or https)
-    registerIPv4: 127.0.0.2 # IP used to register to NRF
-    bindingIPv4: 127.0.0.2 # IP used to bind the service
+    registerIPv4: 127.0.0.12 # IP used to register to NRF
+    bindingIPv4: 127.0.0.12 # IP used to bind the service
     port: 8000 # Port used to bind the service
     tls: # the local path of TLS key
       key: cert/smf.key # SMF TLS Certificate
@@ -24,32 +24,23 @@
         - dnn: internet # Data Network Name
           dns: # the IP address of DNS
             ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
-    - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-        sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-        sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-      dnnInfos: # DNN information list
-        - dnn: internet # Data Network Name
-          dns: # the IP address of DNS
-            ipv4: 8.8.8.8
-            ipv6: 2001:4860:4860::8888
   plmnList: # the list of PLMN IDs that this SMF belongs to (optional, remove this key when unnecessary)
-    - mcc: 208 # Mobile Country Code (3 digits string, digit: 0~9)
-      mnc: 93 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
-  locality: area1 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
+    - mcc: 001 # Mobile Country Code (3 digits string, digit: 0~9)
+      mnc: 01 # Mobile Network Code (2 or 3 digits string, digit: 0~9)
+  locality: loc2 # Name of the location where a set of AMF, SMF, PCF and UPFs are located
   pfcp: # the IP address of N4 interface on this SMF (PFCP)
     # addr config is deprecated in smf config v1.0.3, please use the following config
-    nodeID: 127.0.0.1 # the Node ID of this SMF
-    listenAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
-    externalAddr: 127.0.0.1 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    nodeID: 192.168.0.143 # the Node ID of this SMF
+    listenAddr: 192.168.0.143 # the IP/FQDN of N4 interface on this SMF (PFCP)
+    externalAddr: 192.168.0.143 # the IP/FQDN of N4 interface on this SMF (PFCP)
   userplaneInformation: # list of userplane information
     upNodes: # information of userplane node (AN or UPF)
       gNB1: # the name of the node
         type: AN # the type of the node (AN or UPF)
       UPF: # the name of the node
         type: UPF # the type of the node (AN or UPF)
-        nodeID: 127.0.0.8 # the Node ID of this UPF
-        addr: 127.0.0.8 # the IP/FQDN of N4 interface on this UPF (PFCP)
+        nodeID: 192.168.0.145 # the Node ID of this UPF
+        addr: 192.168.0.145 # the IP/FQDN of N4 interface on this UPF (PFCP)
         sNssaiUpfInfos: # S-NSSAI information list for this UPF
           - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
               sst: 1 # Slice/Service Type (uinteger, range: 0~255)
@@ -57,22 +48,13 @@
             dnnUpfInfoList: # DNN information list for this S-NSSAI
               - dnn: internet
                 pools:
-                  - cidr: 10.60.0.0/16
-                staticPools:
-                  - cidr: 10.60.100.0/24
-          - sNssai: # S-NSSAI (Single Network Slice Selection Assistance Information)
-              sst: 1 # Slice/Service Type (uinteger, range: 0~255)
-              sd: 112233 # Slice Differentiator (3 bytes hex string, range: 000000~FFFFFF)
-            dnnUpfInfoList: # DNN information list for this S-NSSAI
-              - dnn: internet
-                pools:
                   - cidr: 10.61.0.0/16
                 staticPools:
                   - cidr: 10.61.100.0/24
         interfaces: # Interface list for this UPF
           - interfaceType: N3 # the type of the interface (N3 or N9)
             endpoints: # the IP address of this N3/N9 interface on this UPF
-              - 127.0.0.8
+              - 192.168.0.145
             networkInstances: # Data Network Name (DNN)
               - internet
     links: # the topology graph of userplane, A and B represent the two nodes of each link
@@ -93,6 +75,7 @@
   urrPeriod: 10 # default usage report period in seconds
   urrThreshold: 1000 # default usage report threshold in bytes
   requestedUnit: 1000
+  ulcl: false
 logger: # log output setting
   enable: true # true or false
   level: info # how detailed to output, value: trace, debug, info, warn, error, fatal, panic
```

<a id="changes_up1"></a>

### Changes in configuration files of free5GC 5GC U-Plane1

- `free5gc/config/upfcfg.yaml`
```diff
--- upfcfg.yaml.orig    2024-03-30 11:20:35.082415729 +0900
+++ upfcfg.yaml 2024-03-30 11:25:49.066076840 +0900
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
@@ -22,7 +22,7 @@
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
--- upfcfg.yaml.orig    2024-03-30 11:20:35.082415729 +0900
+++ upfcfg.yaml 2024-03-30 11:35:08.624145608 +0900
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
@@ -22,7 +22,7 @@
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
--- free5gc-ue.yaml.orig        2024-03-02 20:21:00.000000000 +0900
+++ free5gc-ue-loc1.yaml        2024-03-31 15:37:50.000000000 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
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
--- free5gc-ue.yaml.orig        2024-03-02 20:21:00.000000000 +0900
+++ free5gc-ue-loc2.yaml        2024-03-31 15:40:20.000000000 +0900
@@ -1,9 +1,9 @@
 # IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
-supi: 'imsi-208930000000001'
+supi: 'imsi-001010000000000'
 # Mobile Country Code value of HPLMN
-mcc: '208'
+mcc: '001'
 # Mobile Network Code value of HPLMN (2 or 3 digits)
-mnc: '93'
+mnc: '01'
 # SUCI Protection Scheme : 0 for Null-scheme, 1 for Profile A and 2 for Profile B
 protectionScheme: 0
 # Home Network Public Key for protecting with SUCI Profile A
@@ -28,7 +28,7 @@
 
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
- free5GC v3.4.1 (2024.03.28) - https://free5gc.org/guide/
- UERANSIM v3.2.6 (2024.03.08) - https://github.com/aligungr/UERANSIM/wiki/Installation

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
NF_LIST2="udr udm nssf ausf chf"

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
[2024-03-31 17:21:07.473] [sctp] [info] Trying to establish SCTP connection... (192.168.0.142:38412)
[2024-03-31 17:21:07.485] [sctp] [info] SCTP connection established (192.168.0.142:38412)
[2024-03-31 17:21:07.486] [sctp] [debug] SCTP association setup ascId[13]
[2024-03-31 17:21:07.487] [ngap] [debug] Sending NG Setup Request
[2024-03-31 17:21:07.494] [ngap] [debug] NG Setup Response received
[2024-03-31 17:21:07.494] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T17:21:07.435189031+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.131:44324
2024-03-31T17:21:07.437356526+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.131:44324
2024-03-31T17:21:07.440151656+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle NGSetupRequest
2024-03-31T17:21:07.440653975+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Send NG-Setup response
```

<a id="run_ran2"></a>

#### Start gNodeB2 with TAC=2 in Loc2

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2024-03-31 17:21:10.192] [sctp] [info] Trying to establish SCTP connection... (192.168.0.143:38412)
[2024-03-31 17:21:10.204] [sctp] [info] SCTP connection established (192.168.0.143:38412)
[2024-03-31 17:21:10.205] [sctp] [debug] SCTP association setup ascId[10]
[2024-03-31 17:21:10.205] [ngap] [debug] Sending NG Setup Request
[2024-03-31 17:21:10.213] [ngap] [debug] NG Setup Response received
[2024-03-31 17:21:10.213] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T17:21:10.443995653+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.132:33287
2024-03-31T17:21:10.446352828+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.132:33287
2024-03-31T17:21:10.449327897+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle NGSetupRequest
2024-03-31T17:21:10.450007938+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Send NG-Setup response
```

<a id="run_ue1"></a>

### Run UERANSIM (UE in Loc1)

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<a id="con_ue1"></a>

#### Start UE connected to gNodeB1 in Loc1

```
# ./nr-ue -c ../config/free5gc-ue-loc1.yaml 
UERANSIM v3.2.6
[2024-03-31 17:22:46.042] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 17:22:46.044] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 17:22:47.789] [nas] [info] Selected plmn[001/01]
[2024-03-31 17:22:47.789] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-31 17:22:47.790] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 17:22:47.791] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 17:22:47.791] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 17:22:47.795] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 17:22:47.796] [nas] [debug] Sending Initial Registration
[2024-03-31 17:22:47.796] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 17:22:47.798] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 17:22:47.799] [rrc] [info] RRC connection established
[2024-03-31 17:22:47.799] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 17:22:47.800] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 17:22:48.963] [nas] [debug] Authentication Request received
[2024-03-31 17:22:48.963] [nas] [debug] Received SQN [000000000033]
[2024-03-31 17:22:48.963] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 17:22:48.997] [nas] [debug] Security Mode Command received
[2024-03-31 17:22:48.997] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 17:22:49.160] [nas] [debug] Registration accept received
[2024-03-31 17:22:49.160] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 17:22:49.160] [nas] [debug] Sending Registration Complete
[2024-03-31 17:22:49.160] [nas] [info] Initial Registration is successful
[2024-03-31 17:22:49.160] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 17:22:49.161] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 17:22:49.371] [nas] [debug] Configuration Update Command received
[2024-03-31 17:22:49.584] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 17:22:49.587] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 17:22:49.613] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T17:22:47.428498763+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle InitialUEMessage
2024-03-31T17:22:47.429273579+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-03-31T17:22:47.430487534+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T17:22:47.433824543+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-03-31T17:22:47.434548433+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T17:22:47.435062337+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T17:22:47.435666317+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T17:22:47.436510813+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T17:22:47.437129672+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T17:22:47.437980393+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T17:22:47.440884757+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:47.447667879+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:47.460706705+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.479173250+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.482746415+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T17:22:48.487409960+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.493510175+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.508346173+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.515291723+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T17:22:48.516158392+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T17:22:48.518566897+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.522230595+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.530287575+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.533210760+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.536277752+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T17:22:48.537913479+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.539818125+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.547643855+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.550855456+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T17:22:48.552185087+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.554786370+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.560206140+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.560802231+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T17:22:48.561085172+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T17:22:48.561211807+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-03-31T17:22:48.563079388+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.565131374+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.568943798+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.570512532+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.571753464+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-03-31T17:22:48.574489385+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T17:22:48.578225072+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T17:22:48.579432609+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T17:22:48.582399903+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T17:22:48.584596306+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T17:22:48.585021806+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T17:22:48.585650930+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T17:22:48.585948233+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T17:22:48.586274365+09:00 [INFO][AUSF][5gAka] XresStar = 6161383039616264356133336366333537613237626465643939623733383439
2024-03-31T17:22:48.586543585+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T17:22:48.587063853+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T17:22:48.587429937+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Send Downlink Nas Transport
2024-03-31T17:22:48.588246457+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T17:22:48.590412836+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport
2024-03-31T17:22:48.590539809+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:22:48.590814563+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T17:22:48.591021661+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T17:22:48.591220045+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T17:22:48.592020190+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.593969943+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.597688839+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.599228713+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T17:22:48.599538187+09:00 [INFO][AUSF][5gAka] res*: 6161383039616264356133336366333537613237626465643939623733383439
Xres*: 6161383039616264356133336366333537613237626465643939623733383439
2024-03-31T17:22:48.600240436+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T17:22:48.600967711+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.602087145+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.606258020+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.607853448+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T17:22:48.608966815+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.612884896+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.616810037+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.618766340+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T17:22:48.619969839+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T17:22:48.620324980+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T17:22:48.620832796+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T17:22:48.621273012+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T17:22:48.621365379+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T17:22:48.621597440+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Send Downlink Nas Transport
2024-03-31T17:22:48.622245582+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T17:22:48.624008237+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport
2024-03-31T17:22:48.624169729+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:22:48.624427438+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T17:22:48.624642744+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T17:22:48.624853067+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T17:22:48.625110532+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T17:22:48.625329496+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T17:22:48.626319540+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.628559649+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.632286147+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.633811844+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.635448955+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T17:22:48.636277701+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.637963488+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.642929586+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.644781092+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T17:22:48.645641480+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.647190028+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.650869495+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.652484435+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T17:22:48.653254005+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T17:22:48.653727707+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:22:48.654249227+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-03-31T17:22:48.654315742+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-03-31T17:22:48.655338841+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.656965635+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.659877533+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.661063014+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.662321115+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T17:22:48.662987108+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.664230257+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.667675740+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.669167991+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T17:22:48.669691256+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T17:22:48.670328368+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.671636801+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.674985199+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.676605219+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T17:22:48.677610765+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T17:22:48.677901757+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T17:22:48.679444513+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.682874051+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.686362397+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.687793993+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T17:22:48.688429703+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.689442893+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.692701009+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.694030941+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T17:22:48.694681013+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:22:48.695023540+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:22:48.696283850+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.697668314+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.701059326+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.702414469+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T17:22:48.702938668+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.704167921+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.707219166+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.708518745+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T17:22:48.709140104+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T17:22:48.709494682+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:22:48.710716297+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.712004652+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.715459254+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.717798463+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T17:22:48.719023393+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.720314560+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.726024716+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.727492179+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T17:22:48.727976733+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T17:22:48.728575710+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T17:22:48.729594483+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.731388942+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.734898327+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.736483495+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T17:22:48.737386907+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.739363623+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.742949404+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.745137961+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T17:22:48.745471618+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T17:22:48.745869005+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T17:22:48.746789601+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.748515506+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.751489076+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.752626234+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.753912894+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T17:22:48.754585849+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.755793440+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.759325069+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.761529048+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T17:22:48.762119902+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.763598664+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.766259699+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.767200415+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.768067077+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T17:22:48.768555961+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.769757411+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.773421196+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.775102162+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T17:22:48.775827898+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T17:22:48.776837409+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:48.778175803+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:48.780733678+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:48.781643738+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:48.782483964+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T17:22:48.782877046+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T17:22:48.783356068+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T17:22:48.783571194+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Send Initial Context Setup Request
2024-03-31T17:22:48.784973462+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T17:22:48.785664248+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle InitialContextSetupResponse
2024-03-31T17:22:48.786066924+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T17:22:48.988744362+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport
2024-03-31T17:22:48.989456672+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:22:48.990183515+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T17:22:48.991065975+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T17:22:48.991650067+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T17:22:48.992207777+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T17:22:48.992790954+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Send Downlink Nas Transport
2024-03-31T17:22:48.995532980+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T17:22:48.998427875+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport
2024-03-31T17:22:48.999340540+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:22:49.002459168+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T17:22:49.006462160+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T17:22:49.007108990+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T17:22:49.008527422+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-03-31T17:22:49.011367304+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.017685420+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.027678557+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.030891913+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.033457023+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T17:22:49.034755646+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.038293625+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.045690490+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.049033826+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T17:22:49.049802218+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=94e12fe8-682a-4186-9f5d-5e12876294b7&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T17:22:49.052334881+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.054487292+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.060007721+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.061813354+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.063825221+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:22:49.064723579+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.067147238+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.071947413+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.074644349+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T17:22:49.075488470+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T17:22:49.075817818+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T17:22:49.076060306+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T17:22:49.077069483+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.078917244+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.083673654+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.085156566+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.087134338+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T17:22:49.088045354+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T17:22:49.090288758+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.092909796+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.096927351+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.099219345+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T17:22:49.099984146+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.101305922+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.104844129+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.105208830+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-03-31T17:22:49.106676889+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T17:22:49.107519929+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:22:49.108036127+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:22:49.108862401+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T17:22:49+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc000308280 0xc000308300]
2024-03-31T17:22:49.111165621+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T17:22:49.111371451+09:00 [INFO][SMF][GSM] &{[0xc000308280 0xc000308300]}
2024-03-31T17:22:49.111620560+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T17:22:49.112902534+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.114030110+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.116995738+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.118041103+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.119170495+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=94e12fe8-682a-4186-9f5d-5e12876294b7&target-nf-type=AMF |
2024-03-31T17:22:49.119665049+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T17:22:49.119939786+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-03-31T17:22:49.120203111+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T17:22:49.120351767+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2024-03-31T17:22:49.121144863+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.122141829+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.124650326+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.125701640+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.126780027+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T17:22:49.127438428+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.128368306+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.131729274+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.132873058+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T17:22:49.133629458+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.134798546+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.138017614+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.139217211+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T17:22:49.140378668+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:22:49.145180983+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.146608201+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.149791702+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.151081321+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T17:22:49.151760710+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-03-31T17:22:49.152208227+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T17:22:49.152941286+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.154420678+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.157548671+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.158939525+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T17:22:49.159185724+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T17:22:49.159690674+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.160855026+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.163624827+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.165039882+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.165626534+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T17:22:49.166411298+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T17:22:49.167574354+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T17:22:49.168513848+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.169609178+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.172106892+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.173132666+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:22:49.173916606+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T17:22:49.174422609+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T17:22:49.174781828+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.175774219+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.178731842+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.182806697+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T17:22:49.183076818+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T17:22:49.183386967+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T17:22:49.183559527+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T17:22:49.183725977+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T17:22:49.184129787+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T17:22:49.184764955+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T17:22:49.184897933+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-03-31T17:22:49.184921578+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T17:22:49.185007949+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-03-31T17:22:49.185247827+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T17:22:49.185441395+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has default path
2024-03-31T17:22:49.185688207+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T17:22:49.186191846+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T17:22:49.186967181+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T17:22:49.193281773+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T17:22:49.194853156+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.196297814+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.200581317+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.205414313+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T17:22:49.205687839+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Send PDU Session Resource Setup Request
2024-03-31T17:22:49.207627255+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T17:22:49.209912524+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:44324] Handle PDUSessionResourceSetupResponse
2024-03-31T17:22:49.210030238+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:44324] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T17:22:49.211022791+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:22:49.212625937+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:22:49.215894057+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:22:49.217583039+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T17:22:49.220161637+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T17:22:49.220423840+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:6ebe8df0-ae6f-4665-a634-046fedb2e379/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2024-03-31T17:22:49.578275802+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionEstablishmentRequest
2024-03-31T17:22:49.578309964+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T17:22:49.581723882+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T17:22:49.582192017+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T17:22:49.609953691+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
12: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::3727:f22f:2910:f3b8/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue1"></a>

#### Ping google.com going through DN=10.60.0.0/16 on Loc1

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.175.78) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.175.78: icmp_seq=1 ttl=61 time=19.6 ms
64 bytes from 172.217.175.78: icmp_seq=2 ttl=61 time=17.6 ms
64 bytes from 172.217.175.78: icmp_seq=3 ttl=61 time=18.3 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
17:27:28.379847 IP 10.60.0.1 > 172.217.175.78: ICMP echo request, id 8, seq 1, length 64
17:27:28.396670 IP 172.217.175.78 > 10.60.0.1: ICMP echo reply, id 8, seq 1, length 64
17:27:29.380470 IP 10.60.0.1 > 172.217.175.78: ICMP echo request, id 8, seq 2, length 64
17:27:29.395884 IP 172.217.175.78 > 10.60.0.1: ICMP echo reply, id 8, seq 2, length 64
17:27:30.381493 IP 10.60.0.1 > 172.217.175.78: ICMP echo request, id 8, seq 3, length 64
17:27:30.397405 IP 172.217.175.78 > 10.60.0.1: ICMP echo reply, id 8, seq 3, length 64
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
[2024-03-31 17:28:07.883] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 17:28:07.884] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 17:28:10.083] [nas] [error] PLMN selection failure, no cells in coverage
[2024-03-31 17:28:10.301] [nas] [info] Selected plmn[001/01]
[2024-03-31 17:28:10.383] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2024-03-31 17:28:10.383] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 17:28:10.384] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 17:28:10.385] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 17:28:10.385] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 17:28:10.386] [nas] [debug] Sending Initial Registration
[2024-03-31 17:28:10.387] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 17:28:10.387] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 17:28:10.389] [rrc] [info] RRC connection established
[2024-03-31 17:28:10.390] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 17:28:10.390] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 17:28:10.495] [nas] [debug] Authentication Request received
[2024-03-31 17:28:10.495] [nas] [debug] Received SQN [000000000034]
[2024-03-31 17:28:10.495] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 17:28:10.526] [nas] [debug] Security Mode Command received
[2024-03-31 17:28:10.526] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 17:28:10.747] [nas] [debug] Registration accept received
[2024-03-31 17:28:10.747] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 17:28:10.747] [nas] [debug] Sending Registration Complete
[2024-03-31 17:28:10.748] [nas] [info] Initial Registration is successful
[2024-03-31 17:28:10.748] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 17:28:10.748] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 17:28:10.951] [nas] [debug] Configuration Update Command received
[2024-03-31 17:28:11.103] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 17:28:11.109] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 17:28:11.132] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T17:28:10.030501982+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle InitialUEMessage
2024-03-31T17:28:10.032006981+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-03-31T17:28:10.032728333+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T17:28:10.036191923+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0100000001]
2024-03-31T17:28:10.037034779+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T17:28:10.037612901+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T17:28:10.038147646+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T17:28:10.038472401+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T17:28:10.039637886+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T17:28:10.040457472+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T17:28:10.043896840+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.049731939+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.063388579+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.067662878+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.070611520+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T17:28:10.072634073+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.076517495+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.084761208+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.088458455+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T17:28:10.088978481+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T17:28:10.090436079+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.092890562+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.098529933+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.100664300+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.102993347+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T17:28:10.104080900+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.105629292+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.111096217+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.113416643+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T17:28:10.114958730+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.117087943+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.122046341+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.122674045+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T17:28:10.123029928+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T17:28:10.123245313+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2024-03-31T17:28:10.124811135+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T17:28:10.125890516+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T17:28:10.126478238+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T17:28:10.127623149+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T17:28:10.129676127+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T17:28:10.130002613+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T17:28:10.130591369+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T17:28:10.130648181+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T17:28:10.130697661+09:00 [INFO][AUSF][5gAka] XresStar = 3961373262333566393864643964613761306535656130623665643831376433
2024-03-31T17:28:10.130758725+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T17:28:10.131032173+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T17:28:10.131144184+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Send Downlink Nas Transport
2024-03-31T17:28:10.131574800+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T17:28:10.135462856+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport
2024-03-31T17:28:10.135706866+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.135929456+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T17:28:10.136673730+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T17:28:10.136986248+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T17:28:10.137888241+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.139765604+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.143481317+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.145157027+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T17:28:10.145380246+09:00 [INFO][AUSF][5gAka] res*: 3961373262333566393864643964613761306535656130623665643831376433
Xres*: 3961373262333566393864643964613761306535656130623665643831376433
2024-03-31T17:28:10.145739452+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T17:28:10.146568801+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.147555663+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.151228848+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.152892457+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T17:28:10.153641779+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.155162935+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.158333864+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.159676408+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T17:28:10.160714866+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T17:28:10.161020667+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T17:28:10.161413969+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T17:28:10.161729405+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T17:28:10.161920804+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T17:28:10.162149418+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Send Downlink Nas Transport
2024-03-31T17:28:10.162709575+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T17:28:10.164394690+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport
2024-03-31T17:28:10.164571979+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.164794880+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T17:28:10.164942547+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T17:28:10.165127685+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T17:28:10.165311954+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T17:28:10.165456527+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T17:28:10.166202703+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.167719472+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.170516448+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.171512981+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.174305272+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T17:28:10.175526943+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.176866329+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.180148256+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.181556546+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T17:28:10.182186744+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.183628203+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.186766761+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.188116135+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T17:28:10.188823197+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T17:28:10.189158182+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:28:10.189686638+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-03-31T17:28:10.189736940+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-03-31T17:28:10.190794391+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.192146985+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.194775895+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.195755371+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.196976285+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T17:28:10.197502583+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.198808616+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.202104511+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.204078057+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T17:28:10.204296487+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T17:28:10.204966278+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.206114529+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.209272021+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.210556744+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T17:28:10.211672248+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T17:28:10.212036008+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T17:28:10.212255161+09:00 [INFO][UDM][UECM] Send DeregNotify to old AMF GUAMI=&{0xc0001eeec0 cafe00}
2024-03-31T17:28:10.213997104+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.214852674+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.216769409+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.220134794+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.221041023+09:00 [INFO][AMF][Callback] Handle Deregistration Notification
2024-03-31T17:28:10.222368397+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.225647599+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.227063414+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T17:28:10.227899103+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.229382686+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.232531110+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.234210657+09:00 [INFO][SMF][PduSess] Receive Release SM Context Request
2024-03-31T17:28:10.234672785+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextRelease
2024-03-31T17:28:10.235500939+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.236789919+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.241041457+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.242741552+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.244606858+09:00 [INFO][PCF][SMpolicy] Handle DeleteSmPolicyContext
2024-03-31T17:28:10.245286246+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.246684322+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.249742265+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.251044948+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifySubscriptionIdDelete
2024-03-31T17:28:10.251382017+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | DELETE  | /nudr-dr/v1/application-data/influenceData/subs-to-notify/23dbd9fc |
2024-03-31T17:28:10.252452554+09:00 [INFO][PCF][GIN] | 204 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies/imsi-001010000000000-1/delete |
2024-03-31T17:28:10.252828531+09:00 [WARN][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Unexpected state, expect: [InActive], actual:[Active]
2024-03-31T17:28:10.252883087+09:00 [INFO][SMF][PduSess] Sending PFCP Session Deletion Request
2024-03-31T17:28:10.254002592+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.257230842+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.258832229+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T17:28:10.259500297+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:28:10.259942655+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:28:10.260141061+09:00 [INFO][SMF][PduSess] Received PFCP Session Deletion Accepted Response
2024-03-31T17:28:10.260974396+09:00 [INFO][SMF][Charging] build MultiUnitUsageFromUsageReport
2024-03-31T17:28:10.260999779+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T17:28:10.261769186+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.263299607+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.262141766+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.264969030+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.268264725+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.269777120+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T17:28:10.272614461+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.274613043+09:00 [INFO][CHF][ChargingPost] HandleChargingdateRelease
2024-03-31T17:28:10.274818950+09:00 [INFO][CHF][ChargingPost] Credit Control are not required for rating group: 1
2024-03-31T17:28:10.274990340+09:00 [INFO][CHF][ChargingPost] Credit Control are not required for rating group: 2
2024-03-31T17:28:10.275142517+09:00 [INFO][CHF][ChargingPost] Close CDR
2024-03-31T17:28:10.275589754+09:00 [INFO][CHF][GIN] | 204 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata/imsi-001010000000000SMF0/release |
2024-03-31T17:28:10.275893782+09:00 [INFO][SMF][Charging] Send Charging Data Request[Termination] successfully
2024-03-31T17:28:10.276136339+09:00 [INFO][SMF][GIN] | 204 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:6ebe8df0-ae6f-4665-a634-046fedb2e379/release |
2024-03-31T17:28:10.276801858+09:00 [INFO][SMF][PduSess] UE[imsi-001010000000000] PDUSessionID[1] Release IP[10.60.0.1]
2024-03-31T17:28:10.276518026+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.278705117+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.278895007+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] smContext[urn:uuid:6ebe8df0-ae6f-4665-a634-046fedb2e379] is deleted from pool
2024-03-31T17:28:10.282257515+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.277510052+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.283842486+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.285293969+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T17:28:10.285965955+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T17:28:10.286470289+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:28:10.290914532+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.287449063+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.292411552+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.295753025+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.297541087+09:00 [INFO][UDM][SDM] Handle Unsubscribe
2024-03-31T17:28:10.298371509+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T17:28:10.299151002+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.300325442+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.303629199+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.304942013+09:00 [INFO][UDR][DataRepo] Handle RemovesdmSubscriptions
2024-03-31T17:28:10.305119261+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | DELETE  | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions/1 |
2024-03-31T17:28:10.305479505+09:00 [INFO][UDM][GIN] | 204 |       127.0.0.1 | DELETE  | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions/1 |
2024-03-31T17:28:10.307053392+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.308283945+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.311942255+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.308630120+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.313195104+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.314527749+09:00 [INFO][PCF][AmPol] Handle AM Policy Association Delete
2024-03-31T17:28:10.314990485+09:00 [INFO][PCF][GIN] | 204 |       127.0.0.1 | DELETE  | /npcf-am-policy-control/v1/policies/imsi-001010000000000-1 |
2024-03-31T17:28:10.315314324+09:00 [INFO][AMF][CTX] AmfUe[imsi-001010000000000] is removed
2024-03-31T17:28:10.315501738+09:00 [INFO][AMF][GIN] | 204 |       127.0.0.1 | POST    | /namf-callback/v1/deregistration/imsi-001010000000000 |
2024-03-31T17:28:10.318980050+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.320487992+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T17:28:10.320985238+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T17:28:10.321366618+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T17:28:10.322600317+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.326007308+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.329621019+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.331114735+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T17:28:10.331939746+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.333083802+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.336255607+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.337630970+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T17:28:10.337942660+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T17:28:10.338634065+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T17:28:10.339775114+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.341463112+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.343962275+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.345003756+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.346233296+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T17:28:10.346891823+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.348095171+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.351480375+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.353514700+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T17:28:10.354288765+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.355795043+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.358574130+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.359573297+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.360459367+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T17:28:10.360909808+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.362165702+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.365104720+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.366542374+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T17:28:10.367048001+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T17:28:10.367551453+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.368745506+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.371513632+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.372819405+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.374092427+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe01%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T17:28:10.374636212+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.375830614+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.379449974+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.380882969+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-03-31T17:28:10.381244548+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2024-03-31T17:28:10.381500133+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-03-31T17:28:10.381978915+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T17:28:10.382483486+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T17:28:10.382554968+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Send Initial Context Setup Request
2024-03-31T17:28:10.383699151+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T17:28:10.384613807+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle InitialContextSetupResponse
2024-03-31T17:28:10.384805386+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.586179309+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport
2024-03-31T17:28:10.586415500+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.586613041+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T17:28:10.586766561+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T17:28:10.586982041+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T17:28:10.587152211+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T17:28:10.587375077+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Send Downlink Nas Transport
2024-03-31T17:28:10.588087370+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T17:28:10.588956783+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport
2024-03-31T17:28:10.589124063+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.589349974+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T17:28:10.589515436+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T17:28:10.589672990+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T17:28:10.589881411+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-03-31T17:28:10.590645613+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.592223410+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.594817804+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.596126335+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.597002013+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T17:28:10.597655870+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.599763492+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.604026367+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.606158960+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T17:28:10.606431015+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=107eb070-ccad-42ea-95fd-1c81ad9fdc81&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T17:28:10.607567745+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.609019787+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.611819462+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.612910561+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.613992760+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc2&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T17:28:10.614612912+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.615785466+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.619034251+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.620890072+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T17:28:10.621705612+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T17:28:10.621983570+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T17:28:10.622133362+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T17:28:10.622853438+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.624088544+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.626720710+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.627788267+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.628826916+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T17:28:10.629510237+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T17:28:10.629936887+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.630937458+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.634231710+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.635723551+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T17:28:10.636307442+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.637599423+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.640677527+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.641169464+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-03-31T17:28:10.642505633+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T17:28:10.643488358+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:28:10.643989025+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:28:10.644670680+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T17:28:10+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0001d87a0 0xc0001d87c0]
2024-03-31T17:28:10.645150964+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T17:28:10.645398957+09:00 [INFO][SMF][GSM] &{[0xc0001d87a0 0xc0001d87c0]}
2024-03-31T17:28:10.645620217+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T17:28:10.646511169+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.647904512+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.650726788+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.652020094+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.653457332+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=107eb070-ccad-42ea-95fd-1c81ad9fdc81&target-nf-type=AMF |
2024-03-31T17:28:10.653894116+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T17:28:10.654198932+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2024-03-31T17:28:10.654434047+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T17:28:10.654599633+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.61.0.1]
2024-03-31T17:28:10.654983925+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.655810610+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.658396839+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.659712486+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.661050302+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T17:28:10.661622828+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.662646247+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.665963616+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.667304991+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T17:28:10.668105422+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.669483908+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.672448624+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.673947588+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T17:28:10.674992058+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T17:28:10.679492027+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.680714367+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.684217997+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.685508646+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T17:28:10.686146796+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-03-31T17:28:10.686482329+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T17:28:10.687345044+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.688836000+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.692048395+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.693316700+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T17:28:10.693754089+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T17:28:10.695158266+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.698811286+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.701525962+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.702799241+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.703339156+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T17:28:10.704238699+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T17:28:10.705173192+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T17:28:10.705980516+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.707111015+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.709754444+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.710842954+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T17:28:10.711714939+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T17:28:10.712037894+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T17:28:10.712475589+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.713433053+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.716511319+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.718838429+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T17:28:10.719032840+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T17:28:10.719368713+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T17:28:10.719526297+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T17:28:10.719742272+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T17:28:10.719937777+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T17:28:10.720508294+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T17:28:10.720778323+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-03-31T17:28:10.721106976+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T17:28:10.721305980+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-03-31T17:28:10.721522234+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T17:28:10.721735043+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has default path
2024-03-31T17:28:10.722094397+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T17:28:10.722614713+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T17:28:10.723694879+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T17:28:10.729139583+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T17:28:10.730857134+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.732298689+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.735761527+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.737527344+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T17:28:10.737856362+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Send PDU Session Resource Setup Request
2024-03-31T17:28:10.738843721+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T17:28:10.741217876+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:33287] Handle PDUSessionResourceSetupResponse
2024-03-31T17:28:10.741393723+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:33287] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T17:28:10.742102168+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T17:28:10.743812441+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T17:28:10.747123329+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T17:28:10.748987209+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T17:28:10.752437216+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T17:28:10.752682916+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:d3839048-767e-4e0e-a320-6ec66185c9a3/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2024-03-31T17:28:11.178757538+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionEstablishmentRequest
2024-03-31T17:28:11.178855662+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805][CPNodeID:192.168.0.143][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T17:28:11.180938379+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T17:28:11.181397047+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T17:28:11.205417787+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
13: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::48d:7441:8fe1:347e/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue2"></a>

#### Ping google.com going through DN=10.61.0.0/16 on Loc2

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.42.142) from 10.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.42.142: icmp_seq=1 ttl=61 time=19.0 ms
64 bytes from 142.251.42.142: icmp_seq=2 ttl=61 time=21.1 ms
64 bytes from 142.251.42.142: icmp_seq=3 ttl=61 time=39.3 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
17:34:07.473338 IP 10.61.0.1 > 142.251.42.142: ICMP echo request, id 9, seq 1, length 64
17:34:07.490112 IP 142.251.42.142 > 10.61.0.1: ICMP echo reply, id 9, seq 1, length 64
17:34:08.473606 IP 10.61.0.1 > 142.251.42.142: ICMP echo request, id 9, seq 2, length 64
17:34:08.492420 IP 142.251.42.142 > 10.61.0.1: ICMP echo reply, id 9, seq 2, length 64
17:34:09.474399 IP 10.61.0.1 > 142.251.42.142: ICMP echo request, id 9, seq 3, length 64
17:34:09.511404 IP 142.251.42.142 > 10.61.0.1: ICMP echo reply, id 9, seq 3, length 64
```
**Note. Make sure the packet does not go through U-Plane1. The UE connects to the DN of U-Plane2 in the same Loc2 according to the connected gNodeB2 in Loc2.**

---
I was able to confirm the very simple configuration in which one UE connects to the UPF in the same location according connected gNodeB. I would like to thank the excellent developers and all the contributors of free5GC and UERANSIM.

<a id="changelog"></a>

## Changelog (summary)

- [2024.03.31] Updated to free5GC v3.4.1 (2024.03.28).
- [2022.08.11] Updated to free5GC v3.2.1.
- [2022.04.04] Updated to free5GC v3.1.0 and UERANSIM v3.2.6.
- [2021.09.25] Updated to free5GC v3.0.6.
- [2021.08.17] Initial release.
