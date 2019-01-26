## 

A gRPC mock server which utilizes WireMock for handling mock requests.

gireMock utilizes the excellent project from salesforce - [jProtoc](https://github.com/salesforce/grpc-java-contrib/tree/master/jprotoc) to generate code as part of protobuf maven plugin code gen and [Wiremock](http://wiremock.org/) to serve mock requests back to gRPC clients.


### How to use `gireMock`

If you are using Maven, follow the instruction below.


* Add a dependency to gireMock

```
<dependency>
    <groupId>io.bal.tools.grpc</groupId>
    <artifactId>gire-mock</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>compile</scope>
</dependency>
```

* if you are using protobuf-maven-plugin, add `gireMock` to the `protoPlugin` section of the `protobuf-maven-plugin` `<execution><configuration>` section.

Sample below.

```
<plugin>
    <groupId>org.xolstice.maven.plugins</groupId>
    <artifactId>protobuf-maven-plugin</artifactId>
    <version>0.5.0</version>
    <configuration>
        <protocArtifact>com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}</protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.6.1:exe:${os.detected.classifier}</pluginArtifact>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>compile</goal>
                <goal>compile-custom</goal>
            </goals>
            <configuration>
                <protocPlugins>
                    <protocPlugin>
                        <id>java8</id>
                        <groupId>io.bal.tools.grpc</groupId>
                        <artifactId>gire-mock</artifactId>
                        <version>1.0-SNAPSHOT</version>
                        <mainClass>io.giremock.generator.mock.MockGenerator</mainClass>
                    </protocPlugin>
                </protocPlugins>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Generated codes
All classes generated will have these classes:

* `<GRPC service name>Mockery.class`
* `<GRPC service name>Mock.class`

For example the following gRPC service definition

```
service PersonManagement {

    rpc GetPersonById (PersonById) returns (Person) {
    }

    rpc randomNames (google.protobuf.Empty) returns (Person) {
    }

    rpc whatsTheNameInTheFile (google.protobuf.Empty) returns (FileContent) {

    }
}
```

The generated classes will be:

`PersonManagementMockery.PersonManagementMock`

## Starting the mock server

Since all the generated codes are part of your project, you can easily start your gRPC server like below.

```
Server server = ServerBuilder.forPort(port).
                addService(new PersonManagementMockery.PersonManagementMock("localhost",5501)).
                build();
server.start();

. . . 
       
server.awaitTermination();
```

###  Start your WireMock instance.

`java -jar wiremock-standalone-2.20.0.jar --port 5501`

### Specifying WireMock mocks

If you have Grpc method (`GetPersonById`)

You should define your `request.url` to be `/getPersonById`

The response body must adhere to the Message returned by the method.


```
{
    "request": {
        "method": "POST",
        "url": "/getPersonById", 
        "bodyPatterns" : [ {
          "equalToJson" : "{ \"id\": 4 }"
        } ]
    },
    "response": {
        "status": 200,
        "body": "{ 'id' : '123', 'firstName' : 'ultimate', 'lastName' : 'warrior', 'description' : 'Wrestler'}"
       
    }
}

```



Limitations:
*  Supports only unary calls.
