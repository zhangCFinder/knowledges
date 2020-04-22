[TOC]
# 一，坐标与依赖
## 1. classifier标签作用
它表示在相同版本下针对不同的环境或者jdk使用的jar
```xml
<dependency>  
      <groupId>net.sf.json-lib</groupId>   
      <artifactId>json-lib</artifactId>   
      <version>2.2.2</version>  
      <classifier>jdk15</classifier>    
</dependency> 
```
输出为 json-lib-2.2.2-jdk15.jar
## 2. 依赖范围scope
共6种，compile (编译)、test (测试)、runtime (运行时)、provided、system、import。不指定，则依赖范围默认为compile.
* compile:编译依赖范围，在编译，测试，运行时都需要。

* test: 测试依赖范围，测试时需要。编译和运行不需要。如Junit

* runtime: 运行时依赖范围，测试和运行时需要。编译不需要。如JDBC驱动包

* provided:已提供依赖范围，编译和测试时需要。运行时不需要。如servlet-api

* system:系统依赖范围。本地依赖，不在maven中央仓库。

**import**：只有在dependencyManagement下才有效果，使用该范围的依赖通常指向一个POM，作用是将目标中的dependencyManagement配置导入到当前POM的dependencyManagement中配置，除了复制配置或者继承这两种方式之外，还可以使用import范围依赖将这一配置导入。
使用import范围依赖导入依赖管理配置
```xml
  <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.juvenxu.mvnbook.account<groupId>
                <artifactId>account-parent</artifactId>
                <version>1.0-SNAPSHOT</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
注意，上面代码中依赖的type为pom,import依赖范围由于其特殊性，一般都指向打包类型为pom的模块。如果有多个项目，他们使用的依赖版本都是一致的，则可以定义一个dependencyManagement专门管理依赖的POM，然后在各个项目中导入这些依赖管理配置。
## 3. 依赖冲突的调节
```
A->B->C->X(1.0)
A->D->X(2.0)
```
由于只能引入一个版本的包，此时Maven按照最短路径选择导入x(2.0)
```    
A->B->X(1.0)
A->D->X(2.0)
```
路径长度一致，则优先选择第一个，此时导入x(1.0)

## 4. 排除依赖
`A->B->C(1.0)`

此时在A项目中，不想使用C(1.0)，而使用C(2.0)

则需要使用exclusion排除B对C(1.0)的依赖。并在A中引入C(2.0).

```xml
<!--排除B对C的依赖-->
    <dependency>
        <groupId>B</groupId>
        <artifactId>B</artifactId>
        <version>0.1</version>
        <exclusions>
            <exclusion>
                <groupId>C</groupId>
                <artifactId>C</artifactId><!--无需指定要排除项目的版本号-->
            </exclusion>
        </exclusions>
    </dependency>
    <!---在A项目中引入C(2.0)-->
    <dependency>
        <groupId>C</groupId>
        <artifactId>C</artifactId>
        <version>2.0</version>
    </dependency>
```
## 5. 可选依赖

peject A的pom
```xml
<project>
  ...
  <dependencies>
    <!-- declare the dependency to be set as optional -->
    <dependency>
      <groupId>sample.ProjectB</groupId>
      <artifactId>Project-A</artifactId>
      <version>1.0</version>
      <scope>compile</scope>
      <optional>true</optional> <!-- value will be true or false only -->
    </dependency>
  </dependencies>
