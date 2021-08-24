# SAP on Azure Network connectivity and Security 

**Authors: Abbas Ali Mir, Daniel Mauser and Madhuri Kaniganti**

**In this article**

[Abstract](#abstract)

[SAP on Azure VMs](#sap-on-azure-vms)

[Customer’s Networking and Security Requirements](#customer-sap-landscape-in-azure)

[Customer SAP Landscape in Azure](#customer-sap-landscape-in-azure)

[Customer Environment](#customer-environment)

[Proposed Architecture Solution](#proposed-architecture-solution)

[SAP on Azure – VMs and HLI (Hana Large Instance)](#sap-on-azure--vms-and-hli-hana-large-instance)

[Hana Large Instance (HLI) Introduction](#hana-large-instance-hli-introduction)

[ExpressRoute FastPath Introduction](#expressroute-fastpath-introduction)

[HLI Integration with VNETs – Enabling ER FastPath in Hub VNET](#hli-integration-with-vnets--enabling-er-fastpath-in-hub-vnet)

## Abstract

This document looks at some of the complex Networking connectivity and
Security requirements of the Customers, and designs SAP on Azure
Networking Solutions, that can meet those requirements.

## SAP on Azure VMs

### Customer’s Networking and Security Requirements
-----------------------------------------------

1. Any communication between SAP on Azure and Internet is routed
    through Firewall.
2. Any communication between SAP on Azure and On-Premises DC’s is
    routed through Firewall.
3. Any Communication within SAP Virtual Network (eg: App to DB) is
    controlled via Network Security Groups only.
4. Any Communication between SAP Virtual Networks (eg: SAP Transports,
    Data loads) in same region or cross-region is routed through
    Firewall.
5. Any SAP Hana System Replication (HSR) Communication (in-region or
    cross-region) is controlled via Network Security Groups only.
6. Any communication between SAP VNETs and Non-SAP VNETs, in same
    region or cross-region is routed thru Firewall NVA.
7. Firewall NVA, NSG’s (Network Security Groups) and User Defined
    Routes to be leveraged to accomplish the above.

### Customer SAP Landscape in Azure

Customer has a 3 environment SAP S4/HANA landscape - Development, QA
(Quality) & Production, that is deployed across two Azure Regions –
South Central US and East US Regions.

The QA Environment also serves as dual purpose DR Environment, with QA
Hana Database and DR Hana Database running on the same Hana VM.

**SAP Spoke VNETs** are provisioned as two types.

1. **SAP Mini-Hub VNET** (aka SAP Shared Services VNET) – one per Azure
    Region. 

    It contains following Subnets

    a.  SAP-Shared-Subnet (hosts SAP Shared Services such as Cloud Connector, SAP Router
        and WTS Servers ….)
    b.  ANF-Subnet
        (hosts ANF Volumes for sapcd, sapmnt, transports and interfaces
        directory)

2. **SAP Environment VNET** (eg: Dev VNET, QA VNET or Prod VNET)

    It contains following Subnets

    a.  Web Subnet - hosts Web Dispatches and any Azure load balancers
        front-ending it.
    b.  App Subnet – hosts App Servers, ASCS/ERS VMs & Azure load
        balancers front-ending it.
    c.  DB Subnet – host Hana Database VMs and any Azure load balancers
        front-ending it.
    d.  ANF Subnet (optional) – to hosts any local ANF Volumes in a
        specific SAP Env. (eg: Dev)

### Customer Environment 

#### Fig. 1-a – defines Azure Regions, Virtual Networks, On-Premise & Internet Connectivity

![](/media/fig1a.png)

1. It consists of SAP on Azure deployments in SCUS and EUS Azure
    Regions
2. It follows a Hub and Spoke VNET Model, with Hub VNET hosting shared
    Infrastructure services and Spoke VNETs hosting Application
    workloads.
3. An SAP Mini-Hub VNET is also created to provide for SAP Shared
    Services to all SAP Deployments in each Azure Region.
4. Customer has two Datacenters in Dallas and Ashburn respectively,
    that are connected thru Dallas and Ashburn ExpressRoute Circuits to
    Azure VNETs (via Hub VNET ExpressRoute Gateway) in each Region.

### Proposed Architecture Solution

#### Fig 1-b – defines required Regional and Global VNET peering's

![Regional and Global VNET peerings](/media/fig1b.png)

**Required Regional VNET peering’s**

1. Establish Regional VNET peering’s between all SAP VNETs and Hub VNET
    in each Azure Region
2. Establish Regional VNET peering’s between all SAP Env. Spoke VNET
    (dev, prod) and SAP Mini-Hub VNET in each Azure Region
3. It is assumed all Non-SAP VNETs that need connectivity to SAP VNETs,
    are peered with Hub VNET in each Azure Region

**Required Global VNET peering’s**

1. Global VNET Peering \#1

    This peering helps establish direct cross-region connectivity
    between SCUS and EUS

2. Global VNET Peering \#2

    This Peering enables cross-region Hana System Replication
    communication between Production Hana Database in SCUS and DR Hana
    Database in EUS

Mapping Customer requirements to Network connectivity & Security defined
in Fig 1-b

|#   |  Customer Requirements |   Solution / Communication Flow |
|---|---|---|
| 1  | Any communication between SAP on Azure (SAP VNETs) and Internet is routed thru Firewall  | SAP VNETs <> Hub VNET FW <> Internet  |
| 2  | Any communication between SAP on Azure (SAP VNETs) and On-Premises DC’s is routed thru Firewall.  | SAP VNETs <> Hub VNET FW <> ER Connection <> On-Premises DC’s  |
| 3  | Cross-Region Hana System Replication via Global VNET Peering  | SAP Prod VNET (SCUS) <> SAP QA VNET (EUS) |
| 4  | Any Communication between SAP VNETs (eg: SAP Transports, Data loads) in same region or cross-region is routed through Firewall.  | SAP VNET (SCUS) <> Hub VNET (SCUS) <> Hub VNET (EUS) <> SAP VNET (EUS)  |
| 5  | Non-SAP VNET (SCUS) to SAP VNET (SCUS) - (In-Region) | Non-SAP VNET (SCUS) <> Hub VNET (SCUS) <> SAP VNET (SCUS) |
| 6  | Non-SAP VNET (SCUS) to SAP VNET (EUS) - (Cross-Region) | Non-SAP VNET (SCUS) <> Hub VNET (SCUS) <> Hub VNET (EUS) <> SAP VNET (EUS)  |




#### Fig 1-c – defines the Virtual Network Subnets, including placement of Firewalls 

![](/media/fig1c.png)

Fig 1-c is the same Architecture as in Fig 1-b, and provides each
individual Virtual Network’s Subnet layout, to get a better
understanding of the overall Architecture Solution.

In above Fig 1-c, the Customer has chosen to use Palo-Alto Firewall
solution in Hub VNET.

They have gone with two Palo-Alto Firewall deployments

1.  Palo-Alto Firewall North South to control inbound / outbound traffic
    between Azure and Internet.

2.  Palo-Alto Firewall East West to control inbound / outbound traffic
    between Azure VNETS and from/to Azure VNETs to On-Premises.

**Note:**

Customers can choose to use Azure Firewall instead of Palo-Alto FW, and
control both North-South and East-West traffic using the same Azure
Firewall.

(Only one single Azure Firewall can be deployed in a given VNET)

If two Azure Firewalls are desired to control North-South vs East-West
traffic separately, then two Azure Firewalls will need to be deployed in
two separate VNETs.

#### Fig 1-d – defines placement of Network Security Groups and User Defined Routes.

![Defines placement of Network Security Groups and User Defined Routes](/media/fig1d.png)

Fig 1-c is the same Architecture as in Fig 1-b and provides more
guidance and understanding related to User Defined Routes (UDR’s) and
Network Security Groups (NSG’s) need to implement the proposed
Architecture Solution.

Defining each individual Route Tables (with UDR’s) and NSGs is beyond
the scope of this document, but the Authors of this document can be
reached for further guidance.

**Route Tables Description and Function**

|  # | Route Table (Color)  | Applies to VNET/Subnets  | Purpose / Function  |
|---|---|---|---|
| 1 | ![Blue](/media/blue.png) Blue | SAP Dev VNET (web subnet, app subnet, db subnet) / SAP QA, Prod VNETs (db subnet only) | Contains UDR’s that ensure all traffic originating (or Return Path) from SAP VNETs and destined to go to either Internet or On-Premises or to any other SAP or Non-SAP VNETs (in-region or cross-region), are being routed thru Hub VNET Firewall |
| 2 | ![Orange](/media/orange.png) Orange | SAP QA & Prod VNETs (web subnet, app subnet) | It’s performing same function as Blue Route Table, with additional more specific UDR’s added to account for HSR Replication. In essence, it ensures non-db traffic between SAP Prod VNET and QA VNETs is not traversing the direct Global VNET Peering connection setup solely for HSR communication., and instead going thru each Region Hub VNET Firewalls. |
| 3  | ![Green](/media/green.png) Green | Hub VNET (Firewall-Subnet-EW) | This ensures any East West Traffic destined to go to Cross-Region Azure SAP VNets is routed to other Region Hub VNET Firewall via Global VNET Peering path, instead of taking the ExpressRoute connectivity. It also ensures any East West Traffic destined to go to Same-Region Azure SAP VNets is routed properly. |
| 4  | ![Yellow](/media/yellow.png) Yellow  | Hub VNET (Gateway Subnet) | This ensures any communication originating (or Return Path) from On-Premises and destined to go to SAP VNETs is routed thru the Hub VNET Firewall. |
| 5  | ![Grey](/media/grey.png) Grey | Non-SAP VNETs Subnets | This ensures any communication originating (or Return Path) in non-SAP VNets and destined to go to SAP VNets is routed thru the Hub VNET Firewall. |

## SAP on Azure – VMs and HLI (Hana Large Instance)


### Hana Large Instance (HLI) Introduction

SAP Hana on Azure HLI is a unique solution, in that it offers the
possibility to run and deploy SAP HANA on bare-metal servers that are
dedicated to Customers.

The SAP HANA on Azure (Large Instances) solution builds on non-shared
host/server bare-metal hardware that is assigned to Customer. The server
hardware is embedded in larger stamps that contain compute/server,
networking, and storage infrastructure.

As HLI’s is a BareMetal offering, they are front-ended by HLI
ExpressRoute Circuit, that Customer’s VNETs can link to establish
connectivity between Azure VNET and HLI Hana Databases.

There are few options to establish integration between Azure Virtualized
(VNETS) and HLI

### ExpressRoute FastPath Introduction

ExpressRoute virtual network gateway is designed to exchange network
routes and route network traffic. FastPath is designed to improve the
data path performance between ExpressRoute Circuit (front-ending HLI or
On-Premises Networks) and your virtual networks (VNETs). When enabled,
FastPath sends network traffic directly to virtual machines in the
virtual network, bypassing the gateway.

This results in ExpressRoute Gateway only required to exchange routes
between Virtual Networks and ExpressRoute Circuits (front-ending HLI or
On-Premises Networks) and is not in the data-path between Azure VMs and
HLI/On-Premises., there by greatly improving network performance.

ExpressRoute FastPath is recommended for SAP App Servers to HLI Hana
database connectivity, as it helps bypass the ExpressRoute Virtual
network Gateway throughput limitations.

### HLI Integration with VNETs – Enabling ER FastPath in Hub VNET 

#### Fig 2-a – HLI & VNET Integration – ER FP in Hub VNET – (HLI’s in Single AZ/DC)

![HLI & VNET Integration – ER FP in Hub VNET](/media/fig2a.png)

The ExpressRoute FastPath support for peered VNETs is in public preview
and a key capability that is enabling the above Architecture.

Mapping HLI use-case scenarios to solution in Fig 2-a

| #  | HLI use-case connectivity Scenarios  | Architecture Solution |
|---|---|---|
| 1 | SAP App Servers need to connect to HLI over ExpressRoute FastPath | VNET Peering + ER Connection #2 |
| 2 | Connectivity between On-premises and SAP VNETs | VNET Peering + ER Connection #1 |
| 3 | Non-SAP VNETs to HLI connectivity | VNET Peering + ER Connection #2 |
| 4 | On-premises to HLI connectivity | ER Global Reach Connection |
| 5 | Internet (SCP…) to HLI connectivity | Internet + ER Connection #2 |

Note: Key Considerations

1. There is no UDR Support yet for the ExpressRoute FastPath
    Connection, which means any RouteTable (w/ UDR’s) applied on the
    GatewaySubnet, will not take effect on the ER FastPath Connection.

    (This can be a concern for some customers, that want to route
    certain traffic through their Firewall NVA, before it reaches
    HLI’s.)

2. On-premises will have direct connectivity to HLI via ER Global
    Reach, and can be controlled via Firewalls at On-premises location,
    or at ExpressRoute Edge location, or via OS level firewall on the
    HLI Nodes.

    Alternatively, Customers can choose not to implement ER Global
    Reach, and use SNAT/DNAT functionality at the Hub Firewall, or setup
    Proxy Server in Azure to establish connectivity between On-Premises
    and HLI.

3. Non-SAP VNETs will have direct connectivity to SAP VNETs, and can be
    controlled via NSGs and/or routed thru Firewall NVA via UDR’s

4. Non-SAP VNETs will have direct connectivity to HLI and can be
    controlled via routing traffic thru Firewall NVA via UDR’s (more
    specific routes).

#### Fig 2-b – HLI & VNET Integration – ER FP in Hub VNET – (HLI’s across two Av. Zones)

![HLI & VNET Integration – ER FP in Hub VNET](/media/fig2b.png)

If HLI’s are setup across two Availability Zones, then please do note
that they will be in two separate VLAN’s and will be front-ended by two
separate HLI ExpressRoute Circuits.

Hence two separate ExpressRoute Connections will need to be established,
from Hub VNET to the two HLI ExpressRoute Circuits.

Likewise, two separate ExpressRoute Global Reach connections will need
to be established from the On-premises ExpressRoute Circuit to the two
HLI ExpressRoute Circuits.

Mapping HLI use-case scenarios to solution in Fig 2-b

| # | HLI use-case connectivity Scenarios  | Architecture Solution |
|---|---|---|
| 1 | SAP App Servers need to connect to HLI over ExpressRoute FastPath  | VNET Peering + ER Connection #2 or #3 |
| 2 | Connectivity between On-premises and SAP VNETs | VNET Peering + ER Connection #1 |
| 3 | Non-SAP VNETs to HLI connectivity | VNET Peering + ER Connection #2 or #3 |
| 4 | On-premises to HLI connectivity | ER Global Reach Connection #1 and #2 |
| 5 | Internet (SCP…) to HLI connectivity | Internet + ER Connection #2 or #3 |

### HLI Integration with VNETs – Enabling ER FastPath in SAP VNET 

#### Fig 3-a – HLI & VNET Integration – ER FP in SAP VNET, Non-SAP to HLI via Hub VNET

![HLI & VNET Integration – ER FP in SAP VNET, Non-SAP to HLI via Hub VNET](/media/fig3a.png)

The above Solution will be useful for those Customers that are not
comfortable with enabling ER Fast Path for Hub VNET ExpressRoute
Gateway, as it will open up HLI to all VNETs that are peered with Hub
VNET

The above solution will deploy a separate ExpressRoute Gateway in one of
the SAP Spoke VNETs (eg: SAP MiniHub VNET), and setup an ExpressRoute
FastPath connection (ER Connection \#2 in Fig 3-a) between SAP VNET and
HLI

It will also setup ExpressRoute Connection without FastPath (ER
Connection \#4 in Fig 3-a) between Hub VNET and HLI, to enable any
non-sap VNETs that need to communicate with HLI’s.

More specific UDR’s (User Defined Routes) will need to be setup, if
Customers want to route non-SAP VNETs to SAP VNets or to HLI through
Firewall NVA’s, as otherwise direct connectivity will be possible.

Mapping HLI use-case scenarios to solution in Fig 3-a

| #  | HLI Connectivity Scenarios | Architecture Solution |
|---|---|---|
| 1 | SAP App Servers need to connect to HLI over ExpressRoute FastPath | VNET Peering + ER Connection # 2 |
| 2 | Connectivity between On-premises and SAP VNETs | VNET Peering + ER Connection # 3 |
| 3 | Connectivity between On-premises and SAP VNETs | VNET Peering + ER Connection # 4 |
| 4 | On-premises to HLI connectivity  | ER Global Reach Connection |
| 5 | Internet (SCP…) to HLI connectivity | Internet + ER Connection # 4 |

### Fig 3-b – HLI & VNET Integration – ER FP in SAP VNET, Non-SAP to HLI via Hub & SAP VNETs FW 

![HLI & VNET Integration – ER FP in SAP VNET, Non-SAP to HLI via Hub & SAP VNETs FW ](/media/fig3b.png)

This solution can be considered if any of the above options are not
feasible or acceptable to Customers.

This solution can create considerable networking complexity, as
SNAT/DNAT configurations will be needed on Hub VNET and SAP VNET
Firewalls, in addition to setting up many UDR’s.

Mapping HLI use-case scenarios to solution in Fig 3-b

| # | HLI Connectivity Scenarios | Architecture Solution |
|---|---|---|
| 1 | SAP App Servers need to connect to HLI over ExpressRoute FastPath | VNET Peering + ER Connection #2 |
| 2 | Connectivity between On-premises and SAP VNETs | VNET Peering + ER Connection #3 |
| 3 | Non-SAP VNETs to HLI connectivity | VNET Peering + Hub VNET Firewall + SAP VNET Firewall + ER Connection #2 |
| 4 | On-premises to HLI connectivity | ER Global Reach Connection  |
| 5 | Internet (SCP…) to HLI connectivity | Internet + Hub VNET Firewall + SAP VNET Firewall + ER Connection #2 |