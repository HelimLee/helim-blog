---
title: '部署隐私友好的开源网站统计——Umami'
categories: Diaries
date: 2023-04-02 16:04:18
---
Umami官方给的自托管部署模式实在是坑太多了。简直是一种折磨。本文是对官方文档的补充。

官方文档：https://umami.is/docs/install

## 环境准备

一台干净安装的Ubuntu或类似发行版（本次安装只在Ubuntu上测试）：“干净的”并非一定代表全新安装。一些文明的、不会把你的系统搞得一团糟的程序是可以安装的。但如果你的系统上已经存在了2个及以上的Web Server或数据库，那么你的确应该考虑调整系统策略。

### 安装Postgresql

这里就来了第一个坑：虽然官方文档里写着“mysql**或**postgresql”，但经过我的实际操作，发现其实**只有**postgresql才能完成最后的配置。所以不要尝试安装mysql，而是直接安装postgresql.且安装postgresql要更加简单。可以直接从ubuntu默认仓库中二进制安装。

```bash
apt install postgresql
```

然后开始创建数据库

```pgsql
sudo -u postgres psql
--通过\password为超级用户创建密码
\password postgres
--创建数据库，请将dbname替换为数据库名称
CREATE DATABASE dbname;
```

接下来是第二个坑。安装完postgresql，并创建数据库后，你可能会看到一些教程指导要求创造一个新的postgresql帐号。这种形式可能会很安全，但并不十分推荐。所以我选择直接允许postgresql的超级用户postgres通过md5密码认证链接。要实现这一点，请导航到postgresql的链路配置文件，通常在`/etc/postgresql/<version>/main/pg_hba.conf`，其中\<version\>是版本号。找到这一行：

```conf
local	all	postgres	peer
#修改为
local	all	postgres	md5
```

现在，如果通过本地链路提起的超级用户登陆请求将通过密码（md5）方式验证。由于来源被限定为本地用户，所以可以称为是安全的。

### 安装Node.js并更新

这块是第三个大坑。很多人，包括我，在安装之前因为没有apt update或者别的原因，安装了旧版的node.js，导致接下来的yarn安装出现了很多问题。所以要防患于未然，请遵循这些步骤：

```bash
#更新apt源
apt update
#安装npm，顺道会自动安装nodejs
apt install npm
#开始更新nodejs
npm install -g n
n lts #使用n latest更新到最新版本而非lts版本
```

完成这些步骤后，请安装yarn

```bash
npm install -g yarn
```

## 安装Umami

安装umami是一个简单的过程。但不当的配置可能使其变得非常复杂。请确保接下来执行的所有命令都在*干净的*目录中。

```bash
#安装umami并使用yarn安装
git clone https://github.com/umami-software/umami.git
cd umami
yarn install
```

接下来需要对umami的环境进行一些配置。使用`vim .env`打开环境变量文件，开始配置

```env
DATABASE_URL=(connection url)
```

这里的connection url代表的是postgresql的连接地址。它应该是：`postgresql://postgresql:你设置的密码@127.0.0.1:5432/数据库名称`

在完成这些步骤后可以进行最后一部的安装：构建和运行。

## 构建、运行和后续配置

使用`yarn build`完成最终配置（依CPU性能而决定耗时，一般5分钟）。并使用`yarn start`开始运行。请访问`http://服务器IP:3000`查看页面。

为了提升安全性，可以使用caddy或其他web server进行反代。并且，可以更改默认的密码。默认帐号`admin`，密码`umami`.
