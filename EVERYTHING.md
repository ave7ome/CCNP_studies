GITHUB formatting:  

    4 spaces and text afterwards makes it as own plate of text

[somename](someurl) - makes letters in "[]" appear as a link to URL in "()"  

3 underscores ("_") makes a big horizontal line (a divider)

"&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;" can be used to indent

Definition - description  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Explanation of the above in more details etc.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;More xpl  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;and even more  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Here i specify on "even more"  

____________________________________________


All terms and points are covered here in a way I understood it.

***( - get to know the definition more

[SDSM series](https://linkmeup.gitbook.io/sdsm/0.-planirovanie)

[Anki cards](https://ankiweb.net/shared/info/1981245053)

[White papers](https://www.reddit.com/r/ccnp/comments/kpeefz/cisco_white_papers_i_used/)


       


#                    Network Design
_Tier 2 network_ - Access and Distribution layer

Tier 3 network - Access, Distribution and Core layer

Collapsed topology - can be collapsed in following ways:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Collapsed Core+Distribution (2 modules) - CD-Access topology  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Collapsed Distribution+Access (2 modules) - Core-DA topology  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Collapsed Core+Distribution+Access (1 module) - CDA topology  

L2 Access layer:
    STP have to be considered - links will be blocked (solution - PVST+ or assigning certain VLANs to certain links with no loops)

L3 Access layer:
    Costs more money but STP and Loops are not an issue anymore. Also can't easily stretch the VLAN


Modern techs to get rid of STP:
    Chassis switches (prevents loops, and no single point of failure)
    Stacking (prevents loops, and no single point of failure)


Access layer - devices, L2 switches
    Port security - MACs, 802.1x;
    QoS;
    VLAN tagging.
Distribution layer - SVIs, East-West traffic, sometimes L3 interfaces
    SVIs;
    FHRPs;
    ACLs;
    Routing protocols.
Core layer - high-speed, high-convergence, high-throughput, access to Internet
    RPs;
    fast l3 switching

DC new cool topology:
    Spine-Leaf

#                    High availability (HA)
FHRP (VRRP, HSRP, GLBP):
VRRP - virtual gateway protocol. 2+ gateways. Can't load balance. Common on all devices.
    "Master" and "Backup" routers
    VRRP exchanges hello messages over the timers (defaults are 3sec for HELLO, 10sec for HOLD)
    Can use one of physical router IPs as virtual. In that case, the router with that IP is an Active by default.
    VRRP authentication is recommended (MD5)
    VRRP priority (1-254). 100 is default. The bigger the better.
    After Master router switchover, "preemt" is required to be switched back over to previous active one. Preemt feature is *enabled by default
    VRRPv2 and VRRPv3 (used for IPv6) exist.
    VMAC = 0000.5e00.01** (256 different ones)
    MulticastIP = 224.0.0.18
    VRRP on Nexus with VPC (in DC) acts same as HSRP

HSRP - Cisco proprietary VRRP implementation. 2+ gateways
    "Active" and "Standby" routers. "Listen" as well
    HSRP exchanges hello messages over the timers (defaults are 3sec for HELLO, 10sec for HOLD)
    HSRP authentication is recommended (MD5)
    HSRP priority (0-255). 100 is default. The bigger the better.
    Preempt feature is *disabled by default
    Each HSRP process requires an HSRP group
    HSRPv1: 
        256 HSRP groups; 
        VMAC = 0000.0c07.ac** (256 different ones);
        MulticastIP = 224.0.0.2
    HSRPv2:
        4096 HSRP groups;
        VMAC = 0000.0c9f.f*** (4096 different ones);
        MulticastIP = 224.0.0.102;
        Millisecond timers are configurable.
    HSRP on Nexus with VPC acts differently. Usually all the traffic goes to Active switch and in VPC scenario it'll have to go through standby sometimes. That's why in given scenario, Standby switch handles the packets just like active. But it's still standby if checked hsrp status on switches. But some requests (like ARP) will still go to Active and be responded from it.
    It's a good idea to LB using HSRP by making one router primary for one VLAN and other router primary for other VLAN.
    
After switches get assigned a virtual IP, also they receive a virtual MAC-address. After first ARP from client, primary switch responces to packet. Whenever it's down and there appears a new primary switch which takes over the virtual IP (virtual MAC as well), it sends a Gratuitous ARP to say "now i have that MAC"
GARP - An ARP response when there were to ARP request.
FHRP object tracking helps to change the VIP holder to another device in case of some event. Following can be tracked:
    Route availability;
    Line protocol of an interface;
    Prefix reachability;
    IP SLA.

GLBP - Load balancing mechanism (Cisco proprietary). Similar to HSRP, but supports no less than 4 gateways
    "Active Virtual Gateway" (AVG) and "Active Virtual Forwarder" (AVF) states of router
    Traffic from connected endpoints is being sent to one of AVFs (AVG is also an AVF) and by that, gets load-balanced
    AVG is choosed based on priority
    Can be load-balanced based on:
        Round-robin
        Weighted
        Host-dependent (based on which host is sending the traffic)
    Timers and authentication are present as well
    VMAC = 0007.B40x.xxyy (x.xx is a Group ID, limited by 1024 different ones, since first "x" is 4 bits; yy is an AVF identifier which is "01-04", can be checkes by show glbp commands)
    MulticastIP = 224.0.0.102 (like HSRPv2)
    
SSO - mechanism that allows second CPU to take over the traffic and processing from other CPU in case of it's failure
    Can be used if device has two Route processes (control planes) and one is going down
NSF - Nonstop Forwarding. Allows data plane to send out packets to a neighbor even if it goes down control plane - wise.
    NSF feature must be configured on both devices to be operable
    Routes are being marked as Stale and will be exchanged once again when CP (control plane) goes up
VSS - stacking mechanism that allows two physical switches to be represented as a single virtual switch. Devices can be located in remote locations


#                    WLAN (wireless LAN) design

Useful on Wireless (+recommended learning in the end of publication)
https://packetpushers.net/askjjx-help-office-wi-fi-is-so-bad-an-intern-is-following-the-ceo-around-with-an-ap/ 

Comparison of basic Access devices (switches) to Autonomous access points:
    SVI (switched virtual interface)   ---    BVI (bridged virtual interface)
    VLAN                               ---    SSID

Autonomous design (old)
    Main issue for Autonomous deployment is low scalability. For Core deployments connected through L3, VLAN cannot be stretched reliably for clients to be able to move from one physical place to another and still be connected to same VLAN.
    Also it's difficult to configure a lot of device manually.
    Two devices below used to help configure a lot of devices. But there still were issues with pushing the config.
    WLSM (Wireless LAN Services Module) - EOS 
    WLSE (Wireless LAN Solution Engine) - EOS

Lightweight design (modern)
    WLC (Wireless LAN Controller)
    Access points are brainless and are used in conjunction with the controller (WLC). Once they're connected to network, they receive an IP through DHCP, retrieve Sotware image and configuration.
    Clients are connected to APs. APs establish a data tunnel (CAPwAP - Control and Provisioning of Wireless Access Points) so clients are connected directly to WLC from L2 perspective. Both Data plane and Control plane traffic are going through the tunnel.

WLC deployment design models
    Centralized (APs are in campus, WLC is in DC, tunnel is used to exchange traffic between APs and WLC).
        Pros: L2 Roam - VLANs are located on all APs, users can safely move from anywhere to anywhere and still be on their same subnet.
        Cons: Logically clients are located in the DC which is not quite secure. Also licensing should be considered. WLC controllers can manage only certain number of devices
    Distributed (Same as Centralized, but there are different WLCs for every building).
        Pros: Still L2 Roam within the building
        Cons: Licensing (1 WLC for certain number of APs) and number of controllers is increased.
    Branch (Flex connect) - used for small buildings. APs and WLC are deployed as in Centralised model but tunnel established between them only transfers Control Plane traffic. Data plane remains at AP and is sent out directly to Switch.
        Requirements: <100ms latency to WLC, <50 APs
        Pros: Efficient traffic route (no need to send traffic to DC first). Simpler WLC topology.
        Cons: No more L2 roam, but since it's a branch design it might not be needed.
    Cloud (Meraki) - Wireless controllers are located in the cloud. APs are installed on site and they transfer control plane traffic to the cloud. DP is going to upstream swtiches.
        Requirements: Cisco Meraki APs
        Pros: No licensing issues (increased with the number of used APs), no need to manage controller Hardware. Data traffic of wireless clients is passed to local switch instead of being tunneled to controller
        Cons: No L2 roam in case the Distribution switches (or access switches) are L3 interconnected 

Wireless AP deployment and placement
    COVERAGE
        Coverage is about wireless signal being available at every part of the office/campus/building with (preferrably) the same good strength. It can overlap between different APs and it actually is a standard. 35% overlap (for voice and video networks) and 20% overlap (for no video/voice) is desirable.
        When checking the signal of certain AP, end of coverage should be considered at the point where strength gets changed from -67 dBm to -68 dBm.
        Wireless CHANNELS should also be considered. There are 2 common ones: 2.4 GHz and 5 GHz.
        APs can have different antennas. Ones that give a signal to 360 degrees and ones that direct the signal in some angle (10-60 degrees)
        Cisco models can be with internal antennas (AP-I) or external (AP-E). External models antennas can be changed.
    CAPACITY
        Capacity is about number of clients per AP. 
        When preparing Wireless design, it should be considered how many people will use the AP to understand how many APs you need 

Cisco Prime Infrastructure - network management software platform. It provides a centralized management solution. The software allows administrators to monitor, configure, and troubleshoot network devices from a single interface, and can also be used for network performance analysis and reporting.

Real Time Location Services (RTLS) - service that helps to understand where does the certain device is physically located in the building (it have to be using WIFI at the moment)
    Can be used with RFID tags to check the location of anything (wheelchair in hospital for example)
    Also if it leaves certain area, there can be an alert
    
    Data is collected by WLC and APs but a separate Software is required to gather and analyze the data
        CPI
        CMX
        DNA Center (SDA)
        DNA Spaces (SDA)

    Accuracy. 
        Basic: We can always tell which AP is device connected to.
        Advanced: Triangulation. Assume there are 3 APs. A wireless device is in a reach of every AP (ex. -42 dBm, -59 dBm, -63 dBm). Every dBm level can be drawn as a circle around each AP. When 3 circles are drawn, there's just a single point where all of them interconnect. Like that location can be determined. But only in a free space without any walls.
                  ***(have to clarify, too tired to write it down correctly) RF Fingerprinting. Technic of profiling the wireless coverage. Just walk around with wireless device around the block for APs to understand how it all look like logically.


#                    Cloud computing solutions
Identification of Cloud.
According to NIST (National Institute of Standards and Technology), some system can be identified as a Cloud computing solution (example is Public Cloud like AWS, Microsoft Azure, Google Cloud Platform) if:
    1. It supports self-service - Customer can log in to account and just spin up a new VM or service.
    2. It has a broad network access - It is highly available from as much regions of the world as possible.
    3. It supports resource pooling - Anyones VM in cloud can be running with few other VMs from other companies. Multitenancy is used for companies to be secured from each other.
    4.* It supports Rapid Elasticity - Resources for clients VMs can be smartly (by itself) expanded whenever the demand is higher than usual and shrinked back to normal when demand falls off.
    5.* Services are Metered - All the usage of resources is being counted just like electricity in houses. Customers pay for all their usage monthly (usually monthly but there can be exceptions).

Cloud models
    No cloud:
        In no cloud environment (basic DataCenter), your systems engineers are responsible for spinning up a VM, installing an OS on it, and an Application usually.
        It can look like below:
        _______    _______    _______
        |  ___ |   |  ___ |   |  __  |
        | |APP||   | |WEB||   | |DB| |
        |  ‾‾‾ |   |  ‾‾‾ |   |  ‾‾  |
        |  __  |   |  __  |   |  __  |
        | |OS| |   | |OS| |   | |OS| |
        |  ‾‾  |   |  ‾‾  |   |  ‾‾  |
        |___VM1|   |___VM2|   |___VM3|
       
        3 VMs, one with OS and Application, another with OS and WEB-server, third one is with OS and DB. All deployed and supported by your company
        (VM, OS, APP, WEB, DB)
    
    IAAS (Infrastructure as a service). Cost $
        VMs are deployed by Cloud provider. You only control which OS is being installed and what services are there
        (OS, APP, WEB, DB)
    
    PAAS (Platform as a service). Cost $$
        VMs and OS are deployed by Cloud provider. You only control applications, WEB-server and DB
        (APP, WEB, DB)
    
    SAAS (Software as a service). Cost $$$
        VMs, OS, WEB-server, DB are deployed by Cloud provider. You only control your Application and support it.
        (APP)
    
Cloud Deployment Models
    Public - all the companies resources are on the cloud. Highly available, but data is stored on remote servers and VMs are on same hardware with other companies.    
    Private - all the companies resources are on the cloud resources dedicated specifically for this company. 
    Hybrid - a combination of both Public and Private cloud. Some apps might be published to public, some to hybrid, it can depend on the sensitivity of stored data.
    Community - a cloud environment created by two or more organizations for access for all of parties to a single common cloud of services

Pros of certain models:
    Private: 
        Security (you can restrict access to the resources);
        Data sovereignity (you own the data therefore it's yours only);
        Lower cost when a lot of users.
    Public:
        Elasticity (you can grow the amount of resources whenever needed or shrink them);
        Services (you can get some services that can't be obtained in Private model, like serverless computing);
        Simplicity (just run servers and use).
    Hybrid:
        All pros of above models are included

In Hybrid model, Cloud brokering is involved.
Users (when spinning up an application) are going to cloud broker, Like Cisco Cloud Center. And then they can choose where to deploy their APP:
    Private cloud;
    Public cloud (AWS);
    Public cloud (Azure);
    Public cloud (GCP).


#                    SD-WAN
SDN (Software-Defined Network) - network that is managed in a centralized way (though a controller or using scripts)
Additionally SDN have an underlay and an overlay. And usually tunnels is used to make sure overlay works as intended
SD-WAN  is an SDN applied to devices related to WAN (edge router for example)

SD-WAN  is separated into 4 planes (Orchestration, Management, Control and Data)
In usual network, Control and Data plane are both living on same device.
In case of SD-WAN (and SDN), controller is responsible for Control Plane (brain of network). Edge devices are responsible for Data plane (legs of network).

Differences:
WAN                                                             |   SD-WAN
________________________________________________________________|__________________________________________________________________________
Each device has own control plane                               |   Centralized control plane
All configuration, upgrades and monitoring done on each box     |   Configuration, upgrades and monitoring done centralized
Automation only through scripts                                 |   Automation can be done using API or scripts
New installation done from scratch                              |   New installation can be done with help of ZTP (Zero-Touch Provisioning)

Cisco solution was initially iWAN (intelligent WAN) but that failed and they made a purchase of Viptela company.
Currect solution is "Cisco SD-WAN". Also they have a Meraki SD-WAN solution

Cisco SD-WAN components:
    vManage - User logs in to it and configures all the settings required to be run on devices. Centralized management and monitoring happens here. Can be configured using REST API.
        DEMO: https://sandbox-sdwan-2.cisco.com/ (devnetuser/RG!_Yw919_83)
        Templates are used to push the predefined configuration to devices.
    vSmart - A controller, CAN ONLY BE VIRTUAL (public/private cloud or on-prem). It sends out the configuration (that has been configured on vManage) to routers. Config like Security/QoS/VPNs
    vEdge - Basically a Router. It is interconnected with other routers via IPsec tunnel. So underlay is a Provider network. Overlay is tunnels
        vEdge can be deployed in a Cloud therefore be virtual.
        Also there's a cEdge - Cisco ISRs with Viptela code running on them. Used to be able to support T1, E1 (Voice, serial) lines, or some security features
        Represented by Cisco devices (ASR 1k, ISR 1k, ISR 4k, ENCS 5400), device runs IOS-XE 18.3.0+
        It is not recommended to run both vEdges and cEdges in the same location
    vBond - An orchestrator (virtual deployment only, it must have a public IP) with OOB view on the SD-WAN. Used to glue all the components together. Orchestrates the connectivity between vEdges and vSmart controller.
        Whenever new vEdge device join SD-WAN, it goes to vBond which sends out the info to vManage. Then Administrator can approve or reject adding the vEdge to SD-WAN
        Can help for two vEdges to build an IPsec tunnel if both of them are behind NAT - that is called NAT traversal
        Example: it sends out a packet to two vEdges which open the certain ports. Devices then use that ports to build a tunnel. Or it can orchestrate so those devices exchange packets themselves.
                
Cisco SD-WAN topologies:
    Full mesh (requires license)
    Partial mesh (requires license)
    Point-to-Point (kind of requires license)
    Hub-and-Spoke (no license required)
    
    There are 4 different options in which SD-WAN solution can work whenever there are more than 1 WAN circuit (WAN or INTERNET):
        Active-Active (traffic is fairly splitted between two circuits)
        Active-Active weighted (traffic is splitted between two links but amount sent through each depends on the bandwidth) 
        Active-Standby (traffic is sent through Active only and is being failovered in case of issues on Active)
        Application-aware SLA - used to be able to see circuit degradation even if it's still up. Used to track the availability of some metrics for the circuit. DPI (required additional license) can help to identify the application.

SD-WAN Programmability
    NETCONF - a protocol that allows to send a configuration (XML) to devices using SSH
    REST APIs - a protocol that allows HTTP/HTTPS requests (GET, POST, DELETE, PUT) to be sent to some device. That allows to receive status and configuration of system

vManage GUI is actually a set of REST API replies received by sending the requests through the web page.
vSmart receives configuration changes from vManage by NETCONF
vEdges receive configuration from vSmart through OMP (Overlay Management Protocol)

Controller deployment models
    Public model: all components are deployed (and MANAGED!) by Cisco in their AWS space (2x vSmarts active/active; 2x vBonds Active/Active; 2x vManage Active/Standby). All of them have to be reachable through public internet space
    Hybrid model: controllers are now deployed in company's public cloud, or in the DC. VMs are Managed by the companies engineers. Their public IPs have to be reachable through all WAN circuits.
    Hybrid model w/private IPs: same as hybrid models but private IPs are used. In that case NATting is required since vBond have to have a PUBLIC IP-address.

Cisco SD-WAN hardware
    vEdge-100b (100Mbit connectivity max, both download&upload, ethernet only)
    vEdge-100m (100 but supports 2G/3G/4G modem)
    vEdge-100wm (100 but mobile + wireless)
    vEdge-1000 (1Gig connectivity throughput)
    vEdge-2000 (10Gig connectivity throughput)
    vEdge-5000 (20Gig connectivity throughput)


#                    SD-Access
SDN applied to Access layer of network.
All border devices in SDA have a VXLAN tunnel to every other device.

Traditional challenges:
    STP
    Manual config
    Poor scalability
    Difficult security policy

SD-Access is Campus Fabric with DNA center as a controller for the network.

Campus Fabric is:
    VXLAN used to build tunnels as overlay mechanism
    Location Identifier Separation Protocol
    CTS (ISE basically) - Cisco TrustSec

DNA Center is a physical appliance only(!)


SDA doesn't operate in subnets. If a new EP comes online, nobody knows a destination to it at first.

Cisco SD-Access layers:
    Physical (equipment)
        Control plane node (LISP) - a device responsible for storing the data of mappings (mappings DB) of each device (hosts, switches) in the network to other devices
        Fabric border device - Core
        Fabric intermediate device - Distribution
        Fabric edge device - Access
        Fabric WLC
    Network (logic and physics of the network)
        VXLAN - Data plane. Builds direct tunnels but don't have information of where to direct traffic to.
            (!)Additional 50 Bytes are used for the encapsulation. MTU of underlay should be increased by that number
            Usual Packet: [eth|IP|data]
            VXLAN Packet: [eth|IP|VNID|above_packet_as_data]
            VXLAN-GPO is introduced by Cisco to serve it's purpose on the Policy plane. It includes SGTs (Scalable group tag) to manage the policies received by hosts
        LISP (SDA) - Control plane. Directs the traffic to required switch or node of network.
            Each network device (switch or router) has an RLOC (Router LOCation) which identifies it in a network (usually it is a LOOPBACK address). 
            Each endpoint has a EID (Endpoint ID) and is bound to current RLOC (EID-RLOC mapping on CPN)
            Whenever traffic has to be sent from an endpoint to another enpoint, switch requests the direction from Mapping server (control plane node) and then just directs traffic as necessary
        CTS (cisco trustsec) - Policy plane. Used to enforce existing policies to ongoing traffic.
            Microsegmentation (two hosts on same network shouln't talk to each other). SGs (scalable groups) used for that.
            Macrosegmentation (two hosts on same switch, diffrent network shoulnd not have any possibility to access each other. VNs (virtual networks) are used for that. Each host is in it's own VN.
            Whenever traffic is received by network device, it receives a VN (virtual network) and SG (scalable group) tags. These are transformed into VNID and SGT afterwards to be trasfered further into network. By receiving device, policy is being applied according to tags.
            Tagging happes on ingress traffic and policy enforcement happens on egress traffic.
    Controller (a controller of whole solution + ISE)
        Cisco DNA center
            Network control platform
                Configures network devices automatically for us
                APIC-EM
                Automation engine (underlay & overlay), CLI/SNMP, NETCONF/YANG
            Network data platform
                Analytics engine -> Data itself. DNA collects all possible data (SNMP, Netflow, Syslog, Streaming Telemetry(retrieve preset bunches of data)) and writed to DB
                Assurance engine -> Value from data. Proactive monitoring (problem is boiling up), visibility (healthscore), configuration suggestions (can be run using autimation engine), path trace
    Management (GUI, REST APIs, CLI)
        4-step workflow
            1. Design (conf settings, profiles, definitions)
            2. Policy (set policy (ISE, AAA and stuff))
            3. Provision (assign role to a device)
            4. Assurance (monitoring of health of network)

ISE is a security solution used in SDA. It provides:
    Visibility (who, when, how, where connects)
    Segmentation (microsegmentation using SGTs - security group tags)
    Stop Threats
    
    Can be integrated with 3rd party solutions (Cisco ACI or other vendors) using pxGrid.

User Authentication process in SDA
    For 802.1x to happen, there have to be a supplicant - some piece of software for 802.1x exchange to happen from the endpoint.
    -- Supplicant (PC) sends a login request (802.1x exchange) to Network Authentication Device (NAD), usually an upstream switch.
    NAD forwards the request using RADIUS to ISE 
    -- Endpoint (Phone) is being authenticated using MAB (MAC Authentication Bypass). Whenever phone is connected to network, NAD expects 802.1x exchange.
    Since it's a phone, it doesn't happen therefore MAB exchange with ISE happens. For that MAC-DB is required and manual adding of every MAC to ISE is required.
    -- WebAuth also exist. An endpoint doesn't have any 802.1x supplicant installed.
    Endpoint connected to network gets a DHCP address with DNS servers. After that when enpoint tries to access a website, request gets intercepted and EP is directed to ISE to login.

SDA EP onboarding
    EID in SDA is the IP-address of Endpoint connected to fabric.
    Whenever device is connected to network, it get's tagged with EID and switch A send out a registration message to control plane node (CPN).
    After that CPN know about that node and whenever some other switch (B,C) wants to send traffic to that EID, it forwards it to required switch A.
    If switch B or C already sent traffic to that EID, info about direction is stored in their DB so they know the RLOC to which they should send that traffic further on (expired if no data is sent in 24H)
    In wireless, another VXLAN tunnel is formed with AP and it's data traffic is sent through that tunnel instead of CAPwAP tunnel (as usual)
Client ROAM
    If client roams (moves frow Switch A to Switch B):
        1. Switch B send info to CPN about client being on it
        2. CPN informs Switch A about client not being on A anymore
        3. If some other switch send a packet to Switch A thinking client is there (saved in cache for example), Switch A responds with "no it's not here" and ALSO send packet to Switch B
           Other switch then updates it's DB from CPN and now stores new cache

External subnets in SDA
    Internal border node - connects with internal subnets (DC, legacy, other) 
    External border node - connects with external subnets (Internet)
    When fabric devices send traffic to other subnets, process is similar, they ask CPN and it forwards traffic to certain border node.
    After that, border node sends out traffic according to basic Routing protocols.

SDN implementation and effect upon planes:
Imperative approach - Only controller has control plane. In case devices loose connectivity to controller - they're dead since no decision can be made
Declarative approach - Devices has their own control plane which replicate the state from controller. In case connectivity to controller is lost, devices can handle traffic on their own

ETA (encrypted traffic analytics) - technical suite allowing to check encrypted traffic for malware without decrypting it
Includes:
    Stealthwatch
    ISE
    Cat 9K
    
    Data is collected by Stealthwatch and forwarded to Cognitive Intelligence - Cisco Cloud system that scans the traffic in 3 steps:
        1. Anomaly detection. Is traffic Malicious or Weird?
        2. Malicious events. What is it, what it has done?
        3. Threat Analysis.
        Then the score (1-10, 10 being the most malicious) is set for traffic.


#                    Hardware

9k Switches
    Good thing about 9k switches is that they support UADP (Unified Access DataPlane ASIC technology) - features can be added to it through patches.
    9200 (stack, UADP-Mini) - used to replace Catalyst 2k
    9300 (stack, Modular) - used to replace Catalyst 3k
    9400 (chassis, SwV(VSS)) - used to replace Catalyst 4k 
    9500 (fixed) - used to replace 4500X, 6880-X (CORE) 
    9600 (chassis, SwV) - used to replace Catalyst 6k (CORE)

Routers
    ISR (4300, 4400 supported by SDA) - focuses on features, not on performance. UCS and other stuff can be deployed on it
    ASR (1000 supported by SDA) - focuses on performance, bandwidth, table sizes (BGP internet table for example)
    
Wireless
    WLC (Catalyst 9800s) - a 9k Cisco that serves as a WLC
        HW (9880, 9840, 9800-L similar to 9200-L, 9300-L)
        SW (Embedded WLC, 9800-CL (CL for cloud)). Cloud version can be deployed in private DC or in Cloud
    APs (9100) - supports mGig technology which allows transfer of big Gbit/s (up to 10) through WIFI.


#                    QoS (Quality of service)
QoS - allows the manipulation of traffic (packets) in a way to prioritize some important data to be transfered with better confidions (sometimes be transferred at all - in case bandwidth of an interface is lower that the amount of traffic)

Mechanics:
1. Traffic is classified
    3 bit number in a frame (in 802.1Q field) that immediately tells Network Device (like a mark on an Amazon package) what is that traffic
2. Traffic is marked
    8 bit ToS field in a packet. Before IPP (IP Precedense) was used with 3 bit. Now it's DSCP (Differentiated services code point) with 6 bit
(*)1 and 2 doesn't do anything by itself, these are just numbers in traffic. If device doesn't have any rules about how to handle such traffic - they're useless
3. Traffic is threated based on 1 and 2

QoS tools:
1. Classification & Marking  sa- used for ingress traffic. Packets get classified (by source, destination IP, protocol, ports) and marked on ingress interface with certain priority for egress interface to understand how it should be forwarded. 
    NBAR (Network-Based Application Recognition) exists which lets the classification to happen with more details (based on word, phrase or URL for example)
2. Policing - used on ingress interface (provide edge interface facing customer edge device) to drop all the traffic that exceeds the configured policing bandwidth. Can be configured for different types of traffic (TCP, UDP, DNS, other apps.)
3. Shaping - used on egress interface to create queues (queueing) which allows traffic that exceeds the configured bandwidth to be sent later, whenever the circuit is less busy. 
4. Queueing - used on egress interface to divide egress queue on sub-queues
5. Scheduling - used to empty the sub-queues in a certain way.

Marking:
On layer 1: based on physical port or interface (subinterface/SVI)
On layer 2: based on MAC-address and on COS (class of service) bits (802.1P)
On layer 3: based on IP-address (source/dest) and on DSCP mark (TOS bits)
On layer 4: based on TCP or UDP port
On layer 7: using NBAR2 - determine the application based on signatures that are stored on cisco devices

Congestion management - a set of tools (egress interface) that manages happened congestion. WFQ, CBWFQ, PQ, LLQ, WRR, SRR, Shaping.
    Weighted Fair - low-bandwidth traffic is done first. Problem: important traffic can be stopped from transmitting
    Class-based - guarantees a service to get a definite amount of bandwidth (40Mbit/s out of 150Mbit/s link). Can't guarantee the priority
    Priority - Guarantees the priority queue over other traffic
    LLQ (low latency queuing)
Congestion avoidance - a set of tools (ingress interface) that prevents congestions from happening. RED, WRED, WTD, Policing.

QoS models:
Best Effort - usual device behaviour. Packets being sent if possible. If there's congestion, packets are queued and are sent on FIFO basis. They are taildropped if buffer is overflown.
Integrated services - unified settings on all the routers on the way. Uses RSVP (Resource Reservation Protocol). A certain bandwidth is reserved by RSVP packet that is being sent before the call establishment. If there's no bandwidth available - call is dropped.
Differentiated services - each router has it's own settings. Uses PHB (Per-Hop Behaviour). DSCP marking happens here that can assure traffic to be passed further.

QoS Policies
Before all QoS settings had to be applied to interfaces.
Now QoS settings are applied globally. Interfaces are used to display the direction of traffic for QoS to be applied.

MQS (Modular QoS Command-Line):
QoS applied globally.
Allows for different QoS tools to be applied to different interfaces.
3 components:
  Class-map - used to match certain type of traffic and classify it
  Policy-map - used to match certain traffic classified by class-maps. To initiate the use of class-maps
  Service-policy - used to apply a policy-map to interface. Inbound/Outbound.

QoS Marking rules:
Layer 2
    CoS (Class of Service) - 3 bit number in a frame (in 802.1Q field). Can take numeric values from 0 to 7
Layer 3
    ToS (Type of Service) - 8 bit field (names ToS) in a packet (L3). It is categorized as per below:
        IPP (IP precedence) - takes 3 bit in ToS field -> as CoS, values can be from 0 to 7.
            Best practices (not mandatory) are:
                value 0 for FIFO traffic
                value 1-4 for priority traffic (1 being scavenger traffic which is some junk from users like bittorrent and others)
                value 5 for VoIP
                value 6-7 for network traffic (STP, routing etc.)
        DSCP - takes 6 bit in ToS field -> divided in "major" and "minor" bit groups of 3. That is for intercompatibility of two methods of marking.
            Major bits are same as used by IPP - "Class Selector"
            Minor bit group is called "Drop preference"
            DSCP is split as: AF (Assured forwarding) with a two numbers - first being it's class (higher is better) and second is a drop preference (lesser is better).
                              CS (Class selecter) is just a usage of first 3 bits which is a class. Drop preference is 0 - not used.
                              EF (Expedited Forwarding) - usually used for Voice. Class 5 and Drop preference value is higher (11)

Very useful "QoS Values Calculator":
https://i.pinimg.com/originals/34/a7/57/34a75767cda2db32f3aca013b65d91af.jpg

1. Traffic from 10.0.0.1 to 10.255.1.1 is matched and classified by class-map.
2. Policy-map initiates the class-map and makes the classified traffic to follow the specified rules. Apply policing for all the traffic from 10.0.0.1 to 10.255.1.1 (from gi0/1 to gi0/3) to prevent it from congesting the interface gi0/1.
3. Apply service-policy to interface gi0/1 (inbound) and interface gi0/3 (outbout) to use the above rules for this traffic.

It is a best practice to move "Trust boundary" as close to Endpoints as possible (wherever this can be safely applied, like IP phones).
Trust boundary is the point where marking can happen - it loads up resources.



#                    Network device architecture
Process Switching - old intra-device frame/packet switching mechanism. Every frame/packet received by device is forwarded to CPU for it to lookup the MAC / Routing table and decide where Data unit should be egressed.
Fast Switching - attempt to remove CPU from the process as much as possible. Cache is introduced. Destination is stored in Cache after single packet has been "process switched"
    can be enabled by command on interface "ip route-cache". Fast Switching happens REACTIVELY

RIB - a "Database" of all the information about learned routes on the device. Routing Table includes a formatted data taken from RIB.
    RIB entry contain: "Letter of used routing protocol" - "AD" - "METRIC" - "IP-address" - "Subnet Mask" - "Next-hop IP" - "outbound interface"
CEF (Cisco Express Forwarding) - Cisco proprietary mechanism for DataUnit switching. It allows for switching decisions to be made PROACTIVELY.
    Before packet with unknown destination arrived, a FIB table is populated from the RIB. FIB table contain Network, Egress interface and Next-hop address (MAC)
    In recent CEF implementations, all the FIB data is collected beforehand from RIB.

FIB - a copy of RIB (network+egress interface+nexthop) + ARP lookup for destination(nexthop) MAC-address. Used for switching decisions in CEF.
Adjacency table - an L2 information with MAC-addresses for packet to be reincapsulated into a new frame (with new source and destination MACs) while forwarding.

https://linkmeup.ru/podcasts/959/ --- CEF, Process Switching deep dive

Below are used for quick lookup by network device. Lookup on MAC-address / Routing, ACL, QoS.
CAM - located in a RAM (high-speed memory for quick lookup). Contains the information about MAC addresses of devices in subnets. MACs are represented as MAC-address table. Can only be used by switches
    Can only be looked up by the exact match for MAC-address
TCAM - located in RAM (high-speed memory for quick lookup). Contains IP-addresses and subnet masks. Also contain ACL/QoS rules. Provides as assistance with decision of best path based on longest match (/30 mask is longer match than /24). Subnets and masks are represented as Routing table. Can be used by routers and multilayer switches.
    A request to TCAM is made by providing IP address and Mask and getting a result (next hop IP, permit/deny responce) in answer.
    Lookup is made in following format: V,M,R (V-Value(IP),M-Mask,R-Result(allow/deny,nexthop))
    TCAM is ternary - there is a possibility to specify that something is 0, 1 or X (whatever). That allows for Mask to be stored (for example 27 mask would be 11111111.11111111.11111111.111XXXXX)
***(get to know more on CAM & TCAM)
RP (route processor)

Centralized and Distributed switching architectures
Centralized - Packets that has been received by interface on Line card are forwarded to Supervisor before switching decision (CAM & FIB lookup)
Distributed - Packet received on Line Card is forwarded firectly out of line card through corresponding interface (CAM & FIB tables are stored on LC but populated by Supervisor). Control traffic (like OSPF) still happens on Supervisor


#                    Virtualization
#        ENDPOINT VIRTUALIZATION 
Hypervisor - a utility that allows management and provides visibility of Guest OSes (VMs)

Hardware server is called "HOST"
Virtual server is called "GUEST"

Type 1 hypervisor - a hypervisor that is installed directly on Hardware (HW --- Hypervisor --- VMs)
Type 2 hypervisor - a hypervisor that is installed on some OS like Microsoft Windows (HW --- OS --- Hypervisor --- VMS)

Usually Type 1 HV is used in Production networks (in DC), Type 2 is used for LABs and not recommended for PROD deployments.
VMs basically is a data, it's all files that are used and manipulated by Hypervisors. That allows to store them on external "Storage arrays". Which allows VMs to be access not just by a single HV but by many others.
So if some HOST goes down, another HOST can overtake the VM and spin it up on itself
Also VMs can be moved between HOSTS (VM migration)
    Cold migration - VM is turned down and spin up on another HOST. Both HOSTs should be able to access storage with VM files
    Live migration - VM is migrated withoup being turned off. It can be configured to happen automatically if some HOST gets clogged with VM resource utilization
    
Additionaly there's a feature called FT (Fault Tolerance) which is basically the HA.
    If there's a VM on a HOST 1, a copy of it is run on HOST 2 which has FT configured with HOST 1. If HOST 1 is down, VM is getting up on HOST 2 with little to no downtime.

Virtual Switch (vSwitch) - literally a virtual switch inside HOST that connects it's virtual interfaces (ports) to VNICs distributed to running VMs. 
vSwitch can distiribute certain connected devices to their own broadcast domains ("VLANs"), this function is represented by port-groups.
Virtual Switches exist on type1 Hypervisors by default (ESXi vSwitch, Microsoft Hyper-V)
Broadcasts received by vSwitch are forwarded only to VMs living on the vSwitch. 
Supported features: VLANs, CDP, SPAN, Trunk
Traffic flow mechanism PINNING is used to prevent vSwitches from learning upstream MACs
Above is Standard deployment of vSwitch.

Also Distributed deployment exists:
    Allows for centralized deployment of vSwitches to many HOSTs and further management (and configuration management)

Cisco ENCS 5000.
    Allows for virtualization of network devices (CSR - CloudServicesRouter, ISRv, vWLC, ASAv, vEdge/cEdge)
Also can be virtualized on other server hardware.


#        DATA PATH VIRTUALIZATION
VRF - A technology that allows router (or Multilayer device) to logically be divided into few other routers (which basically means into few independent Routing tables)
    It is impossible to share routes between VRFs, one VRF can be reachable from another VRF only if there's a connection between them through external interface.
Each VRF will have their own routing table (for example that allows few similar (10.0.0.0/24) subnets to be defined in each VRF and still be reachable).
Usually used in ISP networks therefore requires ISP network technologies (MPLS, VPN, L3VPN, BGP).

***(VRF-Lite - an Enterprise solution that allows routing to be done through common routing protocols (OSPF, EIGRP). No extra VPN protocols required.

VRF Configuration:
    vrf definition <vrf_name>
    address-sapce
    *add interface to vrf forwarding (IP address get's stripped from interface here)

Tunnel - an overlay technology allowing for isolation of traffic from the underlay networks and providing direct L3 connectivity even between devices located far away from each other
GRE (Generic Routing Encapsulation) - a tunnelling protocol that allows an incapsulation of IP or other packets
    GRE encapsulation adds a GRE header (4Byte) and an IP header (20B).
    IP protocol number for GRE encapsulated packet is always 47 (https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)
GRE recursive routing: there might be an issue that the tunnel destination addressed is being routed through the tunnel itself.
To avoid that, tunnel and underlay networks should be in different Routing Domains (OSPF and EIGRP; EIGRP AS 1 and EIGRP AS 2; any other pairs except for the same ones)
!   MTU IS IMPORTANT. Tunnel default MTU is 1476

GRE configuration:
    interface tunnel<#>
    tunnel source
    tunnel destination
    ip address
    keepalive <interval> <retries_until_down> (keepalive is unidirectional - the packet that is originated on the router is destined to ITSELF; GRE header is added to packet, routed to tunnel destination, GRE header is stripped on the other side and uncovered IP packet is destined to the original router)

IPsec - a framework (set of tools) that allows a construction of secure tunnel from one destination to another
    Key features that IPsec provide are data...:
        AUTHENTICATION - packet exchange is happening between certain devices and they make sure it's actually them
        INTERGRITY - allows to make sure that data that contains in packets are the same on source and destination devices
        CONFIDENTIALITY - allows the encryption of the exchanged data

IPsec can have two different packet headers:
    AH (Authentication Header) - IP protocol 51. Allows to fullfill the data AUTHENTICATION and INTEGRITY. Can't encrypt
    ESP (Encapsulation Security Payload) - IP protocol 50. Can do all data features of IPsec. Also can do NAT traversal. 
        ESP can work in two modes:
            TRANSPORT Mode - When original IP packet with payload arrives to router, ESP header is added between Payload an previous IP headers. After that GRE header will add up in front(?)
            TUNNEL Mode - When original IP packet with payload arrives to router, ESP header is added in front of whole packet. another IP header is added in from of ESP header after that.

Both Transform sets have to match between devices.
Transform sets include Hashing algorithm of choice, encryption algorithm as well.
Transform sets are initially exchanged through IKE (Internet Key Exchange) tunnel.

Before establishing an IPsec tunnel between routers, they have to exchange the security settings that should be used (encryption keys as well etc.)
For that, IKE is used to build a Control plane tunnel (also called SA - security association).
IKE is an implementation os ISAKMP (Internet Security Association Key Management Protocol).
IKE:
    IKEv1. Consists of two phases of communication:
        Phase 1. Control plane tunnel is build with no encryption (def lifetime is 24hours, reestablished after). Following information is negotiated:
            Acronym to remember is HAGLE
            HASH 
            AUTHENTICATION METHOD
            GROUP (Diffie-Helman) - used to securely encrypt keys for encryption and decryption of traffic
            LIFETIME of a tunnel
            ENCRYPTION algorithm
       Phase 2. An IPsec tunnel (def lifetime is 12hours, reestablished after) itself can be established after phase 1 is completed 
            PFS - used to force use of different session keys other than ones that were used during phase 1.
    IKEv2. Phases are collapsed and less messages have to be exchanged before tunnels are established.
    Both devices have to use same IKE version.

IPsec configuration steps:
    0. Configure IKE phase 1.
        //crypto isakmp policy <security value> (lesser value - more prioritized)
         //hash <value>
         //authentication <value>
         //group <value>
         //lifetime <value>
         //encryption <value>
        *if auth - psk* //crypto isakmp key <KEYVALUE> address <IP_OF_TUNNEL_DESTINATION>
    1. Configure Transform-set (//crypto ipsec transform-set <TS_NAME> <PACKET_HEADER_VALUE>-<HASH_VALUE> *if ESP*<PACKET_HEADER_VALUE>-<ENCRYPTION_VALUE>)
        a. Choose AH or ESP
        b. Specify HASH and ENCRYPTION
        c. If ESP: Tunnel or Transport mode (//mode <tunnel-or-transport>)
    2. Configure IPsec Profile (prev. Crypto-map). Create a link to transform-set in IPsec profile. (//crypto ipsec profile <PROFILE_NAME> -> //set transform-set <TS_NAME>)
    3. Apply IPsec Profile to Tunnel interface.
        *under tunnel configuration* //tunnel mode ipsec ivp4
        //tunnel protection ipsec <PROFILE_NAME>

But before configuring IPsec, IKE (ISAKMP) have to be configured:
    crypto isakmp policy (HAGLE)
    (if PSK)crypto isakmp key <key_value> address <IP_address_destination>

Tshoot:
    show crypto isakmp sa
    show crypte ipsec sa

VPN types
    Site-To-Site - Basic tunnel between two sites. Tunnel can be established between Cisco and other 3rd party device. Lacks Routing, Multicast, Scalability.
    DMVPN - A hub-and-spoke VPN topology. Many sites establish VPN to the main site. With that it's not required to create a lot of tunnels. NHRP protocol is utilized. Routing is configurable. 
    FlexVPN - A "DMVPN+". It utilizes IKEv2 (can only use version 2 while other VPNs MAY use v1 OR v2) to make DMVPN better but is a different VPN solution
    GET VPN - a VPN for WAN over MPLS networks. Allows to keep the original IP header (with QoSes and other stuff).    


#                    LISP (as an independent protocol)
Was designed to help with the growing amount of routes on the edge Routers (940K+ routes at the moment of writing).
Instead of having all routers have all the prefixes existing in the internet, LISP allows to have a router domain where packet destined to certain IP, have to be transfered just to the router that has this IP-address on itself.
All the routers on the way form an Underlay network and doesn't have to know the location of end subnet because packet is encapsulated with additional LISP, UDP (4341) and IP headers. They forward packet to end router.
That removes the load of knowing all prefixes from many routers.

There are Control plane and Data plane available:
    Control Plane (LISP) - Where should the packet be forwarded. Mapping server helps to understand where should packet be forwarded.
                           Mapping Server has access to all the routers on "as-needed" basis as opposed to "have-all-prefixes-at-the-same-time" of usual Routers.
    Data Plane (LISP) - How should packet traverse the network to reach the destionation. 
                         Answer is - TUNNELING. Router that first receive the packet is ITR (Ingress tunneling router), last router that receive the packet - ETR (Egress tunneling router). These are relative so sometimes named xTR.
                         A usual IP packet in encapsulated on entering the tunnel.
                         LISP header is added to it with INSTANCE ID information (helpes to identify VRFs as VNID in VXLAN)
                         UDP header is added. UDP port that is used - 4341. Source port is dynamically that optimized Load Balancing in the environment ***(OPTIMIZE?)
                         IP header is added. With tunnel IP addresses for packet to be able to reach the destination (RLOC of remote router).
                         
LISP is used to separate Location and Identity.
In LIPS, each router has it's own Locator = RLOC. And each router has a subset of EIDs (endpoint IDs) which are usually subnets.

There are Map Resolver (MR) and Map Server (MS). Usually these are routers. And usually MR and MS are combined in one device.
Whenever new EID is added to RLOC, it is reported to Map Server. 
After that whenever other router has to send traffic to that new EID, Ingress router can send a request to MS and will then know where does subnet lives.

If there's a traffic going outside the LISP domain (external), edge router takes a role of a proxy. 
It can be either Proxy Egress tunnelling router (PETR) or Proxy Ingress tunnelling router (PITR), depending on the destination of traffic.

LISP operation:
Whenever Router gets connected to LISP domain, it shares it's EIDs (subnets) to the MS. That allows MS to know subnets connected to routers.
    1. Router 1 gets into LISP domain.
    2. It sends "MAP REGISTER" message to MS with subnet A in it.
    3. MS adds subnet A to it's table and send back the "MAP NOTIFY" message.

Whenever another router (router 2) wants to send a packet to subnet A on Router 1:
    1. Router 2 sens a "MAP REQUEST" message to MR.
    2. MR checks the table and sends "MAP REQUEST" further to Router 1 since the subnet is there.
    3. Router 1 then sends a "MAP REPLY" message back to Router 2.
    4. Then the LISP tunnel is getting formes to send traffic directly between routers. Packet's IP header includes source IP of Router 2 and Destination of Router 1. Real packet IPs are not known to devices on the way.

Whenever the packet is destined to some subnet that does not exist in the LISP domain (located externally):
    1. Router 2 sends out a  "MAP REQUEST" with subnet Z to MR.
    2. Since there's no subnet Z on MR, it replies with "NEGATIVE MAP REPLY" - subnet not found in DB message. That means that subnet must be outside on LISP domain.
    3. After that Router 2 sends out the packet to PxTR device to handle. 
        
PxTR does not add upstream subnets to the MR DB.


#                    VXLAN (as an independent protocol)
VXLAN allows L2 packets to be successfully traversed over the L3 network.

Usual Frames (with ETHERNET header included) are being encapsulated and following are being added to them:
    1. VXLAN header (8 Byte)
    2. UDP header (8 Byte)
    3. IP header (20 Byte)

    Above is called "MAC-IN-IP/UDP" 
    UDP/4789 is used.

    Also ETH header adds up 14 Bytes. (18 Bytes with VLAN data, but usually it gets added to VXLAN header)
    Since L2 header is not counted up to starting MTU (up to 1500 bytes) of not encapsulated packet.

That means each VXLAN PDU accumulates additional 50 Bytes of MTU to the tunnel.

VXLAN virtual identifier (VNID) allows up to 16 million diffrent values. 
VLAN only allows 4096.

While LISP has ITR and ETR, in VXLAN, routers are called VTEP.

VXLAN is Data Plane protocol only, by itself it cannot move packets in the network since it doesn't know where to move them.
Control Plane protocol is required: 
    - Usually MP-BGP is used for that
    - ACI uses COOP
    - SDA uses LISP


#                    STP (Spanning-tree protocol) 

  !#####!   BPDUs IN A CONVERGED TOPOLOGY ARE INITIATED BY ROOT BRIDGE BY DEFAULT. IF NON-ROOT BRIDGES DOESN'T RECEIVE BPDU FROM ROOT, THEY WON'T SEND THEIR OWN BPDUs.

https://linkmeup.ru/podcasts/977/
STP (802.1D) - protocol that allows switches to become aware of other switches through advertising and receiving of BPDUs. Also build loop-free topology
BPDU - Bridge Protocol Data Unit. Some of BPDU information that is exchanged between switches within STP process:
    Bridge ID - An ID of each switch which consists of:
        - Bridge Priority (default - 32768, steps are 4096)
        - Base MAC-Address
        - VLAN number (number is added to priority value)
        Root Bridge is usually chosen by manually set priority. If both Priority value and VLAN are same, then tiebreaker is MAC
    Port Cost - Cost of port (depends on port bandwidth)
        - Ethernet (10Mbit/s) has a cost of 100
        - FastEthernet (100Mbit/s) has a cost of 19
        - GigabitEthernet (1,000Mbit/s) has a cost of 4
        There are different standarts to STP specifying the cost of ports (802.1d-1990 - unique values up to 600Mbit/s, 802.1d-1998 - unique values up to 50Gbit/s, 802.1t-2001)
    Root Path Cost - Consist of Port Cost sums for all the links on the way to Root Bridge
    Port Priority + ID - An STP Port Priority (default value is 128). Also adds up the port ID in system. gi0/0 is 1, gi0/4 is 5 and on.

There are 2 types of BPDU:
    Configuration BPDUs - is sent out initially during first negotiation and STP domain establishement
    Notification BPDUs (TCN topology change notification) - is sent out when configuration change (or failure) happens. TCN then is being forwarded up to ROOT, then root sends out the BPDU with confirmed topology change.

Elect Root Bridge ID:
    Each switch at the beginning sends out BPDUs saying "I AM ROOT BRIDGE". 
    Whenever device receives BPDU with lower Priority value, or Base MAC-Address, it stops considering itself Root and proceeds to choose root ports.

Elect Root port:
    Root port is chosen based on the Port cost towards the Root. 
    Root switch advertises cost to Root as value 0 (since it is a Root). Each following switch advertise cost to root as 0 + it's own best value.
    If switch has two ways with same port costs towards Root, the port towards the bridge with lower bridge ID will be chosen as Root.
    If all potential Root ports are leading to switches with the same bridge IDs, Root port will be elected based on lower priority + port ID of the connected Switch.

Elect Designated port:
    Designated port is port facing away from the root (basically any NON-ROOT). 
    In the link between two switches, there is only 1 (ONE) designated port and one blocking port.
    1. Chosen based on the path cost to the ROOT (can be checked with "sh spanning-tree vlan 10 root cost")
    2. If path to ROOT is of the same cost, Designated port is chosen on the switch with lowest bridge ID (priority-vlanID-MAC)
    
Used for network to be redundant AND loop-proof. In a topology with many links going between devices, a root switch is elected through certain means, then the topology is build for every device in broadcast domain to be able to reach the root though SINGLE path.
Root gets all ports used. Downstream switches only have a single path to the root, others can never lead to root switch in normal circumstances.

STP port states:
Disabled - Port is admin down
Blocking - Port cannot forward network traffic. Can receive BPDUs
Listening - Can't forward network traffic. Can receive and forward BPDUs
Learning - Can't forward network traffic. Can receive and send BPDUs and alter MAC address table based on received network traffic 
Forwarding - Port operates as usual
Broken - Port discards packets while issue exists

Interface types: 
    P2P = FULL DUPLEX interface (FDI)
    P2P Edge = FDI with portfast turned on
    P2P Peer(STP) = FDI with neighbor on that interface running older version of STP
    Shr = HALD DUPLEX interface
    Shr Edge, Shr Peer(STP) apply as above for shared ports as well.

Timers in 802.1D:
    Hello Time (def 2 sec) - how often does BPDUs are sent out. When non-root bridge receives BPDU (initially sent out by ROOT only), it sents out it's own BPDU to ports int LRN and FWD state
    Max Age (def 20 sec) - if switch doesn't receive BPDUs from neighboring device for that amount of time, port is transitioned to blocking state and (if it was on a root port) new root port is chosen
    Forward Delay (def 15 sec) - how long does switch stay in LISTENING and LEARNING states after topology change
    Aging Time (def 300 sec) - how long does dynamically learned MAC-address is kept in the MAC-address table if there is no traffic received from it
        Timer is set to 15sec if topology has been changed (TCN sent/received)
        
STP customization:

Uplinkfast - if switch has more than one port in the direction of root, this feature allows it to transition that port from blocking directly to forwarding state if the active root port stops being operation (admin shutdown or anything else)
        To turn feature on: "spanning-tree uplinkfast" in global config mode. Used only to response to failure that is detected on switches own link (from other side as well)
        + 3000 to port cost; 
        + a lot to the bridge priority (+ ~16000)

Backbonefast - if switch gets cut out of it's root port (shutdown or failure on any side) and the only other link towards the root is in blocking state from other end, the switch will think it is a root bridge (since blocking port doesn't send BPDUs) and will send out it's own BPDUs "i am now root bridge". Feature allows to detect these occasions (on the switch which has a blocking port) and response by sending a backbonefast request to current root to make sure it's still there and will then transition it's blocking port to designated state (to send out BPDUs) to let switch with failed link know that there is a better bridge (with better bridge ID)

Portfast - feature that allows to specify ports connected to single devices and bypass the LIS and LRN states when they come up. Port up/down events will not be considered as topology change (hence no TCN)
    Can be configured on port or globally (if globally, ACCESS interfaces are switched to portfast)
    Can be configured on trunk - "spanning-tree portfast edge trunk"

BPDU Filter (usually in combo with Portfast) - a feature that prohibits BPDUs from being sent or received on specified interface.
    Can be configured on port or globally (if globally, feature is applied to interfaces with PORTFAST)
    "spanning-tree bpdufilter enable"

Root Guard - a feature enabled on the interface that prohibits STP topology changes if these interfaces receive BPDUs with bridge IDs better that current root bridge ID
    Enabled on the interface - "spanning-tree guard root"
    
BPDU Guard - a feature that is applied to the interface with PORTFAST configured, puts the interface in err-disabled state if it received BPDUs
    Usually, portfast configured on ports towards the endpoint hence it will not send BPDUs. If by mistake some switch is connected to that port, interface will go down
    Can be configured on port or globally (if globally, feature is applied to interfaces with PORTFAST)
    Enabled on the interface - "spanning-tree bpduguard enable"

Loop Guard - a feature that attempts to prevent any loops that could arise by forcing the blocked ports to transition into BROKEN state instead of LIS and LRN after not receiving BPDUs for the duration of Max Age (20 seconds)
    Can be configured on port or globally
    Enabled on the interface - "spanning-tree guard loop"
    
UDLD (UniDirectional Link Detection) - a feature that allows to watch the physical state of links between switches since STP-wise everything can be fine but issues on the physics may break the link with no clear indication of that


RAPID SPANNING TREE
RSTP (802.1W) - faster, younger, cooler version of STP with some additions comparing to initial protocol.
    RSPT MAIN POINTS:
    Enabled globally "spanning-tree mode rapid-pvst" (on Cisco)
    Uplinkfast and Backbonefast are automatically included into the RSTP (no need to configure)
    If case of Topology Change (TCN), switch that detected the change sends out the topology change BPDU instead of forwarding it to ROOT first
    Usually Max Age timer (when no BPDUs received for that time, link is considered as failed) is 20 seconds but in RSTP the issue is detected if no BPDUs are received for 6 seconds

    !#####!  IN RSTP, BPDUs ARE SENT OUT EVERY *hello_timer* SECONDS AUTOMATICALLY. ROOT BRIDGE DOESN'T HAVE TO TRIGGER EVERYONE BY SENDING BPDU FIRST.

What remained the same in 802.1w:
    Root ports
    Designated ports

What changed:
    Blocked ports now divided into two different roles
        Alternate port (port is blocked but could be ROOT PORT) - it is allowed to transition to forwarding if root port fails. Basically it received 2nd best BPDU
        Backup port (port is blocked but could be DESIGNATED port) - it serves as backup to DESIGNATED port. Not seen often nowadays because it required few ports on one switch to be connected to shared medium (like hub)

RSTP Port states:
    (BLK)Discarding (combines Disabled, Blocking and Listening states from original STP) - receives BPDUs on the interface. Doesn't forward user data. Doesn't add MAC-addresses to MAC table
    (LRN)Learning - receives and sends out BPDUs. Adds MAC-address to MAC table for user data traffic but doesn't forward the data
    (FWD)Forwarding - Receives&sends out BPDUs, forwards user traffic


#                    MST (Multiple Spanning Tree) (802.1S)

MST allows creation of STP instances and to include VLANs to that instances. That is next level to PVST since PVST creates a new STP instance for each VLAN. If there are many VLANs - huge resource load will be created.

It can be used for load balancing - Create 2 instances for 2 different paths and use both to forward traffic for half VLANs through one path and other half through another.

    MST configuration:
    In MST, there is a concept of "REGION".
    Region - a nubmer of switches running in the same MST-"domain".
    
    For switches to belong to same Region, following should match for them:
        Region Name;
        Revision Number;
        Instances with VLAN mappings (VLANs are mapped to instances).
        
    There is always Internal Spanning-Tree (IST) Instance - 0. Apart from 0, any number up to 16 can be used.
    When configuring MST, first of all "spanning-tree mode mst" should be run to change STP mode to MST.
    
MST uses long port cost by default:

BW------short---long--|
10mb-----100---2000000|
100mb----19----200000-|
1000mb---4-----20000--|
10000mb--2-----2000---|

MST switches exchange BPDUs just for region instance 0 which is IST (Internal Spanning-tree). 
Within these BPDUs, there is an information about all other MST instances. 
In that BPDUs, there is an information about Instance 0, it's root bridge ID, root cost, protocol identifier (MST).
Also there's MST instances data, it's root bridge ID.

!When there's a switch in topology that doesn't belong to the MST region.
   -If there's a switch that is not in MST region, all the MST switches connected to it will advertise (in BPDUs) the ROOT bridge of IST. And the cost for EACH switch to reach the root will be 0.
    The tiebreaker for which port to be ROOT and which one to be blocking is about the bridge ID of a switch connected to each port.
    Each advertisement has a root cost of 0, root bridge is is the same (ROOT of IST), but advertising switch bridge ID will differ, based on that the decision will be made.
    That sometimes can lead to suboptimal path but it is what it is.
   -If the switch not is MST region is recognized to be Cisco running PVST, all the MST switches will send out a BPDU for each VLAN that should be advertised over trunk between them.



#                    Trunking protocols (802.1Q, VTP, DTP)

802.1Q - the true trunking protocol. Allows to exchange tagged packets between switches (of ANY vendors)
VTP (VLAN trunking protocol) - Allows switches to exchange VLAN DB data between each other. If VLAN is created on some switch, other switch connected to it creates same VLAN automatically
DTP (Dynamic Trunking Protocol) - Allows switches to form TRUNK links automatically if two switches are connected between each other and configured accordingly:
    "Auto" mode - "i'll be trunk if you want to". Only receives packets. Trunk will be formed only if other side is "Desirable" mode or Hardcoded TRUNK (switchport mode trunk). 
    "Desirable" mode - "i want to be a trunk, do you want to?". Switch sends out packets to form a trunk between switches. Trunk will be formes if other device is configured with "desirable" mode or "auto" mode or hardcoded trunk.
    Hardcoded trunk is a one, that is configured by entering "Switchport mode trunk"
    
    DTP is not recommended due to "VLAN HOPPING ATTACK". Trunk should be hardcoded manually.
    

#                    EtherChannels (troubleshoot,configure) - basic, LACP, PAgP(ciscoProprietary)
Usual Etherchannel - just turn on the port-channel on both sides
    When in usual mode, "show etherchannel" won't show that EC is down.
LACP (Link Aggregation Control Protocol) - two modes of EC (passive - enable LACP only if LACP device is detected) (active - enable LACP unconditionally)
PAgP (Port Aggregation Protocol) - two modes of EC (auto - enable PAgP only if PAgP device is detected) (desirable - Enable PAgP unconditionally)

TSHOOT:
    "show interfaces port-channel" - check port-channel interface (members, speed)
    "show lacp/pagp counters" - check counters, DUs that are exchanged
    "show lacp/pagp internal" - check EC mode, rate of DUs
    "show lacp/pagp neighbor" - check EC mode, rate of DUs FOR THE NEIGHBOR

Advanced LACP features:
    "lacp rate fast" - Enable Fast LACPDUs (Slow ones are transmitted once in 30 seconds, fast ones are once every second). Have to be configured on both sides
    "port-channel min-links" - Set a minimum links at which the port-channel will be a port-channel. If there are 4, minimum is 4 and one link fails - port-channel is destroyed to usual links
    "port-channel max-bundle" - Set a maximum links available in PC. If there are 2 IFs in PC, you configure maximum to 1, then one interface will got to "Hot-Standby" mode.
    "show lacp sys-id / lacp system-priority" - Check / Specify LACP priority for the switch. Used if MIN or MAX links are configured. Switch LACP priority allows for device to specify which interface is chosen as an active and which one as HOT-STANDBY.
    "lacp port priority" - Specify PORT priority opposed to whole Switch priority. Also used to manually specify which interface should be turned to Hot-standby
    CHOICE GOES TO THE LEAST PRIORITY!


#                    IPv6

The IPv6 address is **128 bits** (i.e. 16 bytes) long and is written in **8 groups of 2 bytes** in hexadecimal numbers, separated by colons:
    fddd:f00d:cafe:0000:0000:0000:0000:0001

4bits = "nibble"
16bits = "word"

Leading zeros of each block can be omitted, the above address can be written like this:
    fddd:f00d:cafe:0:0:0:0:1

We can abbreviate whole blocks of zeros with `::` and write:
    fddd:f00d:cafe::1

However, this can only be done _once_ per address in order to void ambiguity:
    ff:0:0:0:1:0:0:1 (correct)
    ff::1:0:0:1 (correct)
    ff::1::1 (ambiguous, wrong)
shared from the neighbor
According to RFC 5952 `ff:0:0:0:1::1` is not correct either because the longest group of concurrent zeroes must be shortened.

IPv6 is disabled by default on Cisco devices, to enable: "ipv6 unicast-routing"

Protocols
| Number | Protocol  | Purpose                                                                                                 |
| ------ | --------- | ------------------------------------------------------------------------------------------------------- |
| 6      | TCP       | Stateful - Confirms if packets have arrived. Important for use cases with validation.                   |
| 17     | UDP       | Stateless - Does not confirm if packets have arrived. Good for streaming applications, VoIP calls, etc. |
| 58     | IPv6-ICMP | Information, Error reporting, diagnostics based use cases.                                              |

Methods to Assign IPv6 Addresses
    Static - Fixed Address.  
    SLAAC - Stateless Address Auto-Configuration (Address generated by Host).  
        EUI-64 - Dynamic Interface-ID Assignment. It takes a MAC-address on the NIC (48 bit value), cuts in half and adds "FF:FE" inbetween. BUT:
                 Additionally to above, MAC-address's 7th bit gets flipped while doing that. Example: let's take MAC-address 0694:1023:a123,
                 Applying EUI-64 would make the interface check the bit value of "06" (first byte) - 00000110 and flip 7th bit (1 -> 0) to 00000100.
                 That would change value to 0494:1023:a123 and Interface-ID would be "0494:10FF:FE23:A123" 
    DHCPv6 - Dynamic Host Configuration Protocol (Address assigned by a central DHCP server).

Scopes and Special Addresses
    GLOBAL - Everything (i.e. the whole internet),  
    UNIQUE LOCAL - Everything in our LAN (behind the internet gateway),  
    LINK LOCAL - Everything within the same collision domain that will not be routed (i.e. attached to the same switch).

    | Range     | Purpose                          |
    | --------- | -------------------------------- |
    | ::1/128   | Loopback Address (localhost)     |
    | ::/128    | Unspecified Address              |
    | 2000::/3  | GLOBAL Unicast (Internet)        | /3 mask means than first 3 bits are fixed for the IPv6-address. (2 = 0010 therefore 001 doesn't change for IP-addresses). Like that, any IP starting with 2 or 3 is in range.
    | fc00::/7  | Unique-Local (LAN)               | /7 mask equals to addresses starting from either fc00 or fd00 (first 7 bits cannot change, 8th bit is responsible for decision if it's C or D)
    | fe80::/10 | Link-Local Unicast (Same switch) | /10 mask equals to addresses starting from one of: fe80, fe90, fea0, feb0.
    | ff00::/8  | Multicast addresses              | ff00 - ffff

"%5" in the end of IPv6-address specifies the interface to which the IPv6-address is assigned.

Unique-local was previously named "Site-local"

Should always use the smallest possible scope for communication.  
A host can have multiple addresses in different scopes, even on the same interface.

Subnetting
As in IPv4, IPv6 includes support for network segmentation via Subnetting. 

The following address:
`2003:1000:1000:1600:1234::1` formatted fully as `2003:1000:1000:1600:1234:0000:0000:0001`, consists of the following segments:
- `2003:1000:1000:1600` - Prefix (Combined of Routing Prefix and Subnet ID)
- `2003:1000:1000` - Routing Prefix / Network Address
- `1600` - Subnet ID / Subnet
- `1234::1` - Interface-ID

First 64 bit is a "Prefix"
Second 64 bit is an "Interface-ID"

If my ISP provider delegated a portion of the prefix to me (e.g. `2003:1000:1000:1600/56`), then I could use the subnets `1600` through to `16FF` for my own purposes.

Multicast
Communication from one node to another is called Unicast Communication from one node to many is called Multicast.
The following IPv6 multicast addresses may be used in in the link-local scope:

All IPv6 Multicast routing protocol addresses start with FF02:
| Range     | Purpose                                |
| --------- | -------------------------------------- |
| ff02::1   | All Nodes within the network segment   |
| ff02::2   | All Routers within the network segment |
| ff02::A   | All EIGRP Routers                      | On the link address - applies only for link-local
| ff02::5   | All OSPF Routers                       | On the link address - applies only for link-local
| ff02::5   | All OSPF DR/BDR Routers                | On the link address - applies only for link-local
| ff02::fb  | mDNSv6                                 |
| ff02::1:2 | All DHCP Servers and Agents            |
| ff02::101 | All NTP Servers                        |

In Multicast addresses, FF is unchanged. But further 2 nibbles (nibble is 4bit) can have different values.
Last nibble of FF00 is a SCOPE of Multicast. 0 and 1 are local (packet doesn't leave NIC). 2 is on-the-link, link-local.
3rd nibble of FF00 is either Temporary (value 0) or Permament (value 1) address

A full list is maintained by [IANA]
You can actually ping these addresses, e.g. `ping ff02::1`

ICMP Message Types
ICMP does not use ports in order to communicate, but rather types. Critical/important types have numbers ranging from 1-127, while informational types have the numbers 128 and above. Each type can have subtypes or rather codes that can be used for further specifications.  

Here are some frequently used IPv6 ICMP types:

| Type | Code | Purpose                        |
| ---- | ---- | ------------------------------ |
| 0    |      | Reserved                       |
| 1    |      | Destination Unreachable        |
| 1    | 0    | No Route to Destination        |
| 1    | 2    | Beyond Scope of Source Address |
| 3    |      | Time Exceeded                  |
| 3    | 0    | Hop Limit Exceeded in Transit  |

| Type | Code | Purpose                   |
| ---- | ---- | ------------------------- |
| 128  | 0    | Echo Request ("ping")     |
| 129  | 0    | Echo Reply                |
| 133  | 0    | Router Solicitation       |
| 134  | 0    | Router Advertisement      |
| 135  | 0    | Neighbo(u)r Solicitation  |
| 136  | 0    | Neighbo(u)r Advertisement |

A full list is maintained by [IANA](https://www.iana.org/assignments/icmpv6-parameters/icmpv6-parameters.xhtml)

DHCPv6
IPv6 addresses can be distributed using the IPv6 version of the Dynamic Host Configuration Protocol (DHCPv6). If a host wishes to obtain an IPv6 address via DHCPv6, it sends out a **DHCP Solicitation** from UDP port 546 to port 547 on the DHCP multicast address `ff02::1:2`. The DHCP server then replies to the client (from UDP port 547 to UDP port 546) with **DHCP Advertisement**. This handshake can be completed by the client sending out a **DHCP Request** and the server responding with a **DHCP Reply**
The DHCPv6 protocol is explained in more detail in this [Wikipedia Article](https://en.wikipedia.org/wiki/DHCPv6)

DHCPv6 vs. SLAAC
Depending on how the router and the client are set up, the client can (and will) use both mechanisms (i.e. SLAAC and DHCP) to acquire IPv6 address allocations. The following table highlights the possible configuration combinations:



#                    Routing 

AD:
    Directly connected - 0
    Static             - 1   (can be manually set to any number)
    EBGP               - 20
    IntEIGRP           - 90  (internally learned routes)
    OSPF               - 110
    IS-IS              - 115
    RIPv2              - 120
    ExtEIGRP           - 170 (externally learned routes)
    IBGP               - 200

Dynamic routing protocols:
    IGP (Interior Gateway Protocol)
        RIP;   (for updates and communication: IP protocol 17 - UDP, port 520)
        EIGRP; (for updates and communication: IP protocol 88)
        OSPF;  (for updates and communication: IP protocol 89)
        IS-IS; (for updates and communication: IP protocol 124)
    EGP (Exterior Gateway Protocol)
        BGP.   (for updates and communication: IP protocol 6 - TCP, port 179)

Autonomous System - a set of routers and networks under same management
    IGP is best suited to share routes within same AS.
        - better speed
        - better responsiveness
    EGP is better suited to share routes between AS'es
        - better stability
        - better security

Link State Routing protocols (OSPF, IS-IS). 
    (*)Router knows the whole topology
    Takes more RAM / CPU
    Faster convergence     
Distance Vector Routing protocols (RIP, EIGRP)
    (*)Router knows only about the next hop
    Takes less RAM / CPU
    Slower convergence

    EIGRP - an Advanced distance vector protocol
        EIGRP can work with just the update from the routers. Not all routes have to be advertised
        EIGRP establishes the neighborships (usually no neighborship in distance vector)

To determine the best route, routers check:
    1. Route mask (if there's a route to /25, it is always preferred over /24, /23 etc. route)
    2. Administrative Distance (lower is better)
    3. Metric (lower is better)
    #If all 3 things above are identical, router will load-balance traffic to the subnet using ECMP (Equal Cost Multi-Path)



#                    OSPF (AD - 110)  (Mcast - 224.0.0.5 / FF02::5) (Mcast DR/BDR - 224.0.0.6 / FF02::6, TTL=1) (IP protocol - 89)
Practical OSPF series (https://www.youtube.com/playlist?list=PLIFyRwBY_4bSkwy0-im5ERL-_CeBxEdx3)

Adjacency established if following matches for neighbors:
   +Hello timer
   +Dead timer
   +Area ID
   +Area Type
   +Authentication parameters
   +Subnet on the link
   +MTU
----Link network type DOESN'T have to match to establish the adjacency.

OSPF Tables
    Neighbor Table - a table of every directly connected router (participating in OSPF) 
    Topology Table - a table of everything that is known to OSPF (known networks, ALL routers in OSPF)
        Link State Database (LSDB), same as Topology table - consists of LSAs as entries
        Link State Advertisement (LSA) - a piece of information shared between routers in OSPF process
    Routing Table - table of all the best routes known to the router


OSPF Packets
    Hello packets - packets that each router with OSPF process enabled are sending out to 224.0.0.5 multicast-address. Includes information about router (helps to determine if adjacency should be formed)
                    each router with OSPF enabled in it's turn is listening for packets on IP 224.0.0.5
                    Used to discover other routers with OSPF. 
                    Multicast packets are local to a link segment, they are non-routable.
    Database Descriptor (DBD) - a packet with summary of LSAs in router's LSDB (Topology table). Usually LSDB contains all LSAs separately, DBD has it summarized
    Link State Request (LSR) - a request for certain LSAs (unknown to receiving router) after received DBD with summarized LSAs from other router
                         LSR - basically a request to router that provided DBD (with summarized LSAs) to expand some of LSAs to be able to insert them into the topology
    Link State Update (LSU) - a reply with LSA requested through LSR
                              OR a global OSPF update with newly discovered subnet
    L.S. Acknowledgement (LSAck) - confirmation packet for neighbor routers to make sure LSU have been received.


OSPF Areas
    Areas in OSPF form a 2-tier hierarchy. 
    That means that there is a central "Area 0" (known as Backbone area).
    And also there can be other areas (number 1 to 2^32). Each non-Backbone area have to be connected to Area 0. And all communication between areas can only happen through Area 0.

    Each area has it's own LSDB (and therefore set of LSAs). That allows to exchange only relevant information between routers from different Areas.
    For example: R1 and R2 are connected to R3 which is connected to R4 and R5. R4 and R5 are interconnected. R4 has a 10.0.133.16/28 subnet on it.
    If link from R4 to R3 is dropped, R3 will see route to 10.0.133.16/28 through R5. And since it's OSPF (every route have to know all the topology), will send out the Link-State Update (LSU) with new information to every router upwards.
    But at the same time, R1 and R2 know the subnet 10.0.133.16/28 is located through R3. And it still is (even after LSU). That update is redundant since nothing changed logically for R1 and R2.
    Areas having own LSDB help not to receive all of such updates.

    Area Types
            Normal - a basic area that allows all routes to be inserted into itself (routes from all areas or from other routing protocols distributed within OSPF process)
            Stub - an area that allows only routes from OSPF process and summarizes all the routes received from external routing protocols (serves as a default-gateway to reach a lot of outer prefixes)
                   Stub is an area with nothing behind it and only Area 0 in front.
            NSSA (Not-so-stubby-area) - a stub area that STILL doesn't allow redistributed (from other routing protocols) routes in common but allows redistributed routes from local area
                                        Basically NSSA is a stub area but there are some other subnets behind it (some other routing protocol)

OSPF router types
    Internal Router - A router with all the interfaces in same area (any area)
    Backbone Router - A router at least 1 interface in Area 0
    ABR (Area Border Router) - A router with it's interfaces in Area 0 and some other Area. Stores LSDB for all areas it connects to (different LSDB for each Area). Summarized LSAs between areas.
    ASBR (Autonomous System Border Router) - A router with it's interfaces in some Area and some interfaces out of OSPF process (distributing networks from other routing protocol/directly connected ones or static).
        Router is ASBR if it's in OSPF process AND redistributing external routes into an OSPF process.

Hello Packet (common information above)
("(*)" - value have to be matched for routers to establish the adjacency)
    Each OSPF router is listening to OSPF traffic on 224.0.0.5 IP (Multicast) and sends out Hello packets to that same address. Can be configured to send the Hello's as Unicast if Multicast is restricted. To use unicast, other OSPF Router IPs have to be specified in the OSPF configuration.
    Information transferred in Hello's:
        Router ID - 32 bit value (like 1.1.1.1), an identificator
     (*)Hello Interval(10sec default) - a timer mentioning the frequency of sending Hello Packets (how often hellos are sent)
     (*)Dead Interval(40sec default) - timer specifying when the neighbour is considered unreachable ("dead") and is out of neighbor table
        Neighbors - a list of known neighbours on the link (initially empty)
     (*)Area ID - an ID of area that router's interface belong to
     (*)Authentication data - a password that should be used to establish a connectivity to other router (other router should be configured with the same password)
     (*)Network Mask - subnet mask for the link
     (*)Area Type
        Designated Router (DR) - an IP address of DR
                            DR - A router responsible for link-state Updates (LSUs) on the link with multiple routers. Used to prevent all the routers from spamming all the updates. If update is originated from other router (not BDR) on the link it is getting sent only to the DR
        Backup Designated Router (BDR) - A backup DR. Takes over if something happens to DR
        Priority - a priority in DR election (more priority - more chances to be chosen as DR/BDR)
 
OSPF Neighbor adjacency states:
    DOWN - when router gets configured with the OSPF process and there are no other routers it can establish neighborship to. Adjacency is never displayed as DOWN (technically DOWN is not a state) because whenever OSPF is enabled, Hello packets are being sent but no device replies and initial router doesn't know anyones Router ID. Neighbor is added to Neighbor table only when Router know the Router ID (when some other device replies with Hello or sends it out first)
    ATTEMPT - a state available only in non-broadcast multi-access (NBMA) network where multicast/broadcast traffic is prohibited (serves as a replacement for DOWN state in NBMA network). ATTEMPT is an OSPF state of the router explicitly configured as OSPF neighbor to which connection is attempted to be established.
    INIT - a state that happens when router receives a Hello packet on the link when being in DOWN state. All the outbound Hello packets in this state will start to include neighbor ID in Neighbors attribute
    2-WAY - a state that specifies that there is a 2-way connectivity between devices, both routers sent out Hello packets and received them from each other with the neighbor's Router ID in Neighbors attribute. Main point is that router if router is in 2-WAY state, it received a Hello packet from neighbor and saw own Router ID. DR/BDR election also happen on that stage.
    EXSTART - after 2-WAY state, routers inspect received Hello packets and if all of them have matching "hello interval", "dead interval", "area ID", "area type", "authentication data" and "subnet mask on link" they move to EXSTART state. In that state, they decide who will be a Master and who'll be Slave on that adjacency. It is done by exchanging the DBD (both routers send DBD saying they're the Master). Router with the higher Router ID is elected as Master
            Master-Slave (M-S) relationship doesn't relate in any way to DR/BDR. M-S election occurs for every pair of neighbors. It serves a purpose of an acknowledgement system. Master sets the initial seq number and responsible for incrementing the seq number.
    EXCHANGE - state happening after Master/Slave election has been completed. Router elected as Slave transition from EXSTART to EXCHANGE and sends out DBD confirmation that it's a Slave. Master after that transition to EXCHANGE as well. After that With Slave DBD confirmation, Slave also sends it's LSDB. Master sends out a DBD with it's own LSDB as a Master. Slave answers with an empty DBD for that. In case Slave would like to say something, it adds a bit to empty DBD.
    LOADING - Routers transition into loading state after both of them have successfully exchanged LSDBs. In that state, peers know which LSA are in their neighbors LSDBs and can send out an LSR to specify some of summarized LSAs (LSRequest -> LSUpdate -> LSAck process). 
    FULL - Router transition the neighbor to FULL state after it receives LSU from peer and sent a LSAck. It thinks that now topology table is full since it got all the required information.

________________________________________________OSPF ROUTER STATES
Possible packet exchange (with state changes)
________________________________________________
R1(DOWN,OSPF_ON)     --- R2(DOWN,OSPF_OFF)
-Hello->
                         //turn on OSPF on R2

R1(DOWN,OSPF_ON)     --- R2(DOWN,OSPF_ON)
                     <-Hello-

R1(INIT)             --- R2(DOWN)
-Hello,Neighbor:R2->

R1(INIT)             --- R2(2-WAY)
                     <-Hello,Neighbor:R1-

R1(2-WAY)            --- R2(2-WAY)
//if 6 parameters match, transition to EXSTART

R1(EXSTART)          --- R2(EXSTART)
-DBD,Master->       <-DBD,Master-
-DBD,Slave+LSDB->

R1(EXCHANGE)         --- R2(EXCHANGE)
                     <-DBD,Master+LSDB-
-DBD,Slave->                               //if Slave wants to say something after that, a bit is set in that empty DBD respose           
//after LSDB exchange, transition to LOADING

R1(LOADING)         --- R2(LOADING)
-LSR->
                    <-LSU-
-LSAck->

R1(FULL)         --- R2(LOADING)
                 <-LSR-
-LSU->
                 <-LSAck-

R1(FULL)         --- R2(FULL)
________________________________________________OSPF ROUTER STATES

DR and BDR election metrics:
    1. Highest Priority number (can be set manually 0-255). If priority is 0 - interface will ALWAYS be DROTHER
    2. Highest Router-ID
    3. Highest IP-address on Interface

    In topology of any number of routers, each with same priority, DR/BDR election depends exclusively on which router is enabled first. It can be the one with RouterID 1.1.1.1 and it still will be the DR. Whenever any other router (RouterID 2.2.2.2 for example) gets up, it'll see that R1 is DR and won't be able to do anything apart from becoming BDR.
 (!)There is no Preempt for DR/BDR election in OSPF

    DR/BDR Election happens on 2-WAY state of neighborship.

    Topology of 4 routers. R1, R2, R3 and R4. Each has priority 1. RIDs are 1.1.1.1, 2.2.2.2, 3.3.3.3 and 4.4.4.4. All are disabled.
    1. R1 gets enabled first. Starts to send out Hello packets with own RID, priority, DR and BDR IPs. Since there's yet no DR/BDR, it's 0.0.0.0
    2. After WAIT timer is over, R1 elects itself as DR. All Hellos after that include DR value of R1 interface 10.0.0.1
    3. R2 gets enabled second. Starts to send out Hellos as R1 initially did with R2 values (DR/BDR still 0.0.0.0)
    4. R2 receives a Hello from R1 and it's DR value changes to R1 IP. Since there's no preemt, R2 sets it's own IP as BDR. Hellos now include DR with R1 IP (10.0.0.1) and BDR with R2 IP (10.0.0.2)
    5. R3 and R4 gets enabled after that. Once they receive Hello from R1 or R2, their DR/BDR in Hello will change to new values - DR with R1 IP (10.0.0.1) and BDR with R2 IP (10.0.0.2)
    6. R3 and R4 will become DROTHER.
    7. In OSPF, DR and BDR will establish FULL adjacency with each other router on the link. And will exchange LSDB
    8. DROTHERs on the other hand will only establish FULL adjacency with DR/BDR. DROTHER to DROTHER adjacency will remain 2-WAY
    9. DROTHERs doesn't need to exchange LSDBs to each other since there's a DR/BDR who know all the required information and share it
    10. "R1#show ip ospf neighbor"  will show FULL for every other router
        "R2#show ip ospf neighbor"  will show FULL for every other router
        "R3#show ip ospf neighbor"  will show FULL for R1/R2 (DR/BDR) and 2-WAY for R4 (DROTHER)
        "R4#show ip ospf neighbor"  will show FULL for R1/R2 (DR/BDR) and 2-WAY for R3 (DROTHER)

    WAIT timer (same duration as dead interval, 40 sec default) - If OSPF router does not receive any responce to it's Hello packets for duration of wait timer, it will set itself as a DR in next Hello advertisements (0.0.0.0 before the wait timer passes)

    DR/BDR/DROTHER concept is working on per interface basis. NOT ON PER ROUTER. 
    If some router is considered DR - that only means that it's reachable interface is DR. Router can have another interface which will be in DROTHER state at the same time.

Routing update propagation scenarios (Broadcast network)
    DR/BDR/DROTHER roles of routers. At some time any of those can receive a network update.
    Below are scenarios where DR, BDR or DROTHER receives update and how is it propagated:

    1. (DR). Update comes to DR
    1.1. DR sends out an LSU to MCAST IP 224.0.0.5 (all OSPF routers)
    1.2. BDR sends out an LSAck to 224.0.0.5 after receiving LSU from DR
    1.3. DROTHERs send out an LSAck to 224.0.0.6 (DR/BDR routers)

    2. (BDR). Update comes to BDR
    2.1. BDR sends out an LSU to MCAST IP 224.0.0.5 (all OSPF routers)
    2.2. DR sends out an LSAck to 224.0.0.5 after receiving LSU from BDR
    2.3. DROTHERs send out an LSAck to 224.0.0.6 (DR/BDR routers)

    3. (DROTHER). Update comes to DROTHER
    3.1. DROTHER sends out an LSU to MCAST IP 224.0.0.6 (DR/BDR routers)
    3.2. DR sends out an LSU to 224.0.0.5 (all OSPF routers) after receiving LSU from DROTHER
    3.3. BDR sends out an LSAck to 224.0.0.5 after receiving LSU from DR
    3.4. DROTHERs send out an LSAck to 224.0.0.6 (DR/BDR routers)

OSPF Costs (also called Delay)
    Since each router in OSPF process know the full topology, they also know the path cost to reach the remote destination.
    Cost is calculated based on the link speeds between routers on the way to destination.
    OSPF cost is called DELAY - a metric based on link speed.
    Delay is calculated on the formula - Reference bandwidth / Link bandwidth.
        Reference bandwidth in OSPF is 100mbps. So a 10mbps link has a delay (metric) of 10 - 100/10
        100mbps link is 1 - 100/100.
    Reference bandwidth can be increased/decreased.    
    OSPF cost value is 16 bit. It can have values 0-65535
    Fractional Delay values are rounded up to lower integer value.

    There are different issues with changing the OSPF reference bandwidth:
        If it is left as is (100mbps), links with values of 100mbps, 1gbps, 10gbps and higher will ALL have the delay of 1 (minimum value) and will just be the same for OSPF process.
        If it is set to really high values (1tbps or more), low speed links (1mbps, 10mbps, 100mbps) will ALL have the delay of 65535 (maximum value).

OSPF Routing tweaks
    It's possible to change the AD of
        All routes in OSPF on a certain router - navigate to router ospf command and then: "distance <number>" (number is 0-255)
        All routes in OSPF received from certain router in the area - router ospf and then: "distance <number> <neighbor-RID> <RID-wildcard>" (number is 0-255, neighbor-RID is a neighbor ID to which the change should be applied, RID-wildcard is a mask to the RID)
    It's also possible to change to cost of a certain interface
        Navigate to interface - interface GI0/1 and then: "ip ospf cost <number>" (number is a manual cost value)

OSPF LSA types (Only LSA5 is shared between areas, other types stay in the area)
(all the below LSAs information is shared ONLY for links/prefixes that actually participate in OSPF process) - link is added to OSPF/prefix is shared in OSPF
    LSA type 1 (Router LSA)   - Router LSA is exchanged exclusively within the area of it's origination. Router LSA is sent by every router. Router LSA sent by Router consists of the information about IP Networks / Masks / Costs on each of that Routers link. Used to build topology map of the area. 
    LSA type 2 (Network LSA)  - Network LSA is exchanged exclusively within the area of it's origination. Network LSA is sent only by DR router elected on the multi-access link. Network LSA consist of the same information as LSA type 1, summarized for all the routers on the multi-access link.
    LSA type 3 (Summary LSA)  - Summary LSA is exchanged between Areas. Summary LSAs are sent only by ABRs in both directions. Summary LSAs summarize LSA1, LSA2 and LSA3. Summary LSAs are exchanged to share the routes existing in every area to all other areas. For each route in area, a single Summary LSA is sent from ABR (if area 1 has 30 prefixes, 30 type 3 LSAs will be sent to area 0, for updates from area 0 to area 2, there will be 30 prefixes(area 1) and 20 additional prefixes (area 0), that will sum to 50 total prefixes which is 50 total Summary LSAs).
    LSA type 5 (External LSA) - External LSA is exchanged from ASBR to all areas (area it's part of, then area 0, then all remote areas that exist in OSPF process). External LSA exchange prefixes originated in remote routing processes outside of OSPF and is forwarded unchanged thoughout OSPF domain. External LSA specify that "to reach certain network, go to ASBR". Since initially ASBR only shares LSA1 and LSA2 wihin the area, area 0 and other areas know nothing about it. For LSA5 to work, LSA4 is required:
        LSA type 4 (ASBR-Summary LSA) - ASBR-Summary LSA is exchanged with all the areas. Sent by ABR when ASBR is in foreign area. LSA4 allows all the routers in all the areas to know where to look for ASBR. ABR send out "to reach ASBR, go to myself".
    LSA type 7 (NSSA external LSA) - originated in NSSA ONLY - instead of LSA5 (LSA4 and LSA5 prohibited in NSSA or Stub Areas). Shared as LSA5 further out of the Area. Each LSA7 is shared for each Prefix received from other routing protocols
  Next are much less used LSAs. 
    LSA type 6 (Multicast routing LSA) - Not implemented in Cisco. PIM is used for the cause.
    LSA type 8 (External attribute for BGP)
  There also are Opaque LSAs - LSAs that carry non-OSPF information over OSPF domain (usually some attributes from other routing protocols shared via ASBRs)
  Usually Opaque LSAs are used to pass some routing attributes from ASBR in one area to ASBR in another area (since both these ASBRs have some external subnets available, these could be different BPG sessions)
    LSA type 9 (Local Link Opaque LSA) - information is shared only till the next router on a link. Information such as BGP attributes
    LSA type 10 (Local Area Opaque LSA) - information is shared only within local Area (like LSA1, LSA2)
    LSA type 11 (OSPF Domain Opaque LSA) - information is shared within the whole OSPF Domain.

OSPF LSA types deep dive:
    LSA1 - Router LSA.  
    LSA2 - Network LSA
        In LSA1, Router is identifying itself, it's OSPF attached links and it's costs. There are 4 different options each link can be connected to:
        1. "Point-to-Point to another router". Connection to another OSPF router on a P2P link.
        2. "Transit Network". DR for Multi-Access link with OSPF Neighbors
        3. "Stub Network". Includes network ID, Masks and cost of directly connected network
        (not used or very rare) 4. "Virtual Link". Used to build a virtual connectivity between Area0 ABR and some ABR between two NON-0 areas connected together to form a virtual connectivity making Area 0 think that faraway ABR is actually connected to it.
        
        There can be 6 scenarios:
            (in every scenario, nothing happens before OSPF is configured and interface is not added to OSPF process or network is not defined in it)
        Basic Loopback (LOOPBACK interface type by default) - create loopback and add to OSPF process, then it is added into LSDB and Router LSA is formed to reflect the loopback link that is now participating into the OSPF process as a "Stub network". Router ID, Mask (always /32 for LOOPBACK type interface) and cost is displayed in LSDB
        Loopback set to Point-to-Point interface type manually - same as scenario above, but looback is manually reconfigured to be taken as P2P interface. Same outcome - "Stub Network" but with mask, that you specified on the loopback link.
        Serial interface (P2P interface type by default) with no neighbors - add serial to OSPF process, then it is added into LSDB and Router LSA is formed. Same as 2nd scenario, "Stub Network" with all same details
        Serial interface (P2P interface type by default) with neighbor - add serial on neighbor router to OSPF process, then the neighborship is formed between two routers over serial links. In such option, every router send out an LSA1 with 2 links. "Stub network" with Network ID, Mask and Cost. Also "Point-to-point to another router" link which specifies the Neighbor Router ID, Link IP address and interface cost.
        Ethernet interface (BROADCAST interface type by default) with no neighbors - Interface type is BROADCAST but all in all, it's same as scenario 3.
        Ethernet interface (BROADCAST interface type by default) with neighbor - Once two routers connected over ethernet link form neighborship, a "Transit network" type LSA1 is formed. It specifies that all the information (attached links, mask, costs) is contained in LSA2 (Network LSA). "Transit network" type shared by DR.
            "Transit network" type includes the IP address of DR and cost. Since the DR is responsible for sharing the LSA2

    LSA3 - Summary LSA
        In routing table, routes received by LSA3 are displayed as "O IA" (OSPF Inter-area) 
        LSA3 are used to advertise the prefixes between OSPF areas. Both ways (to Area 0 and away from Area 0).
        Every prefix received from foreign area by area 0 is redistributed further to other areas. Added to LSDB with ABR Router-ID (ABR between area 0 and area which received the prefix)
        ABR REgenerates an LSA3 from other LSA3's received from other areas.        
        
    LSA5 (External LSA) usually is originated with the LSA4 (ASBR summary LSA) 
        LSA4 & LSA5 are originated only if there is an ASBR in OSPF topology.
        LSA5 - update with external subnet, mask an metric
        LSA4 - update with the router ID that originated LSA5 and how to reach it (through ABR that made an advertisement)
        Router in OSPF becomes an ASBR if it starts to redistribute routes not participating in the current OSPF process.
            It can be EIGRP, RIP, BGP, Static routes, directly connected routes, even routes from other OSPF process.
        After redistribution starts, ASBR begin to generate (if there is something to redistribute, if there is no available routes that are NOT already in OSPF - no LSA5 will be originated, only LSA4) LSAs5.
        In LSDB, same area as ASBR, only LSA5 are visible with the subnet, mask and metric to reach the network.                
        Areas other than ASBR area receive IDENTICAL LSA5 (with router-id from different area even though there is no information about that router for other areas)
            In addition other areas receive LSA4 from ABRs with information on how to reach the ASBR. Always reachable through the ABR

OSPF Network types
    Point-to-point - a network with just two routers connected through a single link (for example serial is defaulted to this type)
    Broadcast - a network (basically an ethernet connectivity to switch for router) with multiple OSPF routers available on a single link
    NBMA (non-broadcast-multi-access) - a network with multiple devices where broadcast/multicast traffic is prohibited.
    Point-to-Multipoint - basically a hub-and-spoke OSPF network. There is a single OSPF router that has a neighborship to few other OSPF routers which cannot be interconnected to each other
    Point-to-Multipoint non-broadcast - point-to-multipoint network with broadcast/multicast traffic prohibited

    P2P - Max 2 routers. Full mesh. No DR election. 10/40 timers. Auto neighbor discovery. Discovery, hellos, neighbor communication and LSAs are sent to 224.0.0.5 because there is no other routers and such packets won't reach any other OSPF router. Next-hop for prefixes is a peer
    Broadcast - inf amount of routers. Full mesh. DR/BDR election. 10/40 timers. Auto neighbor discovery. Discovery and hellos sent to 224.0.0.5. Neighbor communication packets are sent as unicast by each router to every other router. LSAs are sent out to DR/BDR and as multicast further on. Next hop for prefix is originating router.
    NBMA - inf amount of routers. Full mesh. DR/BDR election. 30/120 timers. Since multicast is prohibited, no neighbor discovery - every communication between routers is configured as unicast. Discovery and hellos sent to Neighbor IP. Neighbor communication packets are sent as unicast by each router to every other router. LSAs are sent out to DR/BDR as unicast and then DR/BDR imitates the multicast traffic - sends out all updates as unicast to every router. Next hop for prefix is originating router.
    P2Multipeer - inf amount of routers. No full mesh, spokes only connect to hub, hub connects to all spokes. No DR/BDR election since hub is kind of considered as a DR at all times. 30/120 timers. Neighbors discovers automatically using 224.0.0.5. Discovery and hellos sent to 224.0.0.5. Neighbor communication is unicast router-to-router. LSAs are sent as unicast from hub to spokes and from spokes to hub. Next-hop IP for spokes reaching for other spoke subnet is an ip of a Hub.
    P2Multipeer-NB - inf amount of routers. No full mesh, spokes only connect to hub, hub connects to all spokes. No DR/BDR election since hub is kind of considered as a DR at all times. 30/120 timers. Since multicast is prohibited, no neighbor discovery - every communication between routers is configured as unicast. Discovery and hellos sent to Neighbor IP. Neighbor communication packets are sent as unicast by each router to every other router. LSAs are sent as unicast.  Next-hop IP for spokes reaching for other spoke subnet is an ip of a Hub.

OSPF Area Types
    There are 5 different area types. Their difference is based on the allowance or disallowance of certain types of LSA. Also some have default route via ABR
    Normal Area - an area that allows all 5 LSA types. When area is created is is defaulted to normal.
        Normal area receives all LSA3 from other areas, all LSA5 as well.
    Stub Area - an area that allows only LSA1, LSA2 and LSA3. It receives all LSA3 from other Areas. Additionally in Stub Area, ABR distributes default route into the Area as an LSA3.
    Totally Stub Area - same as Stub, but LSA3 is prohibited apart from a single 0.0.0.0/0 LSA3 shared from it's ABR. 
            to configure NSSA - "area * stub" on area's ABR ONLY
        Stub config options for ABR:
            "no-summary" (create Totally Stub) - ABR shares a 0/0 route into Stub Area as a LSA3 (other LSA3s are prohibited)
            "no-ext-capability" - opaque LSAs are prohibited
    NSSA (Not So Stubby Area) - NSSA is like a Stub Area but with ASBR which redistributes routes from other routing protocols. LSA1, LSA2 and LSA3 are allowed in an area. LSA4 and LSA5 are prohibited. 
        Routes from different routing protocols are shared within the Area as Type 7 LSAs (LSA7) - these are basically LSA5 but within NSSA. 
        Possible issue: Type 5 LSAs from other NSSAs are not shared into the area so external routes from other areas should be reachable by some other means.
        There's a setting "default-information-originate" which allows for ABR to share a default route as LSA7 into the NSSA area
    Totally NSSA - Same as NSSA but type 3 LSAs are also prohibited and ABR shares a single 0.0.0.0/0 LSA3 as a default route. That remediates the possible issue of NSSA.  
            to configure NSSA - "area * nssa" on ALL routers in an area
        NSSA config options for ABR:
            "default-information-originate" - ABR shares a 0/0 route into NSSA as a LSA7
            "no-summary" (create Totally NSSA) - ABR shares a 0/0 route into NSSA as a LSA3 (other LSA3s are prohibited)
            "no-ext-capability" - opaque LSAs are prohibited
            "no-redistribution" - if ABR is also ASBR, externally redistributed routes are not shared into NSSA (since one of first two commands are probably already configured) but shared into other areas

OSPF Route codes
    "O" - intra-area route (LSA1,LSA2)
    "O IA" - inter-area route (LSA3)
    "O E2/E1" - external route redistributed into OSPF (E2) with default ASBR to external routing protocol default metric, for (E1) metric is different from default and is summarized on the whole way to the external routing Autonomous System
    "O N2/N1" - external route redistributed into OSPF in NSSA Area (N2) with default ASBR to external routing protocol default metric, for (N1) metric is different from default and is summarized on the whole way to the external routing Autonomous System

OSPF Passive interface
    Passive interface Feature is used to prevent OSPF Router from sending Hello packets off a certain interface
        That allows certain interfaces to share into OSPF process the access subnets located on the interface but not to send Hellos in their direction (since there's probably only endpoints with no routers)
    Routers can be configured with passive interface default - all interfaces are passive by default. Useful if there are a lot of access subnets.

OSPF Authentication (can be configured both in ROUTER OSPF mode and on the INTERFACE --- INTERFACE config is prevalent)
    Simple password authentication - a clear text password auth. Routers send password in clear text. NOT SECURE AT ALL. Like using telnet instead of SSH. Easily compromisable.
        To configure: 
            1. Specify password on the interface "ip ospf authentication-key *****"
            2. Turn on authentication either in ospf process under "area * authentication" (for whole area) command or under interface configuration
    Hash-based authentication
            Done by hashing the authentication password. Hashing is when hashing algorithm takes something (phrase or Packet) and turns it into something else by representing it in some other way.
            Example of hashing: we can take word "hello". It can be represented as 8+5+12+12+15 (number of each letter in the alphabet) = 52. So "hello" = 52. Value 52 in that case is called "Digest"
            And then by taking number 52, we cannot reverse-engineer it into "hello", because there are a lot of other sequences that can give the value of 52. 
        How it works: 
            0. MD5 authentication have to be configured on the OSPF process, some password set up.
            1. Before sending any OSPF packet (Hello, LSA, DBD, LSU, LSR, LSAck), Router calculates the Hash Digest of a packet+set up password (calculate: digest of packet+password)
            2. Then the packet is sent to neighbors but only with the calculated hash digest (send: packet+digest), no password is sent
            3. Neighbors then check the digest and calculate their own digest of packet+password (password should be configured on all OSPF routers communicating with authentication)
            4. If digest matches, packet is accepted.
            5. To further guard the communication, while MD5 authentication is configured, each router adds Sequence number to every packet and the hash key value. 
                (seq# is added to prevent man in the middle from sending same hello packet to neighbors over and over while compromising the sending router - that would allow adjacent routers to think that the router is OK, since correct hello packets are coming fine)
        To Configure:
            1. Set the password on the interface "ip ospf message-digest-key *** md5 *******"
            2. Turn on the authentication eigher under ospf process commands "area ** authentication message-digest" or under interface configuration "ip ospf authentication message-digest"
        Key rotation - a meachinsm allowing to change hashing key withour tearing down the OSPF adjacency. To do that, configure more that one md5 password entries (with different keys) on the interface where key have to be rotated.
            If more than one password (with different keys) is matched, routers will have adjacency established for each of keys and the old password can be safely deleted afterwards.
            OSPF router sends out number of packets equal to number of configured MD5 passwords.
    Key Chains - a technology allowing usage of more secure hashing algorithm (default is md5) like SHA1 (SHA1's digest is 160bit). In combination HMAC-SHA.
            MD5 can still be used in Key chains.
        To Configure:
            1. Create Key chain. "key chain <name>"
            2. Create a key in a key chain. "key <#>"
            3. Provide settings to created key. Required: "cryptographic-algorithm <alg>" (md5, hmac-sha-1, hmac-sha-256 etc.), "key-string <password>"
            4. (optional) key rotation can be configured in a form of when to send the key and when to accept: "sent-lifetime...", "accept-lifetime..."

OSPF algorithm - Dijkstra algorithm. Shortest Path First - Helps to build a route trees for every router to reach every other router with as less cost on a way as possible
    Cost in OSPF is OSPF_reference_bandwidth/link_bandwidth
    In Dijkstra algorithm, there are 2 lists, [visited_nodes] and [unvisited_nodes], 2nd list is filled with all routers before algorithm start. Every time a router is visited it is added to 1st list
    Whenever first node is selected, costs to connected destinations are added to the cost table. Node is added to visited
    Then the lowest cost node is chosen and all the costs to other nodes from it is summed up in a table and updated. Node is added to visited
    This is done to each node and when all of them are visited, result table is the end tree that will be used in the process activity. 

OSPF Summarization
    In OSPF, it is possible to summarize the information about networks shared through LSA3 (command is under router ospf: "area <#> range <IP-prefix> <Subnet-mask>" - # is an area number, IP-prefix is a prefix value to which to summarize, Subnet-mask is a summary mask)
    Example: few subnets like 10.0.0.0/24, 10.0.1.0/24, 10.0.100.0/24, 10.0.200.0/24, 10.0.250.0/24 etc. can be summarized to 10.0.0.0/16 with the command "area 0 range 10.0.0.0 255.255.0.0".
    Also external routes (shared by ASBR) can be summarized: under router ospf, command "summary-address <IP-prefix> <Subnet-mask>" (values are same as in ABR summarization)
    
OSPF Filtering
    In OSPF, advertised routes can be filtered to prevent them from being shared. There are different possibilities to do so:
        "Not-advertise" command - can be done during the area summarization command: "area <#> range <IP-prefix> <Subnet-mask> not-advertise".
                                  badly scalable and also cannot nicely filter out non-backbone area routes to other non-backbone area. It'll either be filtered out from whole OSPF process (backbone area as well) or won't be filtered at all
        Filter-list - used in conjunction with prefix-list. First of all prefix-list have to be specified with "deny" prefixes for those that should not be shared (don't forget "0.0.0.0/0 le 32")
                      After that, filter-list with created prefix-list should be applied under router ospf either for prefixes not to be shared OUT of specified area or IN to the specified area
                      "area <#> filter-list prefix <prefix-list> <in/out>" (IN is for routes to to not be accepted in area, OUT is for router to not leave the area)
        Distribute-list - used to filter the routes from getting into the router's routing table. Route will still be in LSDB, router will know about it but won't input into the routing table.
                          To do: under router ospf enter "distribute-list <ACL-number/prefix-list/route-map>". Have to specify the ACL or prefix-list or route-map before.
        Also routes distributed by ASBR can be filtered out from being spread in OSPF process.
        That can be done by redistributing the routes on ASBR with the route-map specified.
        First route-map sequence have to be created with some matching clauses and deny action - that will deny matched prefixes from redistribution.
        Command is under router ospf "redistribute <*> subnets route-map <RM-NAME>" (* is type of redistribution like static,eigrp,bgp etc.)

OSPFv3 for IPv6
    IPv6 version of OSPF differs from IPv4 version in some additional LSA types:
        LSA1,LSA2 also exist on the OSPFv3 but there's no more prefix information on the existing networks carried in the LSA information
        LSA9 (named Intra Area Prefix Link State) is a new LSA type responsible for carrying the subnet information shared within the area for interfaces participating in the OSPFv3 process. 1 LSA for each subnet
        LSA8 (named Link States) is also a new LSA type which is local in the OSPFv3 LSDB to every router in the process. Each router has it's own LSA8, one LSA8 for each of it's interfaces in OSPFv3 and one LSA8 for every connected to router interface of adjacent router.

        LSA3 in OSPFv3 is named "inter-area prefix"
        LSA4 in OSPFv3 is named "inter-area router"


#                    EIGRP (iEIGRP AD - 90, eEIGRP AD - 170) (Mcast - 224.0.0.10 / FF02::A, TTL=1) (IP protocol - 88)
EIGRP - Enhanced Interior Gateway Routing Protocol
    Proprietary but Cisco opened it to other vendors (around 2010). But vendors don't really introduced into their OS.

(!)Cannot be configured on the interface config level. Directly interfaces can only be used either for timers or with named EIGRP.
    Either granular network statements or a Passive-interface feature should be used to prevent unwanted announcements.

Instead of knowing all the information in the area like in OSPF, EIGRP routers only know the information from their direct neighbors and their direct perspective of some information.

Features:
-Dynamic Neighbor discovery
-RTP (reliable transport protocol) to confirm the advertisement have been received. All the updates are confirmed to originating router.
-DUAL (Diffusing update algorithm) - analogue to Dijkstra algorithm in OSPF. Used by EIGRP to make the routing decision

Adjacency will be established if following matches:
    -AS number
    -K-number values
    -Authentication configuration
    

EIGRP Tables:
    Neighbor Table - Table of available neighbors. "show ip eigrp neighbors" to display on device (+detail to see details like if the neighbor is STUB router)
    Interface Table - 
    Topology Table - 

EIGRP Timers (DON'T HAVE TO MATCH FOR ADJACENCY TO BE ESTABLISHED)
    Hello interval - 5 seconds (default). How often Hello packets are sent out.
    Hold (dead) timer - 15 seconds (default). Example: R1 send Hellos to R2. Hold timer is information advertised by R1 to R2 about how long should R2 wait for Hellos from R1 before considering it unavailable.
    #timers can be changed on the interface basis - "ip hello-interval eigrp <AS#>" and "ip hold-interval eigrp <AS#>"

EIGRP Metric (Mnemonic to remember - "Big Dogs Really Like Me")
    (default)  Bandwidth.
    (default)  Delay.
    (optional) Reliability.
    (optional) Load.
    (optional) MTU. - not in the formula, used as a last resort

    (!)Each router for each subnet DOESN'T RECEIVE THE METRIC VALUE from the neighbor (only the REPORTED DISTANCE - neighbors metric to reach subnet). It just receives the variable values and calculates the metric for itself (sum delay & minimum BW on the path)

    Metric main points in calculation.
        1. Worst BW in the path. For each subnet a neighbor route reports it's worst BW segment that he has on the path (it can include either his own link in the direction of subnet or link of other router on the path to it)
        2. Summary of delays on the path. Each segment between routers on the path to the subnet adds it's own delay value (microseconds)

    EIGRP metric formula = ( (K1*Bmin + (K2*Bmin)/(256-L) + K3*D) * K5/(K4+R) ) * 256              //   If K5 = 0, then K5/(K4+R) = 1 (THAT IS AN EXCEPTION)
                            Bmin = (10^7)/Least-Bandwidth
                                Least-Bandwidth - the lowest BW link on the path to the desired subnet. Each router on the path reports it's available BW to the subnet and the lowest becomes "Least-Bandwidth"
                            Delay (D) is in 10th of microseconds. Can be checked by "show interface" - it's specified in usc there. Divide this value by 10 and it'll be 10th of microseconds.
                                Delay is summed up for each link (router hop) on the path to the subnet. 
                            Reliability (R) - ?
                            Load (L) - ?
                            Default K values: 
                                K1 = 1 
                                K2 = 0 
                                K3 = 1 
                                K4 = 0 
                                K5 = 0

    (!)Simple metric formula with just defaults: Metric = 256 * (10,000,000/Least-Bandwidth + Delays_summary/10)  
        (Delays_summary is a value of delays as specified on devices, not in 10th of microseconds)

EIGRP Path Selectionethanalyzer local interface inband capture-filter "host 10.144.200.33" limit-captured-frames 100 write bootflash:test.pcap
    Reported Distance (RD) - a metric shared by the neighbor for it to reach some subnet (metric for neighbor to reach the subnet).
    Feasible Distance (FD) - RD + routers metric to reach the neighbor that shared the RP metric
    Successor - a route to subnet with the lowest FD
    Feasible successor - a backup route to subnet with 2nd lowest FD (used if primary successor failed)
        Some route can become feasible successor only if it's RD is lower than Successor's FD.

    Active state - Even though route might not match the criteria to become feasible successor (=route cannot be used in case all successors fail), query "how to reach the subnet" is sent into the network.
                   So Active state is when query is sent to other routers and an answer is expected.
        | There were an issue "Stuck in active" (SIA). That's when subnet reachability stops, router sends a query asking if there's another path (in active state at that moment)
        | and it sets timer for 3 minutes for query to come back. If some issues happen on link during that time - there will be no response and it'll bring the link down.
    Now there's SIA-query for such cases. If there's no responce in 90 seconds, router waiting for reply will send an SIA-query asking why there's no answer.
    If it's all good, they'll receive SIA-reply and will not bring link down.

EIGRP Stub routing
    Stub router - router which won't be used for queries about reaching the network if other routers lose the connectivity to some subnet.
                  Stub router is the one that has just a single router it connects to and no routers are connected further ahead.
                  Stub router doesn't distribute any routes to it's neighbors by default, that's why some options should be configured as well.
                  Command is under router eigrp <#>: "eigrp stub <options>" (options being "connected", "summary", "static" - depending on what should be advertised. Usually - connected)

EIGRP Load Balancing
    By default, there's only 1 route to destination and EIGRP chooses the best one.
    Feature "EIGRP Variance" can be configured to load balance the traffic over two links to same destination even if their metric is different. (by default up to 4 different links can be used)
    Variance is configured under router eigrp <#>: "variance <number>"
    Variance number works like following:
        Successors metric (FD) is multiplied by variance number and FD of other routes that will be within the range of multiplication result, will be used to load balance the traffic.
        For example there are two routes to subnet X.X.X.X, one with FD:5000, another with FD:9000. If variance of 2 is configured, 5000*2=10000, the route with FD:9000 will be used to LB the traffic.
        Load balancing will be proportional, since first route is better, it'll receive more traffic.
    
EIGRP Summarization
    There are Manual and Automatic summarization.
    Automatic is configured under router eigrp <#>: "auto-summary", but it summarizes in the classful boundaries (10.0.1.0 + 10.0.0.0 + 10.0.2.0 + 10.0.3.0 will be summarized into 10.0.0.0/8)
    Manual is configured under interface: "ip summary-address eigrp <#> <summarized-IP> <summary-MASK>"
    In addition to summarization, EIGRP router automatically creates a summarized route (10.0.0.0/8 for the above example) to Null0. That is done to provide additional security and prevent loops.

EIGRP for IPv6
    -Next-hop is neighbor link-local address
    -No auto-summarization
    -Neighbors don't have to be on the same subnet
    -Updates are on FF02::A

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ VERY POORLY DESCRIBED BY KEVIN WALLACE @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ CHECK SOME OTHER COURSES @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Named EIGRPv6 - a concept of EIGRP configuration (can be applied both to IPv4 and IPv6 versions) that allows configuring (under different mode) settings for different IP address-families at the same time   
                Done by creating EIGRP virtual instance ("router eigrp <name>")
                 address-families
My understanding: it is possible to configure EIGRP as a common instance (virtual instance) on the router, then distribute the EIGRP processes over address-families (like IPv4 and IPv6).
                  it's possible to configure settings for all interfaces (default) under that address families, also settings for the whole topology (variance)
                  In virtual instance another variable appears in regards to metric formula - K6 (for speeds more than 10gbit/s)
                  "Named" is because usually EIGRP is created "router eigrp <#>" where <#> is an AS number, and for named it should be created with a certain name instead of AS number.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ VERY POORLY DESCRIBED BY KEVIN WALLACE @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ CHECK SOME OTHER COURSES @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

EIGRP Authentication
    Plain text authentication - password is configured on routers and with EIGRP updates, the password is sent. It has to match on two routers to establish adjacency. Sent in plain text - UNSECURE
    Hash-based authentication - Password is configured on routers, with EIGRP updates the password hash-digest is sent. 
                                1. Update is sent from router with hash-key auth configured. It is sent with Hash-digest attached.
                                2. When another router receives update, it hashes the routing update with configured password.
                                3. If both Digest values match then update is valid and will be accepted (it is only possible if the same password is configured on both sides)
    Key-chain can be configured (same as OSPF) to cycle the hashing keys.

    Plain EIGRP supports only MD5 hashing algorithm.
    Named EIGRP supports both MD5 and SHA algorithms.

EIGRP passive-interface
    Hellos are not sent/received for the passive-interface

https://www.youtube.com/playlist?list=PLxyr0C_3Ton1IiuKroFD6i_vm61dapmKQ
https://www.youtube.com/watch?v=fg2flRb5us0&list=PLxyr0C_3Ton1IiuKroFD6i_vm61dapmKQ&index=23


#                BGP (eBGP AD - 20, iBGP AD - 200)

BGP whole-lab from Ivan Pepelnjak
https://github.com/ipspace/bgplab
https://ipspace.github.io/bgplab/

BGP Mind-Map
https://anetworkartist.blogspot.com/2023/06/bgpv4-mindmap.html

BGP Mnemonics:
    "We Love Oranges AS Oranges Mean Pure Refreshment"
     Weight (higher is better)
        Local Preference (higher is better)
             Originated locally
                     AS-path (shorter is better)
                        Origin type ( IGP < EGP < Incomplete )
                                MED (lower is better)
                                     Paths (external>internal)
                                           Router-ID (lower is better)

BGP basics
    BGP runs whole internet on it's shoulders
    Organizations are divided into AS (Autonomous System)
    Generally used to load-balance or filter updates/prefixes from ISPs

AS - "a network in a network of networks". An entity managed by one organization that is represented by some number of publicly available external subnets.
BGP itself is the only available protocol that allows organization to publish their servers (in internal subnets) externally. 
    Generally, some organization can get internet access just by routing everything to 0.0.0.0/0 through ISP. But organization's internal servers won't be available like that.

eBGP and iBGP exist
    eBGP - external BGP used over the internet (BGP is eBGP if neighbor is in different AS)
    iBGP - internal BGP generally used within the organization (BGP is iBGP if neighbor is in same AS) 

BGP Characteristics
    - EGP (Exterior Gateway Protocol)
    - Forms Neighbogships (adjacencies)
    - Neighbor's IP-address is manually configured
    - A TCP session is established (tcp/179) between neighbors
    - Advertises Address Prefix and Lenght (called NLRI - Network Layer Reachability Information)
    - Advertises a collection of Path Attributes used for path selection
    - Path-Vector routing protocol

BGP confederation & BGP Route-Reflector
    Example: there's iBGP of 5 routers. By default (by iBGP design), there must be a full-mesh connectivity between them.
    And since it is a MUST, there's always an assumption that there actually is.
    Therefore routers doesn't share the information received from neighbors by default.
    But if there's more and more routers - number of required adjacencies will be unreasonable.

    To address the above issue, there are some features available:
 (!)BGP confederation - a subset of ASes within one single AS. That way 3 routers from example can be in one sub-AS interconnected in full mesh (3 adjacencies), and other 2 routers in another sub-AS (1 adjacency there + 1 between sub-ASes)
                        that would basically be a set of eBGPs within an iBGP
 (!)Route-Reflector - a BGP router configuration allowing device to share the updates and prefixes (NLRI) received from other router. (works kind of as a DR in OSPF)
                      to configure router as an RR (route-reflector) - under "router bgp <#>", configure "neighbor <neigh_IP> route-reflector-client" on all routers that need to participate in iBGP route sharing

BGP Neighbor formation and States
    BGP neighbor establishment process is SLOW. Give it some time
    Idle state - initial state of forming an adjacency. Activates once the BGP configuration is entered for both routers (IDLE state) and before they started to try connecting via TCP/179 to each other. 
                 After that routers will try to establish a TCP connection. Transition to CONNECT state
                 If router stays in IDLE - probably there's no TCP connectivity between it and neighbor
    Connect state - At this state, the TCP 3-way handshake is attempting to happen. SYN> -> <SYN,ACK -> ACK> . CONENCT state is really brief state usually (since TCP connection is usually quickly happenning)
                    After successfully establishing connectivity, transition to ACTIVE state
    Active state - There's a TCP session established and routers are actively trying to establish the BGP neighborship. Then router sends a request for a BGP adjacency (message "Open sent" is sent). Transition to OPEN SENT state
    Open Sent state - Other router recognizes the "Open Sent" message from another router and transitions to OPEN STATE as well. Sends a confirmaion "Open Confirm" and transitions to OPEN CONFIRM state
    Open Confirm state - Initial router receives "Open Confirm" and then routers transition to ESTABLISHED state
    Established state - router in ESTABLISHED state if number of received prefixes can be seen in "sh ip bgp sum"

To establish neighborship, following parameters (sent in OPEN message) have to:
    - BGP versions must match
    - Source IP of received OPEN message must match the IP of configured neighbor
    - AS number received in OPEN message must match the AS of configured neighbor
    - Router-ID have to unique (lack of RID means condidion is not met)
    - Security parameters have to match

BGP Messages
    Open - sent during the neighborship negotiation. 
           carries "BGP Version Number", "Local AS Number", "Hold Time", "BGP Router ID", some optional params.
    Keepalive - basically a "Hello" message. Routers send a keepalive every 60 seconds (by default) for other router to be sure that the neighbor is still up and running. If keepalives are not received for 180 seconds, neighborship is teared up.
    Update - contains NLRI, path attributes, withdrawn routes, or some new routes. But it doesn't get sent instantly upon event, it waits for some time
    Notification - message to close the BGP connection. Contains an error code, error subcode, information about error

BGP Timers (default are 60s/180s - Keepalive/Hold)
    Keepalive timer
    Hold timer

BGP Path attributes
   -Well-known, mandatory --- attributes that MUST be supported by any router that has BGP implemented in compliance with RFC stardard. Always passed forward in NLRI
        AS-Path (all AS values that traffic has to pass on the way to announced subnet)
        Next-Hop (next-hop value to reach the subnet, usually it's an interface of advertising router)
        Origin (where router has been originated, either further AS, or locally or INCOMPLETE)
   -Well-known, discretionary --- attributes that MUST be supported by any router that has BGP implemented in compliance with RFC stardard. Passed forward in NLRI only if value is defined and non-default
        Local Preference 
        Atomic Aggregate
   -Optional, Transitive --- attributes (usually proprietary) that doesn't have to be supported by routers by default. Always passed forward in NLRI whether it is supported further on or not
        Aggregator
        Community
   -Optional, Non-Transitive --- attributes (usually proprietary) that doesn't have to be supported by routers by default. Doesn't have to be passed forward in NLRI.
        MED (Metric)
        Originator ID
        Cluster

BGP best path algorithm
    Attributes from mnemonics:
    Weight - Cisco proprietary. Used for outbound path decision. Router can set this parameter when receiving updates. Higher Weight is preferred. (Optional, Transitive)
    Local Preference - Used for outbound path decision. Carried throughout the AS. A number applied to some route once it is received in the table. Higher LocPref is preferred (Well-known, Discretionary)
    Originate - Where prefix has been originated. If originated locally, then preferred over any other NLRI.
    AS-Path - Number os AS to reach the prefix. Lower is better. Used for inbound path decision.
              Possible to manipulate using "AS-path prepend" - this allows adding of a lot of ASes to the path to make it less favorable.
    Origin Type - How the route was injected into the BGP. "i" is preferred to "e" and "e" is preferred to "?"
                  "i" - NETWORK command 
                  "e" - EGP (not used anymore) 
                  "?" - redistributed
    MED (Metric) - A number applied to externally distributed prefix. The lower the better. Example: i announce my external subnet and have few ISP connections. If i apply lower MED to one of announcements, it'll become the better path.
    Paths - Prefer eBGP path over iBGP path
    Router-ID - A tie breaker. Lower router ID is preferred.

    Currently used attributes for Cisco BGP process (going from top to bottom and if any attribute is more preferred, check stops and route is added to Routing Table):
    1. Higher Weight (assigned to prefix via route-map)
    2. Higher Local Preference (assigned to prefix via route-map)
    3. Originated locally (in same AS) or externally (cannot be changed as it is origination)
    4. Accumulated IGP (AIGP) - ?
    5. AS-Path (automatically built based on path, or can be changed through prepend)
    6. Origin Type (cannot be changed as it is origination)
    7. Lower MED (assigned to prefix via route-map)
    8. Paths (eBGP over iBGP) (cannot be manually manipulated)
    9. Oldest route
    10. Router-ID (lower RID wins) 
    11. Lower Cluster list length
    12. Lower next-hop address
    
BGP Synchronization
    Old feature (from early 1990-s) that allowed BGP router to advertise a prefix from itself to a eBGP neighbor ONLY if the initial router had the subnet in it's routing table received through some IGP (EIGRP, OSPF etc.)
    Example: There are two AS - 65100 and 65200.
             In 65100, there are two routers, in 65200 there is 1 router. Synchronization is ON for all in BGP processes
             65100 routers (R2 and R3) have iBGP formed between them, also 65200 router (R1) has eBGP formed to R2 from 65100
             R3 has a loopback IP and R2 has a loopback IP configured
          (!)If these loopbacks is not in any IGP (OSPF, EIGRP, IS-IS, RIP etc.), it will not be shared over eBGP neighborship
            To enable sharing - either configure IGP and add route to it's routing table or disable synchonization

BGP Summarization
    Summarized == Aggregated (in BGP)
    to summarize subnet in BGP - under "router BGP <#>", "aggregate-address <subnet_aggregated> <subnet_mask> <OPTIONS>" (OPTIONS can be "summary-only" for example, which makes router to ONLY advertise the aggregated address)

BGP Multihop
    in BGP it is possible to form a neighborship with non-adjacent router, with the router that is located behind the adjacent one, for example
    to do that, additional configuration is required under "router BGP <#>" - configure neighbor with it's remote-as, then configure neighbor with option "neighbor <neighbor_IP> ebgp-multihop <#>" (# - number of hops away)

BGP Peer-group
    peer-group can be created to make configuration easier for many neighbors at once with the same settings
    create a peer group and assign neighbor to it

eBGP vs. iBGP (differences)
    - sending a prefix in eBGP changes "next-hop" attribute for the prefix to the IP of router that shared the information about prefix last. 
            in iBGP - "next-hop" is the one initially received within the AS (if received by AS Edge device, the route is to outside of AS). Can be fixed by "neighbor <neigh_IP> next-hop-self" command on AS edge router
    - iBGP peers doesn't share prefixes received from BGP ("i" or "e") neighbors (route reflectors / BGP confederations help with that)

BGP IPv6
    configured by utilizing address-families (MP-BGP)
    before configuring, route-map has to be created that's going to advertise egress IPv6 address as a next-hop:
        "route-map <RM_NAME>"
        "set ipv6 next-hop <IPv6_on_router>"

    Basically, to configure ipv6 BGP, first under "router bgp <#>", a neighbor (ipv4 format) has to be specified
    then under "address-family ipv4" and "address-family ipv6", shared network should be specified in corresponding format (ipv4 for ipv4, ipv6 for ipv6)
    after that, neighbor have to be activated under "address-family ipv6" and route-map specified to set the next-hop to ipv6 address

https://www.youtube.com/watch?v=SVo6cDnQQm0
59:56
















Understand Wildcard masks
Understand subnetting














 
