# NSUnetwork
## 成都东软学院校园网自动连接/防止断连/开机自动连接校园网

# 使用方法
1. 运行程序（最好是带控制台版本的）
2. 编辑创建的配置文件（data.txt）
3. 保存后重启程序

# 如何获取“一次性密钥” ？
![PixPin_2024-10-22_17-29-30](https://github.com/user-attachments/assets/f5e1629a-0cda-4909-90c0-92561b342692)

# 如何编辑配置文件 ？
![PixPin_2024-10-22_17-32-07](https://github.com/user-attachments/assets/0ba88549-ad55-4d7f-9628-976ee488cd4f)

# 配置文件中的参数：
可以自定义一些参数，初次连接需要填写账号信息（学号、一次性密钥）

配置文件↓参数 说明

```json
{
    "username" : "学号",             #填写自己登录校园网的学号
    "password" : "一次性密钥",       #网页登录后cookie中获取
    "testtime" : 5,                 #检测网络连接状况时差（单位s）
    "timeout" : 5,                  #超时连接失败失败（单位s）
    "outInFirstRequest" : false,    #第一次请求就连接成功是否退出程序
    "autoOutOtherIp" : true,        #自动踢出其他正在连接的IP
    "networkName" : "学生-移动-100M" #自动连接的网络名称
}
```
# 运行图
![PixPin_2024-10-22_17-23-45](https://github.com/user-attachments/assets/38b1c0a5-4a4c-4a04-9b59-4d61a7f07caf)

# 如何开机自动联网
可以把程序放在`%UserProfile%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`文件夹里面或者设置计划任务以达到开机自启动

建议使用` nsunetowrk_NO-console.exe `进行开机自启动，这样会在后台运行

# 运行主逻辑
```py
while 1:
    if exitFlag == True:
        break
    try:
        getStste = testIPState(loginKey['username'], loginKey['password'])
        #print(getStste)
        print('>>>'+getStste['Message'])
        time.sleep(0.1)
        if getStste['Result']:
            if getStste['Message'] == '已上线IP，免认证！':
                time.sleep(loginKey['testtime'])
                continue
            if getStste['Message'] == '已登录IP，免认证！':
                print("!>>尝试上线本机")
                getStste = linkNetwork(loginKey['username'], loginKey['password'])
                if getStste['Message'] == '同时登录数已达上限！':
                    if not loginKey['autoOutOtherIp']:
                        time.sleep(loginKey['testtime'])
                        continue
                    #print("其他设备正在上线中，将在10秒后重试")
                    print(">>>其他设备正在上线中，正在尝试查询其他设备")
                    getStste = listLinkNetwork(loginKey['username'], loginKey['password'])
                    time.sleep(0.1)
                    print(">>>其他设备正在上线中，正在尝试查询其他设备")
                    getStste = delLinkNetwork(loginKey['username'], loginKey['password'], getStste['Data']['OIA'][0]['IP'])
                    if(getStste['Result']):
                        print(f">>>已成功下线IP({getStste['IP']})，正在尝试上线本设备")
                        time.sleep(0.1)
                        continue
                    print(f">>>下线IP({getStste['IP']})失败，请求返回信息{getStste['Message']}")
                    time.sleep(5)
                continue
        if getStste['Result'] == 'needLogin':
            print("!>>尝试自动登录")
            loginNetwork(loginKey['username'], loginKey['password'])
        if not getStste['Result']:
            if getStste['Message'] == '请求太频繁，请稍后再试...':
                print("3秒后重试")
                time.sleep(3)
    except:
        print("!>>something error")
        time.sleep(3)
```
