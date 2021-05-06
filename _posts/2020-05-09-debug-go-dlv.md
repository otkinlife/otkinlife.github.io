---
layout: post
title: DLV 调试工具 【go语言调试神器】
date: 2020-05-09
tags: ["奇淫技巧","工具使用"]
categories: 工具

---

### 安装DLV

*   常规安装方式

    <pre class="theme:vs2012 lang:zsh decode:true">go get github.com/go-delve/delve/cmd/dlv</pre>

*   使用go mod管理依赖推荐使用以下安装方法

    <pre class="theme:vs2012 lang:zsh decode:true  ">$ git clone https://github.com/go-delve/delve.git $GOPATH/src/github.com/go-delve/delve
$ cd $GOPATH/src/github.com/go-delve/delve
$ make install</pre>

*   [其它方式请参考官方说明](https://github.com/go-delve/delve "其它方式请参考官方说明")

### 实例代码

```go
//file main.go 
package main 
import ( "fmt" ) 
func main() {
    fmt.Println("123:") 
    fmt.Println("234:") 
    fmt.Println("456:") 
}
```

<!-- wp:heading {"level":3} -->

### 使用DLV进行本地调试

<!-- /wp:heading -->

<!-- wp:paragraph -->

执行命令,进入调式界面

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"theme:vs2012 lang:zsh decode:true"} -->
<pre class="wp-block-preformatted theme:vs2012 lang:zsh decode:true">dlv debug main.go</pre>
<!-- /wp:preformatted -->

<!-- wp:preformatted {"className":"theme:vs2012 lang:zsh decode:true"} -->
<pre class="wp-block-preformatted theme:vs2012 lang:zsh decode:true">b main.go:8                #在main.go的第8行设置断点
b name1 main.go:9          #在main.go的第9行设置断点，断点名称为name1 
b name2 main.go:10         #在main.go的第10行设置断点，断点名称为name2</pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->

查看所有断点，执行命令bp

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

![]({{site.url}}/images/blog/WeChat201fc1a59dc150835e79cdcbb05dd788.png) 

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

输入c开始运行，n为下一个，p为打印

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

![]({{site.url}}/images/blog/WeChat001b8f52969a3766b8dfa1a33425a761.png)

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

使用方式与gdb类似，也可以输入help查看使用说明 

<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->

### 使用Goland和DLV进行远程调试

<!-- /wp:heading -->

<!-- wp:group -->
<div class="wp-block-group"><div class="wp-block-group__inner-container"><!-- wp:paragraph -->

编译文件

<!-- /wp:paragraph -->

<!-- wp:freeform -->
<pre class="theme:vs2012 toolbar:2 lang:zsh decode:true">go build -gcflags "all=-N -l" main.go</pre>
<!-- /wp:freeform -->

<!-- wp:paragraph -->

运行文件

<!-- /wp:paragraph -->

<!-- wp:freeform -->
<pre class="theme:vs2012 toolbar:2 lang:zsh decode:true  ">dlv --listen=:2345 --headless=true --api-version=2 --accept-multiclient exec ./main</pre>
<!-- /wp:freeform -->


配置Goland


<!-- wp:paragraph -->

<img src="{{site.url}}/images/blog/WeChatfc83ee0205955cf29469f388c5842e9e.png">

<!-- /wp:paragraph -->



最后就可以直接在Goland里debug代码了