</project>
```
* `Project-A -> Project-B`
如上,所以projectA依赖projectB，当A在它的pom中声明B作为它的可选依赖时，这个关系会一直保持。 这就像在通常的构建中，projectB会被加到它的类路径中。

* `Project-X -> Project-A`
 但是当另外一个工程projectX在它的pom中声明A作为其依赖时，可选性依赖这时就发挥作用了。你会发现projectB并不存在projectX的类路径中。如果X的类路径要包含B那么需要在你的pom中直接进行声明。
 
 ## 6. 依赖关系的查看
cmd进入工程根目录，执行
`mvn dependency:tree` 会列出依赖关系树及各依赖关系
`mvn dependency:analyze` 会分析依赖关系

# 二，仓库
## 1. 快照版本
maven2会根据模块的版本号(pom文件中的version)中是否带有-SNAPSHOT来判断是快照版本还是正式版本。
 * 如果是快照版本，那么在mvn deploy时会自动发布到快照版本库中，会覆盖老的快照版本，而在使用快照版本的模块，在不更改版本号的情况下，直接编译打包时，maven会自动从镜像服务器上下载最新的快照版本。
 * 如果是正式发布版本，那么在mvn deploy时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，编译打包时如果本地已经存在该版本的模块则不会主动去镜像服务器上下载。

## 2. 镜像
### 1. 在setting.xml文件中配置镜像仓库
```xml
<mirror>      
   <id>nexus-aliyun</id>    
   <name>nexus-aliyun</name>  
   <url>http://maven.aliyun.com/nexus/content/groups/public</url>    
   <mirrorOf>central</mirrorOf>      
</mirror>
```
* `<mirrorOf>`的值为central，表示该配置为中央仓库的镜像，任何对于中央仓库的请求都会转至该镜像
* `<id>`表示镜像的唯一标识符，
* `<name>`表示镜像的名称，
* `<url>`表示镜像的地址。
#### 示例：maven阿里云中央仓库
```xml
<mirrors>
    <mirror>
       <id>alimaven</id>
       <mirrorOf>central</mirrorOf>
       <name>aliyun maven</name>
       <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
  </mirrors>
```

### 2. 在setting.xml文件中配置私服
```xml
<mirror>      
   <id>internal-repository</id>    
   <name>Internar Repository Manager</name>  
   <url>http://192.168.1.100/maven2</url>    
   <mirrorOf>*</mirrorOf>      
</mirror>
```
该例中<mirrorOf>的值为星号，表示该配置是所有Maven仓库的镜像，任何对于远程仓库的请求都会被转移至192.168.1.100

### 高级镜像配置：
* `<mirrorOf>*<mirrorOf>`:匹配所有远程仓库
* `<mirrorOf>external：*<mirrorOf> `:匹配所有远程仓库，使用localhost的除外，使用file：//协议的除外，也就是说，匹配所有不在本机上的远程仓库。
* `<mirrorOf>repo1，repo2<mirrorOf> `: 配置repo1和repo2，使用逗号分隔多个远程仓库
* `<mirrorOf>*，！repo1<mirrorOf> `:匹配所有远程仓库，repo1除外，使用感叹号将仓库从匹配中排除。

### 同时设定镜像仓库，和私服仓库地址：
1. 在全局配置文件setting.xml配置一个central仓库的镜像。
   
2. 在具体项目的pom里配置个别的仓库，如下
```xml
 <repositories>  
     <repository>  
       <id>奇葩仓库</id>  
       <url>https://奇葩仓库/public/</url>  
     </repository>  
  </repositories>  
```
### 3. 【maven】配置多个仓库
#### 说明
maven的中央仓库很强大，绝大多数的jar都收录了。但也有未被收录的。遇到未收录的jar时，就会编译报错。

除了maven官方提供的仓库之外，也有很多的仓库。尽可能的将可信的仓库（嗯，可信的仓库！）添加几个，弥补maven官方仓库的不足。
#### 多仓库配置方式一：全局多仓库设置
全局多仓库设置，是通过修改maven的setting文件实现的。

**思路：**
在setting文件中添加多个profile（也可以在一个profile中包含很多个仓库），并激活（即使是只有一个可用的profile，也需要激活）。

修改maven的setting文件，设置两个仓库（以此类推，可以添加多个）：
```xml
  <profiles>
    <profile>
        <!-- id必须唯一 -->
        <id>myRepository1</id>
        <repositories>
            <repository>
                <!-- id必须唯一 -->
                <id>myRepository1_1</id>
                <!-- 仓库的url地址 -->
                <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
    </profile>
    <profile>
        <!-- id必须唯一 -->
        <id>myRepository2</id>
        <repositories>
            <repository>
                <!-- id必须唯一 -->
                <id>myRepository2_1</id>
                <!-- 仓库的url地址 -->
                <url>http://repository.jboss.org/nexus/content/groups/public-jboss/</url>
                <releases>
                    <enabled>true</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </repository>
        </repositories>
    </profile>
  </profiles>

  <activeProfiles>
    <!-- 激活myRepository1 -->
    <activeProfile>myRepository1</activeProfile>
    <!-- 激活myRepository2 -->
    <activeProfile>myRepository2</activeProfile>
  </activeProfiles>
