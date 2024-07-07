---
stand_alone: true
category: info
submissionType: IETF
ipr: trust200902
lang: en

title: Network Attestation for Secure Routing (NASR) Architecture
abbrev: NASR-Architecture
docname: draft-liu-nasr-architecture-latest

area: SEC
workgroup: NASR
type: BOF

kw:
 - NASR
 - Secure Routing

author:
 -
    fullname: "Chunchi Liu"
    organization: Huawei
    email: "liuchunchi@huawei.com"
 -
    fullname: "Meiling Chen"
    organization: China Mobile
    email: "chenmeiling@chinamobile.com"
 -
    fullname: "Michael Richardson"
    organization: Sandelman Software Works
    email: "mcr@sandelman.ca"
 -
    fullname: "Diego Lopez"
    organization: Telefonica
    email: "diego.r.lopez@telefonica.com"


normative:

informative:
  RFC9344:
  I-D.liu-nasr-requirements-01: NASRREQ
  I-D.richardson-nasr-terminology-01: NASRTERM
  I-D.ietf-rats-ar4si-06: AR4SI
  I-D.ietf-rats-corim-04: CORIM


--- abstract

This document provides an architecture overview of NASR entities, interactive procedures and messages.

--- middle

# Introduction

Endpoints typically perceives no information of the properties of the paths over which their traffic is carried, especially when the properties are security-related. Therefore, data security (confidentiality, integrity and authenticity) has been insofar primarily protected by traffic signing and encryption mechanisms. Endpoint cannot choose devices with specific properties to bear transmission.

However, clients with high security and privacy requirements are not anymore satisfied with traffic signing and encryption mechanisms only; they now request information of the trustworthiness or security properties of the network paths over which the traffic is carried, preferably to choose the desired properties. For example, some clients may require their data to traverse through trusted devices and trusted links only, in order to avoid data being exposed to insecure devices, causing leakage.

Remote Attestation Procedures (RATS) working group developed procedures to establish a level of confidence in the trustworthiness of a device or a system. RATS provide 1. objective, appraisable evidence of a device’s trust or security properties, and 2. verifiable integrity proof to the evidence (Attestation Result). Devices with integrity proof are expected to function correctly and deterministically, as anticipated.

Following the same RATS philosophy and building on top of it, Network Attestation for Secure Routing （NASR) aims at a solution specifically designed for the routing use case. NASR aims to provide 1. objective, appraisable evidence of a routing path’s trust or security properties, 2. verifiable integrity proof in the path-level, and 3. verifiable proof that certain packets/flows traveled such paths.

Altogether, the NASR goal is to 1. Allow clients to choose desired security attributes of his received network service, 2. Achieve dependable forwarding by routing on top of only devices that satisfies certain trust requirements, and 3. prove to the clients that certain packets or flows traversed a network path that has certain trust or security properties.

This document introduces the architecture, entities, interactive procedures, and messages sent between the entities, so to achieve the NASR objective.


# Use Cases {#usecases}

Please refer to the use cases identified in {{-NASRREQ}}

# Terminology {#term}

Please refer to the terminologies identified in {{-NASRTERM}}. Terminology from RATS Architecture document {{RFC9344}} also applies.

NASR will leverage RATS implementations and specifications, including but not limited to {{-AR4SI}}{{-CORIM}}.

# Architectural Overview

## Single client - single operator (An Oversimplification)
                                                                                      
    +--------------------+                                                          
    |                    |                                                          
    |    Relying Party   |                                                          
    |                    |                                                          
    |                    |                                                          
    +----+---------^-----+                                                          
  Path   |         |                                                                
  Request|         | Report                                                         
         |         |                                                                
    +----v---------+-----+                                    +--------------------+
    |                    |     Path Attestation Result (PAR)  |                    |
    |    Orchestrator    |                                    |      Verifier      |
    |                    <------------------------------------+                    |
    |                    |                                    |                    |
    +----+---------------+                                    +----------^---------+
         |                                                               |          
