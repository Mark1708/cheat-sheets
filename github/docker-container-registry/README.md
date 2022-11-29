# Container registry

### [Russian version](https://github.com/Mark1708/cheat-sheets/blob/main/github/docker-container-registry/README.ru.md)
### [Project repo](https://github.com/Mark1708/github-docker-package-example)

The Container registry stores container images within your organization or personal account, and allows you to associate an image with a repository. 

## Project preparation

> I decided to show you how it works using the example of a simple Spring Boot application.

### Changing pom.xml
> It is done solely for convenience. Now the jar of our project will be called the same as artifactId

Adding this line to build
```xml
<build>
	<finalName>${project.artifactId}</finalName>
	...
</build
```

And it is also interesting to install a dependency from Github Maven Registry (you can find out about it [here](https://github.com/Mark1708/github-maven-package-example)
```xml
<dependency>  
    <groupId>example.com</groupId>  
    <artifactId>github-maven-package-example</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

To install a private package, add it to the root of the repository `settings.xml`
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"  
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0  
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">  
  
    <activeProfiles>  
        <activeProfile>github</activeProfile>  
    </activeProfiles>  
  
    <profiles>  
        <profile>  
            <id>github</id>  
            <repositories>  
                <repository>  
                    <id>central</id>  
                    <url>https://repo1.maven.org/maven2</url>  
                </repository>  
                <repository>  
                    <id>github</id>  
                    <url>https://maven.pkg.github.com/<GITHUB_USERNAME>/<GITHUB_REPO></url> 
                    <snapshots>  
                        <enabled>true</enabled>  
                    </snapshots>  
                </repository>  
            </repositories>  
        </profile>  
    </profiles>  
  
    <servers>  
        <server>  
            <id>github</id>  
		    <username><GITHUB_USERNAME></username>
		    <password><GITHUB_ACCESS_TOKEN></password>
        </server>  
    </servers>  
</settings>
```

> Replacing `<GITHUB_USERNAME>`,` <GITHUB_REPO>` and `<GITHUB_ACCESS_TOKEN>` with real data</br>
> You need an access token with the rights: `publish, install and delete private, internal and public packages`

### Adding Dockerfile

```Dockerfile
FROM maven:3.6.3-jdk-11 AS builder  
  
COPY ./ ./  
# For install private maven dependency  
COPY ./settings.xml /root/.m2/settings.xml  
  
RUN mvn clean package -DskipTests  
FROM openjdk:11.0.7-jdk-slim  
COPY --from=builder /target/<GITHUB_REPO>.jar /app.jar  
  
EXPOSE 8080  
ENTRYPOINT ["java","-jar","/app.jar"]
```

### Log in to Github Packages Registry

> You need an access token with the rights: `read:packages, write:packages, delete:packages`

```shell
export CR_PAT=<YOUR_ACCESS_TOKEN>

echo $CR_PAT | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
```

### Build an image

```shell
docker build -t ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME> -f ./Dockerfile 
```

### Assign the tag

```shell
docker tag ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME> ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0
```

Push in Github Packages Registry

```shell
docker push ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0
```

### Install and run our image from Github Packages Registry

```shell
docker pull ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0

docker run -p 8080:8080 ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0

curl http://localhost:8080/username/qwerty123
```

### Good luck !)