```
#### 多仓库配置方式二：在项目中添加多个仓库
在项目中添加多个仓库，是通过修改项目中的pom文件实现的。

**思路：**
    
在项目中pom文件的repositories节点（如果没有手动添加）下添加多个repository节点，每个repository节点是一个仓库。
    
 修改项目中pom文件，设置两个仓库（以此类推，可以添加多个）：
 ```xml
    <repositories>
        <repository>
            <!-- id必须唯一 -->
            <id>jboss-repository</id>
            <!-- 见名知意即可 -->
            <name>jboss repository</name>
            <!-- 仓库的url地址 -->
            <url>http://repository.jboss.org/nexus/content/groups/public-jboss/</url>
        </repository>
        <repository>
            <!-- id必须唯一 -->
            <id>aliyun-repository</id>
            <!-- 见名知意即可 -->
            <name>aliyun repository</name>
            <!-- 仓库的url地址 -->
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </repository>
    </repositories>
```
**注：以上两种方式的id值均不可以为“central”。**

### 4. 远程仓库的认证
大部分公共的远程仓库无须认证就可以直接访问，但我们在平时的开发中往往会架设自己的Maven远程仓库，出于安全方面的考虑，我们需要提供认证信息才能访问这样的远程仓库。

配置认证信息和配置远程仓库不同，远程仓库可以直接在pom.xml中配置，但是认证信息必须配置在settings.xml文件中。

这是因为pom往往是被提交到代码仓库中供所有成员访问的，而settings.xml一般只存在于本机。因此，在settings.xml中配置认证信息更为安全。

```xml
<settings>
     ...
      <!--配置远程仓库认证信息-->
      <servers>
          <server>
              <id>releases</id>
              <username>admin</username>
              <password>admin123</password>
          </server>
     </servers>
     ...
</settings>
```
上面代码我们配置了一个id为releases的远程仓库认证信息。

Maven使用settings.xml文件中的servers元素及其子元素server配置仓库认证信息。

认证用户名为admin，认证密码为admin123。

这里的关键是id元素，settings.xml中server元素的id必须与pom.xml中需要认证的repository元素的id完全一致。

正是这个id将认证信息与仓库配置联系在了一起。


### 5. 部署构件至远程仓库
我们使用自己的远程仓库的目的就是在远程仓库中部署我们自己项目的构件以及一些无法从外部仓库直接获取的构件。这样才能在开发时，供其他对团队成员使用。

Maven除了能对项目进行编译、测试、打包之外，还能将项目生成的构件部署到远程仓库中。

首先，需要编辑项目的pom.xml文件。配置distributionManagement元素，代码如下：
```xml
<distributionManagement>
        <repository>
            <id>releases</id>
            <name>public</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>Snapshots</name>
            <url>http://59.50.95.66:8081/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
