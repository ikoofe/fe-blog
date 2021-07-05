---
title: WebRTC 学习总结
date: 2021-07-05 19:47:30
tags: WebRTC
---

## 介绍

Google 推出的 WebRTC 实现了浏览器快速的实时通信，它允许网络应用或者站点在不借助中间媒介的情况下，建立浏览器之间点对点 (Peer-to-Peer) 的连接，实现视频流和音频流或者其他任意数据的传输。WebRTC 包含的这些标准使用户在无需安装任何插件或者第三方的软件的情况下，使创建点对点 (Peer-to-Peer) 的数据分享和电话会议成为可能。

其中有三个主要的数据结构

- MediaStream: 音视频流的获取
- RTCPeerConnection: 音视频数据的通信
- RTCDataChannel: 除音视频数据之外的任何其它数据

### MediaStream

MediaStream 通过 getUserMedia 获取，表示媒体数据流，包括音频、视频。

```JavaScript
navigator.mediaDevices.getUserMedia({
  audio: true,
	video: {
    width: {min: 1024, ieda: 1280, max: 1920},
    height: {min: 576, ideal: 720, max: 1080},
    frameRate: {max: 30}
  }
}).then(stream => {
  let video = document.createElement('video')
  document.body.append(video)
  video.srcObject = stream
  video.onloadedmetadata = function() {
    video.play()
  }
})
```

### RTCPeerConnection

可以理解为它是一个功能超强的 socket，RTCPeerConnection 与普通的 socket 一样，在通话的每一端都至少有一个 RTCPeerConnection 对象。在 WebRTC 中它负责与各端建立连接，接收、发送音视频数据，并保障音视频的服务质量。

**适配各种浏览器**

一般情况下，还会在显示页面中添加一个叫做 adapter.js 的脚本，它的作用是为各种浏览器都提供统一的、最新的 WebRTC API 接口。

```HTML
...
<script src="https://WebRTC.github.io/adapter/adapter-latest.js"> </script>
...
```

**RTCPeerConnection 如何工作**

- 为连接的每个端创建一个 RTCPeerConnection 对象，并且给 RTCPeerConnection 对象添加一个本地流，该流是从 getUserMedia() 获取的；

- 获取本地媒体描述信息，即 SDP 信息，并与对端进行交换；

