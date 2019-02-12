---
title: Home Assistant 离线通知
date: 2019-01-09 18:26:13
tags:
---
# Home Assistant 离线通知
有时候因为停电或者网络故障，导致 HA 离线了，这时候想收到个通知。其实逻辑挺简单，可以每个一段时间向某个地址或服务请求，如果多久没有请求则发通知提醒。

因为懒，又看到 AWS 上面有 MQTT 的生命周期事件，则打算用上这2个事件来处理。

1. MQTT 某个客户端断开
`$aws/events/presence/disconnected/clientId`
2.  MQTT 某个客户端连接事件
`$aws/events/presence/connected/clientId` 

> [主题： - AWS IoT](https://docs.aws.amazon.com/zh_cn/iot/latest/developerguide/topics.html)

## 在 IoT 创建行为规则
先创建对应 TOP 的规则查询语句，可以使用 WHERE 条件直接查询 TOP 里面的 JSON 内容。使用`#`作为 Top 的匹配，可以获取到任意 client_id。
`SELECT * FROM '$aws/events/presence/disconnected/#' WHERE clientId = 'basicPubSub'`

然后添加个操作，将消息发送为 SNS 推送通知，这里我提前创建了 SNS 的主题，然后添加了 Email 的订阅。

![B95A4CAD-EFE3-47B0-9A4F-358F9AD500B4](/images/B95A4CAD-EFE3-47B0-9A4F-358F9AD500B4-1.png)



> 在指定将消息发送为 SNS 推送通知的时候，需要选择对应角色去执行 SNS 的 publish，所以选择的角色需要把对应 SNS 的 publish 权限添加上。Top 接收到的数据为 
> `{“clientId”:”basicPubSub”,”timestamp”:1547889064020,”eventType”:”connected”,”sessionIdentifier”:”xxx”,”disconnected”:”xxx”,”versionNumber”:8}
`