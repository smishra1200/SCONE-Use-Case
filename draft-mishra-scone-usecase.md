---
title: "A Use Case for SCONE Implementation"
abbrev: "SCONE Use Case"
docname: draft-mishra-scone-usecase-00
category: info

ipr: trust200902
area: Web and Internet Transport
workgroup: SCONE
keyword: Throttling
keyword: Adaptive Bit-Rate Video

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

This document describes 3GPP network
elements that are capable of rate-limiting a UDP 4-tuple to communicate an upper bound
on achievable bitrate termed "throughput advice".

--- middle

# Introduction

This document describes utilizing the User Plane Function (UPF) in 5G networks
and packet data network gateway in 4G networks (PDN-GW or P-GW and also
referred as a PGW) to transport SCONE signal between the client-application
endpoint on a User Equipment (UE) and the network element (UPF/PDN-GW) in the
mobile networks. Specifically, this use case focuses on using UPF and PDN-GW to
exchange bi-directional communications with client-application end-point on the
UE. 

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Overview of User Plane Network Element in Mobile Packet Core

This section describes 5G mobile packet core to explain the role of user-plane
network element in mobile packet core and reasons why the 5G User Plane
Function (UPF) and 4G P-GW as network elements can be considered candidates for
signaling the "throughput advice" to client-application-endpoint.  However, the
applicability extends to network architectures beyond 4G/5G networks.

The user plane network element in the 5G packet core is the UPF, as shown in
Figure 1. In the 4G packet core, the P-GW (as shown in Figure 2) performs the
same role as the UPF does in the 5G mobile packet core.

The UPF is a fundamental component of the 3GPP's 5G packet core network
architecture. UPF is the data path between the end-user and the Internet, has
access to subscriber policy via standard 3GPP interface and is responsible for
routing and forwarding user data packets. UPF is the anchor point between the
mobile infrastructure and the Packet Data Network.  The UPF is responsible for
functions such as:

- Packet routing, forwarding, and interconnection to the Data Network (DN) 
- Allocation of User Equipment (UE) IP Address/prefix
- Quality of Service policy enforcement
- Handling of traffic filtering and packet inspection
- Traffic usage reporting

Note: This is not an exhaustive list of UPF functions.  For details refer to
{{5G-Arch}}.

To accomplish above mentioned functions, the UPF has four distinct reference
points (interfaces)  as defined by the 3GPP and as shown in the figure below:

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

This interface interconnects two or more UPFs when used in a data path.  The interface uses GTP-U protocol for user traffic tunneling including roaming.

Note: In the scenario of 2 or more UPFs in the data path, only one UPF that has access to subscriber policy would send "throughput advice" to the client-application-endpoint.

# User Plane Interface Between UPF and UE

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
         1. Within the UPF, UPF then removes the GTP-U header, processes the packet, and routes it over the N6 interface toward the destination (Internet, enterprise network, cloud services, etc.).

2. Downlink Data Flow
    - UPF receives incoming data in downlink direction at N6 interface (e.g. from the Internet).
    - The UPF encapsulates incoming data using GTP-U and sends it back over the N3 interface to the gNB.
    - The gNB forwards the packets to the UE over the air-interface.  UE-side modem stack then transparently passes the application packets to the app hosted on the UE.

In summary, the UPF is responsible for packet routing and forwarding, packet
inspection and filtering, subscriber policy enforcement, and QoS handling.  

## Significance of UPF from SCONE Perspective

Through the N4 interface, UPF gets access to the subscriber policy and data plans. This enables UPF to send the throughput advice based on the subscriber data plan to the client application end point. The same interface can be used to get RAN related information if required (e.g. congestion information).

The UPF is on the data path at the mobile packet core network element that routes and forwards application packets between the gNodeB and the DN (e.g. the Internet), it has access to subscriber policy and potentially can get RAN related information if required. 

Usually mobile networks are deployed with multi-verndor support meaning the RAN base station can belong to differnet vendors and might not implement all the features in all the base stations, however, UPFs sits in an unique positions that all the userplane traffic via different base stations would be routed through it. 

In a nutshell, Userplane traffic that carries Internet traffic routed via UPF, hence, it has the best position to send the throughput advice to client application over the data-path.

## 4G Mobile Network Architecture

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
   | UE |---| eNB |--------| S-GW |--| P-GW |----(    DN    )---| Content  |
   +----+   +-----+   S1u  +------+  +------+ SGi (        _)   | Provider |
                                                   (__(___)     +----------+

~~~~
{: #4g-diagram title="4G Mobile Network Architecture"}


# Implementing SCONE In the Mobile Network

As described in sections above, UPF is the 3GPP on-path "network element" that has access to subscriber policy and provides the data pipe connectivity between UE and the Internet over the air interface.  UPF is a network element that is capable of SCONE signaling over the data path.

Below is a high-level view of SCONE signal path in a 5G network.  Please see {{Mishra-2025}} for a more complete version of this diagram.

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
|   App  |\--------------/ | advisor | |    __(  )__
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

Similarly, the SCONE signal for 4G network is shown below.  Please see {{Mishra-2025}} for a more complete version of this diagram.

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

# SCONE Signal Requirements for the mobile networks

- UPF/5G or PGW/4G MUST be able to distinguish QUIC protocol versions with the explicit flow detection for any given SCONE enabled appication traffic .

- UPF/P-GW SHOULD send "throughput advice" and other metadata using on-path SCONE signaling to the client-application-endpoint based on subscriber data-plans.

- Client-application endpoint MAY send acknowledgement receipt of throughput advisory signal from the network element using the SCONE signal.

- SCONE signaling MUST NOT require changes to how a CSP determintes its video policy for a given flow.  (No dependency between a CSP's video policy and the SCONE protocol).

- Dynamic update - "throughput advice" MAY change during the ongoing flow and UPF/PGW SHOULD be able to send "throughput advice" to client-application-endpoint as soon as possible.

- Applications MUST self-adapt the video flow max bit-rate to "throughput advice" value.

- SCONE signal MUST be extensible to networks beyond 4G/5G network.

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

