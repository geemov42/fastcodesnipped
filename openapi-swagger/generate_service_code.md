To generate server part of the swagger, you can use this configuration:

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
                <generatorName>spring</generatorName>
                <library>spring-boot</library>
                <modelNameSuffix>Dto</modelNameSuffix>
                <generateApiTests>false</generateApiTests>
                <generateModelTests>false</generateModelTests>
                <configOptions>
                    <basePackage>com.baitando.openapi.samples.gen.springbootserver</basePackage>
                    <modelPackage>com.baitando.openapi.samples.gen.springbootserver.model</modelPackage>
                    <apiPackage>com.baitando.openapi.samples.gen.springbootserver.api</apiPackage>
                    <configPackage>com.baitando.openapi.samples.gen.springbootserver.config</configPackage>
                    <dateLibrary>java8</dateLibrary>
                    <delegatePattern>true</delegatePattern>
                    <interfaceOnly>true</interfaceOnly>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

After that, you can implements :

```java
/**
 * Implementation of REST API which allows to manage accounts.
 */
@RestController
public class AccountController implements AccountsApi {

    @Override
    public ResponseEntity<GetAccountsResponseDto> getAccounts() {
        return ResponseEntity.ok(
                new GetAccountsResponseDto().items(
                        Arrays.asList(
                                createAccount("DEXXXX"),
                                createAccount("ENYYYY")))
        );
    }

    private AccountDto createAccount(String iban) {
        AccountDto account = new AccountDto();
        account.setIban(iban);
        return account;
    }
}
```