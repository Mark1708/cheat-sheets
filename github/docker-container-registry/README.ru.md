# Container registry

### [English version](https://github.com/Mark1708/cheat-sheets/blob/main/github/docker-container-registry/README.md)
### [Репозиторий проекта](https://github.com/Mark1708/github-docker-package-example)

Реестр контейнеров хранит изображения контейнеров в вашей организации или личной учетной записи и позволяет вам связать изображение с хранилищем.

## Подготовка проекта

> Решил показать вам как это работает на примере простого Spring Boot приложения.

### Изменяем pom.xml
> Делается исключительно для удобства. Теперь jar нашего проекта будет называться также как и artifactId

Добавляем такую строчку в build
```xml
<build>
	<finalName>${project.artifactId}</finalName>
	...
</build
```

А также интересно установить зависимать из Github Maven Registry (об этом можете узнать [тут](https://github.com/Mark1708/github-maven-package-example))
```xml
<dependency>  
    <groupId>example.com</groupId>  
    <artifactId>github-maven-package-example</artifactId>  
    <version>1.0-SNAPSHOT</version>  
</dependency>
```

Для устновки приватного package добавим в корень репозитория `settings.xml`
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

> Заменяем <GITHUB_USERNAME>,  <GITHUB_REPO> и <GITHUB_ACCESS_TOKEN></br>
> Вам нужен токен доступа с правами: `publish, install, and delete private, internal, and public packages`

### Добавляем Dockerfile

```Dockerfile
FROM maven:3.6.3-jdk-11 AS builder  
  
COPY ./ ./  
# Для установки приватной зависимости
COPY ./settings.xml /root/.m2/settings.xml  
  
RUN mvn clean package -DskipTests  
FROM openjdk:11.0.7-jdk-slim  
COPY --from=builder /target/<GITHUB_REPO>.jar /app.jar  
  
EXPOSE 8080  
ENTRYPOINT ["java","-jar","/app.jar"]
```

### Авторизируемся в Github Packages Registry

> Вам нужен токен доступа с правами: `read:packages, write:packages, delete:packages`

```shell
export CR_PAT=<YOUR_ACCESS_TOKEN>

echo $CR_PAT | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
```

### Собираем образ

```shell
docker build -t ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME> -f ./Dockerfile 
```

### Присваиваем тэг

```shell
docker tag ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME> ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0
```

### Пушим в Github Packages Registry

```shell
docker push ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0
```

### Установим и запустим наш образ из Github Packages Registry

```shell
docker pull ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0

docker run -p 8080:8080 ghcr.io/<GITHUB_USERNAME>/<IMAGE_NAME>:v1.0.0

curl http://localhost:8080/username/qwerty123
```

