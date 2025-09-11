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

This document addresses the applicability of the SCONE signal in mobile networks and the operational considerations for managing it in operator deployments. 
It describes how 3GPP user-plane network elements, including the User Plane Function (UPF) and Packet Data Network Gateway (P-GW), can generate “throughput
advice” by rate-limiting a UDP 4-tuple to indicate an upper bound on achievable bitrate for application flows. This advice enables implementation of the SCONE
protocol in support of adaptive applications such as video streaming. While the focus is on mobile networks, the considerations are also relevant to other access networks.

--- middle

# Introduction

This document describes the applicability and manageability of the SCONE protocol in both operator networks and application endpoints. 
The primary focus is on mobile networks, where user-plane functions such as the UPF (5G) or P-GW (4G) are capable of generating throughput 
advice to guide adaptive bit-rate applications. However, the same concepts may also apply to other access networks where similar advisory mechanisms are useful.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Problem Statement
Existing transport feedback mechanisms, such as TCP congestion control or Explicit Congestion Notification (ECN), typically react only 
after congestion occurs and may not provide timely or accurate guidance in mobile environments. They also offer limited visibility into 
operator-managed resources. SCONE addresses this gap by enabling network elements to communicate advisory information directly to endpoints, 
allowing applications to adjust proactively to the achievable throughput.

This document is intended to outline SCONE applicability and mangeability in the operator network and is not a protocol specification.

# Applicability of SCONE Signal in Mobile Networks

Mobile and access networks frequently encounter variable conditions due to congestion, radio interference, or dynamic resource allocation. 
Even applications that use adaptive bit-rate may experience degraded performance or inefficiencies under such conditions. The SCONE protocol enables 
network elements to provide throughput advice directly to applications, allowing them to adjust sending rates proactively, improving end-user 
Quality of Experience (QoE) while helping operators manage network resources efficiently. This document proposes leveraging 3GPP user-plane 
network elements, including the UPF in 5G and the PDN-GW in 4G, to deliver throughput advice over the existing data path in accordance with 
3GPP standards.

## Implementing SCONE in Mobile Networks

In 5G, the User Plane Function (UPF), and in 4G, the Packet Data Network Gateway (P-GW), are on-path network elements with access to 
subscriber policy and data-plane connectivity between the UE and the Internet. These elements can generate SCONE throughput advice per 
application flow, enabling endpoints to adjust sending rates proactively in response to network conditions. SCONE signaling occurs over 
the existing data path in accordance with 3GPP standards.

The following diagrams illustrate how throughput advice is conveyed within the 5G and 4G packet core, highlighting the role of user-plane 
network elements in signaling rate guidance to applications.

