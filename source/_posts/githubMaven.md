---
title: 在GitHub上创建Maven存储库
date: 2018-09-04 18:32:00
tags: java
---

我们可以在github创建Maven存储库，这样可以在github托管java的依赖构件。如果有工程需要依赖，也可以通过maven方便的下载依赖。也许我们将Maven存储库托管在GitHub上有许多理由，包括许可问题、隐私和商业托管成本。无论您为什么选择托管基于GitHub的Maven存储库，设置一个存储库都很容易。具体设置步骤如下:


#### 1.创建仓库
首先在你的github上创建一个maven-repo,创建一个README.md，如清单1.1所示（不然后需mvn clean deploy 会报409错误）

清单1.1

 ![JW2wJL.png](https://i.loli.net/2018/09/04/5b8e5ef91b9f9.png)
   
#### 2.修改maven的setting.xml文件

```xml
<server>
	<id>github</id>
	<username>github登陆名</username>
	<password>github登陆密码</password>
</server>

```

#### 3.配置本地工程功的pom.xml文件

3.1设置一个从中提交文件的基本目录。

首先创建一个临时目录，并将它放在项目的 target 目录中，如清单3.1所示。对于本示例，我将该目录命名为 repo-stage。接下来，因为您想将此插件关联到 deploy 阶段，所以必须在 maven-deploy-plugin 的 <altDeploymentRepository> 中指定存储库位置，如清单 3.1  中的代码片段展示了如何指定存储库位置。

清单3.1

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>


```


3.2 配置site-maven-plugin并将它关联到 deploy 阶段。

请向该插件传递之前创建的 GitHub 服务器的 id，或者配置属性（<github.global.server>github</github.global.server>）。还需要向它传递存储库名称和所有者、您将项目工件提交到的分支，以及针对该提交的注释。您还需要关闭 Jekyll，让它不会认为自己需要生成 GitHub 页面。存储库名称是 dependency（如果该分支不存在，GitHub 将创建它）。配置如清单3.2

清单3.2

```xml
<plugin>
    <groupId>com.github.github</groupId>
    <artifactId>site-maven-plugin</artifactId>
    <version>0.12</version>
    <configuration>
        <!-- Github settings -->
        <!--<server>github</server>-->
        <repositoryName>maven-repo</repositoryName>
        <repositoryOwner>tonyonlian</repositoryOwner>
        <!--dependency分支-->
        <branch>refs/heads/dependency</branch>
        <message>Artifacts for ${project.name}/${project.artifactId}/${project.version}</message>
        <noJekyll>true</noJekyll>
        <!-- Deployment values -->
        <outputDirectory>${project.build.directory}/repo-stage</outputDirectory>
        <includes>
            <include>**/*</include>
        </includes>
    </configuration>
    <executions>
        <execution>
            <phase>deploy</phase>
            <goals>
                <goal>site</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```


3.3 完整的build配置,如清单3.3


清单3.3

```xml
 <build>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
                <configuration>
                    <altDeploymentRepository>
                        repo.stage::default::file://${project.build.directory}/repo-stage
                    </altDeploymentRepository>
                </configuration>
            </plugin>

            <plugin>
                <groupId>com.github.github</groupId>
                <artifactId>site-maven-plugin</artifactId>
                <version>0.12</version>
                <configuration>
                    <!-- Github settings -->
                    <!--<server>github</server>-->
                    <repositoryName>maven-repo</repositoryName>
                    <repositoryOwner>tonyonlian</repositoryOwner>
                     <!--dependency分支-->
                    <branch>refs/heads/dependency</branch>
                    <message>Artifacts for ${project.name}/${project.artifactId}/${project.version}</message>
                    <noJekyll>true</noJekyll>
                    <!-- Deployment values -->
                    <outputDirectory>${project.build.directory}/repo-stage</outputDirectory>
                    <includes>
                        <include>**/*</include>
                    </includes>
                </configuration>
                <executions>
                    <execution>
                        <phase>deploy</phase>
                        <goals>
                            <goal>site</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

#### 4.执行部署命令

在项目根目录下执行部署命令，命令如4.1清单所示，执行结果如清单4.2 所示

清单4.1

```
mvn clean deploy

```

清单4.2

```bash

C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo>mvn clean deploy
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Sampler 3.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ maven-repo-sampler ---

[INFO] Deleting C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\targ
et
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ maven-repo
-sampler ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory C:\Users\dongyl16339\Desktop\MavenSam
pler-github-maven-repo\src\main\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.6.1:compile (default-compile) @ maven-repo-sa
mpler ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to C:\Users\dongyl16339\Desktop\MavenSampler-gith
ub-maven-repo\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ ma
ven-repo-sampler ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory C:\Users\dongyl16339\Desktop\MavenSam
pler-github-maven-repo\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.6.1:testCompile (default-testCompile) @ maven
-repo-sampler ---
[INFO] No sources to compile
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ maven-repo-sampler
 ---
[INFO] No tests to run.
[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ maven-repo-sampler ---
[INFO] Building jar: C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo
\target\maven-repo-sampler-3.0.jar
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ maven-repo-sampl
er ---
[INFO] Installing C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\ta
rget\maven-repo-sampler-3.0.jar to D:\dev_repo\repository\com\tunyl\maven-repo-s
ampler\3.0\maven-repo-sampler-3.0.jar
[INFO] Installing C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\po
m.xml to D:\dev_repo\repository\com\tunyl\maven-repo-sampler\3.0\maven-repo-samp
ler-3.0.pom
[INFO]
[INFO] --- maven-deploy-plugin:2.8.2:deploy (default-deploy) @ maven-repo-sample
r ---
[INFO] Using alternate deployment repository repo.stage::default::file://C:\User
s\dongyl16339\Desktop\MavenSampler-github-maven-repo\target/repo-stage
Uploading: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\ta
rget/repo-stage/com/tunyl/maven-repo-sampler/3.0/maven-repo-sampler-3.0.jar
Uploaded: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\tar
get/repo-stage/com/tunyl/maven-repo-sampler/3.0/maven-repo-sampler-3.0.jar (3 KB
 at 81.1 KB/sec)
Uploading: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\ta
rget/repo-stage/com/tunyl/maven-repo-sampler/3.0/maven-repo-sampler-3.0.pom
Uploaded: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\tar
get/repo-stage/com/tunyl/maven-repo-sampler/3.0/maven-repo-sampler-3.0.pom (3 KB
 at 222.7 KB/sec)
Downloading: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\
target/repo-stage/com/tunyl/maven-repo-sampler/maven-metadata.xml
Uploading: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\ta
rget/repo-stage/com/tunyl/maven-repo-sampler/maven-metadata.xml
Uploaded: file://C:\Users\dongyl16339\Desktop\MavenSampler-github-maven-repo\tar
get/repo-stage/com/tunyl/maven-repo-sampler/maven-metadata.xml (303 B at 21.1 KB
/sec)
[INFO]
[INFO] --- site-maven-plugin:0.12:site (default) @ maven-repo-sampler ---
[INFO] Creating 9 blobs
[INFO] Creating tree with 10 blob entries
[INFO] Creating commit with SHA-1: e2b8c9c481a1e0830bf05559feb72ea650ee7f68
[INFO] Updating reference refs/heads/dependency from 5b8b548bc0ce3fdb11247d8f5b3
a506a2986a64d to e2b8c9c481a1e0830bf05559feb72ea650ee7f68
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 42.614 s
[INFO] Finished at: 2018-09-04T15:43:02+08:00
[INFO] Final Memory: 20M/172M
[INFO] ------------------------------------------------------------------------


```


#### 5. 过程执行的问题


5.1.[ERROR] Failed to execute goal com.github.github:site-maven-plugin:0.12:site (de fault) on project maven-sampler: Error creating blob: Git Repository is empty. ( 409) -> [Help 1]


- 解决方法：在github上创建的仓库里创建README.md文件。

5.2 [ERROR] Failed to execute goal com.github.github:site-maven-plugin:0.11:site (default) on project alta-maven-plugin: Error creating commit: Invalid request.
[ERROR] 
[ERROR] nil is not a string.
[ERROR] nil is not a string. (422)
[ERROR] -> [Help 1]

- 解决的方法：在github的个人设置中，设置好自己的姓名 。这个环节很重要，若不设置姓名，会出现一些一些意想不到的错误，

5.3 [ERROR] Failed to execute goal com.github.github:site-maven-plugin:0.12:site (default) on project parent: Error retrieving user info: Not Found (404) -> [Help 1]

- 解决的方法：使用jdk1.8 或修改jdk1.7的协议版本为高版本
- 

#### 6 使用github仓库中的jar包

6.1 在需要依赖的工程的pom.xml中配置仓库，配置如清单6.1所示

清单6.1

```xml

<repositories>
        <repository>
            <id>maven-repo-github</id>
            <!-- /用户名/仓库名/分支名/-->
            <url>https://github.com/tonyonlian/maven-repo/dependency/</url>
            <snapshots>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
            </snapshots>
        </repository>
</repositories>

```

6.2 添加依赖。如清单6.2所示

清单6.2

```xml
 <dependency>
    <groupId>com.tunyl</groupId>
    <artifactId>maven-repo-sampler</artifactId>
    <version>1.0</version>
 </dependency>
```


###### [下载实例代码](https://github.com/tonyonlian/maven-repo.git/)


