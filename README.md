# kcptun
TCP流转换为KCP+UDP流，[下载地址](https://github.com/xtaci/kcptun/releases/latest) 用于
***高丢包***
环境中的数据传输，工作示意图:      
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
***kcptun是kcp协议的一个简单应用，可以用于任意tcp网络程序的传输承载，可以极大的提高软件网络流畅度(如浏览器，telnet等)，降低掉线，连不上等情况。***   

### 使用方法
```
D:\>client_windows_amd64.exe -h
NAME:
   kcptun - kcptun client

USAGE:
   client_windows_amd64.exe [global options] command [command options] [arguments...]

VERSION:
   20160507

COMMANDS:
   help, h      Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --localaddr, -l ":12948"     local listen addr:
   --remoteaddr, -r "vps:29900" kcp server addr
   --key "it's a secrect"       key for communcation, must be the same as kcptun server [$KCPTUN_KEY]
   --mode "fast"                mode for communication: fast, normal, default
   --tuncrypt                   enable tunnel encryption, adds extra secrecy for data transfer
   --help, -h                   show help
   --version, -v                print the version

D:\>server_windows_amd64.exe -h
NAME:
   kcptun - kcptun server

USAGE:
   server_windows_amd64.exe [global options] command [command options] [arguments...]

VERSION:
   20160507

COMMANDS:
   help, h      Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --listen, -l ":29900"                kcp server listen addr:
   --target, -t "127.0.0.1:12948"       target server addr
   --key "it's a secrect"               key for communcation, must be the same as kcptun client [$KCPTUN_KEY]
   --mode "fast"                        mode for communication: fast, normal, default
   --tuncrypt                           enable tunnel encryption, adds extra secrecy for data transfer
   --help, -h                           show help
   --version, -v                        print the version
```
### 适用范围（包括但不限于）:           
1. 网络游戏的数据传输        
2. 跨运营商的流量传输               
3. 其他高丢包，高干扰通信环境的TCP数据传输

