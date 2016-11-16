---
title: 折腾Leanote android(1)
date: 2016-11-16 23:42:38
tags:
---
这几周一直在鼓捣Leanote android，因为我实在受够了为知笔记的笨重和不美观。我希望可以自己修改笔记软件的UI和排版，让它看起来更cool一些，而开源的Leanote刚好契合了我的需求。
Leanote android的repo好像一直没什么人理，clone下来编译也不通过，好在有人folk并修改了一个可以编译并简单使用的版本，我也就基于这个版本来做一些事情。
### Retrofit、DBFlow和RxJava
也许是年代比较久远，原来的代码用的还是Volley作为网络框架，手写SQL语句实现数据库读写。基于一些考虑，我决定替换掉它们。
#### Retrofit
Volley并不是不好，它的扩展性很高，在API并不是标准RESTful的时候可以比较容易做到适配。但我还是希望用Retrofit替换掉Volley，因为基于Retrofit的实现更简洁，定义API的方式也更清晰。另外一个原因是，原先的代码中api被写在任何需要的地方，这给维护造成了不小的困难，我希望改进它。
#### DBFlow
数据库读写方面，手写SQL语句不是一个好的习惯，它的开发效率会比较低，维护成本也高，因此我打算用DBFlow作为ORM框架。它不像GreenDao那样需要一个额外的Java工程，同时宣称其性能比GreenDao要高一些，实际开发时，DBFlow提供的API使用起来还算舒适，效率也挺高的。
#### RxJava
RxJava使用起来很方便，FP的一大好处是你不需要考虑环境变化对变量的影响，因为无论如何，f(x)就是等于y。不过有些时候总会手脚打架，死活都要引入其他的变量才能做完我要做的事情，这也使我意识到有些时候FP并不是万能的，该面向过程的时候还是老老实实面向过程吧。
值得一提的是，RxJava切换线程非常方便，以至于有时候我在用它做AsyncTask的的活儿:)
###Markdown的支持
我folk下来的leanote android是不支持markdown的，不过我在工程目录下找到了相关的js和html，而iOS版本是支持markdown并且也使用了类似的代码，所以照着iOS做就好了~
用Chrome一点点调试了解js editor的功能之后，发现这个editor还是挺强大的，甚至还有redo/undo的功能，我好像还没有见过哪个笔记软件提供这些功能呢。
工程中使用了一个EditNoteFragment来做UI控制以及与js通信的工作，这样再集成MD editor的时候UI就没法重用。为了解决这个问题，我把JS通信部分抽出来分别实现了RichTextEditor和MarkdownEditor，Fragment只管UI，这样就好多了。