<h1 align="center">OpenClash 保姆级设置教程<br>&<br>个人自用全分组订阅转换模板</h1>

<p align="center">
   <img src="https://img.shields.io/github/stars/Aethersailor/Custom_OpenClash_Rules?style=for-the-badge&logo=github" alt="GitHub stars">
</p>


## 关于本仓库 
可能是目前全网最强的 [OpenClash](https://github.com/vernesong/OpenClash) 保姆级图文教程和订阅转换模板，秒杀一切教程贴！  
终结所有错误设置！让稀奇古怪的套娃设置方法见鬼去吧！  

手把手嘴对嘴指导你将 OpenClash 设置为效率、安全和便利三者兼顾的完美状态，零基础小白也能轻松看懂。  
按照本仓库 [Wiki](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki) 中的教程，搭配本仓库的[订阅模板](https://raw.githubusercontent.com/LIMINGRUIQAQ/Custom_OpenClash_Rules/main/cfg/Custom_Clash.ini)对 OpenClash 进行设置，仅依靠 OpenClash 自身，无需套娃其他工具，即可实现快速、无污染、无泄漏的 DNS 解析以及完善多样的分流功能，同时配合 Dnsmasq 可实现无第三方插件的广告拦截，并且完美兼容 IPv6。  

欢迎 star ！  

## 更新  
本仓库模板包含的规则均为引用的上游规则碎片，上游规则更新与本仓库模板的更新没有直接关系  


2024.8.2
修改模板全英文化， 去除所有Emoji图标，为Glados机场优化模板


## 本仓库教程及订阅转换模板介绍
本仓库的订阅转换模板Fork来自Aethersailod的订阅模板基础上进行了个性化定制。
以下特性涉及的设置需要按照本仓库 [Wiki](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki) 中的教程对 OpenClash 进行配置才可以实现：  
* 无需搭配其他插件，实现 DNS 防泄露；  
* 基于 ACL4SSR_Online_Full 全分组规则魔改，将大部分规则碎片替换成 [blackmatrix7](https://github.com/blackmatrix7/ios_rule_script) 的规则文件，域名分流信息极为全面，增加更多策略组，覆盖大多数日常使用环境，无需自己折腾；  
* 支持节点按地区分类测速优选；  
* 媒体服务（Youtube、Netflix、Disney+ 等）走指定区域测速选优或指定节点，特定网站（电报、ChatGPT 等）走指定区域节点测速选优或指定节点；  
* 单独列出 Steam 规则并强制 Steam 下载 CDN 走直连，解决 Steam 下载 CDN 定位到国外的问题，确保 Steam 下载流量不走代理；  
* 国内域名和 IP 绕过 Clash 内核，提升访问速度和下载性能，并采用运营商 DNS 解析取得最佳解析结果；
* 国外域名和 IP 使用远端节点服务器的 DNS 进行解析，取得最佳解析结果；  
* 国内域名返回真实 IP，国外域名返回 Fake-IP；
* 增加若干冷门域名规则（互动对战平台、猫眼浏览器、蓝点网、EA Desktop 下载 CDN 等），绝无副作用。具体内容详见 Rule\Custom_Direct.list 文件）;  
* 无需手搓配置，每日定时自动更新上游规则，一次设置即可长期无人值守，无需反复折腾；  
* 增加更多的节点区域分组（英国、加拿大等）；    
* 尽力实现海外下载流量强制直连（相关规则完善中）；  
* 广告屏蔽功能（可选）  

## 使用方法  
准备好你的订阅链接，然后按照本仓库 [Wiki 中的图文教程](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-设置教程)对 OpenClash 进行设置程，教程中已包括了本仓库订阅转换模板的使用方法：  
https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-设置教程  
教程非常详尽，只需按部就班设置即可，有手就行！  

此处提供自定义订阅模板的单独下载地址：  
https://raw.githubusercontent.com/LIMINGRUIQAQ/Custom_OpenClash_Rules/main/cfg/Custom_Clash.ini
