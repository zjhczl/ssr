[TOC]	
# 安装谷歌浏览器 chrome

## 方法1
	sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/
	wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
	sudo apt-get update
	sudo apt-get install google-chrome-stable
	启动: google-chrome-stable
## 方法2
    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
    sudo dkpg -i XXX.deb

## chrome jetson 下不断输入密码的解决方法

>Q18：如何解决Chrome需要输入密码解锁问题？

>A18：在终端安装 sudo apt-get install seahorse 删除登录密钥，然后设置密码为空解决。

* 具体操作:
1.按下＂win键＂
2.搜索＂password＂
3.打开后应该有好几个密钥归类，默认应该有个＂login＂或＂登陆＂
4.直接在login上右键,然后删除login的大选项
5.打开chrome
6.则出现提示对话框，提示输入新的密码环密码。不用输入，直接点击 Continue，然后则弹出下面的对话框，警告设置了一个空白密码，存储的密码不能被安全地封装等。直接点击 Continue 即可。 最后重启电脑，测试打开谷歌浏览器，不再提示输入密码环。


>https://www.jianshu.com/p/e73471cbe5cd   中的方法二

# 安装ssr客户端

	下载3.0可以通过 
```shell
# 需要用sudo命令，否则会出现权限不足情况
sudo pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip -U
```
   若有警告提示,可根据提示下载:
      sudo apt-get install shadowsocks

# 新建并编辑配置文件

## **客户端配置文件**

	新建一个命名cc.json的文件,并配置如下：


```json
{
    "server":"代理服务器地址",
    "server_port":代理服务器端口,
    "local_address":"127.0.0.1",// 0.0.0.0  可以让本机变为公共的代理服务器 可以用ubuntu官方代理一起使用（本地地址）
    "local_port":1080,//（本地端口可以修改为别的，只要别和自己的应用或服务端口冲突）
    "password":"代理服务密码",
    "timeout":300,//(超时时长限制)
    "method":"aes-256-gcm"//(加密方式，很多种，自选)
}
```

## **服务端配置文件**

	新建一个命名ss.json的文件,并配置如下：


```json
{
"server":"0.0.0.0",
"server_port":6317,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"wakingwang870417",
"method":"rc4-md5",
"timeout":300,
"fast_open": false
} 
```
## 详细配置
终端输入 ssserver 后回车会见到以下提示：
```
Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
                         Sodium:
                            chacha20-poly1305, chacha20-ietf-poly1305,
                            xchacha20-ietf-poly1305,
                            sodium:aes-256-gcm,
                            salsa20, chacha20, chacha20-ietf.
                         Sodium 1.0.12:
                            xchacha20
                         OpenSSL:
                            aes-{128|192|256}-gcm, aes-{128|192|256}-cfb,
                            aes-{128|192|256}-ofb, aes-{128|192|256}-ctr,
                            camellia-{128|192|256}-cfb,
                            bf-cfb, cast5-cfb, des-cfb, idea-cfb,
                            rc2-cfb, seed-cfb,
                            rc4, rc4-md5, table.
                         OpenSSL 1.1:
                            aes-{128|192|256}-ocb
                         mbedTLS:
                            mbedtls:aes-{128|192|256}-cfb128,
                            mbedtls:aes-{128|192|256}-ctr,
                            mbedtls:camellia-{128|192|256}-cfb128,
                            mbedtls:aes-{128|192|256}-gcm
  -t TIMEOUT             timeout in seconds, default: 300
  -a ONE_TIME_AUTH       one time auth
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers=WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip=IPLIST  comma seperated IP list forbidden to connect
  --manager-address=ADDR optional server manager UDP address, see wiki
  --prefer-ipv6          resolve ipv6 address first
  --libopenssl=PATH      custom openssl crypto lib path
  --libmbedtls=PATH      custom mbedtls crypto lib path
  --libsodium=PATH       custom sodium crypto lib path

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information


```
# 启动服务


## 1.服务端
### 1.1 开启BBR

由于ubuntu设置，并未默认开启BBR。

对内核4.9之前的版本可以直接使用以下语句升级内核打开BBR。

```shell
sudo wget -no-check-certificate -qO 'BBR.sh' 'https://moeclub.org/attachment/LinuxShell/BBR.sh' && sudo chmod a+x BBR.sh && sudo bash BBR.sh -f
```
*打开feature

由于1804内核本身已经是4.15版本，我们可通过以下命令开启BBR。

```shell
sudo modprobe tcp_bbr
echo "tcp_bbr" | sudo tee --append /etc/modules-load.d/modules.conf
echo "net.core.default_qdisc=fq" | sudo tee --append /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee --append /etc/sysctl.conf
sudo sysctl -p
```

