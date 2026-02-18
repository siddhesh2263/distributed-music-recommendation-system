# Distributed Music Recommendation System

## Table of Contents

- [Project Overview](#project-overview)
- [Properties Implemented](#properties-implemented)
- [Literature Review](#literature-review)
- [Use Case Diagram](#use-case-diagram)
- [Component Diagram](#component-diagram)
  - [Server to Dispatcher Component Diagram](#server-to-dispatcher-component-diagram)
  - [Dispatcher to Compute Nodes Component Diagram](#dispatcher-to-compute-nodes-component-diagram)
- [Sequence Diagram](#sequence-diagram)
- [Activity Diagram](#activity-diagram)
- [Kubeview](#kubeview)
- [Conclusion](#conclusion)
- [Initial Work - Deployment Diagram](#initial-work---deployment-diagram)
- [Credits](#credits)

<br>

## Project Overview

This project presents the design of a scalable and fault tolerant distributed music recommendation system built around partitioned RNN and LSTM model inference.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/project_overview.png?raw=true"
    alt="Use Case"
    width="400"
  />
</p>


<br>

## Properties Implemented

Scalability is addressed through horizontal distribution of inference workloads using model partitioning and Kafka topic partitioning. Failure handling is supported through server replication, broker replication, and Kubernetes based health monitoring with automated recovery. Communication reliability is ensured through asynchronous messaging and decoupled components. Other challenges such as heterogeneity, openness, security, concurrency, quality of service, and transparency are identified but not fully implemented. These are analyzed in terms of architectural implications, tradeoffs, and potential extensions required to support production grade distributed deployments.

<br>

## Literature Review

The literature review examines federated learning based recommendation systems and distributed machine learning architectures that address privacy, scalability, and fault tolerance. Federated learning decentralizes model training by keeping user data on local devices and sharing only model updates, reducing privacy risks associated with centralized systems. Key challenges include device heterogeneity, non IID data distribution, security threats such as poisoning attacks, scalability to large user populations, fairness across participants, and fault tolerance. Approaches such as tier based client selection in TiFL mitigate stragglers, while peer to peer architectures like BrainTorrent remove centralized coordinators. Secure aggregation techniques protect model updates during communication.

The review also analyzes the parameter server architecture for large scale distributed training. It enables asynchronous communication, flexible consistency models, elastic scalability, and replication based fault tolerance. Techniques such as bounded delay consistency, vector clocks, message compression, and range based push and pull improve efficiency. Together, these works provide system level foundations for scalable and reliable distributed machine learning.

<br>

## Use Case Diagram

This use case diagram models the functional behavior of the Server subsystem in the distributed music recommendation system. The primary actor is the User, who interacts with the system by searching for songs, selecting songs, uploading a playlist, and viewing recommended songs. The Search Songs and Select Songs use cases are connected to Display Songs, which includes the Fetch Songs operation. Fetch Songs interacts with the Database actor, indicating that song data retrieval is handled externally through persistent storage. The Verify Song Data use case also communicates with the Database to ensure correctness and validation of selected content.

For playlist based recommendations, the Upload Playlist use case includes Verify Playlist Data, ensuring input validation before processing. Once validated, the system proceeds toward generating recommendations. On the right side, the Kafka Cluster actor participates in backend processing operations such as Songs Formatter and Inference Aggregator. The Inference Aggregator interacts with Compute Node actors, reflecting distributed model execution. Model Partitioning includes Model Dispatcher, which coordinates workload distribution across compute nodes. The Display Recommended Songs use case depends on inference aggregation, illustrating that recommendations are generated only after distributed processing completes. Overall, the diagram captures user interactions, backend validation, distributed inference orchestration, and external system dependencies.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/004_use_case_diagram.png?raw=true"
    alt="Use Case"
    width="800"
  />
</p>

<br>

## Component Diagram

This component diagram presents the high level architecture of the distributed music recommendation system. Users interact with the website subsystem to search songs, upload playlists, and view recommendations. Requests are routed to the central server, which contains API services for search, recommendation generation, playlist verification, logging, and circuit breaking. A load balancer distributes traffic across server instances. The server communicates with the database subsystem to access music datasets and model weights. For inference, requests are published to the messaging queue subsystem implemented using Kafka and Zookeeper. The distributed ML subsystem partitions the model across compute nodes, coordinated by a dispatcher. Kubernetes provides monitoring and automatic recovery.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/002_component_diagram_main.png?raw=true"
    alt="Main Component"
    width="800"
  />
</p>

### Server to Dispatcher Component Diagram

This diagram illustrates communication between application servers and the Kafka cluster within the distributed system. User requests arrive at the load balancer, which distributes traffic across Server 1, Server 2, and Server 3. Each server acts as a Kafka producer and consumer, publishing inference tasks to Kafka topics and consuming responses. Within Kafka, Zookeeper coordinates broker state and replication, while brokers manage topic partitions for parallel execution. Partitioning enables workload distribution across multiple consumers. Dispatcher nodes subscribe as Kafka consumers, retrieve partitioned tasks, and act as producers when forwarding results. Kubernetes monitors cluster components and ensures availability through health management and recovery mechanisms.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/006_server_to_dispatcher.png?raw=true"
    alt="Server to Dispatcher"
    width="800"
  />
</p>

<br>

### Dispatcher to Compute Nodes Component Diagram

This diagram represents the core distributed inference workflow between the Kafka cluster and the distributed ML subsystem. The dispatcher node acts as both a Kafka producer and consumer. It receives inference requests from upstream services through a Kafka topic, processes metadata, and publishes partitioned inference tasks back into Kafka. Kafka brokers, coordinated by Zookeeper, manage topic partitions that enable parallel execution. Replication across brokers ensures durability and fault tolerance.

Each compute node subscribes to specific Kafka partitions as a consumer. The partitioning strategy guarantees that inference workloads are distributed across Compute Node 1, Compute Node 2, and Compute Node 3. After processing assigned model layers, compute nodes publish intermediate or final results back to Kafka as producers. This bidirectional producer consumer pattern enables asynchronous coordination without tight coupling between services.

The dispatcher aggregates partial inference outputs retrieved from Kafka and constructs the final recommendation result. Topic partitioning ensures parallelism, while broker replication supports reliability during node or broker failures. The architecture allows horizontal scaling by increasing partition counts and compute nodes without modifying the inference logic. Kubernetes monitors the overall infrastructure and ensures recovery of failed components, maintaining continuous distributed model execution under varying load conditions.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/003_dispatcher_to_compute.png?raw=true"
    alt="Dispathcer to Compute"
    width="800"
  />
</p>

<br>

## Sequence Diagram

This sequence diagram captures the end to end interaction flow of the distributed music recommendation system, including normal execution and failure handling. The process begins when a user interacts with the website to search for songs or upload a playlist. The request is routed through the load balancer to the application server, which validates input and communicates with the database for verification. Once the request is ready for inference, the server publishes the task to a Kafka broker.

Kafka, coordinated by Zookeeper, manages topic partitions and replication. The dispatcher node consumes the published task, partitions the RNN and LSTM model, and forwards sub tasks through Kafka topics to compute nodes. Each compute node processes its assigned model layers and sends intermediate or final results back via Kafka. The dispatcher aggregates results and sends the final recommendation response to the application server, which returns it to the user.

The diagram also models failure scenarios such as broker crashes or dispatcher node failures. Zookeeper detects broker failures and triggers replication mechanisms. Kubernetes monitors services and initiates recovery for failed dispatcher or compute nodes. Conditional flows illustrate retry logic, error responses, and recovery steps, demonstrating fault tolerance, asynchronous communication, and coordinated distributed inference across multiple system components.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/009_sequence_clear.png?raw=true"
    alt="Sequence"
  />
</p>

<br>

## Activity Diagram

This activity diagram shows the end to end control flow from user input to recommendation output across system lanes. The user either searches songs by entering three characters or uploads a Spotify playlist. The Recommender API validates input, fetches songs from the database, and ensures at least five songs are selected. Valid requests are sent to Kafka through a handler that checks topic acceptance and returns errors on failure. The dispatcher consumes tasks, partitions the RNN and LSTM model weights, and dispatches work to distributed compute nodes through Kafka. Compute nodes process assigned layers, return results to Kafka, and the system aggregates outputs to generate the final recommendation for display.

<p align="center">
  <div style="background-color: white; padding: 12px; display: inline-block;">
    <img 
      src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/007_activity_diagram.jpg?raw=true"
      alt="Activity Diagram"
      width="800"
    />
  </div>
</p>

<br>

## Kubeview

This KubeView visualization represents the Kubernetes deployment of the distributed music recommendation system within the `distributed-systems` namespace. At the top layer, two ingress resources, `distributed-systems-ingr` and `ui-ingress`, expose the application to external traffic. These ingress controllers route HTTP requests to internal services, separating user interface traffic from backend system traffic. This structure reflects a clean edge routing strategy and enables controlled access into the cluster.

The middle layer consists of multiple deployments: `dispatcher`, `rec-ui`, `search-app`, and three inference services labeled `server-1`, `server-2`, and `server-3`. Each deployment manages its own pod replica, shown directly beneath them. The dispatcher coordinates distributed inference tasks, while the search application handles query processing. The three server deployments represent partitioned compute nodes responsible for processing segments of the neural network model. This separation reflects logical distribution of inference workloads across independent services.

At the bottom layer, Kubernetes services such as `rec-ui-svc` and `app-svc` abstract pod networking and provide stable internal endpoints. These services enable communication between components without exposing pod-level details. Overall, the deployment illustrates microservice separation, container orchestration, internal service abstraction, and readiness for horizontal scalability within a Kubernetes-managed distributed system.


<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/005_kubeview.png?raw=true"
    alt="Kubeview"
    width="800"
  />
</p>

<br>

## Conculsion

Federated learning and distributed parameter server architectures provide complementary approaches to scalable and privacy aware machine learning. While federated systems prioritize decentralization and data privacy, parameter servers focus on efficient synchronization and large scale training. Addressing heterogeneity, security, consistency, and fault tolerance remains critical for reliable deployment in real world recommendation systems.

<br>

## Initial Work - Deployment Diagram

This deployment diagram illustrates the physical infrastructure of the distributed system across multiple devices. A client device connects to a shard service running inside a container on a Dell server virtual machine. A separate Dell server hosts the database server VM, which runs MySQL in a container storing the Songs_DB. The distributed inference layer is deployed across three independent nodes, each running a containerized App1 service instance. Two additional Dell servers host central server VMs running the Node Dispatcher Service in containers. This layout reflects separation of concerns, containerization, and horizontal distribution of compute and coordination components.

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/008_deployment_diagram.png?raw=true"
    alt="Deployment"
    width="800"
  />
</p>

<br>

## Credits

```
Ruturaj Shitole
Shaival Vora
Sai Kumar Nekkanti
Srinivas Ravindranath
Siddhesh Kocharekar
```