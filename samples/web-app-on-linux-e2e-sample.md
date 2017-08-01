# Web App on Linux E2E sample

### What you will need
- Git
- JDK 1.8
- Maven 3.0+
- Docker
- Azure Subscription

### What you will learn
This tutorial introduces the usage of [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin) 
in the basic development lifecycle of a containerized Spring Boot application. 

You will start from a complete Spring Boot application. 
Then build a local Docker container image and optionally publish your image to Docker Hub.
At last, deploy the image from Docker Hub to an Azure Web App on Linux using [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin).


### Get the sample project
Download the project from [here](https://github.com/Microsoft/gs-spring-boot-docker) or clone it from GitHub using the following command
 
```cmd
git clone -b maven https://github.com/microsoft/gs-spring-boot-docker 
```

Navigate to `complete` folder, you will see a simple Spring Boot application, which has been containerized.
Read more details at [here](https://spring.io/guides/gs/spring-boot-docker/) to learn how to build this app from scratch.

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

### Build Docker container image locally

If you own a Docker account, follow below steps to build a Docker container image and push to Docker Hub.

1. In the project's `pom.xml`, update the value of `<docker.image.prefix>` element to your Docker account name.
1. Run below command to build a Docker image locally.

    ```cmd
    mvn clean package docker:build
    ```

1. Run `docker push` to publish the Docker image to Docker Hub.

> **NOTE**:
> You can also publish your local Docker container image from Maven commands using `-DpushImage` parameter.
> Read more details at [here](https://github.com/spotify/docker-maven-plugin#authentication) on how to do that. 

### Deploy to Azure Web App on Linux
We will use [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin) to deploy a Web App on Linux in Azure. If the Web App does not exist, it will be created.
Run below command to deploy Docker container image to Web App. 

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
        <appName>maven-linux-app-${maven.build.timestamp}</appName>
        <region>westus</region>
        <containerSettings>
            <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
        </containerSettings>
        <appSettings>
            <property>
                <name>PORT</name>
                <value>8080</value>
            </property>
        </appSettings>
    </configuration>
</plugin>
```

Detailed explanation of the configuration is as follow.

- `<authentication>`: Specifies authentication information of Azure.
    - `<serverId>`: Referencing a `<serverId>` named `azure-auth` in Maven's `settings.xml`. Credentials will be read from `settings.xml` and used to authenticate with Azure.
- `<resourceGroup>`: Target resource group is `maven-plugin`.
- `<appName>`: Target Web App is `maven-linux-app-${maven.build.timestamp}`. The `${maven.build.timestamp}` part is used to avoid conflict. You can change that to a unique string as you like.
- `<region>`: Target region is `westus`.
- `<containerSettings>`: Specifies detailed information of the Docker container image to be used.
    - `<imageName>`: Using public image `${docker.image.prefix}/${project.artifactId}` from Docker Hub.
                     If you did not use your own Docker account, the default image is `springio/gs-spring-boot-docker`.
- `<appSettings>`: Specifies application settings of your Web App.
    - `<property>`: Define a new property in the form of name-value pair.
                    Here we add a new setting named `PORT`, telling Web App to route requests to the exposed port of Docker container. 
