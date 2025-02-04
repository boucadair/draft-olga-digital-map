---
title: "Modeling the Digital Map based on RFC 8345: Sharing Experience and Perspectives"
abbrev: Digital Map Modelling
docname: draft-havel-nmop-digital-map-00
category: info

stand_alone: true
submissiontype: IETF  
area: "Operations and Management"
wg: NMOP

consensus: true
v: 3

author:
  -
    fullname: Olga Havel
    org: Huawei
    email: olga.havel@huawei.com
  -
    fullname: Benoit Claise
    org: Huawei
    email: benoit.claise@huawei.com
  -
    fullname: Oscar González de Dios
    org: Telefonica
    email: oscar.gonzalezdedios@telefonica.com
  -
    fullname: Ahmed Elhassany
    org: Swisscom
    email: Ahmed.Elhassany@swisscom.com
  -
    fullname: Thomas Graf
    org: Swisscom
    email: thomas.graf@swisscom.com
  -
    fullname: Mohamed Boucadair
    org: Orange
    email: mohamed.boucadair@orange.com

contributor:
  -
    fullname: Nigel Davis
    org: Ciena
    email: ndavis@ciena.com
  
normative:

informative:
[1]: https://yangcatalog.org/yang-search/impact_analysis/ietf-network-topology@2018-02-26 "Catalog"

--- abstract

This document shares experience in modelling Digital Map based on the IETF RFC 8345 topology YANG modules and 
some of its augmentations. First, the concept of Digital Map is defined and its connection to the Digital Twin is 
explained. Next to Digital Map requirements and use cases, the document identifies a set of open issues encountered 
during the modelling phases, the missing features in RFC 8345, and some perspectives on how to address them.

Discussion Venues

   This note is to be removed before publishing as an RFC.

   Source for this draft and an issue tracker can be found at
   https://github.com/OlgaHuawei.

--- middle

# Introduction

{{!RFC8345}} specifies a topology YANG model with many YANG augmentations for different technologies and service types.  
The modelling approach based upon {{!RFC8345}} provides a standard IETF based API.

At the time of writing (2024), there are at least 59 YANG modules that are augmenting {{!RFC8345}}; 
58 IETF-authored modules and 1 BBF-authored module. 14 of these modules have maturity level of 'ratified', 
17 of them have maturity level of 'adopted', 31 modules have maturity level of 'latest-approved', 
and 27 of these modules have maturity level of 'initial'.  
The up-to-date information can be found in the YANG Catalog available at [Catalog][1].

From this set of IETF RFCs and IETF I-Ds (at different level of maturity), we designed a Digital Map 
Proof of concept (PoC), with the following objectives and functionalities:

* Can the central RFC 8345 YANG module be a good basis to model a Digital Map?
+ How the different topology related IETF YANG modules fit (or not) together?
+ Modelling of Digital Map entities, relationships, and rules how to build aggregated entities and relationships. Does 
the base model support key requirements that emerge for a specific layer?
+ Modelling multiple underlay/overlay layers from Layer 2 to Layer 3 to customer service layer. To what extent it is 
easy to augment the base model to support new technologies?
- Can the base model be augmented for any new layer and technologies?

This I-D documents an experience in the modeling aspects of the Digital Map, based on a PoC implementation, 
basically documenting the effort and the open issues encountered so far.  During the PoC, we also identified a set of 
requirements and verified the PoC approach by demoing it iteratively.

Practically, we developed a PoC with a real lab, based on multi-vendor devices, with {{!RFC8345}} as the base YANG module.  
The PoC successfully modelled the following technologies:

-  Layer 2 network topology (used {{!RFC8944}})
-  Layer 3 network topology (used {{!RFC8346}})
-  OSPF routing (aligned with [I-D.ogondio-opsawg-ospf-topology])
-  IS-IS routing (aligned with [I-D.ogondio-opsawg-isis-topology])
-  BGP routing
-  MPLS LDP
-  MPLS TE tunnels
-  SRv6 tunnels
-  L3VPN service
 
## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", 
"NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in 
BCP 14 {{!RFC2119}} {{!RFC8174}} when, and only when, they appear in all capitals, as shown here.

* Digital Twin -
      Virtual instance of a physical system (twin) that is continually
      updated with the latter's performance, maintenance and health
      status data throughout the physical system's life cycle (as
      defined in Section 2.2 of
      {{?I-D.irtf-nmrg-network-digital-twin-arch}}

* Topology -
      Network topology defines how physical or logical nodes, links and
      interfaces are related and arranges.  Service topology defines how
      service components (e.g., VPN instances, customer interfaces, and
      customer links) between customer sites are interrelated and
      arranged.  There are at least 8 types of topologies: point to
      point, bus, ring, star, tree, mesh, hybrid and daisy chain.
      Topologies may be unidirectional or bidirectional (bus, some
      rings).

