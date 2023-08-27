note: This is the text/instructions used to setup the project
https://www.youtube.com/watch?v=K5lLj6XMawM

//	  #for http requests
//    @Bean
//	  public Function<String, String> function(){
//	      return (input) -> input.toUpperCase();
//	  }
//
//	  #for background aka pubsub topic
//    @Bean
//    public Consumer<String> consume(){
//        return input -> System.out.println("input: "+ input);
//    }
//
//    @Bean
//    public Supplier<String> supply(){
//        return () -> new String("A copy of a json string");
//    }


Google cloud functions with spring cloud function and deployment complete instructions - https://cloud.spring.io/spring-cloud-function/reference/html/gcp.html

Deploy http trigger
gcloud functions deploy function-sample-gcp-http --entry-point org.springframework.cloud.function.adapter.gcp.GcfJarLauncher --runtime java11 --trigger-http --source target/deploy --memory 512MB

Deploy pubsub topic trigger
gcloud functions deploy function-payment-sample-gcp-background --entry-point org.springframework.cloud.function.adapter.gcp.GcfJarLauncher --runtime java11 --trigger-topic payment --source target/deploy --memory 512MB

gcloud functions deploy function-sample-gcp-http \
--entry-point org.springframework.cloud.function.adapter.gcp.GcfJarLauncher \
--runtime java11 \
--trigger-http \
--source target/deploy \
--memory 512MB

Setting POM
1. Start by adding the spring-cloud-function-adapter-gcp dependency to your project.
   <dependencies>
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-function-adapter-gcp</artifactId>
   </dependency>

   ...
   </dependencies>

2. In addition, add the "spring-boot-maven-plugin" which will build the JAR of the function to deploy.

Notice that we also reference spring-cloud-function-adapter-gcp as a dependency of the spring-boot-maven-plugin.
This is necessary because it modifies the plugin to package your function in the correct JAR format for deployment on Google Cloud Functions.
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
<configuration>
<outputDirectory>target/deploy</outputDirectory>
</configuration>
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-function-adapter-gcp</artifactId>
</dependency>
</dependencies>
</plugin>

3. Finally, add the Maven plugin provided as part of the Google Functions Framework for Java. This allows you to test your functions locally via mvn function:run.

The function target should always be set to org.springframework.cloud.function.adapter.gcp.GcfJarLauncher; this is an adapter class which acts as the entry point to your Spring Cloud Function from the Google Cloud Functions platform.
<plugin>
<groupId>com.google.cloud.functions</groupId>
<artifactId>function-maven-plugin</artifactId>
<version>0.9.1</version>
<configuration>
<functionTarget>org.springframework.cloud.function.adapter.gcp.GcfJarLauncher</functionTarget>
<port>8080</port>
</configuration>
</plugin>

A full example of a working pom.xml can be found here -
https://github.com/spring-cloud/spring-cloud-function/blob/main/spring-cloud-function-samples/function-sample-gcp-http/pom.xml


Deploying a Spring Cloud Function as an HTTP Function. -
1. Create the function:

@SpringBootApplication
public class CloudFunctionMain {

	public static void main(String[] args) {
		SpringApplication.run(CloudFunctionMain.class, args);
	}

	@Bean
	public Function<String, String> uppercase() {
		return value -> value.toUpperCase();
	}
}

2. Create and Specify your configuration main class in resources/META-INF/MANIFEST.MF.
   Main-Class: com.example.CloudFunctionMain

3. Then run the function locally. This is provided by the Google Cloud Functions function-maven-plugin described in the project dependencies section.
   mvn function:run

4. Invoke the HTTP function (or from postman):
   curl http://localhost:8080/ -d "hello"

5. Deploy to GCP
   a. Start by packaging your application.
   mvn package -DskipTests=true
   note: you should see the resulting JAR in target/deploy directory. This JAR is correctly formatted for deployment to Google Cloud Functions.

   b. Next, make sure that you have the Cloud SDK CLI installed.
   From the project base directory run the following command to deploy.
   gcloud functions deploy function-spring-cloud-gcp --entry-point org.springframework.cloud.function.adapter.gcp.GcfJarLauncher --runtime java11 --trigger-http --source target/deploy --memory 512MB

   c. Invoke the HTTP function:
   curl https://REGION-PROJECT_ID.cloudfunctions.net/function-sample-gcp-http -d "hello"

