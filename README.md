# Spring Cloud Session 5 Microservices Documentation
In this tutorial we are going to learn how do we document REST API microservices using **OpenAPI/Swagger**.
- Swagger is one of the mostly used standard while documenting REST API services.
- We will use SpringFox to produce documentation.

Overview
- We will be using SpringFox to document employee-api , payroll-api
- API documentation will be viewed via gateway
- Access Swagger UI of employee-api and make few rest calls

Important Notes
- Please refer my previous sessions to understand how these microservices work. In this session we will focus on documenting
API and accessing API documentation. 

# Source Code 
``` git clone https://github.com/balajich/spring-cloud-session-2-microservices-dynamic-ports.git ``` 
# Video
[![Spring Cloud Session 2 Microservices Dynamic ports](https://img.youtube.com/vi/5WuallBaMnw/0.jpg)](https://www.youtube.com/watch?v=5WuallBaMnw)
- https://youtu.be/5WuallBaMnw
# Architecture
![architecture](architecture.png "architecture")
# Prerequisite
- JDK 1.8 or above
- Apache Maven 3.6.3 or above
# Clean and Build
- Java
    - ``` cd spring-cloud-session-2-microservices-introduction ``` 
    - ``` mvn clean install ```
 
# Running components
- Registry: ``` java -jar .\registry\target\registry-0.0.1-SNAPSHOT.jar ```
- Employee API: ``` java -jar .\employee-api\target\employee-api-0.0.1-SNAPSHOT.jar ```
- Payroll API: ``` java -jar .\payroll-api\target\payroll-api-0.0.1-SNAPSHOT.jar ```
- Gateway: ``` java -jar .\gateway\target\gateway-0.0.1-SNAPSHOT.jar ``` 

# Accessing Swagger UI  of employee and payroll api
- Swagger UI of employee api via gateway: ```  http://localhost:8080/employee/swagger-ui.html ```
- Swagger UI of payroll api via gateway: ```  http://localhost:8080/payroll/swagger-ui.html ```
- OpenAPI descriptions of employee api ``` curl -s -L http://localhost:8080/employee/v3/api-docs/  ```
- OpenAPI descriptions of payroll api ``` curl -s -L http://localhost:8080/payroll/v3/api-docs/  ```

# Scale up application
Run multiple instances of employee-api,payroll-api to handle more volume of requests.
- New Employee API instance: ``` java -jar .\employee-api\target\employee-api-0.0.1-SNAPSHOT.jar ```
- New Payroll API instance: ``` java -jar .\payroll-api\target\payroll-api-0.0.1-SNAPSHOT.jar ```

# Code
*Registry(Service Registry)* is a Spring Boot application that uses Eureka Server. Snippet of **RegistryApplication**
```java
@SpringBootApplication
@EnableEurekaServer
public class RegistryApplication {

    public static void main(String[] args) {
        SpringApplication.run(RegistryApplication.class, args);
    }
}
```
*Employee API* is a simple spring boot based rest-api. Snippet of **EmployeeController**
```java
 // Initialize database
    private static final Map<Integer, Employee> dataBase = new HashMap<>();
    static {
        dataBase.put(100, new Employee(100,"Alex"));
        dataBase.put(101, new Employee(101,"Tom"));
    }


    @RequestMapping(value = "/employee/{employeeId}", method = RequestMethod.GET)
    public Employee getEmployeeDetails(@PathVariable int employeeId) {
        logger.info(String.format("Getting Details of Employee with id %s",employeeId ));
        return dataBase.get(employeeId);
    }
```
*Employee API* users Eureka Client , which discovers and registers with Server. Snippet of **EmployeeApiApplication**. It
binds to random port and registers with Eureka Server using name,dynamic id and random value.
```java
@SpringBootApplication
@EnableEurekaClient
public class EmployeeApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(EmployeeApiApplication.class, args);
    }

}
```
Snippet of employee api **application.yml**
```yaml
server:
  port: ${PORT:0}
eureka:
  instance:
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```
*Gateway* is a Spring boot application which uses Spring Cloud Load Balancer for Client Side Load balancing and Eureka Client to
discover healthy microservices. Snippet of **GatewayApplication**
```java
@SpringBootApplication
@EnableEurekaClient
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```
It is mainly configuration driven  **application.yml**  
```yaml
cloud:
    loadbalancer:
      ribbon:
        enabled: false
    gateway:
      routes:
        - id: employee-api
          uri: lb://EMPLOYEE-API
          predicates:
            - Path=/employee/**
        - id: payroll-api
          uri: lb://PAYROLL-API
          predicates:
            - Path=/payroll/**
```

# Next Steps
- How micoservice communicate with each other,Inter microservice communication

# References
- Spring Microservices in Action by John Carnell 
- Hands-On Microservices with Spring Boot and Spring Cloud: Build and deploy Java microservices 
using Spring Cloud, Istio, and Kubernetes -Magnus Larsson 

# Next Tutorial
- https://github.com/balajich/spring-cloud-session-3-inter-microservice-communication-sync