* Topology layer -
      Defines a layer in the multilayer topology.  A multilayer topology
      models relationships between different layers of connectivity,
      where each layer represents a connectivity aspect of the network
      and service that needs to be configured, controlled and monitored.
      The layer can also represent what needs to be managed by a
      specific user, for example IGP layer can be of interest to the
      user troubleshooting the routing, while the optical layer may be
      of interest to the user managing the optical network.  Some
      topology layers may relate closely to OSI layers, like L1 topology
      for physical topology, L2 for link topology and L3 for IPv4 and
      IPv6 topologies.  Some topology layers represent the control
      aspects of L3, like OSPF, IS-IS and BGP.  The top layer represents
      the service view of the connectivity, that can differ for
      different types of services and for different providers/solutions.

* Digital Map -
      Basis for the Digital Twin that provides a virtual instance of the
      topological information of the network.  It provides the core
      multi-layer topology model and data for the Digital Twin and
      connects them to the other Digital Twin models and data.

* Digital Map modelling -
      The set of principles, guidelines, and conventions to model the
      Digital Map using the IETF [RFC8345] approach.  They cover the
      network types (layers and sublayers), entity types, entity roles
      (network, node, termination point or link), entity properties and
      relationship types between entities.

* Digital Map model -
      Defines the core topological entities, their role in the network,
      core properties and relationships both inside each layer and
      between the layers.  It is the basic topological model that is
      linked to other functional parts of the Digital Twin and connects
      them all: configuration, maintenance, assurance (KPIs, status,
      health, symptoms), traffic engineering, different behaviors,
      simulation, emulation, mathematical abstractions, AI algorithms,
      etc

* Digital Map data -
      Consists of instances of network and service topologies at
      different layers.  This includes instances of networks, nodes,
      links and termination points, topological relationships between
      nodes, links and termination points inside a network,
      relationships between instances belonging to different networks,
      links to functional data for the instances, including
      configuration, health, symptoms.  The data can be historical,
      real-time or future data for what-if scenarios.

# Digital Map and Digital Twin Relationship

## Digital Twin
The network Digital Twin (referred to simply as Digital Twin) concepts and a reference architecture are proposed in 
the "Digital Twin Network: Concepts and Reference Architecture" NMRG I-D {{?I-D.irtf-nmrg-network-digital-twin-arch}}.

This document defines the core elements of Digital Twin - Data, Models, Interfaces, and Mapping.  
The Digital Twin, constructed based on the four core technology elements, is intended to analyze, diagnose, emulate, 
and control the physical network in its whole lifecycle with the help of optimization algorithms, management methods, 
and expert knowledge.

Also, this document states that a Digital Twin can be seen as an indispensable part of the overall network system 
and provides a general architecture involving the whole lifecycle of physical networks in the future, serving the 
application of innovative network technologies (e.g., network planning, construction, maintenance and optimization, 
improving the automation and intelligence level of the network).

## Digital Map
Digital Map provides the core multi-layer topology model and data for the Digital Twin and connects them to the 
other Digital Twin models and data.

The Digital Map modelling defines the core topological entities (network, node, link and interface), their role in 
the network, core properties, and relationships both inside each layer and between the layers.

The Digital Map model is a basic topological model that is linked to other functional parts of the Digital Twin and 
connects them all: configuration, maintenance, assurance (KPIs, status, health, symptoms), Traffic Engineering (TE), 
different behaviors and actions, simulation, emulation, mathematical abstractions, AI algorithms, etc.

The Digital Map data consists of virtual instances of network and service topologies at different layers.  
The Digital Map provides the access to this data via standard APIs for both read and write operations 
(write operations for offline simulations), with query capabilities and links to other YANG modules 
(e.g., Service Assurance for Intent-based Networking (SAIN) {{?RFC9417}}, Service Attachement Point (SAP) {{?RFC9408}}, 
Inventory {{?I-D.draft-ietf-ivy-network-inventory-yang}}, and Assets {{?I-D.palmero-opsawg-dmlmo}}) and non-YANG models.

## Digital Map as a Prerequisite for Digital Twin
One of the important requirements for the digitalization and Digital Twin is to ease correlating all models and 
data to topological entities at different layers in the layered twin network.  The Digital Map aims to provide a 
virtual instance of the topological information of the network, based on this Digital Map Model. 
Building a Digital Map is prerequisite towards the Digital Twin.

The Digital Map model/data will provide this missing correlation between the topology models/data and all other 
models/data: KPIs, alarms, incidents, inventory (with UUIDs), configuration, traffic engineering, planning, 
simulation ("what if"), emulations, actions, and behaviors.

