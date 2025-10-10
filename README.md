
# A Resilient Distributed Caching Infrastructure
This is a fault-tolerant, high-throughput distributed caching infrastructure designed to optimize data retrieval performance in large-scale distributed systems. It provides horizontal scalability, dynamic rebalancing, and fault recovery through a robust master–auxiliary architecture. The system ensures minimal latency and high availability, even under heavy load or node failure conditions.

![Architecture](distributed-cache.png)
## Key Highlights

-⚡ Scalable Multi-Node Architecture: Supports horizontal scaling with master and auxiliary nodes running in separate containers.
-🎯 Consistent Hashing: Guarantees efficient and balanced key distribution while minimizing data movement during node changes.
-⚙️ Smart Load Balancing: NGINX is integrated to distribute client requests evenly among multiple master servers.
-🧠 Intelligent Caching Layer: Each auxiliary server maintains an in-memory LRU cache using a hybrid Doubly Linked List + HashMap design for constant-time access and eviction.
-💾 Resilient Recovery Mechanism: Automatic data rebalancing and periodic volume backups ensure data persistence after crashes or node restarts.
-📈 Monitoring & Visualization: Real-time observability with Prometheus for metrics and Grafana for performance dashboards.
-🧰 Containerized Deployment: Fully containerized via Docker Compose for effortless deployment, scaling, and orchestration.

## System Architecture

The Cache System consists of the following components:

1. Master Node
The master acts as the control plane — routing requests, applying consistent hashing to determine the correct auxiliary node, and managing rebalancing operations when nodes join or leave the system. It ensures that key-to-server mappings remain optimal during scaling or recovery events.
2. Auxiliary Nodes
Auxiliary servers form the data layer, handling actual caching operations. Each node maintains its own local cache with an LRU eviction policy, ensuring high-speed access to frequently used data. The nodes are placed in a consistent hash ring for optimal key distribution.
3. Load Balancer
A centralized NGINX load balancer manages request distribution across master instances, supporting round-robin scheduling and connection pooling to handle high throughput gracefully.
4. Observability Stack
Prometheus agents on each node collect system metrics like request latency, throughput, and cache hit ratio. Grafana visualizes this telemetry data in intuitive dashboards for continuous monitoring and debugging.
5. Deployment Layer
All services (master, auxiliary, and monitoring components) are containerized with Docker Compose, allowing rapid setup and teardown for testing or scaling experiments.



## Data flow

1. Client Interaction: Clients interact to distributed cache system via Load Balancer.
   
2. Load Balancer Routing: The load balancer receives the client requests and distributes them among the available instances of the master node using Round-Robin policy. This ensures a balanced workload and improved performance. The configuration can be tweaked to increase the connection pool or increase the number of works processed by each of the worker.
   
3. Master Node Processing: The master node receives the client requests and performs the necessary operations. For write requests (storing key-value pairs), the master node applies a consistent hashing algorithm to determine the appropriate auxiliary server for data storage. It then forwards the data to the selected auxiliary server for caching. For read requests, the master node identifies the auxiliary server holding the requested data and retrieves it from there.
   
4. Auxiliary Server Caching: The auxiliary servers receive data from the master node and store it in their local LRU cache. This cache allows for efficient data access and eviction based on usage patterns.
   
5. Response to Clients: Once the master node receives a response from the auxiliary server (in the case of read requests) or completes the necessary operations (in the case of write requests), it sends the response back to the client through the load balancer. Clients can then utilize the retrieved data or receive confirmation of a successful operation.

## Recovery
- Health of auxiliary servers are monitored by master servers in a regular interval. If any of the auliliary servers go down or respawned, the master knows about it and rebalance the key-val mappings using consistent hashing.
  
- In case if one or more auxiliary nodes are shutdown, the key-vals mappings are sent to the master node which rebalance them using consistent hashing. 
  
- Each auxiliary server backs up data in their container volume every 10 sec, incase a catastrophic failure occurs. These backups are then used when the server is respawned.
  
- In case if one or more auxiliary nodes are respawned, the key-vals mappings from the corresponding nodes in the hash ring are rebalanced using consistent hashing.
  
- When redistributing/remapping the key-vals, a copy is backed up in the shared volume of master containers incase if the whole system goes down and has to be quickly respawned. The backups can be used to salvage as much data as possible. When respawning the backup is rebalanced to corresponding auxiliary server.


## Usage

1. Clone the repository:
   ```
   git clone 
   ``` 
2. Build and run the Docker containers for the master and auxiliary servers:
   ```
    docker-compose up --build
   ```
3. Run load testing:
   ```
   ./load_test/loadTest.sh
   ```

## Configuration

- To adjust the number of auxiliary servers, modify the `docker-compose.yml` file and add/remove auxiliary server instances as needed.

- The cache system's behavior, such as cache size, eviction policies, and request timeouts, can be configured in the server codebase.

