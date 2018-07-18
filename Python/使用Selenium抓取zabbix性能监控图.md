####概要
老样子先来个概要，因为公司一直都子使用企业微信，前两天刚刚将所有zabbix的告警信息接到微信上，总感觉还差那么一点，就想着就高等级的告警加上一个性能监控图，这样只看告警信息大概就清楚问题了，不用在登录到zabbix再去查找信息，就因为这个想法让我掉坑里呆了一整天，蓦然回首还是自己太菜啊，下面先贴出来使用Python登录zabbix并且获取到对应告警项监控图的代码，之后完整代码会贴到github中，链接放到简书上~
####实现抓取
以下几点基础：
1、告警信息的发送message里，可以发送zabbix的宏变量Item ID信息
2、根据Item ID，可以直接使用特殊URL携带Item ID信息获取对应监控截图
以下为获取截图完整代码：

    #!/usr/bin/env python
    # -*- coding:utf-8 -*-
    # name image.py
    from selenium import webdriver
    import os
    import sys
    reload(sys)
    sys.setdefaultencoding('utf-8')

    def get_item_graph(itemid,flag,eventid):
	    temp_name = "/tmp/"+eventid+".png"
            #save_screenshot仅能保存png格式图片，所以文件名定义需要以png结尾
	    driver = webdriver.PhantomJS("/usr/local/zabbix-agent-ops/phantomjs-2.1.1/bin/phantomjs",service_log_path=os.path.devnull)
            #使用PhantomJS可以模拟浏览器进行访问
	    driver.get("http://127.0.0.1/zabbix/")
	    driver.set_window_size(640,480)
	    driver.find_element_by_id("name").send_keys("armo")
	    driver.find_element_by_id("password").send_keys("123456")
	    driver.find_element_by_id("enter").click()
            #模拟访问url，在对应的元素element处输入用户名密码后click登陆
	    if flag:
		    driver.get("http://127.0.0.1/zabbix/history.php?action=showgraph&fullscreen=1&itemids[]="+itemid)
	    else:
		    driver.get("http://127.0.0.1/zabbix/history.php?action=showvalues&fullscreen=1&itemids[]="+itemid)
            #flag如果是1，则对应Item有对应的graph获取；如果是0，则获取最新一段时间的值
            driver.save_screenshot(temp_name)
            #将网页内容保存为png图片
	    driver.close()
	    driver.quit()
	
    if __name__ == "__main__":
        if len(sys.argv) > 1:
            itemid = sys.argv[1]           #脚本传递的第一个参数 Item ID
            flag = sys.argv[2]             #脚本传递的第二个参数 Flag，从zabbix数据库item和graph的对应表查询item是否具有对应的graph，如果有则传递1到脚本，无传递0
            eventid = sys.argv[3]          #脚本传递的第三个参数 告警信息的Event ID，用来命名png图片
	    get_item_graph(itemid,flag,eventid)
            #以上三个参数为前置调用脚本分析zabbix的发送信息后得到，之后会更新在后续文章中
####遇到的问题
1、第一个session和cookies的保存问题，如果登录之后不携带cookies是无法访问对应监控页面的，刚开始选择requests模块，感觉相对较难，对于刚入门的我来说还是selenium比较合适，不用考虑session和cookies的问题
2、脚本在测试环境运行非常好，截图信息也很准确，但是到线上服务器就出了问题，上俩个图：
很明显线上环境返回的图，前面一段居然是乱码。检查了所有环境，最后才发现问题出在线上服务器没有安装和zabbix适配的字体问题上，很是无奈啊，居然还要考虑字体的问题，阿里云的服务器也是真够能偷懒的。
![正常](http://upload-images.jianshu.io/upload_images/6328743-c9f2e9d43bdc711d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![不正常](http://upload-images.jianshu.io/upload_images/6328743-3afa46daa50f8ffc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、安装字体
为了确保万无一失，直接安装zabbix调用的字体到系统中
    [root@zabbix ~]# cp /var/www/html/zabbix/fonts/DejaVuSans.ttf /usr/share/fonts/
    [root@zabbix ~]# mkfontscale 
    [root@zabbix ~]# mkfontdir
    [root@zabbix ~]# fc-list :lang=zh
    Fangsong ti:style=Regular
    AR PL ShanHeiSun Uni:style=Regular
    [root@zabbix ~]# 
安装成功之后再次运行脚本后，截图一切正常了~~~~终于可以安心睡觉了