~~~~
                          +---------+
                          |   PCF   |
                          +---------+
                               | Subscriber
                               V Policy Rules
                          +---------+
                          |   SMF   |
                          +----+----+
                               | Subscriber
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
                               | Subscriber
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
{: #4g-scone title="SCONE Integration with Vido Policy in 4G N/W"}

# SCONE Manageability & Operational considerations
This sections describes how SCONE protocol can be supported on a 3GPP network including supporting SCONE packets
over a given PDU session as defined within the 3GPP specifications.
This section describes how the SCONE protocol can be deployed and managed within 3GPP networks, including support for SCONE packets 
over established PDU sessions. Building on the applicability of SCONE signaling in mobile networks, network elements such as the UPF (5G) 
and P-GW (4G) can provide throughput advice while ensuring that signaling occurs per flow without requiring changes to existing data paths.

Operational considerations include:
- PDU Session Awareness: SCONE signaling occurs over established PDU sessions, allowing network elements to identify the UE and application flows for which
throughput advice is relevant.
- Per-Flow Signaling: Throughput advice is applied on a per-application or per-4-tuple basis, enabling precise rate guidance without impacting unrelated traffic.
- Dynamic Updates: Network conditions such as congestion, radio resource availability, or sudden changes in user load may require frequent updates to throughput advice. The network element must be capable of generating updated SCONE signals dynamically to maintain Quality of Experience (QoE) for applications.
- Conformance Monitoring: Network elements providing SCONE advice should have mechanisms to measure compliance with the advised throughput, either per flow or in aggregate, to ensure that rate guidance is effective.
- Standards Compliance: All SCONE signaling occurs over the existing data path in accordance with 3GPP specifications, ensuring compatibility with established mobile core procedures and avoiding protocol changes.

By addressing these operational considerations, SCONE can be managed effectively in mobile networks, enabling adaptive applications to optimize 
their performance while allowing operators to utilize network resources efficiently.



## 3GPP defined PDU Session establishment procedures
The sections below provide an overview of high-level functions within the 3GPP specifications to support 
the PDU session establishment after which the SCONE packets will run over the established PDU session.  

### Packet Data Network (PDN) Connection / PDU Session (5G)
This is the logical connection established between the UE and the Packet Data Network Gateway (P-GW in 4G) 
or User Plane Function (UPF in 5G). It allows the UE to exchange IP packets with external networks. 
Each PDN Connection/PDU Session is associated with a specific Access Point Name (APN), which identifies 
the type of service or external network the UE wants to connect to (e.g., "internet" for general internet access).

### IP address allocation
During the establishment of a PDN Connection/PDU Session, the UE is allocated an IP address (IPv4, IPv6, or both).
This IP address is used for communication with the internet.

### Bearer establishment
Data traffic flows over bearers. A bearer defines the QoS (Quality of Service) characteristics for a specific 
data flow. For internet access, a default bearer is established first, and dedicated bearers can be set up 
for specific services requiring different QoS.

### Mobility Management
The network handles the UE's mobility (e.g., moving between cells or base stations) while maintaining the 
ongoing data connection.

## Applicability & Mangeability of the SCONE Protocol in the 3GPP network
The sections below describes support for SCONE protocol within the 3GPP networks.

## SCONE signal Hint from Client to the Network
In 3GPP networks (4G/5G), a User Equipment (UE) connects to the internet by establishing data sessions that
traverse various network elements. The key process involves allocating an IP address to the UE and routing its
data traffic through the mobile network's core to the external data networks including the internet. As this
connection to the Internet is established and once the client App on the UE starts communicating with the 
application content provider, a hint for SCONE usage will allow UPF to then look for a SCONE packet for this
specific user connection and avoid PGW/UPF any unnecessary CPU cycles for non-ABR video connections.
The section below provides a more detailed information on the UE and the mobile network for connecting to the 
external network.

## Retransmission of advised bit-rate 
Editor's note: 
- address potential packet loss and no support for ACK from the end-user client
- what support the netwwork element needs from SCONE client and sender (server) to enable retx

## Measuring conformance of advised bit-rate
As the network element capable of advising bit-rate limit, the network element also would need capabilities to measure conformance on the advised bit-rate. 

Issue 35 [https://github.com/ietf-wg-scone/scone/issues/35]
- Need to determine if the conformance is to be measured as an aggregate or on a per flow basis.

Presentation given at interim session 6 provides results based on experimentation that recommends a suitable size for time window to be 120 seconds. This value is compatible with existing VOD applications when ~2 mbps is the advised bitrate.
- [https://datatracker.ietf.org/meeting/interim-2025-scone-06/materials/slides-interim-2025-scone-06-sessa-time-window-duration-for-bitrate-measurement-00.pdf]

## Dynamic updates
In networks, for example - radio networks, the avaible capacity of the network can dynamically change for a period of time or there could be sudden increase of network users, these could result in change of throughput advice for a particular scone capable flow. These changes need to be dynamically and immidiately updated in the rate signal to avoid unnecesarry rate shaping or degradated QoE. This means the network elements need to be able to initiate the sending of the rate signal if there is not sufficient frequency of scone packets send for that particular flow. 
Editor's note:
- Discuss Use-cases that would require the network element to update the advised bit-rate for a flow.
- How soon the netwrok element is expected to send the updated advised bit-rate to the client?
- What support it needs from the scone sender (server) to enable this? For e.g., what is the Max periodicity at
which server is supposed to send Scone packets.

## Frequency of updates to SCONE packets by the Network Element
Editor's note:
- Consider impacts on CPU utilization of network element (UPF/PGW)
- Acceptable periodicity at which network element (UPF/PGW) can update the SCONE packet from a CPU load stand point.

## Other open issues
- SCONE signaling MUST NOT require changes to how a CSP determines its video policy for a given flow. That is there is MUST not be any dependency between a CSP's video policy and the SCONE protocol.

- SCONE signal MUST be extensible to networks beyond 4G/5G network.

- discussion on how the applications/receivers can adapt to the rate signals.
- A question was raised RE one or more network elements in a path may send advised bit-rate. Towards that point.
A typical mobile network deployment, there is only one UPF (PGW) which would send an advised bit-rate.
If additional UPF/PGW are deployed, they may have specific function but will not be configured to communicate
maximum bit-rate. 

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
architecture. UPF is the data path between the end-user and the Internet, has
access to subscriber policy via standard 3GPP interface and is responsible for
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
deletion, and for configuring forwarding rules for user data.  The N4 interface
among others uses Packet Forwarding Control Protocol (PFCP).

Note: SMF also interacts with Policy Control Function (PCF) for functions such
as QoS and Charging policy rules, Unified Data Management (UDM) and Unified
Data Repository (UDR) for functions such as subscription data and policy plans.

## N6 Interface

The N6 interface connects the UPF to external Data Networks, similar to the SGi
interface between the P-GW and the external Data Network for access to services
and applications.  The interface supports various trasnport protocols over IP.

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
gNB and GTP-U tunnel between gNB and UPF (N3 interface).  IP flows (aka service
data flows or SDFs) may belong to one or more services.  All the service data
flows with the same QoS maps onto one PDU session.  Below is an example of data
flow to/from a UE to the UPF.

1. Uplink Data Flow
    - Apps that are hosted on UE that generate application packets for communication (e.g. web brownsing, video streaming).
    - These packets are transmitted to the gNB over the air interface.
    - N3 Encapsulation and Forwarding
         1. The gNB then encapsulates this user-plane data using GTP-U.
         2. It then forwards the encapsulated packets over the N3 interface to the UPF in the 5G mobile packet core.
    - UPF Routes Data to External Networks.
         1. Within the UPF, UPF then removes the GTP-U header, processes the packet, and routes it over the N6 interface
            toward the destination (Internet, enterprise network, cloud services, etc.).

2. Downlink Data Flow
    - UPF receives incoming data in downlink direction at N6 interface (e.g. from the Internet).
    - The UPF encapsulates incoming data using GTP-U and sends it back over the N3 interface to the gNB.
    - The gNB forwards the packets to the UE over the air-interface.  UE-side modem stack then transparently passes the application packets to the app hosted on the UE.

In summary, the UPF is responsible for packet routing and forwarding, packet
inspection and filtering, subscriber policy enforcement, inline services (NAT, firewall, DNS etc) and QoS handling.

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

