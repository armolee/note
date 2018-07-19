## android的持续集成

 * gradle安装
```bash
 curl -s "https://get.sdkman.io" | bash 安装sdk包管理工具
sdk install gradle 3.1  #安装指定的gradle版本
```  

* gradle使用sock代理，在gradle.properties文件中增加
```bash
org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080  
```  
* 下载sdk，http://sdk.android-studio.org/  http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
* sdk不能放到/root下，否则会提示SDK不存在，因为/root下禁止其他用户访问，并且jenkins需要对该目录有可写可执行权限  
* 更新sdk，
```bash
tools/android update sdk --no-ui
```
* 提示Warning: License for package Android SDK Build-Tools 27.0.3 not accepted,版本license问题更新对应版本  
```bash  
tools/android list sdk --all   #查看所有版本，查找缺少的license的版本编号
tools/android update sdk -u -a -t 4  #更新对应版本，4位对应版本编号
```  
* sdk环境变量   
```bash
export ANDROID_HOME=/opt/android-sdk-linux
export PATH=/opt/src/node-v8.11.1-linux-x64/bin:/opt/android-sdk-linux/tools:/opt/android-sdk-linux/platforms:$PATH
```  

* Jenkins环境变量配置


* Jenkins提示构建失败  

```bash
What went wrong:
Gradle build daemon disappeared unexpectedly (it may have been killed or may have crashed)
查看message，被系统oom，gradle打包过程中需要大概4G以上的内存，升级服务器后构建成功
```
