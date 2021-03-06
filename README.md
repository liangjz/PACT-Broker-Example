# PACT-Broker-Example
PACT is a contract testing tool. 
It helps to test communications between services/microservices.
But the main idea of this repository is different. 
This is manual how to use PACT Broker with Maven in Spring Boot.


If you want more information about PACT - follow [next](https://docs.pact.io/ "PACT") docs.

## Consumer
[*Consumer*](https://github.com/SlandShow/PACT-Broker-Example/tree/master/Consumer "Consumer") - A client that wants to receive some data.

Here is an example of how consumer share pact files on broker:
```
          <plugin>
                <groupId>au.com.dius</groupId>
                <artifactId>pact-jvm-provider-maven_2.12</artifactId>
                <version>3.5.11</version>
                <executions>
                    <execution>
                        <phase>install</phase>
                        <goals>
                            <goal>publish</goal>
                        </goals>
                        <!-- Run broker on local machine with docker using docker-compose file -->
                        <configuration>
                            <pactDirectory>${basedir}/target/pacts</pactDirectory>
                            <pactBrokerUrl>http://localhost:80</pactBrokerUrl>
                            <projectVersion>1.1</projectVersion>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

[*Consumer test*](https://github.com/SlandShow/PACT-Broker-Example/blob/master/Consumer/src/test/java/comslandshow/demo/DemoApplicationTests.java "consumer test") generate json in target file, when it exetutes via Spring Boot context. And next step on maven plugin, which share this json on broker. It's pretty easy and can be executed in CI Job.

## Usage of Consumer

Build Comsumer:
```
mvn clean install
```

Then run:
```
mvn spring-boot:run
```

Consumer application have [swagger-ui](https://swagger.io/tools/swagger-ui/ "Swagger UI") is available on `http://localhost:8080/swagger-ui.html`

Example of Consumer interaction on broker:
![broker not found](https://i.ibb.co/G3677qp/2019-06-02-15-56-38.png)

## Provider
[*Provider*](https://github.com/SlandShow/PACT-Broker-Example/tree/master/Provider "Provider") - a service or server that provides the data.

Provider must verify interaction from broker. If interaction succesfully verified - broker mark it via `green` color, if not - via `red` color.
For more info, read [next](https://github.com/SlandShow/PACT-Broker-Example/new/master?readme=1#broker) part of documentation.

## Usage of Provider

Build Provider:
```
mvn clean install
```

If you want to verify interaction on briker, first run application:
```
mvn spring-boot:run
```

And in additional terminal use plugin to verify:
```
mvn pact:verify
```
It's important, because naven plugin for verify PACT works upon of Spring Boot application context. 
The oddity is that when performing Spring Boot tests with a raised context, the consumer still takes the JSON from the broker. Only except for the fact that he does not mark them as "verified".

Provider application have [swagger-ui](https://swagger.io/tools/swagger-ui/ "Swagger UI") is available on `http://localhost:7073/swagger-ui.html`

## Broker
The Pact Broker is an application for sharing for consumer driven contracts and verification results. It is optimised for use with "pacts" (contracts created by the Pact framework), but can be used for any type of contract that can be serialized to JSON.

More info available [here](https://github.com/pact-foundation/pact_broker "Broker").

For example.

1. Empty Broker:

![broker not found](https://i.ibb.co/cJGSCSk/2019-06-01-00-55-45.png)

2. Broker with consumer interaction:

![broker not found](https://i.ibb.co/n8C8Pmg/2019-06-01-00-55-56.png)

3. Broker with verified consumer interaction:

![broker not found](https://i.ibb.co/rsRdRMp/2019-06-01-01-05-55.png)

## HTTP PACT publishing & retrieving
We can use [HTTP REST calls](https://github.com/pact-foundation/pact_broker/wiki/Publishing-and-retrieving-pacts "PACT REST HTTP") to broker. You most likely won't need to get your hands dirty with curl however, as pact publishing is built in to most of the native pact libraries already.

For example:

1. `cd Consumer`

2. In console use next cUrl:
```
curl -v -XPUT -H "Content-Type: application/json" -d@foo-bar.json http://localhost:80/pacts/provider/Bar/consumer/Foo/version/1.0.0+4jvh387gj3
```

## How to run PACT Broker on my local machine?
Just use [docker compose](https://docs.docker.com/compose/overview/ "docker compose"):
```
docker-compose --file docker-compose-pact.yaml  up --build
```
And then open:
[http://localhost:80](http://localhost:80)

## Swagger codegen
In this example swagger definition used for API generation. Here we generate YAML swagger definition for Provider (swagger file located in target folder):
```
             <plugin>
                <groupId>com.github.kongchen</groupId>
                <artifactId>swagger-maven-plugin</artifactId>
                <version>3.1.7</version>
                <configuration>
                    <apiSources>
                        <apiSource>
                            <springmvc>true</springmvc>
                            <locations>
                                <location>com.slandshow.demo</location>
                            </locations>
                            <outputFormats>yaml</outputFormats>
                            <schemes>
                                <scheme>http</scheme>
                            </schemes>
                            <info>
                                <title>PACT Provider</title>
                                <version>1.0</version>
                                <description>Nothing</description>
                            </info>
                            <swaggerDirectory>${basedir}/target/swagger/temp</swaggerDirectory>
                            <swaggerApiReader>com.github.kongchen.swagger.docgen.reader.SpringMvcApiReader
                            </swaggerApiReader>
                            <swaggerFileName>internal-swagger</swaggerFileName>
                        </apiSource>
                    </apiSources>
                </configuration>
                <executions>
                    <execution>
                        <configuration>
                            <apiSources>
                                <apiSource>
                                    <locations>
                                        <location>com.slandshow.demo</location>
                                    </locations>
                                    <swaggerFileName>internal-swagger</swaggerFileName>
                                </apiSource>
                            </apiSources>
                        </configuration>
                        <id>internal-generation</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

And in Consumer we generate code via maven plugin according on this YAML swagger file:
```
            <plugin>
                <groupId>io.swagger</groupId>
                <artifactId>swagger-codegen-maven-plugin</artifactId>
                <version>2.2.3</version>
                <executions>
                    <execution>
                        <id>generate-swagger-javaclient</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                        <configuration>
                            <!-- In enterprise projects this swagger yaml specification must be verified from Nexus-->
                            <!-- But here it just hardcored -->
                            <inputSpec>${basedir}/src/main/resources/api.yaml</inputSpec>
                            <language>java</language>
                            <generateApiTests>false</generateApiTests>
                            <generateModelTests>false</generateModelTests>
                            <library>resttemplate</library>
                            <modelPackage>${project.groupId}.client.model</modelPackage>
                            <apiPackage>${project.groupId}.client.api</apiPackage>
                            <invokerPackage>${project.groupId}.client.invoker</invokerPackage>
                            <importMappings>
                                <importMapping>ByteArrayResource=org.springframework.core.io.ByteArrayResource
                                </importMapping>
                            </importMappings>
                            <configOptions>
                                <dateLibrary>java8-localdatetime</dateLibrary>
                            </configOptions>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

So, Provider generates [RestTemplate](https://www.baeldung.com/rest-template "RestTemplate") api for Consumer. We can invoke Provider from Consumer using this generated client api. This is a main idea for communication between two microservices.

## How can i use this example?

1. [Run Broker](https://github.com/SlandShow/PACT-Broker-Example#how-to-run-pact-broker-on-my-local-machine "Run Broker")

2. [Run Consumer](https://github.com/SlandShow/PACT-Broker-Example/blob/master/README.md#usage-of-consumer "How to run consumer")

3. [Run Provider](https://github.com/SlandShow/PACT-Broker-Example#usage-of-provider "How to run provider")
