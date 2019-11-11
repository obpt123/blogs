# TeamCity 入门教程

### 什么是[TeamCity](https://www.jetbrains.com/teamcity/)
    
TeamCity是Jetbrains的一个CI产品，引用官方的一句话：*Powerful Continuous Integration out of the box*,翻译成中文就是**开箱即用的强大持续集成工具**，官方网站上给出了*FREE FOREVER*，意思就是永久免费，但使用上会稍微有一点限制，比如最大并发的任务数量不能超过3和最大的Build configuration不能超过100，但对于一般的小中型项目来说，免费版本已经足够用了。




### Teamcity工作原理
Teamcity产品分为Server和Agent两部分，Server是统一的入口，类似于全局调度器，Agent负责执行具体的CI任务，一个Server可以对应多个Agent。理解这个架构对于接下来的操作是非常有帮助的，至少搭建环境的时候，除了安装Server端之外，至少还需要安装一个Agent端。

### TeamCity环境搭建

如果以学习为目的，可以在TeamCity官方网站上在线试用60天，只要注册一下或者用你的Github授权一下就可以了。如果是需要在企业环境搭建TeamCity，官方网站上提供了安装包，支持Windows，Mac和Linux版的，点击[这里](https://www.jetbrains.com/teamcity/download/#section=section-get)即可链接到下载页面。除了安装包之外，TeamCity还提供了Docker的安装方式，本文采用Docker的方式安装TeamCity。当然使用Docker安装的话需要你有Docker的环境并且掌握了Docker的使用方法，但这不在本文的范畴之内，如果读者还不熟悉Docker的话请先学习Docker的使用后再来学习本教程。


##### 安装Server
1. Docker安装并启动Server服务
    直接运行以下命令，注意替换掉命令中<>内的参数为实际有效参数。
    ```sh
    docker run -it --name teamcity-server-instance \
    -v <path to data directory>:/data/teamcity_server/datadir \
    -v <path to logs directory>:/opt/teamcity/logs \
    -p <port on host>:8111 \
    jetbrains/teamcity-server
    ```
    这里解释下上面的命令，上面第二行尖括号中需要指定一个本地目录用作存储Teamcity Server的数据，第三行需要指定一个本地目录用于存储TeamCity Server的日志文件，第四行需要指定TeamCity Server对外提供的端口号。

    ![docker install](/assets/docker%20install.PNG)
1. 获取Super user Token
    上一步命令执行完之后，等待一段时间，主要是需要下载image，之后会启动Teamcity Server，当你看到输出类似`Super user authentication token：xxxxxxxxxxxx`这样的内容时，说明Teamcity Server 已经启动成功，这里需要将此token的内容记录下来，方便下一步操作。

    ![installok](/assets/installok.PNG)

1. 在浏览器里访问Teamcity的服务
    访问地址格式如下：`http://<你的ip地址>:<第一步中配置的端口号>`,初次访问时，会让你选择数据库，同意协议等，接下来Teamcity会花一些时间初始化系统。
    
    ![starting](/assets/starting.PNG)

1. 登录TeamcityServer
    上一步初始化完成后，会自动跳转到创建管理员账户的界面。
    ![FirstLogin](/assets/FirstLogin.PNG)
    这里你可以创建一个Teamcity的管理员账户，或者点击`Login as Super user`,跳转到超级用户登录的界面，用第二步中获取到的Token就可以直接登录。
    ![superLogin](/assets/superLogin.PNG)
1. 登录成功
    看到以下界面，说明已经登录成功。
    ![createproject](/assets/createproject.PNG)
    此时Teamcity Server已经安装完成，但注意顶部菜单栏上面的Agents菜单，显示0个Agent，说明此时还没有注册Agent，接下来我们安装Angent并注册。

##### 安装Agent
1. docker安装并启动Agent服务
    ```sh
    docker run -it -e SERVER_URL="http://<Teamcity Server服务的ip>:<port>" \
    -v <path to agent data directory>:/data/teamcity_agent/conf \
    jetbrains/teamcity-agent
    ```
    运行是需要替换上面命令中的<>部分为实际有效值，<path to agent data directory>是agent的数据目录，这个和Server的目录没有任何关系。特别要注意一点的是Server_URL的ip地址部分，必须是在agent所在的容器能够访问到的server的ip地址。假如你在一台机器上既安装了TeamCity的 Server 和 Agent，访问地址一定不能是`127.0.0.1`或者是`localhost`,因为对于Agent的容器来说，这两个地址都是它本身，这样肯定是不能访问到Server服务的。
    ![installAgent](/assets/installAgent.PNG)

1. 注册Agent服务
    等待上一步完成后，在Teamcity的Web界面上选择Agent选项卡就可以看到`Unauthorized`状态的一个新Agent，点击`Authorize`即可对新加入的Agent注册。
    ![addagent](/assets/addagent.PNG)

1. 完成Agent安装 
    当在Teamcity的Web界面顶部看到Agent的数量不再是0的时候，就说明Server和Agent的连接状态已经正常，就可以正常运行Build任务了。如果想要注册多个Agent，重复安装Agent的步骤即可。

### 在TeamCity上创建PipeLine

