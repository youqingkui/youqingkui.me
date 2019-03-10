---
title:  HA 升级0.8.8后的automation time 错误
date: 2019-03-10 17:17:51
tags:
---



前端时间看到Home Assistant 的更新日志，增加了smart tings的支持。就更新到了最新的0.88版本。后面发现在一些自动化没有执行了，通过设置里面的配置检查，发现提示`Invalid config for [automation]: [minutes] is an invalid option for [automation].`错误。

后面通过搜索才知道，platform 的时间控制拆分了2种。如果要使用hours ，minutes 或者seconds，需要把time 改为 time_pattern。如果还是需要具体的某个时间只需，如使用了at，依旧使用time时没有问题的。

![60CD80CA-FD08-4665-A1E8-4EFD9ED8E3E1](/images/60CD80CA-FD08-4665-A1E8-4EFD9ED8E3E1.png)



```
- id: "72"
  alias: "空净:23:30睡眠模式"
  trigger:
    platform: time
    at: '23:30:00'
  condition:
    condition: state
    entity_id: fan.xiaomi_air_purifier_2s
    state: 'on'
  action:
   - service: script.air_silent_model
```

下面就是 time_pattern的使用
```
### 空调
- id: "61"
  alias: "空调:离家10分钟关"
  trigger:
    platform: time_pattern
    minutes: '/10'
    seconds: 00
  condition:
    condition: and
    conditions:
    - condition: state
      entity_id: device_tracker.rendandeiphone
      state: 'not_home'
    - condition: state
      entity_id: device_tracker.youqingkui_note9
      state: 'not_home'
    - condition: state
      entity_id: binary_sensor.airconditioning
      state: 'on'
  action:
    - service: script.close_airconditioning
```

另外自动化配置的文档里面的使用方法也变了。[Automation Trigger - Home Assistant](https://www.home-assistant.io/docs/automation/trigger/)

> 另外还发现了新版本个坑，我`known_devices.yaml` 里面配置 youqingkui-note9，在`device_tracker`组建里面的名字是`youqingkui_note9`了。变成下划线了，导致我一些自动化配置无法执行，因为识别不到。  
