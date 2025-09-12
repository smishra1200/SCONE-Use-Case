---
title: "Applicability & Manageability of SCONE signal for a mobile network"
abbrev: "SCONE Applicability & Manageability"
docname: draft-mishra-scone-applicability-manageablity-01
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
    ins: A. Tomar
    name: Anoop Tomar
    organization: Meta
    email: anooptomar@meta.com
  -
    ins: K. Abbas
    name: Khurram Abbas
    organization: Verizon
    email: khurram.abbas@verizonwireless.com
  -
    ins: Z. Sarker
    name: Zaheduzzaman Sarker
    organization: Nokia
    email: zaheduzzaman.sarker@nokia.com


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

  Mishra-2025:
    target: https://datatracker.ietf.org/meeting/interim-2025-scone-01/materials/slides-interim-2025-scone-01-sessa-leveraging-the-user-plane-function-for-network-side-advisory-signal-00
    title: Leveraging the user plane function for network-side advisory signal
    author:
    -
      name: Sanjay Mishra
    date: 2025-02-06
 
--- abstract

This document addresses the applicability and mangeability of the SCONE signal in mobile networks and the operational considerations for managing it in operator deploymenta including an ability to provide throughput advice to the application endpoints. 

--- middle

# Introduction

Existing transport feedback mechanisms, such as TCP congestion control or Explicit Congestion Notification (ECN), typically react only 
after congestion occurs and may not provide timely or accurate guidance in mobile environments. They also offer limited visibility into 
operator-managed resources. SCONE addresses this gap by enabling network elements to communicate advisory information directly to endpoints, 
allowing applications to adjust proactively to the achievable throughput. Purpose of this document is to address applicability and manageability 
of the SCONE protocol in both the operator networks and application endpoints. The primary focus is on mobile networks, where user-plane functions 
such as the UPF (5G) or P-GW (4G) are capable of generating throughput advice to guide adaptive bit-rate applications. However, the same concepts
may also apply to other access networks where similar advisory mechanisms are useful.

This document focuses on SCONE's applicability and manageability in the operator network and is not a protocol specification.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Applicability of SCONE Signal in Mobile Networks

Mobile and access networks frequently encounter variable conditions due to congestion, radio interference, or dynamic resource allocation. 
Even applications that use adaptive bit-rate may experience degraded performance or inefficiencies under such conditions. The SCONE
protocol enables network elements to provide throughput advice directly to applications, allowing them to adjust sending rates
proactively, improving end-user Quality of Experience (QoE) while helping operators manage network resources efficiently. This document
proposes leveraging 3GPP user-plane network elements, including the UPF in 5G and the P-GW in 4G, to deliver throughput advice over the
existing data path in accordance with 3GPP standards.

## Scope of Deployment

SCONE is intended for deployment within operator-controlled networks, such as 3GPP mobile systems, fixed broadband access networks, or
enterprise-managed domains. In these environments, network functions (e.g., UPF, P-GW, or equivalent user-plane entities) are capable 
of originating throughput advisory in response to locally observed policy and network conditions. SCONE is not designed for open,
unmanaged Internet environments where no single administrative entity has end-to-end control.

## Implementing SCONE in Mobile Networks

In 5G, the User Plane Function (UPF), and in 4G, the Packet Data Network Gateway (P-GW), are on-path network elements with access to 
subscriber policy and data-plane, aka User Plane, connectivity between the UE and the Internet. These elements are capable to generate SCONE throughput advice per 
application flow, enabling endpoints to adjust sending rates proactively in response to network conditions. SCONE signaling occurs over 
the existing data path in accordance with 3GPP standards.

The following diagrams illustrate how throughput advice is conveyed within the 5G and 4G packet core, highlighting the role of user-plane 
network elements in signaling thourhgput advice to applications.

~~~~
                          +---------+
                          |   PCF   |
                          +---------+
                               | Subscriber
                               V Policy Rules
                          +---------+
                          |   SMF   |
                          +----+----+
                               | Flow
                               v Policy Rules
+--------+               + +---------+-+
| Client |/--------------\ |  SCONE  | |       __
|   App  |\--------------/ | Advisor | |    __(  )__
+--------+     SCONE     | +---------+ |   (        )   +----------+
|   OS   |  (advised bit |             +--( Internet )--+ Content  |
+--------+   rate and    |     UPF     |   (         )  | Provider |
|  Modem |   other IEs)  |             |    (__)(___)   +----------+
+----+---+               +------+------+      
     |                          |
     |         +-----+          |    
     +---------+ gNB +----------+     
               +-----+       
