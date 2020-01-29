# best-practice-java
best-practice-java
- Strings util Strings.isNullOrEmpty
- use javax.ws.rs.core.Response;

https://dzone.com/articles/how-to-identify-blocking-code-using-asyncprofiler?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

# Handling Service Timeouts Using Istio
https://dzone.com/articles/handling-service-timeouts-using-istio?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

https://dzone.com/articles/advancing-application-performance-with-nvme-storag-1?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

## Springboot
- @ConditionalOnProperty
https://www.baeldung.com/spring-boot-custom-auto-configuration

- Use retrofit for async call for non blocking request - retrofit2.Call; retrofit2.Response;

- Use Lombok
@Getter
@ToString
@NoArgsConstructor
@Builder
@AllArgsConstructor
@EqualsAndHashCode
@JsonIgnoreProperties(ignoreUnknown = true)

- Enum type nameware 
public interface NameAware {

    String getName();

}


- Use spring org.springframework.core.env.Environment env for get Env variables.


- Use okhttp3.mockwebserver
import okhttp3.mockwebserver.MockResponse;
import okhttp3.mockwebserver.MockWebServer;
import okhttp3.mockwebserver.QueueDispatcher;
import okhttp3.mockwebserver.RecordedRequest

--Validation
https://www.baeldung.com/javax-validation

- Spring profile 
+@Profile("!profile-service")
+@Primary


### https://www.ionos.com/community/hosting/redis/using-redis-in-docker-containers/
## Running Redis in a Docker Container
sudo docker run --name my-redis-container -d redis 

## Connecting to Redis Running in a Docker Container
sudo docker run --name my-redis-application --link my-redis-container:redis -d centos

sudo docker run -it --name my-redis-cli --link my-redis-container:redis --rm redis redis-cli -h redis -p 6379

Connect to a Redis Container From a Remote Server
sudo docker run --name my-redis-container -p 7001:6379 -d redis



##Spring 
https://www.baeldung.com/spring-inject-prototype-bean-into-singleton

Own certificate authority
Create a simple, internal CA for your microservice architecture or integration testing
https://opensource.com/article/19/4/certificate-authority


@ConditionalOnProperty(name = "tes.svcs.key.cache.enabled", havingValue = "true", matchIfMissing = true)

##JWT, JWE, JWS
https://medium.facilelogin.com/jwt-jws-and-jwe-for-not-so-dummies-b63310d201a3

##DB

spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
spring.datasource.url=jdbc:oracle:thin:@host:port/db.service
spring.datasource.username=$login}
spring.datasource.password=${password}
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.maximum-pool-size=50
spring.datasource.hikari.connection-test-query=select 1 from dual
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.connection-timeout=3000
spring.datasource.hikari.validation-timeout=3000
spring.datasource.hikari.login-timeout=6000
spring.datasource.hikari.max-lifetime=180000
spring.datasource.hikari.data-source-properties.cachePrepStmts=true
spring.datasource.hikari.data-source-properties.prepStmtCacheSize=100
spring.datasource.hikari.data-source-properties.prepStmtCacheSqlLimit=2048
spring.datasource.hikari.data-source-properties.useServerPrepStmts=true

##Service Discovery
https://www.youtube.com/watch?v=Dd9-UxrNiD0

## IOT
https://devpost.com/software/iot-smart-kitchen-knob
