---
title: "SLP协议的GO语言实现推荐，附HTTP接口一则"
date: 2023-06-22T12:17:02+08:00
draft: false
---

Minecraft是世界上最受欢迎的游戏之一。它的联机功能（服务器）也应用极为广泛。对于喜欢联机游戏的用户而言，可以显示服务器状态和简介的“服务器列表”功能十分实用。而用户向服务器列表中添加了服务器的IP和端口，Minecraft获取有关此服务器元数据的详细过程，则很有探讨的价值。通过一系列调查，爱好者们已经明确，Minecraft客户端会使用自研的一套应用层协议，与远端的机器进行元数据交换。这套协议就是SLP协议，SLP代表`Server List Ping` （服务器列表Ping）

在[wiki.vg](https://wiki.vg/Server_List_Ping)上，SLP的具体原理已经被详细记录。以1.7+版本的SLP为例，端侧会首先向云侧发起Handshake（握手）请求，即发送HandShake包。这个第一个包使用的是为大家所熟悉的常规Client-Server协议，内容大致如下：

| Packet ID | Field Name | Field Type |
| --- | --- | --- |
| 0x00 | Protocol Version | Int |
| | Server Address | String |
| | Server Port | Unsigned Short |
| | Next state | Int |

这些信息需要发送给服务端。此外还要处理接下来的一系列回包。总之，如果我们要在编程语言中处理这些信息，使用“纯手动”的模式的话，还是有些困难的。

所幸，在GO语言中的一些包可以帮助我们方便地发送一系列数据包，用一种相对“自动档”的模式。其中我推荐的是[MineQuery](https://github.com/dreamscached/MineQuery)，主要因为它流畅的调用体验，以及对于高版本GO语言的兼容性。要安装它，你可以在适当的位置运行

```bash
go get -u "github.com/dreamscached/minequery/v2"
```

然后，你可以打开需要编程的.go文件，通过 `minequery.Ping*()`函数 (`*`代表协议使用的版本)来自由地获取Minecraft服务器的数据。有关更多用法，你可以访问该项目的Github Repo.

## 我的例子

这里提供使用例一则，把这个函数绑定到某个HTTP服务，作为其回调接口，请求这个接口就可以获得Ping包返回的JSON信息（原封不动）

```golang
func MinecraftServerInfoHandler(w http.ResponseWriter, r *http.Request) {
	hostname := r.URL.Query().Get("ip")
	port, err := strconv.Atoi(r.URL.Query().Get("port"))
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		errMessage := McServerQueryError{Err: err, Msg: "Invalid Port Number"}
		msg, _ := json.Marshal(errMessage)
		w.Write(msg)
		return
	}
	res, err := minequery.Ping17(hostname, port)
	if err != nil {
		w.WriteHeader(http.StatusInternalServerError)
		errMessage := McServerQueryError{Err: err, Msg: "Failed to access remote host."}
		msg, _ := json.Marshal(errMessage)
		w.Write(msg)
		return
	}
	tores, _ := json.Marshal(res)
	w.Write(tores)
}
```
