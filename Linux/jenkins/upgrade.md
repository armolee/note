## Jenkins系统版本升级

* web界面右上角提示需要升级，点击下载download，浏览器会自动下载最版本的war包
* 获取到新waf包的下载链接后
```bash
  wget http://mirrors.shu.edu.cn/jenkins/war-stable/2.121.2/jenkins.war
```
* ps aux 获取目前正在运行的waf包路径
```bash
jenkins  29249  0.4 15.2 5886544 1223008 ?     Ssl  Jul25   5:01 /etc/alternatives/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
```
* 使用最新的waf包替换/usr/lib/jenkins/jenkins.war，重启服务
```bash
  mv /usr/lib/jenkins/jenkins.war /usr/lib/jenkins/jenkins.war.bak
  mv /root/jenkins.war /usr/lib/jenkins/
  systemctl restart jenkins
```
