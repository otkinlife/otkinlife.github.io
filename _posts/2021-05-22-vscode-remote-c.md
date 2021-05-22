---
title: 使用VsCode 远程调试C语言
date: 2021/5/22 09:52:25
tags: 
    - ["奇淫技巧","工具使用"]
categories:
    - 工具
---

## 使用VsCode 远程调试C语言



> 最近学习系统调用的时候发现Mac上有很多系统调用都没有，只能在Linux系统上实践。
> 
> 但是我思前想后也没有勇气去Linux的小黑窗里敲代码&断点调试
> 
> 于是我决定使用VsCode进行远程调试，下面记录一下过程

### 准备本地的环境

#### 安装VsCode

这一步不用多说，懂得都懂

这里贴一个传送门 [VsCode官方网站](https://code.visualstudio.com)

#### 安装VsCode插件

1. 打开VsCode，选择左侧边栏的插件 》 搜索“C/C++” 》安装插件
<img src = "{{site.url}}/images/blog/vscode-c-1.png"/>
2. 安装完上述插件之后，继续搜索“Remote Development” 》 安装插件
<img src = "{{site.url}}/images/blog/vscode-c-2.png"/>

### 准备远程环境
1. 一定要支持ssh，VsCode的远程都是基于这个。配置好ssh的免密登录 
2. 进入远程机器开始准备环境（我的远程环境是Centos，其他Linux系统大同小异）
   - 安装gcc编译环境
        ```shell
        yum -y install gcc gcc-c++ kernel-devel
        ```

### 开始调试
1. 环境准备好之后就可以开始实践了，首先我们打开VsCode配置远程连接
   - 选择左侧边栏远程设备按钮 》 选择SSH Targets 》 点击+ 》 输入远程连接的ssh命令
   - 弹出配置存储的地方根据个人习惯即可（一般是.ssh/config）
    <img src = "{{site.url}}/images/blog/vscode-c-3.png"/>
   - 配置完成之后在“SSH TARGETS”下发会出现刚刚配置的设备
   - 在设备右侧有一个添加目录的按钮 <img src = "{{site.url}}/images/blog/vscode-c-4.png"/>。点击配置目录
   - 该目录将会是远程设备的工作目录，也就是在VsCode中编写的文件其实都是直接操作该目录下的文件
    <img src = "{{site.url}}/images/blog/vscode-c-5.png"/>
   - 最后右键刚刚配置好的远程设备，选择一个打开方式</br>
     <img src = "{{site.url}}/images/blog/vscode-c-6.png"/> 

2. 完成上述步骤就可以使用VsCode在服务器上开发了，开发的文件都会在远程设备上
     <img src = "{{site.url}}/images/blog/vscode-c-7.png"/>
3. 在完成开发之后，开始准备调试工作
   - 在VsCode中进行调试主要有两个工作：1. 配置编译任务； 2. 配置Debug运行；
      1. 配置编译任务，即配置使用什么编译指令去编译文件，这里主要是在项目目录的.vscode/tasks.json下配置（没有的话请自己创建）
        ```json
        {
            // https://go.microsoft.com/fwlink/?LinkId=733558
            "version": "2.0.0",
            "tasks": [
                {
                    "type": "process",
                    "label": "buildC", //
                    "command": "/usr/bin/gcc", //执行编译的命令，这里使用gcc也就是之前在远程设备上安装过的
                    //args是gcc后面跟的参数
                    "args": [
                        "-g",
                        "${file}", //需要编译的源文件
                        "-o", 
                        "${fileDirname}/out/${fileBasenameNoExtension}.out",
                    ],
                    "options": {
                        "cwd": "${workspaceFolder}"
                    },
                    "problemMatcher": [
                        "$gcc"
                    ],
                    "presentation": {
                        "echo": true,
                        "focus": false,  
                        "panel": "shared"   // 不同的文件的编译信息共享一个终端面板
                    },
                    "group": {
                        "kind": "build",
                        "isDefault": true // 不为true时ctrl shift B就要手动选择了
                    },
                    
                }
            ]
        }
        ```
        2. 配置Debug运行，VsCode的Debug需要配置相对应的参数，这个配置文件在.vscode/launch.json里（没有自行创建即可）。
        ```json
        {
            // https://go.microsoft.com/fwlink/?linkid=830387
            "version": "0.2.0",
            "configurations": [
                {
                    "name": "(gdb) Launch",
                    "type": "cppdbg",
                    "request": "launch",
                    //这里要改成task中配置的输出的文件地址
                    "program": "${fileDirname}/out/${fileBasenameNoExtension}.out",
                    "args": [],
                    "stopAtEntry": false,
                    "cwd": "${workspaceFolder}",
                    "environment": [],
                    "externalConsole": false,
                    "MIMode": "gdb",
                    "setupCommands": [
                        {
                            "description": "Enable pretty-printing for gdb",
                            "text": "-enable-pretty-printing",
                            "ignoreFailures": true
                        }
                    ],
                    "preLaunchTask": "buildC" //这里表示debug开启之前执行的任务，对应task中的label字段
                }
            ]
        }
        ``` 
    - 切换到开发的文件，定义断点，按F5开始调试
      <img src = "{{site.url}}/images/blog/vscode-c-8.png"/>
  