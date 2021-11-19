---
title: 使用 Wechaty 实现微信聊天机器人
date: 2021-11-12 19:14:18
tags: Wechaty
---


## 背景

众所周知，在钉钉的聊天群里，实现机器人推送消息，是一件很简单的事情；但是在微信中这是一件很困难的事情，因为微信本身并不支持这个功能。那么，我们能否实现一个微信聊天机器人呢？答案是可以的，但是要借助一些工具，本文主要介绍使用 [Wechaty](https://wechaty.js.org/) 实现微信聊天机器人。

![](https://camo.githubusercontent.com/8663c7fa27f9849e4002ca4d8f4b032c420e0ecce53757c2f6c1859a250561ee/68747470733a2f2f776563686174792e6a732e6f72672f696d672f776563686174792d6c6f676f2e737667)


Wechaty 是一个聊天机器人开源项目，提供了开发聊天机器人的 SDK，方便开发人员快速实现聊天机器人。通过 Wechat 可以获取到微信的聊天内容、联系人、群组、好友关系等信息，也可以实现创建群组、发送消息等功能。

![](https://wechaty.js.org/assets/images/architecture-ef4e78c0bf9d9b359328f3de8751ebb1.png)

## 实现

Wechaty 拥有了操作微信的能力，相当于在微信的基础上做了一层封装，通过它提供的 API 可以很方便的实现聊天机器人。在写代码之前，需要安装下面几个 npm 包:

- `wechaty` 
- `wechaty-puppet-wechat` 使用网页版微信
- `qrcode-terminal` 支持二维码在控制台中展示

首先，要建一个聊天机器人。Wechaty 会启动一个 Puppeteer 并打开网页版微信，然后使用手机上的微信 APP 扫描 Wechaty 提供的二维码即可完成登陆。登陆成功之后，我们就可以通过 Wechaty 进行各种操作了。

```JavaScript
const qrTerm = require('qrcode-terminal');
const Wechaty = require('wechaty');

const { ScanStatus, WechatyBuilder, log } = Wechaty;

function onScan (qrcode, status) {
  if (status === ScanStatus.Waiting || status === ScanStatus.Timeout) {
    qrTerm.generate(qrcode, { small: true });  // show qrcode on console

    const qrcodeImageUrl = [
      'https://wechaty.js.org/qrcode/',
      encodeURIComponent(qrcode),
    ].join('');

    log.info('StarterBot', 'onScan: %s(%s) - %s', ScanStatus[status], status, qrcodeImageUrl);
  } else {
    log.info('StarterBot', 'onScan: %s(%s)', ScanStatus[status], status);
  }
}

// get a Wechaty instance
const bot = WechatyBuilder.build({
  name: 'wechat-bot',
  puppet: 'wechaty-puppet-wechat',
})

// emit when the bot needs to show you a QR Code for scanning
bot.on('scan', onScan);

// start the bot
bot.start()
  .then(() => log.info('StarterBot', 'Starter Bot Started.'))
  .catch(e => log.error('StarterBot', e))
```
然后，实现消息发送功能。在微信聊天机器人成功启动之后，可以用机器人发送消息，代码如下：

```JavaScript
// find a room
const room = await bot.Room.find({ topic: 'yours-wechat-group-name' })
if (room) {
  // send a message
  await room.say(`hello world`)
}
```

不难发现，使用 Wechaty 实现微信聊天机器人是很方便的，但要注意以下两点：
- Webchaty 需要 Node.js 的版本在 16+ 版本，如果低于 16 版本的话会报错。
- 相关 npm 包下载缓慢时，可以加如下设置：
  
  ```Bash
  npm config set puppeteer_download_host "https://npm.taobao.org/mirrors"
  npm config set sharp_binary_host "https://npm.taobao.org/mirrors/sharp"
  npm config set sharp_libvips_binary_host "https://npm.taobao.org/mirrors/sharp-libvips"
  ```

## 总结

由于私域流量这个概念逐渐被人熟知，基于个人微信的社群管理工具拥有广泛的应用场景，Webchaty 为我们提供了一种解决问题的途径，能够更加轻松的实现相关的业务功能。