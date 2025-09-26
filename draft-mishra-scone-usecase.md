---
title: "Applicability & Manageability consideration for SCONE"
abbrev: "SCONE Applicability & Manageability"
docname: draft-mishra-scone-applicability-manageablity-02
category: info

ipr: trust200902
area: Web and Internet Transport
workgroup: SCONE
keyword: Throttling
keyword: Adaptive Bit-Rate Video, scone

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: S. Mishra
    name: Sanjay Mishra
    organization: Verizon
    email: sanjay.mishra@verizon.com
  -
    ins: Z. Sarker
    name: Zaheduzzaman Sarker
    organization: Nokia
    email: zaheduzzaman.sarker@nokia.com 
  -
    ins: A. Tomar
    name: Anoop Tomar
    organization: Meta
    email: anooptomar@meta.com
  -
    ins: K. Abbas
    name: Khurram Abbas
    organization: Verizon
    email: khurram.abbas@verizonwireless.com

normative:

informative:
  I-D.joras-scone-video-optimization-requirements:

  SCONE-Charter:
    target: https://datatracker.ietf.org/wg/scone/about/
    title: SCONE Working Group Charter
    author:
    -
      name: IETF
    date: 2024-10-31

  5G-Arch:
    target: https://portal.3gpp.org/desktopmodules/Specifications/SpecificationDetails.aspx?specificationId=3144
    title: System architecture for the 5G System (5GS)
    author:
    -
      name: 3GPP
    date: 2025-01-07

 
--- abstract
This document addresses the applicability, manageability, and operational considerations involved in providing throughput advice to application endpoints in telecommunications service provider networks supporting the Standard Communication with Network Elements (SCONE) protocol.

--- middle

# Introduction

The SCONE protocol is a signaling mechanism that enables access network providers to communicate a maximum allowable bit-rate to application endpoints, specifically targeting adaptive bit-rate applications. This document describes on the applicability, manageability, and operational considerations of deploying the SCONE protocol within telecommunications provider networks and at application endpoints. 

The SCONE protocol operates on the UDP 4-tuple, where network elements capable of rate limiting on a UDP 4-tuple. A network element can provide send notificatio about rate limiting for both upstream and downstream traffic that it observes. It is capable of dropping or delaying packets on the path of the respective UDP 4-tuple flows. This means the scone protocol has some assumption on the charateristic of a network element. A network element, sitting in the access networkis, is capable of detecting and maintaining a UDP 4-tuple flow, have rate limiting policies, and can detect flows that include SCONE packets, then put a rate limiting advice in the those SCONE packets.   

Current Intenet has diverse access networks, however, not all the access network operate the same way. The mobile network among all the access networks has more fine grain views on the traffic flows that passes through the network and can operate on individual flow level. In Mobile networks, a User Plane Function (UPF) in 5G and the Packet Data Network Gateway (P-GW) in 4G generate can generate throughput advice to guide adaptive applications as per UDP 4-tuple. A wifi access network can have policies per users or Service Set Indentifiers (SSID)s in terms of Quality or Services (QoS) and speed limit. However, may not have UDP 4-tuple level flow visibilty. In wired network access, the limit is usually done in a centralized Broder NEtwork Gateway or at some aggregation points where number of the Customer premises equiptment (CPE)s are connected.  

Hence, the applicability and manageability consederations need to cover wide range of access network cases where rate limiting per UDP 4-tuple would be differently done. This document describes the generic consideation for SCONE protocol and then provides details on network specific considerations where throughput advisory signaling can enhance network resource utilization and user experience.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

- 5G - Fifth Generation Mobile Networks
The fifth generation of cellular mobile network technology defined by 3GPP.

- Adaptive Bit-Rate (ABR) Video
Video streaming technology that adjusts video quality dynamically based on network conditions.

- BNG (Broadband Network Gateway)
A network element that serves as the access point for subscribers in wireline broadband networks. It establishes and manages subscriber 
sessions, aggregates traffic from multiple subscriber access nodes, and routes this traffic to the service provider's core network. 
BNG functions include subscriber authentication, IP address assignment, policy enforcement, and quality of service management. It 
typically supports subscriber session protocols such as DHCP, PPPoE, or IPoE, and interacts with AAA and DHCP servers to enable secure 
and managed access to broadband services.

- Client App
The user-facing application running on an operating system, which receives network throughput advice.

