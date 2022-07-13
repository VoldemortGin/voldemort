# 林晗的博客

## 使用frp+docker实现内网穿透

> 注: 这里的IP指的都是IPv4.
>
> 因为工作的原因, 需要对一台没有内网IP的ubuntu系统电脑进行一些纯命令行的操作.
>
> 之前使用的是向日葵界面连上去后打开Terminal, 但是免费版的向日葵有一堆毛病, 文件不让传, 命令不让复制只能手打等.
>
> 机缘巧合得知了frp这个内网工具(一开始想用Zerotier, 也是免费的, 但前辈告知这个免费版的也不稳定), 简单学习了一个就部署上去了, 分享一下过程.

### 首先要有一台有公网IP的云服务器

最基础的配置就行.

在里面运行如下命令:

```bash
wget --no-check-certificate https://raw.githubusercontent.com/clangcn/onekey-install-shell/master/frps/install-frps.sh -O ./install-frps.sh
chmod 700 ./install-frps.sh
./install-frps.sh install
```

这里aws的好处之一是不需要为githubusercontent网站在国内连不上而发愁. 安装过程中需要填入一些端口号之类的参数, 基本都填默认即可.

安装完成后会有各种参数提示, 记得截图保存下来.

三个重要命令:

```bash
frps start
frps stop
frps restart
```

至此, 服务器端的frps安装完成. 可以通过 http://你的服务器公网ip:6443 查看面板了(如果安装中的端口号你都选默认的话这里端口就是6443).

![image](https://blog.e9china.net/wp-content/uploads/2018/12/1-1.jpg)

### 目标机器安装frpc

这里用到运维界的小无相功--Docker大法了. 

先在某处新建一个`frpc.ini`文件, 并编辑如下信息:
```ini
[common]
server_addr = aws的公网ip地址
server_port = 5443
token = 前面aws的frps安装结束时出现的信息

[ssh]
type = tcp
local_ip = 192.168.3.18
local_port = 22
remote_port = 2222
```

直接在命令行运行:

```bash
docker run --name frpc --volume frpc.ini的绝对路径:/etc/frp/frpc.ini voldemort/frpc
```

完成.

### 使用
在命令行运行:
```bash
ssh -p 2222 用户名@aws的公网ip地址
```
输入ubuntu远程机器账户的用户名对应的密码, 成功, enjoy it!