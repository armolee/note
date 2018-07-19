## 企业微信发送Zabbix报警图文信息
####概要
入职之后，邮箱正常的业务邮件没几个，倒是每天被几百封Zabbix的报警邮件充斥，不仅杂乱无章，而且也不是那么愿意去看每封邮件的内容，刚好公司在用企业微信，所以打算把Zabbix的报警信息迁移到企业微信上去，大致思路如下：

- 高等级的报警信息，以图文格式发送报警内容和ITEM一个小时之内的趋势图
- 较低等级的报警信息，以卡片格式的消息发送（纯文本消息太丑）
- 以小时为单位整理出还在PROBLEM状态的EVENT告知管理员
####实现过程
- 第一个Python实现抓取告警ITEM对应的趋势图，具体代码如下，详细解释见前一篇文章：http://www.jianshu.com/p/7451a2a5af46


    #!/usr/local/bin/python
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
- 第二个Python实现连接企业微信API图文发送消息，此脚本中为调用卡片消息发送函数，我在实现分级发送时，采用了两个脚本，在Zabbix中判断告警级别使用不同脚本进行发送。
 

    #!/usr/local/bin/python
    #_*_coding:utf-8 _*_
    import requests,sys,json
    import urllib3
    urllib3.disable_warnings()
    reload(sys)
    sys.setdefaultencoding('utf-8')

    #获取Token
    def GetToken(Corpid,Secret):
        Url = "https://qyapi.weixin.qq.com/cgi-bin/gettoken"
        Data = {
            "corpid": Corpid,
            "corpsecret": Secret
        }
        r = requests.get(url=Url,params=Data,verify=False)
        Token = r.json()['access_token']
        return Token

    #将获取到的趋势图，上传至企业微信临时素材，返回MediaId发送图文消息是使用
    def GetImageUrl(Token,Path):
        Url = "https://qyapi.weixin.qq.com/cgi-bin/media/upload?access_token=%s&type=image" % Token
        data = {
            "media": open(Path,'r')
            }
        r = requests.post(url=Url,files=data)
        dict = r.json()
        return dict['media_id']

    #卡片消息    
    def SendCardMessage(Token,User,Agentid,Subject,Content):
        Url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=%s" % Token
        Data = {
            "touser": User,                                 # 企业号中的用户帐号
            "msgtype": "textcard",                          # 消息类型
            "agentid": Agentid,                             # 企业号中的应用id
            "textcard": {
                "title": Subject,
                "description": Content,
                "url": "http://127.0.0.1/zabbix/",          #点击详情后打开的页面
                "btntxt": "详情"
            },
        "safe": "0"
        }
        r = requests.post(url=Url,data=json.dumps(Data),verify=False)
        return r.text

    #图文消息
    def SendnewsMessage(Token,User,Agentid,Subject,Content,Image,Itemid):
        Url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=%s" % Token
        UUrl = "http://127.0.0.1/zabbix/history.php?action=showgraph&fullscreen=1&itemids[]="+Itemid
        Data = {
                "touser": User,                                 # 企业号中的用户帐号
                "msgtype": "mpnews",                            # 消息类型
                "agentid": Agentid,                             # 企业号中的应用id
                "mpnews": {
                        "articles": [
                        {
                                "title": Subject,
                                "thumb_media_id": Image,
                                "content": Content,
                                "content_source_url": UUrl,      #点击阅读原文后，打开趋势图大图，第一次需要登录
                                "digest": Content
                        }
                        ]
                },
                "safe": "0"
        }
        headers = {'content-type': 'application/json'}
        data = json.dumps(Data,ensure_ascii=False).encode('utf-8')
        r = requests.post(url=Url,headers=headers,data=data)
        return r.text

    if __name__ == '__main__':
        User = sys.argv[1]                                                            
        Subject = sys.argv[2]                                                       
        Content = sys.argv[3]                                                       
        Path = sys.argv[4]							                          
        Itemid = sys.argv[5]
        Corpid = "xxxxxxxxxxx"                                    # CorpID是企业号的标识
        Secret = "xxxxxxxxx"                                      # Secret是管理组凭证密钥
        Agentid = "100000x"                                       # 应用ID
        Token = GetToken(Corpid, Secret)
        Image = GetImageUrl(Token,Path) 
        SendnewsMessage(Token,User,Agentid,Subject,Content,Image,Itemid)
- 第三步，将zabbix数据库中graphs_items表的信息拉去到一个文件中，我们可以通过读取该文件获取对应Item的GraphID号，从而进行抓取趋势图。（也可以直接去库中查询，但是这个表除非有新的监控项一般不会变化，所以我这里将表拉去到文件中，通过在文件中对比减少库查询的压力，将拉取动作每天执行一次即可）

      [root@zabbix ~]# 
      1 1 * * *  /usr/bin/mysql -h127.0.0.1 -uzabbix -pPASSWD -e "select graphid,itemid from zabbix.graphs_items" > /tmp/zabbix.txt
