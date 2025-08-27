# Routing Algorithms & Protocols

Brief: Routing algorithms determine the best paths for data packets across networks, while routing protocols implement these algorithms to build and maintain routing tables. Understanding both theoretical algorithms and practical protocols is essential for network design and troubleshooting.

## Detailed Theory

### Routing Fundamentals

#### Basic Concepts
- **Routing Table**: Database of network destinations and next-hop information
- **Metric**: Cost measurement for path selection (hop count, bandwidth, delay, cost)
- **Convergence**: Time for all routers to have consistent routing information
- **Administrative Distance**: Trustworthiness of routing information sources
- **Next Hop**: The immediate router to forward packets toward destination

#### Routing Decisions
1. **Longest Prefix Match**: Most specific route wins
2. **Administrative Distance**: Lower AD preferred (Connected=0, Static=1, OSPF=110, RIP=120)
3. **Metric**: Lower metric preferred within same protocol
4. **Load Balancing**: Equal-cost or unequal-cost path usage

### Routing Algorithms

#### Distance Vector Algorithm (Bellman-Ford)

**Theory**
- Each router maintains distance (metric) to all known destinations
- Periodically shares entire routing table with neighbors
- Uses Bellman-Ford algorithm: `D[x,y] = min{c[x,v] + D[v,y]}`
- Count-to-infinity problem with slow convergence

**Algorithm Steps**
```
1. Initialize: Distance to self = 0, others = infinity
2. For each neighbor, receive distance vector
3. For each destination in received vector:
   new_distance = neighbor_distance + link_cost_to_neighbor
   if new_distance < current_distance:
       update_distance = new_distance
       next_hop = neighbor
4. Send updated vector to all neighbors
5. Repeat until convergence
```

**Advantages**
- Simple implementation
- Low memory requirements
- Automatic load balancing

**Disadvantages**
- Slow convergence
- Count-to-infinity problem
- Routing loops during convergence
- Full table exchange (bandwidth intensive)

#### Link State Algorithm (Dijkstra)

**Theory**
- Each router discovers complete network topology
- Floods link state information to all routers
- Uses Dijkstra's shortest path algorithm
- Fast convergence, loop-free

**Algorithm Steps**
```
1. Discover neighbors and link costs
2. Flood link state advertisements (LSAs) to all routers
3. Build link state database (LSDB) of complete topology
4. Run Dijkstra's algorithm to compute shortest path tree
5. Install best routes in routing table
```

**Dijkstra's Algorithm**
```python
def dijkstra(graph, source):
    distances = {node: float('infinity') for node in graph}
    distances[source] = 0
    unvisited = set(graph.keys())
    
    while unvisited:
        current = min(unvisited, key=lambda x: distances[x])
        unvisited.remove(current)
        
        for neighbor, weight in graph[current].items():
            distance = distances[current] + weight
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                previous[neighbor] = current
    
    return distances, previous
```

**Advantages**
- Fast convergence
- Loop-free routing
- Detailed topology knowledge
- Event-driven updates

**Disadvantages**
- Higher memory requirements
- More complex implementation
- CPU intensive calculations
- Flooding overhead

### Interior Gateway Protocols (IGP)

#### RIP (Routing Information Protocol)

**RIP Version 1**
- Distance vector protocol
- Hop count metric (max 15, 16 = unreachable)
- 30-second periodic updates
- No subnet mask support (classful)
- Broadcast updates

**RIP Version 2**
- Subnet mask support (classless)
- Multicast updates (224.0.0.9)
- Authentication support
- Route tags for external routes
- Next-hop field

**Configuration Example**
```
router rip
 version 2
 network 192.168.1.0
 network 10.0.0.0
 no auto-summary
 passive-interface gi0/1
```

**Advantages**
- Simple configuration
- Low resource usage
- Automatic summarization
- Wide vendor support

**Disadvantages**
- Limited scalability (15 hop limit)
- Slow convergence (3 minutes)
- Periodic updates waste bandwidth
- No load balancing support

#### OSPF (Open Shortest Path First)