Some of these models/data provide a device view, some provide a network or subnetwork view, while others focus 
more on the customer service perspective.  All these views are needed for both inner and outer closed-loops. It is 
debatable what is part of the Digital Map itself versus what are pointers from the Digital Map to some 
other sources of information.  As an example, the Digital Map should not specifically include all information 
about the device inventory (product name, vendor, product series, embedded software, and
hardware/software versions, as specified in Network Inventory (IVY) WG, 
https://datatracker.ietf.org/doc/charter-ietf-ivy/ ): a pointer from the Digital Map to another inventory system 
might be sufficient. 

Similarly, the Digital Map should not specifically contain incidents, configuration, 
data plane monitoring, or even assurance information, simply to name a few.

The following are some Digital Twin use cases that require Digital Map:

* Generic inventory queries
+ Service placement feasibility checks
+ Service-> subservice -> resource
+ Resource -> subservice -> service
+ Intent/service assurance
+ Service E2E and per-link KPIs on the Digital Map (delay, jitter, and loss)
+ Capacity planning
+ Network design
+ Simulation
- Closed loop

Overall, the Digital Map is needed to break down data islands and have a Digital Twin for emulation and closed loop, 
which is a catalyst for autonomous networking.

# The IETF Network Topology Approaches

## IETF Network Topology 
{{!RFC8345}} provides a simple generic topological model.  It defines the abstract /generic /base model for network 
and service topologies. It provides the mechanism to model networks and services as layered topologies with common 
relationships at the same layer and underlay/overlay relationships between the layers.

{{!RFC8345}} consists of two modules: 'ietf-network' and 'ietf-network-topology'.  The 'ietf-network' module defines 
networks and nodes, while 'ietf-network-topology' module adds definitions for links and termination points.

The relationships inside the layer are containment/aggregation (a network has nodes, a network has links, a node has 
termination points), source (a link has one source termination point) and destination (a link has one destination 
termination point).

The relationships between the layers are modelled via supporting relationship

* network A is supported by network B - this may model overlay/underlay relationship

* nodes, links and termination points of network A are supported by nodes, links and termination points of network B.  
Overlay and underlay nodes, links and termination points must match underlay and overlay networks supporting it

## IETF Network Topology TE 
{{?RFC8795}} defines a YANG model for representing, retrieving and manipulating TE topologies.  This is a more complex 
model which augments {{!RFC8345}} with traffic engineering topology information as follows:

* TE topology, node, link and termination point are defined using the core RFC8345 concepts

    - TE topology augments network with topology identifier (provider, client and topology id), as well as other 'te' 
  information

    - TE node augments node with 'te-node-id' and other 'te' information
    
    - TE link augments link with te information
    
    - TE termination point augments termination point with 'te-tp-id' and te information

* Tunnel, tunnel termination point, local link connectivity, node connectivity matrix, and some supporting and underlay 
relationships are defined outside of the core RFC 8345 entities and relationships

# Digital Map Requirements

// We discussed if requirements should be in a separate document. We would leave them in this document for now, 
later we can create a separate draft

The following are the core requirements for the Digital Map (note that some of them are supported by default by {{!RFC8345}}:

1.   Basic model with network, node, link, interface entity types

2.   Layered Digital Map, from physical network (ideally optical, layer 2, layer 3) up to customer service and intent

3.   Open and programmable Digital Map. This includes "Read" to understand the view of the network.  It also includes 
"Write", not for the ability to directly change the Digital Map Data (ex: changing the network or service parameters), 
but for offline simulations, also known as what-if scenarios.  Doing what-if analysis requires the ability to take 
snapshots and to switch easily between them.  Because we have to distinguish between a change on the Digital Map 
for future simulation and a change that reflects the current reality of the network.

4.   Standard based Digital Map Models and APIs, for multi-vendor support.  Digital Map must provide the standard APIs 
that provide for Read / Write and queries.  This API must also provide the capability to retrieve the links to external data / models.

5.   Digital Map models and APIs must be common over different network domains (campus, core, data center, etc.)

6.   Digital Map must provide semantics for layered network topologies and for linking external models/data

7.   Digital Map must provide inter-layer and between layer relationships

8.   Digital Map must be extensible with meta data

9.   Digital Map must be pluggable a) We must connect to other YANG modules for inventory, configuration, assurance, etc 
b) Not everything can be in YANG, we need to connect digital map YANG model with other modelling mechanisms, 
both southbound, northbound and internally

10.  Digital Map must be optimized for graph traversal for paths. This means that only providing link nodes and 
source and sink relationships to termination-points may not be sufficient, we may need to have the direct 
relationship between the termination points or nodes

## Why RFC8345 is a Good Approach for Digital Map Modelling

The main reason for selecting RFC8345 for modelling is its simplicity and that is supports majority of the core 
requirements. The requirements [1-7] are automatically fulfilled with RFC8345 and extensions:

* Basic model with network, node, link and interface entity types

* Layered topology

* Open and programmable

* Standard, multi-vendor

* Multi-domain

* Semantics for layered topology

* Inter-layer and between-layer relationships

The requirements [6-7] for semantics for layered topology and relationships are partially fulfilled, there may be 
need for some additional semantics.  Other core requirements [8-10] are not supported by RFC8345:

*  Extensible with meta data

*  Pluggable to other YANG modules and non-YANG data

*  Optimized for graph traversal

In some cases, for [9] for pluggable to other YANG modules, the links are already done by augmenting 'ietf-network' 
and 'ietf-network- topology'. For others, we need to add some mechanisms to link the models and data.

## Design Requirements

The following are design requirements for modelling the Digital Map:

1.  Digital Map should contain only topological information.  Digital Map should not contain all data required for 
all the management and use cases, but should have pointers to other functional Digital Twin data and models.

2.  Digital Map entities should contain only properties used to identify topological entities at different layers, 
identify their roles and topological relationships between them

3.  Digital Map should contain only topological relationships inside each layer or between the layers (underlay/overlay)

4.  ACLs and Route Policies will be outside of the Digital Map, they would be linked to Digital Map

