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
* Command - Decouples API calling functionality and mapping functionality. Makes required API calls, get response and return back.
* Conditional routing - Enabling JMS implementation based on rules defined in application.properties.
* Singleton - creates one XStream instance and used across application. Springboot internally creates singleton for all beans by default.  

# Schema defenition 
Single schema defenition is used in both micro services, as the difference is only in data.
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

# Pre-requsites
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