**Core Concepts**
- Link state protocol
- Hierarchical design with areas
- SPF algorithm (Dijkstra)
- Cost metric based on bandwidth
- LSA flooding within areas

**OSPF Areas**
- **Area 0 (Backbone)**: Must exist, connects all other areas
- **Standard Areas**: Connected to backbone via Area Border Routers (ABR)
- **Stub Areas**: Don't receive external LSAs (Type 5)
- **Totally Stubby**: Don't receive external or inter-area LSAs
- **NSSA**: Not-So-Stubby Area, limited external route injection

**LSA Types**
1. **Type 1 (Router LSA)**: Router's directly connected links
2. **Type 2 (Network LSA)**: Multi-access network information
3. **Type 3 (Summary LSA)**: Inter-area routes (ABR generated)
4. **Type 4 (ASBR Summary)**: Path to ASBR in different area
5. **Type 5 (External LSA)**: External routes (ASBR generated)
7. **Type 7 (NSSA External)**: External routes within NSSA

**Configuration Example**
```
router ospf 1
 router-id 1.1.1.1
 area 0 authentication message-digest
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.255.255.255 area 1
 area 1 stub
 default-information originate
```

**Advantages**
- Fast convergence
- Hierarchical design
- VLSM support
- Authentication
- Load balancing

**Disadvantages**
- Complex configuration
- High memory/CPU usage
- Requires careful design
- Area 0 dependency

#### EIGRP (Enhanced Interior Gateway Routing Protocol)

**Hybrid Protocol**
- Combines distance vector and link state features
- DUAL algorithm (Diffusing Update Algorithm)
- Feasible successor concept
- Unequal cost load balancing

**DUAL Algorithm**
- **Feasible Distance (FD)**: Best metric to destination
- **Reported Distance (RD)**: Neighbor's metric to destination
- **Feasible Successor**: Backup path where RD < FD
- **Active/Passive States**: Query process for lost routes

**Configuration Example**
```
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 10.0.0.0 0.255.255.255
 no auto-summary
 variance 2
 maximum-paths 6
```

### Exterior Gateway Protocols (EGP)

#### BGP (Border Gateway Protocol)

**Path Vector Protocol**
- Policy-based routing
- Autonomous System (AS) path attribute
- Prevents loops through AS path
- Supports routing policies

**BGP Attributes**
- **AS Path**: List of ASes packet traversed
- **Next Hop**: Next router IP address
- **Local Preference**: Path preference within AS
- **MED**: Multi-Exit Discriminator, preference hint to neighbors
- **Origin**: How route was injected into BGP
- **Communities**: Route tagging for policy application

**BGP Session Types**
- **eBGP**: Between different ASes
- **iBGP**: Within same AS (full mesh or route reflectors)

**Configuration Example**
```
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 203.0.113.1 remote-as 65002
 neighbor 203.0.113.1 description "ISP Connection"
 network 192.168.0.0 mask 255.255.0.0
 aggregate-address 192.168.0.0 255.255.0.0 summary-only
```

### Advanced Routing Concepts

#### Route Redistribution
```
router ospf 1
 redistribute eigrp 100 metric 20 subnets
 redistribute static metric 10 subnets

router eigrp 100
 redistribute ospf 1 metric 1000 10 255 1 1500
```

#### Policy Routing
```
ip access-list extended MATCH_TRAFFIC
 permit ip 192.168.1.0 0.0.0.255 any

route-map POLICY_ROUTE permit 10
 match ip address MATCH_TRAFFIC
 set ip next-hop 10.0.0.2

interface gi0/1
 ip policy route-map POLICY_ROUTE
```

#### Load Balancing
```
# Equal Cost Load Balancing
router ospf 1
 maximum-paths 4

# Unequal Cost Load Balancing (EIGRP)
router eigrp 100
 variance 2
 traffic-share balanced
```

### Routing Security

#### Authentication
```
# OSPF MD5 Authentication
interface gi0/0
 ip ospf message-digest-key 1 md5 mypassword

router ospf 1
 area 0 authentication message-digest
```