5.  Dynamic paths may either be outside of the Digital Map, part of traffic engineering data/models

6.  Provide capability for conditional retrieval of parts of Digital Map

7.  Must support geo-spatial, temporal, and historical data.  The temporal and historical can be supported external 
to the Digital Map.

The following are the architectural requirements for the Digital Map:

1.  Scale, performance, integration

2.  Initial discovery and dynamic (change only) synch

# Digital Map Modelling Experience

## What is not in the basic model


Based on our PoC and experience, the following are the {{!RFC8345}} extensions needed for Digital Map modelling and APIs:

*  Bidirectional links

*  Multi-point connectivity

*  Links between domains/networks

*  A network decomposition into sub-networks

*  Nodes, links, and termination points belonging to different networks

*  Supporting relationships between different types of entities

*  More network topology related semantic is needed

### Bidirectional Links

The RFC8345 defines all links as unidirectional, it does not support bidirectional links.  It was done intentionally 
to keep the model as simple as possible.  The RFC suggests to model the bidirectional connections as pairs of 
unidirectional links.

Nevertheless, while simplifying the model itself, we are making data and APIs more complex for the cases where we 
have bidirectional links.  And we are increasing the amount of instances / data transferred via API, stored in the 
database, or managed/monitored.

One of the core characteristics of any network topology is the link directionality.  While data flows are 
unidirectional, the bidirectional links are also common in networking.  Examples are Ethernet cables, 
bidirectional SONET rings, socket connection to the server, etc.  We also encounter requirements for simplified 
service layer topology, where we want to model link as bidirectional to be supported by unidirectional links at the 
lower layer.

{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} further elaborates on the need for bidirectional links in the 
network topologies and in the Digital Map.  It also proposes how RFC8345 can be augmented to support missing components.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 approach and implement it via multiple unidirectional links 

* Don't change RFC8345, leave to different augmentations to solve the problem their own way

* Augment RFC8345 via basic approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
fully backward compatible, appears simple and sufficient

* Augment RFC8345 via more sophisticated approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
more complex but improves the integrity of the model, same instance structures produced

* Start working on RFC8345 bis 

We suggest to start the work on RFC8345 bis to provide the backward compatible way to support bidirectional 
links in the core topology model defined in ietf-network-topology. The starting point can be the basic approach from
{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} that adds the following:

* direction-of-link with value BIDIRECTIONAL 
* direction-of-point with value BIDIRECTIONAL
* list of termination points that could be used for bidirectional links as an alternative to having source and 
destination for unidirectional (although we can also implement it via the existing source and destination when 
direction BIDIRECTIONAL)

### Multipoint Connectivity

The RFC8345 defines all links as point to point and unidirectional, it does not support multi-point links 
(hub and spoke, full mesh, complex).  It was done intentionally to keep the model as simple as possible. 
The RFC suggests to model the multi-point networks via pseudo nodes.

Same as with unidirectionality, while simplifying the model itself, we are making data and APIs more complex for 
multi point topologies and we are increasing the amount of data transferred via API, stored in the database or 
managed/monitored.

One of the core characteristics of any network topology is its type and link cardinality.  Any topology model 
should be able to model any topology type in a simple and explicit way, including point to multipoint, bus, ring, 
star, tree, mesh, hybrid and daisy chain.  Any topology model should also be able to model any link cardinality in 
a simple and explicit way, including point to point, point to multipoint, multipoint to multipoint or hybrid.

By forcing the implementation of all topology types and all options for cardinality via unidirectional links and 
pseudo nodes, we are significantly increasing the complexity of APIs and data, but also lacking full topology 
semantics in the model.  The model cannot be fully used to validate if topology instances are valid or not.

Note that the point to multipoint network type is also required in some cases at the OSPF layer.

{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} further elaborates on the need for multipoint connectivity in 
network topologies and in the Digital Map, in general.  It also proposes how RFC8345 can be augmented to 
support these missing components.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 and implement via virtual nodes

* Don't change RFC8345, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 via basic approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
fully backward compatible, appears simple and sufficient

* Augment RFC8345 via more sophisticated approach as suggested in {{?I-D.davis-opsawg-some-refinements-to-rfc8345}}, 
more complex but improves the integrity of the model, same instance structures produced 

* Start working on RFC8345 bis that provides backward compatible enhancement 
(similar to {{?I-D.davis-opsawg-some-refinements-to-rfc8345}} approach without augmentations)

We suggest to start to work on RFC8345 bis to provide the backward compatible way to support multipoint connectivity 
in the core topology model defined in ietf-network-topology. The starting point can be the basic approach from 
{{?I-D.davis-opsawg-some-refinements-to-rfc8345}} that adds the following:
* list of termination points for multipoint links as an alternative to having source and destination for point to point 
links via the existing source and destination
* role-of-point
* type-of-link

