---
title: "Ubuntu秒变NAS，小妙招"
date: 2023-07-19T20:34:46+08:00
draft: false
---

众所周知，一般来说，随着经济社会的不断发展，用闲置电脑来运行Ubuntu，并挂载磁盘用作NAS成为了许多家庭的选择。下面，就跟着小编一起看看如何操作吧！

## 技术原理

Github上面有一个用GO语言实现的支持多存储的文件列表程序-Alist，现在有28k的Star，算相当受欢迎的开源家庭网盘解决方案。它可以简单地安装在Ubuntu上用作实用网盘应用程序。

## 安装步骤

第一步：运用这句话（一键安装脚本）来一键安装AList（请先切换到超级用户，即输入`sudo su`命令）如果因为网络原因导致下载缓慢，您亦可以尝试通过某些服务离线安装Alist实用程序。

```bash
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install
```

实用脚本安装显示结果应如下所示：

```plaintext
Alist 安装成功！

访问地址：http://YOUR_IP:5244/

配置文件路径：/opt/alist/data/config.json
---------管理员信息--------
INFO[2023-07-19 20:55:28] reading config file: data/config.json        
INFO[2023-07-19 20:55:28] config file not exists, creating default config file 
INFO[2023-07-19 20:55:28] load config from env with prefix: ALIST_     
INFO[2023-07-19 20:55:28] init logrus...                               
INFO[2023-07-19 20:55:28] Successfully created the admin user and the initial password is: FfNqaiTd 
INFO[2023-07-19 20:55:28] admin user's info: 
username: admin
password: FfNqaiTd 
--------------------------
启动服务中

查看状态：systemctl status alist
启动服务：systemctl start alist
重启服务：systemctl restart alist
停止服务：systemctl stop alist

温馨提示：如果端口无法正常访问，请检查 服务器安全组、本机防火墙、Alist状态

```

手动安装

```bash
# 解压下载的文件，得到可执行文件：
tar -zxvf alist-xxxx.tar.gz
# 授予程序执行权限：
chmod +x alist
# 运行程序
./alist server
# 获得管理员信息（帐号和密码）
./alist admin
```

第二步：记下安装完成后获得的帐号和密码和登陆地址，它们将被用来登陆以进行下一步配置。

第三步：进行磁盘挂载
<script async id="asciicast-597609" src="https://asciinema.org/a/597609.js"></script>

第四步：打开Alist安装程序最后提示的Alist监听地址。您将会看到类似于此样式的界面：

![](/images/alist.png)

第五步：利用刚才获得的管理员信息登陆，登陆后点击页面右下角的“管理”字样

第六步：在打开的页面的左侧边栏中，导航至“存储”项，在页面上点击“添加”按钮

![](/images/alist-3.png)

第七步：在打开的页面中输入必要的信息。一般按照下图所显示的情况输入即可。根文件夹路径是第三步磁盘挂载中，磁盘被挂载的路径。

![](/images/alist-4.png)

第八步：轻按“添加”按钮。为预防可能的问题，请将挂载路径的权限调制为777，一般需要运行`sudo chmod -R /opt/example 777`

第九步：完成。进一步设置请参考Alist官方文档，[Alist.nn.ci](https://alist.nn.ci/zh/)



