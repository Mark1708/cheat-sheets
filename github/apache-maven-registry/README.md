# Apache Maven registry
### [Russian version](https://github.com/Mark1708/cheat-sheets/blob/main/github/apache-maven-registry/README.ru.md)
### [Project repo](https://github.com/Mark1708/github-maven-package-example)
You can configure Apache Maven to publish packages to GitHub Packages and to use packages stored on GitHub Packages as dependencies in a Java project.


## Generate a standard Maven project

```bash
mvn archetype:generate -DgroupId=example.com -DartifactId=github-maven-package-example -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false

mvn validate

mvn clean install
```

## Edit pom.xml

#### Update properties

```xml
<properties>  
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
  <maven.compiler.source>11</maven.compiler.source>  
  <maven.compiler.target>11</maven.compiler.target>  
  <java.version>11</java.version>  
  
  <github.maven-plugin>0.12</github.maven-plugin>  
  <github.repository.owner>GITHUB_USERNAME</github.repository.owner>  
  <github.repository.name>GITHUB_REPO</github.repository.name>  
</properties>
```
> Replacing <GITHUB_USERNAME> and <GITHUB_REPO> with real data

#### Add distributionManagement

```xml
<distributionManagement>  
  <repository>  
    <id>github</id>  
    <name>GitHub ${github.repository.owner} Apache Maven Packages</name>  
	<url>https://maven.pkg.github.com/${github.repository.owner}/${github.repository.name}</url>  
  </repository>  
</distributionManagement>
```


#### Update build

```xml
<build>  
  <pluginManagement>  
    <plugins>  
      <plugin>  
        <groupId>org.apache.maven.plugins</groupId>  
        <artifactId>maven-plugin-plugin</artifactId>  
        <version>3.6.0</version>  
        <configuration>  
          <skipErrorNoDescriptorsFound>true</skipErrorNoDescriptorsFound>  
        </configuration>  
      </plugin>  
      <plugin>  
        <groupId>org.apache.maven.plugins</groupId>  
        <artifactId>maven-site-plugin</artifactId>  
        <version>3.9.1</version>  
      </plugin>  
    </plugins>  
  </pluginManagement>  
  <plugins>  
    <plugin>  
      <groupId>org.apache.maven.plugins</groupId>  
      <artifactId>maven-compiler-plugin</artifactId>  
      <version>3.8.1</version>  
      <configuration>  
        <source>${java.version}</source>  
        <target>${java.version}</target>  
      </configuration>  
    </plugin>  
  
    <plugin>  
      <groupId>org.apache.maven.plugins</groupId>  
      <artifactId>maven-release-plugin</artifactId>  
      <version>3.0.0-M1</version>  
    </plugin>  
  </plugins>  
</build>
```

## Add simple class User

```java
public class User {  
  
    private String username;  
    private String password;  
  
    public User(String username, String password) {  
        this.username = username;  
        this.password = password;  
    }  
    
    public User() {  
    }  
    
    public String getUsername() {  
        return username;  
    }  
    
    public void setUsername(String username) {  
        this.username = username;  
    }  
    
    public String getPassword() {  
        return password;  
    }  
    
    public void setPassword(String password) {  
        this.password = password;  
    }  
    
    @Override  
    public boolean equals(Object o) {  
        if (this == o) return true;  
        if (!(o instanceof User)) return false;  
        User user = (User) o;  
        return getUsername().equals(user.getUsername()) && getPassword().equals(user.getPassword());  
    }  
    
    @Override  
    public int hashCode() {  
        return Objects.hash(getUsername(), getPassword());  
    }  
    
    @Override  
    public String toString() {  
        return "User{" +  
                "username='" + username + '\'' +  
                ", password='" + password + '\'' +  
                '}';  
    }
}
```

## Let's make sure we didn't make a mistake and install the packages

```bash
mvn validate

mvn clean install
```

## Preparing for deployment

Edit the file `~/.m2/settings.xml ` to prove to Github the seriousness of their intentions)

`vim ~/.m2/settings.xml`

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

> Replace <GITHUB_USERNAME>,  <GITHUB_REPO> and <GITHUB_ACCESS_TOKEN> with real data</br>
> You need an access token to `publish, install, and delete private, internal, and public packages`.

## Deploy

Everything is simple here:
```shell
mvn deploy
```

## Result
In your repository you can find such a charm:
```xml
<dependency>  
  <groupId>example.com</groupId>  
  <artifactId>github-maven-package-example</artifactId>  
  <version>1.0-SNAPSHOT</version>  
</dependency>
```
You can check how use it [here](https://github.com/Mark1708/github-docker-package-example)

If the repository is public, then the dependency is available to everyone!
> But if you did it in private, then on the device where you will use this dependency, you will need to configure the same `~/.m2/settings.xml ` file.
 
### Good luck !)