###  Links Between Networks

   The RFC8345 defines all links as belonging to one network instance
   and having the source and destination as node and termination point
   only, not allowing to link to termination point of another network.
   This does not allow for links between networks in the case of multi-
   domains or partitioning.  The only way would be to model each domain
   as node and have links between them.

   In our IS-IS PoC (following [I-D.ogondio-opsawg-isis-topology]), We
   modelled IS-IS areas as networks and we needed to extend the
   capability to have links between different areas.  We added network
   reference as well to the source / destination of the link.  The
   {{?RFC8795}} also augments links with external-domain info for the case of
   links that connect different domains.

   The IS-IS topology [I-D.ogondio-opsawg-isis-topology] models
   Autonomous System (AS) or IS-IS domain as a network, and IS-IS areas
   as attributes of IS-IS nodes.  The RFC8345 extension can be used to
   model IS-IS areas as networks and IS-IS links between L1-2 nodes as
   links between two different areas.  This is not problem for OSPF,
   although the OSPF nodes can belong to multiple areas, the links can
   belong to only one area.

   The following are some benefits of lifting the RFC 8345 limitations
   that all links must belong to only one network instance:

   *  IS-IS processes would be grouped in an area via the standard IETF
      RFC8345 network->node relationship.

   *  Applications and algorithms will consume topologies based on the
      generic entities and relationships, they will not need to
      understand the meaning of specific IS-IS attributes.

   *  The approach would be aligned with the IS-IS topology model and
      the IS-IS network view in manuals and documentation.

   *  Provide capability to link different IGP domains and links between
      them.

   *  Link between multiple networks/sub-networks is the common concept
      in network topology.

The following are the candidate approaches of how we can address this limitation:

* Use the current RFC8345 and implement domains via properties in augmentations

* Don't change RFC8345, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach for multi-domain to RFC8345 digital map)

* Start working on RFC8345 bis 

We suggest to start to work on RFC8345 bis to provide the backward compatible way to support links between networks 
in the core topology model defined in ietf-network-topology. The starting point can be to evaluate the approach 
from {{?RFC8795}} that adds the external domain reference to the link via the external network, node and tp reference.



###   Networks part of other networks

   RFC8345 does not model networks being part of other networks,
   therefore cannot model subnetworks and network partitioning.  We
   encountered this problem with modelling IS-IS and OSPF domains and
   areas.  The initial goal was to model AS/domain with multiple areas
   so that the Digital Map model contains information about how the AS
   is first split into different IGP domains and how each IGP domain is
   split into different areas.  This is a common problem for both IS-IS
   and OSPF.

The following are the candidate approaches of how we can address this limitation:

   *  Leave it as it is in RFC8345, don't model AS and IS-IS/OSPF domain directly, they would be
      modelled via the underlying IP network and IS-IS/OSPF enabled
      routers.  This could be achieved via supporting relationship to L3
      network and L3 nodes
   * Leave it as it is in RFC8345, leave to different augmentations to solve the problem their own way 
   * Leave it as it is in RFC8345, model AS, IGP domains and areas as networks and use supporting
      relationship to model the network topology design, with only areas
      having nodes.  This does not describe the correct nature of the
      relationship, semantic is missing.
   * Augment RFC8345 by adding some simple solution to support additional partitioning relationship between
      networks.
   * Start working on RFC8345 bis 

We suggest to start to work on RFC8345 bis to provide the backward compatible way to support partitioning of networks 
in the core network model defined in ietf-network. The solution needs to add a part-of relation between different 
networks, where one network (e.g. OSPF Domain) can contain multiple networks (e.g. OSPF areas)


###   Nodes, tps and links in multiple networks

   RFC8345 does not allow nodes and TPs to belong to multiple areas
   without splitting them into separate entities with separate keys.  In
   OSPF case we have nodes that can belong to different areas, but
   interfaces can only belong to one area.  In the case of IS-IS,
   although all tutorial are stating that nodes can belong to one area
   only, the ietf, openconfig and vendor yang models and CLI allow IS-IS
   processes with all its interfaces to belong to multiple areas.  

The following are the candidate approaches of how we can address this limitation:

   *  Use the current RFC8345 approach, this is not the problem for read
      but may be an issue for write for simulation as we would expect
      that the interface lists in different nodes and networks be the
      same without being able to validate.

   *  Augment RFC8345 to optionally allow nodes to be
      defined outside of network tree and enable reference as an
      alternative to the containment in the tree.  This may be a bigger
      change to the network topology approach as it would have bigger
      impact on the topology tree. Nevertheless, it can be an optional approach so would be backward compatible for 
      those augmentations that do not want to use it

   * Start working on RFC8345 bis 

We suggest to work on RFC8345 bis to provide the simple backward compatible way to support both the current RFC8345 approach 
of creating multiple instances and the approach of sharing the instances. The solution needs further analysis as it has 
bigger impact on the topology tree than other enhancements.

###  Missing Supporting Relationships

   RFC8345 defines supporting relationships only between the same type
   of entities.  Networks can only be supported by networks, nodes can
   only be supported by nodes, termination points can only be supported
   by terminations points and links can only be supported by links.

   During the PoC, we encountered the need to have TP supported by node
   and node supported by Network.  The RFC8795 also adds additional
   underlay relationship between node and topology and link and
   topology, but via a new underlay topology and not via the core
   supporting relationship.

The following are the candidate approaches to address this limitation:

* Use the current RFC8345 approach, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach to RFC8345)

