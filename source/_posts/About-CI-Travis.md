---
title: 'About CI, Travis'
date: 2016-08-23 00:51:14
tags:
---
这段时间因为要出一个依赖库给合作方使用，算是对CI有了更深入一点的了解，现在总结出其中比较实用的地方记录下来，也许以后还用得上：）
- KEY的管理
在build过程中总会有各种的KEY，管理起来总是比较麻烦的。一般来说，Staging版本的key可以放在工程内，开发人员debug会比较方便。但是对于production版本来说，建议在CI的过程中导入，而不是随代码存放。
对于这种需求，Travis提供了两种方式，一种是直接在对应repo中设置environment varialbe，另一种安全程度更高的做法则是利用RSA对KEY进行加密，将加密后的字串放在travis脚本中，build过程利用私钥解密，即使去查看build log也看不到KEY的明文。
后一种的做法如下：
```
gem install travis
cd project_dir
travis encrypt SPECIFIED_NAME=XXXXXXXXX
```
此时会得到一段形如“secure: "xxxxxxxxxx"”的加密字串， 将其加入到.travis.yml中
```
env:
    global:
        - secure: "xxxxxxxxxx"
```
这样在build过程中即可通过`$SPECIFIED_NAME`得到KEY
- gradle版本
travis默认提供的是gradle-2.2.0, 我鼓捣了一阵子没找到修改版本号的地方，最后采用手工的方式解决了，具体代码如下：
```
wget http_gradle_distribution_url
unzip gradle_zip
export GRADLE_HOME=gradle_dir
#replace the original gradle path
export PATH=${PATH/'usr/local/gradle/bin'/$GRADLE_HOME'/bin'}
```
说到这里有一段故事，周五快下班的时候把测试好的代码push上去等着发版，结果发现原有的CI脚本采用的是gradle xxx命令，travis上提供的是gradle-2.2.0，而工程已经在使用gradle-2.14.0的wrapper了，真是尴尬，只好拖到了下周一发版。这个故事给了我两个教训：
    - 如果可以一定尽量用gradle wrapper
    - 下班前不要提交代码