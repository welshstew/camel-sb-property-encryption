# Spring-Boot, Camel XML, and Jasypt Property Encryption

### Generate the project from an archetype:

```
mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate \
  -DarchetypeCatalog=https://maven.repository.redhat.com/ga/io/fabric8/archetypes/archetypes-catalog/2.2.195.redhat-000004/archetypes-catalog-2.2.195.redhat-000004-archetype-catalog.xml \
  -DarchetypeGroupId=org.jboss.fuse.fis.archetypes \
  -DarchetypeArtifactId=spring-boot-camel-xml-archetype \
  -DarchetypeVersion=2.2.195.redhat-000004 \
  -DgroupId=com.nullendpoint \
  -DartifactId=camel-sb-property-encryption \
  -Dversion=1.0.0  
```

OR

use this project (clone it) and run

`mvn clean install`

ensure you have the [maven settings setup as per the Red Hat FIS documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.3/html-single/fuse_integration_services_2.0_for_openshift/#get-started-configure-maven)

Or use this in your ~/.m2/settings.xml 
```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups>
  </pluginGroups>

  <proxies>
  </proxies>

  <servers>
  </servers>

  <mirrors>
  </mirrors>

  <profiles>
    <profile>
      <id>fusesource.repo</id>
      <repositories>
        <repository>
          <id>redhat.ea</id>
          <name>Red Hat Early Release Repository</name>
          <url>https://maven.repository.redhat.com/earlyaccess/all</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </repository>
        <repository>
          <id>central</id>
          <name>central</name>
          <url>https://repo1.maven.org/maven2</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </repository>
        <repository>
          <id>redhat.ga</id>
          <name>Red Hat GA</name>
          <url>https://maven.repository.redhat.com/ga</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>redhat.ea</id>
          <name>Red Hat EA Community Release Repository</name>
          <url>https://maven.repository.redhat.com/earlyaccess/all</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </pluginRepository>
        <pluginRepository>
          <id>central</id>
          <name>central</name>
          <url>https://repo1.maven.org/maven2</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </pluginRepository>
        <pluginRepository>
          <id>redhat.ga</id>
          <name>Red Hat GA</name>
          <url>https://maven.repository.redhat.com/ga</url>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
          </releases>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>


  <activeProfiles>
    <activeProfile>fusesource.repo</activeProfile>
  </activeProfiles>

</settings>


```

### Create an encrypted value using a password:



```
java -cp ~/.m2/repository/org/jasypt/jasypt/1.9.2/jasypt-1.9.2.jar  org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="proof of concept" password=redhat1! algorithm=PBEWithMD5AndDES

----ENVIRONMENT-----------------

Runtime: Oracle Corporation OpenJDK 64-Bit Server VM 25.141-b16 



----ARGUMENTS-------------------

algorithm: PBEWithMD5AndDES
input: proof of concept
password: redhat1!



----OUTPUT----------------------

SJBzQyfPRJnvEG4I8lyt+UZf3PpEoG730ovyhf5wOkc=

```

### Add the following dependencies to your pom:

```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>1.9</version>
</dependency>

```

### Add a property to the application.properties and modify the camel xml...

application.properties
```
my.encrypted.value=ENC(SJBzQyfPRJnvEG4I8lyt+UZf3PpEoG730ovyhf5wOkc=)
```

camel-route:
```
<route id="simple-route">
  <from id="route-timer" uri="timer:foo?period=2000"/>
  <log id="route-log" message="hello! ${properties:my.encrypted.value}"/>
</route>
```

### Run the application locally

```
mvn -Djasypt.encryptor.password=redhat1! spring-boot:run
```

### Running on Openshift/Minishift

We need to set the JAVA_OPTIONS system property via an environment variable.  Done via OC, or automatically
in the template (see src/main/fabric8/deployment.yml - and autogenerated yaml) - obviously don't do this - your password
 is then commited to source control :-/

`
oc import-image my-jboss-fuse-6/fis-java-openshift --from=registry.access.redhat.com/jboss-fuse-6/fis-java-openshift --confirm -all
mvn fabric8:deploy
oc set env dc/camel-sb-property-encryp JAVA_OPTIONS=-Djasypt.encryptor.password=redhat1!`

```
Starting the Java application using /opt/run-java/run-java.sh ...
exec java -Djasypt.encryptor.password=redhat1! -javaagent:/opt/jolokia/jolokia.jar=config=/opt/jolokia/etc/jolokia.properties -XX:ParallelGCThreads=1 -XX:ConcGCThreads=1 -Djava.util.concurrent.ForkJoinPool.common.parallelism=1 -cp . -jar /deployments/camel-sb-property-encryption-1.0.0.jar
Picked up JAVA_TOOL_OPTIONS: -Duser.home=/home/jboss -Duser.name=jboss
I> No access restrictor found, access to any MBean is allowed
Jolokia: Agent started with URL https://172.17.0.6:8778/jolokia/
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.1.RELEASE)
08:42:52.967 [main] INFO  com.nullendpoint.Application - Starting Application on camel-sb-property-encryp-2-g8gkt with PID 1 (/deployments/camel-sb-property-encryption-1.0.0.jar started by jboss in /deployments)
08:42:53.088 [main] INFO  com.nullendpoint.Application - No active profile set, falling back to default profiles: default
08:42:53.428 [main] INFO  o.s.b.c.e.AnnotationConfigEmbeddedWebApplicationContext - Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@51521cc1: startup date [Fri Aug 25 08:42:53 UTC 2017]; root of context hierarchy
08:42:58.640 [main] INFO  o.s.b.f.xml.XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [spring/camel-context.xml]
08:43:04.837 [main] WARN  o.s.c.a.ConfigurationClassPostProcessor - Cannot enhance @Configuration bean definition 'beanNamePlaceholderRegistryPostProcessor' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as 'static'.
08:43:06.092 [main] INFO  c.u.j.EnableEncryptablePropertySourcesPostProcessor - Post-processing PropertySource instances
...
08:43:30.389 [http-nio-0.0.0.0-8081-exec-1] INFO  o.s.web.servlet.DispatcherServlet - FrameworkServlet 'dispatcherServlet': initialization started
08:43:30.478 [http-nio-0.0.0.0-8081-exec-1] INFO  o.s.web.servlet.DispatcherServlet - FrameworkServlet 'dispatcherServlet': initialization completed in 89 ms
08:43:32.142 [Camel (camel) thread #0 - timer://foo] INFO  simple-route - hello! proof of concept
08:43:34.158 [Camel (camel) thread #0 - timer://foo] INFO  simple-route - hello! proof of concept

```

#### Article references

- [https://www.ricston.com/blog/encrypting-properties-in-spring-boot-with-jasypt-spring-boot/](https://www.ricston.com/blog/encrypting-properties-in-spring-boot-with-jasypt-spring-boot/)
- [https://stackoverflow.com/questions/37404703/spring-boot-how-to-hide-passwords-in-properties-file](https://stackoverflow.com/questions/37404703/spring-boot-how-to-hide-passwords-in-properties-file)


### Building

The example can be built with

    mvn clean install

### Running the example in OpenShift

It is assumed that:
- OpenShift platform is already running, if not you can find details how to [Install OpenShift at your site](https://docs.openshift.com/container-platform/3.3/install_config/index.html).
- Your system is configured for Fabric8 Maven Workflow, if not you can find a [Get Started Guide](https://access.redhat.com/documentation/en/red-hat-jboss-middleware-for-openshift/3/single/red-hat-jboss-fuse-integration-services-20-for-openshift/)

The example can be built and run on OpenShift using a single goal:

    mvn fabric8:deploy

To list all the running pods:

    oc get pods

Then find the name of the pod that runs this quickstart, and output the logs from the running pods with:

    oc logs <name of pod>

You can also use the OpenShift [web console](https://docs.openshift.com/container-platform/3.3/getting_started/developers_console.html#developers-console-video) to manage the running pods, and view logs and much more.

### Running via an S2I Application Template

Application templates allow you deploy applications to OpenShift by filling out a form in the OpenShift console that allows you to adjust deployment parameters.  This template uses an S2I source build so that it handle building and deploying the application for you.

First, import the Fuse image streams:

    oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/GA/fis-image-streams.json

Then create the quickstart template:

    oc create -f https://raw.githubusercontent.com/jboss-fuse/application-templates/GA/quickstarts/spring-boot-camel-xml-template.json

Now when you use "Add to Project" button in the OpenShift console, you should see a template for this quickstart. 

