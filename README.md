# map2cloud - 映射本地端口至云端
2017.9.13
开源工具

>很多时候，我们需要远程访问本地的服务，一般我们是借助于Teamviewer或者花生壳，而这往往需要人在对方电脑旁，还有一点，无法真正做到本机操作，体验比较差；而借助该工具就可以直接将远端服务映射到云端，我们只需要访问云端对应端口就能访问远程服务。

### 准备
在使用该工具，你至少需要准备一下几个内容

* 装有sshd的云端主机
* 一台装有linux系统的本地主机

### 安装
#### 1. 安装依赖工具 - autossh

可以使用yum或apt-get安装
```bash
# centos
yum install autossh

# debain
apt-get install autossh
```
#Ubuntu
sudo apt-get install autossh

如果以上方法无法安装，则请直接到[autossh官网](http://www.harding.motd.ca/autossh/)，下载源码并进行安装.

#### 2. 安装map2cloud
```bash
git clone https://github.com/yangwei1987/map2cloud.git

cd map2cloud 
cp map2cloud map2cloud
sudo chmod +x /usr/local/bin/map2cloud
```

#### 3. 免密登录
6.1 ssh每次重连都需要键入密码，故在此首先设置免密码登陆到内网

在内网的机器A上面执行：

ssh-copy-id 外网用户名@外网IP 

按照之前我设定的端口，这个指令就是如下

ssh-copy-id root@123.123.123.123

那以后这台内网的A机器ssh登陆我外网的B机器就可以免密码登陆啦~
检验是否已经可以使用免密码登陆可以使用如下指令来检验：

ssh root@123.123.123.123



#### 3. 生成rsa密钥
请使用ssh-keygen生成密钥对

```
ssh-keygen -t rsa
```
PS: 一路Enter就行，会在$Home/.ssh/下生成id_rsa和id_rsa.pub

#### 4. 开启云端sshd的GatewayPorts功能
默认情况sshd是不开启GatewayPorts，这样无法外部访问.

vim /etc/ssh/sshd_config 
```
...
GatewayPorts yes
...

```
保存退出后，重启sshd `service sshd restart`

### 使用

```
USAGE: map2cloud [OPTIONS] ssh-user@cloud-ip [local-ip:]local-port cloud-port

DESCRIPTION:
    ssh-user        The ssh user to access to cloud server.
    cloud-ip        The cloud server ip or domain.
    local-ip        The local ip to map, default is localhost.
    local-port      The local port to map.
    cloud-port      The port which local port to map to. note it must be an even number.

OPTIONS:
    -h              Show this help message and exit.
    -i              Assign ssh private key path.
    -p              Assign ssh port.
    -n              Assign name of the service.
    -r              Unmap local port.
```

参数说明：

* ssh-user - 云端sshd用户
* cloud-ip - 云端主机IP或域名
* local-ip - 可选，本地主机IP，不填，则为localhost
* local-port - 待映射的本地端口
* cloud-port - 需要映射至云端的端口，必须偶数

选项：

* -i 指定ssh私钥路径
* -p 指定云端sshd端口，默认22
* -n 指定本地服务名字，默认r-端口

可以通过`map2cloud -h`查看详细的帮助

### 例子

* 映射本机ssh服务端口22至云端22222

```bash
map2cloud -i #私钥路径# -p 22 -n rssh root@your-cloud-ip 22 22222
根据返回文件
systemctl start/stop/restart/status rssh
或者
/etc/init.d/rssh start
```
PS：这样我执行要执行`ssh -p 22222 user@your-cloud-ip`，就可以访问本地主机


* 映射192.168.12.100端口80至云端10080

```bash
map2cloud -n rhttp root@your-cloud-ip 192.168.12.100:80 10080
/etc/init.d/rhttp start
```
PS: 只需在浏览器下输入http://your-cloud-ip:10080，就可以访问本地服务

* 取消映射

```bash
map2cloud -r rhttp
```
