## High-Level Design (HLD) for Lift-and-Shift Migration of WebServerVM and SQLVM Application 
The application consists of two virtual machines: the WebServerVM and the SQLVM. To ensure high availability in the cloud, the application will be deployed in a multi-region setup with load balancing between regions. The application can handle no more than 6 hours of downtime.

### 1. A solution diagram showing the target architecture with load balancers and multi-region VMs.
This is my solution diagram:
``` mermaid
graph TD
    Client[Client] --> ALB[Application Load Balancer - Multi-Region]
    
    subgraph Region 1 - Primary
        WebServerVM1[WebServerVM - Primary]
        SQLVM1[SQLVM - Primary]
    end
    
    subgraph Region 2 - Secondary
        WebServerVM2[WebServerVM - Secondary]
        SQLVM2[SQLVM - Secondary]
    end

    ALB --> WebServerVM1
    ALB --> WebServerVM2
    WebServerVM1 --> SQLVM1
    WebServerVM2 --> SQLVM2

    SQLVM1 <-- Replication --> SQLVM2
```
In this diagram:
- Application Load Balancer (ALB): Routes client requests to web servers in both regions.
- WebServerVM Replication: WebServerVMs are replicated across both regions for redundancy.
- SQLVM Replication: SQLVMs in both regions are set up with replication for failover.
- Multi-region Architecture: Ensures high availability with load balancing and automatic failover between Region 1 and Region 2.
### 2. Target Architecture: Redundancy and Failover Mechanisms
The application will be deployed across two cloud regions to ensure high availability and redundancy, minimizing downtime to no more than 6 hours. The target architecture includes the following components:

A. Multi-Region Setup

- The architecture spans two regions: Region 1 (Primary) and Region 2 (Secondary).
- Each region contains a WebServerVM (for the frontend) and a SQLVM (for the backend database).
- This setup ensures geographical redundancy, protecting the application from regional failures and ensuring continuity.

B. Web Server Redundancy and Load Balancing

- WebServerVM instances are deployed in both regions, hosting static content for the application.
- An Application Load Balancer (ALB) distributes incoming traffic between the WebServerVMs in an active-active configuration, balancing the load between both regions.
- The ALB continuously monitors the health of the web servers through health checks. If a web server in Region 1 becomes unavailable, the load balancer automatically routes traffic to the web server in Region 2.

C. Database Redundancy and Replication

- The SQLVM in Region 1 acts as the primary database server, while the SQLVM in Region 2 serves as the secondary server.
- Asynchronous database replication is set up between the primary and secondary SQLVMs to keep data synchronized across regions.
- In the event of a failure in Region 1, the system will trigger an automatic failover to SQLVM in Region 2, ensuring minimal data loss and service disruption. The failover process is designed to take place within the allowed downtime window (less than 6 hours).

D. Failover Mechanism

- Web Server Failover: If the primary web server (WebServerVM in Region 1) fails, the load balancer routes all traffic to the web server in Region 2. The architecture can continue serving static content even if Region 1 is unavailable.
- Database Failover: If the primary database server (SQLVM in Region 1) fails, the secondary database in Region 2 will automatically take over, ensuring that the backend service is restored within the allowed downtime window. The failover process involves promoting the secondary SQLVM to primary and ensuring that all web servers point to it.

This architecture ensures fault tolerance, high availability, and minimal downtime, with the application capable of sustaining regional outages while maintaining operational continuity.

### 3. Migration Steps

#### Step 1: Replication of Virtual Machines Across Regions
- WebServerVM Replication: Create a new WebServerVM in Region. Ensure that both instances in Region 1 and Region 2 are identical in terms of configuration, OS, and software installed.
- SQLVM Replication: Set up SQLVM in Region 2 and configure it as a secondary database to the SQLVM in Region 1. Use database replication (asynchronous) to synchronize data between the primary and secondary databases.
#### Step 2: Configuration of Load Balancers
- Application Load Balancer (ALB): Configure the ALB to distribute traffic across the WebServerVM instances in both regions. The ALB should be set up in an active-active configuration to ensure load balancing and high availability.
- Set up health probes on both web servers so that the ALB can route traffic to healthy instances only.
#### Step 3: Implementation of Database Replication and Failover
- Configure SQL replication between the primary SQLVM in Region 1 and the secondary SQLVM in Region 2.
Implement automatic failover to switch to the secondary SQLVM if the primary one fails. Ensure that the failover process is tested to meet the 6-hour maximum downtime requirement.