#### Route Filtering
```
# Distribute List
ip access-list standard FILTER_ROUTES
 deny 192.168.10.0 0.0.0.255
 permit any

router ospf 1
 distribute-list FILTER_ROUTES in gi0/0
```

### Practical Examples

#### Network Scenario
```
Internet --- [R1 BGP] --- [R2 OSPF] --- [R3 EIGRP] --- LAN
            65001 AS      Area 0         AS 100      192.168.1.0/24
```

#### Routing Table Analysis
```
R3# show ip route
Codes: C - connected, S - static, D - EIGRP, O - OSPF, B - BGP

Gateway of last resort is 10.0.0.1 to network 0.0.0.0

B*   0.0.0.0/0 [20/0] via 10.0.0.1, 00:30:25
O    192.168.2.0/24 [110/20] via 10.0.0.1, 00:15:30, GigabitEthernet0/0
D    192.168.3.0/24 [90/156160] via 192.168.1.2, 00:45:12, GigabitEthernet0/1
C    192.168.1.0/24 is directly connected, GigabitEthernet0/1
```

## Interview Questions

1. **Q: Explain the difference between distance vector and link state routing algorithms.**
   **A:** Distance vector routers know distance/direction to destinations but not complete topology. They share routing tables with neighbors. Link state routers know complete network topology and use Dijkstra's algorithm to compute shortest paths.

2. **Q: What is the count-to-infinity problem in distance vector protocols?**
   **A:** When a route fails, routers may learn about the failed route from each other, gradually incrementing the metric until it reaches infinity. This causes slow convergence and temporary routing loops.

3. **Q: How does OSPF use areas and why is Area 0 special?**
   **A:** OSPF uses areas to create hierarchy and reduce LSA flooding. Area 0 is the backbone area that all other areas must connect to, either directly or through virtual links. This prevents routing loops and ensures connectivity.

4. **Q: What is the difference between IGP and EGP protocols?**
   **A:** IGP (Interior Gateway Protocols) like OSPF, EIGRP, RIP operate within a single autonomous system. EGP (Exterior Gateway Protocols) like BGP operate between different autonomous systems on the Internet.

5. **Q: Explain how BGP prevents routing loops.**
   **A:** BGP uses the AS path attribute containing the list of autonomous systems a route has traversed. If a router sees its own AS number in the path, it rejects the route, preventing loops.

6. **Q: What is administrative distance and how is it used?**
   **A:** Administrative distance indicates the trustworthiness of routing information sources. Lower values are preferred: Connected=0, Static=1, EIGRP=90, OSPF=110, RIP=120. It's used when multiple protocols provide routes to the same destination.

7. **Q: How does EIGRP achieve fast convergence?**
   **A:** EIGRP uses DUAL algorithm with feasible successors (backup routes) that are pre-computed and loop-free. When primary route fails, it immediately switches to feasible successor without recalculation.

8. **Q: What is route summarization and why is it important?**
   **A:** Route summarization combines multiple specific routes into a single summary route, reducing routing table size and update traffic. It improves scalability and stability by hiding subnet changes from other areas.

9. **Q: Explain the concept of routing metrics and how different protocols use them.**
   **A:** Metrics determine best path selection. RIP uses hop count, OSPF uses cost (based on bandwidth), EIGRP uses composite metric (bandwidth, delay, reliability, load), BGP uses path attributes and policies.

10. **Q: What happens during OSPF convergence after a link failure?**
    **A:** Router detects failure → floods LSA about topology change → all routers update LSDB → run SPF algorithm → install new routes. Process typically completes in seconds with proper tuning.

11. **Q: How does BGP handle route selection when multiple paths exist?**
    **A:** BGP uses path selection algorithm: highest weight → highest local preference → locally originated → shortest AS path → lowest origin → lowest MED → eBGP over iBGP → lowest IGP cost → oldest route → lowest router ID.

12. **Q: What is the difference between equal-cost and unequal-cost load balancing?**
    **A:** Equal-cost load balancing splits traffic equally across paths with identical metrics. Unequal-cost (like EIGRP variance) allows load balancing across paths with different metrics proportional to their costs, enabling better link utilization.
