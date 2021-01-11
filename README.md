# Logging Connectors

Camunda Connectors use the [Apache http client](https://hc.apache.org/httpcomponents-client-4.5.x/index.html) under the hood. To see the request and the response data, you can enable logging for the http client.

The most interesting log levels are `org.apache.http.headers` and `org.apache.http.wire`. 

An easy way to activate the output is to use logback as your logging output and wire the Camunda logging done by slf4j and the apache http logging done by commons-logging via logback.

For this you need the jars `lockback.jar` and `jcl-over-slf4j.jar` on your classpath.

Maven manages them to you with this dependencies:

```xml
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.3</version>
    </dependency>
    
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>jcl-over-slf4j</artifactId>
      <version>1.7.30</version>
    </dependency>
```

This `logback.xml` shows the logging output of the connector on the console.

```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="info">
    <appender-ref ref="STDOUT" />
  </root>
  
  <logger name="org.camunda.bpm.connect" level="DEBUG" />
  <logger name="org.apache.http.headers" level="DEBUG" />
  <logger name="org.apache.http.wire" level="INFO" /> 
</configuration>
```

## Some pitfalls

If you have `camunda-connect-connectors-all.jar` on your classpath, it includes a shaded implementation of the Apache http client and commons logging classes. They default to jdk14-logging and need a different setup.  (TBD).

As the `camunda-connect-connectors-all` artifact is a dependency on the `camunda-engine` artifact, you have to exclude it from the dependency:

```xml
    <dependency>
      <groupId>org.camunda.bpm</groupId>
      <artifactId>camunda-engine</artifactId>
      <scope>provided</scope>
      <exclusions>
        <exclusion>
          <groupId>org.camunda.connect</groupId>
          <artifactId>camunda-connect-connectors-all</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
```

and add the `camunda-connector-http-client` artifact explicitly:

```xml
    <dependency>
      <groupId>org.camunda.connect</groupId>
      <artifactId>camunda-connect-http-client</artifactId>
      <scope>provided</scope>
    </dependency>
```

## Configure Spring Boot Logging

After removing the `camunda-connect-connectors-all.jar` from the classpath and add a client implementation like `camunda-connect-http-client`, you can add the logging levels to `application.yml` like this:

```yaml
logging:
  level:
    '[org.apache.http]': DEBUG
    '[org.camunda.bpm.connect]': DEBUG 
```

## Configure Tomcat Logging

```
org.apache.http.level = FINE
```

Remove the `camunda-connect-connectors-all-1.5.0.jar`.

Add:

* `jcl-over-slf-1.7.30.jar`
* `httpclient-4.5.9.jar`
* `httpcore-4.4.9.jar`

* `camunda-connect-http-client-1.5.0.jar`

## Configure Camunda BPM Run

Handle Logging with `camunda-connect-connector-all.jar`.

## TODO

Setup logging with the http client under the `connectorjar` package at `camunda-connect-connectors-all`.