Path     |                                                               |  Path    
Evidence |                                                               |  Evidence
(PE)     |                                                               |  (PE)    
    +----v---------------+       +--------------------+       +----------+---------+
    |                    |       |                    |       |                    |
    |      Attester      +------->     Attester...    +------->      Attester      |
    |                    |       |                    |       |                    |
    +--------------------+       +--------------------+       +--------------------+
                      Update PE with                Update PE with                  
                        AR/RE/PoT                     AR/RE/PoT                     
                        
Figure 1. NASR Architecture-- Oversimplified

Fig. 1 is an oversimplification to ease understanding of the concept. In a single client - single operator scenario, a client (Relying Party) sends a Path Request containing his security or trustworthiness requirements of a leased line. The Orchestrator, run by the operator, would choose qualifying devices (Attesters) and send out an empty Path Evidence inquiry. The Attesters update the Path Evidence with its own Raw Evidence or Attestation Results one-by-one. The Verifier verifies the filled Path Evidence, produce a Path Attestation Result (PAR), and sends it back to the Relying Party. The Relying Party now have confidence over the trustworthiness of received network. After the end-to-end service is delivered, during service, Proof-of-Transits are also created by each Attester, being sent in-band accumulatively or out-of-band, to detect unexpected routing deviation.

This process is repeated periodically to continuously assure compliance.


## Multi Client - Multi Operator

                           ┌────────────────────────────────────────────────────────────────┐                          
                           │                                                                │                          
                           │       ┌────────────┐                 ┌────────────┐            │                          
                           │       │ Verifier   │                 │ Verifier   │    ...     │                          
                           │       │ Vendor A   │                 │ Vendor B   │            │                          
 ┌───────────────────┐     │       └────▲─────┬─┘                 └───▲────┬───┘   Vendors  │    ┌───────────────────┐ 
 │                   │     │            │     │                       │    │                │    │                   │ 
 │                   │     └────────────┼─────┼───────────────────────┼────┼────────────────┘    │                   │ 
 │    Client  A      │                  │     │                       │    │                     │    Client  B      │ 
 │                   │  Path            │     │                       │    │         Path        │                   │ 
 │  ┌──────────────┐ │  Request         │     │                       │    │         Attestation │   ┌────────────┐  │ 
 │  │  Relying     ├─┼──────────┐ ┌─────┼─────┼───────┐        ┌──────┼────┼───────┐ Result (PAR)│   │  Relying   │  │ 
 │  │  Party       │ │          │ │     │     │       │        │      │    │       │     ┌───────┼───┤  Party     │  │ 
 │  └───┬──────────┘◄├────────┐ │ │     │     │       │  Intra │      │    │       │     │       │   └────────────┘  │ 
 │      │            │        │ │ │  RE │     │AR     │  ISP   │   RE │    │ AR    │     │       │           ▲       │ 
 │      │            │ Answer │ └─► ┌───┴─────▼─────┐ │  API   │  ┌───┴────▼────┐  │     │       │           │       │ 
 │      │Path        │ Report │   │ │ Orchestrator  ├─┼────────┼─►│ Orchestrator│◄─┼─────┘       │           │       │ 
 │      │Evidence    │        └───┤ └───▲─────┬─────┘ │        │  └───▲────┬────┘  │             │           │       │ 
 │      │            │            │     │     │       │        │      │    │       │             │           │       │ 
 │      │            │            │  RE │     │AR     │        │   RE │    │ AR    │             │           │       │ 
 │   ┌──▼────────┐   │            │ ┌───┴─────▼─────┐ │        │  ┌───┴────▼─────┐ │             │   ┌───────┴───┐   │ 
 │   │ Attester  │   │            │ │  Attester     │ │        │  │  Attester    │ │             │   │ Attester  │   │ 
 │   │           ├───┼────────────┤►│  Vendor A     ├─┼────────┼─►│  Vendor B    ├─┼─────────────┼──►│           │   │ 
 │   └───────────┘   │  Update PE │ └───────────────┘ │        │  └──────────────┘ │  Update PE  │   └───────────┘   │ 
 │                   │    with    │                   │        │                   │    with     │                   │ 
 │                   │  AR/RE/PoT │  Operator 1       │        │   Operator 2      │  AR/RE/PoT  │                   │ 
 └───────────────────┘            └───────────────────┘        └───────────────────┘             └───────────────────┘ 
                                                                                                                       
