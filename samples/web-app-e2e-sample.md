# Web App (on Windows) E2E sample

### What you will need
- Git
- JDK 1.8
- Maven 3.0+
- Azure Subscription

### What you will learn
This tutorial introduces the usage of [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin) 
in the basic development lifecycle of a Spring Boot application. 

You will start from a complete Spring Boot application. 
Then build the project to generate artifacts (JAR file).
At last, deploy artifacts to an Azure Web App using [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin).


### Get the sample project
Download the project from [here](https://github.com/xscript/gs-spring-boot) or clone it from GitHub using the following command
 
```cmd
git clone -b maven https://github.com/xscript/gs-spring-boot 
```

Navigate to `complete` folder, you will see a simple Spring Boot application.
Read more details at [here](https://spring.io/guides/gs/spring-boot/) to learn how to build this app from scratch.

### Create a new Azure Service Principal
Follow instructions at [here](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli#create-the-service-principal) 
to create an Azure Service Principal, which will be used in the following tutorial.

### Add a new server element in Maven's settings.xml
Find Maven's `settings.xml`. There are two locations where a `settings.xml` file may live:
- The Maven install: ${maven.home}/conf/settings.xml
- A user’s install: ${user.home}/.m2/settings.xml

Within the `<servers>` element, add a new server definition with your Azure Service Principal credentials.

```xml
<server>
    <id>azure-auth</id>
    <configuration>
        <client>your-client-id</client>
        <tenant>your-tenant-id</tenant>
        <key>your-key</key>
        <environment>AZURE</environment>
    </configuration>
</server>
```

### Build the project

Run below command to build the project. You will find file `gs-spring-boot-0.1.0.jar` in the `target` folder of the project root.

```cmd
mvn clean package
```

### Deploy to Azure Web App
We will use [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin) to deploy a Web App in Azure. If the Web App does not exist, it will be created.
Run below command to do the deployment. 

```cmd
mvn webapp:deploy
```

Configuration of the Web Apps Maven Plugin is as below.

```xml
<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>webapp-maven-plugin</artifactId>
    <version>0.1.1</version>
    <configuration>
        <authentication>
            <serverId>azure-auth</serverId>
        </authentication>
        <resourceGroup>maven-plugin</resourceGroup>
        <appName>maven-web-app-${maven.build.timestamp}</appName>
        <region>westus</region>
        <javaVersion>1.8</javaVersion>
        <javaWebContainer>tomcat 8.5</javaWebContainer>
        <deploymentType>ftp</deploymentType>
        <resources>
            <resource>
                <directory>${project.basedir}/target</directory>
                <targetPath>/</targetPath>
                <includes>
                    <include>*.jar</include>
                </includes>
            </resource>
            <resource>
                <directory>${project.basedir}</directory>
                <targetPath>/</targetPath>
                <includes>
                    <include>web.config</include>
                </includes>
            </resource>
        </resources>
    </configuration>
</plugin>
```

Detailed explanation of the configuration is as follow.

- `<authentication>`: Specifies authentication information of Azure.
    - `<serverId>`: Referencing a `<serverId>` named `azure-auth` in Maven's `settings.xml`. Credentials will be read from `settings.xml` and used to authenticate with Azure.
- `<resourceGroup>`: Target resource group is `maven-plugin`.
- `<appName>`: Target Web App is `maven-web-app-${maven.build.timestamp}`. The `${maven.build.timestamp}` part is used to avoid conflict. You can change that to a unique string as you like.
- `<region>`: Target region is `westus`.
- `<javaVersion>`: Specifies the Java Runtime version of your Web App. Here we uses Java 8.
- `<javaWebContainer>`: 
- `<deploymentType>`: Specifies deployment type of your Web App. Only `ftp` is supported now. Support of other deployment types is on the way. 
- `<resources>`: Specifies resources to be deployed to your Web App.
    - `<resource>`: Specifies single resource object to be deployed to your Web App.
        - `<directory>`: Specifies the absolute path where the resources are stored.
        - `<targetPath>`: Specifies the target path where the resources will be deployed to. 
                        It is a relative path to the `/site/wwwroot` folder in your Web App server.
        - `<includes>`: A list of patterns to include, e.g. `*.jar`. Here we include the generated JAR file and `web.config` from project root.


