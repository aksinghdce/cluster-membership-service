# cluster-membership-service
This is an assignment project in the course "EECE513:Error resilient systems"

# Introduction
We are building a cluster membership service to demonstrate the algorithms that are deployed
to manage a fault-tolerant distributed file system. the membership service will provide an api
to return the real-time state of the cluster.

In order to manage the cluster nodes we are using [docker-compose](https://docs.docker.com/compose/).
 
The [predecessor repository](https://github.com/aksinghdce/docker_compose_cluster) contains
the foundational code in which we are using docker-compose command to launch a cluster. 
In the [predecessor repository](https://github.com/aksinghdce/docker_compose_cluster) we 
have implemented distributedgrep service, we will reuse most of the lower layer functionality
from the  [predecessor repository](https://github.com/aksinghdce/docker_compose_cluster).

In this repository we are planning to build over libcompose library for docker-compose.

# Requirements

The [requirement document](https://drive.google.com/open?id=1GmCzOCRRUJUxvJtFG9HMxuTbIBlGs61F)
specifies the constraints of the programming assignment. The requirements are listed below 
for easy reference:

## Learning outcomes : How to program a distributed membership service!
1. You will develop the basis for a group communication system - A membership service.
2. This service should maintain a list of machines that are part of the group, are running
 and are connected to the membership service.

## Constraints on the membership service
1. It must run as a daemon process.
### The daemon process must react to the following events : 
1. A new machine joins the group (i.e., the daemon process on that machine connects to the service)
2. A machine leaves the group voluntarily (i.e., the daemon process on that machine sends a request to disconnect from the service)
3. A machine fails (i.e., the daemon process on that machine is no longer responding to requests/messages).

### Assumptions
1. There is only one group at any point in time
2. The failure model is crash-stop
3. The machines are identified, in part, by their IP address
4. When a machine rejoins the group, it must do so with an identifier that is linked to a timestamp (distinguishing a restarted machine from a prior incarnation of itself)
5. At most three machines may fail simultaneously
6. The separation time between three back-to-back failures is at least 20 seconds

### Efficiency constraints
1. Time-bounded completeness: A failure must be reflected in at least one of the active membership lists within 2 seconds and this must be ensured irrespective of network latencies (assuming synchronized clocks and the condition that most network links have much lower latencies)
2. An update to the membership list must be reflected in all membership lists within 6 seconds
3. Your implementation must be scalable to a large number of members
4. You must use heartbeats to detect failures, and heartbeats are exchanged along an extended ring topology (see below for more details)
5. You must use the minimum number of heartbeat monitors and the smallest needed bandwidth to achieve the time-boundedness requirement (use UDP connections).

### Suggestions
1. An extended ring is a (virtual) topology where every machine has some predecessors and some successors, and heartbeat messages are received from the predecessors and sent to the successors. 
2. You can choose where a new machine should be added in an extended ring so as to satisfy the requirements. Think of how many successors and predecessors are needed to satisfy the requirements.
3. For detecting failures, or for tracking voluntary departures of machines, you should not use a master node because such a node can fail and its failure must be detected. 
4. It is, however, acceptable to designate a gateway node that processes join requests. If — and when — the gateway node is down, new joins cannot be processed but failures should be detected and departures should be processed.

[Comment: Amit] We can use the following architecture to design a gateway node in a redundant manner:

![Design for Gateway Node](cluster-membership-service/doc/images/membership_service_topology.png)


5. You will have to pay attention to the message format for messages exchanged by the membership service. You may need to marshal platform-dependent fields (such as ints) into a platform-independent format. An example is Google Protocol Buffers that you may choose to use. This is not a requirement. You should clearly document your protocol message format.
6. Have the membership service daemon processes log messages sent or received. You can then use your implementation from the previous assignment to debug. Integration with an implementation of the distributed log query mechanism is important.