# best-practice-java
best-practice-java

https://dzone.com/articles/how-to-identify-blocking-code-using-asyncprofiler?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

# Handling Service Timeouts Using Istio
https://dzone.com/articles/handling-service-timeouts-using-istio?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

https://dzone.com/articles/advancing-application-performance-with-nvme-storag-1?edition=485324&utm_source=Zone%20Newsletter&utm_medium=email&utm_campaign=performance%202019-07-05

## Springboot
@ConditionalOnProperty
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

-Use spring org.springframework.core.env.Environment env for get Env variables.

- Use okhttp3.mockwebserver
import okhttp3.mockwebserver.MockResponse;
import okhttp3.mockwebserver.MockWebServer;
import okhttp3.mockwebserver.QueueDispatcher;
import okhttp3.mockwebserver.RecordedRequest
