---
title: '给Cloudreve配上在线查看文档的玩意'
categories: Uncategorized
date: 2023-03-05 14:55:20
---
Cloudreve可以说是一款质量上乘的开源软件开源网盘解决方案了。拿它给我的[NAS(仅限HelimLAN局域网)](https://nas.helim.net/)做网盘服务用，那是相当方便。

只是我的这个网盘服务是只给局域网用的，所以自带的那个Office APP在线文档预览是用不得的。况且Office在线版的加载速度，大家也都心知肚明。于是我就想能不能用比如Onlyoffice来完成个纯前端的文档编辑和预览。

## 初步配置

当然，首先是把我的Cloudreve基础服务搭配好。在以前，我一直都是用IP地址直接访问。不过我去Cloudflare那边改了几条记录，发现这个DNS竟然可以解析一个内网地址。那我乐坏了，把nas.helim.net解析到了我这边的内网地址。这样我的那些设备就不用想方设法地改hosts或者输一大长串IP地址了。

为了去掉端口号顺便加个HTTPS，我去下载了个Caddyserver（GO语言写的，所以效率高一些罢），然后配置了一些反代。

```
{
	auto_https off #关闭https
}

nas.helim.net {
	tls path/to/cert path/to/key
	reverse_proxy ${cloudreve_port}
}
```

完成这些配置之后，在局域网中访问nas.helim.net就能直接打开Cloudreve网盘了。不能说比较简陋，只能说过于华美，跟那些一般的网盘已经几乎没有二样了。

## 安装Onlyoffice

我看了一下，那个应用广泛的LibreOffice,很可惜，已经停止运行了。但是Onlyoffice还是有社区维护的。我使用这个命令安装了docker

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

安装完docker之后安装onlyoffice，都是简单直截的命令

```bash
sudo docker run -i -t -d -p 5780:80 --restart=always -e JWT_SECRET=my_jwt_secret onlyoffice/documentserver
```

这里5780:80代表把80端口映射到宿主机的5780端口。不到80端口是有原因的，是因为待会要配置caddy反代。我觉得配置反代比直接配置tls要方便多了，当然代价是网络拓扑会复杂一丁点。

然后要进到docker里面修改WOPI设置，由于docker里面没有vi或者vim，所以需要使用一个比较陌生但其实非常好用的

```bash
cd /etc/onlyoffice/documentserver #打开目录
nano local.json #开始编辑
```

修改local.json到如下

```json
{
  "services": {
    "CoAuthoring": {
      "sql": {
        "type": "postgres",
        "dbHost": "localhost",
        "dbPort": "5432",
        "dbName": "onlyoffice",
        "dbUser": "onlyoffice",
        "dbPass": "onlyoffice"
      },
      "token": {
        "enable": {
          "request": {
            "inbox": false,
            "outbox": false
          },
          "browser": false
        },
        "inbox": {
          "header": "Authorization",
          "inBody": false
        },
        "outbox": {
          "header": "Authorization",
          "inBody": false
        }
      },
      "secret": {
        "inbox": {
          "string": "YN2KSbqLwYrB0T038eOL"
        },
        "outbox": {
          "string": "YN2KSbqLwYrB0T038eOL"
        },
        "session": {
          "string": "YN2KSbqLwYrB0T038eOL"
        }
      }
    }
  },
  "rabbitmq": {
    "url": "amqp://guest:guest@localhost"
  },
  "storage": {
    "fs": {
      "secretString": "8jx9nirmmx8UUkQlzURJ"
    }
  },
  "wopi": {
        "enable": true
  }
}
```

接下来还要进行设置，为了启用跨域访问，需要修改default.json

```json
{
        "statsd": {
                "useMetrics": false,
                "host": "localhost",
                "port": "8125",
                "prefix": "ds."
        },
        "log": {
                "filePath": "",
                "options": {
                        "replaceConsole": true
                }
        },
        "queue": {
                "type": "rabbitmq",
                "visibilityTimeout": 300,
                "retentionPeriod": 900
        },
        "storage": {
                "name": "storage-fs",
                "fs": {
                        "folderPath": "",
                        "urlExpires": 900,
                        "secretString": "verysecretstring"
                },
                "region": "",
                "endpoint": "http://localhost/s3",
                "bucketName": "cache",
                "storageFolderName": "files",
                "cacheFolderName": "data",
                "urlExpires": 604800,
                "accessKeyId": "AKID",
                "secretAccessKey": "SECRET",
                "sslEnabled": false,
                "s3ForcePathStyle": true,
                "externalHost": ""
        },
        "rabbitmq": {
                "url": "amqp://guest:guest@localhost:5672",
                "socketOptions": {},
                "exchangepubsub": "ds.pubsub",
                "queueconverttask": "ds.converttask",
                "queueconvertresponse": "ds.convertresponse",
                "exchangeconvertdead": "ds.exchangeconvertdead",
                "queueconvertdead": "ds.convertdead",
                "queuedelayed": "ds.delayed"
        },
        "activemq": {
                "connectOptions": {
                        "port": 5672,
                        "host": "localhost",
                        "name": "admin",
                        "reconnect": false
                },
                "queueconverttask": "ds.converttask",
                "queueconvertresponse": "ds.convertresponse",
                "queueconvertdead": "ActiveMQ.DLQ",
                "queuedelayed": "ds.delayed",
                "topicpubsub": "ds.pubsub"
        },
        "dnscache": {
                "enable" : true,
                "ttl" : 300,
                "cachesize" : 1000
        },
        "openpgpjs": {
                "config": {
                },
                "encrypt": {
                        "passwords": ["verysecretstring"]
                },
                "decrypt": {
                        "passwords": ["verysecretstring"]
                }
        },
        "bottleneck": {
                "getChanges": {
                }
        },
        "wopi": {
                "enable": false,
                "host" : "",
                "htmlTemplate" : "../../web-apps/apps/api/wopi",
                "wopiZone" : "external-http",
                "favIconUrlWord" : "/web-apps/apps/documenteditor/main/resources/img/favicon.ico",
                "favIconUrlCell" : "/web-apps/apps/spreadsheeteditor/main/resources/img/favicon.ico",
                "favIconUrlSlide" : "/web-apps/apps/presentationeditor/main/resources/img/favicon.ico",
                "fileInfoBlockList" : ["FileUrl"],
                "pdfView": ["pdf", "djvu", "xps", "oxps"],
                "wordView": ["doc", "dotx", "dotm", "dot", "fodt", "ott", "rtf", "mht", "html", "htm", "xml", "epub", "fb2"],
                "wordEdit": ["docx", "docm", "docxf", "oform", "odt", "txt"],
                "cellView": ["xls", "xlsb", "xltx", "xltm", "xlt", "fods", "ots"],
                "cellEdit": ["xlsx", "xlsm", "ods", "csv"],
                "slideView": ["ppt", "ppsx", "ppsm", "pps", "potx", "potm", "pot", "fodp", "otp"],
                "slideEdit": ["pptx", "pptm", "odp"],
                "publicKey": "BgIAAACkAABSU0ExAAgAAAEAAQD/NVqekFNi8X3p6Bvdlaxm0GGuggW5kKfVEQzPGuOkGVrz6DrOMNR+k7Pq8tONY+1NHgS6Z+v3959em78qclVDuQX77Tkml0xMHAQHN4sAHF9iQJS8gOBUKSVKaHD7Z8YXch6F212YSUSc8QphpDSHWVShU7rcUeLQsd/0pkflh5+um4YKEZhm4Mou3vstp5p12NeffyK1WFZF7q4jB7jclAslYKQsP82YY3DcRwu5Tl/+W0ifVcXze0mI7v1reJ12pKn8ifRiq+0q5oJST3TRSrvmjLg9Gt3ozhVIt2HUi3La7Qh40YOAUXm0g/hUq2BepeOp1C7WSvaOFHXe6Hqq",
                "modulus": "qnro3nUUjvZK1i7UqeOlXmCrVPiDtHlRgIPReAjt2nKL1GG3SBXO6N0aPbiM5rtK0XRPUoLmKu2rYvSJ/Kmkdp14a/3uiEl788VVn0hb/l9OuQtH3HBjmM0/LKRgJQuU3LgHI67uRVZYtSJ/n9fYdZqnLfveLsrgZpgRCoabrp+H5Uem9N+x0OJR3LpToVRZhzSkYQrxnERJmF3bhR5yF8Zn+3BoSiUpVOCAvJRAYl8cAIs3BwQcTEyXJjnt+wW5Q1VyKr+bXp/39+tnugQeTe1jjdPy6rOTftQwzjro81oZpOMazwwR1aeQuQWCrmHQZqyV3Rvo6X3xYlOQnlo1/w==",
                "exponent": "AQAB",
                "privateKey": "MIIEowIBAAKCAQEAqnro3nUUjvZK1i7UqeOlXmCrVPiDtHlRgIPReAjt2nKL1GG3SBXO6N0aPbiM5rtK0XRPUoLmKu2rYvSJ/Kmkdp14a/3uiEl788VVn0hb/l9OuQtH3HBjmM0/LKRgJQuU3LgHI67uRVZYtSJ/n9fYdZqnLfveLsrgZpgRCoabrp+H5Uem9N+x0OJR3LpToVRZhzSkYQrxnERJmF3bhR5yF8Zn+3BoSiUpVOCAvJRAYl8cAIs3BwQcTEyXJjnt+wW5Q1VyKr+bXp/39+tnugQeTe1jjdPy6rOTftQwzjro81oZpOMazwwR1aeQuQWCrmHQZqyV3Rvo6X3xYlOQnlo1/wIDAQABAoIBAQCKtUSBs8tNYrGTQTlBHXrwpkDg+u7WSZt5sEcfnkxA39BLtlHU8gGO0E9Ihr8GAL+oWjUsEltJ9GTtN8CJ9lFdPVS8sTiCZR/YQOggmFRZTJyVzMrkXgF7Uwwiu3+KxLiTOZx9eRhfDBlTD8W9fXaegX2i2Xp2ohUhBHthEBLdaZTWFi5Sid/Y0dDzBeP6UIJorZ5D+1ybaeIVHjndpwNsIEUGUxPFLrkeiU8Rm4MJ9ahxfywcP7DjQoPGY9Ge5cBhpxfzERWf732wUD6o3+L9tvOBU00CLVjULbGZKTVE2FJMyXK9jr6Zor9Mkhomp6/8Agkr9rp+TPyelFGYEz8hAoGBAOEc09CrL3eYBkhNEcaMQzxBLvOGpg8kaDX5SaArHfl9+U9yzRqss4ARECanp9HuHfjMQo7iejao0ngDjL7BNMSaH74QlSsPOY2iOm8Qvx8/zb7g4h9r1zLjFZb3mpSA4snRZvvdiZ9ugbuVPmhXnDzRRMg45MibJeeOTJNylofRAoGBAMHfF/WutqKDoX25qZo9m74W4bttOj6oIDk1N4/c6M1Z1v/aptYSE06bkWngj9P46kqjaay4hgMtzyGruc5aojPx5MHHf5bo14+Jv4NzYtR2llrUxO+UJX7aCfUYXI7RC93GUmhpeQ414j7SNAXec58d7e+ETw+6cHiAWO4uOSTPAoGATPq5qDLR4Zi4FUNdn8LZPyKfNqHF6YmupT5hIgd8kZO1jKiaYNPL8jBjkIRmjBBcaXcYD5p85nImvumf2J9jNxPpZOpwyC/Fo5xlVROp97qu1eY7DTmodntXJ6/2SXAlnZQhHmHsrPtyG752f+HtyJJbbgiem8cKWDu+DfHybfECgYBbSLo1WiBwgN4nHqZ3E48jgA6le5azLeKOTTpuKKwNFMIhEkj//t7MYn+jhLL0Mf3PSwZU50Vidc1To1IHkbFSGBGIFHFFEzl8QnXEZS4hr/y3o/teezj0c6HAn8nlDRUzRVBEDXWMdV6kCcGpCccTIrqHzpqTY0vV0UkOTQFnDQKBgAxSEhm/gtCYJIMCBe+KBJT9uECV5xDQopTTjsGOkd4306EN2dyPOIlAfwM6K/0qWisa0Ei5i8TbRRuBeTTdLEYLqXCJ7fj5tdD1begBdSVtHQ2WHqzPJSuImTkFi9NXxd1XUyZFM3y6YQvlssSuL7QSxUIEtZHnrJTt3QDd10dj",
                "publicKeyOld": "BgIAAACkAABSU0ExAAgAAAEAAQD/NVqekFNi8X3p6Bvdlaxm0GGuggW5kKfVEQzPGuOkGVrz6DrOMNR+k7Pq8tONY+1NHgS6Z+v3959em78qclVDuQX77Tkml0xMHAQHN4sAHF9iQJS8gOBUKSVKaHD7Z8YXch6F212YSUSc8QphpDSHWVShU7rcUeLQsd/0pkflh5+um4YKEZhm4Mou3vstp5p12NeffyK1WFZF7q4jB7jclAslYKQsP82YY3DcRwu5Tl/+W0ifVcXze0mI7v1reJ12pKn8ifRiq+0q5oJST3TRSrvmjLg9Gt3ozhVIt2HUi3La7Qh40YOAUXm0g/hUq2BepeOp1C7WSvaOFHXe6Hqq",
                "modulusOld": "qnro3nUUjvZK1i7UqeOlXmCrVPiDtHlRgIPReAjt2nKL1GG3SBXO6N0aPbiM5rtK0XRPUoLmKu2rYvSJ/Kmkdp14a/3uiEl788VVn0hb/l9OuQtH3HBjmM0/LKRgJQuU3LgHI67uRVZYtSJ/n9fYdZqnLfveLsrgZpgRCoabrp+H5Uem9N+x0OJR3LpToVRZhzSkYQrxnERJmF3bhR5yF8Zn+3BoSiUpVOCAvJRAYl8cAIs3BwQcTEyXJjnt+wW5Q1VyKr+bXp/39+tnugQeTe1jjdPy6rOTftQwzjro81oZpOMazwwR1aeQuQWCrmHQZqyV3Rvo6X3xYlOQnlo1/w==",
                "exponentOld": "AQAB",
                "privateKeyOld": "MIIEowIBAAKCAQEAqnro3nUUjvZK1i7UqeOlXmCrVPiDtHlRgIPReAjt2nKL1GG3SBXO6N0aPbiM5rtK0XRPUoLmKu2rYvSJ/Kmkdp14a/3uiEl788VVn0hb/l9OuQtH3HBjmM0/LKRgJQuU3LgHI67uRVZYtSJ/n9fYdZqnLfveLsrgZpgRCoabrp+H5Uem9N+x0OJR3LpToVRZhzSkYQrxnERJmF3bhR5yF8Zn+3BoSiUpVOCAvJRAYl8cAIs3BwQcTEyXJjnt+wW5Q1VyKr+bXp/39+tnugQeTe1jjdPy6rOTftQwzjro81oZpOMazwwR1aeQuQWCrmHQZqyV3Rvo6X3xYlOQnlo1/wIDAQABAoIBAQCKtUSBs8tNYrGTQTlBHXrwpkDg+u7WSZt5sEcfnkxA39BLtlHU8gGO0E9Ihr8GAL+oWjUsEltJ9GTtN8CJ9lFdPVS8sTiCZR/YQOggmFRZTJyVzMrkXgF7Uwwiu3+KxLiTOZx9eRhfDBlTD8W9fXaegX2i2Xp2ohUhBHthEBLdaZTWFi5Sid/Y0dDzBeP6UIJorZ5D+1ybaeIVHjndpwNsIEUGUxPFLrkeiU8Rm4MJ9ahxfywcP7DjQoPGY9Ge5cBhpxfzERWf732wUD6o3+L9tvOBU00CLVjULbGZKTVE2FJMyXK9jr6Zor9Mkhomp6/8Agkr9rp+TPyelFGYEz8hAoGBAOEc09CrL3eYBkhNEcaMQzxBLvOGpg8kaDX5SaArHfl9+U9yzRqss4ARECanp9HuHfjMQo7iejao0ngDjL7BNMSaH74QlSsPOY2iOm8Qvx8/zb7g4h9r1zLjFZb3mpSA4snRZvvdiZ9ugbuVPmhXnDzRRMg45MibJeeOTJNylofRAoGBAMHfF/WutqKDoX25qZo9m74W4bttOj6oIDk1N4/c6M1Z1v/aptYSE06bkWngj9P46kqjaay4hgMtzyGruc5aojPx5MHHf5bo14+Jv4NzYtR2llrUxO+UJX7aCfUYXI7RC93GUmhpeQ414j7SNAXec58d7e+ETw+6cHiAWO4uOSTPAoGATPq5qDLR4Zi4FUNdn8LZPyKfNqHF6YmupT5hIgd8kZO1jKiaYNPL8jBjkIRmjBBcaXcYD5p85nImvumf2J9jNxPpZOpwyC/Fo5xlVROp97qu1eY7DTmodntXJ6/2SXAlnZQhHmHsrPtyG752f+HtyJJbbgiem8cKWDu+DfHybfECgYBbSLo1WiBwgN4nHqZ3E48jgA6le5azLeKOTTpuKKwNFMIhEkj//t7MYn+jhLL0Mf3PSwZU50Vidc1To1IHkbFSGBGIFHFFEzl8QnXEZS4hr/y3o/teezj0c6HAn8nlDRUzRVBEDXWMdV6kCcGpCccTIrqHzpqTY0vV0UkOTQFnDQKBgAxSEhm/gtCYJIMCBe+KBJT9uECV5xDQopTTjsGOkd4306EN2dyPOIlAfwM6K/0qWisa0Ei5i8TbRRuBeTTdLEYLqXCJ7fj5tdD1begBdSVtHQ2WHqzPJSuImTkFi9NXxd1XUyZFM3y6YQvlssSuL7QSxUIEtZHnrJTt3QDd10dj",
                "refreshLockInterval": "10m"
        },
        "tenants": {
                "baseDir": "",
                "baseDomain": "",
                "filenameSecret": "secret.key",
                "filenameLicense": "license.lic",
                "defaultTenant": "localhost",
                "cache" : {
                        "stdTTL": 300,
                        "checkperiod": 60,
                        "useClones": false
                }
        },
        "services": {
                "CoAuthoring": {
                        "server": {
                                "port": 8000,
                                "workerpercpu": 1,
                                "mode": "development",
                                "limits_tempfile_upload": 104857600,
                                "limits_image_size": 26214400,
                                "limits_image_download_timeout": {
                                        "connectionAndInactivity": "2m",
                                        "wholeCycle": "2m"
                                },
                                "callbackRequestTimeout": {
                                        "connectionAndInactivity": "10m",
                                        "wholeCycle": "10m"
                                },
                                "healthcheckfilepath": "../public/healthcheck.docx",
                                "savetimeoutdelay": 5000,
                                "edit_singleton": false,
                                "forgottenfiles": "forgotten",
                                "forgottenfilesname": "output",
                                "maxRequestChanges": 20000,
                                "openProtectedFile": true,
                                "editorDataStorage": "editorDataMemory",
                                "assemblyFormatAsOrigin": true,
                                "newFileTemplate" : "../../document-templates/new",
                                "downloadFileAllowExt": ["pdf", "xlsx"],
                                "tokenRequiredParams": true
                        },
                        "requestDefaults": {
                                "headers": {
                                        "User-Agent": "Node.js/6.13",
                                        "Connection": "Keep-Alive"
                                },
                                "gzip": true,
                                "rejectUnauthorized": false
                        },
                        "autoAssembly": {
                                "enable": false,
                                "interval": "5m",
                                "step": "1m"
                        },
                        "utils": {
                                "utils_common_fontdir": "null",
                                "utils_fonts_search_patterns": "*.ttf;*.ttc;*.otf",
                                "limits_image_types_upload": "jpg;jpeg;jpe;png;gif;bmp"
                        },
                        "sql": {
                                "type": "postgres",
                                "tableChanges": "doc_changes",
                                "tableResult": "task_result",
                                "dbHost": "localhost",
                                "dbPort": 5432,
                                "dbName": "onlyoffice",
                                "dbUser": "onlyoffice",
                                "dbPass": "onlyoffice",
                                "charset": "utf8",
                                "connectionlimit": 10,
                                "max_allowed_packet": 1048575,
                                "pgPoolExtraOptions": {}
                        },
                        "redis": {
                                "name": "redis",
                                "prefix": "ds:",
                                "host": "localhost",
                                "port": 6379,
                                "options": {}
                        },
                        "pubsub": {
                                "maxChanges": 0
                        },
                        "expire": {
                                "saveLock": 60,
                                "presence": 300,
                                "locks": 604800,
                                "changeindex": 86400,
                                "lockDoc": 30,
                                "message": 86400,
                                "lastsave": 604800,
                                "forcesave": 604800,
                                "saved": 3600,
                                "documentsCron": "0 */2 * * * *",
                                "files": 86400,
                                "filesCron": "00 00 */1 * * *",
                                "filesremovedatonce": 100,
                                "sessionidle": "1h",
                                "sessionabsolute": "30d",
                                "sessionclosecommand": "2m",
                                "pemStdTTL": "1h",
                                "pemCheckPeriod": "10m",
                                "updateVersionStatus": "5m",
                                "monthUniqueUsers": "1y"
                        },
                        "ipfilter": {
                                "rules": [{"address": "*", "allowed": true}],
                                "useforrequest": false,
                                "errorcode": 403
                        },
                        "request-filtering-agent" : {
                                "allowPrivateIPAddress": true,
                                "allowMetaIPAddress": true
                        },
                        "secret": {
                                "inbox": {"string": "secret", "file": ""},
                                "outbox": {"string": "secret", "file": ""},
                                "session": {"string": "secret", "file": ""}
                        },
                        "token": {
                                "enable": {
                                        "browser": false,
                                        "request": {
                                                "inbox": false,
                                                "outbox": false
                                        }
                                },
                                "inbox": {
                                        "header": "Authorization",
                                        "prefix": "Bearer ",
                                        "inBody": false
                                },
                                "outbox": {
                                        "header": "Authorization",
                                        "prefix": "Bearer ",
                                        "algorithm": "HS256",
                                        "expires": "5m",
                                        "inBody": false,
                                        "urlExclusionRegex": ""
                                },
                                "session": {
                                        "algorithm": "HS256",
                                        "expires": "30d"
                                },
                                "verifyOptions": {
                                        "clockTolerance": 60
                                }
                        },
                        "plugins": {
                                "uri": "/sdkjs-plugins",
                                "autostart": []
                        },
                        "themes": {
                                "uri": "/web-apps/apps/common/main/resources/themes"
                        },
                        "editor":{
                                "spellcheckerUrl": "",
                                "reconnection":{
                                        "attempts": 50,
                                        "delay": "2s"
                                },
                                "binaryChanges": false,
                                "websocketMaxPayloadSize": "1.5MB",
                                "maxChangesSize": "0mb"
                        },
                        "sockjs": {
                                "sockjs_url": "",
                                "disable_cors": true,
                                "websocket": true
                        },
                        "socketio": {
                                "connection": {
                                        "path": "/doc/",
                                        "serveClient": false,
                                        "pingTimeout": 20000,
                                        "pingInterval": 25000,
                                        "maxHttpBufferSize": 1e8
                                }
                        },
                        "callbackBackoffOptions": {
                                "retries": 0,
                                "timeout":{
                                        "factor": 2,
                                        "minTimeout": 1000,
                                        "maxTimeout": 2147483647,
                                        "randomize": false
                                },
                                "httpStatus": "429,500-599"
                        }
                }
        },
        "license" : {
                "license_file": "",
                "warning_limit_percents": 70,
                "packageType": 0
        },
        "FileConverter": {
                "converter": {
                        "maxDownloadBytes": 104857600,
                        "downloadTimeout": {
                                "connectionAndInactivity": "2m",
                                "wholeCycle": "2m"
                        },
                        "downloadAttemptMaxCount": 3,
                        "downloadAttemptDelay": 1000,
                        "maxprocesscount": 1,
                        "fontDir": "null",
                        "presentationThemesDir": "null",
                        "x2tPath": "null",
                        "docbuilderPath": "null",
                        "args": "",
                        "spawnOptions": {},
                        "errorfiles": "",
                        "streamWriterBufferSize": 8388608,
                        "maxRedeliveredCount": 2,
                        "inputLimits": [
                                {
                                "type": "docx;dotx;docm;dotm",
                                "zip": {
                                        "uncompressed": "50MB",
                                        "template": "*.xml"
                                }
                                },
                                {
                                "type": "xlsx;xltx;xlsm;xltm",
                                "zip": {
                                        "uncompressed": "300MB",
                                        "template": "*.xml"
                                }
                                },
                                {
                                "type": "pptx;ppsx;potx;pptm;ppsm;potm",
                                "zip": {
                                        "uncompressed": "50MB",
                                        "template": "*.xml"
                                }
                                }
                        ]
                }
        }
}
```

然后重启程序，才能实现生效。接下来

```bash
supervisorctl restart all
```

完事就重启成功了。接下来再访问一下网站，看看是否还在服务。

好接下来就进入<s>抄</s>写代码时间。我们要实现一个html用来call我们的在线Office后端。结合了Github上的代码写了这些：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <---
title>OnlyOffice Viewer</---
title>
</head>
 
<body>
    <div id="placeholder"></div>
    <script type="text/javascript" src="https://office.helim.net/web-apps/apps/api/documents/api.js"></script>
    <script>
        function getQueryParamValue(name) {
            const searchParams = new URLSearchParams(window.location.search);
            return searchParams.get(name);
        }
        
        const url = decodeURIComponent(getQueryParamValue("src"));
            const filePath = decodeURIComponent(getQueryParamValue("fpath"));
        const fileName = filePath.substring(url.lastIndexOf('/') + 1, url.lastIndexOf('?') != -1 ? url.lastIndexOf('?') : url.length);
        const fileExtension = filePath.split('.').pop();
           console.log(fileExtension);
        const docEditor = new DocsAPI.DocEditor("placeholder", {
            "document": {
                "fileType": fileExtension,
                "permissions": {
                    "edit": false,
                    "comment": true,
                    "download": true,
                    "print": true,
                    "fillForms": true,
                },
                "---
title": fileName,
                "url": url,
            },
            "editorConfig": {
                "lang": "zh-CN",
                "mode": "view",
            },
            "height": "1080px",
            "type": "desktop",
        });
    </script>
</body>
</html>
```

虽然很简陋，但真的能工作。在Cloudreve后台“Office 文档预览服务”这个选项中填上：`https://server/path/to/html.html?src={$src}&fpath={$name}`

接下来，打开任何一个office文件都能很好的显示了。完工！

