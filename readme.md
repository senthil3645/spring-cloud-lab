# Spring Cloud Lab

A simple Spring Cloud "Slot Machine" :cherries: :cherries: :cherries: application, demoing cloud and microservice patterns.  

The lab consists of a number of interconntected components including:
- A Slot Machine Service
- A Random Number Service
- A Service Registry (using Eureka)

## 0 - Initial setup
### Create the root project directory 
```sh
$ mkdir spring-cloud-lab
```
Subsequent projects / services will be places within this folder as they are generated using the Spring Initlizer quickstart generator.

A number of seperate Java projects will be created. 

To faciliate testing it is a good idea to keep a seperate console and IDE window open for each Java project.

## 1 - Create the Random Number Service  
### 1.1 - Generate a Spring Boot Template from https://start.spring.io
Stick to the default settings, however update:
- artifact name to random-number-service
- for dependencies add Web, Actuator  

### 1.2 - Download the project folder into our spring-cloud-lab directory
### 1.3 - Open the project by importing the generated pom.xml with your IDE of choice
### 1.4 - Update the code base

Implement a /randomNumber endpoint that returns a random integer. 

This can be done by creating a RandomNumberController Java class file with:


```java
@RestController
public class RandomNumberController {

    @RequestMapping
    public int getRandomNumber(){
        return new Random().nextInt();
    }
}
```
### 1.5 - Run the application (from /spring-cloud-lab/random-number-service)
```sh
$ ./mvnw spring-boot:run
```

### 1.6 - Test the /randomNumber endpoint at localhost:8080/randomNumber

You should see a randomly generated number response. 

## 2 - Create the Slot Machine Service
### 2.1 - Generate a Spring Boot Template from https://start.spring.io
Stick to the default settings, however update:
- artifact name to slot-machine-service
- for dependencies add Web, Actuator  

### 2.2 - Download the project folder into our spring-cloud-lab directory
### 2.3 - Open the project by importing the generated pom.xml with your IDE of choice
### 2.4 - Update the code base

Update the **application.properties** file to not have a conflicting port with our Random Number Service. 

```properties
server.port=8081
```

Implement a /spin endpoint that returns 3 random slot machine symbols. i.e. Cherry, Bar, Orange, Plum, etc

This can be done by creating a SlotMachineController class file with:


```java
@RestController
public class SlotMachineController {

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping
    public String spin(){

        String[] slotMachineSymbols = {"Cherry", "Bar", "Orange", "Plum"};
        
        return "Cherry Cherry Cherry"; 
        //TODO update return statement to return 3 random slot machine symbols  
        // using the below example usage of the Random Number Service .. numerous ways exist of doing this ! 
        // EXAMPLE int randomNumber = restTemplate.getForObject("http://localhost:8080/randomNumber", Integer.class);
    }
    
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

__Implenent your own solution at the TODO mark or scroll down for one such solution__

```java
return IntStream.range(0, 3).mapToObj(x-> {
            int randomNumber = restTemplate.getForObject("http://localhost:8080/randomNumber", Integer.class);
            return slotMachineSymbols[Math.abs(randomNumber%slotMachineSymbols.length)];}
        ).collect(Collectors.joining(" "));
```


### 2.5 - Run the application (from /spring-cloud-lab/slot-machine-service)
```sh
$ ./mvnw spring-boot:run
```

### 2.6 - Test the spin endpoint at localhost:8081/spin

You should get a randomly generated slot machine response.

## 3 - Create the Service Registry  
### 3.1 - Generate a Spring Boot Template from https://start.spring.io
Stick to the default settings, however update:
- artifact name to service-registry
- for dependencies add EurekaServer, Actuator 

### 3.2 - Download the project folder into our spring-cloud-lab directory
### 3.3 - Open the project by importing the generated pom.xml with your IDE of choice
### 3.4 - Update the code base
We need to add the **@EnableEurekaServer** annoation to the ServiceRegistryApplication class file 

```java
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
```

Update the **application.properties** file to turn-off self-registry and configure Euruka to use the standard Euraka port of 8761 instead of the Spring Boot default of 8080

```properties
server.port=8761 
eureka.client.registerWithEureka=false
eureka.client.fetchRegistry=false
```
### 3.5 - Run the application (from /spring-cloud-lab/service-registry)
```sh
$ ./mvnw spring-boot:run
```

### 3.6 - Open the Service Registry Eureka Dashboard at localhost:8761

You should see the Eureka main page.

## 4 - Update the Random Number and Slot Machine Services to use the Service Registry  
### 4.1 - Update both service's pom.xml build scripts with requireed Eureka dependencies. 

Add the Eureka dependency:

```xml
    <dependencies>
        <!-- exisitng dependencies are here -->
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
    </dependencies>
```

Ensure the Spring-Cloud dependency block is present:

```xml
</dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
<build>
```

Addd the spring-cloud-version definition if needed:

```xml
<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR2</spring-cloud.version>
    </properties>
```

### 4.2 - Update the code base 
Add the **@EnableEurekaClient** annoation to both service's application class files .. i.e. : 

```java
@SpringBootApplication
@EnableEurekaClient
public class SlotMachineServiceApplication {
```
Explicity set the spring application name property in the application properties file for each service :

Random Number Service 

```properties
spring.application.name=random-number-service 
```

Slot Machine Service

```properties
spring.application.name=slot-machine-service 
```

Update the Slot Machine Service Rest Template code for auto discovery: 

Add the **@LoadBalanced** annoation to the RestTemplate Bean 
```java
    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
```

Update the RestTemplate call to use the service name (random-number-service) instead of address and port (localhost:8080). 

```java
restTemplate.getForObject("http://random-number-service/randomNumber"
```

### 4.3 - Restart both services 

For both service directories .. kill the existing processes (Ctrl-C) and restart:
```sh
$ ./mvnw spring-boot:run
```

### 4.4 - Confirm registration of both Services in the Eureka portal at localhost:8761

You should see both the Random-Number-Service and Slot-Machine-Service listed under __Instances currently registered with Eureka__

### 4.5 - Test the spin endpoint at localhost:8081/spin

You should get a randomly generated slot machine response.

## 5 - Enable a Circuit Breaker

### 5.1 - Add the Hystrix dependency to the Slot Machine Service dependecy block in the pom.xml 

```xml
    <dependencies>
        <!-- exisitng dependencies are here -->
        
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
    </dependencies>
```

### 5.2 - Update the code base

Add the **@EnableCircuitBreaker** annotation to the SlotMachineServiceApplication class

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class SlotMachineServiceApplication {
```

Add the **@HystrixCommand(fallbackMethod = "defaultResult")** annotation the the spin method
```java
@HystrixCommand(fallbackMethod = "defaultResult")
@RequestMapping
    public String spin(){
```

Implement a default result method in the Slot Machine Controller
```java
private String defaultResult() {
        return "? ? ?";
    }
```

### 5.3 - Restart the Slot Machine Serivce 

```sh
$ ./mvnw spring-boot:run
```

### 5.4 - Confirm Circuit Breaker functionality 

Terminate the Random Number Service (Ctrl-C)

Attempt the call the /spin endpoint 

You should see the default fail back response of "? ? ?" in lieu of a complete failure

Restart the Random Number Service 

Try the /spin endpoint again .. eventually it will reenable commeunication with the now health Random Number Service -- the default wait time is 5seconds.