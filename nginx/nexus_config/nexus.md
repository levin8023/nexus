#### 制品库地址：[Welcome - Nexus Repository Manager](http://localhost:8000/nexus/)

## npm

### 配置拉取仓库

* 方法一：执行脚本：

  Mac/Linux

  ```shell
  curl -s http://localhost:8000/config/set_npm_registry.sh | bash
  ```

  或 Windows

  ```shell
  wget -O - http://localhost:8000/config/set_npm_registry.sh | bash
  ```

* 方法二：

  `vim ~/.npmrc` 配置 `registry=http://localhost:8000/nexus/repository/npm-public/`

### 发布npm包

1. 推送release包（账号密码同域帐号推送，release仓库同一版本包无法重复推送，必须升级版本）

   执行脚本

   Mac/Linux 

   ```shell
   curl -s http://localhost:8000/config/publish_npm_release | bash
   ```

   或 Windows  

   ```shell
   wget -O - http://localhost:8000/config/publish_npm_release | bash
   ```

   

   脚本内容：

   ```shell
   npm login --registry=http://localhost:8000/nexus/repository/npm-releases/ && npm publish --registry=http://localhost:8000/nexus/repository/npm-releases/
   ```

   

2. 推送snapshot包（账号密码同域帐号推送，snapshot仓库同一版本包可重复推送，用作开发）

   执行脚本
   
   
   Mac/Linux 
   
   ```shell
   curl -s http://localhost:8000/config/publish_npm_snapshot | bash
   ```
   
   或 Windows  
   
   ```shell
   wget -O - http://localhost:8000/config/publish_npm_snapshot | bash
   ```
   
   
   
   脚本内容：
   
   ```shell
   npm login --registry=http://localhost:8000/nexus/repository/npm-snapshots/ && npm publish --registry=http://localhost:8000/nexus/repository/npm-snapshots/		
   ```

### 更新snapshot包依赖

1. npm: ```npm install xxx@xxx```
2. yarn: ```yarn upgrade xxx@xxx```



## Gradle

### build.gradle 配置nexus

1. 引入maven_repositories.gradle
   

   ```groovy
   apply from: resources.text.fromInsecureUri("http://localhost:8000/config/maven_repositories.gradle")
   ```
   低版本使用：
   ```groovy
   apply from: "http://localhost:8000/config/maven_repositories.gradle"
   ```
  

### 发布jar包

1. 引入maven_publish.gradle

   
   ```groovy
   apply from: resources.text.fromInsecureUri("http://localhost:8000/config/maven_publish.gradle")
   ```
   低版本使用：
   ```groovy
   apply from: "http://localhost:8000/config/maven_publish.gradle"
   ```
   
   
   <p style="color: red">注意：此行代码需要放置在build.gradle version&group后面，不然发布脚本无法读取到项目的version&group<p>
   
   
   
   
   


4. 发布脚本

   * 方法一（命令行参数，gradle version >= 6.0）：

     执行`./gradlew publish -PnexusUsername=xxx -PnexusPassword=xxx`  

     或者 `gradle publish -PnexusUsername=xxx -PnexusPassword=xxx`

     - PnexusUsername: 帐号

     - PnexusPassword：密码

       

   * 方法二（环境变量，推荐本机使用）：

     ```shell
     export ORG_GRADLE_PROJECT_nexusUsername=xxx
     export ORG_GRADLE_PROJECT_nexusPassword=xxx
     ./gradlew publish
     ```

     - ORG_GRADLE_PROJECT_nexusUsername: 帐号

     - ORG_GRADLE_PROJECT_nexusPassword：密码

       

   #### 注意：

   * release仓库同一版本包无法重复推送，必须升级版本

   * snapshot仓库同一版本包可重复推送，用作开发



### 更新snapshot包依赖

点击 reload 或者 `gradle build --refresh-dependencies`/ 或者 `./gradlew build --refresh-dependencies`



## 问题解决方案

1. 403/Could not resolve all files for configuration ':runtimeClasspath

   注释gradle代理

   ```properties
   #systemProp.http.proxyHost=xxx
   #systemProp.http.proxyPort=xxx
   #systemProp.https.proxyHost=xxx
   #systemProp.https.proxyPort=xxx
   ```

2. the remote archive doesn't match the expected checksum

   - yarn cache clean

   - yarn --update-checksums

3.  Task :publishMavenPublicationToMavenRepository FAILED
   Could not transfer artifact xxx from/to remote (http://localhost:8000/nexus/repository/maven-releases/): Could not write to resource 'xxx’
   * 配置仓库账号密码环境变量
   

4. 项目依赖冲突

   A使用hibernate-c3p0-3.6.10.Final.jar，但是B使用的是org.hibernate:hibernate-core:5.3.3.Final，两者不兼容

   ```groovy
   implementation('cn.com.genechem.gcweb:api:3.0.0-SNAPSHOT') {
       exclude group: 'org.hibernate', module: 'hibernate-core'
   }
   ```
   
5. Cannot cast org.glassfish.jersey.servlet.init.JerseyServletContainerInitializer to javax.servlet.ServletContainerInitializer

   升级servlet-api版本

   ```
   providedCompile "javax.servlet:javax.servlet-api:4.0.1"
   ```

### 注意事项

* SNAPSHOT无法推送至Release仓库

* 非SNAPSHOT无法推送至Snapshot仓库

* 判断是否是SNAPSHOT依据：Version以`-SNAPSHOT`结尾 
