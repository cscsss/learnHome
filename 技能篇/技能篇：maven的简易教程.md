 # 前言
 maven是最流行的项目构建系统，如果是java相关的开发，可以说是不可或缺的。虽然还有另外一个模块管理工具grade正在崛起， 不过多数公司常用的还是maven
 - Maven 和 Maven的仓库
 - Maven 命令和构建生命周期
 - maven项目pom.xml格式文件详解
 - Maven怎么做到传递依赖与排除依赖
 - Maven的聚合和继承
 - mvn使用实例
 
 关注公众号，一起交流，微信搜一搜: 潜行前行
 ---
 
 ## Maven 和 Maven的仓库
 Maven仓库用来存放Maven所管理Jar包。分为两种：本地仓库 和 中央仓库
 - 本地仓库：Maven本地的Jar包仓库
 - 中央仓库：Maven官方提供的远程仓库，Maven 中央仓库是由 Maven 社区提供的仓库，其中包含了大量常用的库。中央仓库包含了绝大多数流行的开源Java构件
 - 远程仓库: Maven 在中央仓库中也找不到依赖的文件，它会停止构建过程并输出错误信息到控制台。为避免这种情况，Maven 提供了远程仓库的概念，它是开发人员自己定制仓库，包含了所需要的代码库或者其他工程中用到的 jar 文件
 
 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2833261d00234d48b0b7ee22a1cf8496~tplv-k3u1fbpfcp-watermark.image)
 
 ## Maven 命令和构建生命周期
 阶段   |	处理   |	描述
 --- | --- |--- 
 validate	 | 验证项目	 |验证项目是否正确且所有必须信息是可用的
 compile	 | 执行编译	 |源代码编译在此阶段完成
 test	 | 测试	 | 使用适当的单元测试框架（例如JUnit）运行测试。
 package	 | 打包	 | 创建JAR/WAR包如在 pom.xml 中定义提及的包
 verify 	 | 检查	 | 对集成测试的结果进行检查，以保证质量达标
 install	 | 安装	 | 安装打包的项目到本地仓库，以供其他项目使用
 deploy	 | 部署	 | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程
 
 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12e387284576440b95bbea7633203e5c~tplv-k3u1fbpfcp-watermark.image)
 
 - 当一个阶段通过 Maven 命令调用时，如`mvn compile` ，该阶段之前以及包括该阶段在内的所有阶段都会被执行
 - maven 还有一个clean命令，用于移除所有上一次构建生成的文件
 - 在构建环境中，常使用`mvn clean deploy`命令来清除，及构建、部署项目到远程仓库中
 
 ## maven项目 pom.xml 格式文件详解
 ```mvn
 <project xmlns="http://maven.apache.org/POM/4.0.0"     
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0http://maven.apache.org/maven-v4_0_0.xsd">     
     <!--父项目的坐标-->    
     <parent>  
         <artifactId/> <!--被继承的父项目的构件标识符-->    
         <groupId/>  <!--被继承的父项目的全球唯一标识符-->      
         <version/>  <!--被继承的父项目的版本-->    
         <!-- 父项目的pom.xml文件的相对路径-->    
         <relativePath/>    
     </parent>    
     <!--声明项目描述符遵循哪一个POM模型版本。模型本身的版本很少改变-->       
     <modelVersion>4.0.0</modelVersion>     
     <!--项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成， 如com.ex.app生成的相对路径为：/com/ex/app-->     
     <groupId>com.ex.app</groupId>     
     <!-- 构件的标识符，它和group ID一起唯一标识一个构件-->     
     <artifactId>erhuowang-maven2</artifactId>     
     <!--项目产生的构件类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型-->     
     <packaging>war</packaging>     
     <!--项目当前版本，格式为:主版本.次版本.增量版本-限定版本号-->     
     <version>1.0-SNAPSHOT</version>     
     <!--项目的名称, Maven产生的文档用-->     
     <name>erhuo-maven</name>     
     <!--项目主页的URL, Maven产生的文档用-->     
     <url>http://erhuowang.cn</url>     
     <!-- 项目的详细描述-->     
     <description>A maven project to study maven.</description>     
     <!-- 描述了这个项目构建环境中的前提条件。 --> 
     <prerequisites> 
         <!-- 构建该项目或使用该插件所需要的Maven的最低版本 --> 
         <maven></maven> 
     </prerequisites>
 ------------------------------------------------------------------------------------     
     <!--该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。-->    
     <dependencies>     
       <dependency>    
         <!--依赖的group ID-->    
         <groupId>org.apache.maven</groupId>     
         <!--依赖的artifact ID-->    
         <artifactId>maven-artifact</artifactId>     
         <!--依赖的版本号。 在Maven 2里, 也可以配置成版本号的范围。-->    
         <version>3.8.1</version>     
         <!-- 依赖类型，默认类型是jar。它通常表示依赖的文件的扩展名，但也有例外。一个类型可以被映射成另外一个扩展名或分类器。类型经常和使用的打包方式对应， 尽管这也有例外。一些类型的例子：jar，war，ejb-client和test-jar。如果设置extensions为 true，就可以在 plugin里定义新的类型。所以前面的类型的例子不完整。-->    
         <type>jar</type>    
         <classifier></classifier>    
         <!--依赖范围。在项目发布过程中，帮助决定哪些构件被包括进来。欲知详情请参考依赖机制。    
             - compile ：默认范围，用于编译      
             - provided：类似于编译，但支持你期待jdk或者容器提供，类似于classpath      
             - runtime: 在执行时需要使用      
             - test:    用于test任务时使用      
             - system: 需要外在提供相应的元素。通过systemPath来取得      
             - systemPath: 仅用于范围为system。提供相应的路径      
             - optional:   当项目自身被依赖时，标注依赖是否传递。用于连续依赖时使用-->     
         <scope>test</scope>       
         <!--当计算传递依赖时， 从依赖构件列表里，列出被排除的依赖构件-->    
         <exclusions>    
          <exclusion>     
                 <artifactId>spring-core</artifactId>     
                 <groupId>org.springframework</groupId>     
             </exclusion>     
         </exclusions>       
         <!--可选依赖，如果你在项目B中把C依赖声明为可选，你就需要在依赖于B的项目（例如项目A）中显式的引用对C的依赖。可选依赖阻断依赖的传递性。-->     
         <optional>true</optional>    
      </dependency>    
     </dependencies>  
 ----------------------------------------------------------------------------------------------
     <build>    
       <!--该元素设置了项目源码目录，当构建项目的时候，构建系统会编译目录里的源码。该路径是相对于pom.xml的相对路径。-->    
       <sourceDirectory/>    
       <scriptSourceDirectory/> <!--绝大多数情况下，该目录下的内容 会被拷贝到输出目录(因为脚本是被解释的，而不是被编译的)。-->      
       <testSourceDirectory/>   <!--该元素设置了项目单元测试使用的源码目录。该路径是相对于pom.xml的相对路径。-->     
       <outputDirectory/>     <!--被编译过的应用程序class文件存放的目录。--> 
       <testOutputDirectory/>  <!--被编译过的测试class文件存放的目录。-->       
       <!--使用来自该项目的一系列构建扩展-->    
       <extensions>     
            <extension>   <!--描述使用到的构建扩展。-->      
             <groupId/>   <!--构建扩展的groupId-->   
             <artifactId/> <!--构建扩展的artifactId-->  
             <version/> <!--构建扩展的版本-->       
            </extension>    
       </extensions>    
       <!--当项目没有规定目标（Maven2 叫做阶段）时的默认值-->    
       <defaultGoal/>    
       <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。-->    
       <resources> <!--这个元素描述了项目相关或测试相关的所有资源路径-->    
           <resource>  <!-- 描述了资源的目标路径。该路径相对target/classes目录-->    
             <targetPath/>    
             <filtering/> 
             <directory/>   <!--描述存放资源的目录，该路径相对POM路径-->   
             <includes/> <!--包含的模式列表，例如**/*.xml.-->    
             <excludes/> <!--排除的模式列表，例如**/*.xml-->     
             </resource>    
       </resources>    
       <!--这个元素描述了单元测试相关的所有资源路径，例如和单元测试相关的属性文件。-->    
       <testResources>    
          <!--这个元素描述了测试相关的所有资源路径-->    
          <testResource></testResource>    
       </testResources>    
       <targetPath/>
       <filtering/>
       <directory/> <!--构建产生的所有文件存放的目录-->    
           <includes/><excludes/>    
       <directory/> 
       <finalName/> <!--产生的构件的文件名，默认值是${artifactId}-${version}。-->    
       <!--当filtering开关打开时，使用到的过滤器属性文件列表-->    
       <filters/>    
       
        <!--使用的插件列表-->    
       <plugins>    
         <!--参见build/pluginManagement/plugins/plugin元素-->    
         <plugin> </plugin>    
       </plugins>  
       
       <!--子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本地配置都会覆盖这里的配置-->    
       <pluginManagement>  
            <plugins>  
                <!--plugin元素包含描述插件所需要的信息。-->    
                <plugin>    
                   <groupId/>   <!--插件在仓库里的group ID--> 
                   <artifactId/> <!--插件在仓库里的artifact ID-->  
                   <version/>   
                   <extensions/>    
                   <executions> <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。-->   
                     <execution> <!--execution元素包含了插件执行需要的信息--> 
                       <id/>  <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标--> 
                       <phase/> <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->      
                       <goals/>  <!--配置的执行目标-->    
                       <inherited/>    <!--配置是否被传播到子POM-->    
                       <configuration/> <!--作为DOM对象的配置-->     
                     </execution>    
                   </executions>    
                   <!--项目引入插件所需要的额外依赖-->    
                   <dependencies>    
                    <!--参见dependencies/dependency元素-->    
                    <dependency>    
                     ......    
                    </dependency>    
                   </dependencies>           
                   <!--作为DOM对象的配置-->    
                   <configuration/>    
                </plugin>    
            </plugins>    
       </pluginManagement>    
     </build>    
  -----------------------------------------------------------------------------------
     <!--在列的项目构建profile，如果被激活，会修改构建处理-->    
     <profiles>    
       <!--根据环境参数或命令行参数激活某个构建处理-->    
       <profile>    
        <id/>      
        <activation>    
         <!--profile默认是否激活的标志-->    
         <activeByDefault/> 
         <jdk/>jdk版本，如:1.8</jdk>    
         
         <!--当匹配的操作系统属性被检测到，profile被激活。os元素可以定义一些操作系统相关的属性。-->
         <!--如果Maven检测到某一个属性（其值可以在POM中通过${名称}引用），其拥有对应的名称
         值，Profile就会被激活。如果值字段是空的，那么存在属性名称字段就会激活profile，
         否则按区分大小写方式匹配属性值字段-->    
         <property> 
           <name>mavenVersion</name> <!--激活profile的属性的名称-->       
           <!--激活profile的属性的值-->    
           <value>2.0.3</value>    
         </property>    
         <!--提供一个文件名，通过检测该文件的存在或不存在来激活profile。missing检查文件是否存在，如果不存在则激活 -->    
         <file>    
          <!--如果指定的文件存在，则激活profile。-->    
          <exists>/usr/local/workspace/</exists>    
          <!--如果指定的文件不存在，则激活profile。-->    
          <missing>/usr/local/workspace/</missing>    
         </file>    
        </activation>    
        <!--构建项目所需要的信息。参见build元素-->    
        <build>    
             <defaultGoal/>    
             <resources>    
              <resource>    
               <targetPath/><filtering/><directory/><includes/><excludes/>    
              </resource>    
             </resources>    
             <testResources>    
              <testResource>    
               <targetPath/><filtering/><directory/><includes/><excludes/>    
              </testResource>    
             </testResources>    
             <directory/><finalName/><filters/>    
             <pluginManagement>    
              <plugins>    
               <!--参见build/pluginManagement/plugins/plugin元素-->    
               <plugin>    
                <groupId/><artifactId/><version/><extensions/>    
                <executions>    
                 <execution>    
                  <id/><phase/><goals/><inherited/><configuration/>    
                 </execution>    
                </executions>    
                <dependencies>    
                 <!--参见dependencies/dependency元素-->    
                 <dependency>    
                  ......    
                 </dependency>    
                </dependencies>    
                <goals/><inherited/><configuration/>    
               </plugin>    
              </plugins>    
             </pluginManagement>    
             <plugins>    
              <!--参见build/pluginManagement/plugins/plugin元素-->    
              <plugin>    
               <groupId/><artifactId/><version/><extensions/>    
               <executions>    
                <execution>    
                 <id/><phase/><goals/><inherited/><configuration/>    
                </execution>    
               </executions>    
               <dependencies>    
                <!--参见dependencies/dependency元素-->    
                <dependency>    
                 ......    
                </dependency>    
               </dependencies>    
               <goals/><inherited/><configuration/>    
              </plugin>    
             </plugins>    
        </build>    
        <modules/>   
          <!--子项目相对路径-->
          <module></module>
        </modules>          
        <!--发现依赖和扩展的远程仓库列表。-->    
        <repositories>    
            <!--参见repositories/repository元素-->    
            <repository>    
              <releases>    
               <enabled/><updatePolicy/><checksumPolicy/>    
              </releases>    
              <snapshots>    
               <enabled/><updatePolicy/><checksumPolicy/>    
              </snapshots>    
              <id/><name/><url/><layout/>    
            </repository>    
        </repositories>    
        <!--发现插件的远程仓库列表，这些插件用于构建和报表-->    
        <pluginRepositories>    
            <!--包含需要连接到远程插件仓库的信息.参见repositories/repository元素--> 
            <pluginRepository>    
              <releases>    
               <enabled/><updatePolicy/><checksumPolicy/>    
              </releases>    
              <snapshots>    
               <enabled/><updatePolicy/><checksumPolicy/>    
              </snapshots>    
              <id/><name/><url/><layout/>    
            </pluginRepository>    
        </pluginRepositories>    
        <!--该元素描述了项目相关的所有依赖 -->    
        <dependencies>    
          <!--参见dependencies/dependency元素-->    
          <dependency/>   
        </dependencies>    
        <dependencyManagement>  
            <dependencies>    
              <!--参见dependencies/dependency元素-->    
              <dependency>    
              ......    
              </dependency>    
            </dependencies>    
        </dependencyManagement>    
        <!--参见distributionManagement元素-->    
        <distributionManagement>    
         ......    
        </distributionManagement>    
        <!--参见properties元素-->    
        <properties/>    
       </profile>    
     </profiles>    
  ------------------------------------------------------------------------------------     
     <!-- 聚合模块被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径 --> 
     <modules>
         <!--子项目相对路径-->
         <module></module>
     </modules> 
  ------------------------------------------------------------------------------------ 
     <repositories>  <!--发现依赖和扩展的远程仓库列表。-->      
      <!--包含需要连接到远程仓库的信息-->    
      <repository>    
          <releases> <!--如何处理远程仓库里发布版本的下载-->    
           <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->    
          <enabled/>    
           <!-- 这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。-->    
           <updatePolicy/>    
           <!--当Maven验证构件校验文件失败时该怎么做：ignore（忽略），fail（失败），或者warn（警告）。-->   
           <checksumPolicy/>    
          </releases>    
          <!-- 如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的 策略-->    
          <snapshots>    
           <enabled/><updatePolicy/><checksumPolicy/>    
          </snapshots>    
          <!--远程仓库唯一标识符。可以用来匹配在settings.xml文件里配置的远程仓库-->    
          <id>banseon-repository-proxy</id>     
          <!--远程仓库名称-->    
          <name>banseon-repository-proxy</name>     
          <!--远程仓库URL，按protocol://hostname/path形式-->    
          <url>http://192.168.1.169:9999/repository/</url>  
          <layout>default</layout>               
      </repository>     
     </repositories>    
 ------------------------------------------------------------------------------------
     <!--发现插件的远程仓库列表，这些插件用于构建和报表-->    
     <pluginRepositories>    
      <!--包含需要连接到远程插件仓库的信息.参见repositories/repository元素-->    
      <pluginRepository>    
       ......    
      </pluginRepository>    
     </pluginRepositories>        
 ------------------------------------------------------------------------------------
     <!-- 继承自该项目的所有子项目的默认依赖信息。这部分的依赖信息不会被立即解析,而是当子项目声明一个依赖（必须描述group ID和 artifact ID信息），如果group ID和artifact ID以外的一些信息没有描述，则通过group ID和artifact ID 匹配到这里的依赖，并使用这里的依赖信息。-->    
     <dependencyManagement>    
       <dependencies>    
        <!--参见dependencies/dependency元素-->    
        <dependency>    
         ......    
        </dependency>    
       </dependencies>    
     </dependencyManagement>       
 ---------------------------------------------------------------------------------------------  
     <!--项目分发信息，在执行mvn deploy后表示要发布的位置。有了这些信息就可以把网站部署到远程服务器或者把构件部署到远程仓库。-->     
     <distributionManagement>    
         <!--部署项目产生的构件到远程仓库需要的信息-->    
         <repository>    
          <!--是分配给快照一个唯一的版本号-->    
          <uniqueVersion/>    
          <id>banseon-maven2</id>     
          <name>banseon maven2</name>     
          <url>file://${basedir}/target/deploy</url>     
          <layout/>    
         </repository>   
         <!--部署项目的网站需要的信息-->     
         <site>    
          <!--部署位置的唯一标识符，用来匹配站点和settings.xml文件里的配置-->     
             <id>banseon-site</id>     
             <!--部署位置的名称-->    
             <name>business api website</name>     
             <!--部署位置的URL，按protocol://hostname/path形式-->    
             <url>     
                 scp://svn.baidu.com/banseon:/var/www/localhost/banseon-web      
             </url>     
         </site>    
         <!-- 给出该构件在远程仓库的状态。不得在本地项目中设置该元素，因为这是工具自动更新的。有效的值有：none（默认），converted（仓库管理员从 Maven 1 POM转换过来），partner（直接从伙伴Maven 2仓库同步过来），deployed（从Maven 2实例部 署），verified（被核实时正确的和最终的）。-->    
        <status/>           
     </distributionManagement>  
 ---------------------------------------------------------------------------------------------- 
     <!--以值替代名称，Properties可以在整个POM中使用，也可以作为触发条件（见settings.xml配置文件里activation元素的说明）。格式是<name>value</name>。-->    
     <properties/>    
 </project>    
 ```
 
 ## Maven怎么做到传递依赖与排除依赖
 - 传递依赖：如果我们的项目引用了一个Jar包，而该Jar包又引用了其他Jar包，那么在默认情况下项目编译时，Maven会把直接引用和简洁引用的Jar包都下载到本地
 - 排除依赖：如果我们只想下载直接引用的Jar包，那么需要在pom.xml中做如下配置
 ```mvn
 <exclusions>
     <exclusion>
         <groupId>ch.qos.logback</groupId>
         <artifactId>logback-classic</artifactId>
     </exclusion>
 </exclusions>
 ```
 - 依赖冲突：若项目中多个Jar同时引用了相同的Jar时，会产生依赖冲突，但Maven采用了两种避免冲突的策略，因此在Maven中是不存在依赖冲突的
     * 短路优先，例如`A.jar —> B.jar —> X.jar；C.jar —> X.jar`，优先使用短路径的X.jar
     * 声明优先；若引用路径长度相同时，在pom.xml中谁先被声明
     
 ## Maven的聚合和继承
 - 同时配置多个运行项目称之聚合，聚合的项目有一个父类pom.xml文件，而各个项目也有属于自己的pom.xml
 - 在聚合多个项目时，如果这些被聚合的项目中需要引入相同的Jar，那么可以将这些Jar写入父pom.xml中，各个子项目继承该pom即可
 - 父pom.xml
 ```mvn
 <!-- 聚合多个项目  -->
 <modules>
     <module>../模块A</module>
     <module>../模块B</module>
     <module>../模块C</module>
 </modules>
 
 <dependencyManagement>
     <dependencies>
         <dependency>
             <groupId>org.apache.shiro</groupId>
             <artifactId>shiro-spring</artifactId>
             <version>1.2.2</version>
         </dependency> 
     </dependencies>
 </dependencyManagement>
 ```
 - 子pom.xml配置
 ```mvn
 <!-- 继承了 父类的 org.apache.shiro jar  -->
 <parent>
     <groupId>父pom所在项目的groupId</groupId>
     <artifactId>父pom所在项目的artifactId</artifactId>
     <version>父pom所在项目的版本号</version>
 </parent>
 ```
 
 ## mvn使用实例
 ```mvn
 //创建Maven的普通java项目
 mvn archetype:create -DgroupId=packageName -DartifactId=projectName
 //编译源代码
 mvn compile 
 //只测试而不编译，也不测试编译
 mvn test -skipping compile -skipping test-compile 
 //运行测试
 mvn test 
 //清除产生的项目
 mvn clean
 //在本地Repository中安装jar
 mvn install
 //清除旧的项目，并生成新的jar
 mvn clean install
 //清除旧的项目，并生成新的jar且上传远程仓库
 mvn clean install deploy
 ```
 
 欢迎指正文中错误
 ---
 
 #  参考文章
 - [Maven 教程](https://www.runoob.com/maven/maven-tutorial.html)
 - [史上最全的maven的pom.xml文件详解](https://www.cnblogs.com/hafiz/p/5360195.html)
 - [Maven使用详解](https://blog.csdn.net/u010425776/article/details/52027706)
 - [maven常用命令集合](https://blog.csdn.net/xushuanglu_csdn/article/details/84298639)
