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
[2024-03-31 16:22:50.633] [sctp] [info] Trying to establish SCTP connection... (192.168.0.142:38412)
[2024-03-31 16:22:50.645] [sctp] [info] SCTP connection established (192.168.0.142:38412)
[2024-03-31 16:22:50.646] [sctp] [debug] SCTP association setup ascId[10]
[2024-03-31 16:22:50.646] [ngap] [debug] Sending NG Setup Request
[2024-03-31 16:22:50.653] [ngap] [debug] NG Setup Response received
[2024-03-31 16:22:50.653] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T16:22:50.495395200+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.131:56800
2024-03-31T16:22:50.497435442+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.131:56800
2024-03-31T16:22:50.499787182+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle NGSetupRequest
2024-03-31T16:22:50.500321834+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Send NG-Setup response
```

<a id="run_ran2"></a>

#### Start gNodeB2 with TAC=2 in Loc2

```
# ./nr-gnb -c ../config/free5gc-gnb.yaml
UERANSIM v3.2.6
[2024-03-31 16:22:53.585] [sctp] [info] Trying to establish SCTP connection... (192.168.0.143:38412)
[2024-03-31 16:22:53.589] [sctp] [info] SCTP connection established (192.168.0.143:38412)
[2024-03-31 16:22:53.589] [sctp] [debug] SCTP association setup ascId[7]
[2024-03-31 16:22:53.589] [ngap] [debug] Sending NG Setup Request
[2024-03-31 16:22:53.592] [ngap] [debug] NG Setup Response received
[2024-03-31 16:22:53.592] [ngap] [info] NG Setup procedure is successful
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T16:22:53.831920646+09:00 [INFO][AMF][Ngap] [AMF] SCTP Accept from: 192.168.0.132:43308
2024-03-31T16:22:53.832794813+09:00 [INFO][AMF][Ngap] Create a new NG connection for: 192.168.0.132:43308
2024-03-31T16:22:53.833518931+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle NGSetupRequest
2024-03-31T16:22:53.833762310+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Send NG-Setup response
```

<a id="run_ue1"></a>

### Run UERANSIM (UE in Loc1)

Confirm that the packet goes through the DN of U-Plane1 in the same Loc1 by connecting to gNodeB1 in Loc1.

<a id="con_ue1"></a>

#### Start UE connected to gNodeB1 in Loc1

```
# ./nr-ue -c ../config/free5gc-ue-loc1.yaml 
UERANSIM v3.2.6
[2024-03-31 16:23:58.882] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 16:23:58.884] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 16:24:00.296] [nas] [info] Selected plmn[001/01]
[2024-03-31 16:24:00.297] [rrc] [info] Selected cell plmn[001/01] tac[1] category[SUITABLE]
[2024-03-31 16:24:00.297] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 16:24:00.298] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 16:24:00.298] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 16:24:00.299] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 16:24:00.299] [nas] [debug] Sending Initial Registration
[2024-03-31 16:24:00.300] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 16:24:00.301] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 16:24:00.303] [rrc] [info] RRC connection established
[2024-03-31 16:24:00.303] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 16:24:00.304] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 16:24:00.439] [nas] [debug] Authentication Request received
[2024-03-31 16:24:00.439] [nas] [debug] Received SQN [00000000002F]
[2024-03-31 16:24:00.439] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 16:24:00.472] [nas] [debug] Security Mode Command received
[2024-03-31 16:24:00.473] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 16:24:00.638] [nas] [debug] Registration accept received
[2024-03-31 16:24:00.638] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 16:24:00.638] [nas] [debug] Sending Registration Complete
[2024-03-31 16:24:00.639] [nas] [info] Initial Registration is successful
[2024-03-31 16:24:00.639] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 16:24:00.639] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 16:24:00.852] [nas] [debug] Configuration Update Command received
[2024-03-31 16:24:01.054] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 16:24:01.057] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 16:24:01.086] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.60.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T16:24:00.486586229+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle InitialUEMessage
2024-03-31T16:24:00.487931802+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-03-31T16:24:00.488863200+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T16:24:00.492133745+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-03-31T16:24:00.493044634+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T16:24:00.493659173+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T16:24:00.494546141+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T16:24:00.495090587+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T16:24:00.496089984+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T16:24:00.496908272+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T16:24:00.500136867+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.507008350+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.518601767+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.532870318+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.539680547+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T16:24:00.542217718+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.546538309+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.556836198+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.561437735+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T16:24:00.562040655+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T16:24:00.563545797+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.566564188+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.572798039+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.575072800+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.577297292+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T16:24:00.578653801+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.580296744+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.586248663+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.589480628+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T16:24:00.590462303+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.592442760+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.597395407+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.598110510+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T16:24:00.598528606+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T16:24:00.598766125+09:00 [INFO][UDM][Suci] SUPI type is IMSI
http://127.0.0.10:8000
2024-03-31T16:24:00.599451108+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.601158764+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.604524285+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.606407923+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.607432828+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=UDM&target-nf-type=UDR |
2024-03-31T16:24:00.609639818+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T16:24:00.611590268+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T16:24:00.612602836+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T16:24:00.613785937+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T16:24:00.615963594+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T16:24:00.616457463+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T16:24:00.616779131+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T16:24:00.616818884+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T16:24:00.616835306+09:00 [INFO][AUSF][5gAka] XresStar = 3966613066616465333838343431623933613666653132323039613366323230
2024-03-31T16:24:00.616971434+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T16:24:00.617453979+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T16:24:00.617669858+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Send Downlink Nas Transport
2024-03-31T16:24:00.618119007+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T16:24:00.620128638+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport
2024-03-31T16:24:00.620347971+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:24:00.620550108+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T16:24:00.620766930+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T16:24:00.620930571+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T16:24:00.621654452+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.624024043+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.628351751+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.631255275+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T16:24:00.631584382+09:00 [INFO][AUSF][5gAka] res*: 3966613066616465333838343431623933613666653132323039613366323230
Xres*: 3966613066616465333838343431623933613666653132323039613366323230
2024-03-31T16:24:00.632381274+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T16:24:00.633032007+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.634379109+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.638320906+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.640248275+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T16:24:00.640926195+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.642402445+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.645982599+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.647511120+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T16:24:00.648842460+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T16:24:00.649491721+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T16:24:00.649886105+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T16:24:00.650299734+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T16:24:00.650522662+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T16:24:00.650776551+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Send Downlink Nas Transport
2024-03-31T16:24:00.651742455+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T16:24:00.653577772+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport
2024-03-31T16:24:00.653761645+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:24:00.653963448+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T16:24:00.654145361+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T16:24:00.654375941+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T16:24:00.654566833+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T16:24:00.654731070+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T16:24:00.655530542+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.657216010+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.660566935+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.661671080+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.663014629+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T16:24:00.663609371+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.664956494+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.668696419+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.670166648+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T16:24:00.671112575+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.672527465+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.675863840+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.677437938+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T16:24:00.678259347+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T16:24:00.678693218+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:24:00.679116902+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-03-31T16:24:00.679501886+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-03-31T16:24:00.680532441+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.682096222+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.684978837+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.686128868+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.687401765+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T16:24:00.688024697+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.689281323+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.692927137+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.694345798+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T16:24:00.694664460+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T16:24:00.695324446+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.696581759+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.699621511+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.701404961+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T16:24:00.702472311+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T16:24:00.702783312+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T16:24:00.704244016+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.705635931+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.709137656+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.710599952+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T16:24:00.711361612+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.713339449+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.717526531+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.719250458+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T16:24:00.720213659+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:24:00.720779366+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:24:00.721838295+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.723480075+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.726742553+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.728084255+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T16:24:00.729113136+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.730388632+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.733621277+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.734993744+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T16:24:00.735727672+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T16:24:00.736054370+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:24:00.736923930+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.738252497+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.742072607+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.743340939+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T16:24:00.744272908+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.745474537+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.748707223+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.750088489+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T16:24:00.750724531+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T16:24:00.751402612+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T16:24:00.753323986+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.754786266+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.759172869+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.761112110+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T16:24:00.761958767+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.763271198+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.766297154+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.767807781+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T16:24:00.768085770+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T16:24:00.768513105+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T16:24:00.769225242+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.770621299+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.773517637+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.774604861+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.775902258+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T16:24:00.776605323+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.777774454+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.781164224+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.783376871+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T16:24:00.784037675+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.785299196+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.788135102+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.789166556+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.790003450+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T16:24:00.790523737+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.792366593+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.796394735+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.798704803+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T16:24:00.799365178+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T16:24:00.800357601+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.801915562+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.804497953+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.805430194+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:00.807264909+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T16:24:00.807929120+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:00.809174116+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:00.812541209+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:00.814230589+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-03-31T16:24:00.814515127+09:00 [INFO][AMF][Comm] new AMF Status Subscription[1]
2024-03-31T16:24:00.814711507+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-03-31T16:24:00.815186181+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T16:24:00.815613706+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T16:24:00.815893052+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Send Initial Context Setup Request
2024-03-31T16:24:00.817315618+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T16:24:00.818139678+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle InitialContextSetupResponse
2024-03-31T16:24:00.818329701+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T16:24:01.025658684+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport
2024-03-31T16:24:01.026463802+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:24:01.027085738+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T16:24:01.027624827+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T16:24:01.028042795+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T16:24:01.028675399+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T16:24:01.029128604+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Send Downlink Nas Transport
2024-03-31T16:24:01.031280584+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T16:24:01.033484022+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport
2024-03-31T16:24:01.034092573+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:24:01.034668085+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T16:24:01.035723826+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T16:24:01.036264682+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T16:24:01.037400749+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-03-31T16:24:01.039722528+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.044793384+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.053899553+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.057460247+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.060094697+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T16:24:01.061810842+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.065164187+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.073002353+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.076650944+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T16:24:01.077427278+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=3061c72e-af95-4794-b836-a4af1c4ef127&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T16:24:01.079904780+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.082705668+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.088035733+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.089980069+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.091985716+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc1&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:24:01.092919625+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.094917616+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.100082378+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.102623955+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T16:24:01.103604929+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T16:24:01.103931891+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T16:24:01.104258106+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T16:24:01.105154821+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.106770220+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.110499668+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.111853457+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.113394093+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T16:24:01.113906252+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T16:24:01.114558508+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.115754440+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.119995118+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.121789441+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T16:24:01.122683719+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.124226962+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.127703805+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.128202776+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-03-31T16:24:01.129846781+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T16:24:01.131087285+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:24:01.131578175+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:24:01.132196407+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T16:24:01+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0001d87a0 0xc0001d87c0]
2024-03-31T16:24:01.132465906+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T16:24:01.132636102+09:00 [INFO][SMF][GSM] &{[0xc0001d87a0 0xc0001d87c0]}
2024-03-31T16:24:01.132836803+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T16:24:01.133311154+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.134379247+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.137381778+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.138618957+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.141975995+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=3061c72e-af95-4794-b836-a4af1c4ef127&target-nf-type=AMF |
2024-03-31T16:24:01.142848561+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T16:24:01.143114851+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.60.0.1
2024-03-31T16:24:01.143296474+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T16:24:01.143758781+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.60.0.1]
2024-03-31T16:24:01.144696526+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.145827460+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.148466892+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.149555745+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.150753494+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc1&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T16:24:01.151435404+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.152412376+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.155949078+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.157112829+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T16:24:01.158178023+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.159446472+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.162576094+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.163874839+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T16:24:01.165176294+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:24:01.170444160+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.171873835+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.174871462+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.176209102+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T16:24:01.176855900+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-03-31T16:24:01.177222924+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T16:24:01.178039109+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.179383314+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.182520597+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.183948380+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T16:24:01.184249857+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T16:24:01.185289213+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.186598576+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.189233316+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.190237739+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.190862655+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T16:24:01.191587471+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T16:24:01.192530924+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T16:24:01.193127796+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.194136764+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.196800612+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.197806740+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:24:01.198680812+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T16:24:01.199163428+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T16:24:01.199499643+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.200462244+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.203586639+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.207289915+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T16:24:01.207561392+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T16:24:01.207806699+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T16:24:01.208165225+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T16:24:01.208340723+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T16:24:01.208760249+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T16:24:01.209531267+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T16:24:01.209767324+09:00 [INFO][SMF][CTX] No Default Data Path
2024-03-31T16:24:01.209968557+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-03-31T16:24:01.210187003+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T16:24:01.210440383+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-03-31T16:24:01.210612124+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T16:24:01.210815407+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has default path
2024-03-31T16:24:01.211095065+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T16:24:01.211339679+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T16:24:01.212244817+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T16:24:01.218243383+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T16:24:01.221201484+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.223709712+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.227279369+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.229745015+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T16:24:01.231442392+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Send PDU Session Resource Setup Request
2024-03-31T16:24:01.232472814+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T16:24:01.234632503+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.131:56800] Handle PDUSessionResourceSetupResponse
2024-03-31T16:24:01.234793192+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.131:56800] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T16:24:01.235514851+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:24:01.237065098+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:24:01.240402900+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:24:01.242190421+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T16:24:01.245510051+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T16:24:01.245745776+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:790d284e-71bb-415f-a529-19cbe3895c0b/modify |
```
The free5GC U-Plane1 log when executed is as follows.
```
2024-03-31T16:24:00.855387123+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionEstablishmentRequest
2024-03-31T16:24:00.855449565+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805][CPNodeID:192.168.0.142][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T16:24:00.857923482+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T16:24:00.859232306+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T16:24:00.886287134+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.144:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
8: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.60.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::e613:f56e:bc0d:d5b2/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue1"></a>

#### Ping google.com going through DN=10.60.0.0/16 on Loc1

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane1.
```
# ping google.com -I uesimtun0 -n
PING google.com (172.217.26.238) from 10.60.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 172.217.26.238: icmp_seq=1 ttl=61 time=18.9 ms
64 bytes from 172.217.26.238: icmp_seq=2 ttl=61 time=20.3 ms
64 bytes from 172.217.26.238: icmp_seq=3 ttl=61 time=18.4 ms
```
The `tcpdump` log on U-Plane1 is as follows.
```
16:27:15.729363 IP 10.60.0.1 > 172.217.26.238: ICMP echo request, id 6, seq 1, length 64
16:27:15.747611 IP 172.217.26.238 > 10.60.0.1: ICMP echo reply, id 6, seq 1, length 64
16:27:16.731447 IP 10.60.0.1 > 172.217.26.238: ICMP echo request, id 6, seq 2, length 64
16:27:16.749501 IP 172.217.26.238 > 10.60.0.1: ICMP echo reply, id 6, seq 2, length 64
16:27:17.731321 IP 10.60.0.1 > 172.217.26.238: ICMP echo request, id 6, seq 3, length 64
16:27:17.748989 IP 172.217.26.238 > 10.60.0.1: ICMP echo reply, id 6, seq 3, length 64
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
[2024-03-31 16:28:13.072] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2024-03-31 16:28:13.074] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2024-03-31 16:28:13.698] [nas] [info] Selected plmn[001/01]
[2024-03-31 16:28:13.698] [rrc] [info] Selected cell plmn[001/01] tac[2] category[SUITABLE]
[2024-03-31 16:28:13.698] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2024-03-31 16:28:13.698] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2024-03-31 16:28:13.698] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2024-03-31 16:28:13.701] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 16:28:13.702] [nas] [debug] Sending Initial Registration
[2024-03-31 16:28:13.702] [rrc] [debug] Sending RRC Setup Request
[2024-03-31 16:28:13.702] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2024-03-31 16:28:13.702] [rrc] [info] RRC connection established
[2024-03-31 16:28:13.702] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2024-03-31 16:28:13.703] [nas] [info] UE switches to state [CM-CONNECTED]
[2024-03-31 16:28:13.744] [nas] [debug] Authentication Request received
[2024-03-31 16:28:13.744] [nas] [debug] Received SQN [000000000030]
[2024-03-31 16:28:13.744] [nas] [debug] SQN-MS [000000000000]
[2024-03-31 16:28:13.772] [nas] [debug] Security Mode Command received
[2024-03-31 16:28:13.772] [nas] [debug] Selected integrity[2] ciphering[0]
[2024-03-31 16:28:13.933] [nas] [debug] Registration accept received
[2024-03-31 16:28:13.933] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2024-03-31 16:28:13.934] [nas] [debug] Sending Registration Complete
[2024-03-31 16:28:13.934] [nas] [info] Initial Registration is successful
[2024-03-31 16:28:13.934] [nas] [debug] Sending PDU Session Establishment Request
[2024-03-31 16:28:13.934] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2024-03-31 16:28:14.149] [nas] [debug] Configuration Update Command received
[2024-03-31 16:28:14.352] [nas] [debug] PDU Session Establishment Accept received
[2024-03-31 16:28:14.357] [nas] [info] PDU Session establishment is successful PSI[1]
[2024-03-31 16:28:14.397] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 10.61.0.1] is up.
```
The free5GC C-Plane log when executed is as follows.
```
2024-03-31T16:28:13.880601319+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle InitialUEMessage
2024-03-31T16:28:13.880640122+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] New RanUe [RanUeNgapID:1][AmfUeNgapID:1]
2024-03-31T16:28:13.880676158+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] 5GSMobileIdentity ["SUCI":"suci-0-001-01-0000-0-0-0000000000", err: <nil>]
2024-03-31T16:28:13.881184741+09:00 [INFO][AMF][CTX] New AmfUe [supi:][guti:00101cafe0000000001]
2024-03-31T16:28:13.881208507+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Deregistered] to [Deregistered]
2024-03-31T16:28:13.881215896+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Registration Request
2024-03-31T16:28:13.881223062+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] RegistrationType: Initial Registration
2024-03-31T16:28:13.881230917+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] MobileIdentity5GS: SUCI[suci-0-001-01-0000-0-0-0000000000]
2024-03-31T16:28:13.881240909+09:00 [INFO][AMF][Gmm] Handle event[Start Authentication], transition from [Deregistered] to [Authentication]
2024-03-31T16:28:13.881247428+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Authentication procedure
2024-03-31T16:28:13.882248304+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.883685905+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.886982057+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.887976371+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:13.888776242+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=AUSF |
2024-03-31T16:28:13.889268150+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.890523044+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.893504709+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.895008501+09:00 [INFO][AUSF][UeAuth] HandleUeAuthPostRequest
2024-03-31T16:28:13.895299173+09:00 [INFO][AUSF][UeAuth] Serving network authorized
2024-03-31T16:28:13.895958961+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.896801136+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.899424637+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.900503974+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:13.901661204+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AUSF&service-names=nudm-ueau&target-nf-type=UDM |
2024-03-31T16:28:13.902149233+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.903031618+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.906346425+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.907876104+09:00 [INFO][UDM][UEAU] Handle GenerateAuthDataRequest
2024-03-31T16:28:13.908443509+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.909829603+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.912908920+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.913323113+09:00 [INFO][UDM][Suci] suciPart: [suci 0 001 01 0000 0 0 0000000000]
2024-03-31T16:28:13.913491708+09:00 [INFO][UDM][Suci] scheme 0
2024-03-31T16:28:13.913684934+09:00 [INFO][UDM][Suci] SUPI type is IMSI
2024-03-31T16:28:13.914941226+09:00 [INFO][UDR][DataRepo] Handle QueryAuthSubsData
2024-03-31T16:28:13.915623247+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T16:28:13.916016469+09:00 [INFO][UDM][UEAU] Nil Op
2024-03-31T16:28:13.916894495+09:00 [INFO][UDR][DataRepo] Handle ModifyAuthentication
2024-03-31T16:28:13.918693980+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PATCH   | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-subscription |
2024-03-31T16:28:13.919152736+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | POST    | /nudm-ueau/v1/suci-0-001-01-0000-0-0-0000000000/security-information/generate-auth-data |
2024-03-31T16:28:13.919556503+09:00 [INFO][AUSF][UeAuth] Add SuciSupiPair (suci-0-001-01-0000-0-0-0000000000, imsi-001010000000000) to map.
2024-03-31T16:28:13.919600724+09:00 [INFO][AUSF][UeAuth] Use 5G AKA auth method
2024-03-31T16:28:13.919623995+09:00 [INFO][AUSF][5gAka] XresStar = 3865646362366566326262353661623739303361303663303863646537323237
2024-03-31T16:28:13.919675129+09:00 [INFO][AUSF][GIN] | 201 |       127.0.0.1 | POST    | /nausf-auth/v1/ue-authentications |
2024-03-31T16:28:13.919972812+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Send Authentication Request
2024-03-31T16:28:13.920051795+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Send Downlink Nas Transport
2024-03-31T16:28:13.920508500+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Start T3560 timer
2024-03-31T16:28:13.923483590+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport
2024-03-31T16:28:13.923755290+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:28:13.923946368+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Authentication] to [Authentication]
2024-03-31T16:28:13.924126111+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Handle Authentication Response
2024-03-31T16:28:13.924298926+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:] Stop T3560 timer
2024-03-31T16:28:13.924953335+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.926584912+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.929797976+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.931416952+09:00 [INFO][AUSF][5gAka] Auth5gAkaComfirmRequest
2024-03-31T16:28:13.931623701+09:00 [INFO][AUSF][5gAka] res*: 3865646362366566326262353661623739303361303663303863646537323237
Xres*: 3865646362366566326262353661623739303361303663303863646537323237
2024-03-31T16:28:13.932140862+09:00 [INFO][AUSF][5gAka] 5G AKA confirmation succeeded
2024-03-31T16:28:13.932856114+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.933927330+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.937515087+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.938911156+09:00 [INFO][UDM][UEAU] Handle ConfirmAuthDataRequest
2024-03-31T16:28:13.939784535+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.941351355+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.944416470+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.945784780+09:00 [INFO][UDR][DataRepo] Handle CreateAuthenticationStatus
2024-03-31T16:28:13.946771510+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/authentication-data/authentication-status |
2024-03-31T16:28:13.947052821+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-ueau/v1/imsi-001010000000000/auth-events |
2024-03-31T16:28:13.947484070+09:00 [INFO][AUSF][GIN] | 200 |       127.0.0.1 | PUT     | /nausf-auth/v1/ue-authentications/suci-0-001-01-0000-0-0-0000000000/5g-aka-confirmation |
2024-03-31T16:28:13.947752356+09:00 [INFO][AMF][Gmm] Handle event[Authentication Success], transition from [Authentication] to [SecurityMode]
2024-03-31T16:28:13.947797105+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Security Mode Command
2024-03-31T16:28:13.948033880+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Send Downlink Nas Transport
2024-03-31T16:28:13.948471246+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3560 timer
2024-03-31T16:28:13.950158634+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport
2024-03-31T16:28:13.950359785+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:28:13.950598658+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [SecurityMode] to [SecurityMode]
2024-03-31T16:28:13.950763171+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Security Mode Complete
2024-03-31T16:28:13.950954727+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3560 timer
2024-03-31T16:28:13.951143593+09:00 [INFO][AMF][Gmm] Handle event[SecurityMode Success], transition from [SecurityMode] to [ContextSetup]
2024-03-31T16:28:13.951307207+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle InitialRegistration
2024-03-31T16:28:13.952019514+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.954389308+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.957037751+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.959512067+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:13.960796914+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T16:28:13.961478815+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.962735087+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.966210294+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.967596056+09:00 [INFO][UDM][SDM] Handle GetNssai
2024-03-31T16:28:13.968292372+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.969416068+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.972698603+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.973972635+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T16:28:13.974763507+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data |
2024-03-31T16:28:13.975123319+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/nssai?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:28:13.975614560+09:00 [INFO][AMF][Gmm] RequestedNssai: &{Iei:47 Len:5 Buffer:[4 1 1 2 3]}
2024-03-31T16:28:13.975784601+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] RequestedNssai - ServingSnssai: &{Sst:1 Sd:010203}, HomeSnssai: <nil>
2024-03-31T16:28:13.976892053+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.978183853+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.981005236+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.981997345+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:13.983194949+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=UDM |
2024-03-31T16:28:13.983778922+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.985004365+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.988389637+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.989906611+09:00 [INFO][UDM][UECM] Handle RegistrationAmf3gppAccess
2024-03-31T16:28:13.990368031+09:00 [INFO][UDM][UECM] UEID: imsi-001010000000000
2024-03-31T16:28:13.991004045+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:13.992192159+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:13.995208462+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:13.996661681+09:00 [INFO][UDR][DataRepo] Handle CreateAmfContext3gpp
2024-03-31T16:28:13.997736038+09:00 [INFO][UDR][GIN] | 204 |       127.0.0.1 | PUT     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/amf-3gpp-access |
2024-03-31T16:28:13.998038339+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | PUT     | /nudm-uecm/v1/imsi-001010000000000/registrations/amf-3gpp-access |
2024-03-31T16:28:13.999167659+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.000580395+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.005484959+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.006803577+09:00 [INFO][UDM][SDM] Handle GetAmData
2024-03-31T16:28:14.007540944+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.008640826+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.011743841+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.013244232+09:00 [INFO][UDR][DataRepo] Handle QueryAmData
2024-03-31T16:28:14.013929278+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/am-data?supported-features=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:28:14.014268354+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/am-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:28:14.015412924+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.016813250+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.020570030+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.024120698+09:00 [INFO][UDM][SDM] Handle GetSmfSelectData
2024-03-31T16:28:14.024829519+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.026018819+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.029057882+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.030306814+09:00 [INFO][UDR][DataRepo] Handle QuerySmfSelectData
2024-03-31T16:28:14.031232655+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/smf-selection-subscription-data |
2024-03-31T16:28:14.031553263+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/smf-select-data?plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:28:14.032388551+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.033641923+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.037254335+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.038512194+09:00 [INFO][UDM][SDM] Handle GetUeContextInSmfData
2024-03-31T16:28:14.039522777+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.041047536+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.044283302+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.045615561+09:00 [INFO][UDR][DataRepo] Handle QuerySmfRegList
2024-03-31T16:28:14.046304942+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/smf-registrations |
2024-03-31T16:28:14.046616664+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/ue-context-in-smf-data |
2024-03-31T16:28:14.047463362+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.048752949+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.052286369+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.053654841+09:00 [INFO][UDM][SDM] Handle Subscribe
2024-03-31T16:28:14.054618474+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.055705045+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.058860579+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.060249418+09:00 [INFO][UDR][DataRepo] Handle CreateSdmSubscriptions
2024-03-31T16:28:14.060438849+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/subscription-data/imsi-001010000000000/context-data/sdm-subscriptions |
2024-03-31T16:28:14.060772130+09:00 [INFO][UDM][GIN] | 201 |       127.0.0.1 | POST    | /nudm-sdm/v1/imsi-001010000000000/sdm-subscriptions |
2024-03-31T16:28:14.061671689+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.063266548+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.065846907+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.066912155+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.068241502+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=AMF&supi=imsi-001010000000000&target-nf-type=PCF |
2024-03-31T16:28:14.068780088+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.070031821+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.073506026+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.075725558+09:00 [INFO][PCF][AmPol] Handle AM Policy Create Request
2024-03-31T16:28:14.076413326+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.077834941+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.080413387+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.081650477+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.082460414+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=UDR |
2024-03-31T16:28:14.083051008+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.085208387+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.089396586+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.090994189+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdAmDataGet
2024-03-31T16:28:14.091866985+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/am-data |
2024-03-31T16:28:14.093062516+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.094490366+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.097012113+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.098049280+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.099821803+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?guami=%7B%22plmnId%22%3A%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D%2C%22amfId%22%3A%22cafe00%22%7D&requester-nf-type=PCF&target-nf-type=AMF |
2024-03-31T16:28:14.100527116+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.101719243+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.105452832+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.106938567+09:00 [INFO][AMF][Comm] Handle AMF Status Change Subscribe Request
2024-03-31T16:28:14.107158072+09:00 [INFO][AMF][Comm] new AMF Status Subscription[2]
2024-03-31T16:28:14.107363694+09:00 [INFO][AMF][GIN] | 201 |       127.0.0.1 | POST    | /namf-comm/v1/subscriptions |
2024-03-31T16:28:14.107878486+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-am-policy-control/v1/policies |
2024-03-31T16:28:14.108390253+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Registration Accept
2024-03-31T16:28:14.108615148+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Send Initial Context Setup Request
2024-03-31T16:28:14.109825049+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Start T3550 timer
2024-03-31T16:28:14.110791906+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle InitialContextSetupResponse
2024-03-31T16:28:14.110974762+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle InitialContextSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T16:28:14.318150946+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport
2024-03-31T16:28:14.319022248+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:28:14.319842590+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [ContextSetup] to [ContextSetup]
2024-03-31T16:28:14.320430600+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle Registration Complete
2024-03-31T16:28:14.321066576+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Stop T3550 timer
2024-03-31T16:28:14.321771103+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Send Configuration Update Command
2024-03-31T16:28:14.322350686+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Send Downlink Nas Transport
2024-03-31T16:28:14.325183707+09:00 [INFO][AMF][Gmm] Handle event[ContextSetup Success], transition from [ContextSetup] to [Registered]
2024-03-31T16:28:14.328287893+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport
2024-03-31T16:28:14.329057285+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle UplinkNASTransport (RAN UE NGAP ID: 1)
2024-03-31T16:28:14.329866300+09:00 [INFO][AMF][Gmm] Handle event[Gmm Message], transition from [Registered] to [Registered]
2024-03-31T16:28:14.330544092+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Handle UL NAS Transport
2024-03-31T16:28:14.331066112+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Transport 5GSM Message to SMF
2024-03-31T16:28:14.331846716+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] Select SMF [snssai: {Sst:1 Sd:010203}, dnn: internet]
2024-03-31T16:28:14.335134900+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.341765883+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.354375154+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.357864140+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.361118846+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=AMF&target-nf-type=NSSF |
2024-03-31T16:28:14.362849305+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.366573954+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.375493298+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.379040231+09:00 [INFO][NSSF][NsSel] Handle NSSelectionGet
2024-03-31T16:28:14.379783917+09:00 [INFO][NSSF][GIN] | 200 |       127.0.0.1 | GET     | /nnssf-nsselection/v1/network-slice-information?nf-id=27a0c773-5e25-4f7d-8179-79c4850af923&nf-type=AMF&slice-info-request-for-pdu-session=%7B%22sNssai%22%3A%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D%2C%22roamingIndication%22%3A%22NON_ROAMING%22%7D |
2024-03-31T16:28:14.383008303+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.385723938+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.390631063+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.392803879+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.394840601+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?dnn=internet&preferred-locality=loc2&requester-nf-type=AMF&service-names=nsmf-pdusession&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&target-nf-type=SMF&target-plmn-list=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D |
2024-03-31T16:28:14.395900515+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.398078893+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.403568783+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.406131264+09:00 [INFO][SMF][PduSess] Receive Create SM Context Request
2024-03-31T16:28:14.407264518+09:00 [INFO][SMF][PduSess] In HandlePDUSessionSMContextCreate
2024-03-31T16:28:14.407569634+09:00 [INFO][SMF][CTX] UrrPeriod: 10s
2024-03-31T16:28:14.407782585+09:00 [INFO][SMF][CTX] UrrThreshold: 1000
2024-03-31T16:28:14.408699268+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.410529188+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.414034934+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.415399614+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.416800836+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=UDM |
2024-03-31T16:28:14.417293208+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Send NF Discovery Serving UDM Successfully
2024-03-31T16:28:14.417740530+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.419291666+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.423213218+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.424854035+09:00 [INFO][UDM][SDM] Handle GetSmData
2024-03-31T16:28:14.425763944+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.427131881+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.430989459+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.431360414+09:00 [INFO][UDM][SDM] getSmDataProcedure: SUPI[imsi-001010000000000] PLMNID[00101] DNN[internet] SNssai[{"sst":1,"sd":"010203"}]
2024-03-31T16:28:14.432763153+09:00 [INFO][UDR][DataRepo] Handle QuerySmData
2024-03-31T16:28:14.433714013+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/subscription-data/imsi-001010000000000/00101/provisioned-data/sm-data?single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:28:14.434103142+09:00 [INFO][UDM][GIN] | 200 |       127.0.0.1 | GET     | /nudm-sdm/v1/imsi-001010000000000/sm-data?dnn=internet&plmn-id=%7B%22mcc%22%3A%22001%22%2C%22mnc%22%3A%2201%22%7D&single-nssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:28:14.434694320+09:00 [INFO][SMF][GSM] In HandlePDUSessionEstablishmentRequest
2024-03-31T16:28:14+09:00 [INFO][NAS][Convert] ProtocolOrContainerList:  [0xc0001d87a0 0xc0001d87c0]
2024-03-31T16:28:14.435125124+09:00 [INFO][SMF][GSM] Protocol Configuration Options
2024-03-31T16:28:14.435147376+09:00 [INFO][SMF][GSM] &{[0xc0001d87a0 0xc0001d87c0]}
2024-03-31T16:28:14.435158602+09:00 [INFO][SMF][GSM] Didn't Implement container type IPAddressAllocationViaNASSignallingUL
2024-03-31T16:28:14.436283912+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.437567775+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.440338445+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.441407608+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.444042122+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-instance-id=27a0c773-5e25-4f7d-8179-79c4850af923&target-nf-type=AMF |
2024-03-31T16:28:14.445453683+09:00 [INFO][SMF][Consumer] SendNFDiscoveryServingAMF ok
2024-03-31T16:28:14.446027847+09:00 [INFO][SMF][CTX] Allocated UE IP address: 10.61.0.1
2024-03-31T16:28:14.446221652+09:00 [INFO][SMF][CTX] Selected UPF: UPF
2024-03-31T16:28:14.446421894+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Allocated PDUAdress[10.61.0.1]
2024-03-31T16:28:14.447269493+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.448296288+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.451176544+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.452365071+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.453601256+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?preferred-locality=loc2&requester-nf-type=SMF&target-nf-type=PCF |
2024-03-31T16:28:14.454109612+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.455122844+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.458534199+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.459897947+09:00 [INFO][PCF][SMpolicy] Handle CreateSmPolicy
2024-03-31T16:28:14.460512765+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.461783094+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.464956972+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.466391289+09:00 [INFO][UDR][DataRepo] Handle PolicyDataUesUeIdSmDataGet
2024-03-31T16:28:14.467360524+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/policy-data/ues/imsi-001010000000000/sm-data?dnn=internet&snssai=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D |
2024-03-31T16:28:14.472356515+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.473598928+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.476667067+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.477902620+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataGet
2024-03-31T16:28:14.478582819+09:00 [INFO][UDR][GIN] | 200 |       127.0.0.1 | GET     | /nudr-dr/v1/application-data/influenceData?dnns=internet&internal-Group-Ids=&snssais=%7B%22sst%22%3A1%2C%22sd%22%3A%22010203%22%7D&supis=imsi-001010000000000 |
2024-03-31T16:28:14.478875792+09:00 [INFO][PCF][SMpolicy] Matched [0] trafficInfluDatas from UDR
2024-03-31T16:28:14.479718711+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.481197406+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.484531106+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.485903410+09:00 [INFO][UDR][DataRepo] Handle ApplicationDataInfluenceDataSubsToNotifyPost
2024-03-31T16:28:14.486131786+09:00 [INFO][UDR][GIN] | 201 |       127.0.0.1 | POST    | /nudr-dr/v1/application-data/influenceData/subs-to-notify |
2024-03-31T16:28:14.486582000+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.487792859+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.490511083+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.491886072+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.492384560+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=PCF&target-nf-type=BSF |
2024-03-31T16:28:14.493194891+09:00 [INFO][PCF][GIN] | 201 |       127.0.0.1 | POST    | /npcf-smpolicycontrol/v1/sm-policies |
2024-03-31T16:28:14.494139824+09:00 [INFO][SMF][PduSess] CHF Selection for SMContext SUPI[imsi-001010000000000] PDUSessionID[1]
2024-03-31T16:28:14.494885803+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.495918321+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.498573403+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.499590309+09:00 [INFO][NRF][DISC] Handle NFDiscoveryRequest
2024-03-31T16:28:14.500424483+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | GET     | /nnrf-disc/v1/nf-instances?requester-nf-type=SMF&target-nf-type=CHF |
2024-03-31T16:28:14.500751594+09:00 [INFO][SMF][Charging] Handle SendConvergedChargingRequest
2024-03-31T16:28:14.501057451+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.502077531+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.505196797+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.507505357+09:00 [INFO][CHF][ChargingPost] HandleChargingdataInitial
2024-03-31T16:28:14.507780518+09:00 [INFO][CHF][ChargingPost] SMF charging event
2024-03-31T16:28:14.508026992+09:00 [ERRO][CHF][ChargingPost] Charging gateway fail to send CDR to billing domain dial tcp 127.0.0.1:2122: connect: connection refused
2024-03-31T16:28:14.508045693+09:00 [INFO][CHF][ChargingPost] Open CDR for UE imsi-001010000000000
2024-03-31T16:28:14.508077229+09:00 [INFO][CHF][ChargingPost] NewChfUe imsi-001010000000000
2024-03-31T16:28:14.508209789+09:00 [INFO][CHF][GIN] | 201 |       127.0.0.1 | POST    | /nchf-convergedcharging/v3/chargingdata |
2024-03-31T16:28:14.508933238+09:00 [INFO][SMF][Charging] Send Charging Data Request[Init] successfully
2024-03-31T16:28:14.509167690+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-1]
2024-03-31T16:28:14.509320368+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T16:28:14.509572523+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Install PCCRule[PccRuleId-2]
2024-03-31T16:28:14.509726184+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] No srcTcData and tgtTcData. Nothing to do
2024-03-31T16:28:14.510011553+09:00 [INFO][SMF][PduSess][pdu_session_id:1][supi:imsi-001010000000000] Has no pre-config route. Has default path
2024-03-31T16:28:14.510331132+09:00 [INFO][SMF][GIN] | 201 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts |
2024-03-31T16:28:14.510672496+09:00 [INFO][SMF][PduSess] Sending PFCP Session Establishment Request
2024-03-31T16:28:14.511842720+09:00 [INFO][AMF][Gmm][amf_ue_ngap_id:RU:1,AU:1(3GPP)][supi:SUPI:imsi-001010000000000] create smContext[pduSessionID: 1] Success
2024-03-31T16:28:14.517582094+09:00 [INFO][SMF][PduSess] Received PFCP Session Establishment Accepted Response
2024-03-31T16:28:14.519462799+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.520785608+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.524400322+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.526147524+09:00 [INFO][AMF][Producer] Handle N1N2 Message Transfer Request
2024-03-31T16:28:14.526473514+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Send PDU Session Resource Setup Request
2024-03-31T16:28:14.527365398+09:00 [INFO][AMF][GIN] | 200 |       127.0.0.1 | POST    | /namf-comm/v1/ue-contexts/imsi-001010000000000/n1-n2-messages |
2024-03-31T16:28:14.529763818+09:00 [INFO][AMF][Ngap][ran_addr:192.168.0.132:43308] Handle PDUSessionResourceSetupResponse
2024-03-31T16:28:14.530439237+09:00 [INFO][AMF][Ngap][amf_ue_ngap_id:RU:1,AU:1(3GPP)][ran_addr:192.168.0.132:43308] Handle PDUSessionResourceSetupResponse (RAN UE NGAP ID: 1)
2024-03-31T16:28:14.531511550+09:00 [INFO][NRF][Token] In HTTPAccessTokenRequest
2024-03-31T16:28:14.533776473+09:00 [WARN][NRF][Token] Certificate verify: x509: certificate signed by unknown authority (possibly because of "x509: invalid signature: parent certificate cannot sign this kind of certificate" while trying to verify candidate authority certificate "free5gc")
2024-03-31T16:28:14.539006914+09:00 [INFO][NRF][GIN] | 200 |       127.0.0.1 | POST    | /oauth2/token |
2024-03-31T16:28:14.541475488+09:00 [INFO][SMF][PduSess] Receive Update SM Context Request
2024-03-31T16:28:14.548516767+09:00 [INFO][SMF][PduSess] Received PFCP Session Modification Accepted Response from AN UPF
2024-03-31T16:28:14.548939839+09:00 [INFO][SMF][GIN] | 200 |       127.0.0.1 | POST    | /nsmf-pdusession/v1/sm-contexts/urn:uuid:c9baec74-249a-44ce-b98c-aa72bb5e0d02/modify |
```
The free5GC U-Plane2 log when executed is as follows.
```
2024-03-31T16:28:14.432187741+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionEstablishmentRequest
2024-03-31T16:28:14.432244144+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805][CPNodeID:192.168.0.143][CPSEID:0x1][UPSEID:0x1] New session
2024-03-31T16:28:14.434680597+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:1 period:10000000000}]
2024-03-31T16:28:14.435491378+09:00 [INFO][UPF][Perio] recv event[TYPE_PERIO_ADD][{eType:1 lSeid:1 urrid:2 period:10000000000}]
2024-03-31T16:28:14.464035237+09:00 [INFO][UPF][PFCP][LAddr:192.168.0.145:8805] handleSessionModificationRequest
```
The TUNnel interface `uesimtun0` is created as follows.
```
# ip addr show
...
9: uesimtun0: <POINTOPOINT,PROMISC,NOTRAILERS,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.61.0.1/32 scope global uesimtun0
       valid_lft forever preferred_lft forever
    inet6 fe80::f13a:ef19:4bea:f7b4/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
...
```

<a id="ping_ue2"></a>

#### Ping google.com going through DN=10.61.0.0/16 on Loc2

Confirm by using `tcpdump` that the packet goes through `if=upfgtp` on U-Plane2.
```
# ping google.com -I uesimtun0 -n
PING google.com (142.251.222.14) from 10.61.0.1 uesimtun0: 56(84) bytes of data.
64 bytes from 142.251.222.14: icmp_seq=1 ttl=61 time=20.1 ms
64 bytes from 142.251.222.14: icmp_seq=2 ttl=61 time=60.2 ms
64 bytes from 142.251.222.14: icmp_seq=3 ttl=61 time=21.2 ms
```
The `tcpdump` log on U-Plane2 is as follows.
```
16:33:35.331554 IP 10.61.0.1 > 142.251.222.14: ICMP echo request, id 7, seq 1, length 64
16:33:35.349152 IP 142.251.222.14 > 10.61.0.1: ICMP echo reply, id 7, seq 1, length 64
16:33:36.332757 IP 10.61.0.1 > 142.251.222.14: ICMP echo request, id 7, seq 2, length 64
16:33:36.390582 IP 142.251.222.14 > 10.61.0.1: ICMP echo reply, id 7, seq 2, length 64
16:33:37.334804 IP 10.61.0.1 > 142.251.222.14: ICMP echo request, id 7, seq 3, length 64
16:33:37.352633 IP 142.251.222.14 > 10.61.0.1: ICMP echo reply, id 7, seq 3, length 64
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
