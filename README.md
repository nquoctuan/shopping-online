# Technical stack
- Framework: SpringBoot, SpringCloud 
- CI/CD: Kubernetes, Gitlab (GitLab Runner) and Ansible
- API Management: JavaDoc Or Swagger
- Server Framework: SpringBoot, JPA
- Security: JWT
- Databases: MySQL and Redis
- Message Broker: Kafka
- Log System: Elasticsearch (ELK), LogStash and Kibana
- Gradle to manage master project and sub projects

# Prerequisites 
- Java Development Kit LTS (>= 8) Or Kotlin 
- Gradle
- Docker

# Components
The entire system has 6 components, and each component has own responsibility. My business feature in this design is **Shopping Cart and Order Management**

![Components](./images/ShoppingOnline-Shopping%20Online.drawio.png)


# High Level Design
Based on C4Model concept, the view starts from Container Diagram to Component Diagram. Besides, the main flow also draws in sequence actions.

## Container Diagram
![Container Diagram](./images/ShoppingOnline-Container%20Diagram.drawio.png)
## Component Diagram
![Component Diagram](./images/ShoppingOnline-Component%20Diagram.drawio.png)

## Design Pattern
- Database Per Service
- Choreography-based SAGA
- Event Sourcing

# Database Design (ERD)
![Class Diagram](./images/ShoppingOnline-Class%20Diagram.drawio.png)

# Sequence Diagram of An Order
![Sequence Diagram](./images/ShoppingOnline-Sequence%20Diagram%20.drawio.png)

Other diagrams has not mentioned, but utilizes Event Messages to perform their jobs 
- SMS Notification 
- Email Notification

# Log Operation
As mentioned on Component Diagram, each service has own logs and sync them by LogStash and Kafka
- Every order should have an unique request Id, similar with Correlation_Id. This Id goes through the entire services to fulfil their jobs.
- Service Logs can be queried on ELK and visualized on Kibana to estimate its performance


# Security
- AWS Route53
- JWT Token
- Internal API Gateway (nginx)
- External API Gateway (nginx) (whilelist: partner Ids)

# Scalablity 
![Scale out/in](./images/ShoppingOnline-Scale%20Out_In.drawio.png)

# Concerns
The model is complex and what if there is an exception on code base or even network interrupt, therefore the compensation is a must
## Compensation 
![State Change](./images/ShoppingOnline-State%20Change.drawio.png)

The sequence of state are stored in Event Store based on the Event Sourcing pattern
- Auto Revert
    + Every event state change is recorded, so an additional application is built up to perform reserve 
- Manual Revert
    + Consequence of wrong info could stuck the flows, so manually updating data is an option to keep the flow going 

## Auto Scale Out Wrong because of wrong information
- Messed-up CPU/RAM usage could be from an error on code base that could be a misinformation to cause scaling out wrongly. Therefore, the metrics needs to collect service logs to decide threshold.
    + Metric-Server collects info from CAdvisor(kubelet)
    + Metric-Server also query to ELK on the pod that has problems with CPU/RAM hight usage

# Pros and Cons
## Pros
1. Database Per Service, avoid conflict data because of cross accessing - loose coupling  
2. Kafka Message Broker, also a data streaming, meanings the programing is so nature
3. Able to scale out/in  

## Cons
1. It is complex model, so needs to involve many parts to build up, including programming and additional tools to fit in scalability
2. Compensation needs to consider serious