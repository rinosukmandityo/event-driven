# Event Sourcing Sample
Sample of event driven architecture with event sourcing / CQRS using gRPC, Kafka and MongoDB  
We are trying to implement simple use case for this project like order, payment, and shipment

Prerequisites
---
In this project I'm using AWS EC2 instance with AWS Linux 2 AMI.  
So my linux command will be different if you use different platform like using `apt-get` or `brew` instead of `yum`  
Here is some prerequisites for this project:  

#### Docker and Docker Compose
- If you don't have any docker installed on your machine, run this command:  
`sudo yum install -y docker`  
- Check docker version to check whether your docker is installed properly or not:  
`docker --version`  
- To check whether your docker service is running or not, run this command:  
`docker info`  
- To run docker service, run this command:  
`sudo service docker start`  
- To check if docker run properly, run this command:  
`docker ps`  
- If you have problem with permission, add your username into docker group and then re-login into your ssh:  
`sudo usermod -a -G docker ec2-user`  
- We need to install 3 kafka zookeper containers and 3 kafka server so we need to install docker compose to manage it.  
To install a different version of Compose, substitute `1.25.3` with the version of Compose you want to use  
`sudo curl -L "https://github.com/docker/compose/releases/download/1.25.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`  
- Apply executable permissions to the binary  
`sudo chmod +x /usr/local/bin/docker-compose`  
- Test the docker compose installation  
`docker-compose --version`  

#### Kafka Containers
We are using Kafka for producing and consuming an event. This is how we develop Kafka into our server using container. 
 
- Create new directory inside your $HOME directory:  
`sudo mkdir $HOME/kafka-docker`  
- Put docker-compose.yml under this project root directory into $HOME/kafka-docker directory  
- Find out your IP Address by running following command:  
`ifconfig -a`  
usually it's located in `eth0` and `inet`
- Run docker compose up to run our containers  
`MY_IP=your-ip docker-compose up -d` for example `MY_IP=172.32.42.72 docker-compose up -d`  
- To check whether your container is running or not, run this command:  
`docker-compose ps`  
- To check connection we need to install telnet  
`sudo yum install -y telnet`  
- Run following command to check zookeeper and kafka container
```
telnet localhost 12181  
telnet localhost 22181  
telnet localhost 32181  
telnet localhost 19092  
telnet localhost 29092  
telnet localhost 39092  
```
If you want to try to install Kafka into your local machine you can find tutorial how to install Kafka in your local machine.  

#### gRPC for Inter-Process Communication
- gRPC uses protobuf to communicate, in order to generate relevant files, you will need to install protoc plugin for Go:  
`go get -u github.com/golang/protobuf/proto`  
`go get -u github.com/golang/protobuf/protoc-gen-go`  
- Install gRPC package for Go  
`go get google.golang.org/grpc`


#### Basic Workflow

Here’s the basic workflow in the project:  

1. A client app post an Order to an HTTP API.
2. An **_Order Service_** receives the order via HTTP API, then executes a command onto an immutable log of events called **_Event Store_** via gRPC API.
3. The **_Event Store_** API executes the command and then publishes an event `order-created` to Kafka server to let other services know that an event is created.
4. The **_Payment Service_** subscribes the event `order-created`, then make the payment, and then create another event `order-payment-debited` via **_Event Store_** gRPC API.
5. The Query syncing workers (orderquery-store1 and orderquery-store2 as queue subscribers) are also subscribing the event `order-created` that synchronise the data models to provide state of the aggregates for query views.
6. The **_Event Store_** API executes a command onto **_Event Store_** to create an event `order-payment-debited` and publishes an event to Kafka server to let other services know that the payment has been debited.
7. The **_Shipping Service_** finally approves the order and ship the thing.

How to run
---
#### TO DO
