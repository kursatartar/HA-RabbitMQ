# HA-RabbitMQ
RabbitMQ Cluster with High Availability


RabbitMQ Cluster Documentation
Overview
This documentation provides detailed information about the RabbitMQ cluster setup. The cluster is designed to provide high availability, fault tolerance, and data reliability using quorum queues and a load balancer. This setup ensures continuous operation even if one of the nodes becomes unavailable.

Architecture
Cluster Nodes
The cluster consists of three RabbitMQ nodes running on separate physical machines with the following configurations:

Node Name	IP Address	Role	Type
rabbit1	192.168.0.1	Seed Node	Disk Node
rabbit2	192.168.0.2	Joining Node	Disk Node
rabbit3	192.168.0.3	Joining Node	RAM Node


Features


High Availability (HA):

A load balancer ensures that client connections are distributed across the nodes. If one node becomes unavailable, the other nodes continue to handle requests seamlessly using the same port and domain.
Quorum Queues:

Quorum queues provide data replication across multiple nodes, minimizing the risk of data loss. Messages are persisted in a distributed manner to ensure availability even in case of node failures.
Fault Tolerance:

Each node is capable of handling RabbitMQ processes independently. If the seed node (rabbit1) fails, the remaining nodes (rabbit2 and rabbit3) continue to operate as part of the cluster.
Dynamic Node Recovery:

Nodes can rejoin the cluster automatically upon recovery, provided the Erlang cookie and node configuration remain consistent.


Cluster Configuration
Networking
Ports:
15672: Management UI
5672: AMQP protocol for messaging
25672: Cluster inter-node communication


Load Balancer:
Configured to distribute traffic to nodes at 192.168.0.1, 192.168.0.2, and 192.168.0.3.
Environment Variables
Each node is configured with the following:


RABBITMQ_ERLANG_COOKIE=secure_erlang_cookie
RABBITMQ_NODENAME=rabbit@<Node_IP>
RABBITMQ_USE_LONGNAME=true
Volumes
Persistent data storage is enabled for disk nodes:

/var/lib/rabbitmq
High Availability Features


1. Load Balancer
A load balancer ensures clients connect to the cluster via a single domain (e.g., rabbitmq.local) without being aware of individual node IP addresses.
If one node fails, the load balancer automatically redirects traffic to the remaining nodes.


2. Quorum Queues
Definition: Quorum queues replicate messages across multiple nodes in a majority-based fashion.
Key Benefits:
Redundancy: Messages are available on more than one node.
Consistency: In case of node failure, remaining nodes provide message continuity.
Minimal Data Loss Risk: With quorum queues, a majority of nodes must be available to process messages.


3. Fault Tolerance
RabbitMQ nodes maintain independent operation and rely on disk or memory as appropriate:
Disk Nodes: Store persistent data and provide cluster stability.
RAM Nodes: Use memory for faster access but rely on disk nodes for recovery.


Failure Scenarios and Recovery
Scenario 1: A Node Fails
Impact:
If rabbit1 (seed node) fails, the cluster continues to operate with rabbit2 and rabbit3.
The load balancer reroutes traffic to the active nodes.
Recovery:
Restart the failed node.
Ensure the Erlang cookie and node configuration are intact.
Rejoin the node to the cluster:

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit1@192.168.27.41
rabbitmqctl start_app

Scenario 2: Network Partition

Impact:
Nodes may temporarily lose connectivity with one another.

Recovery:
Ensure network connectivity is restored.
Use the rabbitmqctl command to verify the cluster status:

rabbitmqctl cluster_status

Scenario 3: Load Balancer Failure

Impact:

Client connections are disrupted.

Recovery:

Restart the load balancer service.
Configure a redundant load balancer for higher fault tolerance.

Node Status Commands
Check Node Status
rabbitmqctl status

Check Cluster Status
rabbitmqctl cluster_status

Add a Node to the Cluster

rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit1@192.168.27.41
rabbitmqctl start_app

Best Practices
Erlang Cookie Consistency:

Ensure the same erlang.cookie file is used across all nodes.

Set correct permissions:

chmod 400 /var/lib/rabbitmq/.erlang.cookie

Load Balancer Configuration:

Use tools like HAProxy or Nginx for load balancing.
Configure health checks to detect node failures.

Monitoring and Alerts:

Enable monitoring via RabbitMQ Management UI.
Use tools like Prometheus and Grafana for cluster metrics.

Backup Strategy:

Regularly back up /var/lib/rabbitmq for disk nodes.

Conclusion
This RabbitMQ cluster setup ensures high availability, scalability, and data reliability. With features like quorum queues and load balancing, it is well-suited for critical production environments where downtime and data loss are unacceptable.