- 获得网络信息，即 Candidate（IP 地址和端口），并与远端进行交换。

  例子见 [RTCPeerConnection例子](https://github.com/avdance/WebRTC_web/tree/master/12_peerconnection)

###  RTCDataChannel

非音视频数据都是通过 RTCDataChannel 进行传输，是 WebRTC 中专门用来传输非音视频数据的类，RTCDataChannel 支持的数据类型也非常多，包括：字符串、Blob、ArrayBuffer 以及 ArrayBufferView。数据通道 (`DataChannel`) 接口表示一个在两个节点之间的双向的数据通道，该通道可以设置成可靠传输或非可靠传输

RTCDataChannel 创建方式如下：

```JavaScript
...
var pc = new RTCPeerConnection(); // 创建 RTCPeerConnection 对象
var dc = pc.createDataChannel("dc", options); //创建 RTCDataChannel对象 参数1: 标签名称 参数2: 配置项
...
```

options 配置项

- ordered： 消息的传递是否有序
- maxPacketLifeTime：重传消息失败的最长时间
- maxRetransmits：重传消息失败的最大次数
- protocol：用户自定义的子协议，默认为空
- negotiated：如果为 true，则会删除另一方数据通道的自动设置。这也意味着你可以通过自己的方式在另一侧创建具有相同 ID 的数据通道
- id：当 negotiated 为 true 时，允许你提供自己的 ID 与 channel 进行绑定

除了上面介绍的三个对象，在 WebRTC 中还有两个重要过程，就是媒体协商和网络协商。

## 媒体协商

媒体协商就是看看你的设备都支持哪些编解码器，我的设备是否也支持？如果我的设备也支持，那么咱们双方就算协商成功了。

### SDP

SDP（Session Description Protocol）是一种会话描述协议，用来描述多媒体会话，主要用于协商双方通讯过程，传递基本信息。

SDP的为多行\<type>=\<value>的结构
- type:字符，代表一个特定属性，比如v代表版本
- value: 属性值，结构与type有关

例如：

```JavaScript
v=0 // sdp版本号，一直为0
o=- 3409821183230872764 2 IN IP4 127.0.0.1 // 会话发起者描述信息
...
m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 110 112 113 126 // 音频流信息
...
```

### 如何协商

- 首先，通信双方将它们各自的媒体信息，如编解码器、媒体流的 SSRC、传输协议、IP 地址和端口等，按 SDP 格式整理好。
- 然后，通信双方通过信令服务器交换 SDP 信息，并待彼此拿到对方的 SDP 信息后，找出它们共同支持的媒体能力。
- 最后，双方按照协商好的媒体能力开始通信。

## 网络协商

在了解了彼此的媒体能力后，接下来需要确认的是双方的网络通信能力。只有彼此要了解对方的网络情况，这样才有可能找到一条相互通讯的链路。

我们假设一个例子：

通信的双方我们称为 A 和 B；A 为呼叫方，B 为被呼叫方；C 为中继服务器，也称为 relay 服务器或 TURN 服务器。

一种连接方式是我们通过p2p的方式直接进行连接，即A->B,b->A；二是通过中继服务器进行中转， A->C， C->B， B->C， C->A，对于WebRTC，会优先选择第一种，如果不通，再选择第二种。

在 WebRTC 建立通讯时，会使用 ICE Candidate。

**Candidate**: WebRTC 与远端通信时使用的协议、IP 地址和端口，分为下面三种类型：

- host 类型，即本机内网的 IP 和端口；
- srflx 类型, 即本机 NAT 映射后的外网的 IP 和端口；
- relay 类型，即中继服务器的 IP 和端口。

**ICE**: 获取各种类型 Candidate 的过程,即在本机收集所有的 host 类型的 Candidate，通过 STUN 协议收集 srflx 类型的 Candidate，使用 TURN 协议收集 relay 类型的 Candidate， 它整合了 `STUN` 和 `TURN`。

### NAT (Net Address Transport)

**作用**

进行内外网的地址转换，当我们访问公网资源，NAT先将内网地址转换为外网地址，发送请求给要访问的服务器，服务器将结果返回给公网地址+端口， 再通过NAT最终转给内网主机

**背景**

- 解决IPv4地址不够用的问题，让多台主机共用一个公网 IP 地址，然后在内部使用内网 IP 进行通信，这种方式大大减缓了 IPv4 地址不够用的问题
- 解决安全问题，主机隐藏在内网，外面有 NAT 挡着，这样的话黑客就很难获取到该主机在公网的 IP 地址和端口，从而达到防护的作用

**种类**

NAT 种类分为完全锥型 NAT、IP 限制锥型、端口限制锥型和对称型。以完全锥形举例：

![](https://static001.geekbang.org/resource/image/88/af/8836f91edfcc9a2420e3fd11098f95af.png)

完全锥形的特点是当 host 主机通过 NAT 访问外网的 B 主机时，就会在 NAT 上打个“洞”，所有知道这个“洞”的主机都可以通过它与内网主机上的侦听程序通信。所谓的“打洞”就是在 NAT 上建立一个内外网的映射表，形如下结构：

```
{ 内网IP， 内网端口， 映射的外网IP， 映射的外网端口}
```

如果 host 主机与 B 主机“打洞”成功，且 A 与 C 从 B 主机那里获得了 host 主机的外网 IP 及端口，那么 A 与 C 就可以向该 IP 和端口发数据，而 host 主机上侦听对应端口的应用程序就能收到它们发送的数据。

IP 限制锥形比完全锥形多了个 IP 限制，只有登记了的 IP 地址才能通过。端口限制锥型比IP限制锥形多了个端口限制。而对称型与端口限制型 NAT 最大的不同在于，如果 host 主机访问 A 时，它会在 NAT 上重新开一个 “洞”，而不会使用之前访问 B 时打开的 “洞”。也就是说对称型 NAT 对每个连接都使用不同的端口，甚至更换 IP 地址，而端口限制型 NAT 的多个连接则使用同一个端口，这对称型 NAT 与端口限制型 NAT 最大的不同。

### STUN (Session Traversal Utilities for NAT)

当你要访问公网上的资源时，NAT 首先会将该主机的内网地址转换成外网地址，然后才会将请求发送给要访问的服务器；服务器处理好后将结果返回给主机的公网地址和端口，再通过 NAT 最终中转给内网的主机, STUN 做的事情就是：拿到公网的 IP + 端口。

上面的四种NAT类型，有三种可以进行 NAT 穿透，即完全锥型、IP 限制型、端口限制型，但是一般大公司会采用对称型，也就是说，路由器只会接受你之前连线过的节点所建立的连线。这类网络就需要 TURN 技术。

### TURN (Traversal Using Relays around NAT)

TURN 是 `STUN/RFC5389` 的一个拓展, 使得NAT绕过了对称型NAT的限制。

## 媒体协商 + 网络协商数据的交换通道

### 信令

信令是在两个设备之间发送控制信息以确定通信协议、信道、媒体编解码器和格式以及数据传输方法以及任何所需的路由信息的过程。

### 信令服务器

2 个客户端协商媒体信息 (`SDP`) 和网络信息 (`candidate`) 交换，就需要一个服务器，我们称之为信令服务器，真实应用中还会处理房间管理 、人员进出房间等功能。

## WebRTC 实战

### 流程图

![](https://p3.koolearn.com/6f925f30.png)

```JavaScript
...
// 创建PeerConnection
let peer = new PeerConnection(iceServer);
peer.addStream(this.localStream);
...
...
createOffer(account, peer) {
  // 创建offer
  peer.createOffer({
      offerToRecieveAudio: 1,
      offerToReceiveVideo: 1
  }).then((desc) => {
      // offer创建成功后执行setLocalDescription
      peer.setLocalDescription(desc, () => {
          socket.emit('offer', {'sdp': peer.localDescription, roomid: this.$route.params.roomid, account: account})
      })
  })
},
...

...
socket.on('offer', v=> {
  //远端 pc 将 offer 保存起来
  this.peerList[v.account] && this.peerList[v.account].setRemoteDescription(v.sdp, () => {
    // 远端pc创建answer
      this.peerList[v.account].createAnswer().then((desc) => {
        // 远端保存answer
          this.peerList[v.account].setLocalDescription(desc, () => {
              socket.emit('answer', {'sdp': this.peerList[v.account].localDescription, roomid: this.$route.params.roomid, account: v.account})
          })
      })
  }, (err) => {
      console.log(err)
  })
})
...
...
//本端 pc 保存 answer
socket.on('answer', v=> {
    this.peerList[v.account] && this.peerList[v.account].setRemoteDescription(v.sdp, function() {}, (err) => {
        console.log(err)
    })
})
...
...
socket.on('__ice_candidate', v=> {
    if(v.candidate){
      // addIceCandidate
        this.peerList[v.account] && this.peerList[v.account].addIceCandidate(v.candidate).catch(() => {})
    }
})
...
...
peer.onaddstream = function (event) {
  // 拿到数据流进行渲染处理
  console.log(event.stream)
};
...
...
peer.onicecandidate = (event) => {
    if(event.candidate) {
        socket.emit('__ice_candidate', {
            'candidate': event.candidate, roomid: this.$route.params.roomid, account: v.account
        })
    }
}
...
```

## WebRTC 数据统计

在 Chrome 浏览器下输入 `chrome://WebRTC-internals` 这个 URL 就可以看到所有的统计信息了

## 注意事项

如果只是本地局域网测试无需搭建 stun/turn 服务器，但是如果要外网访问，那你需要一台云主机&支持 https 的域名，你可参考这篇文章进行搭建：[WebRTC信令控制与STUN/TURN服务器搭建](https://juejin.cn/post/6844903844904697864)

## 代码示例

https://github.com/xieyanxieyan/WebRTC-demo/tree/master

## 参考文章

[STUN/TURN服务搭建](https://juejin.cn/post/6844903844904697864)

[WebRTC](https://hpbn.co/webrtc/)

[WebRTC API](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)

[极客时间：从 0 到 1 打造音视频系统](https://time.geekbang.org/column/article/107948)
