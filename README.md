# cloud-eureka-client1
This is Eureka client and rest api will return the work "Welcome"


    **Part 2, create clients**  
 '''   
    In this next section we will create several client applications that will work together to compose a sentence.  The sentence will be "Welcome Tarun".  2 services will get the one word each, and a 6th service will assemble them into a sentence.
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
        getWord("LAB-4-SUBJECT") + " "
        + getWord("LAB-4-VERB") + " "
        + getWord("LAB-4-ARTICLE") + " "
        + getWord("LAB-4-ADJECTIVE") + " "
        + getWord("LAB-4-NOUN") + "."
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
