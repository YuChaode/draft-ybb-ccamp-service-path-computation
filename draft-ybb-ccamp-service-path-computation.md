---
title: "A YANG Data Model for Service Path Computation"
abbrev: "Service Path Computation"
category: std

docname: draft-ybb-ccamp-service-path-computation-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Common Control and Measurement Plane"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Common Control and Measurement Plane"
  type: "Working Group"
  mail: "ccamp@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ccamp/"
  github: "YuChaode/draft-ybb-ccamp-service-path-computation"
  latest: "https://YuChaode.github.io/draft-ybb-ccamp-service-path-computation/draft-ybb-ccamp-service-path-computation.html"

author:
 -
    fullname: "Chaode Yu"
    organization: Huawei Technologies
    email: "yuchaode@huawei.com"
 -
    name: Sergio Belotti
    org: Nokia
    email: sergio.belotti@nokia.com
 -
    name: Italo Busi
    org: Huawei Technologies
    email: italo.busi@huawei.com
 -
    name: Aihua Guo
    org: Futurewei Technologies
    email: aihuaguo.ietf@gmail.com
 -
    name: Dieter Beller
    org: Nokia
    email: dieter.beller@nokia.com

normative:

informative:


--- abstract

This document defines a YANG data model for client signal service's path computation and path management.

--- middle

# Introduction

For transport technology, the service's path is hierarchical, including OTN-layer trails and WDM-layer trails. According to the funcational structure defined by ITU-T, the OTN-layer trails include lower-order ODUk, high-order ODUk and OTUk trails. The WDM-lay trails include OTS, OMS and OCh trail. These trails and the configuration of client-side port configuration form an end-to-end client signal service.

Path computation is a common network management funcation. For traditional transport services, OTS and OMS trails are automatically generated once the fibers are connected and they don't require any path computation. The path computation is performed on the OCh layer trails and above. Traditionally, the path computation is performed from OCh to low-order ODUk trail layer by layer. This is called bottom-up approach.

One disavantage of bottom-up approach is that the client and server need to interact with each other multiple times. And sometimes, the client layer trail cannot be computed until the server layer trail is setup. This doea not comply with the intention of path computation which will not introduce actual configuration on the network.

In addition, the client prefers to transfer intent-based configuration instead of complex network configuration to achieve path computation and obtain all the layers' path computation result at one time. The server, usually it is played by the domain controller, can help to deal with the complex network configuration computation. This is called top-down approach.

This document focuses on the top-down path computation for transparent client signal service and non-transparent client signal service (Ethernet service).

We also focus on addressing some undiscussed requirements, such as how to ultize a generic structure to display path information for all the layers' path at once, and how to specify some parameters on the server layer's trail in the service path computation request, and how to support more complex path constraints, and how to manage the path computation results and how to reference them in service provisioning request.

## Terminology and Notations

  The following terms are defined in {{!RFC7950}} and are not
  redefined here:

  *  client

  *  server

  *  augment

  *  data model

  *  data node

  The terminology for describing YANG data models is found in
  {{!RFC7950}}.

## Tree Diagram

A simplified graphical representation of the data model is used in {{ spp-tree}} of this document.
The meaning of the symbols in this diagram is defined in {{!RFC8340}}.

## Prefix in Data Node Names

  In this document, names of data nodes and other data model objects
  are prefixed using the standard prefix associated with the
  corresponding YANG imported modules, as shown in the following table.

| Prefix    | Yang Module                     | Reference     |
| --------- | ------------------------------- | ------------- |
| clntsvc   | ietf-trans-client-service       | RFC XXXX      |
| clnsvcpc  | ietf-clnt-svc-path-computation     | RFC YYYY  |

{: #tab-prefixes title="Prefixes and corresponding YANG modules"}

RFC Editor Note:
Please replace XXXX and YYYY with the RFC number assigned to this document.
Please remove this note.

# End to End Management of Transport Network Client Signal Service

The hierarchical relationship of transport client signal service can reference to {{fig-rel-service-tunnel}}:
(Please note that, the OTN tunnels in this figure and subsequent figures include lower order OTN tunnels, higher order OTN tunnels, and OTU tunnels. The supporting relationship should comply with ITU-T G.XXXX definition. WDM tunnel also include the WDM, flexi-grid scenario and Media channel scenario. The hierarchical relationship should comply with ITU-T G.XXXX definition.)

~~~~ ascii-art
|<----------Transparent client signal---------->|
     |<--------------OTN Tunnel---------->|
       |<------------WDM Tunnel-------->|
~~~~
{: #fig-rel-service-tunnel title="Transparent Client Signal Service and its Server tunnel"}

For Ethernet client signal service, the client signal can be encapsulated into multiple approach. For example, it can be encapsulated into OTN tunnel directly, or it can be encapsulated into MPLS-TP/SDH tunnels and then into OTN tunnels. Note that the SDH tunnel is considered as an outdated technology and lacks standardization. Therefore, the scenario where Ethernet client signals are encapsulated into the SDH tunnel is not described in this document.
The {{fig-rel-eth-service-tunnel}} shows the hierarchical relationship of Ethernet service:

~~~~ ascii-art
|<----------Ethernet client signal---------->|
   |<----MPLS-TP Tunnel (Optional) ----->|
    |<-----------OTN Tunnel----------->|
      |<---------WDM Tunnel--------->|
~~~~
{: #fig-rel-eth-service-tunnel title="Ethernet Client Signal Service and its Server tunnel"}

The reference method is defined in the {{!draft-ietf-ccamp-client-signal-yang}}. The supporting relationship between the tunnels can be also found by the dependency-tunnel structure define by the {{!draft-ietf-teas-yang-te}}.

# Requirements for Service Path Computation

## Top-down Approach
For the Top-down approach, the path computation request should be based on client signal service. It is needed to specify the source and destination access port of in the request. In addition, some common parameters, such as number of path to be calculated, protection and restoration capabities, path computation policy and path constraint (see {{path-constraint}}).etc.
The path computation result should contain all the layers' path information. For the OTN tunnel and WDM tunnel, it can be existing 



## Multi-layer Path Display

{{path-constraint}}
## Path Constraint

## Path Management
### Saving of Path Computation Result

### Path Reference in Service Provisioning

# Tree Diagram for Service Path Computation


# YANG Data Model for Service Path Computation



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This document was prepared using kramdown.
