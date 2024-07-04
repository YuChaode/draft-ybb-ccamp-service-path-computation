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
    name: "Chaode Yu"
    org: Huawei Technologies
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
For the Top-down approach, the path computation request should be based on client signal service. It is needed to specify the source and destination access port of in the request. In addition, some common parameters, such as number of path to be calculated, protection and restoration capabities, path computation policy and path constraint (see {{path-constraint}}).etc. And then the domain controller will trigger the computation on all the layers. The path computation result should contain all the layers' path information.
For the OTN tunnel and WDM tunnel in the service path computation result, they can be non-existing before service path compuation and can be totally designed by the domain controller or control plane. The tunnels can also be existing before the service computation request. The domain controller can design to reuse some existing tunnels based on the consideration of maximum resource utilization.
Similar to TE tunnel path computation, service path computation should not create any tunnels on the network during the whole computation process, and will not introduce any other changes on the network.

~~~~ ascii-art
module: ietf-trans-client-service-path-computation
rpcs:
   +---x client-service-precompute
      +--ro input
      |  +--ro request* [request-id]
      |     +--ro request-id                        string
      |     +--ro path-count?                       uint8
      |     +--ro te-topology-identifier
      |     +--ro src-access-ports
      |     +--ro dst-access-ports
      |     +--ro tunnel-policy
      |     |  +--ro protection
      |     |  +--ro restoration
      |     |  +--ro optimizations
      |     +--ro explicit-route-exclude-objects
      |     |  +--ro route-object-exclude-object* [index]
      |     +--ro explicit-route-include-objects
      |        +--ro route-object-include-object* [index]
      +--ro output    
         +--ro result* [request-id]
            +--ro request-id                string
            +--ro result-code?              enumeration
            +--ro (result-detail)?
               +--:(failure)
               |  +--ro failure-reason?           uint32
               |  +--ro error-message?            string
               +--:(success)
                  +--ro computed-paths* [path-id]
                  |  +--ro path-id        yang:uuid
                  |  +--ro path-number?   uint8
                  +--ro te-topology-identifier
                  +--ro src-access-ports
                  +--ro dst-access-ports
                  +--ro underlay-tunnels
                     +--ro underlay-tunnel* [index]
                        +--ro index                     uint8
                        +--ro tunnel-name?              leafref
                        +--ro te-topology-identifier
                        +--ro computed-lsp* [lsp-id]
                           +--ro lsp-id               uint8
                           +--ro lsp-type?            enumeration
                           +--ro lsp-metrics
                           |  +--ro lsp-metric* [metric-type]
                           +--ro lsp-route-objects* [index]
~~~~

## Multi-layer Path Display
For the MDSC, if it wants to show the whole information of computed path, the path computation result provided by the domain controller must contain tunnels at all the layers. The number of these tunnels is not fixed. For example, for OTN tunnel, the client signal can be encapsulated into a low order OTN tunnel at first, and then be multiplexed into a higher OTN tunnel. The client signal can be encapsulated into a high order OTN tunnel directly without the multiplexing scenario. There is another common scenario that an OTN tunnel is supported by multiple WDM tunnels.
This document provides a generic structure in a list to present the actual path information regardless which layer it is. The tunnels will be ordered by index from 0 for the topmost tunnel. The correlation with server tunnel will be provided by the "server-tunnel" attribute in the LSP hop.

~~~~ ascii-art
............................
+--ro underlay-tunnels
   +--ro underlay-tunnel* [index]
      +--ro index                     uint8
      +--ro tunnel-name?              leafref
      +--ro te-topology-identifier
      |  +--ro provider-id?   te-types:te-global-id
      |  +--ro client-id?     te-types:te-global-id
      |  +--ro topology-id?   te-types:te-topology-id
      +--ro computed-lsp* [lsp-id]
         +--ro lsp-id               uint8
         +--ro lsp-type?            enumeration
         +--ro lsp-metrics
         |  +--ro lsp-metric* [metric-type]
         |     +--ro metric-type     identityref
         |     +--ro metric-value?   uint32
         |     +--ro metric-unit?    string
         +--ro lsp-route-objects* [index]
            +--ro index            uint8
            +--ro node-id?         te-types:te-node-id
            +--ro node-uri-id?     yang:uuid
            +--ro link-tp-id?      te-types:te-tp-id
            +--ro ltp-uri-id?      yang:uuid
            +--ro te-label
            |  +--ro (technology)?
            |  |  +--:(wson)
            |  |  |  +--ro (grid-type)?
            |  |  |     +--:(dwdm)
            |  |  |     |  +--ro (single-or-super-channel)?
            |  |  |     |     +--:(single)
            |  |  |     |     |  +--ro dwdm-n?
            l0-types:dwdm-n
            |  |  |     |     +--:(super)
            |  |  |     |        +--ro subcarrier-dwdm-n*
            l0-types:dwdm-n
            |  |  |     +--:(cwdm)
            |  |  |        +--ro cwdm-n?              l0-types:cwdm-n
            |  |  +--:(otn)
            |  |     +--ro otn
            |  |        +--ro tpn?       otn-tpn
            |  |        +--ro tsg?       identityref
            |  |        +--ro ts-list?   string
            |  +--ro direction?           te-types:te-label-direction
            +--ro server-tunnel?   leafref

~~~~

{{path-constraint}}
## Path Constraint
It is common for service path computation request to specify path constrain like node/link-tp inclusion/exclusion like TE tunnel path computation. And service path computation needs to support some more kind of path constraint, such as to specify service/tunnel/path included/excluded. There are also scenarios to specify path constrain across layers. For example, some people would like to specify a WDM node included/excluded or wavelength in the service path computation.