- 第三个脚本，接收Zabbix传递的参数，并且过滤出ItemID，判断是有具有Graph并获取其ID，调用前两个脚本并且传递对应参数


    #!/bin/bash
    # name:wechat.sh

    send_to=$1
    subject=$2
    message=$3

    #获取itemid,eventid,PROBLEM或OK状态以命名PNG趋势图
    #这块几个grep的格式需要特别注意，需要和zabbix发送过来的message格式相同
    itemid=`echo $message | egrep -o "item ID: [0-9]*"| awk '{print $NF}'`
    eventid=`echo $message | egrep -o "event ID: [0-9]*"| awk '{print $NF}'`
    stat=`echo $message | egrep -o "Trigger status: PROBLEM|Trigger status: OK"| awk '{print $NF}'`
    image=/tmp/$eventid"_"$stat".png"
    #获取graphid和flag
    graphid=`/bin/cat /tmp/zabbix.txt | awk '{ if($2=="'"$itemid"'"){print $1}}'`
    if [ ! $graphid ]
    then
        flag=0
    else
        flag=1
    fi
    
    #在zabbix自动执行该脚本时，该脚本调用两个PY脚本，不太清楚原因，直接写脚本绝对路径无法执行，使用
    解释器后跟脚本绝对路径也无法执行，但是尝试cd到目录后执行确可以，很蛋疼
    cd /usr/local/zabbix/share/zabbix/alertscripts/
    #使用image.py生成/tmp/$eventid"_"$stat".png图片后传递给wechat.py才有意义
    ./image.py "$itemid" "$flag" "$image"
    ./wechat.py "$send_to" "$subject" "$message" "$image" "$itemid"
    #log
    echo $image >> /tmp/zabbix_event.log

- 到上面结束后，我们可以配置Zabbix之后就可以正常发送微信消息了，但是/tmp下生成的PNG图片还没有处理，上面以EVENT_ID+问题状态（PROBLEM|OK）来创建的文件，我们可以利用这些文件完成我的第三个需求


    #!/bin/bash
    # 定时任务脚本
    # 根据生成的PNG检查时段内未处理的问题，并删除已处理的PNG

    cat /tmp/zabbix_event.log | awk -F "_" '{print $1}' | sort | uniq -c | awk '{if($1%2==0){print $2}}' > /tmp/zabbix_rm.log
    event=`cat /tmp/zabbix_rm.log`
    for i in $event
    do
        sed -i '/[$i]/d' /tmp/zabbix_event.log
        rm -rf $i*
    done

    cat /tmp/zabbix_event.log | awk -F "_" '{print $1}' | awk -F "/" '{print $NF}' > /tmp/zabbix_problem.log
    #最后生成的/tmp/zabbix_problem.log中，保存了目前所有PROBLEM的EVENT_ID,将此ID通过第二个脚本提供的方式再发发送消息给微信即可。
    #将此脚本添加到Crontab任务定时执行

####Zabbix配置简述（脚本放置文件夹为zabbix配置参数AlertScriptsPath定义）
- 新增报警媒介调用第三个脚本，wechat.sh

![调用第三个脚本，wechat.sh](http://upload-images.jianshu.io/upload_images/6328743-317625d2ca3b40d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 新增用户，调用上述报警媒介，用户名为需要接收消息人的企业微信用户名

![新增用户调用新增的报警媒介](http://upload-images.jianshu.io/upload_images/6328743-3387d50b7e54c571.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 配置动作，使用新增方式发送消息。动作的消息中需要注意以下三个参数的格式，在wechat.sh中需要精确匹配后提取相关信息。别切需要以html编码格式进行换行，否则发送后无换行动作看着很乱。
- 问题动作和恢复动作都需要发送信息，以下三个内容要相同  
    
      Trigger status: {TRIGGER.STATUS}
      item ID: {ITEM.ID}
      event ID: {EVENT.ID}


![告警动作](http://upload-images.jianshu.io/upload_images/6328743-01e6fcccffc47f67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![恢复动作](http://upload-images.jianshu.io/upload_images/6328743-8a0fbae3d9b56bb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 具体告警信息

      <p>1、Trigger status: {TRIGGER.STATUS}</p>
      <p>2、Trigger severity: {TRIGGER.SEVERITY}</p>
      <p>3、{EVENT.DATE}--{EVENT.TIME}</p>
      <p>4、{ITEM.NAME1}--{ITEM.VALUE1}</p>
      <p>5、Original item ID: {ITEM.ID}</p>
      <p>6、Original event ID: {EVENT.ID}</p>
####效果展示
#####图文消息

![图文消息](http://upload-images.jianshu.io/upload_images/6328743-15a596a390e835ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点进去之后
![点进去之后](http://upload-images.jianshu.io/upload_images/6328743-bf5f4a057ff290f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点阅读全文，第一次需要登录，之后打开其他面板的阅读全都都不需要输入密码啦

![阅读全文](http://upload-images.jianshu.io/upload_images/6328743-40215bcc317997cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####卡片消息
是不是看着很清爽，低等级的告警，点详情后会跳转到zabbix主页面，卡片上一共可以显示八行消息，可以将需要的信息定义在前八行就可以了

![卡片消息](http://upload-images.jianshu.io/upload_images/6328743-fc5ce67496508081.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)