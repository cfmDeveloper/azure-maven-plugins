# Web App on Linux with Azure Container Registry E2E sample

### What you will need
- Git
- JDK 1.8
- Maven 3.0+
- Docker
- Azure Subscription

### What you will learn
This tutorial introduces the usage of [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin) 
in the basic development lifecycle of a containerized Spring Boot application with private Docker container registry. 

You will start from a complete Spring Boot application.
Then you will create a new Azure Container Registry to host your Docker container image.
And you will build a local Docker container image and publish your image to the Azure Container Registry just created.
At last, deploy the image from Azure Container Registry to an Azure Web App on Linux using [Azure Web Apps Maven Plugin](https://github.com/Microsoft/azure-maven-plugins/tree/master/webapp-maven-plugin).


### Get the sample project
Download the project from [here](https://github.com/Microsoft/gs-spring-boot-docker/tree/private-registry) or clone it from GitHub using the following command
 
```cmd
git clone -b private-registry https://github.com/microsoft/gs-spring-boot-docker 
```

Navigate to `complete` folder, you will see a simple Spring Boot application, which has been containerized.
Read more details at [here](https://spring.io/guides/gs/spring-boot-docker/) to learn how to build this app from scratch.

### Create a new Azure Private Registry
By following instructions at [here](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal), 
create a new Azure Container Registry to host your Docker container images.

Find Maven's `settings.xml`. There are two locations where a `settings.xml` file may live:
- The Maven install: ${maven.home}/conf/settings.xml
- A user’s install: ${user.home}/.m2/settings.xml

Within the `<servers>` element, add a new server definition with the credentials of your container registry.

```xml
<server>
    <id>your-container-registry-name</id>
    <username>your-container-registry-name</username>
    <password>your-container-registry-password</password>
    <configuration>
        <email>contoso@microsoft.com</email>
    </configuration>
</server>
```

### Build and Push Docker container image

Follow below steps to build a Docker container image and push to your container registry.

1. In the project's `pom.xml`, update the value of `<azure.containerRegistry>` element to the name of your container registry.
1. Run below command to build a Docker image locally and push to your container registry.

    ```cmd
    mvn clean package docker:build -DpushImage
    ```

Go to Azure Portal to verify that there is an Docker container image named `gs-spring-boot-docker` in your container registry.

The build and publish of your Docker container image is handled by the `docker-maven-plugin` from Spotify.
Configuration of the plugin is as below.
```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.4.11</version>
    <configuration>
        <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
        <registryUrl>https://${docker.image.prefix}</registryUrl>
        <serverId>${azure.containerRegistry}</serverId>
        <dockerDirectory>src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

The key configurations for your private registry are listed as following.
- `<registryUrl>`: Specifies URL of your private container registry, to which local container image will be pushed to.
- `<serverId>`: Specified the credentials to access your private container registry. 
                Here we are referencing the server defined in last step.  


### Create a new Azure Service Principal
Follow instructions at [here](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli#create-the-service-principal) 
to create an Azure Service Principal, which will be used in the following tutorial.

In Maven's `settings.xml`, within the `<servers>` element, add a new server definition with your Azure Service Principal credentials.

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
            <registryUrl>https://${docker.image.prefix}</registryUrl>
            <serverId>${azure.containerRegistry}</serverId>
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
    - `<imageName>`: Using Docker container image `${docker.image.prefix}/${project.artifactId}` from your private container registry.
    - `<registryUrl>`: Specifies URL of your container registry, so that Web App knows where to retrieve the specified Docker container image.
    - `<serverId>`: Referencing the credentials of your container registry.
- `<appSettings>`: Specifies application settings of your Web App.
    - `<property>`: Define a new property in the form of name-value pair.
                    Here we add a new setting named `PORT`, telling Web App to route requests to the exposed port of Docker container. 