~~~~ ascii-art
................................
+--ro route-object-include-object* [index]
   +--ro index                 uint8
   +--ro node-id?              te-types:te-node-id
   +--ro node-uri-id?          yang:uuid
   +--ro link-tp-id?           te-types:te-tp-id
   +--ro ltp-uri-id?           yang:uuid
   +--ro te-label
   |  +--ro (technology)?
   |  |  +--:(wson)
   |  |  |  +--ro (grid-type)?
   |  |  |     +--:(dwdm)
   |  |  |     |  +--ro (single-or-super-channel)?
   |  |  |     |     +--:(single)
   |  |  |     |     |  +--ro dwdm-n?              l0-types:dwdm-n
   |  |  |     |     +--:(super)
   |  |  |     |        +--ro subcarrier-dwdm-n*   l0-types:dwdm-n
   |  |  |     +--:(cwdm)
   |  |  |        +--ro cwdm-n?              l0-types:cwdm-n
   |  |  +--:(otn)
   |  |     +--ro otn
   |  |        +--ro tpn?       otn-tpn
   |  |        +--ro tsg?       identityref
   |  |        +--ro ts-list?   string
   |  +--ro direction?           te-types:te-label-direction
   +--ro server-tunnel-name?   leafref
   +--ro lsp-type?             enumeration
~~~~

## Path Management
### Storage of Path Computation Result
It is useful to save the path computation results after they are return. Sometimes, people from operators will make the decision on the results in a short time. If the path is not saved in the domain controller, the MDSC needs to send the full path in the service creation request which is complex. So in this document, we recommend that the domain controller should be capable of saving path computation results to make it possible that the MDSC can reference the path computation result in service creation request.
The path information in the path management structure should be same with the output of service path computation RPC. Once there is a RPC succeed in operating, the path management structure will add one more item.
It is noted that the path computation result is not required to be saved forever in the domain controller. How long could it be saved is implementation-specific. But if the path has been adopted by a service creation request, including path inclusion/exclusion, the path can not be deleted from data store.

Note: the service path computation request is defined as an RPC, which is stateless. According to the requirement of RESTCONF, RPC should not generate any impact on the data model. So it is recommended to discuss with RESTCONF protocol expert to find a workaround solution.

~~~~ ascii-art
module: ietf-trans-client-service-path-computation
   +--rw path-management
      +--rw path* [path-id]
         +--rw path-id             yang:uuid
         +--rw creation-time?      yang:date-and-time
         +--rw validity-period?    uint8
         +--rw underlay-tunnels
            +--rw underlay-tunnel* [index]
               +--rw index                     uint8
               +--rw tunnel-name?              leafref
               +--rw te-topology-identifier
               |  +--rw provider-id?   te-types:te-global-id
               |  +--rw client-id?     te-types:te-global-id
               |  +--rw topology-id?   te-types:te-topology-id
               +--rw computed-lsp* [lsp-id]
                  +--rw lsp-id               uint8
                  +--rw lsp-type?            enumeration
                  +--rw lsp-metrics
                  |  +--rw lsp-metric* [metric-type]
                  |     +--rw metric-type     identityref
                  |     +--rw metric-value?   uint32
                  |     +--rw metric-unit?    string
                  +--rw lsp-route-objects* [index]
                     +--rw index            uint8
                     +--rw node-id?         te-types:te-node-id
                     +--rw node-uri-id?     yang:uuid
                     +--rw link-tp-id?      te-types:te-tp-id
                     +--rw ltp-uri-id?      yang:uuid
                     +--rw te-label
                     |  +--rw (technology)?
                     |  |  +--:(wson)
                     |  |  |  +--rw (grid-type)?
                     |  |  |     +--:(dwdm)
                     |  |  |     |  +--rw (single-or-super-channel)?
                     |  |  |     |     +--:(single)
                     |  |  |     |     |  +--rw dwdm-n?
                     l0-types:dwdm-n
                     |  |  |     |     +--:(super)
                     |  |  |     |        +--rw subcarrier-dwdm-n*
                     l0-types:dwdm-n
                     |  |  |     +--:(cwdm)
                     |  |  |        +--rw cwdm-n?
                     l0-types:cwdm-n
                     |  |  +--:(otn)
                     |  |     +--rw otn
                     |  |        +--rw tpn?       otn-tpn
                     |  |        +--rw tsg?       identityref
                     |  |        +--rw ts-list?   string
                     |  +--rw direction?
                     te-types:te-label-direction
                     +--rw server-tunnel?   leafref
~~~~

### Path Reference in Service Provisioning
In the current service provisioning approach, the MDSC needs to specify the correlation of tunnel underlay. If the path computation result is saved in the domain controller. It is much easier to reference the path computation result instead of specifying tunnel underlay to do the provisioning.

~~~~ ascii-art

~~~~

# Tree Diagram for Service Path Computation
~~~~ ascii-art
{::include ./ietf-trans-client-service-path-computation.tree}
~~~~
{: #fig-spc-tree title="Service path computation tree diagram"
artwork-name="etf-trans-client-service-path-computation.tree"}

# YANG Data Model for Service Path Computation
~~~~ yang
{::include ./ietf-trans-client-service-path-computation.yang}
~~~~
{: #fig-rpm-yang title="Service Path Computation YANG module"
sourcecode-markers="true" sourcecode-name="ietf-trans-client-service-path-computation@2024-06-26.yang"}

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

# Acknowledgments
{:numbered="false"}

This document was prepared using kramdown.
