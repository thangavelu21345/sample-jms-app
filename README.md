# sample-jms-app
Sample program to check YouTube videos and transform it

# story 1 
Consume YouTube API and search for "telecom". The key fields to be taken from are - Video title and URL. Store the results into a queue for further usage.
# story 2 
Get the items from queue, stored as part of story #1 and change "Telecom" to "Teleco" and store the modified message into another queue.

# number of micro services created : 2
* telecom micro service, targeting story #1
* MessageTransformation micro service, targeting story #2

# Design patterns used
* Controller pattern in first micro service, which exposed Rest API and takes the requests from client. It then forwards to service layer.
* Service pattern - handles business logic and decision making layer of the entire atomic operation. Calls invoker to get results from each data source.
* Service Invoker - Decided which command to be invoked based on conditions.
* Command - Decouples API calling functionality and mapping functionality. Makes required API calls, get response and return results.
* Conditional routing - Enabling JMS implementation based on rules defined in application.properties.
* Singleton - creates one XStream instance and used across application. Springboot internally creates singleton for all beans by default.  

# Schema definition  
Single schema definition  is used in both micro services, as the difference is only in data.
The XML message format / schema consists of title, description of the video and url to launch the video

```xml
<xs:schema attributeFormDefault="unqualified" elementFormDefault="qualified" xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="VideoSearchMessage">
    <xs:complexType>
      <xs:sequence>
        <xs:element type="xs:string" name="title"/>
        <xs:element type="xs:string" name="desc"/>
        <xs:element type="xs:string" name="url"/>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>
```

# Pre-requisites
* Java
* Maven
  * Settings.xml pointing to public maven repo (preferred)
* ActiveMQ installed in local / JMS queue destination details https://activemq.apache.org/getting-starte
* Youtube API key  - https://developers.google.com/youtube/v3/docs

# Telecom Micro service
Configure the following details in src/main/resources/application.properties

* Server port to run the application
  * server.port=8082
* Active MQ broker details
  * spring.activemq.broker-url=tcp://localhost:61616
  * spring.activemq.user=admin
  * spring.activemq.password=admin
* Obtain youtube API key for searching videos and provide it here
  * youtube.api.key=xxxxxxxxxxxxxxxxxxxxxxxxx 
* Create JMS queue according to below property value, or update existing queue name
  * unprocessed.message.queue=jms/samplegroup.youtube.1.unprocessed.queue

**steps to compile source code**
* Navigate to root folder of telecom micro service and issue the below command in command prompt
  * mvn clean install 
* Navigate to target folder of telecom micro service and issue the below command in command prompt
  * java -jar telecom-0.0.1-SNAPSHOT.jar

Springboot application will start running after this step.

# Message transformation micro service
Configure the following details in src/main/resources/application.properties

* Server port to run the application
  * server.port=8083
* Active MQ broker details
  * spring.activemq.broker-url=tcp://localhost:61616
  * spring.activemq.user=admin
  * spring.activemq.password=admin
* Update the same JMS queue used in Telecom micro service
  * unprocessed.message.queue=jms/samplegroup.youtube.1.unprocessed.queue 
* Create a new JMS topic or use existing topic to store final results
  * processed.message.queue=jms/samplegroup.youtube.1.processed.queue

**steps to compile source code**
* Navigate to root folder of MessageTransformation micro service and issue the below command in command prompt
  * mvn clean install 
* Navigate to target folder of MessageTransformation micro service and issue the below command in command prompt
  * java -jar message-transformation-0.0.1-SNAPSHOT.jar
 
# Procedure to start the program and checking the results
* Access Telecom micro service url using the below link from chrome browser or simply curl 
 * http://localhost:8082/
 * Change port number accordingly.
 * This will trigger an API request to Telecom micro service.
 * API request will then make YouTube API call and gets video information
* Access ActiveMQ admin console using the link - http://127.0.0.1:8161/admin/queues.jsp
* Un processed messages would be in "unprocessed" queue
* Processed output messages, with "Telco" in title would be in "processed" queue. 

# Limitations and different implementation considerations
* This method will only get first 25 results from YouTube search API
* We can enable scheduling in springboot to get next 25 messages and conutied the processing (instead of one time)
 * Youtube has around 10 million results for this search, we need to set some target on the number of entries we like to process.
* Covered unit test cases for verifying functionality.
* Implemented Factory design pattern to choose queue implementation

 
 
 
