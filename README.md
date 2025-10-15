
# A Resilient Multi Node Caching Infrastructure
This is a fault-tolerant, high-throughput distributed caching infrastructure designed to optimize data retrieval performance in large-scale distributed systems. It provides horizontal scalability, dynamic rebalancing, and fault recovery through a robust master–auxiliary architecture. The system ensures minimal latency and high availability, even under heavy load or node failure conditions.


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

-Client Request: Incoming requests first reach the NGINX load balancer.  
-Master Routing: The master applies consistent hashing to locate the appropriate auxiliary node.  
-Cache Operation: The auxiliary node performs a GET or PUT in its local LRU cache.  
-Response Delivery: The result is returned to the client through the master and load balancer layers.  
-Metrics Pipeline: All events are logged to Prometheus for performance tracking.  

## Fault Recovery & Rebalancing
-The master continuously monitors auxiliary node health via heartbeat checks.  
-Upon node failure, data is rebalanced using consistent hashing with minimal redistribution overhead.  
-Each auxiliary node backs up its cache snapshot every 10 seconds to persistent storage volumes.  
-During system restart, master containers restore data from shared volume backups for rapid recovery.  


## Testing & Benchmarking
A dedicated load testing suite built with Locust simulates high concurrency to measure system performance under stress.  
Scenarios include:  
-Parallel read/write operations with varying payload sizes  
-Node addition/removal simulations  
-Latency and throughput tracking under scaled workloads  


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

-The number of auxiliary servers can be scaled dynamically by updating the docker-compose.yml configuration to add or remove instances.

-Core cache parameters — including cache size, eviction policy, and request timeout — are customizable within the server’s configuration files.

