
# How to build a ci pipeline with maven and jenkins


# Maven

1. create project boilerplate `mvn archetype:generate -DgroupId=at.fhv.teamxx -DartifactId=mySampleProject2 -DarchetypeArtifactId=maven-archetype-quickstart`

Result:
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>at.fhv.teamxx</groupId>
  <artifactId>mySampleProject</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>mySampleProject</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

Useful property settings:
- set encoding: `<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>`
- set java version: `<maven.compiler.source>1.8</maven.compiler.source>` and `<maven.compiler.target>1.8</maven.compiler.target>`
- build a war file: `<packaging>war</packaging>`
- metadata for eclipse: `mvn eclipse:eclipse`


2. build and test (`mvn clean package -U`)

inspect folder (`target/**`)

3. further actions

- find java packages: `http://search.maven.org`
- build and update dependencies: `mvn package -U`
- clean the build: `mvn clean` 
- test: `mvn test`
- skip test: `mvn -DskipTests ...`
- set version: `mvn versions:set -DnewVersion=1.0.4711-SNAPSHOT`
- release: `mvn -B release:prepare` and `mvn -B release:perform`, also make sure that `scm connection` is fine:
  ```
  <scm>
		<connection>scm:git:git@github.com:gtonic/mySampleProject.git</connection>
		<developerConnection>scm:git:git@github.com:gtonic/mySampleProject.git</developerConnection>
		<url>git@github.com:gtonic/mySampleProject.git</url>
	        <tag>HEAD</tag>
  </scm>
  ```
- install in local repository: `mvn install`
- deploy (only when using remote maven mirror): `mvn deploy`

## hook maven up with nexus repository

 1. `.m2/settings.xml`:
 
 ```
 <settings>
  <!-- the path to the local repository - defaults to ~/.m2/repository
  -->
  <!-- <localRepository>/path/to/local/repo</localRepository>
  -->
    <mirrors>
    <mirror> <!--Send all requests to the public group -->
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://10.0.51.99:8081/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
  <activeProfiles>
    <!--make the profile active all the time -->
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Override the repository (and pluginRepository) "central" from the Maven Super POM
          to activate snapshots for both! -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>

  <pluginGroups>
    <pluginGroup>org.sonatype.plugins</pluginGroup>
  </pluginGroups>

  <servers>
    <server>
      <id>nexus-snapshots</id>
      <username>deployment</username>
      <password>fhvr0xx2018</password>
    </server>
    <server>
      <id>nexus-releases</id>
      <username>deployment</username>
      <password>fhvr0xx2018</password>
    </server>
  </servers>
</settings>
 ```
 
2. `mvn clean package -U`


# Git

1. messing around with git (aka create a git repo)


# Jenkins

## install and configure jenkins

1. download jenkins (https://jenkins.io/download/), select `Java Package .war`)
2. `java -jar jenkins.war` (or if port is blocked, specify another one with the `--httpPort=4711` flag)
3. localhost:8080
4. enter initial admin password (autom. generated, from console)
5. configure proxy (if needed) and select 'install suggested plugins'
6. create admin user

7. inspect
8. install maven (`apt-get install maven`)

## configure build pipeline

1. Prepare Jenkins pipeline as code definition:

```
pipeline {
    agent any 

    stages {
        stage('Build') { 
            steps { 
                sh 'mvn clean -DskipTest -U package' 
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test pmd:pmd'
                junit 'target/surefire-reports/*.xml' 
                pmd canRunOnFailed: true, pattern: 'target/pmd.xml'
                
            }
        }
        stage('Deploy') {
            steps {
                sh 'mvn -DskipTest deploy'
            }
        }
    }
}
```


2. In jenkins create buildplan (new item -> multibranch pipeline 'mySampleProject')
3. Goto project, `configure`
4. Enter git path/credentials (Branch resources), adjust scan trigger (periodically every x minutes) and save
5. Now you're ready - inspect the build

6. pmd: 

``` 
   <reporting>
        <plugins>
            <plugin>
            	<groupId>org.apache.maven.plugins</groupId>
            	<artifactId>maven-pmd-plugin</artifactId>
            	<version>2.7.1</version>
            	<configuration>
                	<linkXRef>false</linkXRef>
                	<targetJdk>1.8</targetJdk>
                	<rulesets>
                    		<ruleset>/rulesets/basic.xml</ruleset>
                	</rulesets>
            	</configuration>
            </plugin>
        </plugins>
    </reporting>
```

7. plugins `blueocean`, `static analysis collector` and `pmd` plugin
8. configure pmd in step
9. provoke a warning ;)

## configure release pipeline

e.g. a maven type project which performs `-B release:prepare -B release:perform`


## future extension

https://medium.com/@MichaKutz/create-a-declarative-pipeline-jenkinsfile-for-maven-releasing-1d43c896880c