Figure 2. NASR Architecture

In a more generalized scenario, due to geographic distances, a single operator cannot span across a long distance to deliver an end-to-end service-- multiple operators collaborate to deliver it. The Customer A would send the Path Request to the Operator nearest to him (Operator 1). Operator 1 pass down the Path Request to the collaborating operators, through an intra-ISP API. Operators of different domains choose qualifying devices to altogether orchestrate the path.

Relying Party (customer) then sends the Path Evidence inquiry to check and attest to this path. Following the same procedure, the client of other side would return the Path Attestation Result back, through the operators. The Operator 1 would send back a comprehensive answer/report to the Client.

Also, the operators may have heterogeneous network devices from different vendors. Since vendors provide Verifier software/hardware and Reference Values, Verifiers can be deployed either outside the operators (Fig 2) or inside of the operators (Fig 3).
                                                                                                                        
                           ┌────────────────────────────────────────────────────────────────┐                           
                           │                                                                │                           
                           │       ┌────────────┐                 ┌────────────┐            │                           
                           │       │ Verifier   │                 │ Verifier   │            │                           
                           │       │ Owner      │                 │ Owner      │    ...     │                           
                           │       │ Vendor A   │                 │ Vendor B   │            │                           
                           │       └───────┬────┘                 └──────┬─────┘   Vendors  │                           
                           │               │                             │                  │                           
                           └───────────────┼─────────────────────────────┼──────────────────┘                           
                                           │  Verifier software/hardware │                                              
                                           │  Reference Value            │                                              
 ┌───────────────────┐            ┌────────┼──────────┐        ┌─────────┼─────────┐             ┌───────────────────┐  
 │                   │            │        │          │        │         │         │             │                   │  
 │                   │            │  ┌─────▼──────┐   │        │   ┌─────▼──────┐  │             │                   │  
 │    Client  A      │            │  │ Verifier   │   │        │   │ Verifier   │  │             │    Client  B      │  
 │                   │  Path      │  │ from       │   │        │   │ from       │  │ Path        │                   │  
 │  ┌──────────────┐ │  Request   │  │ Vendor A   │   │        │   │ Vendor B   │  │ Attestation │   ┌────────────┐  │  
 │  │  Relying     ├─┼──────────┐ │  └──┬─────┬───┘   │        │   └──┬────┬────┘  │ Result (PAR)│   │  Relying   │  │  
 │  │  Party       │ │          │ │     │     │       │        │      │    │       │     ┌───────┼───┤  Party     │  │  
 │  └───┬──────────┘◄├────────┐ │ │     │     │       │  Intra │      │    │       │     │       │   └────────────┘  │  
 │      │            │        │ │ │  RE │     │AR     │  ISP   │   RE │    │ AR    │     │       │           ▲       │  
 │      │            │        │ └─► ┌───┴─────▼─────┐ │  API   │  ┌───┴────▼────┐  │     │       │           │       │  
 │      │Path        │ Report │   │ │ Orchestrator  ├─┼────────┼─►│ Orchestrator│◄─┼─────┘       │           │       │  
 │      │Evidence    │        └───┤ └───▲─────┬─────┘ │        │  └───▲────┬────┘  │             │           │       │  
 │      │            │            │     │     │       │        │      │    │       │             │           │       │  
 │      │            │            │  RE │     │AR     │        │   RE │    │ AR    │             │           │       │  
 │   ┌──▼────────┐   │            │ ┌───┴─────▼─────┐ │        │  ┌───┴────▼─────┐ │             │   ┌───────┴───┐   │  
 │   │ Attester  │   │            │ │  Attester     │ │        │  │  Attester    │ │             │   │ Attester  │   │  
 │   │           ├───┼────────────┼─┤  Vendor A     ├─┼────────┼─►│  Vendor B    ├─┼─────────────┼──►│           │   │  
 │   └───────────┘   │  Update PE │ └───────────────┘ │        │  └──────────────┘ │  Update PE  │   └───────────┘   │  
 │                   │    with    │                   │        │                   │    with     │                   │  
 │                   │  AR/RE/PoT │  Operator 1       │        │   Operator 2      │  AR/RE/PoT  │                   │  
 └───────────────────┘            └───────────────────┘        └───────────────────┘             └───────────────────┘  
                                                                                                  

