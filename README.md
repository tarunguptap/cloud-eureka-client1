# cloud-eureka-client1
This is Eureka client and rest api will return the word "Welcome Tarun"


    **Part 2, create clients** 
    
 '''   
    In this next section we will create several client applications that will work together to compose a sentence.  The sentence will be "Welcome Tarun".  2 services will get the one word each, and a 3rd service will assemble them into a sentence.
'''

1. Create a new Spring Boot web application.  
  - Name the project "cloud-eureka-client1”, and use this value for the Artifact.  
  - Use JAR packaging and the latest versions of Java.  
  - Use Boot version 1.5.x or the latest stable version available.  
  - Add actuator,  and web as a dependencies.
  - Add a dependency for group "org.springframework.cloud" and artifact "spring-cloud-starter-netflix-eureka-client".

2. Modify the Application class.  Add @EnableDiscoveryClient.

3. Save an application.yml (or properties) file in the root of your classpath (src/main/resources recommended).  Add the following key / values (use correct YAML formatting):
  - eureka.client.serviceUrl.defaultZone=http://localhost:8011/eureka/
  - words=Welcome
  - server.port=${PORT:${SERVER_PORT:0}}
(this will cause a random, unused port to be assigned if none is specified)

4. Save a bootstrap.yml (or properties) file in the root of your classpath.  Add the following key / values (use correct YAML formatting):
  - spring.application.name=cloud-config-client1

