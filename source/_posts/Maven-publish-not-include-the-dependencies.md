---
title: Maven-publish not include the dependencies
date: 2016-08-22 13:54:54
tags: gradle, maven
---
最近在使用Maven-publish的时候遇到一个问题：生成的pom文件中并不会包含项目所需的依赖，因此即使在其他工程中设置了`transitive=true`, 调用自己的库依然会出现找不到类的问题。

解决办法也很简单，maven-publish提供了自定义pom的方法，见[Chapter 34. Maven Publishing (new)](https://docs.gradle.org/current/userguide/publishing_maven.html#sec:modifying_the_generated_pom)

示例如下：
```java
publications {
        ourLibrary(MavenPublication) {
            groupId 'org.houxg'
            artifactId 'libA'
            version '0.1'
            artifact("$buildDir/outputs/aar/libA-release.aar")
            pom.withXml {
                def dependenciesNode = asNode().getAt('dependencies')[0]
                if (dependenciesNode == null) {
                    dependenciesNode = asNode().appendNode('dependencies')
                }
                configurations.compile.allDependencies.each {
                    if ("unspecified".equals(it.name)) {
                        return
                    }
                    def dependencyNode = dependenciesNode.appendNode('dependency')
                    dependencyNode.appendNode('groupId', it.group)
                    dependencyNode.appendNode('artifactId', it.name)
                    dependencyNode.appendNode('version', it.version)
                    dependencyNode.appendNode('scope', "compile")
                }
            }
        }
    }
```
