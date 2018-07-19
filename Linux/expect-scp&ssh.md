## expect-scp和ssh-返回值
```bash
  #!/bin/bash
  # 全局变量：用户名，普通用户密码，root密码
  loginname="xxxx"
  userpwd="xxxx"
  rootpwd="xxxx"

  # 读取iplist(逐行存储IP地址)
  cat /root/iplist | while read line
  do
  ip=($line)

  # scp脚本及ssh登录执行
  /usr/bin/expect<<EOF
  set timeout 10
  spawn scp /root/check_and_fix.sh $loginname@$ip:/home/xxxx
  expect {
          "*yes/no" {send "yes\r";exp_continue}
          "*password:" {
                  send "$userpwd\r"
          expect eof
                  }
          }
  spawn ssh $ip -l $loginname -p 22
  expect {
          "*yes/no" {send "yes\r";exp_continue}
          "*password:" {
                  send "$userpwd\r"
                  expect "~]$"
                  send "su -\r"
                  expect "*assword:"
                  send "$rootpwd\r"
                  expect "~]#"
                  send "md5sum /home/xxx/check_and_fix.sh\r"
                  expect {
  # a4e3eb6a16f78129cf78d67d1c737ce9为上述文件正确的md5值，若正确则执行该脚本
                      "a4e3eb6a16f78129cf78d67d1c737ce9" { send "/home/xxxx/check_and_fix.sh\r"}
                  }
                  expect {
                      "存在漏洞并已修复" { exit 0 }
  # "存在漏洞并已修复"为脚本执行输出结果，匹配后退出expect并向bash提供返回值0
                  }
                  expect eof
                  exit 1    # 若md5值不正确，则匹配上一条expect eof后匹配exit 1，退出expect并向bash返回1
          }
  }
  EOF

  # 记录修复日志
  if [ $? -eq 0 ]
  then
          echo "$ip 漏洞修复完成" >> /var/log/patch.log
  else
          echo "$ip 漏洞修复失败" >> /var/log/patch.log
  fi
  done
```
