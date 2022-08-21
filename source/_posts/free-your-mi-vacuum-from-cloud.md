---
title: 小米扫地机器人反隐私搜集改造
date: 2019-06-23 22:32:19
tags: [智能家居]
category: [消费电子]
lede: 让你的小米扫地机器人脱离米家APP工作
---
&emsp;&emsp;米家系列产品以其廉价但生态丰富的解决方案为消费者带来了甜品级的智能家居体验。在享受其精致的硬件产品的同时，对伴随而来的品牌生态绑架和不平等用户使用条款，我们亦有权利与技术手段对它们说不。越来越多的人选择使用开放智能家居平台来接管各种物联网产品，在有效防止商业或政治用途的隐私数据采集隐患，同时也让多品牌产品之间的功能联动成为可能。  
&emsp;&emsp;本文作为以[Home Assistant](https://www.home-assistant.io/)为核心构建高客制化程度、隐私保护护级别和设备选型灵活性的智能家居系统的一部分，对小米扫地机器人及其后续机型提供了一种不同于[HA官方集成](https://www.home-assistant.io/)的接入方式，在保留绝大部分原厂功能的前提下完全阻断向小米公司广域网服务器的数据传输。  

## 支持型号
1. Xiaomi Vacuum Gen1  
2. Roborock Vacuum Gen2 S50/S51/S55  

## 前期准备工作
1. 获取米家设备token  
与HA官方集成步骤一致，我们需要通过`xiaomi_miio`接管设备的数据读写，安装步骤本例程不再赘述。
* 对于安卓用户，下载修改版的[米家App](https://cloud.mail.ru/public/33J3/t817u4Snr)，按照常规方式进行配对连接后，在应用程式里选中扫地机器人，进入菜单配置/常规设置/网络信息选项卡查看当前token(更快捷)
* 对于PC用户，可以遵循[这篇文章](https://www.home-assistant.io/components/vacuum.xiaomi_miio/#retrieving-the-access-token)跳过常规方式配对连接直接获取token(更安全)  
2. 在路由器中固化扫地机器人IP地址  
为方便后期调试与Home Assistant接入，建议在路由器中设置不变的IP地址并做ARP映射。  

&emsp;&emsp;如果你完成了上述步骤，请在纸上记下扫地机器人的token和IP地址直到它下一次系统重置。

## 将扫地机器人固件更新为国际版
&emsp;&emsp;此步骤非必要。但考虑到消费电子产品在中国大陆地区与其他地区的用户使用条款之间的巨大差异，即使不进行进一步的Hack，刷写国际版固件的设备也将使用户享受到诸如[GDPR](https://eugdpr.org/)等政策的保护，推荐遵循[这篇文章](https://xiaomirobot.wordpress.com/transformer-un-modele-china-edition-vers-international/)执行转区操作。  

## 自制固件  
1. 下载特定版本的国际版固件作为修改基础  
* [小米机器人1代固件](https://cdn.awsbj0.fds.api.mi-img.com/updpkg/v11_003468.fullos.pkg)  
* [石头扫地机器人2代(S5x)固件](https://cdn.awsbj0.fds.api.mi-img.com/rubys/updpkg/v11_001886.fullos.pkg)  

2. 使用Valetudo重构固件  
遵循[这篇文章](https://github.com/Hypfer/Valetudo/wiki/Installation-Instructions)将root权限的ssh登录密钥和带有高级功能的WebUI编入固件并刷写。

3. 定制Valetudo  
可以通过[社区镜像生成网站](https://builder.dontvacuum.me/)快速获取定制固件。  

## 语音客制化
如果想让扫地机器人语音更换为特定国家，可以在[这里](https://dustbuilder.xvm.mit.edu/pkg/voice/)下载对应语种的语音包并刷写

```
mirobo --ip [vacuum_ip] --token [your_token] install-sound /path/to/xx.pkg
```

## 感受
&emsp;&emsp;物联网的发展正在消费者与企业之间日趋不对称的技术力与情报量下逐渐失去控制。万物联网的下一个10年，消费者应当关心的不再只有商品的物质价值，更多的是其背后巨大信息量带来的种种权宜。当代人过分看轻自己所产生的信息的价值，在远高于大脑处理能力的信息流中迷失自我，使得赛博科幻作品中的担忧逐渐成为现实。而在下一次更有人性和温度的信息技术革命来临之前，隐私保护更像是一种自发的的止损行为，尽管从旁观者角度看起来是那么地徒劳。  
