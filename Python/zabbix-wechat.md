#### zabbix 微信报警

1. 基于python3、Zabbix 3.2.4
2. 默认使用卡片类型消息发送
3. Zabbix告警联系人配置为企业微信的用户名，如同事张三用户名为zhangsan，报警接收人填写zhangsan即可，若多个接收人，使用"|"分隔
4. 报警媒介发送参数
    &nbsp;&nbsp;{ALERT.SENDTO}
    &nbsp;&nbsp;{ALERT.SUBJECT}
    &nbsp;&nbsp;{ALERT.MESSAGE}
5. 代码片段,"wechat.conf"微信参数配置文件，其中Corpid为企业号标识，Secret为管理凭证的密钥，Agentid为自定义应用的ID。该应用要对报警接收人部门有权限。将"wechat.conf"和"sendmessage.py"放置zabbix的alertscripts同一级目录。


```bash
[root@localhost zabbix]# more wechat.conf 
[wechat]
Corpid = xxxxxxxx 
Secret = xxxxxxxx
Agentid = 1000001
Zabbix_url = http://localhost/zabbix
```

```python
[root@localhost zabbix]# more sendmessage.py 
#!/usr/bin/python
#_*_coding:utf-8 _*_
import requests,sys,json,os
import configparser
from requests.packages.urllib3.exceptions import InsecureRequestWarning

# 由于requests设置移除SSL导致会产生安全请求警告,这里禁用安全请求警告
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

# 获取Token
def GetToken(Corpid,Secret):
    Url = "https://qyapi.weixin.qq.com/cgi-bin/gettoken"
    Data = {
        "corpid": Corpid,
        "corpsecret": Secret
    }
    r = requests.get(url=Url,params=Data,verify=False)
    Token = r.json()['access_token']
    return Token

# 发送普通文字消息
def SendMessage(Token,User,Agentid,Subject,Content):
    Url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=%s" % Token
    Data = {
        "touser": User,
        "msgtype": "text",
        "agentid": Agentid,
        "text": {
            "content": Subject + '\n' + Content
        },
        "safe": "0"
    }
    r = requests.post(url=Url,data=json.dumps(Data),verify=False)
    return r.text


# 发送卡片消息
def SendCardMessage(Token,User,Agentid,Subject,Content,Zabbix_url):
    Url = "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=%s" % Token
    Data = {
        "touser": User,
        "msgtype": "textcard",                             
        "agentid": Agentid,
        "textcard": {
            "title": Subject,
            "description": Content,
            "url": Zabbix_url,
            "btntxt": "Zabbix"
        },
        "safe": "0"
    }
    r = requests.post(url=Url,data=json.dumps(Data),verify=False)
    return r.text

if __name__ == '__main__':
    cf = configparser.ConfigParser()
    cf.read(os.path.dirname(os.path.abspath(__file__))+"/wechat.conf")
    User = sys.argv[1]
    Subject = sys.argv[2]
    Content = sys.argv[3]
    Corpid = cf.get("wechat","Corpid")
    Secret = cf.get("wechat","Secret")
    Agentid = cf.get("wechat","Agentid")
    Zabbix_url = cf.get("wechat","Zabbix_url")
    Token = GetToken(Corpid, Secret)
    SendCardMessage(Token,User,Agentid,Subject,Content,Zabbix_url)
[root@localhost zabbix]# 
```