~~~~
{: #5g-scone title="SCONE Integration with Video Policy in 5G SA N/W"}

Similarly, the SCONE signal for 4G network is shown below.  

~~~~
                          +---------+
                          |  PCRF   |
                          +----+----+
                               | Flow
                               v Policy Rules
+--------+               + +---------+-+
| Client |/--------------\ |  SCONE  | |       __
|   App  |\--------------/ | Advisor | |    __(  )__
+--------+     SCONE     | +---------+ |   (        )   +----------+
|   OS   |  (advised bit |             +--( Internet )--+ Content  |
+--------+   rate and    |     P-GW    |   (         )  | Provider |
|  Modem |   other IEs)  |             |    (__)(___)   +----------+
+----+---+               +------+------+      
     |                          |
     |         +-----+       +--+---+
     +---------+ eNB +-------+ S-GW |
               +-----+       +------+
~~~~
{: #4g-scone title="SCONE Integration with Video Policy in 4G N/W"}

# SCONE Manageability & Operational considerations
SCONE is designed to be transport-agnostic (not bound to TCP/QUIC semantics). It provides a signaling path at the network/user plane
boundary rather than per-flow congestion feedback and is explicitly designed to work in 3GPP / operator-controlled domains, where 
the UPF or another network function can generate the throughput advisory. Towards that goal, this section describes how the SCONE protocol
can be deployed and managed within 3GPP networks, including support for SCONE packets over established PDU sessions. 

## 3GPP defined PDU Session establishment procedures
The sections below provide an overview of high-level functions within the 3GPP specifications that are relevant to SCONE manageability 
to support the PDU session establishment after which the SCONE packets will run over the established PDU session.  

1. Packet Data Network (PDN) Connection / PDU Session (5G)
    This is the logical connection established between the UE and the Packet Data Network Gateway (P-GW in 4G) 
or User Plane Function (UPF in 5G). It allows the UE to exchange IP packets with external networks. 
Each PDN Connection/PDU Session is associated with a specific Access Point Name (APN), which identifies 
the type of service or external network the UE wants to connect to (e.g., "internet" for general internet access).

2. IP address allocation
    During the establishment of a PDN Connection/PDU Session, the UE is allocated an IP address (IPv4, IPv6, or both).
This IP address is used for communication with the internet.

3. Bearer establishment
    Data traffic flows over bearers. A bearer defines the QoS (Quality of Service) characteristics for a specific 
data flow. For internet access, a default bearer is established first, and dedicated bearers can be set up 
for specific services requiring different QoS.

4. Mobility Management
    The network handles the UE's mobility (e.g., moving between cells or base stations) while maintaining the 
ongoing data connection.

Given above context, the following section describes key manageability and operations considerations:

## PDU Session Awareness

SCONE signaling occurs over established PDU sessions, allowing network elements to identify the UE and application flows for which throughput advice is   relevant. Each session is associated with an Access Point Name (APN) and IP address allocation (IPv4, IPv6, or both), enabling precise routing of SCONE packets without affecting other traffic.

## Per-Flow Signaling

Throughput advice is applied on a per-4-tuple basis. This enables applications to receive targeted thourhgput advice while preventing unintended impact on unrelated flows. Network elements must maintain flow-specific context for SCONE signaling to ensure correctness.

## QoS and Bearer Considerations

SCONE signaling may be carried over either the default bearer or a dedicated bearer, depending on operator policy. Operators may configure a distinct QoS Flow Identifier (QFI) for SCONE packets to ensure predictable handling, or alternatively allow SCONE packets to traverse the same bearer as user-plane traffic when no differentiated treatment is required.

The PCF (Policy Control Function) and SMF (Session Management Function) MUST be capable of assigning appropriate QoS attributes to SCONE flows to prevent congestion-control signaling from being degraded under high-load conditions.

## Mobility Handling Considerations

When mobility events (e.g., handover or UPF relocation) occur, SCONE state may need to persist across control-plane and user-plane transitions. The SMF and UPF MUST ensure that SCONE packets continue to be delivered consistently after mobility procedures are complete.

Where stateful advisory logic is deployed at the UPF, operators SHOULD provide a synchronization mechanism to prevent advisory discontinuities during mobility.

## SCONE Hint to the Network

A hint from the SCONE aware application is important for the network element as it can only rely on the hint to set throughput advise on the SCONE packet
for a given 4-tuple. This also helps network avoid any additional CPU cycles to determine if a given PDU session is a SCONE aware application.
   
## Retransmission of Advised Bit-Rate

Packet loss or non-delivery of SCONE advice may reduce the effectiveness of thoughput advice. Network elements and applications should support retransmission or periodic re-sending of SCONE packets to ensure that throughput advice is received reliably. Conformance to the advised bit-rate depends on both network and endpoint behavior.

## Dynamic Updates

Network conditions in mobile environments can change rapidly due to congestion, radio resource allocation, or sudden variations in user load. Mobile networks also have the concept of Guaranteed Bit Rate (GBR) and Maximum Bit Rate (MBR) and the flows can be set to any of these rate limits and change status during the session which would impact the bitrate allocation to the flow. If the client is on mobile network that has MBR per service then the client need to adapt to the any changes on the MBR value to avoid QOE artifacts, hence, timely SCONE signals will be need. SCONE-capable network elements must be able to generate updated throughput advice dynamically, ensuring that adaptive applications can respond promptly to maintain QoE. The frequency and granularity of updates should balance responsiveness with CPU and network overhead.

## Frequency of Updates

The rate at which a network element issues SCONE updates depends on flow characteristics and available computational resources. Excessively frequent updates may increase CPU load on UPF/P-GW elements, while infrequent updates could reduce the effectiveness of throughput advice. Operators should define acceptable update periodicity based on application requirements, network capacity, and operational constraints.

## Conformance Monitoring

Network elements providing SCONE throughput advice should have mechanisms to measure compliance with the advised throughput, either per application flow or in aggregate. This allows operators to validate that throughput advice is effective and to adjust signaling or network policies if necessary. SCONE protocol defines a monitoring period for the conformance monitoring.

## Standards Compliance

All SCONE signaling occurs over the existing data path in accordance with 3GPP specifications, ensuring compatibility with established mobile core procedures and avoiding protocol changes. SCONE operates without interfering with standard QoS enforcement or subscriber policies.

## Operations Monitoring and Logging

Network Operators MAY integrate SCONE signaling into their existing network management systems (NMS/OSS) to enable monitoring, troubleshooting, and fault isolation.

Metrics of interest include:

    Rate of SCONE advisory messages issued per session,

    Correlation between SCONE advisories and user-plane throughput changes,

    Error conditions where SCONE signaling fails to reach the UE.

Integration with existing telemetry frameworks (e.g., 3GPP NWDAF for analytics) MAY be used to assess the effectiveness of SCONE advisories and their impact on service quality.

## Interworking with Other Congestion Management Mechanisms

SCONE throughput advisory operate independently of transport-layer mechanisms such as ECN or L4S. Operators MAY want to ensure that when multiple congestion signaling methods are deployed concurrently, they are either harmonized by policy or scoped to avoid conflicting feedback.

## Other open issues
  - SCONE signaling MUST NOT require changes to how a CSP determines its video policy for a given flow. That is there MUST not be any dependency between a CSP's video policy and the SCONE protocol.

  - SCONE signal MUST be extensible to networks beyond 4G/5G network.

  - discussion on how the applications/receivers can adapt to the rate signals.
  - A question was raised RE one or more network elements in a path may send advised bit-rate. Towards that point.
  - A typical mobile network deployment may have multiple UPF deployed, however, for one PDU session there typically is only one UPF (PGW) which would send an advised bit-rate. If additional UPF/PGW are deployed, they may have specific function but will not be configured to communicate maximum bit-rate for the PDU session..

By addressing these operational considerations, SCONE can be managed effectively in mobile networks, enabling adaptive applications to optimize 
their performance while allowing operators to utilize network resources efficiently.


# Detailed view of the User Plane Network Element in Mobile Packet Core

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
gNB and GTP-U tunnel between gNB and UPF (N3 interface). Application traffic flows with different QoS requirements get mapped to different QoS treatments based on packet filters and QoS rules configured on the UPF and UE. 
Below is an example of data flow to/from a UE to the UPF.

1. Uplink Data Flow
    - Apps that are hosted on UE that generate application packets for communication (e.g. web browsing, video streaming).
    - These packets are transmitted to the gNB over the air interface and get mapped to different QoS treatments based on packet filters and QoS rules provided to the UE
    - N3 Encapsulation and Forwarding
         1. The gNB then encapsulates this user-plane data using GTP-U.
         2. It then forwards the encapsulated packets over the N3 interface to the UPF in the 5G mobile packet core.
    - UPF Routes Data to External Networks.
         1. Within the UPF, UPF then removes the GTP-U header, processes the packet, and routes it over the N6 interface
            toward the destination (Internet, enterprise network, cloud services, etc.).

2. Downlink Data Flow
    - UPF receives incoming data in downlink direction at N6 interface (e.g. from the Internet).
    - The UPF encapsulates incoming data using GTP-U and forwards it over the N3 interface to the gNB. It maps traffic flows with different QoS requirements to different QoS treatments based on packet filters and QoS rules configured by SMF.
    - The gNB forwards the packets to the UE over the air-interface.  UE-side modem stack then transparently passes the application packets to the app hosted on the UE.

In summary, the UPF is responsible for packet routing and forwarding, packet
inspection and filtering, participating in subscriber and flow policy enforcement, inline services (NAT, firewall, DNS etc) and QoS handling.

# Security Considerations

Security considerations are included separately in the SCONE protocol documents.  Specific to the use case description in this document, there are no additional security considerations.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgments
{:numbered="false"}

This document represents collaboration, comments, and inputs from others,
including:

- Wesley Eddy
- Renjie Tang

