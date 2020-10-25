---
layout:     post
title:      "如何将jar包发布到中央仓库"
category:   git
date:       2019-06-21 18:00:00
author:     "HQ"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Maven
---

>之前在github上开发了一个简单的项目，一直想把这个jar包发布到中央仓库。一直没有时间弄这个，今天抽出点时间，按照网上的例子，操作了一遍，顺便记录一下，一遍将来参考。我的开发环境是ubuntu18.04，maven3，整个过程还是比较顺利的。

## 注册JIRA账号 
打开https://issues.sonatype.org/secure/Dashboard.jspa ，用邮箱注册即可。

## 创建issue 
需要填写group id项目地址之类的，页面都有示例提示。

## 等待审核
提交的issue需要人工审核，审核之后，issue的状态变为：RESOLVED，到了这一步就可以上传jar包了。

## 配置maven Setting.xml文件 
maven的Setting.xml文件可以在安装路径的conf目录下，也可以只修改当前用户`.m`目录下，如果你的电脑是多账户共享的话。在Settings.xml文件中，找到`<servers>`标签，然后在标签中增加如下配置：
```
<server>
    <id>自行替换</id>
    <username>替换成自己的JIRA账号</username>
    <password>替换成自己的JIRA账号密码</password>
</server>
```
id一般写oss就行。需要和pom文件中保持一致。

## 修改pom文件
我这次发布的pom文件内容如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.sonatype.oss</groupId>
        <artifactId>oss-parent</artifactId>
        <version>7</version>
    </parent>

    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
        </license>
    </licenses>
    <scm>
        <url>https://github.com/heqiao2010</url>
        <connection>https://github.com/heqiao2010/LunarCalendar.git</connection>
    </scm>
    <developers>
        <developer>
            <name>Joel Herb</name>
            <email>he_qiao_2010@yeah.net</email>
            <organization>heqiao2010</organization>
        </developer>
    </developers>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <groupId>com.github.heqiao2010</groupId>
        <artifactId>lunar</artifactId>
    <version>1.0</version>

    <name>LunarCalendar</name>
    <description>A Java implementation of Chinese lunar calendar. </description>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.5</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.7</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <profiles>
        <profile>
            <id>release</id>
            <build>
                <plugins>
                    <!-- Source -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- Javadoc -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- GPG -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <distributionManagement>
                <snapshotRepository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
                </snapshotRepository>
                <repository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
        </profile>
    </profiles>
</project>
```

## 生成并上传密钥
运行如下命令，根据提示生成密钥，需要设置pharse。
```
gpg --gen-key
```
生成的密钥类似：
```
➜  ~ gpg --list-keys 
/home/qiaohe/.gnupg/pubring.gpg
-------------------------------
pub   rsa3072 2019-06-21 [SC] [有效至：2021-06-20]
      812FAC0448CBA6C3E8EA8EA67BA82CF4A21310B2
uid           [ 绝对 ] Joel Herb <he_qiao_2010@yeah.net>
sub   rsa3072 2019-06-21 [E] [有效至：2021-06-20]
```
然后上传密钥：
```
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 812FAC0448CBA6C3E8EA8EA67BA82CF4A21310B2
```

## 执行部署
```
mvn clean deploy -P release
```
-P命令用于指定pom文件中profile的id。在编译之后，上传到中央服务器的时候，系统会提示让你输入密钥中的phrase，如果是windows系统可以在编译命令后加上`-Dgpg.passphrase=设置的pharse`

上传完成之后在https://oss.sonatype.org （登陆账号和新建issue的账号一致）这个页面的staging repositories页签中可以看到刚才上传的内容，默认是open状态的，先点击上方的close然后再点击release即可。注意relesea版本的jar中的版本号不能带SNAPSHOT字样，因为SNAPSHOT版本是不稳定版，不应该release，否则会有歧义，容易引起误解。

## 通知管理员
在之前创建的issue下，评论通知管理员，已经上传jar包了。等管理员操作之后，在中央仓库就可以看到上传的jar包了。例如：

```
<dependency>
  <groupId>com.github.heqiao2010</groupId>
  <artifactId>lunar</artifactId>
  <version>1.0</version>
</dependency>
```
## 参考
https://blog.csdn.net/ljbmxsm/article/details/78009268
