To generate client part of the swagger, you can use this configuration:

```xml
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>4.3.1</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/../../spec/banking.yaml</inputSpec>
                <generatorName>java</generatorName>
                <modelNameSuffix>Dto</modelNameSuffix>
                <generateApiTests>false</generateApiTests>
                <generateModelTests>false</generateModelTests>
                <library>resttemplate</library>
                <configOptions>
                    <basePackage>com.baitando.openapi.samples.gen.springbootclient</basePackage>
                    <modelPackage>com.baitando.openapi.samples.gen.springbootclient.model</modelPackage>
                    <apiPackage>com.baitando.openapi.samples.gen.springbootclient.api</apiPackage>
                    <invokerPackage>com.baitando.openapi.samples.gen.springbootclient.client</invokerPackage>
                    <dateLibrary>java8</dateLibrary>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

After that, you can implements :

```java
@Service
public class RestAccountService implements AccountService {

    private final DefaultApi apiClient;

    public RestAccountService(DefaultApi apiClient) {
        this.apiClient = apiClient;
    }

    @Override
    public List<Account> getAccounts() {
        return AccountConverter.convertFromDto(apiClient.getAccounts());
    }
}
```