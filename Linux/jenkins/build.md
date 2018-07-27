## 持续构建任务

* 源码管理
  * git url
  * Credentials
    * 链接git的凭证，如果使用http方式连接git，此处使用用户名密码的凭证；若使用ssh方式，此处使用密钥的方式，gitlab中放置公钥，此处放置私钥
  * Branches
    * 分支管理，哪些分支会触发该构建任务，支持正则。常用*/test和*/master

* 构建触发器
  * SCM 轮询机制，此处可填写定时任务类型的工作频率，如果为空，则只执行post-commit钩子触发的任务
  * post-commit钩子需要安装Gitlab Hook Plugin的插件
    * http://your-jenkins-server/git/notifyCommit?url=<URL of the Git repository for the Gitlab project>&branch=branchname
    * &branch=branchname 可省略，省略会检测所有分支
    * hook的配置在gitlab仓库-settings-integrations，url填上上述hook地址即可，trigger可勾选触发hook的动作类型
* 构建
  * shell的执行
    * 因为jenkins服务是以jenkins用户运行的，所有默认执行shell的用户也是jenkins
    * jenkins在创建时shell为/bin/false，所有加载环境变量是个问题，所以我们需要这样来做以加载环境变量
    ```bash
      #!/bin/bash -ilex
    ```
