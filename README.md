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
***kcptun是kcp协议的一个简单应用，可以用于任意tcp网络程序的传输承载，可以极大的提高软件网络流畅度(如浏览器，telnet等)，降低掉线，连不上等情况。***   

### 使用方法
执行client -h , server -h 查看

### 适用范围（包括但不限于）:           
1. 网络游戏的数据传输        
2. 跨运营商的流量传输               
3. 其他高丢包，高干扰通信环境的TCP数据传输      

# 免责申明
用户以各种方式使用本软件（包括但不限于修改使用、直接使用、通过第三方使用）的过程中，不得以任何方式利用本软件直接或间接从事违反中国法律、以及社会公德的行为。软件的使用者需对自身行为负责，因使用软件引发的一切纠纷，由使用者承担全部法律及连带责任。作者不承担任何法律及连带责任。       

对免责声明的解释、修改及更新权均属于作者本人所有。
