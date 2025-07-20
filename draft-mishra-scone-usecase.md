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

This document identifies applicability of SCONE signal in a mobile network and outlines operational considerations, 
or manageability of SCONE signal in the operator network. Importantly, this document also describes 3GPP network
elements that are capable of rate-limiting a UDP 4-tuple to communicate an upper bound
on achievable bitrate termed "throughput advice" to implement SCONE protocol. 

--- middle

# Introduction

This document describes applicablity and manageablity of SCONE protocol in the networks and applicaiton endpoints. 
It focuses on mobile networks, however, this document is also applicable to other access networks.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# User Plane Network Element in Mobile Packet Core

This section describes 5G mobile packet core to explain the role of user-plane
network element in mobile packet core and reasons why the 5G User Plane
Function (UPF) and 4G P-GW as network elements can be considered candidates for
signaling the "throughput advice" to client-application-endpoint.  However, the
applicability extends to network architectures beyond 4G/5G networks.

The user plane network element in the 5G packet core, termed as the UPF, as shown in
Figure 1. In the 4G packet core, the P-GW (as shown in Figure 2) performs the
same role as the UPF does in the 5G mobile packet core.

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

# Applicability of SCONE Signal in Mobile Networks

The UPF is a data path mobile packet core network element that routes
and forwards application packets between the gNodeB and the DN and it
has access to subscriber policy via standard 3GPP N3 interface. 

As a result, UPF is in the best position to send the throughput advice to client application over the data-path.

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
   | UE |---| eNB |--------| S-GW |--| P-GW |----( Internet )---| Content  |
   +----+   +-----+   S1u  +------+  +------+ SGi (        _)   | Provider |
                                                   (__(___)     +----------+
 
~~~~
{: #4g-diagram title="4G Mobile Network Architecture"}


## Implementing SCONE In the Mobile Network

As described in sections above, UPF is the 3GPP on-path "network element" that has access to subscriber policy and provides
the data pipe connectivity between UE and the Internet. UPF is a network element that is capable of SCONE signaling over the
data path.

Below is a high-level view of SCONE signal path in a 5G network.  Please see {{Mishra-2025}} for a more complete version 
of this diagram.

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

Similarly, the SCONE signal for 4G network is shown below.  Please see {{Mishra-2025}} for a more complete 
version of this diagram.

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
The sections below describe SCONE signal manageability.

## SCONE signal Hint from Client to the Network
In 3GPP networks (4G/5G), a User Equipment (UE) connects to the internet by establishing data sessions that traverse
various network elements. The key process involves allocating an IP address to the UE and routing its data traffic
through the mobile network's core to the external data networks including the internet.
As this connection to the Internet is established and once the client App on the UE starts communicating with the 
application content provider, a hint for SCONE usage will allow UPF to then look for a SCONE packet for this specific
user connection and avoid PGW/UPF any unnecessary CPU cycles for non-ABR video connections.
The section below provides a more detailed information on the UE and the mobile network for connecting to the 
external network.

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

## Measuring conformance of advised bit-rate
As the network element capable of advising bit-rate limit, the network element also would need capabilities to measure conformance
on the advised bit-rate. 

Issue 35 [https://github.com/ietf-wg-scone/scone/issues/35]
- Need to determine if the conformance is to be measured as an aggregate or on a per flow basis.

Presentation given at interim session 6 provides results based on experimentation that recommends a suitable size for time window to be 120 seconds. This value is compatible with existing VOD applications when ~2 mbps is the advised bitrate.
- [https://datatracker.ietf.org/meeting/interim-2025-scone-06/materials/slides-interim-2025-scone-06-sessa-time-window-duration-for-bitrate-measurement-00.pdf]

## Dynamic updates
In networks, for example - radio networks, the avaible capacity of the network can dynamically change for a persistance of time that, 
or there could be sudden increase of network users, these could result in change of throughput advice for a particular scone 
capable flow. These changes need to be dynamically and immidiately updated in the rate signal to avoid unnecesarry rate shaping 
or degradated QoE. This means the network elements need to be able to initiate the sending of the rate signal if there is not 
sufficient frequency of scone packets send for that particular flow. 

## Other open issues
- SCONE signaling MUST NOT require changes to how a CSP determines its video policy for a given flow. That is there is MUST not be any dependency between a CSP's video policy and the SCONE protocol.

- SCONE signal MUST be extensible to networks beyond 4G/5G network.

- discussion on how the applications/receivers can adapt to the rate signals.


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