Figure 3. Verifier deployed in operators

# Roles {#roles}

The existing roles from RATS Architecture document {{RFC9344}} applies.

Attester: The definition in {{RFC9344}} applies. Additionally, it can be performed by either a physical device or a virtual function. The Attester can update Path Evidence with his Attestation Result/Raw Evidence/Proof of Transit.

    Produces: (updated) Path Evidence

Relying Party: The definition in {{RFC9344}} applies. Additionally, it creates Path Request to the Orchestrator, and receive Reports from Orchestrator as an auditable result, comparing the actually received network service versus the requested PR attributes.

    Produces: Path Request

    Consumes: Report

In the case where an Attester is deployed in the customer premises, the Relying Party could also start the unfilled Path Evidence inquiry at his side.

New role(s) are defined below.

Orchestrator: A role performed by an entity (typically a controller or a special device) that performs two functions: path orchestration and path attestation. The input and output of different functions are different.

- Path Orchestration: The Orchestrator receives a Path Request from the Relying Party. After path computation/orchestration, he creates configurations to be distributed to the Attesters/devices.

    Consumes: Path Request

    Produces: Configurations

- Path Attestation: The Orchestrator receives a Path Request from the Relying Party, send unfilled Path Evidence (PE) inquiry to Attesters, collects Path Attestation Result (PAR) from the Verifier, and send PAR back to the Relying Party.

    Consumes: Path Request, Path Attestation Result

    Produces: (unfilled) Path Evidence

Verifier: A role performed by an entity that appraises the validity of filled Path Evidence about a set of Attesters and produces Path Attestation Results to be used by an Orchestrator.

    Consumes: (filled) Path Evidence

    Produces: Path Attestation Results

# Conceptual Messages {#messages}

The existing artifacts from RATS Architecture document {{RFC9344}} applies. New conceptual message(s) are defined here.

Path Request: A set of claims, describing the properties of a network path that a Relying Party requested.

    Consumed By: Orchestrator

    Produced By: Relying Party

Path Attestation Result: The output generated by a Verifier, including information about a set of Attesters, where the Verifier vouches for the validity of the results.

    Consumed By: Relying Party

    Produced By: Verifier

Path Evidence: The output generated by the Orchestrator and a set of Attesters, to be appraised by a Verifier. Path Evidence may include each Attester's raw Evidence {{RFC9344}}, Attestation Results, Proof-of-Transit, or other proof suggesting correctness of functioning of each Attester.

    Consumed By: Verifier

    Created By: Orchestrator

    Updated By: Attester(s)

Report: An auditable result that compares the actually received network service versus the requested PR attributes.

    Created By: Orchestrator

    Consumed By: Relying Party

# Orchestration

The orchestration process collects client's path requests and output configurations. The Orchestrator/Controller then distribute them to the attester/device using NETCONF/YANG or other control plane protocols. In the first case, a new YANG module needs to be defined.

               +------------------------+
               |                        |
Path Request   |Orchestrator/Controller |
-------------->|                        |
               +----------+-------------+
                          |
                          |Path and Security Configuration
                          |(YANG/NETCONF)
                          |
                    +-----v------------+
                    | Attester/Device  |
                    +------------------+

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

We sincerely thank contribution from NASR mailing list.
