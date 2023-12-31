---
title: '更加GPT的“一言”接口：HitokotoGPT'
categories: Uncategorized
date: 2023-05-20 13:04:00
---
HitokotoGPT 是我一时兴起做的一个简单小脚本，用来提供一个基于 ChatGPT 的 “一言” 接口。

这个原版所谓的 “一言” 接口是一个项目，它在每一次请求 API 时都会返回一句话，用来装点网站的文艺风范。而 HitokotoGPT 则更进一步，它使用 ChatGPT 来动态生成这句话，使得网站上的文学装点也显得更加新颖。

HitokotoGPT 建立在 ChatGPT 上，是一个简单部署的 Python 程序。通过这一程序，你将可以简单地获取由 ChatGPT 生成的 “名言警句”。你可以自己部署这一程序，也可以使用公开的接口。本项目自带的公开接口仅供测试用，不对其稳定性负责。

## 部署指南

### 满足系统需求

本项目非常轻量，一般能够运行 python 的机器都能顺利运行本程序。但您需要确定运行本程序的机器可以访问 OpenAI 的 API 接口。

本项目依赖 uvicorn 和 fastapi 等库运行。所以，请在 CLI 中执行这一条命令，自动化地安装所需的全部依赖

```bash
pip install -r requirements.txt
```

这个安装过程将很快结束。完成后请检查防火墙的端口状态，是否开放`65530`端口。本程序在`65530`端口监听。如果你认为高位端口不方便使用，请在`config.json`中将`port`变量修改为你期望开放的端口号。

### 进行必要设置

本项目的全部配置都存储在`config.json`文件中，请打开默认目录下的 config.json 文件，遵照遵照下面的提示进行配置。

```json
{
    "api_key" : "your openai api key", //OpenAI API鉴权密钥
    "access_token" : "access token to access the web server", //用来给HitokotoGPT接口鉴权的密钥
    "port" : 65530, //API 服务监听的端口，如无必要请勿修改
    "database" : "testdebug", //SQLite数据库的名称。如无必要请勿修改
    "rate" : "1/min", //对于每个来源IP的请求速率限制
    "rate_cached" : "3/min" //对于非即时生成的内容的请求速率限制
}
```

## 调用接口

***重要：本项目可以设置 Rate Limit，但请您务必注意使本项目的 Rate Limit 和 OpenAI 的频率限制相吻合，否则，将会造成不可预料的后果。***

请求体请统一通过 JSON 发送，接口只处理内容类型为 JSON 的 POST 请求，其他请求一概报错。

### 鉴权

本项目使用简单鉴权。请在访问所有接口时都使用`gptauth`参数，其值为在 json 中设置的`api_key`

### 即时生成的一言

在这种模式下，本程序将请求 OpenAI 的接口，通过默认的提示词生成一言并返回。需要注意：此请求可因 OpenAI 的速率问题而严重拖延时长，甚至超时而失败。请不要将即时生成的一言用于页面醒目位置，因为它可能会在很长一段时间内加载不出来。

接口地址：`/hitokoto` (POST)

请求参数：

* `gptauth` 字符串，用于鉴权的密钥（必须）
* `encycle` 布尔值，设置此次生成产生的一言是否允许供给不重复缓存生成使用，即如果调用 `/hitokoto-cached` 接口并选择 “不重样” 模式时，是否使用此生成。（可选，默认为否，即`false`）

### 预生成 / 已缓存的一言

在调用 “即时生成的一言” 端口时，所有生成都会被记录进入一个 SQLite 数据库中，这是为了供这个接口调用已经预提交过的生成。同时，你还可以使用`./batch.py`在低峰时段大量生成一言，并存储进数据库中。你可以选择使用 “重样” 或 “不重样” 模式，但需要注意，尽管在这个接口选择了 “不重样” 模式，你还是有可能看到相同的一言，这是因为在 “即时生成的一言” 接口中它是否被使用过的状态设置为 “没有”。当然你不会再看到它了，因为现在它的使用状态已经被设置为 “已使用”

但如果 “不重样” 模式已经找不到从未被使用过的一言，那接口将直接返回已经使用过的一言，且不会有任何提示，请务必注意。

接口地址 `/hitokoto-cached` (POST)

请求参数：

* `gptauth` 已介绍
* `recycle` 布尔值，设置为真时使用 “重样”，设置为否时使用 “不重样”（可选，默认为否，即`False`）

## 低峰批量生成

通过`batch.py`，你可以大量生成一言，并将它们存进数据库中供日后调用使用。

具体操作是在`batch.py`中设置`sentence_num`变量的数值，以便决定生成的数量。同时，还可以使用其他程序来让`batch.py`在夜深人静的低峰时段执行，或是隔一两个小时定时执行，以提高成功率并保证总能有一言调用。