- Content Provider
Entity or service that delivers media and data content accessed by end-users.

- DHCP - Dynamic Host Configuration Protocol
A network management protocol used to dynamically assign IP addresses and other configuration parameters to devices on a network, 
enabling automatic and centralized network configuration.

- EPS Bearer - Evolved Packet System Bearer
In 4G LTE networks, an EPS bearer is a virtual transmission path with specific Quality of Service (QoS) parameters that carries user 
data between the User Equipment (UE) and the Packet Data Network Gateway (P-GW). The EPS bearer ensures end-to-end delivery of IP packets 
with particular handling characteristics, such as priority, latency, and guaranteed bit rate. There are two main types: the Default EPS 
Bearer which provides always-on best-effort connectivity, and Dedicated EPS Bearers configured for services with specialized QoS requirements, 
such as voice or video.

- EPS Gateway
In 4G LTE networks, the EPS Gateway primarily refers to the combination of the Serving Gateway (S-GW) and the Packet Data Network Gateway 
(P-GW). The Serving Gateway routes and forwards user data packets between the E-UTRAN access network and the Packet Data Network, acting 
as a mobility anchor during handovers. The Packet Data Network Gateway provides connectivity from the user equipment (UE) to external packet 
data networks, performing functions such as policy enforcement, charging, and lawful interception. Together, these gateways form the core 
user-plane interface of the Evolved Packet System (EPS).

- gNB - Next Generation Node B
5G radio access network node connecting user equipment to the 5G core network.

- IPoE IP over Ethernet
A protocol that delivers IP packets directly over Ethernet without requiring a login or session establishment, commonly used in 
broadband networks in conjunction with DHCP for IP address assignment.

- LTE - Long-Term Evolution
4G wireless broadband technology and related network architecture.

- P-GW - Public Data Network Gateway
LTE/EPC network gateway managing data plane and policy enforcement.

- PDU - Protocol Data Unit
In 3GPP terminology, a PDU is a unit of information at a given protocol layer, such as an IP packet at the network layer. Specifically 
in 5G, a PDU Session represents a logical connection that carries one or more PDUs between the User Equipment (UE) and a Data Network 
(DN) through the User Plane Function (UPF). PDU Sessions support multiple types of PDUs, including IPv4, IPv6, Ethernet frames, and 
unstructured data, and are associated with one or more QoS Flows that define handling and quality requirements. The PDU framework is 
essential for managing application data transport and quality of service within the 3GPP system architecture.

- PPP - Point-to-Point Protocol
A data link layer communication protocol used to establish a direct connection between two nodes, commonly used for dial-up and 
broadband internet connections to provide authentication, encryption, and compression.

- SCONE - Standard Communication with Network Elements
Protocol allowing throughput or rate advice signaling from the network to application endpoints.

- SCONE Advisor
Logical function within network elements (e.g., UPF, P-GW) responsible for computing and sending throughput advice.

- SMF - Session Management Function
5G network function that manages sessions and enforces policies.

- UE - User Equipment
The mobile device or endpoint used by the subscriber to access the network.

- UPF - User Plane Function
5G core network element responsible for user-plane traffic routing and applying policy decisions.

- Wireline Network
Broadband network based on fixed infrastructure (e.g., DSL, cable, fiber).

# Generic Applicability and Manageability considerations

## Flow session awareness
## Per-Flow Signaling
## QoS awareness
## SCONE Hint to the Network
## Retransmission of Advised Bit-Rate
## Frequency of Updates
The rate at which SCONE updates are issued depends on flow characteristics and available computational resources. Excessively frequent 
updates may increase CPU load on network elements responsible for generating throughput advice, while infrequent updates may reduce 
advisory effectiveness. Telecommunications Service Providers may consider defining an adjustable update intervals based on application 
requirements, network capacity, and operational constraints.

## Monitoring and Logging
SCONE signaling maybe integrated into existing OSS/NMS frameworks to enable monitoring, troubleshooting, and fault isolation. 
For example, metrics of interest can include:

  - Rate of SCONE advisory messages issued per session
  - Correlation between SCONE advisories and user-plane throughput changes
  - Error conditions where SCONE signaling fails to reach the intended endpoints.

## Conformance Monitoring
Network Elements providing SCONE throughput advice may consider implementing mechanisms to measure compliance, either per 
application flow or in aggregate. This allows network operators to validate advisory effectiveness and adjust policies. 