</distributionManagement>
```
distributionManagement包含repository和snapshotRepository子元素，

* repository表示发布版本（稳定版本）构件的仓库，

* snapshotRepository表示快照版本（开发测试版本）的仓库。
 
这两个元素都需要配置id、name和url，id为远程仓库的唯一标识，name是为了方便人阅读，关键的url表示该仓库的地址。

往远程仓库部署构件的时候，往往需要认证，配置认证的方式同上。

配置正确后，运行命令mvn clean deploy，Maven就会将项目构建输出的构件部署到配置对应的远程仓库，

如果项目当前的版本是快照版本，则部署到快照版本的仓库地址，否则就部署到发布版本的仓库地址。

# 三，生命周期和插件
Maven有三套相互独立的生命周期，请注意这里说的是“三套”，而且“相互独立”，初学者容易将Maven的生命周期看成一个整体，其实不然。这三套生命周期分别是：

## 1. Clean Lifecycle 在进行真正的构建之前进行一些清理工作。Clean生命周期一共包含了三个阶段：
* pre-clean  执行一些需要在clean之前完成的工作

* clean  移除所有上一次构建生成的文件

* post-clean  执行一些需要在clean之后立刻完成的工作

## 2. Default Lifecycle 构建的核心部分，编译，测试，打包，部署等等
* validate

* generate-sources

* process-sources

* generate-resources

* process-resources     复制并处理资源文件，至目标目录，准备打包。

* compile     编译项目的源代码。

* process-classes

* generate-test-sources 

* process-test-sources 

* generate-test-resources

* process-test-resources     复制并处理资源文件，至目标测试目录。

* test-compile     编译测试源代码。

* process-test-classes

* test     使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。

* prepare-package

* package     接受编译好的代码，打包成可发布的格式，如 JAR 。

* pre-integration-test

* integration-test

* post-integration-test

* verify

* install     将包安装至本地仓库，以让其它项目依赖。

* deploy     将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

## 3. Site Lifecycle 生成项目报告，站点，发布站点
* pre-site     执行一些需要在生成站点文档之前完成的工作

* site    生成项目的站点文档

* post-site     执行一些需要在生成站点文档之后完成的工作，并且为部署做准备

* site-deploy     将生成的站点文档部署到特定的服务器上

在一个生命周期内，运行任何一个阶段的时候，它前面的所有阶段都会被运行，这也就是为什么我们运行mvn install 的时候，代码会被编译，测试，打包。

# 四，聚合与继承
## 1. 聚合
Maven聚合（或者称为多模块），是为了能够使用一条命令就**构建多个模块**.

例如已经有两个模块，分别为account-email,account-persist，

我们需要创建一个额外的模块（假设名字为account-aggregator，然后通过该模块，来构建整个项目的所有模块，accout-aggregator本身作为一个Maven项目，它必须有自己的POM,不过作为一个聚合项目，其POM又有特殊的地方，看下面的配置：

```xml
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifact>account-aggregator</artifact>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Aggregator</name>
    <modules>
        <module>account-email</module>
        <module>account-persist</module>
    </modules>
</project>
```

上面有一个特殊的地方就是packaging,其值为pom,如果没有声明的话，默认为jar，**对于聚合模块来说，其打包方式必须为pom**，否则无法构建。

modules里的每一个module都可以用来指定一个被聚合模块，这里每个module的值都是一个当前pom的相对位置，

本例中account-email、account-persist位于account-aggregator目录下，当三个项目同级的时候，上面的两个module应该分别为../account-email和../account-persist

## 2. 继承
在构建多个模块的时候，往往会多有模块有相同的groupId、version，

或者有相同的依赖，例如：spring-core、spring-beans、spring-context和junit等，

或者有相同的组件配置，例如：maven-compiler-plugin和maven-resources-plugin配置，

在Maven中也有类似Java的继承机制，那就是POM的继承。

### 继承POM的用法
面向对象设计中，程序员可以通过一种类的父子结构，在父类中声明一些字段和方法供子类继承，这样可以做到“一处声明、多处使用”，类似的我们需要创建POM的父子结构，然后在父POM中声明一些配置，供子POM继承。

下面声明一个父POM，如下：
```xml
<project
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:shemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
 <modelVersion>4.0.0</modelVersion>
 <groupId>com.juvenxu.mvnbook.account</groupId>
 <artifactId>account-parent</artifactId>
 <version>1.0.0-SNAPSHOT</version>
 <packaging>pom</packaging>
 <name>Account Parent</name>
</project>
```

这个父POM中，groupId和version和其它模块一样，它的packaging为pom，这一点和聚合模块一样，作为父模块的POM，其打包类型也必须为pom,

由于父模块只是为了帮助消除配置的重复，因此它本身不包含除POM之外的项目文件，也就不需要src/main/java之类的文件夹了。

有了父模块，就需要其它模块来继承它。首先将account-email的POM修改如下：
account-email继承account-parent的POM
```xml
<project
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:shemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
 <parent>
 <groupId>com.juvenxu.mvnbook.account<groupId>
 <artifactId>account-parent</artifactId>
 <version>1.0.0-SNAPSHOT</version>
 <relativePath>../account-parent/pom.xml</relativePath>
 </parent>


