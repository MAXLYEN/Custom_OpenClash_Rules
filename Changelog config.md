<h1 align="center">个人自用 OpenClash 全分组订阅转换模板</h1>

<p align="center">
  <a href="https://github.com/MAXLYEN/Custom_OpenClash_Rules/stargazers"><img src="https://img.shields.io/github/stars/MAXLYEN/Custom_OpenClash_Rules?style=flat-square&logo=github" alt="stars"></a>
  <a href="https://github.com/MAXLYEN/Custom_OpenClash_Rules/commits/main"><img src="https://img.shields.io/github/last-commit/MAXLYEN/Custom_OpenClash_Rules?style=flat-square" alt="last commit"></a>
</p>

<p align="center">
  Fork 自 <a href="https://github.com/Aethersailor/Custom_OpenClash_Rules">Aethersailor/Custom_OpenClash_Rules</a>，在其基础上做了个性化定制
</p>

---

## 关于本仓库

一份**个人自用**的 OpenClash 订阅转换模板：全英文分组、无 Emoji 图标，规则碎片主要引用自本人维护的 [Openclash-Rule](https://github.com/MAXLYEN/Openclash-Rule) 规则库，并以 GeoSite / GeoIP 作为兜底补漏。

上游项目提供了非常完整的 OpenClash 设置教程，本模板的使用方式与之一致，设置步骤请参考上游 Wiki：

**[OpenClash 设置教程（上游 Wiki）](https://github.com/Aethersailor/Custom_OpenClash_Rules/wiki/OpenClash-设置教程)**

模板下载地址：

```
https://raw.githubusercontent.com/MAXLYEN/Custom_OpenClash_Rules/refs/heads/main/cfg/Custom_Clash.ini
```

---

## 与上游的差异

| 项 | 上游 | 本模板 |
|---|---|---|
| 分组命名 | 中文 + Emoji | 全英文，无 Emoji |
| 规则来源 | GeoSite / GeoIP 为主 | 自维护规则库为主，GeoSite / GeoIP 兜底 |
| 地区分组 | 6 个 | 8 个常用 + 27 个备用（注释状态，取消注释即可启用） |
| 专项分组 | — | 新增 PT、Instant Messaging、Cryptocurrency、UKNet、EUNet 等 |
| 规则文件 | 域名与 IP 混合 | 按平台成对拆分为 `_Domain` / `_IP` |

---

## 主要特性

部分特性需按上游 Wiki 完成 OpenClash 配置后才能生效。

**DNS 防泄漏** —— 无需搭配其他插件。国内域名返回真实 IP 并用运营商 DNS 解析，国外域名返回 Fake-IP 并用节点服务器 DNS 解析，各自取得最佳解析结果。

**国内流量绕过内核** —— 国内域名和 IP 直连，提升访问速度与下载性能。

**Steam 下载强制直连** —— 单独列出 Steam 规则，并将 `SteamCN`、`Steam_CDN` 排在 `GEOSITE,steam` 之前，解决 Steam 下载 CDN 定位到国外的问题。

**按地区测速优选** —— 节点按地区自动分类并测速选优。正则采用 `\bXX[-_ ]?\d*\b` 词边界写法，可正确匹配 `AU` / `AU1` / `AU-01`，不会把 `AUTO` 误判为澳洲节点、不会把节点名中的 `100GB` 误判为英国节点。

**媒体与专项分流** —— Netflix、Disney+、YouTube、Emby 等媒体服务，以及 Telegram、ChatGPT、Claude 等特定站点，均可指定区域测速选优或固定节点。

**交易所固定落地** —— Binance、Bybit 走欧洲节点，OKX 走 Cryptocurrency 组，避免交易所因出口 IP 归属地变动触发风控。

**PT 独立分组** —— 候选中刻意不包含 `Proxy` 与 `Auto-Test` 两个会自动切换节点的分组，因为 PT 做种与下载需长期固定同一出口 IP，节点漂移可能被 tracker 判定异常。

**规则源分档更新** —— 金融、AI 平台、私人定制规则走 raw 地址、1 小时间隔（无 CDN 缓存，改动即时生效）；其余公共规则走 jsdelivr、8 小时间隔（国内可达性更好，降低软路由负担）。

**广告拦截** —— 配合 Dnsmasq 实现，无需第三方插件。

**完美兼容 IPv6**，规则自动更新，一次设置长期无人值守。

---

## 规则检索顺序

整体遵循**由精确到泛化**：

```
内网直连 → 广告拦截 → 金融 / 虚拟货币 → 国内直连
  → 平台专属（自维护规则集 + GeoSite 兜底）
  → 泛分类（category-games / category-communication / category-entertainment）
  → 地区兜底（HK / JP）
  → gfw → 国内兜底 → FINAL
```

**GeoSite / GeoIP 作为兜底而非优先。** 平台类 GeoSite 一律排在同组规则集之后，由自维护规则主导匹配。这样做是因为 GeoSite 分类粒度较粗，一个大类常横跨多个专属组 —— 例如 `category-communication` 包含 Telegram、`category-games` 包含 Steam 与任天堂，前置会直接架空这些平台的专属分组。同时 GeoSite 由社区维护、内容变动无通知，前置等于把分流主导权交给外部数据源。

三类例外保持前置：`private` 内网、国内直连类（`google-cn` / `games@cn` / 游戏平台下载 / 公共 tracker）、`gfw` 与 `cn` 全局兜底。

> 模板中带「必须排在…之前」注释的行存在顺序依赖，调整位置前请先阅读该注释。

---

## 规则集引用格式

所有规则集引用 `rules/yaml/` 目录，**必须带 `clash-classic:` 前缀**：

```
ruleset=Netflix,clash-classic:https://.../rules/yaml/Netflix_Domain.yaml,28800
```

两点都不能省：

- **缺少前缀** → 订阅转换按 `domain` 格式解析 classical 规则，带类型前缀的规则被整体丢弃
- **引用 `.list` 而非 `.yaml`** → 带更新间隔的 ruleset 会生成 rule-provider，而 classical provider 要求 payload 为 YAML 数组结构，纯文本解析不出内容，provider 规则数为 0

这两个问题都表现为「规则看似加载成功，实则完全不生效」，且因 GeoSite 兜底而难以察觉。详见 [CHANGELOG.md](CHANGELOG.md)。

---

## 更新记录

模板引用的规则碎片来自 [Openclash-Rule](https://github.com/MAXLYEN/Openclash-Rule)，规则内容的更新与本模板的更新没有直接关系。

**2026.08.05**
- 修复规则集完全不生效的问题：补充 `clash-classic:` 前缀，并将引用切换到 YAML 格式的规则文件
- GeoSite / GeoIP 调整为兜底定位，补全至 41 条分类
- 修正 6 处检索顺序问题，消除 5 处死规则
- 重写节点正则，修复 `AU`→`AUTO`、`GB`→`100GB`、`新`→`新北` 等误匹配
- 规则集按平台拆分为 `_Domain` / `_IP` 成对结构
- 新增 PT、Instant Messaging、Social Media、Talkatone 分组
- 健康检查地址改为 `cp.cloudflare.com/generate_204`

**2026.03.28** 修改并补充分流规则

**2025.01.01** 修改规则顺序，增加分流规则，提供更细化的分流规则

**2024.08.02** 模板全英文化，去除所有 Emoji 图标，为 GLaDOS 机场优化模板

完整变更说明见 [CHANGELOG.md](CHANGELOG.md)。

---

## 相关项目

- [MAXLYEN/Openclash-Rule](https://github.com/MAXLYEN/Openclash-Rule) —— 本模板引用的分流规则库
- [Aethersailor/Custom_OpenClash_Rules](https://github.com/Aethersailor/Custom_OpenClash_Rules) —— 上游项目，提供完整的 OpenClash 设置教程
- [vernesong/OpenClash](https://github.com/vernesong/OpenClash) —— OpenClash 本体
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) —— 部分规则碎片来源

## 致谢

感谢 [Aethersailor](https://github.com/Aethersailor) 的订阅模板与设置教程，本仓库在其基础上定制而成。