* Start working on RFC8345 bis that provides backward compatible enhancement (e.g. via {{?RFC8795}} basic approach)

We suggest to work on RFC8345 bis to provide the backward compatible way to add the missing supporting relationships:
tp->supporting->node, node->supporting->network. 

###  Missing Topology Semantics

   We already mentioned that some semantic is missing from the RFC8345
   topology model, like bidirectional and multi-point.  The following is
   also missing from the model:

   *  Relationship Properties.  The supporting relationship could have
      additional attributes that give more information about the
      supporting relationship.  That way we could use it for
      aggregation, underlay with primary/backup, load balancing, hop,
      sequence, etc.

   *  Termination point roles.  We are missing semantics for the common
      topology roles: primary, backup, hub, spoke

   *  Layers / Sublayers.  We need further analysis to determine in
      network types are sufficient to support all scenarios for layers/
      sublayers.  The network types are more related to technologies so
      in the case that the same technology is used at different layers,
      we may need some additional information in the model.  For further
      study.

   *  Tunnels and paths.  We modelled tunnels ad paths via {{!RFC8345}} but
      we lost some semantics that is in RFC8795.

   *  Supporting or underlay.  We modelled all underlay relationships
      via supporting, further analysis is needed to determine pros and
      cons of this approach versus RFC8795 approach of using underlay
      topology.

The following are the candidate approaches to address this limitation:

* Use the current RFC8345 approach, leave to different augmentations to solve the problem their own way 
(see how it is done in {{?RFC8795}})

* Augment RFC8345 by adding some simple solution (e.g. move {{?RFC8795}} approach to RFC8345)

* RFC8345 bis 

We suggest to work on RFC8345 bis to provide the backward compatible way to add the minimum semantics that the 
community agrees is required for the core topology. We need to further investigate the {{?RFC8795}} approach and 
evaluate if some parts could be moved to RFC8345. 

##  Open Issues (for Further Analysis or Resolved)

   The following are the open issues that need further analysis or have been resolved:

   *  Do we need separation of L2 and L3 topologies?

      During the PoC we encountered different solutions with separate
      set of requirements.  In one solution, the L2 and L3 topology were
      the same with separate set of attributes, while in another
      solution we had difference in L2 and L3 topology (e.g.  Links
      Aggregation, Switches and Routers).

      The RFC8944 defines L2 topology and RFC8346 defines the L3
      topology.  They allow to have either one or two instances of this
      topology.

      The decision if we need separate network instances for L2 and L3
      topologies may be based on specific network topology and
      provider's preferences.

      Resolved: the RFC8345 is flexible and it can support both the same network instance with L2 + L3 augmentations 
      or separate network instances with supporting relationship between. The operator should decide what option is 
      needed for their solution.

   *  Do we need sublayers as well?  Layers versus sublayers versus
      layered instances?  

      Resolved: Layers/sublayers could be implemented via multiple network types. The new data nodes for layer 
are present only when the network type for the layer is present. The new data nodes for the sublayer are 
present when the network types for both layer and sublayer are present. 
The solution could also enable either single or multiple instances, like in the previous point.

   *  Same technology at service versus underlay?  BGP per VPN vs common
      BGP shared between multiple VPNs.  Different layers, same model,
      relationship define the layer. 
   
      Further analysis is needed.

   * There are potential circular dependencies in layering. For example routing can be underlay for tunnels, 
but tunnel interface can also be in the routing table. 
   * 
     Further analysis is needed.

##  How to Augment for Generic Digital Map Extensions

   Generic Digital Map extensions are the RFC8345 extensions required
   for all technologies and layers in the Digital Map. We already discussed some options for individual limitations in
   the previous sections, here is the summary:

    1. Use the current RFC8345 approach, leave to different augmentations to solve the problem their own way 

    2. Augment RFC8345 network, node, link and termination point for any
    changes needed from a new digital map module
~~~~
   module: dm-network-topology
           augment /nw:networks/nw:network:
                   .... additions
           augment /nw:networks/nw:network/nw:node:
                   .... additions
           augment /nw:networks/nw:network/nt:link:
                   .... additions
           augment /nw:networks/nw:network/nw:node/nt:termination-point:
                   .... additions


   // This can be a separate draft with describing pros and cons of
   // different approaches and yang model proposal.  Add reference to
   // this draft when submitted
~~~~

    3. Make backward compatible updates to RFC8345, work on RFC8345 bis

The following are some important guidelines mentioned in the RFC8345 that should be taken into account when suggesting 
the approach:

* "The data models allow applications to operate on an inventory or topology of any network at a generic level, 
where the specifics of particular inventory/topology types are not required.
At the same time, where data specific to a network type comes into play and the data model is augmented, 
the instantiated data still adheres to the same structure and is represented in a consistent fashion.  
This also facilitates the representation of network hierarchies and dependencies between different network components 
and network types"

* “It is possible for links at one level of a hierarchy to map to multiple links at another level of the hierarchy.  
For example, a VPN topology might model VPN tunnels as links.  Where a VPN tunnel maps to a path that is composed of 
a chain of several links, the link will contain a list of those supporting links.  Likewise, it is possible for a 
link at one level of a hierarchy to aggregate a bundle of links at another level of the hierarchy."

