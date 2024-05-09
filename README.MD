# clash-for-pfSense
pfSense上运行的命令行代理工具，在pfSense plus 24.03上测试成功。

本项目也可以在OPNsense上运行。

# 项目来源
源代码来自：
https://github.com/junyu33/clash-for-freebsd

clash程序来自：
https://github.com/MetaCubeX/mihomo/releases

# 使用须知

1、 运行脚本建议使用root用户，或者使用sudo提权。

2、 项目基于 [clash（mihomo）](https://github.com/MetaCubeX/mihomo/releases) 、[yacd](https://github.com/haishanh/yacd) 进行整合。

3、 脚本不提供任何订阅信息，请自行准备Clash订阅地址。

4、 运行脚本前请手动更改`.env`文件中的`CLASH_URL`变量值，否则无法正常运行。

5、 当前只支持x86_64平台。
<br>

# 使用教程

## 下载项目

以root用户登录pfSense的shell环境。

1、安装依赖
```bash
pkg install sudo
pkg install git
pkg install bash
pkg add https://pkg.freebsd.org/FreeBSD:13:amd64/latest/All/zsh-5.9_4.pkg
pkg add https://pkg.freebsd.org/FreeBSD:14:amd64/latest/All/zsh-5.9_4.pkg
pkg add https://pkg.freebsd.org/FreeBSD:15:amd64/latest/All/zsh-5.9_4.pkg
```
> **注意：** `zsh应安装对应防火墙FreeBSD平台系统的版本。
2、下载本项目
```bash
git clone https://github.com/fxn2020/clash-for-pfSense.git
```
3、重命名文件夹
```bash
mv /root/clash-for-pfSense /root/clash
```
4、修改配置文件
```bash
vi /clash/conf/config.yaml
```
config.yaml`可以使用从支持Clash Meta核心的客户端程序（如：ClashN、Clash Verge等）导出的配置文件。

allow-lan参数必须设置为true，以允许防火墙的客户端使用代理。

为了使用Clash面板，需要设置面板访问地址、安全密钥和路径。

DNS监听端口修改为其他端口，避免与防火墙DNS解析器或转发器使用的默认端口53冲突。

可以参考下面的示例：
```bash
port: 7890
allow-lan: true
mode: rule
log-level: info
unified-delay: true
global-client-fingerprint: chrome
external-controller: '0.0.0.0:9090'
secret: 123456
external-ui: /root/clash/dashboard/public
dns:
  enable: true
  listen: :5353
  ipv6: true
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  default-nameserver: 
    - 223.5.5.5
    - 8.8.8.8
  nameserver:
    - https://dns.alidns.com/dns-query
    - https://doh.pub/dns-query
  fallback:
    - https://1.0.0.1/dns-query
    - tls://dns.google
  fallback-filter:
    geoip: true
    geoip-code: CN
    ipcidr:
      - 240.0.0.0/4
proxies:
......
```
4、修改变量
```bash
vi /clash/.env
```
> **注意：** `.env`文件中的变量`CLASH_SECRET`值为空时，脚本将自动生成随机字符串。

## 启动程序

1、启动脚本
```bash
sudo bash start.sh
```
将会出现以下运行信息：

```bash
添加运行参数...
clash_enable: YES -> YES

启动Clash代理服务...
Starting clash.
代理启动成功！   OK  

Clash仪表盘访问地址：
http://<LAN ip>:9090/ui 
访问密钥: 123456 

命令说明：
开启代理: sh /usr/local/etc/rc.d/clash start 
关闭代理: sh /usr/local/etc/rc.d/clash stop 
重启代理: sh /usr/local/etc/rc.d/clash restart 
查看状态: sh /usr/local/etc/rc.d/clash status 
```
2、检查端口
```bash
netstat -f inet -na | grep -i LISTEN
tcp46      0      0 *.7891                 *.*                    LISTEN     
tcp46      0      0 *.7890                 *.*                    LISTEN     
tcp46      0      0 *.9090                 *.*                    LISTEN     
tcp4       0      0 127.0.0.1.953          *.*                    LISTEN     
tcp4       0      0 *.53                   *.*                    LISTEN     
tcp4       0      0 127.0.0.1.3129         *.*                    LISTEN     
tcp4       0      0 127.0.0.1.3128         *.*                    LISTEN     
tcp4       0      0 192.168.101.4.3128     *.*                    LISTEN 
```
如果7890和9090端口被占用，说明clash程序启动成功，现在客户端就可以通过代理上网了。

3、透明代理

pfSense透明代理设置，请参阅鐵血男兒的[博客](https://pfchina.org/?p=10626)。
<br>

## 控制面板

1、 通过地址：`http://LAN IP:9090/ui`访问Clash控制面板。

2、 在`API Base URL`一栏中输入：http://\<ip\>:9090 ，在`Secret(optional)`一栏中输入访问安全密钥。

3、 点击Add，并单击刚刚输入的管理地址，之后便可以通过Web管理界面对clash进行一些配置。

# 常见问题

1、 必须使用 `bash xxx.sh` 运行脚本。

2、 部分用户在UI界面找不到代理节点，是因为机场提供的clash配置文件是经过base64编码的，且配置文件格式不符合clash配置标准所致。

3、 项目集成的订阅转换功能如果无法使用，可以通过自建或者第三方平台对订阅地址进行转换。

# 免费订阅
```bash
https://neko-warp.nloli.xyz/neko_warp.yaml
https://subs.zeabur.app/clash
```