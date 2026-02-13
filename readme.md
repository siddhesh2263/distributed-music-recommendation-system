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

This project presents the design of a scalable and fault tolerant distributed music recommendation system built around partitioned RNN and LSTM model inference. The architecture follows a client server model where users search songs or upload playlists through a web interface. Requests are routed through a load balancer to application servers, which publish inference tasks to Kafka topics. A dispatcher node partitions the neural network into sequential sub networks and distributes computation across multiple compute nodes. Kafka, coordinated by Zookeeper, enables asynchronous communication, topic partitioning, and broker replication. Kubernetes manages containerized services, monitors node health, and replaces failed instances to maintain availability. The system design addresses scalability through horizontal distribution of compute nodes and load balancing, and addresses failure handling through server replication and message queue redundancy. Performance comparison between traditional monolithic inference and distributed inference shows significant latency reduction under parallel execution.

<br>

## Properties Implemented

Scalability is addressed through horizontal distribution of inference workloads using model partitioning and Kafka topic partitioning. Failure handling is supported through server replication, broker replication, and Kubernetes based health monitoring with automated recovery. Communication reliability is ensured through asynchronous messaging and decoupled components. Other challenges such as heterogeneity, openness, security, concurrency, quality of service, and transparency are identified but not fully implemented. These are analyzed in terms of architectural implications, tradeoffs, and potential extensions required to support production grade distributed deployments.

<br>

## Literature Review

The literature review examines federated learning based recommendation systems and distributed machine learning architectures that address privacy, scalability, and fault tolerance. Federated learning decentralizes model training by keeping user data on local devices and sharing only model updates, reducing privacy risks associated with centralized systems. Key challenges include device heterogeneity, non IID data distribution, security threats such as poisoning attacks, scalability to large user populations, fairness across participants, and fault tolerance. Approaches such as tier based client selection in TiFL mitigate stragglers, while peer to peer architectures like BrainTorrent remove centralized coordinators. Secure aggregation techniques protect model updates during communication.

The review also analyzes the parameter server architecture for large scale distributed training. It enables asynchronous communication, flexible consistency models, elastic scalability, and replication based fault tolerance. Techniques such as bounded delay consistency, vector clocks, message compression, and range based push and pull improve efficiency. Together, these works provide system level foundations for scalable and reliable distributed machine learning.

<br>

## Use Case Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/004_use_case_diagram.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

## Component Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/002_component_diagram_main.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

### Server to Dispatcher Component Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/006_server_to_dispatcher.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

### Dispatcher to Compute Nodes Component Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/003_dispatcher_to_compute.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

## Sequence Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/001_sequence_diagram.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

## Activity Diagram

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

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/005_kubeview.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

## Conculsion

Federated learning and distributed parameter server architectures provide complementary approaches to scalable and privacy aware machine learning. While federated systems prioritize decentralization and data privacy, parameter servers focus on efficient synchronization and large scale training. Addressing heterogeneity, security, consistency, and fault tolerance remains critical for reliable deployment in real world recommendation systems.

<br>

## Initial Work - Deployment Diagram

<p align="center">
  <img 
    src="https://github.com/siddhesh2263/distributed-music-recommendation-system/blob/main/assets/008_deployment_diagram.png?raw=true"
    alt="Maven Lifecycle"
    width="800"
  />
</p>

<br>

## Credits