<artifactId>account-email</artifactId>
 <name>Account Email</name>

 <dependencies>

....
 </dependencies>
 <build>
 <plugins>

....
 </plugins>
 </build>
</project>
```
上面POM中使用parent元素声明父模块，**parent下的子元素groupId、artifactId和version指定了父模块的坐标，这三个元素是必须的**。

元素relativePath表示了父模块POM的相对位置。当项目构建时，Maven会首先根据relativePath检查父POM，如果找不到，再从本地仓库查找。
relativePath的默认值是../pom.xml,Maven默认父POM在上一层目录下。

上面POM没有为account-email声明groupId，version,不过并不代表account-email没有groupId和version，实际上，这个子模块隐式的从父模块继承了这两个元素，这也就消除了不必要的配置。
上例中，父子模块使用了相同的groupId和version，如果遇到子模块需要使用和父模块不一样的groupId或者version的情况，可以在子模块中显式声明。

对于**artifactId元素来说，子模块更应该显式声明**，因为如果完全继承groupId、artifactId、version,会造成坐标冲突；
另一方面，即使使用不同的groupId或version,同样的artifactId容易造成混淆。

account-persist继承account-parent的POM
```xml
<project
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:shemaLocation="http://maven.apache.org/POM/4.0.0

http://maven.apache.org/maven-v4_0_0.xsd">
 <parent>
 <groupId>com.juvenxu.mvnbook.account<groupId>
 <artifactId>account-parent</artifactId>
 <version>1.0.0-SNAPSHOT</version>
 <relativePath>../account-parent/pom.xml</relativePath>
 </parent>


<artifactId>account-persist</artifactId>
 <name>Account Persist</name>

 <dependencies>

....
 </dependencies>
 <build>
 <testResources>
 <testResource>
 <directory>src/test/resources</directory>
 <filtering>true</filtering>
 </testResource>
 </testResources>
 <plugins>

....
 </plugins>
 </build>
</project>
```
最后，同样需要把account-parent加入到聚合模块accountp-aggregator中，代码如下：
将account-parent加入到聚合模块
```xml
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifact>account-aggregator</artifact>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Aggregator</name>
    <modules>
        <module>account-email</module>
        <module>account-persist</module>
        <module>account-parent</module>
    </modules>