## Standards Compliance
SCONE signaling is expected to traverse the existing data path. For example, in 3GPP-compliant networks, SCONE packets traverse 
over the Protocol Data Unit (PDU) sessions established between the User Equipment (UE) and Internet endpoints.

## Interworking with Other Congestion Management Mechanisms
SCONE operates independently of transport-layer mechanisms such as ECN or L4S. Operators MAY harmonize multiple congestion signaling 
methods by policy, or scope deployments to avoid conflicting feedback.

# SCONE Usage in a 5G Network
5G systems are built on a cloud-native Service-Based Architecture (SBA), which provides flexibility for introducing new functions such as SCONE. 
The User Plane Function (UPF) serves as the natural anchor point for SCONE signaling because it handles packet forwarding, QoS enforcement, and 
interaction with the Session Management Function (SMF) and Policy Control Function (PCF).

## Applicability of SCONE in a 5G Network

In 5G, the UPF is the on-path network element with access to subscriber policy and user-plane connectivity between the User Equipment 
(UE or the client App end-point) and the Internet. The UPF is capable of generating SCONE throughput advice per application flow, enabling 
endpoints to adjust sending rates proactively. SCONE signaling occurs over the existing data path. The following diagrams illustrate how 
throughput advice is conveyed within the 5G, highlighting the role of user-plane. network elements in signaling throughgput advice to applications.

NOTE: SCONE Advisor shown in the diagram is a logical representation and is illustrative of a function within the UPF that is responsible 
for determining the Throughput advise value. The implementation of SCONE signal is up to the network equipment vendor.

~~~~
+---------+
|   PCF   |
+---------+
     |
     v Policy Rules
+---------+
|   SMF   |
+----+----+
     | Policy Rules 
     v
+--------+                 +----------------------------+
| Client |<===============>|                            |
|   App  |     SCONE       |                            |
+--------+     Advice      |            UPF             |
|   OS   |                 |   +--------------------+   |
+--------+                 |   |     SCONE Advisor  |   |
|  Modem |                 |   +--------------------+   |
+----+---+                 +----------------------------+
     |                             |      |
     |   +-----+                   |      |
     +---+ gNB +-------------------+      |
         +-----+                          |
              |                           v
              v                    +--------------+
     +-----------------+          |  Internet    |
     | Content Provider|          +--------------+
     +-----------------+

