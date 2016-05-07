# kcptun
TCP流转换为KCP+UDP流，用于***高丢包***环境中的数据传输，工作示意图:      
```
                +---------------------------------------+
                |                                       |
                |                KCPTUN                 |
                |                                       |
+--------+      |  +------------+       +------------+  |      +--------+
|        |      |  |            |       |            |  |      |        |
| Client | +--> |  | KCP Client | +---> | KCP Server |  | +--> | Server |
|        | TCP  |  |            |  UDP  |            |  | TCP  |        |
+--------+      |  +------------+       +------------+  |      +--------+
                |                                       |
                |                                       |
                +---------------------------------------+
```
***kcptun可以用于任意tcp网络程序的传输承载，可以极大的提高软件网络流畅度(如浏览器，telnet等)，降低掉线，连不上等情况。***   

# 快速上手
***注意，请确保防火墙允许UDP包通过***

点 [这里下载](https://github.com/xtaci/kcptun/releases/latest) 最新的对应平台的版本(***内含x86/x64/arm***)。         
执行 ```client -h 和server -h``` 查看详细使用方法.  不同平台分别为```client_windows_amd64.exe或server_linux_amd64```这种平台对应文件名。

我们以加速ssh -D访问为例示范使用方法如下：         

1. 假定服务器(比如ubuntu) IP地址为:```xxx.xxx.xxx.xxx```

2. 在服务器端开启ssh -D, 监听127.0.0.1:8080端口，***或者你可以在服务器端启动任意的socks代理，例如dante(更快)，监听127.0.0.1:8080***         
```ssh -D 127.0.0.1:8080 ubuntu@localhost```   

3. 在服务器启动kcp server:     
```server -t "127.0.0.1:8080"  ```   // 所有接收到的数据包转发到sshd进程的socks 8080端口           
 ***_----------------------------  分割线，上面是服务器，下面是客户端  ----------------------------_***  
4. 在本地(比如win10)启动kcp client:          
```client -r "xxx.xxx.xxx.xxx:29900"   ```   // 连接到kcp server，默认kcp server端口是29900           

5.  将浏览器socks5代理设置为127.0.0.1:12948   // 默认kcp client的端口是12948           

***完整转发过程为: 浏览器 -> kcp-client(:12948) -> kcp-server(:29900) -> ssh -D(:8080)***

# 特性      
1. 超级快     
2. 跨平台       
3. 采用高安全性[AES-256-CFB](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)双重加密(包+流)             
4. UDP数据包一次一密([OTP](https://en.wikipedia.org/wiki/One-time_password))无特征，防非法深度检测       
5. 消息摘要采用[MD5](https://en.wikipedia.org/wiki/MD5)保证消息完整性      
6. [PSK](https://en.wikipedia.org/wiki/Pre-shared_key)防止[MITM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)攻击       
7. kcptun客户端和服务端分别只有一个main.go文件，易于使用      
8. 核心基于[kcp-go](https://github.com/xtaci/kcp-go)      
9. 基于[yamux](https://github.com/hashicorp/yamux) 的多路流复用( N:1 <<------>> 1:N)，自动重连
10. 三种传输模式: fast/normal/default         

***(注意：要实现高可靠的信息安全，上层TLS/SSL等加密手段依然不可或缺。)***

### 适用范围（包括但不限于）:           
1. 网络游戏的数据传输        
2. 跨运营商的流量传输               
3. 其他高丢包，高干扰通信环境的TCP数据传输      

# 参数调整
初步运行成功后，***强烈建议***通过命令行改变如下参数加强传输安全:         
1. kcp server默认端口        
2. 默认密码 ***必须修改*** 最好是字母数字大小写特殊字符的组合(前往 [密码生成](https://identitysafe.norton.com/password-generator/))  
3. 额外的隧道安全，可以通过 -tuncrypt 在server/client两端同时开启，即使PSK被猜到也难以破译                   

例如:       
```server -tuncrypt -l ":41111" -key "yqRhM5T5"```       

# 加密流程         
```
                               /dev/urandom
                                     +
+-----------+                        |
|           |                        |                 +---------+   PSK   +-----------+
| PLAINTEXT |                     +--v--+              |                               |
|           |                     | OTP |              |                               |
+-----+-----+                     +--+--+              |                               |
      |                              |                 |                               |
 +----v----+    +--------+       +---v----+       +----+----+                     +----+----+        +--------+                      +-----------+
 |         |    |        |       |        |       |         |                     |         |        |        |                      |           |
 | AES+CFB +----> CIPHER +-------> PACKET +-------->AES+CFB +----> Internet +------>AES+CFB +--------> PACKET |     Copyright by     | PLAINTEXT |
 | ENCRYPT |    |  TEXT  |       |        |       | ENCRYPT |                     | DECRYPT |        |        |     @xtaci           |           |
 |         |    |        |       +---^----+       |         |                     |         |        +---+----+                      +-----^-----+
 +----^----+    +---+----+           |            +---------+                     +---------+            |                                 |
      |             |           +----+----+                      /dev/urandom                            |                                 |
      |             |           |         |                            +                             +---v----+       +--------+      +----+----+
      |             +----------->   MD5   |                            |                             |        |       |        |      |         |
      |                         |         |                        +---v----+                        |  MD5   +-------> CIPHER +------->AES+CFB |
      |                         +---------+                        |        |                        | VERIFY |       |  TEXT  |      | DECRYPT |
      |                                                  +---------+   IV   +-----------+            |        |       |        |      |         |
      |                                                  |         |        |           |            +--------+       +--------+      +----^----+
      |                                                  |         +--------+           |                                                  |
      |                                                  |                              |                                                  |
      +--------------------------------------------------v--------+   PSK   +-----------v--------------------------------------------------+
```

# 从源码的安装
## 预备条件:       
1. 安装好```golang```       
2. 设置好```GOPATH```  以及```PATH=$PATH:$GOPATH/bin``` (例如: ```export GOPATH=/home/ubuntu;  export PATH=$PATH:$GOPATH/bin```), 最好放到.bashrc 或 .zshrc中 

## 安装命令
1. 服务端: ```go get github.com/xtaci/kcptun/server;  server```        
2. 客户端: ```go get github.com/xtaci/kcptun/client;  client```      

# 常见问题
Q: client/server都启动了，但无法传输数据，服务器显示了stream open        
A: 先杀掉client/server，然后重新启动就能解决绝大部分的问题             

Q: client/server都启动了，但服务器没有收到任何数据包也没有stream open          
A: 某些IDC默认屏蔽了UDP协议，需要在防火墙中打开对应的端口

Q: 出现不明原因降速严重，可能有50%丢包         
A: 可能该端口被运营商限制，更换一个端口就能解决

# 免责申明
用户以各种方式使用本软件（包括但不限于修改使用、直接使用、通过第三方使用）的过程中，不得以任何方式利用本软件直接或间接从事违反中国法律、以及社会公德的行为。         
对免责声明的解释、修改及更新权均属于作者本人所有。

![secure](secure.jpg)
