# 1. 前言

1. 通过云服务器+frp实现，内网穿透
   - 此处使用阿里的ECS服务器

2. 此处使用`192.168.10.1`代指公网可访问的服务器IP

<!--more--> 
   

# 2. 服务端配置

## 2.1 frp配置

1. 下载后解压

   ```bash
   wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_linux_arm64.tar.gz
   ```

2. 服务端对应文件为

   ```
   ├── frps
   ├── frps.toml
   ```

   修改`frps.toml`，指定frp使用的端口

   ```
   bindPort = 7000
   ```

3. 启动服务

   ```bash
   ./frps -c frps.toml
   ```
   
	输出
	
   ```
   2024/02/08 11:05:55 [I] [root.go:105] frps uses config file: frps.toml
   2024/02/08 11:05:55 [I] [service.go:225] frps tcp listen on 0.0.0.0:7000
   2024/02/08 11:05:55 [I] [root.go:114] frps started successfully
   ```

4. 设置ufw

   可以直接关闭：`ufw disable`

   

## 2.2 阿里云ECS安全组配置

1. 网址：https://ecs.console.aliyun.com/

   `安全组` -> `入方向` -> `手动添加`

2. 添加端口`7000`，我为了方便直接添加7000-7100的全部端口

   - 7000 是用来frp客户端和服务端之间访问使用的
   - 其他的是用来给开放给客户的

   注意：
   - 源要写成`0.0.0.0/0`

   ![image-20240208132414549](/images/frp服务搭建/image-20240208132414549.png)



## 2.3 验证端口是否可用

1. 找一台ubuntu

    ```
    apt install telnet
    ```

    ```
    telnet 192.168.10.1 7000
    ```

   应该可以看到
   ```
   Trying 192.168.10.1...
   Connected to 192.168.10.1.
   Escape character is '^]'.
   ```



# 3. 客户端配置

1. 下载后解压

   ```bash
   wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_linux_arm64.tar.gz
   ```

2. 服务端对应文件为

   ```
   ├── frpc
   ├── frpc.toml
   ```

   修改`frpc.toml`，指定frp使用的端口

   ```
   serverAddr = "192.168.10.1"
   serverPort = 7000
   
   [[proxies]]
   name = "test-tcp"
   type = "tcp"
   localIP = "127.0.0.1"
   localPort = 22
   remotePort = 7001
   ```

   - `serverAddr`和`serverPort`设置成frp服务端对应的
   - `remotePort`设置成一个除7000之外的端口，用于公网ssh连接
   - `localPort`是指将`remotePort`对应的服务映射到哪个端口上

3. 设置ufw

   可以直接关闭：`ufw disable`

4. 启动服务

   ```
   ./frpc -c frpc.toml
   ```

   可以看到

   ```
   2024/02/08 05:19:28 [I] [root.go:142] start frpc service for config file [frpc.toml]
   2024/02/08 05:19:28 [I] [service.go:287] try to connect to server...
   2024/02/08 05:19:28 [I] [service.go:279] [33fe641c603eba63] login to server success, get run id [33fe641c603eba63]
   2024/02/08 05:19:28 [I] [proxy_manager.go:173] [33fe641c603eba63] proxy added: [test-tcp]
   2024/02/08 05:19:28 [I] [control.go:170] [33fe641c603eba63] [test-tcp] start proxy success
   ```



# 4. ssh连接

此时直接使用

```
ssh -p 7001 root@192.168.10.1
```

即可连接到对应的客户端





# 参考

1. https://gofrp.org/zh-cn/docs/
2. https://github.com/fatedier/frp/releases