~~~~
{: #5g-scone title="SCONE Integration within the 5G SA Network"}

## 5G specific considerations 
This section describes how the SCONE protocol can be deployed and managed within 3GPP networks, including support for SCONE packets 
over established PDU sessions.

### 3GPP Defined PDU Session Establishment Procedures
The following high-level functions, defined within 3GPP specifications, are relevant to SCONE manageability, as SCONE packets traverse 
established PDU sessions:

1. Packet Data Network (PDN) Connection / PDU Session (5G)
    A logical connection between the UE and the P-GW (in 4G) or UPF (in 5G), allowing the UE to exchange IP packets with external networks.
   Each PDN Connection/PDU Session is associated with an APN (4G) or DNN (5G).

3. IP address Allocation

    During PDN Connection/PDU Session establishment, the UE is allocated an IP address (IPv4, IPv6, or both) used for communication with
   external networks.

5. Bearer Establishment
    Data traffic flows over bearers, each defining QoS characteristics for a specific flow. In 4G, a default bearer is created for Internet
   access, while dedicated bearers may be set up for specialized services. In 5G, the equivalent construct is the QoS Flow.

7. Mobility Management
    The network ensures seamless UE mobility across cells and base stations while maintaining the ongoing session.

### PDU Session Awareness
SCONE signaling operates only over established PDU sessions. This enables network elements to unambiguously associate throughput advice with 
specific UEs and application flows. Each session is bound to a DNN (5G) or APN (4G) and to an allocated IP address, ensuring SCONE packets are 
routed precisely without affecting unrelated traffic.

### Per-Flow Signaling
Throughput advice is applied on a per-4-tuple basis. Network elements MUST maintain flow-specific context to ensure signaling correctness. 
This enables applications to receive targeted throughput advice while preventing unintended impact on unrelated flows.

### QoS and Bearer Considerations
In 5G, QoS is enforced at the granularity of QoS Flows, identified by a QoS Flow Identifier (QFI). A single PDU session can contain multiple 
QoS Flows. Operators MAY configure a distinct QFI for SCONE packets to ensure predictable handling, or allow SCONE packets to traverse the 
same bearer as user-plane traffic when no differentiated treatment is required.

The PCF and SMF MUST be capable of assigning appropriate QoS attributes to SCONE flows to ensure that congestion-control signaling is not 
degraded under high-load conditions.

### Mobility Handling
During mobility events (e.g., handover or UPF relocation), SCONE state MUST persist across control-plane and user-plane transitions. 
The SMF and UPF MUST ensure consistent delivery of SCONE packets following mobility procedures.

Where advisory logic is stateful at the UPF, operators SHOULD provide a synchronization mechanism to prevent discontinuities during mobility.

### SCONE Hint to the Network
SCONE-aware applications MUST provide hints to the network element, enabling it to generate appropriate throughput advice for a given 4-tuple. 
Such hints prevent unnecessary default rate-limiting and allow the network to generate the maximum allowable bit rate. Hints also reduce CPU 
overhead by eliminating flow classification for SCONE awareness.
   
### Retransmission of Advised Bit-Rate
Packet loss or non-delivery of SCONE advice reduces effectiveness. Both network elements and applications SHOULD support retransmission or 
periodic re-sending of SCONE packets to ensure reliable delivery. Conformance depends on both network and endpoint behavior.

### Dynamic Updates
Mobile networks may enforce dynamic rate limits during a sessions due to:

  - Changes in RAT Type (requiring updated throughput advice).
  
  - Changes in subscriber policy (exceeding usage thresholds).
  - Frequency of updates to maximum allow throughput
  - Periodic refreshes of maximim allowable throughput (Define timers for optimal and/or maximum update periodicity).

### Frequency of Updates
The rate at which SCONE updates are issued depends on flow characteristics and available computational resources. Excessively frequent 
updates may increase CPU load, while infrequent updates may reduce advisory effectiveness. Operators SHOULD define acceptable update 
periodicity based on application requirements, network capacity, and operational constraints.

### Conformance Monitoring
Network elements providing SCONE throughput advice MUST implement mechanisms to measure compliance, either per application flow or 
in aggregate. This allows operators to validate advisory effectiveness and adjust policies. SCONE protocol defines a minimum monitoring 
period for the conformance monitoring.

### Standards Compliance
All SCONE signaling occurs over the existing data path in accordance with 3GPP specifications, ensuring compatibility with 
established mobile-core procedures and avoiding protocol modifications. SCONE operates without interfering with QoS enforcement 
or subscriber policies.

### Operations Monitoring and Logging
Operators MAY integrate SCONE signaling into existing OSS/NMS frameworks to enable monitoring, troubleshooting, and fault isolation. 
Metrics of interest include:

  - Rate of SCONE advisory messages issued per session

  - Correlation between SCONE advisories and user-plane throughput changes

  - Error conditions where SCONE signaling fails to reach the UE

Integration with analytics frameworks (e.g., NWDAF in 5G) MAY be used to assess effectiveness.

# SCONE Usage in a 4G/LTE Network
In LTE/Evolved Packet Core (EPC) systems, SCONE can be integrated at the PDN Gateway (P-GW) or the Serving Gateway (S-GW). Unlike 5G, 
traffic granularity is bearer-based rather than per-flow.

Below is an example diagram illustrating SCONE integration within the P-GW:

~~~~
+---------+
|  PCRF   |
+----+----+
     | Flow
     v Policy Rules
+--------+          +-----------------+
| Client |<========>|  P-GW           |
|  App   |   SCONE  |  (SCONE Advisor)|
+--------+   advice +-------+---------+
|   OS   |                  |
+--------+                  |
|  Modem |                  |
+----+---+                  |
     |                      |
     v                      v
  +--+---+              +---+---+
  |  eNB  |--------------|  S-GW |
  +--+---+              +---+---+
                           |
                           v
                    +-------------+
                    |  Internet   |
                    +-------------+
                           |
                           v
                    +-----------------+
                    | Content Provider|
                    +-----------------+

~~~~
{: #4g-scone title="SCONE Integration within the 4G Network"}

## Applicability of SCONE in a 4G/LTE Network
- SCONE signaling maps to EPS bearers, enabling secure and targeted throughput advice between endpoints and EPC gateways.

## 4G specific considerations 

 TBD

# SCONE usage in a Wireline Network
SCONE can be deployed in wireline broadband networks at key access aggregation points such as 
Broadband Network Gateways (BNGs) or equivalent subscriber access nodes. These network elements 
serve as the originators of throughput advice, signaling maximum sustainable data rates to 
application endpoints for each subscriber session, typically identified by DHCP, PPP, or IPoE 
session contexts.

Session granularity is typically based on subscriber sessions using PPP, DHCP, or IPoE protocols. 
Below is a high-level view of SCONE within the wireline network:

~~~~

+----------------+        +-----------------+        +------------------+
|  Subscriber    |<------>|       BNG       |<------>|  Content /       |
|  Session / UE  |  SCONE |  +-----------+  |  SCONE |  Endpoint /      |
+----------------+  Advice|  |  SCONE    |  |  Advice|                  |
                          |  |  Advisor  |  |        |                  |
                          |  +-----------+  |        +------------------+
                          +-----------------+
~~~~
{: #Wireline-scone title="SCONE Integration within the Wireline Network"}

## Wireline specific considerations

TBD

## Other Miscellaneous topics
  - SCONE signaling MUST NOT require changes to how a CSP determines video policy for a flow.
  
  - The SCONE signal MUST be extensible beyond 4G/5G.

  - Receiver adaptation behavior requires further specification.
  
  - In multi-UPF deployments, only the UPF associated with a given PDU session will send throughput advice. Other UPFs may serve
    specialized roles but MUST NOT duplicate advisory functions.

By addressing these above operational considerations, SCONE can be managed effectively in mobile networks to enable adaptive bit-rate 
applications optimize their performance while allowing network operators to utilize network resources efficiently.

# Security Considerations
Security considerations are included separately in the SCONE protocol documents.  Specific to the use case description in this document, 
there are no additional security considerations.

# IANA Considerations
This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document represents collaboration, comments, and inputs from others,
including:

- Wesley Eddy
- Renjie Tang
- All the reviewers who provided invalueable input in their reviews

# Appendix

## Detailed view of the User Plane Network Element in Mobile Packet Core
This section describes 5G mobile packet core to explain the role of user-plane
network element in mobile packet core and reasons why the 5G User Plane
Function (UPF) and 4G P-GW as network elements can be considered candidates for
signaling the "throughput advice" to client-application-endpoint.  However, the
applicability extends to network architectures beyond 4G/5G networks.

The user plane network element in the 5G packet core, termed as the UPF, as shown in
Figure 1. 

~~~~
               +-----+  Nudm/Nudr  +---------+
               | PCF +-------------+ UDM/UDR |
               +--+--+             +----+----+
                   |                    |
              Npcf |      +-----+       |Nudm
                   +------+ SMF +-------+
                          +--+--+      ___  __
                             | N4     (   )(  )
   +----+   +--------+    +--+--+    (         )    +------------------+
   | UE |---| gNodeB |----| UPF |----( Internet )---| Content Provider |
   +----+   +--------+ N3 +- -+-+ N6  (        )    +------------------+
                              | N9     (__(___)
                            +-+---+
                            | UPF |
                            +-----+
~~~~
{: #5g-diagram title="5G Mobile Network Architecture"}

In the 4G packet core, the P-GW (as shown in Figure 2) performs the
same role as the UPF does in the 5G mobile packet core.

~~~~
                    +-----+
                    | HSS |
                    +-----+
                       |
                    +-----+          +------+
                    | MME |          | PCRF |
                   /+-----+\         +------+
                  /         \            |
                 /           \           |         ___  __
                /             \          |        /   )(  \
   +----+   +-----+        +------+  +------+    (         )    +----------+
   | UE |---| eNB |--------| S-GW |--| P-GW |----( Internet )---| Content  |
   +----+   +-----+   S1u  +------+  +------+ SGi (        _)   | Provider |
                                                   (__(___)     +----------+
 
~~~~
{: #4g-diagram title="4G Mobile Network Architecture"}

## 5G Mobile Network Architecture
The UPF is a fundamental component of the 3GPP's 5G packet core network
architecture. UPF is on the data path between the end-user and the Internet, has
access to subscriber policy via standard 3GPP N4 interface and is responsible for
routing and forwarding user data packets. UPF is the anchor point between the
mobile infrastructure and the Packet Data Network.  The UPF is responsible for
functions such as:

- Packet routing, forwarding, and interconnection to the Data Network (Internet) 
- Allocation of User Equipment (UE) IP Address/prefix, in conjunction with Session Management Function (SMF)
- Quality of Service policy enforcement
- Handling of traffic filtering, steering and application detection
- Traffic usage reporting

Note: This is not an exhaustive list of UPF functions.  For details refer to
{{5G-Arch}}.

To accomplish above mentioned functions, the UPF has four distinct reference
points (interfaces)  as defined by the 3GPP and as shown in the figure 1 above:

1. The N3 interface is between the UPF and the 5G Base station.

2. The N4 interface is a connection between the UPF and the Session Management Function (SMF).

3. The N6 interface is between the UPF and the public data network or the Internet.

4. The N9 interface is between instances of UPFs.

## N3 Interface
The N3 interfaces transfers user plane traffic, that is, user data packets
between the gNodeB and the UPF.  It uses GPRS Tunneling Protocol - User Plane
or GTP-U.  It replaces the S1-U interfaces from the 4G mobile packet core.

## N4 Interface
The N4 interface connects the UPF and the 5G Session Management Function (SMF).
Through N4, the SMF informs the UPF about the subscriber policy and data plans.
Additionally, this interface is used to manage session setup, modification,
deletion, and for configuring QoS and forwarding rules for user data. The QoS 
rules contain parameters such as MBR. The N4 interface
among others uses Packet Forwarding Control Protocol (PFCP).

Note: SMF also interacts with Policy Control Function (PCF) for functions such
as QoS and Charging policy rules, Unified Data Management (UDM) and Unified
Data Repository (UDR) for functions such as subscription data and policy plans.

## N6 Interface
The N6 interface connects the UPF to external Data Networks, similar to the SGi
interface between the P-GW and the external Data Network for access to services
and applications.  The interface supports various transport protocols over IP.

## N9 Interface
This interface interconnects two or more UPFs when used in a data path.  The interface uses GTP-U protocol for user 
traffic tunneling including roaming.

Note: In the scenario of 2 or more UPFs in the data path, only one UPF that has access to subscriber policy would send "throughput 
advice" to the client-application-endpoint.

## User Plane Interface Between UPF and UE
This section describes the N3 interface (between the UPF and gNodeB or gNB) and
the air interface between the gNB and UE.  For purposes of nomenclature, a
Protocol Data Unit (PDU) session is a logical path between a UE and UPF to
carry packets belonging to one or more IP flows between UE and DN.  A PDU
session within a 5G mobile network consists of an air-interface between UE and
gNB and GTP-U tunnel between gNB and UPF (N3 interface). Application traffic flows with different QoS requirements get 
mapped to different QoS treatments based on packet filters and QoS rules configured on the UPF and UE. 
Below is an example of data flow to/from a UE to the UPF.

1. Uplink Data Flow
    - Apps that are hosted on UE that generate application packets for communication (e.g. web browsing, video streaming).
    - These packets are transmitted to the gNB over the air interface and get mapped to different QoS treatments based on packet
      filters and QoS rules provided to the UE
    - N3 Encapsulation and Forwarding
         1. The gNB then encapsulates this user-plane data using GTP-U.
         2. It then forwards the encapsulated packets over the N3 interface to the UPF in the 5G mobile packet core.
    - UPF Routes Data to External Networks.
         1. Within the UPF, UPF then removes the GTP-U header, processes the packet, and routes it over the N6 interface
            toward the destination (Internet, enterprise network, cloud services, etc.).

2. Downlink Data Flow
    - UPF receives incoming data in downlink direction at N6 interface (e.g. from the Internet).
    - The UPF encapsulates incoming data using GTP-U and forwards it over the N3 interface to the gNB. It maps traffic flows with
      different QoS requirements to different QoS treatments based on packet filters and QoS rules configured by SMF.
    - The gNB forwards the packets to the UE over the air-interface.  UE-side modem stack then transparently passes the application
      packets to the app hosted on the UE.

In summary, the UPF is responsible for packet routing and forwarding, packet
inspection and filtering, participating in subscriber and flow policy enforcement, inline services (NAT, firewall, DNS etc) and QoS handling.


