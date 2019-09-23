# Nexus与Maven、NPM私服

依赖管理是软件开发中重要的一环之一，目前广泛使用的各类开发语言或者操作系统都采用了远程仓库来进行依赖管理，比如：Maven、NPM、Yum、Docker等都为各自的服务对象提供了统一的远程仓库服务。

但是有的时候开发人员可能会由于各种原因，无法连接到这些远程仓库；或者需要将自己的包放到仓库中供其他内网用户使用。这时候就需要在合适的位置上搭建一个远程仓库的镜像服务来支撑开发工作，通常称这个镜像服务器为“私服”。Nexus便是一个基于Java、开源、功能强大的仓库管理器，使用它我们便可以方便的搭建属于自己的私服。由于Nexus相关的互联网信息非常丰富，本文仅作为简要归档，不做详细介绍。



### Nexus的安装

Nexus的安装并不复杂，可以直接参考[官方文档](https://help.sonatype.com/repomanager3)；

也可参考以下内容：

[使用Nexus搭建maven私有仓库](https://www.jianshu.com/p/9740778b154f)；
[Nexus Repository Manager 3 搭建 npm 私服](https://zhuanlan.zhihu.com/p/35907412)。

搭建并配置完成Nexus后，还需要完成开发环境中的配置。



### 配置Maven私服地址

在`setting.xml`配置文件中，`profiles`标签中添加如下配置，指定Nexus私服地址：

```xml
<profile>
    <id>developer</id>
    <activation>
        <jdk>jdk-1.8</jdk>
    </activation>
    <repositories>
        <repository>
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://xxx/nexus/content/groups/public/</url>
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
            <id>nexus</id>
            <name>local private nexus</name>
            <url>http://xxx/nexus/content/groups/public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</profile>
```

若Nexus仓库设置了访问权限，可在`servers`标签中添加权限信息：

```xml
<server>
    <id>nexus-snapshots</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

若需要配置仓库镜像地址，则在`mirrors`标签中添加如下配置：

```xml
<mirror>
    <id>nexus-local</id>
    <!-- 此处配置所有的构建均从私有仓库中下载，*代表所有，也可以写成central -->
    <mirrorOf>*</mirrorOf>
    <name>Nexus osc</name>
    <url>http://xxx/nexus/content/groups/public/</url>
</mirror>
```

最后还需要在`activeProfiles`标签中激活该配置：

```xml
<activeProfiles>
    <activeProfile>developer</activeProfile>
</activeProfiles>
```



### 配置NPM私服地址

直接使用`config`命令指定npm仓库地址即可：

```shell
npm config set registry https://registry.npm.taobao.org
```

或者编辑`~/.npmrc`文件，添加如下内容：

```
registry = https://registry.npm.taobao.org
```



###### 参考内容链接

1. [npm设置仓库](https://blog.csdn.net/xinluke/article/details/52330916)
2. [maven使用私服的settings.xml配置](https://zhuanlan.zhihu.com/p/46410377)