5. Add a Controller class
  - Place it in the 'demo' package or a subpackage of your choice.
  - Name the class anything you like.  Annotate it with @RestController.
  - Add a String member variable named “words”.  Annotate it with @Value("${words}”).
  - Add the following method to serve the resource (optimize this code if you like):
  ```
    @GetMapping("/")
    public @ResponseBody String getWord() {
      return "Welcome";
    }
  ```
------------------------------------------------------------------
6. Repeat steps 1 thru 5 (copy the entire project if it is easier), except use the following values:
  - Name of application: “cloud-eureka-client2”
  - spring.application.name: “cloud-eureka-client2”
  - Add a String member variable named “words” in controller. Annotate it with @Value("${words}”).
  - Keep words: “Tarun” in cloud config **(We will see the change in the value in cloud config will result into client3 output  )**

7. Create a new Spring Boot web application.  
  - Name the application “cloud-eureka-client3”, and use this value for the Artifact.  
  - Use JAR packaging and the latest versions of Java and Boot.  
  - Add actuator and web as a dependencies.  
  - Alter the POM (or Gradle) just as you did in step 8. 

8. Add @EnableDiscoveryClient to the Application class.  

9. Save an application.yml (or properties) file in the root of your classpath (src/main/resources recommended).  Add the following key / values (use correct YAML formatting):
  - eureka.client.serviceUrl.defaultZone=http://localhost:8011/eureka/
  - server.port: 8020

10. Add a Controller class to assemble and return the sentence:
  - Name the class anything you like.  Annotate it with @RestController.
  - Use @Autowired to obtain a DiscoveryClient (import from Spring Cloud).
  - Add the following methods to serve the sentence based on the words obtained from the client services. (feel free to optimize / refactor this code as you like:
  ```
    @GetMapping("/sentence")
    public @ResponseBody String getSentence() {
      return 
        getWord("CLOUD-EUREKA-CLIENT1") + " "
        + getWord("CLOUD-EUREKA-CLIENT2") + "."
        ;
    }
    
    public String getWord(String service) {
      List<ServiceInstance> list = client.getInstances(service);
      if (list != null && list.size() > 0 ) {
        URI uri = list.get(0).getUri();
	if (uri !=null ) {
	  return (new RestTemplate()).getForObject(uri,String.class);
	}
      }
      return null;
    }
  ```

11. Run all of the word services and sentence service.  (Run within your IDE, or build JARs for each one (mvn clean package) and run from the command line (java -jar name-of-jar.jar), whichever you find easiest).  (If running from STS, uncheck “Enable Live Bean support” in the run configurations).  Since each service uses a separate, random port, they should be able to run side-by-side on the same computer.  Open [http://localhost:8020/sentence](http://localhost:8020/sentence) to see the completed sentence.  





---

**BONUS - Refactor to use Spring Cloud Config Server.**  

  We can use Eureka together with the config server to eliminate the need for each client to be configured with the location of the Eureka server

12. Add a new file to your GitHub repository (the same repository used in the last lab) called “application.yml” (or properties).  Because this file is named "application.*", the properties set within apply to all clients of the Config server.  This is great for us as we want all clients to find the Eureka server.  Add the following key / values (use correct YAML formatting):
  - eureka.client.serviceUrl.defaultZone=http://localhost:8010/eureka/ 

13. Open the common-config-server project.   Alter the application.yml to point to your own github repository.  Save all and run this server.  (You can use it as the config server for almost all of the remaining labs in this course.)  

14. In each client project, add an additional dependency for spring-cloud-config-client: 

```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
```

15. Edit each client application’s application.properties file.  Remove the eureka client serviceUrl defaultZone key/value.  Now we will get this from the config server.

16. In each client project, add the following key/value to bootstrap.yml (or bootstrap.properties), using correct YAML formatting: 
  - spring.cloud.config.uri: http://localhost:8001.
  
17. Make sure the Eureka server is still running.  Start (or restart) each client. Open [http://localhost:8020/sentence](http://localhost:8020/sentence) to see the completed sentence.

18. If you like, you can experiment with moving the “words” properties to the GitHub repository so they can be served up by the config server.  You’ll need to use separate profile sections within the file (yml) or files with names that match the application names (yml or properties).  A single application.yml file would look something like this:

  ```
  ---
  spring:
    profiles: firstword
  lucky-word: Hello
  
  ---
  spring:
    profiles: secondword
  lucky-word: Tarun

  ```

  **BONUS - Refactor to use multiple Eureka Servers**  
    
  To make the application more fault tolerant, we can run multiple Eureka servers.  Ordinarily we would run copies on different racks / data centers, but to simulate this locally do the following:

19.  Stop all of the running applications.

20.  Edit your computer's /etc/hosts file (c:\WINDOWS\system32\drivers\etc\hosts on Windows).  Add the following lines and save your work:

  ```
  # START section for Microservices with Spring Course
  127.0.0.1       eureka-primary
  127.0.0.1       eureka-secondary
  127.0.0.1       eureka-tertiary
  # END section for Microservices with Spring Course
  ```

21.  Within the cloud-eureka-server project, add application.yml with multiple profiles:
primary, secondary, tertiary.  The server.port value should be 8011, 8012, and 8013 respectively.  The eureka.client.serviceUrl.defaultZone for each profile should point to the "eureka-*" URLs of the other two; for example the primary value should be: http://eureka-secondary:8012/eureka/,http://eureka-tertiary:8013/eureka/

22.  Run the application 3 times, using -Dspring.profiles.active=primary (and secondary, and tertiary) to activate the relevant profile.  The result should be 3 Eureka servers which communicate with each other.

23.  In your GitHub project, modify the application.properties eureka.client.serviceUrl.defaultZone to include the URIs of all three Eureka servers (comma-separated, no spaces).

24.  Start all clients.  Open [http://localhost:8020/sentence](http://localhost:8020/sentence) to see the completed sentence.

25.  To test Eureka’s fault tolerance, stop 1 or 2 of the Eureka instances.  Restart 1 or 2 of the clients to ensure they have no difficulty finding Eureka.  Note that it may take several seconds for the clients and servers to become fully aware of which services are up / down.  Make sure the sentence still displays.


**Reflection:**  There are a number of remaining issues with the current application which can be addressed.

1. These services contain duplicated code.  This was done only to make the instructions straightforward.  You can easily implement this system using a single ‘word’ server which selects different words based on a @Profile.  (This is done in the solution)

2. What happens if one of the “word” servers is down?  Right now our entire application will fail.  We will improve this later when we discuss circuit breakers with Hystrix.

3. You may be puzzling about which properties should be set in bootstrap.yml (or properties) vs application.yml (or properties).  Very simply, bootstrap.* is loaded early and is used when contacting the Config server, so it should contain spring.cloud.config.uri, spring.application.name, and generally nothing else.  All other properties can be set later when application.* is loaded.  

4. To improve performance, can we run each of the calls in parallel?  We will improve this later when discussing Ribbon and Hystrix.

5. We will see an alternative to the RestTemplate when we discuss Feign.