Our recommendation on how to extend RFC8345 for Digital Map is to stay in spirit of RFC8345 and augment with 
non-topological info only. Reuse network, node, link, tp for all topological entities, reuse supporting for 
layering and add new properties/attributes and references to other modules
Therefore, we suggest to work on RFC8345 bis to provide the backward compatible way to address all 
identified limitations. 

The alternative of having the core topology augmentations in either te modules or technology specific modules is not 
generic enough and is not in the spirit of having the core topology model to model topology in the consistent manner 
between different technologies and te and non-te topologies.

## How to Augment for New Technologies/Layers

   There are already drafts that support augmentation for specific
   technologies.  These drafts augment network, node, termination point
   and link, but also add different topological entities inside
   augmentations.  For example, we have examples like this:

~~~~
   module: new-module
           augment /nw:networks/nw:network/nw:network-types:
         +--rw new-topology!
                   augment /nw:networks/nw:network:
                                   ....
                   augment /nw:networks/nw:network/nw:node:
                           .... adding list of tps of other type
                                    (e.g. tunnel TPs in te draft)
                           ... adding new supporting relationship
                                    supporting-tunnel-termination-point
                                    (te draft)
                   augment /nw:networks/nw:network/nt:link:
                                   .... adding tunnels (te draft)
                   augment /nw:networks/nw:network/nw:node/
                                                 nt:termination-point:
                                   ....
~~~~
   There is need to agree some guidelines for augmenting IETF network
   topology, so that additional topological information is not added in
   the custom way.  There is also need to categorize the current augmentations and the impact of RFC 8345 bis 
   based on what has been added for different technologies:

* new properties/attributes (e.g. ietf-l2-topology, ietf-l3-unicast-topology, ietf-isis-topology)
* new events (e.g. ietf-l2-topology)
* new topological entities (e.g. ietf-te-topology, ietf-dc-fabric-topology)
* new topological relations (e.g. ietf-te-topology)
* type reuse (e.g. ietf-dots-telemetry, ietf-dc-fabric-types, ietf-dc-fabric-topology)

   // This can be a separate draft.  Guidelines with examples?  Add reference to this draft when submitted


## How to connect to the external world

Digital Map must be pluggable:
* We must connect to other YANG modules for inventory, configuration, assurance, etc 
* Not everything can be in YANG, we need to connect digital map YANG model with other modelling mechanisms, 
both southbound, northbound and internally

### How to connect to other YANG modules

There are already some modules that connect network topology to other YANG modules. 
We will investigate different approaches and propose the best practices. 
The following are some existing approaches proposed in IETF:
 
* How to connect network topology to interface {{?I-D.ietf-tp-interface-reference-topology}}
* How to connect network topology to hardware inventory {{?I-D.draft-ietf-ccamp-network-inventory-yang}}
* How to connect network topology to ivy inventory {{?I-D.ietf-network-inventory-topology}}
* How to connect network topology to performance monitoring {{?RFC9375}}

### How to connect YANG models with other modelling mechanisms
There is need to connect YANG network topology to models and data outside of YANG, for example BMP, IPFIX, logs, etc.

## Digital Map APIs

   This will include hierarchical APIs for cross-domain figure, IETF
   YANG Based API (read and write, change subscription and notify) and
   Query API

##  Digital Map Knowledge

   The following knowledge was needed to build Digital Map:

   *  Abstract IETF Entities and Relationships as in {{!RFC8345}}

   *  {{!RFC8345}} Augmentations for a)Layers/sublayers b)Entities
      (services and subservices), c)Properties

   *  Rules for aggregating entities

   *  Rules for instantiating relationships (inter-layer and intra-
      layer)

   *  Mapping from devices and controllers

   What can be achieved with existing RFC8345 YANG:

   *  Entities (base class IETF Network, IETF Node, IETF Link, IETF TP)

   *  Properties

   *  Relationships

   Next steps

   *  How to support temporal

   *  How to support spacial

   *  How to support historical

#  Related IETF Activities

##  Network Topology

   Interestingly, we could not find any network topology definition in
   IETF RFCs (not even in {{!RFC8345}}) or drafts.  However, it's mentioned
   in multiple documents.  As an example, in Overview and Principles of
   Internet Traffic Engineering {{?I-D.ietf-teas-rfc3272bis}}, which
   mentioned:


   To conduct performance studies and to support planning of existing
   and future networks, a routing analysis may be performed to determine
   the paths the routing protocols will choose for various traffic
   demands, and to ascertain the utilization of network resources as
   traffic is routed through the network.  Routing analysis captures the
   selection of paths through the network, the assignment of traffic
   across multiple feasible routes, and the multiplexing of IP traffic
   over traffic trunks (if such constructs exist) and over the
   underlying network infrastructure.  A model of network topology is
   necessary to perform routing analysis.  A network topology model may
   be extracted from:

   *  Network architecture documents

   *  Network designs

   *  Information contained in router configuration files

   *  Routing databases such as the link state database of an interior
      gateway protocol (IGP)

   *  Routing tables

   *  Automated tools that discover and collate network topology
      information.

   Topology information may also be derived from servers that monitor
   network state, and from servers that perform provisioning functions.