执行完毕后可通过以下指令确认是否已开启BBR。
```shell
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr
```
若输出行中包含bbr字样则表示开启正确。

系统和BBR都准备完毕后就可以安装SSR了。
### 1.2 启动ssr服务端

```shell
	ssserver -c ss.json -d start 
```

## 2.客户端

```shell
sslocal -c cc.json start
```

# 使用
## 启动命令：

```
ssserver -c /etc/shadowsocks.json -d start
```
 


## 命令查看
``` 
netstat -tunlp 
```


## 命令停止
```
ssserver -c /etc/shadowsocks/config.json -d stop
```


 
## 完整路径命令
```
/usr/bin/ssserver -c /etc/shadowsocks.json --log-file=/tmp/ss.log -d start
```

## 代理客户端使用
	启动ubuntu系统自带的 sockets  代理设置即可
	最后打开 chrome 直接上网 (谷歌浏览器默认的代理只有是系统自带的)
	good luck!
## 其他ubuntu的联合使用
> 当一台ubuntu电脑终端打开客户端时,本机会在设置界面的网络--代理--socks设置里填写 0.0.0.0:1080的字样连接本机的代理,同样在同一个局域网下的另一台ubuntu电脑也可以通过界面的网络--代理--socks设置里填写 ${那台运行ssr代理的电脑的IP}:1080 来共享ssr代理.


## ubuntu 使用全局代理的一般方法
```shell
export http_proxy=http://127.0.0.1:1080
export https_proxy=https://127.0.0.1:1080
export ALL_PROXY=socks5://127.0.0.1:1080
```
这种用法只限于在终端中使用，在当前终端中使用是只是在此终端使用，写到bashrc里的话则是所有终端都可以使用。但不适合终端的子进程和带gui的程序。

## ubuntu 使用全局代理的终极方法 proxychains4
前面的ssr 在ubuntu的使用，多多少少有局限，比如在终端中就不能全部使用全部代理，子进程会出现报错。
这里推荐使用proxychains4
使用方法： 在终端命令前加  proxychains4 即可。
安装配置方法如下：
```shell
# 1.安装proxychains
sudo apt-get install proxychains4   # proxychains为旧方案，这里不推荐使用，这里使用proxychains4
# 2.配置 
sudo vim /etc/proxychains4.conf
#修改 >   socks 127.0.0.1 9095 -->
socks5 127.0.0.1 1080

#3. 使用方法
## 然后对于任何程序，只要在其前面加上proxychains命令就可以，例如：

proxychains4 xxx

xxx的所有连接就可以走proxychains4 了
```
## 手机使用SSR

<center>

![image](./file/v2rayNG/pic/v2rayNG.png)
<b>
推荐APP:v2rayNG
平台:安卓
</b>
</center>

下载连接(google paly):https://play.google.com/store/apps/details?id=com.v2ray.ang
本地下载:[点击下载](./file/v2rayNG/v2rayNG.apk)



# 参考

>https://www.ubuntukylin.com/ukylin/forum.php?mod=viewthread&tid=188059

>https://blog.csdn.net/qq_30101131/article/details/84641200?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3

>https://wmdpd.com/awsshang-bu-shu-ssr/


# 问题

```shell
gray@localhost:~$ sudo ssserver -c /etc/shadowsocks.json -d start
sudo: ssserver: command not found
```

这里就提示命令没找到。明明已经安装成功为什么加了sudo就提示命令找不到？查看文件路径

```shell
gray@localhost:~$ whereis ssserver
ssserver: /home/gray/.local/bin/ssserver
```

被安装到在了个人文件目录下面，不是全局环境，所以加了sudo不能找到，到时候要随系统自动启动也不方便。开始以为是版本问题，发现问题在第二步没加sudo直接安装了，加上sudo重新安装测试

```shell
sudo pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip
```

查看安装路径

```shell	
gray@localhost:~$ sudo ssserver --version
Shadowsocks 3.0.0
gray@localhost:~$ whereis ssserver
 ssserver: /usr/local/bin/ssserver
```

已经安装到/usr目录下面，重新启动就正常了。

## python3.10 的问题

运行 ssserver 报错：
```shell
AttributeError: module 'collections' has no attribute 'MutableMapping'
```
原因是python 3.10 那些 MutableMapping，MutableSet等放的位置变了，他们的上级模块原本直属collections的变成了abc，也就是说，需要把

```shell
collections.MutableMapping
```
改成
```shell
collections.abc.MutableMapping
```
直接修改源码既可以。