</project>
```
### 可继承的POM元素
* groupId:项目组ID,项目坐标的核心元素
* version:项目版本,项目坐标的核心元素
* description:项目的描述信息
* organnization:项目的组织信息
* inceptionYear:项目的创始年份
* url:项目的URL地址
* developers:项目的开发者信息
* contributors:项目的贡献者信息
* distributionManagement:项目的部署配置
* issueManagement:项目的缺陷跟踪系统信息
* ciManagement:项目的集成信息
* scm:项目的版本控制系统信息
* mailingLists:项目的邮件列表信息
* properties:自定义的Maven属性
* dependencies:项目的依赖配置
* dependencyManagement:项目的依赖管理配置
* repositories:项目的仓库配置
* build:包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等

* reporting:包括项目的报告输出目录配置，报告插件配置等。

## 3. 依赖管理
当多有模块中有相同的依赖时，我们可以将这些依赖提取出来，同时在父POM中声明，这样就可以简化子模块的配置了，
但是这样还是存在问题，当想项目中加入一些，不需要这么多依赖的模块，如果让这个模块也依赖那些不需要的依赖，显然不合理。

Maven提供的**dependencyManagement元素**既能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活度。在dependencyManagement元素下的依赖声明不会引入实际的依赖，不过他能够约束denpendencies下的依赖使用。对上面的accoun-parent进行改进，代码如下：
```xml
<project
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:shemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.juvenxu.mvnbook.account</groupId>
    <artifactId>account-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>Account Parent</name>

    <properties>
        <springframework.version>2.5.6</springframework.version>
        <junit.version>4.7</junit.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>${springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${springframework.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
上面使用了dependencyManagement声明的依赖既不会给account-parent引入依赖，也不会给它的子模块引入依赖，不过这段配置是会被继承的。
修改account-email的POM如下：

继承dependencyManagement的account-email POM

```xml
<properties>
        <javax.mail.version>1.4.1</javax.mail.version>
        <greenmail.version>1.3.1b</greenmail.version>
    </properties>
    <dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
            <version>${javax.mail.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.icegreen</groupId>
            <artifactId>greenmail</artifactId>
            <version>${greenmail.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
子模块只需要配置简单的groupId和artifactId就能获得对应的依赖信息，从而引入正确的依赖。
这种依赖管理机制似乎不能减少太多的POM配置，但是其好处很大，

原因在于在POM中使用dependencyManagement声明依赖能够统一规范项目中依赖的版本，当依赖的版本在父POM中声明之后，子模块在使用依赖的时候就无需声明版本，也就不会发生多个子模块使用依赖版本不一致的情况。这可以降低依赖冲突的几率。

如果在子模块不声明依赖的使用，即使该依赖已经在父POM的dependencyManagement中声明了，也不会产生任何实际的效果。如account-persist的POM。

```xml
   <properties>
        <dom4j.version>1.6.1</dom4j.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>dom4j.version</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
```
这里没有声明spring-context-support,那么该依赖就不会被引入。这正是dependencyManagement的灵活性所在.

## 4. 插件管理
Maven也提供了pluginManagement元素帮忙管理插件。该元素中配置的依赖不会造成实际的插件调用行为，当POM中配置了真正的plugin元素，并且其groupId和artifactId与pluginManagement中配置的插件匹配时，pluginManagement的配置才会影响实际的插件行为。

在父POM中使用pluginManagement配置插件，代码如下：

```xml
 <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-source-plugin</artifactId>
                    <version>2.1.1</version>
                    <executions>
                        <execution>
                            <id>attach-sources</id>
                            <phase>verify</phase>
                            <goals>
                                <goal>jar-no-fork</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```
当子模块需要生成源码包的时候，只需要如下简单的配置，代码如下：
```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
            </plugin>
        </plugins>
</build>
```
子模块声明使用了maven-source-plugin插件，同时又继承了父模块的pluginManagement配置，两者基于groupId和artifactId匹配合并。

有了pluginManagement元素，accout-email和accout-persist的POM也能得以简化了他们都配置了maven-compiler-plugin和maven-resources-plugin。

可以将这两个插件的配置移到account-parent的pluginManagement元素中，代码如下：

```xml
 <build>
        <pluginMangement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <source>1.5</source>
                        <target>1.5</target>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifact>maven-resources-plugin</artifact>
                    <configuration>
                        <encoding>UTF-8</encoding>
                    </configuration>
                </plugin>
            </plugins>
        </pluginMangement>
    </build>
```
# 5. 配置文件管理，隔离生产环境与开发环境
## 1. 使用Filter管理
1. 建立不同的配置文件。
2. 配置pom.xml，编译部分如下
```xml

    <build>
        <filters>
            <!-- 指定过滤变量的位置 -->
            <filter>src/main/filters/dev.properties</filter>
        </filters>
        <plugins>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <configuration>
                    <!-- 使用默认的变量标记方法即${*} -->
                    <useDefaultDelimiters>true</useDefaultDelimiters>
                </configuration>
            </plugin>
        </plugins>

        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <!-- 启用过滤 -->
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
```
4. 此时，修改<filter>变量即可更改配置文件。

## 2. 使用profile管理
1. 修改pom.xml，并且添加profile，我们配置了两个profile，一个dev默认是激活状态，一个pro
```xml

    <build>
        <filters>
            <!-- 指定过滤变量的位置 -->
            <filter>src/main/filters/${env}.properties</filter>
        </filters>
        <!-- ...与刚才的内容相同 -->
    </build>
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <env>dev</env>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>

        <profile>
            <id>pro</id>
            <properties>
                <env>pro</env>
            </properties>
        </profile>
    </profiles>
```
2. 运行maven命令，使用-P参数指定使用profile
```shell
## 指定profile为pro时候，可以看到application.properties文件中的内容为maven.app.name=aihe-pro
mvn clean compile -P pro

## profile为dev的时候，文件中的内容为maven.app.name=aihe-dev
mvn clean compile -P dev
```