##  Core Digital Map Components

   The following standards are core for the Digital Map.

   *  IETF network model and network topology model {{!RFC8345}}

   *  A YANG grouping for geographic location {{?RFC9179}}

   *  IETF modules that augment {{!RFC8345}} for different technologies:

   *  - A YANG data model for Traffic Engineering (TE) Topologies
      [RFC8795]

   *  - A YANG data model for Layer 2 network topologies [RFC8944]

   *  - A YANG data model for OSFP topology
      {{?I-D.ogondio-opsawg-ospf-topology}}

   *  - A YANG data model for IS-IS topology
      {{?I-D.ogondio-opsawg-isis-topology}}

##  Additional Digital Map Components

The Digital Map may need to link to the following models, some are already augmenting [RFC8345], we need further 
investigation for each of these items:

*  Service Access Point (SAP) {{?RFC9408}}, augments 'ietf-network' data model {{!RFC8345}} by adding the SAP.

*  SAIN {{?RFC9417}} and {{?RFC9418}}

*  Network Inventory Model {{?I-D.draft-ietf-ivy-network-inventory-yang}} is the latest inventory model proposed by ivy, 
focusing on physical and virtual inventory. Logical inventory is currently outside of the scope. 
It does not augment RFC8345 like the two drafts that it evolved from {{?I-D.ietf-ccamp-network-inventory-yang}} 
and {{?I-D.wzwb-opsawg-network-inventory-management}}. The {{?I-D.draft-wzwb-ivy-network-inventory-topology}} 
correlates the network inventory with the general topology via RFC8345 augmentations that reference inventory.

*  Data Model for Lifecycle Management and Operations [I-D.palmero-opsawg-dmlmo]

*  KPIs: delay, jitter, loss

*  Attachment Circuits (ACs) {{?I-D.boro-opsawg-ntw-attachment-circuit}} and {{?I-D.boro-opsawg-teas-attachment-circuit}}

*  Configuration: L2SM {{?RFC8466}}, L3SM {{?RFC8299}}, L2NM {{?RFC9291}}, L3NM {{?RFC9182}}

*  Incident Management for Network Services {{?I-D.feng-opsawg-incident-management}}

##  Network Inventory (IVY) Proposed Working

   The charter of the Network Inventory (IVY) IETF Working Group (WG)
   can be found at https://datatracker.ietf.org/doc/charter-ietf-ivy/.
   Understanding how the two efforts complement each other is important.

   The IVY effort focuses on the network inventory (as the charter says,
   "including a variety of information such as product name, vendor,
   product series, embedded software, and hardware/software versions").

   The network inventory is probably the first use cases for the digital
   map.  Therefore, it is important to have a consistent view of what a
   network node is.  While a Digital Map must include a pointer to the
   hardware and software inventory information, we don't consider that
   all the inventory information is actually part of the Digital Map.
   It must also be noted that the set of use cases for the Digital Map
   is wider than just the network inventory.

   The IVY charter also says that, "The Working Group will consider
   existing IETF work, including RFC 8348 and RFC 8345.".  In this
   document, RFC 8345 is considered as the base YANG module, therefore,
   there is clearly common ground with the work of the ivy working
   group.  This document goes beyond RFC 8345 to evaluate whether that
   RFC, along with all the augmented YANG modules, would be a good fit
   for all the Digital Map requirements.

   Additionally, the IVY charter says, "The WG will also identify a set
   of requirements and guidelines to ensure consistency across models
   related to inventory."  While the IVY requirements and guidelines
   focus on inventory, this document looks at the full set of
   requirements, guidelines, and building blocks for all the Digital Map
   use cases.  The inventory use case does not cover that full set, and
   other building blocks will be required: for example, to support
   point-to-multipoint connectivity

   Thus, this Digital Map modelling effort is complementary to the
   inventory work in IVY.  It has a broader outlook covering all Digital
   Map use case requirements, and will correlate with the existing IETF
   models, e.g., topology, service attachment points (SAP), etc.

# Conclusions

Digital Map Modelling and Data are basis for the Digital Twin. During our PoC we have proven that Digital Map can be 
modelled using {{!RFC8345}}.  Nevertheless, we proposed some extensions/augmentations to {{!RFC8345}} to support 
Digital Maps. There must be some constraints in regards to how to augment the core Digital Map model for different 
Layers and Technologies in order to support the approach recommended in RFC8345 and implemented in our PoC. 
All entities must augment IETF node, IETF network, IETF link or IETF Termination Point and augmentation can only 
include new properties, events, references. 

We suggest to start the work on RFC8345 bis to provide the backward compatible way to support the following basic 
topological features:

* bidirectional links
* multipoint connectivity
* cross-domain links via links between networks
* multi-domain via network partitioning
* shared topological entities between different domains
* additional supporting relations
* additional semantics required for core topologies

# Security Considerations

The Digital Map exposes a lot of key details about the network that may be confidential and might be valuable to an 
attacker.

Future revisions will add more details of what information needs to be protected and how it may be protected

# IANA Considerations

This document has no actions for IANA.


--- back

# Acknowledgments
{:numbered="false"}

Thanks to xx for their reviews and comments.
